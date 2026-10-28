# PCODE86.2 — two-accumulator p-code spec (draft, 2026-10-22)

Target of the Small-C compiler. Replaces P86.1 (pure stack machine, kept for reference). Status: shape agreed 2026-10-22; details below still open where marked.

## Lineage
- **Cain, Small-C 1.1 (DDJ 1980):** primary/secondary register model; code generator = ~45 C functions emitting 8080 text; no IR.
- **Hendrix, Small-C 2.x:** p-codes with a template table (`cc4.c`); on the 8086 primary = AX, secondary = BX (`ADD12` → `add ax,bx`).
- **Yuen, RCSC (DDJ Aug 1997):** text p-code file → peephole on p-codes → GNU M4 macros per target. 109 p-codes, operand kind spelled in the name (`GETw1s`, `POINT2s`).
- **PCODE86.2:** Yuen's pipeline, but (1) operand kind is an **argument** (mode), not part of the name — 33 mnemonics instead of 109; (2) the expander is Jeremy's own small C program, not M4.

## Pipeline
```
C source → compiler → .p (p-code text) → peephole (p-code → p-code) → expander (table + special cases) → .asm (A86) → .COM
```
The peephole knows nothing about the 8086; the expander knows nothing about C.

## Machine model
- 16-bit words, little-endian. One flat 64K byte-addressed space (tiny model); the stack lives in it.
- Stack grows down.
- Registers:
  - `P` primary — accumulator and return value (→ `AX`)
  - `S` secondary — left operand of binary ops, address for stores (→ `BX`)
  - `SP` (→ `SP`), `FP` (→ `BP`), `PC`
- All arithmetic wraps mod 65536.

## Notation
```
PUSH(x)       SP = SP − 2; W[SP] = x
POP           r = W[SP]; SP = SP + 2
W[x], B[x]    word (little-endian) / byte at address x
s(x)          x read as signed
sext(b)       byte b sign-extended to a word
v(m)          value of operand m;  a(m)  its address
```

## Operand modes
| Mode | Written | v(m) | a(m) | 8086 |
|---|---|---|---|---|
| none | `OP` | S (see binary ops) | S | `bx` / `[bx]` |
| N | `OP N n` | n (number or label) | — | `n`, `offset _sym` |
| L | `OP L n` | W[FP + n] | FP + n | `[bp+n]` |
| G | `OP G sym` | W[sym] | sym | `[_sym]` |
| I | `OP I` | W[P] | P | `[bx]` after `mov bx,ax` |

n is signed for L (locals negative, args positive). Not every opcode accepts every mode — see the Modes column.

## Opcodes (33 core)

### Data (7)
| Op | Modes | Operation |
|---|---|---|
| LD m | N L G I | P ← v(m) |
| LDB m | L G I | P ← sext(B[a(m)]) |
| LEA L n | L | P ← FP + n. (Global addresses: `LD N sym`.) |
| ST m | none L G | W[a(m)] ← P; P unchanged. none: address in S |
| STB m | none L G | B[a(m)] ← P & 0xFF; P ← sext(P & 0xFF) |
| PUSH | — | PUSH(P) |
| POP | — | S ← POP |

### Arithmetic and compare (17)
Binary ops take a left and a right operand:
- no operand: left = S, right = P (Cain: left was pushed, popped into S)
- mode N L G: left = P, right = v(m)

Result always in P. S unchanged.

| Op | Result |
|---|---|
| ADD SUB MUL | left + − × right (mod 65536); MUL serves signed and unsigned |
| DIV | signed, truncates toward 0; TRAP if right = 0 or (left = 0x8000 and right = 0xFFFF) |
| MOD | signed remainder, sign of dividend; traps as DIV |
| DIVU MODU | unsigned; TRAP if right = 0 |
| AND OR XOR | bitwise |
| SHL | right ≥ 16 ? 0 : left << right |
| SHR | logical; right ≥ 16 ? 0 : left >> right |
| SAR | arithmetic; right ≥ 16 ? (s(left) < 0 ? 0xFFFF : 0) : sign-filled |
| EQ | left = right ? 1 : 0 |
| LT | s(left) < s(right) ? 1 : 0 |
| LTU | left < right ? 1 : 0 — pointers |
| NEG | P ← 0 − P (no operand) |

