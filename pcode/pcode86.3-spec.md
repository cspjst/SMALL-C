# PCODE86.3 — 8087-style register stack, two deep (draft, 2026-10-22)

Target of the Small-C compiler. Alternative to PCODE86.2 (two accumulators, explicit PUSH/POP); both to be compared on the same test programs before one is chosen.

## Idea
- The IR is a **pure stack machine** (as P86.1) with the **8087's instruction forms**: memory operands, pop forms (`ADDP`), reversed forms (`SUBR`), `XCH`, and compare-then-branch (`COM` / `COMP` / `COMPP`, `TST`).
- The abstract stack is **unbounded**. Its top two items are called `ST0` and `ST1`; the translator keeps them in `AX`/`BX` (**static stack caching**, cf. Ertl, "Stack Caching for Interpreters", PLDI 1995). Deeper items live on the hardware stack.
- Expression depth is static, so the translator always knows which items are in registers. Spilling is the translator's job, not the compiler's.
- Lineage: Cain's two registers (P = ST0, S = ST1) + the 8087's discipline + P86.1's stack model.

## Pipeline
```
C source → compiler → .p (p-code text) → (optional peephole) → translator → .asm (A86) → .COM
```
The translator is the PCODE86.2 expander plus three variables of state (see Translation).

## Machine model
- 16-bit words, little-endian. One flat 64K byte-addressed space (tiny model).
- **Evaluation stack E**: unbounded; `ST0` = top item, `ST1` = next. Items below ST1 are not addressable.
- Hardware stack, frame and globals as P86.1; `SP`, `FP`, `PC`.
- **Condition state**: set by `COM`, `COMP`, `COMPP`, `TST`; read by `Jcc`.
- All arithmetic wraps mod 65536.

### Static depth rules
- d = depth of E, known at every instruction.
- d = 0 at function entry.
- At every label, d is the same on all incoming paths (compiler guarantees; a verifier can check).
- `RET` requires d ∈ {0, 1}.
- No IR limit on d; depth > 2 only costs memory traffic.

## Notation
```
push(x)       E gains x on top
pop           E loses its top item
W[x], B[x]    word / byte at address x
s(x), sext(b) signed reading / byte sign-extended to a word
v(m), a(m)    value / address of operand m
```

## Operand modes
| Mode | Written | v(m) | a(m) |
|---|---|---|---|
| N | `OP N n` | n (number or label) | — |
| L | `OP L n` | W[FP + n] | FP + n |
| G | `OP G sym` | W[sym] | sym |

## Opcodes

### Data
| Op | Modes | Operation |
|---|---|---|
| LD m | N L G | push(v(m)) |
| LDB m | L G | push(sext(B[a(m)])) |
| LEA L n | L | push(FP + n). (Global addresses: `LD N sym`.) |
| DUP | — | push(ST0) — the 8087's `FLD ST(0)` |
| LDI | — | ST0 ← W[ST0] (dereference in place) |
| LDBI | — | ST0 ← sext(B[ST0]) |
| ST m | L G | W[a(m)] ← ST0 (ST0 kept) |
| STP m | L G | W[a(m)] ← ST0; pop |
| STB m / STBP m | L G | B[a(m)] ← ST0 & 0xFF; ST0 ← sext(ST0 & 0xFF); STBP then pops |
| STI | — | W[ST1] ← ST0; ST1 removed, ST0 (the value) kept |
| STBI | — | B[ST1] ← ST0 & 0xFF; ST1 removed; ST0 ← sext(ST0 & 0xFF) |
| XCH | — | swap ST0 and ST1 |
| DROP | — | pop |

### Arithmetic — four forms (8087)
Operations: `ADD SUB MUL DIV MOD AND OR XOR SHL SHR SAR DIVU MODU`.

| Form | Example | Operation | 8087 |
|---|---|---|---|
| OP m | `SUB L -6` | ST0 ← ST0 op v(m) | `FSUB m` |
| OPR m | `SUBR N 10` | ST0 ← v(m) op ST0 | `FSUBR m` |
| OPP | `SUBP` | ST1 ← ST1 op ST0; pop | `FSUBP ST(1),ST` |
| OPRP | `SUBRP` | ST1 ← ST0 op ST1; pop | `FSUBRP ST(1),ST` |

- Mode m ∈ N L G. R forms exist only for the non-commutative ops (SUB DIV MOD DIVU MODU SHL SHR SAR).
- `NEG`: ST0 ← 0 − ST0. `~x` is `XOR N -1`.
- Semantics of each op as PCODE86.2:
  - MUL serves signed and unsigned.
  - DIV/MOD signed, truncating toward 0; TRAP on divide by 0 or 0x8000 / −1.
  - DIVU/MODU trap on 0.
  - Shift counts ≥ 16 defined (SHL/SHR → 0; SAR → sign fill).

