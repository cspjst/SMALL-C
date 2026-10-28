# PCODE86.4 — two named registers, orthogonal operands, 8087 compare (final, 2026-10-23)

Target of the Small-C compiler. Chosen over PCODE86.2 (accumulators, S filled only by POP) and PCODE86.3 (8087-style stack, translator caching); both are kept for reference. Arose from the byte-copy comparison (below).

Revisions 2026-10-23:
- Operand syntax replaces the mode letters (`[FP-2]`, `[sym]`, `[B]`, `5`).
- All loads and stores are one orthogonal `MOV` (word) and `MOVB` (byte); there are no separate load/store opcodes.
- `[A]` addressing accepted (costs `mov si,ax`).
- Memory destinations for `ADD SUB AND OR XOR`.
- Orthogonal `COM` and `TST` (memory operands).
- **Assembler: Open Watcom WASM + WLINK** replaces A86 (tested with Open Watcom v2 snapshot, 2026-10-23).

## Idea
- Two IR registers, **A** (→ `AX`) and **B** (→ `BX`), named in the p-code and chosen by the compiler.
- Convention: **values in A, addresses in B**, matching the 8086 (AX for arithmetic, `cbw`, results; BX can address memory).
- **Operands written as on the 8086**, with the 8086's rule: no two memory operands, and an immediate only as a source.
- Kept from 86.3:
  - reversed (R) forms;
  - the 8087's compare-then-branch (`COM`, `TST`, `Jcc`).
- `INC` / `DEC` on memory.
- The translator is a **stateless template table** (as 86.2). Spills are explicit (`PUSH`/`POP`) and are the compiler's job.

## Pipeline
```
C source → compiler → .p (p-code text) → (optional peephole) → expander → .asm (WASM, MASM syntax)
         → wasm → .o / .obj → wlink (+ runtime objects) → .COM → DOSBox
```

## Toolchain (Open Watcom v2)
```
wasm -0 -mt prog.asm                                         → prog.o  (prog.obj on DOS/Windows)
wlink format dos com name prog.com file prog.o,rt.o option quiet
```
- `-0`: 8086 instructions only. `-mt`: tiny model.
- **Use `format dos com`, not `system com`**: `system com` (in `wlink.lnk`) adds Watcom's C startup `cstart_t.obj`.
- Native Linux and Windows builds; assemble and link on the host, run the .COM in DOSBox.
- The runtime (crt0, `__sysN` stubs) is ordinary `.asm`, assembled once and linked in, or collected with `wlib`.
- Symbols are case-sensitive (`-cx` turns that off).
- `option map` writes a map file (segment addresses and sizes) for checking layout.

## Machine model
- 16-bit words, little-endian. One flat 64K byte-addressed space (tiny model).
- Registers:
  - `A` — values; the return value (→ `AX`)
  - `B` — addresses by convention (→ `BX`)
  - `SP`, `FP` (→ `BP`), `PC`
- Hardware stack: frames as P86.1, spills (`PUSH`/`POP`), call arguments.
- **Condition state**: set by `COM`, `TST`; read by `Jcc`.
- All arithmetic wraps mod 65536.

## Operands
| Kind | Written | Value | 8086 (WASM) |
|---|---|---|---|
| immediate | `5`, `-1`, `sym` | the number / the address of sym | `5`, `offset _sym` |
| register | `A`, `B` | the register | `ax`, `bx` |
| local / argument | `[FP-2]`, `[FP+4]` | W[FP ± n] | `[bp-2]` |
| global | `[sym]` | W[sym] | `_sym` / `[_sym]` |
| through B | `[B]` | W[B] | `[bx]` |
| through A | `[A]` | W[A] | `mov si,ax` first, then `[si]` |