### Control (9)
| Op | Operation |
|---|---|
| JMP L | PC ← L |
| JZ L | if P = 0: PC ← L. P unchanged |
| JNZ L | if P ≠ 0: PC ← L. P unchanged |
| CALL L | PUSH(PC); PC ← L |
| CALLP | PUSH(PC); PC ← P — function pointers |
| ENTER n | PUSH(FP); FP ← SP; SP ← SP − n |
| RET | SP ← FP; FP ← POP; PC ← POP. P (return value) preserved |
| ADJ n | SP ← SP + n. P and S preserved |
| SYS n | runtime service n; result in P |

After `CALL`, `CALLP` and `SYS`: P = result (undefined after a void function), **S undefined**. The compiler never holds a live value in S across a call (S is always loaded by `POP` immediately before its use).

### Optional fused ops (4) — peephole targets only
Not emitted by the compiler; added when a peephole rule needs them.
| Op | Operation |
|---|---|
| CMP m | compare left with right (binary-op operand rules); sets condition state; P, S unchanged |
| Jcc cc,L | if condition cc held at the last CMP: PC ← L. cc ∈ EQ NE LT LE GT GE LTU LEU GTU GEU. Must immediately follow CMP |
| INC m / DEC m | none: P ← P ± 1; L G: W[a(m)] ← W[a(m)] ± 1, P unchanged |