### Compare (the 8087's COM)
| Op | Operation | 8087 |
|---|---|---|
| COM m | compare ST0 with v(m) | `FCOM m` |
| COMP m | compare ST0 with v(m); pop | `FCOMP m` |
| COMPP | compare ST1 with ST0; pop both | `FCOMPP` — **orientation differs**, see below |
| TST | compare ST0 with 0 (= `COM N 0`) | `FTST` |

- "Compare x with y" records x − y for the following `Jcc`.
- **COMPP takes postfix order:** `a b COMPP` compares a with b. The 8087's FCOMPP compares ST(0) with ST(1); here left = ST1 so lowering reads naturally.
- One compare serves signed and unsigned — signedness is chosen by the `Jcc`, as on the 8086.

### Control
| Op | Operation |
|---|---|
| JMP L | jump |
| Jcc L | jump if the condition holds. cc as the 8086: `JE JNE` · signed `JL JLE JG JGE` · unsigned `JB JBE JA JAE` |
| CALL f,n | the top n items are the args, first arg = ST0; they are removed and the result is pushed |
| CALLI n | ST0 = target, below it n args; target and args removed; result pushed |
| SYS s,n | runtime service s with n args, as CALL |
| ENTER n | push FP; FP ← SP; SP ← SP − n |
| RET | d = 0: void return. d = 1: ST0 is the result |

- **Condition state** is valid only until the next instruction that is not a `Jcc`. Several `Jcc`s may follow one compare (three-way branches).
- **CALL always pushes a result** (undefined after a void function); the compiler `DROP`s it when unused.
- **Args right to left**, first arg on top, cleaned up by the call itself. There is no `ADJ`, and varargs need nothing extra.

## Design decisions
- **Spilling is the translator's job**; the IR has no register names and no depth limit.
- **Leaves need no stack slot**: `OP m`, `OPR m` and `COMP m` take them as memory/immediate operands. A Small-C expression needs a third stack slot only when both children of some node need two (e.g. `(a+b)*(c+d) - (e+f)*(g+h)`). That is rare, and costs one push/pop.
- **R forms** let the compiler evaluate the deeper operand first (Sethi-Ullman order; C leaves evaluation order unspecified) without `XCH`.
- **`XCH` is free** in the translator (the flip bit) — the Pentium's free `FXCH`, in miniature.
- **Plain `char` is signed.** Unsigned-char load: `LDB m`, `AND N 255`.
- **The value of a C comparison** is produced with branches (the 8086 has no SETcc).

## Lowering cheat sheet (C → PCODE86.3)
`{x}` = code that pushes x.
```
a op b          {a} {b} OPP          ; b simple:  {a} OP m b
                                     ; a simple, b complex:  {b} OPR m a
x = e           {e} ST L n           ; as a statement:  {e} STP L n
*p = e          {p} {e} STI          ; as a statement:  + DROP
x += b          {b} ADD L n, ST L n
x -= b          {b} SUBR L n, ST L n
*p += b         {p} DUP LDI {b} ADDP STI
x++             LD L n, DUP, ADD N 1, STP L n      ; old value left
++x             LD L n, ADD N 1, ST L n
-x / ~x         {x} NEG  /  {x} XOR N -1
uchar load      LDB m, AND N 255
f(x, y)         {y} {x} CALL f,2
(*fp)(x)        {x} {fp} CALLI 1
f(x);           {x} CALL f,1, DROP
```
In conditions (jump to `else` when false):
```
if (a < b)      {a} {b} COMPP, JGE else      ; b simple:  {a} COMP m b, JGE else
if (p < q)      … JAE else                   ; pointers: unsigned branch
if (x)          {x} COMP N 0, JE else
if (!x)         {x} COMP N 0, JNE else
if (a || b)     {a} COMP N 0, JNE then, {b} COMP N 0, JE else; then:
if (a && b)     {a} COMP N 0, JE else, {b} COMP N 0, JE else
```
As values:
```
a < b           {a} {b} COMPP, JL T, LD N 0, JMP E; T: LD N 1; E:
a || b          {a} COMP N 0, JNE T, {b} COMP N 0, JNE T, LD N 0, JMP E; T: LD N 1; E:
```
A signed divide by 2 must **not** become `SAR`.

## Translation (recommended scheme, not IR semantics)

### State
- **d**: depth, from the IR.
- **c ∈ {0, 1, 2}**: how many of the top items are in registers (c ≤ d). The other d − c items are on the hardware stack, in order.
- **flip bit**: whether ST0 is in AX or BX (when c = 2 the other register holds ST1).