- Shorthand in this spec: **r** = A or B; **mem** = any bracketed operand; **src** = immediate, register or mem; **dst** = register or mem.
- **Rule** (the 8086's): an instruction may not have two mem operands; an immediate is only ever a source (for `COM`, the second operand).
- `[A]` costs one extra `mov si,ax` per instruction; the compiler keeps addresses in B where it can.

## Notation
```
W[x], B[x]    word / byte at address x
s(x), sext(b) signed reading / byte sign-extended to a word
```

## Opcodes

### Data
| Op | Operation |
|---|---|
| MOV dst,src | dst ← src |
| MOVB A,mem | A ← sext(B[address]) — byte load; destination A only (`cbw`) |
| MOVB mem,r | B[address] ← r & 0xFF — byte store; r unchanged |
| MOVB mem,n | B[address] ← n & 0xFF |
| LEA r,[FP±n] | r ← FP ± n. (Global addresses: `MOV r,sym`.) |
| XCH | swap A and B |
| PUSH r | push r on the hardware stack |
| POP r | r ← pop |
| INC mem | W[address] ← W[address] + 1 |
| DEC mem | W[address] ← W[address] − 1 |

### Arithmetic
Operations: `ADD SUB MUL DIV MOD AND OR XOR SHL SHR SAR DIVU MODU`.

| Form | Ops | Operation |
|---|---|---|
| OP dst,src | ADD SUB AND OR XOR | dst ← dst op src — memory destination allowed |
| OP r,src | SHL SHR SAR | r ← r op src (src is the count) |
| OP A,src | MUL DIV MOD DIVU MODU | A ← A op src — destination A only |
| OPR r,src | SUB SHL SHR SAR | r ← src op r — register destination |
| OPR A,src | DIV MOD DIVU MODU | A ← src op A — destination A only |
| NEG r | — | r ← 0 − r |

- R forms exist only for the non-commutative ops.
- `~x` is `XOR dst,-1`.
- Semantics as 86.2/86.3:
  - MUL serves signed and unsigned.
  - DIV/MOD signed, truncating toward 0; TRAP on divide by 0 or 0x8000 / −1.
  - DIVU/MODU trap on 0.
  - Shift counts ≥ 16 defined (SHL/SHR → 0; SAR → sign fill).

### Compare
| Op | Operation |
|---|---|
| COM dst,src | compare dst with src — memory first operand allowed (not two mems) |
| TST dst | compare dst with 0 — register or memory |

- "Compare x with y" records x − y for the following `Jcc`.
- One compare serves signed and unsigned; signedness is chosen by the `Jcc`.
- With memory operands no register is needed: `if (x)` is `TST [FP-2]`, `i < 10` is `COM [FP-2],10`.

### Control
| Op | Operation |
|---|---|
| JMP L | jump |
| Jcc L | jump if the condition holds: `JE JNE` · signed `JL JLE JG JGE` · unsigned `JB JBE JA JAE` |
| CALL f,n | call f; afterwards remove the n arg words; result in A |
| CALLI r,n | call the address in r; as CALL |
| SYS s,n | runtime service s with n args; as CALL |
| ENTER n | push FP; FP ← SP; SP ← SP − n |
| RET | return; A is the result (undefined for void functions) |

- **Condition state** is valid only until the next instruction that is not a `Jcc`. Several `Jcc`s may follow one compare.
- **Arguments** are pushed right to left with `PUSH r`; the first arg is nearest SP. `CALL f,n` cleans them up. No `ADJ`; varargs need nothing extra.
- **After CALL, CALLI, SYS:** A = result, B undefined. Both registers are caller-saved: the compiler `PUSH`es any live register before the args and `POP`s it after the call.

## Design decisions
- **The compiler allocates the two registers.** Values go in A and addresses in B; MUL/DIV/MOD and byte loads must target A.
- **One orthogonal MOV** for loads, stores, register moves and immediate-to-memory stores (`x = 0;` is one instruction).
- **Read-modify-write on memory** for ADD SUB AND OR XOR: `x += 5` is `ADD [FP-2],5`.
- **Orthogonal COM and TST**: conditions on simple variables need no register.
- **Byte stores leave the register unchanged.** When the value of a byte assignment is used (`i = c = 300`), the compiler reloads it with `MOVB A,mem` and gets 44. That costs nothing when the value is unused, which is the common case.
- **Post-increment is "use, then increment"**: no DUP, no undo, no extra register.
- **R forms** let the compiler evaluate the needier operand first without a move.
- **Plain `char` is signed.** Unsigned-char load: `MOVB A,mem`, `AND A,255`.
- **The value of a C comparison** is produced with branches (the 8086 has no SETcc).

## Lowering cheat sheet (C → PCODE86.4)
`{e→r}` = code leaving e in r. "Simple" = constant, local or global (usable directly as an operand). x, y, p, m stand for their `[FP±n]` or `[sym]` operands.

Binary operators, into r (Sethi-Ullman: the needier operand first):
```
b simple                 {a→r} OP r,b
a simple, b complex      {b→r} OP r,a              ; non-commutative: OPR r,a
b needs one register     {a→r} {b→o} OP r,o        ; o = the other register
both need two            {a→r} PUSH r {b→r} POP o OPR r,o     ; r ← o op r = a op b
```
Assignments:
```
x = 5           MOV x,5
x = y           MOV A,y, MOV x,A
x = e           {e→A} MOV x,A
x += 5          ADD x,5                    ; likewise SUB AND OR XOR
x += y          MOV A,y, ADD x,A
x += e          {e→A} ADD x,A
x -= e          {e→A} SUB x,A
x *= e          MOV A,x, MUL A,{e}, MOV x,A    ; MUL/DIV/MOD/shifts: via a register
*p = e          {e→A} MOV B,p, MOV [B],A
*p += e         {e→A} MOV B,p, ADD [B],A
*p = e (p complex)   {p→A} PUSH A {e→A} POP B MOV [B],A
```
Increments:
```
x++ / ++x       INC x                      ; as a statement
x++  (value)    MOV A,x, INC x
++x  (value)    INC x, MOV A,x
(*p)++          MOV B,p, INC [B]
*p++ (value)    MOV B,p, MOV A,[B], INC p  ; char *: MOVB A,[B]
*++p (value)    INC p, MOV B,p, MOV A,[B]
int *p: p++     ADD p,2
*m++ = *p++     MOV B,p, MOVB A,[B], INC p, MOV B,m, MOVB [B],A, INC m
```
Other:
```
-x / ~x         NEG A  /  XOR A,-1
uchar load      MOVB A,mem, AND A,255
f(x, y)         {y→A} PUSH A, {x→A} PUSH A, CALL f,2     ; result in A
```
In conditions (jump to `else` when false):
```
if (x)          TST x, JE else
if (!x)         TST x, JNE else
if (x < 10)     COM x,10, JGE else
if (a < b)      MOV A,a, COM A,b, JGE else            ; two mems: one goes through A
if (e < b)      {e→A} COM A,b, JGE else
if (p < q)      … JAE else                            ; pointers: unsigned branch
if (*p)         MOV B,p, MOVB A,[B], TST A, JE else   ; byte: load (sign-extend), then test
if (a || b)     TST a, JNE then, TST b, JE else; then:          ; a, b simple
if (a && b)     TST a, JE else, TST b, JE else
```
As values:
```
a < b           … COM …, JL T, MOV A,0, JMP E; T: MOV A,1; E:
```
A signed divide by 2 must **not** become `SAR`.

## Expander (target: WASM, MASM syntax)
Template per (op, operand kinds); A/B substituted with `ax`/`bx`. Special cases:
- `[A]` operand: emit `mov si,ax` first and use `[si]`. SI is scratch.
- `MOV r,r` (same register): nothing.
- **Size overrides** (`word ptr` / `byte ptr`) where no register fixes the size:
  - memory ← immediate (`MOV`, `MOVB`, `ADD SUB AND OR XOR`): `mov word ptr [bp-2],0`, `mov byte ptr [bx],65`;
  - `COM mem,imm`: `cmp word ptr [bp-2],10`;
  - `TST mem`: `cmp word ptr …,0`;
  - `INC`/`DEC` memory: `inc word ptr [bp-2]`.
- Increments:
  - `ADD dst,1` / `SUB dst,1` → `inc` / `dec` (registers and memory).
  - `ADD p,2` on memory → `add word ptr …,2`.
- Compare with zero:
  - `TST r` and `COM r,0` → `or r,r`.
  - `TST mem` and `COM mem,0` → `cmp word ptr mem,0`.
- Byte loads: `MOVB A,mem` → `mov al,mem`, `cbw`. Byte stores: `mov mem,al` / `bl`.
- MUL/DIV/MOD:
  - DIV/MOD signed: `cwd`, `idiv`; MOD's result is moved from DX.
  - Unsigned (DIVU/MODU): DX ← 0, `div`.
  - An immediate source goes via CX (no `imul imm` on the 8086).
  - A memory source can be used directly (`idiv word ptr [bp-6]`).
- R forms without an 8086 equivalent:
  - `SUBR A,src` → `neg ax`, `add ax,src`.
  - `DIVR` / `MODR` / shift R forms → the original r goes to CX, the src into r, then the op.
- Shifts: count 1 → a single shift; otherwise the count goes into CL (from a register, immediate or memory).
- **Conditional jumps: emit `Jcc label` as-is.** WASM lengthens an out-of-range Jcc itself, even with `-0` (`jge far` → `jl $+5` / `jmp near far`), without a warning. Tested 2026-10-23.
- Names: mangle C names with `_` (keeps clear of MASM reserved words); `MOV r,sym` → `mov r,offset _sym`; `SYS s` → `call __sysS`. Internal labels from a counter (`L1`, `L2` …).
- **External symbols**: every `__sysN` the file calls, and every function or global defined in another file, is declared at the top with `extrn name:near` (functions) or `extrn name:word` / `:byte` (data). Functions and globals the file defines are declared `public`.
- **Scratch registers** the expander may use: CX, DX, SI. A and B are never clobbered except as specified.

## Examples
Byte copy `while (*p) *m++ = *p++;` — `char *p` at FP−2, `char *m` at FP−4:
```
top:
MOV  B,[FP-2]       mov bx,[bp-2]
MOVB A,[B]          mov al,[bx]
                    cbw
TST  A              or ax,ax
JE   end            je end
MOV  B,[FP-2]       mov bx,[bp-2]     ; p again: no CSE
MOVB A,[B]          mov al,[bx]
                    cbw
INC  [FP-2]         inc word ptr [bp-2]
MOV  B,[FP-4]       mov bx,[bp-4]
MOVB [B],A          mov [bx],al
INC  [FP-4]         inc word ptr [bp-4]
JMP  top            jmp top
end:
```
Copy loop, one iteration:

| Version | p-codes | 8086 |
|---|---|---|
| 86.3 stack, as specified | 16 | 21 |
| 86.4 without INC | 13 | 15 |
| **86.4** | **11** | **13** |
| 86.4, compiler remembers B still holds p | 10 | 12 |
| Hand-written, pointers in the frame | — | 9 |
| Open Watcom C (`-os`), pointers in BX/SI | — | 7 |

The gap to 9 is front-end knowledge: reusing `*p` from the test, and the dead `cbw` before a byte store and before a byte test. The gap from 9 to 7 is register-resident locals, which two IR registers can't express.

Watcom's loop, for reference:
```
L1: mov al,[bx] / test al,al / je L2 / mov [si],al / inc bx / inc si / jmp L1
```

Counting loop `for (i = 0; i < 10; i++) s += i;` — i, s at FP−2, −4:
```
MOV [FP-2],0        mov word ptr [bp-2],0
top:
COM [FP-2],10       cmp word ptr [bp-2],10
JGE end             jge end
MOV A,[FP-2]        mov ax,[bp-2]
ADD [FP-4],A        add [bp-4],ax
INC [FP-2]          inc word ptr [bp-2]
JMP top             jmp top
end:
```

`(a+b)*(c-d)`, globals:
```
MOV A,[a]       mov ax,_a
ADD A,[b]       add ax,_b
MOV B,[c]       mov bx,_c
SUB B,[d]       sub bx,_d
MUL A,B         imul bx
```
`x = a - f(b);` — x, a, b at FP−2, −4, −6 (a simple, f(b) complex → R form):
```
MOV A,[FP-6]    mov ax,[bp-6]
PUSH A          push ax
CALL f,1        call _f
                add sp,2
SUBR A,[FP-4]   neg ax
                add ax,[bp-4]
MOV [FP-2],A    mov [bp-2],ax
```
`x += y;  n = 0;  *p += 5;` — x, y, n, p at FP−2, −4, −6, −8:
```
MOV A,[FP-4]    mov ax,[bp-4]
ADD [FP-2],A    add [bp-2],ax
MOV [FP-6],0    mov word ptr [bp-6],0
MOV B,[FP-8]    mov bx,[bp-8]
ADD [B],5       add word ptr [bx],5
```

## Worked call
`x = add(1, 2);` with `int add(int a, int b) { return a + b; }`:
```
add:  ENTER 0
      MOV A,[FP+4]
      ADD A,[FP+6]
      RET

      MOV A,2
      PUSH A
      MOV A,1
      PUSH A
      CALL add,2
      MOV [FP-2],A     ; x
```

## Program structure
P-code directives and their WASM equivalents:

| P-code | WASM |
|---|---|
| (file start) | `.8086` / `.model tiny` / `extrn …` |
| `.CODE` | `.code` (first one followed by `org 100h`) |
| `.DATA` | `.data` |
| `.BSS` | `.data?` — **not stored in the .COM and not zeroed by DOS** |
| `DB`, `DW` | `db`, `dw` |
| `RESB n` | `db n dup (?)` |
| labels `name:` | code labels `name:`; data labels without the colon (`_msg db …`) |
| (file end) | `end start` (the file holding crt0); `end` otherwise |

- WLINK places `.code`, then `.data` (word-aligned), then `.data?` in DGROUP; all within one 64K segment (CS = DS = ES = SS).
- **BSS rule (v1):** crt0 zeroes BSS before calling `main`, using `rep stosb` from the start of `_BSS` to its end (labels exported by the runtime, or a final segment marker). Until that exists, the expander may emit `.BSS` items into `.data` as `db n dup (0)`.

Start-up and hello world, p-code:
```
        .CODE
        CALL main,0
        PUSH A
        SYS 0,1         ; EXIT(main's result); does not return
main:   ENTER 0
        MOV A,15        ; n      (args right to left)
        PUSH A
        MOV A,msg       ; buf
        PUSH A
        MOV A,1         ; fd = stdout
        PUSH A
        SYS 7,3         ; WRITE
        MOV A,0
        RET
        .DATA
msg:    DB 'Hello, world!',13,10
```
Expander output, `hw.asm` — **tested 2026-10-23**: with `rt.asm` below it assembles and links into an 80-byte .COM, and the disassembly checks out (call targets, stub offsets, `_msg` at 0140h after word alignment). Not yet run in DOSBox.
```
        .8086
        .model  tiny
        extrn   __sys0:near, __sys7:near
        .code
        org     100h
start:  call    _main
        push    ax
        call    __sys0
_main:  push    bp
        mov     bp,sp
        mov     ax,15
        push    ax
        mov     ax,offset _msg
        push    ax
        mov     ax,1
        push    ax
        call    __sys7
        add     sp,6
        mov     ax,0
        mov     sp,bp
        pop     bp
        ret
        .data
_msg    db      'Hello, world!',13,10
        end     start
```
Compiler emits `MOV A,0, RET` before `main`'s closing brace (C99 5.1.2.2.3). `ENTER 0` → `push bp` / `mov bp,sp`; `ENTER n` adds `sub sp,n`. `RET` → `mov sp,bp` / `pop bp` / `ret`.

Build:
```
wasm -0 -mt rt.asm
wasm -0 -mt hw.asm
wlink format dos com name hw.com file hw.o,rt.o option quiet
```
`hw.o` must come first in the file list: the module holding `org 100h` and `start` must be placed at the start of the .COM.

## SYS services
As P86.1 (args pushed right to left; result in A):
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
Stubs live in the runtime `rt.asm`, declared `public`; the expander's output declares the ones it uses `extrn __sysN:near`. First two stubs (tested with the hello world above):
```
        .8086
        .model  tiny
        .code
        public  __sys0, __sys7
__sys0: mov     bp,sp           ; never returns, so BP need not be saved
        mov     al,[bp+2]       ; code (8086 can't address [sp])
        mov     ah,4Ch
        int     21h
__sys7: push    bp
        mov     bp,sp
        mov     bx,[bp+4]       ; fd
        mov     dx,[bp+6]       ; buf
        mov     cx,[bp+8]       ; n
        mov     ah,40h
        int     21h             ; AX = bytes written (or error code if CF set)
        pop     bp
        ret
        end
```
@todo: error convention for OPEN/READ/WRITE (CF set → return −1, or −DOS error code).

## Target assumptions
86.4 encodes the word size and addressing of the 8086 / PDP-11 family:
- 16-bit registers;
- cheap indirect addressing through a register;
- read-modify-write on memory;
- a sign-extending byte load.

It would retarget reasonably to a 68000 or a RISC load-store machine: memory-destination ops become load, op, store using scratch registers. It does **not** suit 8-bit machines.

Measured on the 6502 (stateless templates; A and B as zero-page pairs; locals via `(fp),y`), the copy loop expands to **59 instructions per iteration**, against 13 on the 8086 and about 5 hand-written. The causes:
- word registers are byte-serial;
- signed-char sign extension costs 8 instructions per load (cc65 makes `char` unsigned for this reason);
- `[B]` needs the pointer copied into zero page;
- `INC [FP-n]` is 9 instructions.

An 8-bit target would want a register-rich, byte-aware IR instead: many registers, 8-bit ops and an index operand, in the manner of SWEET16 or cc65's zero-page registers.

## Testing
- **Reference interpreter**: A, B, condition state, hardware stack — the oracle for the IR.
- **Open Watcom C as a reference compiler**: each test program is also built with `wcl -0 -mt -l=com -s -j test.c` (`-s`: no stack checks; `-j`: signed `char`, matching this spec). Both .COMs run in DOSBox; outputs compared.
- **Differential**: interpreter vs expanded .COM in DOSBox. The peephole on or off must give identical output.
- **Code-size yardstick**: compare instruction counts against Watcom (`wcc -0 -ms -os`, disassembled with `wdis`) on the same functions.
- **Checks in the interpreter**:
  - B is poisoned after CALL/SYS, so a use of B before reloading it fails.
  - Condition state is poisoned after any non-`Jcc` instruction.
  - Illegal operand combinations are rejected:
    - two mem operands;
    - an immediate as a destination, or as the first operand of `COM`;
    - destination B for MUL/DIV/MOD or a byte load;
    - a memory destination for anything but MOV, MOVB, ADD, SUB, AND, OR, XOR, INC, DEC, COM and TST.
- **Compiler stress**: generated expressions with every combination of simple, one-register and two-register operands, with calls at every position.

## Not in v1
`long`, `float`, struct-by-value, `switch` jump tables.

## Open questions
- Exact text syntax details of a p-code line (spacing, case, `CALL f,n`).
- Register tracking in the compiler ("B still holds p"): worth it in v1?
- Restrictions to destination A (MUL/DIV/MOD, byte loads) vs letting the expander fix them with `xchg`.
- Indexed operands `[sym+B]` (8086 `[bx+disp]`) for global arrays — left out for now.
- Peephole: is one needed at all, if the compiler emits memory forms, R forms and `INC` directly?
- How crt0 finds the end of BSS (a marker segment after `_BSS`, or `_end` from WLINK).