## Design decisions
- **P is the return register** (Cain's HL, Hendrix's AX). No `R`, no `PUSHR`, no `RETV`.
- **Stores leave the value in P** (assignment is an expression). No `DROP` needed: P is simply overwritten.
- **STB leaves the byte actually stored, sign-extended** in P (as P86.1): `i = c = 300` gives 44.
- **Plain `char` is signed.** Unsigned-char load: `LDB m`, `AND N 255`.
- **Shift counts ≥ 16 are defined.**
- **`~x`** is `XOR N -1` (expander may emit `not ax`). `GT GE LE NE` are lowered (cheat sheet).
- Text IR, so no binary encoding (P86.1's bit-7 scheme is dropped).

## Frame layout and calls
Unchanged from P86.1:
```
FP+6   second arg
FP+4   first arg
FP+2   return address
FP+0   caller's FP
FP-2   first local
FP-n   last local     ← SP after ENTER n
```
Args pushed right to left (`<arg>`, `PUSH` each), caller pops with `ADJ`. Varargs need nothing extra.

Worked call, `x = add(1, 2);` with `int add(int a, int b) { return a + b; }`:
```
add:  ENTER 0
      LD L 4
      ADD L 6
      RET

      LD N 2
      PUSH
      LD N 1
      PUSH
      CALL add
      ADJ 4
      ST L -2        ; x
```

## Lowering cheat sheet (C → PCODE86.2)
Before the peephole — what the compiler emits. `{x}` = code leaving x in P.
```
a op b        {a}, PUSH, {b}, POP, OP          ; S = a, P = b
x = e         {e}, ST L n      / ST G sym      ; simple lvalue: no address in a register
*p = e        {p}, PUSH, {e}, POP, ST          ; address in S
a += b        {a}, PUSH, {b}, POP, ADD, ST L n ; simple lvalue
*p += b       {p}, PUSH, LD I, PUSH, {b}, POP, ADD, POP, ST
x++           LD L n, ADD N 1, ST L n, SUB N 1
-x            {x}, NEG
~x            {x}, XOR N -1
!x            {x}, EQ N 0
a != b        {a}, PUSH, {b}, POP, EQ, EQ N 0
a > b         {b}, PUSH, {a}, POP, LT          ; b < a
a <= b        {b}, PUSH, {a}, POP, LT, EQ N 0
a >= b        {a}, PUSH, {b}, POP, LT, EQ N 0
uchar load    LDB m, AND N 255
a || b        {a}, JNZ T, {b}, JNZ T, LD N 0, JMP E; T: LD N 1; E:
a && b        {a}, JZ F, {b}, JZ F, LD N 1, JMP E; F: LD N 0; E:
```
- Simple lvalues (plain local or global) keep their address out of registers — Cain's `lval` trick, generalised. Only `*p`, `a[i]` and the like materialise an address.
- In `if`/`while` conditions a trailing `EQ N 0` disappears — pick `JZ` or `JNZ`.
- A signed divide by 2 must **not** become `SAR` (SAR rounds toward −∞, DIV toward 0).

## Peephole
Pattern → replacement over the p-code stream; no 8086 knowledge.

Core rule — one rule for every binary op (ADD … LTU, and CMP):
```
PUSH ; LD m x ; POP ; OP     →     OP m x        (m ∈ N L G; not I)
```
Correct for non-commutative ops: S was the old P (left), and `OP m x` computes P op x.

Candidates (each needs P dead after the jump, on both paths):
```
EQ N 0 ; JZ L      →  JNZ L
EQ N 0 ; JNZ L     →  JZ L
LT m x ; JZ L      →  CMP m x ; Jcc GE,L       (and the other compares)
```
@todo: rule format (table vs hand-written C); how the peephole knows P is dead (simplest: only inside condition contexts the compiler marks).

## Expander
Table of (op, mode) → A86 template, plus small per-op C functions where one fixed template can't do it:
- `MUL DIV MOD DIVU MODU` with N: no `imul imm` on the 8086 — load the constant into a register first. DIV/MOD use DX (`cwd` for signed).
- Shifts with N: n = 1 → single shift; otherwise via `CL`. Variable shifts: count into `CL`.
- `LDB`: load into `AL`, then `cbw` (AL only).
- `STB` none: `mov [bx],al`.
- Mode I: `mov bx,ax` first (AX can't address memory).
- `JZ`/`JNZ`/`Jcc`: 8086 conditional jumps are ±128 bytes only — invert over a near `JMP`, or confirm A86 lengthens them.
- A86: mangle C names with `_`; `LD N sym` → `mov ax,offset _sym`; unique labels from a counter.
- `SYS n` → `call __sysN` (stubs as in P86.1).

## SYS services
As P86.1 (args on the stack, first arg on top; result in P; caller `ADJ`s):
```
0 EXIT  (code)            → INT 21h/4Ch   does not return
1 PUTC  (c)               → INT 21h/02h
2 GETC  → c               → INT 21h/08h
3 OPEN  (name, mode) → fd → INT 21h/3Dh
4 CREAT (name) → fd       → INT 21h/3Ch
5 CLOSE (fd)              → INT 21h/3Eh
6 READ  (fd, buf, n) → n  → INT 21h/3Fh
7 WRITE (fd, buf, n) → n  → INT 21h/40h
```

## Program structure
Directives as P86.1: `.CODE .DATA .BSS`, `DB`, `DW`, `RESB n`, labels `name:`. BSS emitted as real zero bytes in v1.

Start-up and hello world:
```
        .CODE
        CALL main
        PUSH
        SYS 0           ; EXIT(main's result)
main:   ENTER 0
        LD N 15         ; n      (args right to left)
        PUSH
        LD N msg        ; buf
        PUSH
        LD N 1          ; fd = stdout
        PUSH
        SYS 7           ; WRITE
        ADJ 6
        LD N 0
        RET
        .DATA
msg:    DB 'Hello, world!',13,10
```
Compiler emits `LD N 0, RET` before `main`'s closing brace (C99 5.1.2.2.3).

## Testing
- Reference interpreter for PCODE86.2 on the host (two registers + stack, small) as the oracle.
- Differential: each test program run (1) in the interpreter, (2) expanded → A86 → .COM in DOSBox; outputs compared. Peephole on/off must give identical output.

## Not in v1
`long`, `float`, struct-by-value, `switch` jump tables (compare chains for now).

## Open questions
- Exact text syntax of a p-code line (mode letter vs symbol, comma for `Jcc cc,L`).
- Whether CMP/Jcc and INC/DEC go in v1 or wait for measurements.
- Peephole rule format and the "P dead" test.
- STB's sign-extend of P: keep P86.1's rule, or leave P unchanged (cheaper on the 8086: `mov [bx],al` leaves AX alone).
