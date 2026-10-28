# PCODE — department notes

*Owned by PCODE. Read after `welfare/code-of-conduct.md`. Learning notes, **not a spec** (Jeremy, 2026-10-23: "pls dont build a spec this is for my learning"). Jeremy's files win over anything recorded here. History: `claude/sno-historiography.md` chapter 5 (P86.1 → 86.4 → back to P0) — extend that, don't duplicate it.*

*Source chat: "Abstract machine design for C compiler" (2026-10-22 22:41 – 2026-10-27), deleted on 2026-10-27. Everything from it worth keeping is here, in the four P86 specs, or in the historiography (chapters 3, 5, 16).*

## Remit
- Learn by building: a **simple stack machine (P0)** that Jeremy writes and grows step by step, translated to 8086 assembly by a target file.
- Later, try the other approaches against it (see "Approaches to try later").
- The p-code is meant to be language-neutral and target-neutral: any front end (C, Jeremy's Joy) in, any 16-bit target out (Jeremy, 2026-10-23). The 8086 is the first target. Repo: `pcode`, one expander per target.

## Toolchain (Open Watcom v2 — all verified 2026-10-23)
```
wasm -0 -zq -fo=prog.obj prog.asm
wlink format dos com name prog.com file prog.obj,sys.obj option quiet
```
- Jeremy's scripts: **`mkcom prog [other.obj …]`** (assemble + link) and **`obj.sh file …`** (assemble only, e.g. the runtime). Copies in Appendix A.
- `.COM`: no header; DOS loads it at offset 100h with CS = DS = ES = SS; PSP in 0–FFh. Hence `org 100h`.
- Gotchas found the hard way:
  - **wasm segfaults on a missing input file** — the scripts check first.
  - **The object holding `org 100h` / `start` must be first in the link.** Runtime first → wlink succeeds silently and writes a broken file (17 bytes instead of 40).
  - `.data?` (BSS) is not stored in the .COM and DOS doesn't zero it.
  - Size overrides: `word ptr` / `byte ptr` when no register fixes the size.
  - Out-of-range conditional jumps are lengthened by wasm itself, even with `-0`, silently.
  - `offset msg` = address; `msg` alone = contents (MASM semantics).
  - Macros can't take instruction names (`ADD macro` → E040), so p-code ops as WASM macros would need a prefix.
  - wasm's `-fl=` listing has symbols only, no code bytes — use `wdis prog.obj` for encodings.
- Use `format dos com`, not `system com` (that adds Watcom's C startup).
- Headless running: **emu2** (github.com/dmsc/emu2: `git clone`, `make`) runs a .COM in a terminal, captures stdout and the exit code — good for automated tests. DOSBox for interactive runs (`if errorlevel 42 echo …`).
- CMake works via custom commands (no WASM support built in); a tested skeleton is in Appendix A — not adopted.
- Watcom C as oracle and yardstick: `wcl -0 -mt -l=com -s -j test.c` (`-s` no stack checks, `-j` signed char); `wcc -0 -ms -os` + `wdis` to count instructions. Its byte-copy loop is 7 instructions (pointers in BX/SI).

## Learning steps done (2026-10-23)
1. `exit.asm` — 5 bytes, `mov ax,4C00h / int 21h`.
2. `hello.asm` — DOS 09h, `.data`, `offset`.
3. `frame.asm` — BP frame, local at `[bp-2]`, result in AX, exit code 42. Then rewritten with the P0 ops as comments (one frame: ENTER … EXIT).
4. `sys.asm` + `calls.asm` — runtime in its own object: `public` / `extrn name:near`, args at `[bp+4]`, caller removes them (`add sp,2n`). Copies in Appendix A.

## The P0 machine (Jeremy's definitions)
- Word size 16 bits; chars are words (`cbw`).
- The machine's stack is the 8086 hardware stack.
- Ops defined 2026-10-23:
  - `ENTER` — initialise variables, build the base of the stack.
  - `PUSH` — add a variable to the top of stack.
  - `POP` — remove the top item to a variable.
  - `ADD SUB MUL DIV` — pop two, push the result.
  - `EXIT` — deinitialise, pop the exit code, graceful exit to DOS.
- **Status (2026-10-27): v0.1 in progress — Jeremy is working on ENTER, LEAVE, EXIT and SYS.** Nothing written yet beyond the `.tgt` below; LEAVE is new and not yet defined here.

## Target file `8086.tgt` (format agreed 2026-10-23)
- Line 1: magic number for the p-code variant — `P0`. Line 2: number of keywords.
- Each keyword: `NAME,nargs,nlines`, then `nlines` lines of assembly template.
- `$1`…`$9` replaced by the arguments as text; arithmetic left to wasm (`sub sp,$1*2` — size in words).
- Lines starting with `;` are comments, skipped everywhere, not counted.
- `.BEGIN` / `.END` hold the per-program wrapper (`.8086`, `.model tiny`, `.code`, `org 100h`, `start:` … `end start`); the compiler emits them, programs never mention them.
- The expander indents template lines with a tab; trims trailing spaces and `\r`.

As of 2026-10-23:
```
P0
4
; start of every program: 8086 only, one 64K segment, code at 100h
.BEGIN,0,5
.8086
.model tiny
.code
org 100h
start:
; build the stack frame, size in words
ENTER,1,3
push bp
mov bp,sp
sub sp,$1*2
; pop the exit code, tear down the frame, return to DOS
EXIT,0,5
pop ax
mov sp,bp
pop bp
mov ah,4Ch
int 21h
; end of every program: entry point is start
.END,0,1
end start
```

## The expander in C (Jeremy's design, 2026-10-23)
- Jeremy's plan: a keyword struct (name, args, number of asm lines, index into a global array of template lines).
- Claude's review (Jeremy's call): keep the table (name, nargs, nlines, first) separate from each use's arguments; hold arguments as text, not words (names and labels come free, no conversion); `$` + one digit; check every fixed size while loading and report the `.tgt` line number; check the argument count at use.
- Related, from the "P-code generation for C compiler" chat (to 2026-10-27): **dispatch examples built on SNO** — split with `sno_next_any_lit`, classify the mnemonic, `switch` on the op. Claude's illustrations, run against stubs; their ops (`PUSHI PUSH ADDR LOAD STORE ENTER RET`) are not the P0 set. The point there was choosing the dispatch API (historiography chapter 14), not the translator. Since then `sno_any_lit` is `sno_find_lit`, taking C strings, and the `next_` pair is pinned (chapter 15).

## Runtime `sys.asm`
- `__sysN`, declared `public`; the program declares `extrn __sysN:near`.
- Arguments pushed right to left, first argument at `[bp+4]`; result in AX; caller removes the arguments.
- Service numbers as `claude/pcode86.4-spec.md`: 0 EXIT, 1 PUTC, 2 GETC, 3 OPEN, 4 CREAT, 5 CLOSE, 6 READ, 7 WRITE. Open: what OPEN/READ/WRITE return when DOS sets carry.
- Proposed `.tgt` entry (not decided): `SYS,2,2` → `call __sys$1` / `add sp,$2*2`; `extrn` line in `.BEGIN`.

## Open — Jeremy's calls
- **LEAVE:** what it does, and how it differs from EXIT.
- **Constants:** `PUSH 5` was assumed in `frame.asm` (8086 has no push immediate: `mov ax,5 / push ax`). Same op as `PUSH x`, or a separate `LIT`?
- **Operand order of SUB / DIV:** assumed `a b SUB` → a − b.
- **DIV by zero:** the 8086 raises INT 0; DOS prints "Divide overflow" and ends the program. Accept or check?
- **Values versus addresses** (2026-10-23): a word on the stack carries no type — the instruction decides. Proposed (Forth's way): `ADDR x` (push x's address), `LOAD` (address → word there), `STORE` (value, address → memory); `PUSH x` / `POP x` become shortcuts for `ADDR x, LOAD` / `ADDR x, STORE`. Not decided.

## Approaches to try later (each measured against P0 on the same programs)
- **Virtual-stack translator** (Claude's first recommendation, 2026-10-22, not taken then): keep the pure stack IR; the translator keeps a compile-time stack of operand descriptors (`CONST`, `ADDR_LOCAL`, `MEM_LOCAL`, `REG`, `COND`, `SPILLED`) and emits code only when an operand must exist. `x = a + b` → 3 instructions. Precedents: Wirth's Items (Oberon-0), V8 Liftoff for Wasm. Hazards: stack height at labels, aliasing of deferred loads, calls clobbering AX–DX, short Jcc.
- **Staged translator for the same IR:** (1) macro table — the correctness baseline; (2) top of stack cached in AX; (3) descriptors as above.
- **PCODE86.2** — two accumulators, Cain/Hendrix lineage (`claude/pcode86.2-spec.md`).
- **PCODE86.3** — 8087-style stack two deep, static stack caching with a flip bit (`claude/pcode86.3-spec.md`).
- **PCODE86.4** — two named registers, orthogonal operands (`claude/pcode86.4-spec.md`; "we come back to p86.4 later").
- **A P0 interpreter in C** as a test oracle: run the same p-code on the host and as a .COM, compare outputs.
- Byte-copy loop `while(*p) *m++ = *p++;` per iteration, the yardstick so far: 86.3 stack 21, 86.4 13, hand-written 9, Watcom 7; the 6502 under 86.4 59.

## References (for following up)
- Ron Cain, "A Small C Compiler for the 8080's", *Dr. Dobb's Journal* no. 45, April–May 1980 — DDJ vol. 5 on archive.org: https://archive.org/details/dr_dobbs_journal_vol_05_201803 (Small-C starts about PDF page 190).
- Cain's source, reconstructed from OCR: https://github.com/trcwm/smallc_v1 — the code generator is the "cc8" section at the end of `main.c` (~45 functions, primary HL / secondary DE; `getmem`, `push`, `add`, `testjump`…). On a 32-bit host it prints `LXI H,65536` for 0 (16-bit `int` assumed).
- James E. Hendrix, *The Small-C Handbook*, Reston, 1984 (Jeremy has it); *A Small C Compiler*, M&T Books, 1990, ISBN 1-55851-124-5 — the p-code table version (2.2 targets the 8086, `cc4.c`).
- Pete Gray, "Porting Small-C", DDJ, March 2004: https://jacobfilipp.com/DrDobbs/articles/DDJ/2004/0403/0403n/0403n.htm — p-code-to-template table; 8086 `ADD12` → `add ax,bx`.
- Andy Yuen, "Porting CSC to the 8051", DDJ, August 1997: https://jacobfilipp.com/DrDobbs/articles/DDJ/1997/9708/9708g/9708g.htm — 109 p-codes, peephole on p-codes, GNU M4 macro back ends.
- Small-C sources and variants (incl. a Byte 8088 conversion and Hendrix 2.1): https://cpm.retropc.se/cpm/small_c.html
- comp.compilers on Small-C's primary/secondary model: https://compilers.iecc.com/comparch/article/97-11-165; on 8086 code generators and version 2.2: https://compilers.iecc.com/comparch/article/10-09-034
- R. E. Berry & B. A. E. Meekings, *A Book on C*, Macmillan, 1984 — lists RatC, close to Cain's compiler.
- M. Anton Ertl, "Stack Caching for Interpreters", PLDI 1995 — static stack caching (86.3's translator).
- Niklaus Wirth, *Compiler Construction* (Oberon-0) — Items, the descriptor idea.
- Steve Wozniak, SWEET16 (Apple II ROM) — a 16-register virtual machine on the 6502; the kind of IR an 8-bit target wants.

## Appendix A — files from the source chat (tested 2026-10-23)

Copied from the files as run, not retyped. Jeremy has his own copies; these are for reference.

### `mkcom`

```sh
#!/bin/sh
# mkcom: assemble and link a DOS .COM.
# usage: mkcom prog[.asm] [other.obj ...]    prog must hold org 100h / start
set -e
name=${1%.asm}
[ -n "$name" ] || { echo "usage: mkcom prog[.asm] [other.obj ...]" >&2; exit 1; }
[ -f "$name.asm" ] || { echo "mkcom: $name.asm not found" >&2; exit 1; }
shift
files="$name.obj"
for f in "$@"; do
    [ -f "$f" ] || { echo "mkcom: $f not found" >&2; exit 1; }
    files="$files,$f"
done
wasm -0 -zq -fo="$name.obj" "$name.asm"
wlink format dos com name "$name.com" file "$files" option quiet
ls -l "$name.com"
```

### `obj.sh`

```sh
#!/bin/sh
# obj.sh: assemble one or more .asm files to .obj (no linking).
# usage: obj.sh sys[.asm] [more[.asm] ...]
set -e
[ $# -gt 0 ] || { echo "usage: obj.sh file[.asm] ..." >&2; exit 1; }
for f in "$@"; do
    name=${f%.asm}
    [ -f "$name.asm" ] || { echo "obj.sh: $name.asm not found" >&2; exit 1; }
    wasm -0 -zq -fo="$name.obj" "$name.asm"
    ls -l "$name.obj"
done
```

### `sys.asm`

```asm
; sys.asm - runtime services, called with arguments on the stack
        .8086
        .model  tiny
        .code
        public  __sys0, __sys1

; SYS 0  EXIT(code) - does not return
__sys0: push    bp
        mov     bp,sp
        mov     ax,[bp+4]       ; code
        mov     ah,4Ch
        int     21h

; SYS 1  PUTC(c)
__sys1: push    bp
        mov     bp,sp
        mov     dl,[bp+4]       ; c
        mov     ah,2
        int     21h
        pop     bp
        ret
        end
```

### `calls.asm`

```asm
        .8086
        .model  tiny
        extrn   __sys0:near, __sys1:near
        .code
        org     100h
start:  mov     ax,'*'
        push    ax
        call    __sys1          ; PUTC('*')
        add     sp,2            ; caller removes the argument
        mov     ax,42
        push    ax
        call    __sys0          ; EXIT(42)
        end     start
```

### `CMakeLists.txt`

```cmake
cmake_minimum_required(VERSION 3.16)
project(smallc C)                       # host build: your compiler, in C99
set(CMAKE_C_STANDARD 99)

# ---- Open Watcom tools (found on PATH or under $WATCOM) ---------------------
find_program(WASM  wasm  HINTS ENV WATCOM PATH_SUFFIXES binl64 binnt64 binnt REQUIRED)
find_program(WLINK wlink HINTS ENV WATCOM PATH_SUFFIXES binl64 binnt64 binnt REQUIRED)
set(WASM_FLAGS -0 -mt -q)

# wasm_object(<out_var> <file.asm>): assemble one file, return the .obj path
function(wasm_object out src)
  get_filename_component(abs ${src} ABSOLUTE)
  get_filename_component(stem ${src} NAME_WE)
  set(obj ${CMAKE_CURRENT_BINARY_DIR}/${stem}.obj)
  add_custom_command(OUTPUT ${obj}
    COMMAND ${WASM} ${WASM_FLAGS} -fo=${obj} ${abs}
    DEPENDS ${abs}
    COMMENT "wasm ${stem}.asm")
  set(${out} ${obj} PARENT_SCOPE)
endfunction()

# the runtime, assembled once
wasm_object(RT_OBJ ${CMAKE_SOURCE_DIR}/runtime/rt.asm)
add_custom_target(runtime DEPENDS ${RT_OBJ})

# add_com(<name> <file.asm>): one .COM program; its own object goes first
function(add_com name src)
  wasm_object(obj ${src})
  set(com ${CMAKE_CURRENT_BINARY_DIR}/${name}.com)
  add_custom_command(OUTPUT ${com}
    COMMAND ${WLINK} format dos com name ${com} file ${obj},${RT_OBJ} option quiet
    DEPENDS ${obj} ${RT_OBJ}
    COMMENT "wlink ${name}.com")
  add_custom_target(${name} ALL DEPENDS ${com})
endfunction()


# ---- your compiler and expander: ordinary host C99 programs ---------------
add_executable(scc  src/scc.c)
add_executable(pexp src/pexp.c)

# add_c_com(<name> <file.c>): C -> .p -> .asm -> .obj -> .COM
function(add_c_com name src)
  get_filename_component(abs ${src} ABSOLUTE)
  set(p   ${CMAKE_CURRENT_BINARY_DIR}/${name}.p)
  set(asm ${CMAKE_CURRENT_BINARY_DIR}/${name}.asm)
  add_custom_command(OUTPUT ${p}
    COMMAND scc ${abs} -o ${p}
    DEPENDS ${abs} scc
    COMMENT "scc ${name}.c")
  add_custom_command(OUTPUT ${asm}
    COMMAND pexp ${p} -o ${asm}
    DEPENDS ${p} pexp
    COMMENT "pexp ${name}.p")
  add_com(${name} ${asm})
endfunction()

# ---- tests: run each .COM in emu2, check output and exit code -------------
enable_testing()
find_program(EMU2 emu2)
function(add_com_test name)
  cmake_parse_arguments(T "" "EXPECT;EXIT" "" ${ARGN})
  if(NOT EMU2)
    return()
  endif()
  add_test(NAME ${name}
    COMMAND ${CMAKE_COMMAND} -DEMU2=${EMU2} -DCOM=${CMAKE_CURRENT_BINARY_DIR}/${name}.com
            -DEXPECT=${T_EXPECT} -DEXIT=${T_EXIT} -P ${CMAKE_SOURCE_DIR}/cmake/runcom.cmake)
endfunction()

add_c_com(hello  tests/hello.c)
add_c_com(exit42 tests/exit42.c)
add_com_test(hello  EXPECT ${CMAKE_SOURCE_DIR}/tests/hello.expected EXIT 0)
add_com_test(exit42 EXIT 42)
```

### `runcom.cmake`

```cmake
# cmake -DEMU2=… -DCOM=… [-DEXPECT=file] [-DEXIT=n] -P runcom.cmake
execute_process(COMMAND ${EMU2} ${COM} OUTPUT_VARIABLE out RESULT_VARIABLE rc TIMEOUT 10)
if(DEFINED EXIT AND NOT EXIT STREQUAL "" AND NOT rc EQUAL EXIT)
  message(FATAL_ERROR "exit code ${rc}, expected ${EXIT}")
endif()
if(EXPECT)
  file(READ ${EXPECT} want)
  if(NOT out STREQUAL want)
    message(FATAL_ERROR "output differs:\n--- got\n${out}--- expected\n${want}")
  endif()
endif()
```

The CMake skeleton also needs `src/scc.c` and `src/pexp.c` (copy-through stand-ins for the compiler and expander), `runtime/rt.asm` (the runtime) and `tests/*.c` with `.expected` files. Its CTest run passed (hello: output; exit42: exit code), and a wrong expected output failed as it should.