### Rules
- **LD with c = 2**: push the ST1 register; load into it; it becomes ST0.
- **LD with c < 2**: load into a free register.
- **An op needing ST1 with c = 1**: `pop` into the free register.
- **XCH, c = 2**: toggle the flip bit, emit nothing.
- **Register constraints**, fixed with `xchg ax,bx` (1 byte) or a choice of flip bit:
  - MUL/DIV/MOD and byte loads (`cbw`) need AX.
  - LDI/STI need the address in BX (AX can't address memory).
  - Shift counts go via CL.
  - DIV/MOD use DX (`cwd`); MOD's result is moved back from DX.
  - Immediate MUL/DIV operands go via CX (no `imul imm` on the 8086).
- **CALL / CALLI / SYS**:
  - flush to c = 0, so args sit on the hardware stack with the first arg nearest SP (CALLI keeps the target in a register: `call bx`);
  - `add sp,2n`;
  - the result is in AX: c = 1, ST0 in AX.
- **Labels and jumps**: canonical state **c = min(d, 1), ST0 in AX**. Before a jump, and at a label reached by fall-through, the translator emits moves to reach it.
- **Between a compare and its Jcc** only flag-preserving instructions may be emitted (`mov`, `xchg`, `push`, `pop` — not `add sp`). Loads needed by COMPP are emitted before the `cmp`.
- **Templates use the 8087 names `ST0` and `ST1`** for "the register holding ST0" / "the register holding ST1"; the translator substitutes `ax` or `bx` according to the flip bit. E.g. `SUB L n` → `sub ST0,[bp+n]`; `SUBP` → `sub ST1,ST0`, then ST1's register becomes ST0.
- `COM N 0` / `TST` → `or ST0,ST0`. Conditional jumps on the 8086 are ±128 bytes — invert over a near `JMP`, or confirm A86 lengthens them.
- A86: mangle C names with `_`; `LD N sym` → `mov ST0,offset _sym`; `SYS s` → `call __sysS`.

### Examples
`(a+b)*(c-d)`, globals:
```
LD G a          mov ax,[_a]
ADD G b         add ax,[_b]
LD G c          mov bx,[_c]          ; free register: flip bit → ST0 = BX
SUB G d         sub bx,[_d]
MULP            imul bx              ; ST1 in AX × ST0; result AX
```
`if (a < b)`, locals at −4, −6:
```
LD L -4         mov ax,[bp-4]
COMP L -6       cmp ax,[bp-6]        ; pop costs nothing
JGE else        jge else
```
`x = a - f(b);` — x, a, b at −2, −4, −6:
```
LD L -4         mov ax,[bp-4]
LD L -6         mov bx,[bp-6]
CALL f,1        push ax / push bx / call _f / add sp,2     ; result AX, a on the hw stack
SUBP            pop bx / sub bx,ax   ; ST0 now in BX
STP L -2        mov [bp-2],bx
```

## Peephole (optional)
The compiler can emit memory forms directly (Cain's `lval` trick), so the peephole is small:
```
LD m x ; OPP     →  OP m x
LD m x ; OPRP    →  OPR m x
LD m x ; COMPP   →  COMP m x
XCH ; OPP        →  OPRP          (and OPRP → OPP)
XCH ; XCH        →  (nothing)
ST m x ; DROP    →  STP m x
```

## Worked call and program structure
`x = add(1, 2);` with `int add(int a, int b) { return a + b; }`:
```
add:  ENTER 0
      LD L 4
      ADD L 6
      RET              ; d = 1: result

      LD N 2
      LD N 1
      CALL add,2
      STP L -2         ; x
```
Directives as P86.1: `.CODE .DATA .BSS`, `DB`, `DW`, `RESB n`, labels `name:`. BSS emitted as real zero bytes in v1.

Hello world:
```
        .CODE
        CALL main,0
        SYS 0,1         ; EXIT(main's result); does not return
main:   ENTER 0
        LD N 15         ; n      (args right to left)
        LD N msg        ; buf
        LD N 1          ; fd = stdout, first arg on top
        SYS 7,3         ; WRITE
        DROP
        LD N 0
        RET
        .DATA
msg:    DB 'Hello, world!',13,10
```
Compiler emits `LD N 0, RET` before `main`'s closing brace (C99 5.1.2.2.3).

## SYS services
As P86.1 (first arg on top; result pushed):
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

## Testing
- **Reference interpreter**: unbounded stack, no caching — the oracle.
- **Depth verifier**: static d at every instruction, consistent at labels, d ∈ {0, 1} at RET.
- **Differential**: interpreter vs translated .COM in DOSBox. The peephole on or off must give identical output.
- **Translator stress**: generated expressions of depth 1–4, with calls nested at every position, to exercise every (d, c, flip bit) transition and the canonicalisation at labels.

## Not in v1
`long`, `float`, struct-by-value, `switch` jump tables.

## Open questions
- Exact text syntax of a p-code line (mode letters, `CALL f,n`).
- STB/STBI byte rule: keep P86.1's sign-extended result (`i = c = 300` → 44, costs `cbw`), or leave ST0 unchanged.
- Canonical state at labels: c = min(d, 1) (simple), or keep two cached (fewer moves, more cases).
- Whether CALL should push nothing for void functions (needs the callee's type at every call; the current rule is "always push, DROP if unused").
- Keep `COMPP`'s postfix orientation (current) or the 8087's ST(0)-vs-ST(1)?
