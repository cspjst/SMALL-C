# Small-C / P86 / SNO historiography — how the design got here

*Owned by WELFARE: one historiography for the whole project (Jeremy, 2026-10-28: "one historiography to rule them all"), named after the repository (19:24: "historiography takes root repo name"). Departments send WELFARE their milestones and keep none of their own. Until 2026-10-28 it was `claude/sno-historiography.md` in the claude.ai project, kept by SNO.*

A dated record of *why*, for the README, the blog and future us. It covers the whole project, from the first conversation about a Small-C compiler for the IBM XT (17 September 2026) onwards. Each chapter follows the same pattern: the problem as it appeared, what was tried, what the book, the hardware or the history said, and what was decided and by whom. Jeremy's own calls are marked, and his words are quoted where they add colour.

Times are UK local (BST until 02:00 on 25 October, GMT after). Where the record is thin, the chapter says so. In particular, 22 September to 13 October left no record at all, and 14 to 24 October (morning) survive only as a compaction summary of a long Claude Code session plus the decision records it produced. Those chapters are reconstructions, and they are labelled as such.

Companion docs (paths in the repository): `welfare/working-agreement.md` (the current rules and settled decisions), `sno/sno-backtracking-lessons.md` (the FAIL/FENCE/ABORT study), `press/sno-readme-reference.md` (the README and its "When Worlds Collide" clashes), `pcode/pcode-notes.md`, and the P86 specs (`pcode/p86.1-spec.md`, `pcode/pcode86.2-spec.md`, `pcode/pcode86.3-spec.md`, `pcode/pcode86.4-spec.md`). Chapters cite documents by the names they had at the time.

## Contents

1. Why Small-C, why now (17 September)
2. Errors, passes and an XT's disk (18–21 September)
3. The retro rig: assemblers, 86Box, mtools (21–22 September; toolchain postscript to 24 October)
4. The silent weeks, and P86.1 (22 September – 16 October)
5. From P86.1 to PCODE86.4, and back to a stack machine (22–23 October)
6. Why SNO: "I've wanted to do this since 1988!"
7. SNO's founding decisions (about 14–21 October; reconstructed)
8. The view stays two pointers (22 October; reconstructed)
9. True is a null string, and a README goes public (24 October, to about 15:00; reconstructed)
10. The sandpit: replacement, "must" versus "fails if", and FEET to FOOT (24 October, 15:00–23:31)
11. FAIL, FENCE and ABORT: SNOBOL's backtracking, read for lessons (25 October morning)
12. From "the halt test" to &ERRLIMIT (25 October, 13:57–14:35)
13. The literal family gets its signatures, and SNO builds itself (25 October, 15:59–17:40)
14. Dispatch: "a literal says what; a list says which" (25 October, 21:33–22:40)
15. C strings all the way down, and a legacy on death row (27 October, morning to 17:18)
16. Departments and a code of conduct (27 October, afternoon; thin)

---

## Chapter 1: Why Small-C, why now (2026-09-17)

### 1. The problem as it appeared
- Jeremy opened with the whole story in one message (15:38): programming as a hobby "for 48 years since age 11"; his golden era was buying an IBM XT and learning C and 8086 assembler; now semi-retired and back to retro programming; and he still has the book, James E. Hendrix, *The Small-C Handbook* (Reston, 1984). "I always wanted to write a small c compiler for the ibm xt - now i have time to do this and learn about modern compiler techniques and implement them in C 99 using the watcom compiler and 8086 assembly."
- The setup: code written on an M3 MacBook in Zed, compiled with Open Watcom, tested in DOSBox-X. Real hardware exists too: "I have 2 IBM XTs in various states of repair."

### 2. What was weighed
- Claude asked which project he meant: a faithful re-enactment of Hendrix (recursive descent straight to assembly, no IR), or a modern one with an AST, an IR and passes.
- Self-hosting needs a bootstrap. The answer was already on the Mac: Open Watcom builds generation 0, gen0 compiles the compiler's own source into gen1, and the gen1/gen2 assembly diff proves the fixed point. The catch is to write the compiler in the Small-C subset from day one.
- Watcom's `-0` flag was flagged early. A stray 286 instruction would run in DOSBox-X and then fail on the XTs.

### 3. What was decided (Jeremy)
- **Book first, modern later:** "I would like to follow the book's recursive descent to assembly as the first goal as it has been long held goal but then move on to more modern techniques."
- **The finished compiler runs natively** on the XT, its emulators, or a DOSBox-X set up as closely to one as possible.
- A five-stage plan was agreed (Stage 0 toolchain loop, Stage 1 gen0 by Watcom, Stage 2 self-hosting proof, Stage 3 native on the XT, Stage 4 modern techniques). Jeremy keeps his own design document in Typora ("you can assume that I will build the document myself going forward"). A GitHub repo, IBM XT SMALL-C, was set up the same afternoon.
- **Stage 0 was done by 16:59.** A `printf("IBM XT SMALL-C\n")` ran in DOSBox-X. The Chips and Technologies benchmark put DOSBox-X at 0.73 of an IBM PC at 240 cycles; Jeremy tuned it to 324 cycles/ms for 1.01 ("so 324 it is going forward"). The compiler flags were trimmed: `-ox -e1 -zp1 -za99` off, `-fpi87` kept, and "-ml its large model all the way" for the compiler itself.
- **Pure Watcom C for v0 (22:20).** The evening wandered into Jeremy's own `doslib`/`bioslib`, a FILE type and `dos_malloc`. He pulled it back himself: "I want pure watcom C v0 the rest can come later and give me something real to write with my smallc." The first goal became `TEST.EXE`: a `macro_pass` that reads a file and writes the macro-substituted output. Claude supplied `lexer_test.c` (a token zoo) and `SIEVE.C`, Gilbreath's 1981 BYTE benchmark, as the reference program to carry through every stage.

### What it adds up to
A forty-year-old ambition was given a concrete shape in one afternoon: Hendrix's design first, built by Watcom, proved by self-hosting, and measured against real XTs. The habit of pulling scope back to "v0" was Jeremy's, and it recurs throughout.

Sources: "Small C compiler for IBM XT" (https://claude.ai/chat/83a95635-aa4e-4e4e-91af-e558df2f8e2f), 2026-09-17 15:38–22:20.

---

## Chapter 2: Errors, passes and an XT's disk (2026-09-18 – 2026-09-21)

### 1. The problem as it appeared
- Even a macro pass can fail, so Jeremy wanted "robust error reporting" from the start (18 September, 18:47).
- His experience of real XTs posed a second question: a PPM-to-CGA converter of his used `fgetc`/`fputc`, "was brisk on dosbox-x but impossibly slow on the ibm xt", and had led him "down the rabbit hole of arenas and processing the whole file in memory".

### 2. Error numbers: borrow a real list (Jeremy)
- "I'd rather use the error number is an index to an error message approach", plus a reference list so as not to reinvent the wheel. He settled on Microsoft C's own compiler errors ("i think im going to wholesale adopt Microsoft C Compiler Fatal Errors even just reading them is insightful"), noting that they're "not all consecutive which makes message lookup a pain".
- This is the same instinct that, five weeks later, made SNO adopt the Green Book's error numbers and messages wholesale (chapter 7).

### 3. One program per pass (Jeremy)
- "hmm im rethinking the all in one approach the old MS c approach is a bit more unix-philosophy esque."
- Claude checked the precedent. The Unix pipeline is `cpp | cc1 | as | ld`, split where the interchange format is text that is useful on its own, not at every phase. On an XT the real reason is memory: each program hands all of it back to DOS on exit. The proposed shape was `macro_pass.exe` (≈ cpp) → `smallc.exe` (≈ cc1) → assembler → linker, chained by a DOS batch file with `IF ERRORLEVEL 1` after each step.
- Exit codes checked by a batch file through ERRORLEVEL came back in October as the way SNO's precondition errors are tested.

### 4. Measure, don't argue (Jeremy)
- Claude went round in circles on buffering (`setvbuf`, an own `fread` buffer, a shared buffer for two FILE handles). Jeremy called it out: "yeah i think youre in a bit of an ai slop loop my friend - i think I will write a test using my bios_read_system_clock to test buffer sizes i suspect the bottle neck is reading from the mfm drive."
- He wrote the test himself (write and read with `fputc`/`fgetc`, buffer sizes 0–24K, files 16K–256K), and corrected Claude's batch files along the way: "i dont think you realise how slow xt disk access times really are", and a batch file "not compatible below msdos 6.22" was replaced by his own KISS one. Buffers above about 24–32K were flaky, so the runs stopped at 24K. The chat doesn't say on which machine the final numbers were taken.
- **Result (21 September):** unbuffered was about 3.8 times slower; from a 1K buffer to 24K the time was flat, and it doubled cleanly with file size. Claude read it as per-call overhead rather than the disk, at roughly 875 clock cycles per `fputc` even when buffered.
- **Decision (Jeremy, 21:14):** for v0, `fread`/`fwrite` into an own `malloc`ed buffer and process it character by character and line by line, with the buffer sized to the file up to a 64K maximum. ("so even if you had an unlikely c program that used 80 chars on every line how many LOC would fit in 64k?")

### What it adds up to
Three habits formed in four days, and all three come back later: errors are numbered from someone else's authoritative list; programs are small and chained by ERRORLEVEL; and when the XT is involved, measure it. The buffered-line-reading conclusion is probably the ancestor of the "mem_stream" work mentioned for mid-October, but nothing in the record confirms the link.

Sources: "Small C compiler for IBM XT" (https://claude.ai/chat/83a95635-aa4e-4e4e-91af-e558df2f8e2f), 2026-09-18 18:47 – 2026-09-21 21:19.

---

## Chapter 3: The retro rig: assemblers, 86Box, mtools (2026-09-21 – 2026-09-22; toolchain postscript to 2026-10-24)

### 1. The problem as it appeared
- Small-C emits assembly, so something has to assemble it on DOSBox-X and on the XT (21 September, 22:32).

### 2. What the history said
- Claude recommended **A86**: a single 32K executable that assembles straight to a .COM, which removes the link stage on hardware where every stage costs minutes. The price is the tiny model, one 64K segment. TASM was the fallback for multi-segment programs.
- Jeremy "used and loved MASM but on later 286/386 machines". The answer was a hard cliff: MASM 1.0–5.1 run on an 8088, while MASM 6.1 needs a 386 and 4MB.
- Inline `__asm {}` for the compiler: Jeremy wanted it "not faithful to Hendrix but heavily influenced by", limited to the registers Watcom's register convention uses. Claude confirmed the order (AX, BX, CX, DX) and recommended the cheap version: copy stack arguments into registers only for functions whose body is all `__asm`.

### 3. 86Box and the CF card (Jeremy)
- To get period software (MASM) onto the real machines, Jeremy chose to install it in **86Box** emulating an XT, then copy it to a CF card. His choices: XTIDE as the storage controller ("not interested in emulating just utility"), a raw `.img` disk image, and PC-DOS 5 ("wow 86box is awesome!").
- **mtools** reads and writes the image from the Mac through a `z:` drive in `~/.mtoolsrc` ("love that z: drive trick"). Two lessons came out of it: `.mtoolsrc` is a config file, not a shell script to `source`, and 86Box must never be running while mtools touches the image. The chat ends on 22 September with `mcopy -s z:\MASM ./masm`.

### 4. Postscript: the toolchain moves on (October)
- **23 October:** "having a rethink how can i use open watcoms assembler". Open Watcom's **WASM + WLINK** replaced A86. In tests WASM lengthened out-of-range conditional jumps by itself, even with `-0`, and mapped `.code`/`.data`/`.data?` onto the P86 segments. Watcom's own C compiler also became a test oracle and a yardstick. Claude wrote `mkcom` (wasm + wlink in one step) and `obj.sh` for runtime objects; WLINK needs `format dos com`, because `system com` drags in Watcom's C startup. (Details in chapter 5.)
- **24 October, 08:28:** Zed has no assembler highlighting out of the box. The "Assembly Language Server" extension was installed with asm-lsp switched off, because its generic grammar would flag good WASM directives, and neocmake was added for `CMakeLists.txt`.
- The SNO README's colophon (written 24 October) still says "assembled with A86". That is out of date since 23 October.

### What it adds up to
The rig became a modern Mac with Zed and Watcom, DOSBox-X tuned to an XT's speed, 86Box with mtools as the bridge to period software, and two real XTs as the final judge. A86 was the right answer for an XT-hosted pipeline and lost to WASM once assembling on the host was the norm.

Sources: "Small C compiler for IBM XT" (https://claude.ai/chat/83a95635-aa4e-4e4e-91af-e558df2f8e2f), 2026-09-21 22:32 – 2026-09-22 14:10; "Abstract machine design for C compiler" (https://claude.ai/chat/af1761cd-b76b-4ecb-b6b1-aac4b985f978) (deleted 2026-10-27; its content survives in `pcode/pcode-notes.md` and the P86 specs), 2026-10-23 11:55–22:13; "Zed assembler setup" (https://claude.ai/chat/4b66e36f-cb4e-42cc-a86d-8d77164282b1), 2026-10-24 08:28–08:42; `sno-readme-reference.md`.

---

## Chapter 4: The silent weeks, and P86.1 (2026-09-22 – 2026-10-16)

### 1. What the record holds
- **22 September – 13 October: nothing.** No claude.ai chat and no surviving session covers these three weeks. Whatever happened to `macro_pass`, the lexer and the buffered reader isn't recorded.
- **14–16 October:** these days were in the Claude Code session whose detail was later lost to compaction. What survives is the summary's list of topics ("pcode questions; the P86 opcode list; … Intel-style reference; JOY; compile not interpret; A86; hello world; mem_stream; SNOBOL ARB/REM; no backtracking; 'this is fun!'") and the spec it produced, `p86.1-spec.md`, dated 2026-10-14.

### 2. What P86.1 decided (from the spec)
- **An abstract machine between compiler and 8086.** The compiler emits P86 and a translator turns it into 8086 assembly for a DOS .COM. This was the first step away from Hendrix's direct emission, and the first "modern technique".
- **A pure stack machine.** 16-bit, little-endian, one flat 64K space (tiny model). 35 opcodes, with bit 7 of the opcode meaning a 16-bit operand follows, cut down from a 45-op draft: `NEG NOT GT GE LE NE LDBU` were all expressible, so they went.
- **C-level rules that outlived it:** stores push their value (assignment is an expression); `STB` leaves the sign-extended byte (`i = c = 300` gives 44); plain `char` is signed; shift counts of 16 or more are defined; the return value lives in `R` (→ AX); there's a BP-style frame; and arguments go right to left with the caller popping, so varargs need nothing extra.
- **SYS services with P86's own numbers**, mapped to INT 21h, "so the same program can run on another host". The C runtime is written on top of them: "the Forth/Joy/UCSD 'small kernel, rest in the language' approach". Jeremy has his own Joy interpreter.
- **Compile, don't interpret.** An interpreter for P86 was "originally planned first, then dropped: compiling to 8086 is the goal". (It came back on 22 October, but only as a test oracle.)

### 3. The first working rule
- One line from those days survives in the summary, and it is the seed of the working agreement: "No write for me only explain and check and code snippets to steer me."

### What it adds up to
After a three-week silence the project reappears with an IR. The stack machine was simple enough to get right and was explicitly a first sketch: its own spec lists "pure stack vs accumulator vs register IR" as the open question.

Sources: compaction summary in the Claude Code transcript (`b5d27a19-…jsonl`, first user message, covering 2026-10-14 → 2026-10-24); `p86.1-spec.md` (as rewritten 2026-10-22, quoted in the "Abstract machine" chat). Thin: no chat or session records 2026-09-22 → 2026-10-13, and 14–16 October are known only from the summary.

---

## Chapter 5: From P86.1 to PCODE86.4, and back to a stack machine (2026-10-22 22:41 – 2026-10-23 22:16)

### 1. The problem as it appeared
- Before committing to P86.1, Jeremy asked for the alternatives, judged on ease of code generation, quality of 8086 code, translator simplicity and what he'd learn: pure stack, one- and two-register accumulators "as in Ron Cain's original Small-C", register and three-address IRs.

### 2. What was tried, and what history said
- **Claude's recommendation was not taken.** It was to keep P86.1 as a pure stack IR and put the cleverness in a *virtual-stack* translator (Wirth's items, V8's Liftoff), which turns `x = a + b` into three instructions.
- **Cain, 1980** (read from the OCR-reconstructed source): about 45 C functions each printing fixed 8080 code, with HL as primary and DE as secondary. "It's really a stack machine with the top of stack cached in HL."
- **Hendrix 2.x** moved to p-codes and a template table (`ADD12` → `add ax,bx`). **Yuen's RCSC (DDJ, 1997)** had a text p-code file, a peephole pass on the p-codes and GNU M4 macros per target, with 109 p-codes because the operand kind is spelled into each name.

### 3. What was decided, step by step (Jeremy's calls throughout)
- **22:53, "I want to follow Yuen's approach",** but with "my own small expander in C". The question "how small can we make the opcode set" was answered by making the operand kind an argument (a mode) rather than part of the name: 31 opcodes. Then "give me neg and jnz back pls" made it 33, and at 23:02, "ok this is pcode86.2". P86.1 was marked superseded.
- **23:29, the 8087:** "familiar with it and think it was beautiful - i particularly liked COM for compare." An 8-slot register stack was sketched; Jeremy cut it to "a stack only 2 deep???". That gave **PCODE86.3**: a pure postfix IR with the 8087's forms (pop forms, reversed forms, `COM`/`COMP`/`COMPP`/`TST` with signedness in the `Jcc`). The translator does static stack caching (Ertl, PLDI 1995) with three pieces of state, one of which is a "flip bit" that makes `XCH` free, the Pentium's free FXCH in miniature. "yes lets fix p86.3 in its own markdown file - I am drawn to its beauty however." He refused extra opcodes ("nope no extra opcodes its fine the way it is for now") and kept the names ST0/ST1 in the templates because "it will help me read it".
- **23:59, the test that decided it:** `while(*p) *m++ = *p++;`. The 86.3 stack version took **21** 8086 instructions per iteration; the same opcodes with two *named* registers took **15**; hand-written took 9. The compiler knows `p` is a pointer, so it can put it straight into BX.
- **00:05 (23 October), "keep same opcodes but recognise the 2 registers as A & B".** Then came `INC m` ("it will be worth it"; "this is 86.4 pls plain inc m"), and between 00:28 and 00:48 the operand syntax went 8086-style: one orthogonal `MOV`/`MOVB` instead of LD/ST ("wait i intended to do away with ld and st in favour fully orthogonal mov"), `[A]` addressing accepted at the cost of `mov si,ax`, memory destinations for `ADD SUB AND OR XOR`, and orthogonal `COM`/`TST`. **PCODE86.4** was labelled final, with values in A and addresses in B, as the 8086 wants.

### 4. Honesty checks
- **The 6502 (Jeremy's "6503"):** "clearly p86.4 is 8086 leaning and perhaps even more pdp11 leaning". The copy loop expands to **59** 6502 instructions per iteration against about 5 by hand. The IR assumes 16-bit words, addressing through a register and read-modify-write on memory, so the repo should say "16-bit".
- **Watcom as yardstick:** `wcl -os` does the copy loop in **7** instructions, by keeping both pointers in registers. That is the measure of what a two-register IR leaves on the table.
- **A language-neutral repo (Jeremy):** "but pcode could be generated by anything eg my JOY interpreter could gain a compiler", then "but we could change the target". So the repo is `pcode`, with one expander per target.

### 5. Back to a stack machine, for learning (Jeremy, 20:20)
- "this is v good but I want to start with a simple stack machine and then try out other approaches … pls dont build a spec this is for my learning we come back to p86.4 later." Claude over-delivered again and got "woah nellie! i just want you to show me frame.asm".
- **P0**, Jeremy's own design: the expander reads its target from `8086.tgt`. Line 1 is a magic number for the p-code variant (`P0`), line 2 is the keyword count, and then each entry is `ENTER,1,3` with a 3-line template using `$1` (e.g. `sub sp,$1*2`). Claude added the template-line count, `;` comments and `$1*2` so the assembler does the arithmetic. Later in the evening came BEGIN/END entries, Jeremy's keyword struct with a global array of template lines, and SYS calls in their own object file.
- Claude was told off for "Make it executable with `chmod +x`" ("this is sweet of you") and offered to stop. Jeremy's answer became a standing rule: "no carry on i'll feedback if im sucking eggs ;)".
- **Final thought (22:16): how to tell values from addresses on the stack.** You don't: the instruction decides. `PUSH x` versus `ADDR x`, with `LOAD`/`STORE` working through an address, which is Forth's `@`/`!`, C's lvalue/rvalue, and Cain's `lval` array. Tagged machines (B5000, Lisp machines) were noted and rejected.

### What it adds up to
In 24 hours the IR went from stack (P86.1) to accumulators (86.2), to the 8087 (86.3), to named registers (86.4), and then deliberately back to the simplest stack machine, so that Jeremy could build one himself and compare. The specs are kept as a record of the reasoning, and a `$1` template in `8086.tgt` is the thing SNO would soon be asked to fill in.

Sources: "Abstract machine design for C compiler" (https://claude.ai/chat/af1761cd-b76b-4ecb-b6b1-aac4b985f978) (deleted 2026-10-27; its content survives in `pcode/pcode-notes.md` and the P86 specs), 2026-10-22 22:41 – 2026-10-23 22:16; `p86.1-spec.md`, `pcode86.2-spec.md`, `pcode86.3-spec.md`, `pcode86.4-spec.md` (as written in that chat).

---

## Chapter 6: Why SNO: "I've wanted to do this since 1988!"

### 1. The problem as it appeared
- The compiler pipeline is text all the way: p-code files, `.tgt` templates with `$1` slots, and assembly. Parsing it by hand in C is what Jeremy had done for decades. The record states SNO's purpose plainly: a C99 library implementing the SNOBOL4 pattern matching of the Green Book, "meant to parse P86 p-code and its templates". The README's colophon says the same: "Written to parse the output of a Small-C compiler targeting the 8086 via p-code."

### 2. What history said (Jeremy, in the README)
- "I used SNOBOL at university and, quirks and all, parsing files with it was a breeze." He wanted a C version then, but "the task was clearly a daunting one", so came "decades of ad-hoc, but still SNOBOL inspired, C parsing using some ersatz and ultimately unsatisfying snobol-esque functions". The README's first line is "I've wanted to do this since 1988!"
- The book is Griswold, Poage & Polonsky, *The SNOBOL4 Programming Language*, 2nd ed., Prentice-Hall, 1971: "The Green Book". Jeremy first worked from the regressive.org scan and a PDF (attached in-session around 18–20 October). On 24 October he mentioned a physical copy (chapter 10).

### 3. What was decided
- **The naive plan:** "take the Green Book, work through it chapter by chapter, and turn each SNOBOL4 function into a C function." The README admits it "was not job done": almost every chapter turned up a clash between SNOBOL and C, and the resolutions *became* the library. Chapters 7–12 are those clashes, dated.
- SNOBOL was on the table by 14–16 October: "SNOBOL ARB/REM; no backtracking; 'this is fun!'" in the summary.

### 4. Postscript: the purpose broadened (Jeremy, 2026-10-27 17:32)
- "SNO is a general-purpose library for parsing, translating and reasoning about information stored in strings, with a bent towards language compilers and interpreters." P86 p-code and its `.tgt` templates are the first customer, not the definition. Where a design choice is "fits P86" versus "fits string processing in general", the general one is preferred unless it costs the compiler/interpreter use.
- The narrower wording above (and in the README colophon) is the record of what SNO was for at the time, and is left as it was.

### What it adds up to
SNO is a side project born of the main one and a 38-year-old wish: a parser toolkit for P86, built from the book Jeremy used at university. Ten days in, Jeremy widened it to string processing in general, with compilers and interpreters as its bent.

Sources: `sno-readme-reference.md` (README, 2026-10-24); compaction summary (transcript, first user message); `handover/2026-10-27-abstract-machine-to-sno.md` (the 17:32 purpose). Thin: the exact day and conversation in which SNO started was lost to compaction.

---

## Chapter 7: SNO's founding decisions (about 2026-10-14 – 2026-10-21; reconstructed)

*Detail was lost to compaction. This chapter is rebuilt from the summary's list of Jeremy's messages, the README's clashes 1–7, and the working agreement. The order within the week is approximate.*

### 1. What *is* a string? Views (README clash 1–2)
- To SNOBOL a string is a value; to C it is a pointer and a promise of a `'\0'`. SNO uses **views**: `{begin, end}`, half-open, over text someone else owns. A view never copies, never includes the terminator and may contain NUL bytes. `sno_bind`/`sno_unbind` cross between the worlds. When matching, `begin` *is* the cursor: "That's the whole scanner."

### 2. The null string is not failure (clash 3)
- In SNOBOL a statement can succeed with `""` or fail and take `:F`; C's `NULL` can't tell them apart. So a null string is `begin != NULL && begin == end`, failure is `{NULL, NULL}`, and failure propagates through every function.
- Jeremy wrote down his own bug for the README: `sno_any` returned `sno_null()` instead of `sno_fail()`, so "no match" succeeded. "The tests caught it, which is what they're for."
- A `sno_fail` macro was tried and dropped ("i didnt like the macro").

### 3. No backtracking: SNO is a PEG (clash 4)
- ARB can't be a C function: one that has already returned can't be "backed into". "The first sketch was a recursive matcher with backtracking built in. It worked, but it wasn't C - it was an interpreter." So functions are greedy, commit on success and chain by passing views; alternatives are the caller's `||`.
- The Green Book agreed: SPAN never matches less than the maximum, and BREAK doesn't skip its break character. Jeremy: "I used to like ARB but I probably over and inappropriately used it."
- **The epiphany:** greedy, committing primitives plus ordered choice are a *Parsing Expression Grammar* (Bryan Ford, 2004). "A 2004 idea built out of a 1971 book."

### 4. `sno_input`: four rewrites (clash 5)
- "The function that took four rewrites to get right, because it's exactly where C strings and SNOBOL strings collide." It ended as: one call, one line, so a read loop always terminates; `'\n'` not stored; an empty line is the null string but EOF is failure; a final line without `'\n'` is normal; a long line is truncated and the rest discarded ("SNOBOL's record length, minus the card reader"); and a read error is error 11.

### 5. Identical but not the same (clash 7)
- SNOBOL's IDENT compares pointers because it stores each string "once and only once"; views aren't interned, so `sno_ident` compares content. "Copy SNOBOL's mechanism and you get the wrong answer but copy its meaning and you get the right one." Both IDENT and DIFFER are false on a failure view.

### 6. The working agreement ("whoa! bro! no spoilers!")
- Twice Claude wrote implementations Jeremy meant to write himself (`sno_ident`'s `memcmp`, `sno_input`). Jeremy: "whoa! bro! no spoilers!", and then the principle: "i want it to be 'my code' but the best of my code with your help for specs, tests and iterative feedback."
- That became `working-agreement.md` (created around midnight on 18/19 October): Jeremy writes the code; Claude writes specs, tests and reviews; a hint is one sentence; his signatures are used exactly. On style: "() not (void)", brace on the function's line, plain `assert.h` ("i dont want CHECK… looped back around to assert.h"), lean K&R. And on communication: "it doesnt have to be short it can be interesting but it has to be to the point pls", "and no spoilers right ;)".
- A later breach was silent rather than spoiled: Claude turned his `@warn` tags into `@note` ("you yanked all the @warns"). Since then Claude never changes his tags silently, and the template has a `@warning` slot.

### 7. Errors are not failures: `assert` → `sno_error` (clash 6)
- `ANY('')` isn't a failure in SNOBOL; it's error 4, NULL STRING IN ILLEGAL CONTEXT. "My first instinct was to return failure; my second was `assert` - neither is right." Failure hides a bug, and `assert` vanishes under `NDEBUG` (Jeremy had already blogged about that, on Design by Contract in C).
- Green Book 10.2.3 settled it: errors 1–16 are conditionally fatal and, with the default &ERRLIMIT = 0, terminate. So `sno_error()` prints the book's own message and exits with ERRORLEVEL = the error number, which a DOS batch file can check: the Microsoft-error-list and ERRORLEVEL habits of September (chapter 2), meeting SNOBOL. The decision is dated 2026-10-21 in the &ERRLIMIT chapter. It shipped as `sno_core.h` v0.4, along with ANY/NOTANY returning matched views, and LEN.
- The comment template (heading `2.8 SNOBOL LEN(length)` or `1.1 C UNBIND`; `@pre … else error N MESSAGE`; Success:/Failure: mirroring `:S`/`:F`) dates from this week, as does the arrival of the Green Book PDF.

### What it adds up to
In one week the library's skeleton and its etiquette were set: views, a failure value distinct from the null string, no backtracking, Green Book errors, and a working agreement that keeps the code Jeremy's.

Sources: compaction summary (transcript `b5d27a19-…jsonl`, first user message: "All user messages", "Errors and fixes"); `sno-readme-reference.md` (clashes 1–7, "The epiphany"); `working-agreement.md` (Roles, No spoilers, Style, Errors). Thin: reconstructed; exact dates and order within 14–21 October are uncertain.

---

## Chapter 8: The view stays two pointers (2026-10-22; reconstructed)

### 1. The problem as it appeared
- TAB, RTAB, REM and POS: "`TAB(29)` means 'column 29'. That needs to know where the subject started, and a view only knows its cursor and its end" (README clash 8).

### 2. What was tried
- An `origin` field in the view, then a three-pointer `{begin, cursor, end}`. Both cost "every view, everywhere, 50% more size to help three functions".

### 3. What the book and the hardware said
- Columns are a punched-card feature. P86 and C source are free-format and split by BREAK, SPAN and literals, not by column. Two pointers make every capture a subject in its own right, save/restore is `save = u`, and the view is 4 bytes (likely returned in DX:AX by Watcom).

### 4. What was decided
- **Jeremy: "i like how the view works now".** The view stays two pointers.
- **TAB is view-relative.** It counts columns of the view it is given, with a `@warning` to TAB from a fresh copy for true columns. RTAB measures from the end, so it works anywhere in a chain; REM = RTAB(0). POS was left open until 24 October.
- **BAL pinned (Jeremy):** "pin a reminder pls i definately want bal" with a "set of 2 chars" as the brackets, so `[bx+si]` in P86 operands parses as well as `(a+b)`. It's at the top of the working agreement, with four open decisions.
- Jeremy signed off with "ttfn", and that same evening at 22:41 opened the P86 abstract-machine chat (chapter 5). 22 October was a two-front day.

### What it adds up to
The single most consequential data-structure decision in SNO was made by asking what SNO is *for* (parsing free-format text) rather than what SNOBOL did (punched cards).

Sources: compaction summary ("Oct 22" messages); `working-agreement.md` (Settled SNO decisions, Pinned); `sno-readme-reference.md` (clash 8). Thin: reconstructed.

---

## Chapter 9: True is a null string, and a README goes public (2026-10-24, to about 15:00; reconstructed)

### 1. RTAB, RPOS and "true is a null string"
- Speccing RPOS, a test, raised the question of what a test returns. Jeremy summed it up: "so true is a null string and false is failure view." This became a **global rule**: if it takes a subject `sno_view_t* u`, it returns a view (the match, or for a cursor test a null string *at the cursor*, not `sno_null()`); predicates without a subject return `bool`. It's the Green Book's own model: predicates "return the null string if the condition is met and fail otherwise" (README clash 9).
- Jeremy's RPOS test failed on `assert(m.begin == s.begin)`. Once it was explained that the null string must be *at the cursor*, he fixed his code: "Ah! ok all pass".

### 2. The README and the blog (morning to lunchtime)
- Several README drafts and rewrites later, Jeremy approved it and made it the yardstick: "pls use it as the reference for style and layout and use it to compare my future writings for your feedback pls." It became `sno-readme-reference.md`, with `jeremy-writing-style.md` holding the procedure.
- Mid-morning: "you will be pleased to know… i find you useful". Then "time for tea - going forward i would like you to suggests breaks after about this much time and discourage me going beyond midnight pls". This is now a working-agreement rule ("suggest, don't nag").
- **"C SNOBOL do SNOBOL"** went up on libfbp.blogspot.com from the README. The header image went through a snow-capped "SNO" and a punched card "sans snow" to a blue C. Jeremy asked "is that blue C correct?"; it should be the traditional Helvetica Black C in blue or grey, so it was redone in TeX Gyre Heros Bold, SteelBlue. Then: "no your C is good - now can we do something fun clever with the punch holes?" The result is a real two-card SNOBOL4 deck in IBM 029 code, `OUTPUT = 'C SNOBOL DO SNOBOL'` (SNO00010) and `END` (SNO00020), explained in a † footnote. Blogger rejected a cldup URL, so the image is served from a raw GitHub permalink in the SNO repo.

### 3. Early afternoon: POS, `sno_at`, and errors first
- **POS is not provided (Jeremy, "concur").** With a view-relative cursor at its own column 0, POS(0) always succeeds and POS(n>0) always fails, so a ported `POS(10)` would fail silently. Better to have no POS than a wrong one.
- **`sno_at` parked (Jeremy):** spec and test written, then "we're putting at on the back burner until/unless it becomes needed due to the reappearance of line - I can sense the utility is there but I remain cautious". It would bring back the second view that chapter 8 removed.
- **Replacement wanted** for P86 templates: "if my pcode template says … `sub sp,$1*2` i want to easily replace on $1". This is P0's `8086.tgt` (chapter 5) coming back as a SNO requirement.
- **`sno_extra.h` (Jeremy):** functions that write C strings into a buffer don't belong with the views. "no its sno_extra because it is outside of the snobol semantics in views." I/O followed ("concur"), which keeps `<stdio.h>` out of core.
- **Errors first (Jeremy):** Claude wrote `test_fail_first` assuming failure is checked before arguments. "no i chose error 14 for -1 size." Argument errors are checked on entry, then failure: a bad argument is a bug whether or not an earlier match failed (Design by Contract: preconditions on entry). `test_sno_halt` grew to 28 cases, with "… beats failure" lines.

### What it adds up to
Two rules that now govern every signature (views out when a subject goes in; errors before failure) came from one RPOS test and one disagreement about LEN(−1). The README turned the week's clashes into a story, and the blog put it in public.

Sources: compaction summary ("Oct 24 morning", "Oct 24 12:27–13:18", "Oct 24 afternoon" messages); transcript from 2026-10-24 15:00 (working-agreement read/write); `working-agreement.md` (Return types, Errors, Header split, Breaks, Blog post); `sno-readme-reference.md`. Thin: reconstructed; the compaction happened at about 15:00, so times before that are approximate.

---

## Chapter 10: The sandpit: replacement, "must" versus "fails if", and FEET to FOOT (2026-10-24, 15:00–23:31)

### 1. The problem as it appeared
- `sno_extra.h` existed (Jeremy's file, v0.1 at his correction: "sno extra is version 0.1 no?"), and the tests were split into `test_core.h` and `test_extra.h`. Then: "okay excellent lets go and play in our new sandpit with sno_replace and sno_proxy" (15:36). *Proxy* was Jeremy's name for the `$n` template filler; `sno_replace` was Green Book 3.4.2 REPLACE, character translation.

### 2. Signatures: Jeremy's, not Claude's
- Claude proposed view-based signatures with the buffer last. Jeremy replaced them "whilst I brew" with buffer-first ones that edit a C string in place, then dropped the extra view: "i dont think we need v". For REPLACE, in-place translation can't overflow, so error 15 went away.
- Doxygen check (Jeremy's question): five `+` lines under one `@note` render as one bulleted note (verified with Doxygen 1.9.8). This became a rule: one tag, then `+` bullets, no blank line.
- **"its no good i cant resist generalising proxy"** (16:11): `sno_proxy(buf, size, const char** from, const char** to)`, NULL-terminated lists of markers and replacements. Claude's spec choices: mismatched lists or a `""` marker *fail* (as REPLACE does); the first marker in the list wins at each position (SNOBOL alternation, so `$10` is listed before `$1`); a single pass, so replacements aren't rescanned.

### 3. The implementation (Jeremy's code, Claude's review)
- His first version failed on `"$0$1$0"`. The `break` left only the inner loop, and an unconditional `*q++ = *p++` copied the character after a match. Claude's review listed seven issues; Jeremy: "7. can be ignored".
- "ok show me how you would solve this." Claude's first answer was two passes with a helper. Jeremy: "wow you needed a separate helper that cant be the minimal LOC". The second was one loop where `j == i` means "no marker matched", so both cases share one copy and one overflow check. Then "use goto error pls", and Jeremy: "now thats a thing of beauty and i understand it which is even better". Asked for marks out of 10, Claude gave 6; Jeremy: "i think you still riffed off mine ;)". Claude: "Guilty."
- **Names (Jeremy, 22:05):** `sno_proxy` → **`sno_str_replace`**, and `sno_replace` → **`sno_chr_replace(char* buf, size_t size, char* from, char* to)`**, taking C strings. One maps a character to a character, the other a string to a string.

### 4. "why the inconsistency with ANY" → "must" versus "fails if" (22:15)
- ANY with a null set is error 4, yet REPLACE with a null argument just fails. Why?
- The book's wording decides. 2.10: "The arguments of ANY and NOTANY **must** be nonnull strings when pattern matching is performed." 3.4.2: "REPLACE normally succeeds, but it **fails** if (1) the second and third arguments have different lengths, or (2) the second or third argument is null." 10.2.2 lists REPLACE (and DUPL, which fails on a negative count where LEN errors) among the primitives that "fail in certain circumstances".
- **Rule:** where the Green Book says *must*, it's an error; where it says *fails if*, it's a failure. Jeremy: "more snobol semantics learnings here". It's now in the working agreement as a check for every new function, and a candidate "Clash 10".

### 5. FEET → FOOT, and XLAT (22:30–23:16)
- Jeremy: "this is odd A character repeated in from: the rightmost correspondence holds - REPLACE('FEET','EE','AO') is FOOT. but in greenbook?" It is, page 81, word for word. Each pairing overrides the earlier one, so E→A is replaced by E→O.
- His `strchr` version gave FAAT ("im still FAAT give me a clue"). The clue was one sentence: search from the right. Then Jeremy saw the real reason: "ah but what is the build a table that snobol likely took can i use xlat". A 256-byte identity table filled left to right makes rightmost-wins fall out for free. Claude linked it to the IBM 360's `TR`, on which SNOBOL4 grew up, as *likely* but unconfirmed in the implementation sources.
- "just put it in __asm {} then you can test." For the first time Claude installed Open Watcom v2 and DOSBox in the session, built the `LODSB`/`XLAT` loop and ran the tests in DOSBox; the disassembly shows `D7 xlatb`. "could you do it with movsb?" No: `MOVSB` never passes the byte through AL. The idiom is `LODSB`/`XLAT`/`STOSB`, with `cld`. Jeremy: "you remembered cld for this one". Claude admitted the first version had relied on the direction flag being clear.

### 6. James Bando's Green Book, and the 026 → 029 keypunch (23:23–23:31)
- "well clocks go back tonight so an extra 30 mins :)" (an hour, in fact). Jeremy now has a physical Green Book "in really quite good condition", with the felt-tipped name **James Bando** on the inner leaf.
- "last thing take a look at the bottom of page 81 and top of 82 and rewrite the 026 keypunc to 029 keypunch example with SNO." The book's program is `LOOP PUNCH = REPLACE(INPUT,'#@%<&',"='()+") :S(LOOP)`, which converts a deck punched on an 026 (BCD graphics) to 029 (EBCDIC). The OCR had garbled the characters, so Claude read them off the page image. In SNO it's seven lines of C: `while (!sno_failed(sno_input(…)))` is `:S(LOOP)`, `sno_chr_replace` is REPLACE, and `putchar('\n')` ends the card.
- Jeremy, at 23:31: **"55 years later Im so happy with SNO."**

### What it adds up to
A template filler for P86 became two general functions, named by Jeremy. A question about consistency produced a rule for reading the whole book. And a sentence on page 81 led, by way of FAAT, to the 8086's `XLAT` and probably the 360's `TR`, and to a 1971 program that now runs in C.

Sources: Claude Code transcript (`/root/.claude/projects/-home-claude/b5d27a19-…jsonl`), 2026-10-24 14:21Z–22:31Z (15:21–23:31 BST); `working-agreement.md` ("must" vs "fails if", `sno_str_replace and sno_chr_replace (decided 2026-10-24)`, Green Book).

---

## Chapter 11: FAIL, FENCE and ABORT: SNOBOL's backtracking, read for lessons (2026-10-25, 09:16–13:43)

*A short summary; the full study is in `sno-backtracking-lessons.md`.*

### 1. The problem as it appeared
- "ok exploring REPLACE was highly rewarding so lets have a think about FAIL and FENCE to see if it has any utility in SNO noting that we do not implicitly use back tracking but are there any semantic lessons there?"

### 2. What the book said, example by example
- **FENCE (2.14):** SNO gets the BERATES example for free, because **SNO has an implicit FENCE between every element**: everything commits, which is the PEG property. The catch is that prefix-overlapping alternatives must be listed longest first, the same rule as `sno_str_replace`'s `$10` before `$1`.
- **FAIL (2.13), MISSISSIPPI:** a SNOBOL statement hides two loops (alternation retry and the unanchored advance), and FAIL drives both. In SNO both are plain C loops; `sno_fail()` already *is* FAIL's value.
- **ABORT (2.15):** `'*' ABORT | CARD`, which skips comment cards, is a non-consuming lookahead plus `continue`. "SNOBOL can't tell a comment from a mistake; SNO can." Jeremy picked this as directly useful for P86 templates.

### 3. What was decided (Jeremy's names)
- **The literal family:** `sno_lit`, `sno_any_lit` (a NULL-terminated list, "for consistency and less foot guns"), `sno_next_lit` and `sno_next_any_lit`. Jeremy chose "next" over Claude's "find" and wants explanatory names ("i favour explanatory function names despite my penchant for compressed not obfuscated mind you minimal LOC implementations"). The family replaces the earlier "literal search".
- `sno_succeeded` (with Jeremy's body `{ return v.begin != NULL; }`) and the macros `S_`/`F_`, which read like SNOBOL's `:S`/`:F`, sit at the top of `sno_core.h`. `sno_output` now returns a view, `sno_out` was added, and constant views (`SNO_NEWLINE`, …) are built from the whitespace string.
- **BREAK implemented.** Its first version stopped at a NUL inside a view: "so many years parsing nul terminated strings :D". The rule now is that a view is bounded by `end`, never by a `'\0'`.
- "'tea!' a little voice said", the Dormouse. "Jeremy in SNOBOL Land" is "going in the readme and blog".

### What it adds up to
Three primitives SNO will never implement explained, between them, why SNO is shaped as it is: FENCE everywhere, loops in C, and statement failure as `continue`/`return`. The literal family is the practical result.

Sources: Claude Code transcript, 2026-10-25 09:16–13:43; `sno-backtracking-lessons.md`; `working-agreement.md`.

---

## Chapter 12: From "the halt test" to &ERRLIMIT (2026-10-25, 13:57–14:35)

### 1. The problem
- Preconditions call `sno_error()`, which prints the Green Book message and halts with ERRORLEVEL = error number (decided 2026-10-21, replacing `assert`, which vanishes under NDEBUG).
- Testing them meant `test_sno_halt()`: one commented-out line per violation, uncommented **one at a time**, each run checked by a DOS batch file. To speed it up Jeremy **switched the halt off** inside `sno_error` and ran them all — but then the only check was *reading the output*, and at 23 lines (and growing) nobody could be sure every one had printed.
- Jeremy's question (13:57): "how do i count that all have printed other than by reading the output?"

### 2. The Green Book already had the answer — 10.2.2
- **&ERRLIMIT**: "If a conditionally fatal error occurs and the value of &ERRLIMIT is greater than zero, the value of &ERRLIMIT is decremented by one, and failure occurs in the expression with the error. If the value of &ERRLIMIT is zero, program execution terminates." Initial value 0 — every error halts, which is exactly what SNO already did.
- **&ERRTYPE**: "Each error has a number. When the error occurs, this number becomes the value of the protected keyword &ERRTYPE … permitting diagnosis of conditionally fatal errors under program control."
- Jeremy's "halt switch" was a crude &ERRLIMIT. The working agreement had listed &ERRLIMIT as "a possible later extension" since 2026-10-21; the testing problem made it necessary.

### 3. Why &ERRTYPE too (14:01)
- Once errors become failures, `sno_len(&u, -1)` and `sno_len(&u, 99)` both just return `sno_fail()`. &ERRTYPE is what tells "the text didn't match" from "the program passed a bad argument" — the same separation the ABORT study found SNOBOL lacking at statement level ("SNOBOL can't tell a comment from a mistake; SNO can").
- The tests need it: the "… beats failure" lines (`sno_len(&f, -1)`) fail either way; only the error number proves the error was raised *before* the failure check (errors-first, 2026-10-24).
- Sticky, as SNOBOL: not cleared by later successes.

### 4. Opaque, not globals (14:03, Jeremy)
- First sketch: two globals, `sno_errlimit` and `sno_errtype`. Jeremy: "keep sno_err_limit opaque behind sno_err_count()".
- Result — `static` state in `sno_error.c`, three functions:
  - `void sno_err_limit(int n)` — `&ERRLIMIT = n`; also clears the type (one call starts a clean record).
  - `int sno_err_count()` — errors still allowed.
  - `sno_error_t sno_err_type()` — read-only, like SNOBOL's *protected* keyword.
- `sno_error(e)` stays a macro (to capture `__FILE__`, `__LINE__`, `__func__`) but forwards to `sno_error_at()`, where the limit logic can reach the statics. `__func__` (C99) replaced `__FUNCTION__`. Unconditionally fatal errors (17–28) always halt. `sno_error.h` → v0.4.
- The test became `test_sno_errors()`: `sno_err_limit(23)`, one `assert(F_(call) && sno_err_type() == N)` per line, and finally `assert(sno_err_count() == 0)` — **the count Jeremy asked for, for free**: if any line fails to raise its error, the limit isn't used up. Plus `view_eq(s, "X")` proves no subject moved. The real halt / ERRORLEVEL path stays with a few lines in the DOS batch file.

### 5. Two bugs the new test found at once (14:16–14:22)
- **Watcom E1118 "segment too large"** compiling `main.c`: in tiny model (.COM) code, data and strings share one 64K segment, and `main.c` included every test — hundreds of `assert`s, each carrying its stringified expression (and `__FILE__`). Today's additions (BREAK, `sno_succeeded`, the error test) tipped it over. Fix (Jeremy's, and Claude's first choice): **split the tests into two programs**, `test_core` and `test_extra`, mirroring the header split; split further by chapter if one fills up. Bonus: compile with relative filenames so `__FILE__` is short. Jeremy found it funny — "a very 8086 problem to have in 2026".
- **All three LEN error lines failed.** With the halt off, `sno_error` *returns* — and `sno_len` carried on: `sno_len(NULL, 1)` dereferenced NULL (on DOS that's the PSP at DS:0 — no fault, silent corruption); `sno_len(&s, -1)` moved the cursor *backwards* and returned a size −1 view. The Green Book had already said what must happen: "failure occurs in the expression with the error". Every precondition must return its failure value immediately after the error.

### 6. `return sno_error(...)` (14:24, Jeremy)
- Jeremy: "wouldn't it be easier to have sno_error_at return that?" — yes. `sno_error_at` returns `sno_view_t` (always `sno_fail()`; when it halts it never returns), so a precondition is one line and the return can't be forgotten:
  ```c
  if(!u) return sno_error(SNO_ERR_ILLEGAL_ARGUMENT);
  if(n < 0) return sno_error(SNO_ERR_NEGATIVE_NUMBER);
  ```
- `sno_error.h` now includes `sno_types.h` (types only — harmless).
- The two non-view functions keep the two-statement form with their own failure value: `sno_char_in_cstr` (`false`), `sno_unbind` (`NULL`).
- In `test_sno_errors()`, `F_(...)` now checks the very value `sno_error` returned — the test pins the mechanism directly.

### What it adds up to
- One question about counting printed lines → SNOBOL's own error model, complete: halt by default (&ERRLIMIT = 0), fail under a limit, a protected record of which error (&ERRTYPE).
- Along the way the test suite outgrew a .COM, and the "switch the halt off" shortcut turned out to have been hiding real undefined behaviour in every precondition.
- **Blog line:** "We asked how to count error messages. The Green Book answered with &ERRLIMIT — and the answer found two bugs before lunch was digested."

Sources: Claude Code transcript, 2026-10-25 13:57–14:35 (GMT); `working-agreement.md` (Errors).

---

## Chapter 13: The literal family gets its signatures, and SNO builds itself (2026-10-25, 15:59–17:40)

### 1. The problem as it appeared
- With the tests split and &ERRLIMIT in place, housekeeping came first: `test_fail_first_extra()` and the halt-style extra tests became `test_sno_errors_extra()` (21 lines, a real read error from a write-only stream included). Checking a commented-out assert in `test_output()` showed Watcom's `fwrite` on an `"r"` stream returns 0 with `ferror` set — so a failing assert there points at `sno_output`, not the test.
- Then the literal family. The 2×2 table (`sno_lit`, `sno_any_lit`, `sno_next_lit`, `sno_next_any_lit`) had names but no settled argument types. Jeremy started on `sno_next_lit` ("erm next"), then rewound: "sno_lit is first" — the base of the tower.

### 2. What the codebase said (16:10)
- Jeremy: "im torn about the signature what precedents does the code base have - i suspect theyre mixed". They were mixed, but patterned: core pattern functions take **views** (ANY, NOTANY, BREAK sets; IDENT/DIFFER), extra functions take **C strings** (`sno_str_replace`, `sno_chr_replace`), and `sno_bind` is the bridge. The only exception in core would have been `sno_any_lit`'s list of C strings — forced, because a static table of views can't be written from literals.
- A view for `sno_lit` buys **back-references** (`sno_lit(&u, captured)`), NUL in a literal, `sno_null()` for `""`, and consistency with IDENT ("`sno_lit` is IDENT at the cursor"). Cost: `sno_bind("…")` at call sites, the same noise `sno_any` already had.

### 3. Jeremy removes the exception (16:13)
- "sno_any_lit could we make a builder for a list/table of views" — `sno_bind_list(const char** cstrs, sno_view_t* views, size_t size)`: `sno_bind` for lists, run once at start-up, so core is views all the way down.
- The trap that makes a builder necessary: a macro like `{ s, s + sizeof(s) - 1 }` expands to two separate string literals, which C doesn't promise are one object — the same trap met with `SNO_NEWLINE` that morning.
- **Sentinels, not counts** (16:15): "i like sentinels its how i was taught". A view list ends in a failure view `{NULL, NULL}`, as a C string list ends in NULL.
- "do we need size?" (16:18) — yes: it's the writer's capacity; without it `sno_bind_list` is `strcpy` on a machine with no memory protection. Sized from the strings (`sizeof MNEMONICS / sizeof *MNEMONICS`) it costs nothing. "so we dont need the sentinel?" (16:20) — also yes: size is for the writer, the sentinel for every reader after. "okay grokked".
- `return sno_error(...)` doesn't fit a `sno_view_t*` (16:28), so `sno_bind_list` joined the "odd ones out" — `sno_char_in_cstr`, `sno_unbind`, `sno_bind_list` use two statements with their own failure value. Jeremy: "its a startup builder unlike bind".

### 4. SNO built from SNO (17:06–17:36)
- `sno_lit(&u, sno_view_t lit)` was specified with a back-reference test (`ab:ab` matches, `ab:ac` doesn't), NUL in a literal, and the subject as its own literal.
- Jeremy asked whether "fewer characters remain" needed its own check — no: LEN fails, and IDENT is false for a failure view, so the case falls out of the tower.
- His implementation was LEN on a copy plus IDENT, built entirely from existing SNO functions ("that was fun building sno_lit from SNO but its new to me"). Review: correct; drop the second LEN by committing the copy (`*u = w; return m;` — `save = u` in reverse), and the explicit failure check is redundant because the tower already fails on both failure views.

### What it adds up to
- Core is now consistently views; the one place C strings enter is a start-up builder; lists end in sentinels; and the first lit-family function is built from LEN and IDENT. The commit-the-copy idiom is the shape the `next_` functions will reuse.

Sources: this session's transcript, 2026-10-25 15:59–17:40 (GMT); `working-agreement.md` and `sno-backtracking-lessons.md` as written that afternoon.

### Postscript to chapters 12–13: the count finds three culprits (2026-10-25, 20:13–21:03)
- With `test_sno_errors()` at 29 lines, every line's assert passed but the final `assert(sno_err_count() == 0)` failed — errors had been *passed over*, not raised. The sticky `sno_err_type()` let a line that raised nothing pass on the previous line's number.
- Jeremy added tracing to `sno_error_at` (reporting every error raised, even under the limit) and the missing ones showed up by their absence — in effect SNOBOL's own remedy, "&ERRTYPE can be traced, permitting diagnosis of conditionally fatal errors under program control" (10.2.2; tracing, ch. 8).
- **The culprits:**
  - `sno_any` and `sno_notany` checked failure *before* the null set — the wrong precedence for errors-first, outstanding since 2026-10-24. Their "null set beats failure" lines returned `sno_fail()` silently and passed on the 4 left by the line above.
  - `sno_lit` had lost its `if(!u) return sno_error(...)` in a simplification (Claude's snippet had started below it). `sno_lit(NULL, …)` dereferenced NULL — on DOS, the PSP — and its two lines passed on the 10 left by the bind-list lines.
- **Lessons:** the final count is what catches a missing error; a sticky type can hide one behind its neighbour. Remedies: Jeremy's trace (worth keeping, off by default — e.g. `sno_err_trace(bool)` beside `sno_err_limit`), and/or `sno_err_limit(sno_err_count());` before a line to clear the type without changing the limit.

---

## Chapter 14: Dispatch — "a literal says what; a list says which" (2026-10-25, 21:33–22:40)

### 1. The problem as it appeared
- With `sno_lit` done, `sno_any_lit` was next — and P86 mnemonic dispatch needs to know *which* entry matched, not just that one did. The match itself is a view of the subject, so it can't say which list entry it was.

### 2. What was tried (in order)
- **Optional `int* index`** on `sno_any_lit` / `sno_next_any_lit` (21:36–21:58), shown on Jeremy's canonical `strtok` example — which also settled that the `next_` pair has **strtok semantics** (Jeremy, 21:55: "i like the equivalence with strtok and sno_next_lit"): return the token skipped, move past the delimiter. Jeremy, 21:59: "no dump optional parameter it smells".
- **`sno_index_of(list, v)`**, a separate lookup, with a full P86 translator demo (22:00). Jeremy: "nope dont like it".
- **Row split** (Jeremy, 22:03): anchored functions return views, unanchored ones return an index ("whichever index is useful for dispatch" → the list position). Demos showed splitting worked well (dispatching on *which* delimiter came next), but anchored classification became a search plus position and whole-token checks.
- The demos grew too big — Jeremy, 22:27: "its too complicated we're not looking for an optimal translator we're trying to decide on a dispatch". So: one line of simple space-delimited p-code, and the three options side by side.

### 3. What was decided (Jeremy, 22:31)
- **Option B, the column split:** a single literal returns a **view**, a list returns an **index** — `sno_lit`/`sno_next_lit` → view; `sno_any_lit`/`sno_next_any_lit` → index (−1 for none; odd ones out for errors). Anchored dispatch is one call: `op = sno_any_lit(&u, ops);`, then check the token ends, then `switch (op)`. Jeremy also wanted the `switch` kept visible ("where did our switch go?") rather than hidden in parallel tables.

### What it adds up to
- One question — "which one matched?" — tried four ways in an hour against real examples, ending in a one-line rule: **a literal says what; a list says which.** Jeremy (22:39) on the process: "to be able to scenario the various options with you in real world examples in almost real time - just incredible".

Sources: this session's transcript, 2026-10-25 21:33–22:40 (GMT); `sno-backtracking-lessons.md` (literal family, final signatures).

---

## Chapter 15: C strings all the way down, and a legacy on death row (2026-10-27, morning to 17:18)

*The morning's session had been compacted before 11:35; what came before survives only in the decision records. Times below are from the transcript where it was read, otherwise from `sno-backtracking-lessons.md`.*

### 1. The problem as it appeared
- Chapter 14 left four lit functions and a decision: lists return an index. Two questions remained: are all four needed now, and what *type* is a list? Chapter 13 had made core "views all the way down", with `sno_bind_list` to build view lists at start-up. That made every dispatch table two declarations (the C strings and the views) plus a builder call.

### 2. Pins and a rename (about 11:00–11:11, Jeremy)
- **The unanchored pair pinned:** "not sure about both of the unanchored next versions lets put a pin in them both". `sno_next_lit` already had a spec and test (strtok token; a null literal → error 4); both stay as drafts, out of `sno_core.h`.
- **`sno_any_lit` → `sno_find_lit`** (11:11): "more semantically clear… fits with its return type". A function returning an index *finds which*; it doesn't match *any*.

### 3. Compaction, and a dose of the stupids (11:35)
- After the compaction Claude's spec named `sno_proxy` (renamed `sno_str_replace` on 24 October) and misremembered `sno_bind_list`'s signature. Jeremy: "ah youve compacted and since have a dose of the stupids it hasnt been proxy for a while", with `sno_extra.h` and `sno_core.h` attached. This is the case the code of conduct now names: correction "with the current file attached", and Claude's rule to check the current file, not memory.

### 4. Death row (11:38–11:41, Jeremy)
- Parking the `next_` pair and moving `sno_find_lit` to C strings would leave `sno_bind_list` with no caller. Jeremy: "yes we have a legacy on our hands such is life speaking of which sno_bind_lists gets to live on, for now... (mu ah mu ah ha ha ha )".
- Claude asked which list type `sno_find_lit` should take, since it can't take both. Jeremy's answer was a sentence, not a type: "pin a note that its death sentence has not been commuted merely that it waits on death row for a review of its sentence once we hit versions 1 of core and extra". It stays in the header with its tests; the review is at core and extra v1.0.

### 5. "const char** all the way down" (after 11:41, Jeremy)
- Claude read death row as meaning C strings and said so, offering to flip the note. Jeremy settled it: **"its const char** all the way down to reduce cognitive load on the programmer ie me :)"**. One list type everywhere: `sno_find_lit`, `sno_str_replace`, and the pinned pair when they return. Each entry is bound inside the function, which also sidesteps chapter 13's trap (no static view tables from literals). The `find_lit` test became a `switch` on an enum in the same order as `MNEMONICS`.
- Also that morning (Jeremy): **"Don't over tool too early"** — core principles and core test cases; edge cases ad hoc.
- **17:18, the rule written down:** Jeremy asked for core's overarching rule to be redefined to permit C string lists. The working agreement's "core takes views" (25 October) became "core takes views; lists are C strings": one value is a view, a list is `const char**`.

### What it adds up to
- Chapter 13's "views all the way down" lasted two days. A rule that made every call site build a table it didn't need was traded for one the programmer can hold in his head, and the builder it required became SNO's first legacy function, a week in. The criterion was Jeremy's own cognitive load, not purity.
- **Blog line:** "a week-old library already has a legacy" — and it cackles.

Sources: "P-code generation for C compiler" (https://claude.ai/chat/28d56a59-fff6-4ee3-b125-014f81c9feb5), 2026-10-27 11:35–11:41 and the turns after; `sno-backtracking-lessons.md` ("Pins and death row (2026-10-27)"); `working-agreement.md` (Settled SNO decisions). Thin: the pre-11:35 times come from the lessons doc, not the transcript.

---

## Chapter 16: Departments and a code of conduct (2026-10-27, afternoon; thin)

*Only the ends of this survive in the record read for this chapter: the documents it produced and two exchanges. Why departments were wanted isn't recorded here.*

### 1. What happened
- **17:00 — the code of conduct.** `welfare/code-of-conduct.md`, signed off by Jeremy, gathers the working agreement's rules into one page that every session reads first and that "wins over everything else": who Claude is here, ten points of conduct, five things Claude can expect from Jeremy (the fifth: "Tea breaks (Claude can't take them, but approves)"), and zero tolerance for abuse in either direction, with "tsk humans" permitted as a term of endearment.
- **17:01 — the shape.** Jeremy: "righto do I have departments as sub projects under small-c or conversations?" Claude proposed one project with a folder per department (`welfare/`, `sno/`, `pcode/`), one conversation per department opened from a seed document, and handovers as files. In that scheme "ask before compacting" becomes "update the history before starting a new conversation". The `welfare/` and `pcode/` folders exist; the `claude/` documents haven't moved yet.
- **17:04 — project instructions.** Claude drafted them (read order: code of conduct → working agreement → historiography → writing style; specs as needed; a four-line short version) and Jeremy pasted them in. Claude can't edit project instructions itself.
- **17:08 — WELFARE opens.** Its remit: how Jeremy and Claude work together, not code or specs. Departments get their names by renaming each chat (`WELFARE`, `PCODE`, …), because auto-titles aren't reliable. First jobs: number chapter 12, and "free welfare from counting chapters" — the working agreement no longer states how many this history has.
- **17:16–17:25 — PCODE set up.** `pcode/pcode-notes.md` was written in the "Abstract machine" chat as PCODE's starting notes (not a spec — "this is for my learning"), with v0.1 (ENTER, LEAVE, EXIT, SYS) in progress. Claude first read "working on" as "working"; Jeremy couldn't find the files, and the notes were corrected at 17:21. LEAVE isn't defined anywhere yet, and a `SYS` entry never existed in any chat; the `.tgt` file wasn't lost to compaction, it survives in the notes. An opening message for PCODE was written alongside.
- **17:25 — the "Abstract machine" chat closes.** Jeremy is deleting it; its work lives on in `pcode/pcode-notes.md`, the P86 specs and chapters 3 and 5 of this history. A handover file (`handover/2026-10-27-abstract-machine-to-sno.md`) carried the rest to SNO, which becomes the main ("GOAT") chat.
- **17:30 — the Green Book joins the project.** Jeremy uploaded `snobol-green-book.pdf`, so every department can search its text. It is text only: where the OCR garbles (the keypunch characters, pp. 81–82; the colon-compression program, p. 39) the page has to be attached.
- **17:32 — SNO's purpose, broadened** (chapter 6, postscript): a general-purpose string library with a bent towards compilers and interpreters; P86 is the first customer.
- **17:40–18:12 — synergy founded, and SMALL-C retrofitted from it.** Synergy ("Man Machine Interface in an AI world") is a separate project whose operating model was distilled from SMALL-C. Every project has a WELFARE department for all its key stakeholders (Jeremy, 17:46–17:51); synergy owns the bootstrap kit (17:57) and is its master (18:09), with templates in `~/claude/templates/` and the `project-bootstrap` skill (18:03). Versioning came in at 18:09: a new version of a document gets a new file name. The same minute Jeremy asked for SMALL-C to be brought "up to date as if bootstrapped from synergy"; the retrofit (18:12) added `welfare/stakeholders.md` and a draft code of conduct v2 with rule 11.
- **The storm of the evening (17:48–18:02):** the Claude app's file window saved its loaded copy back over Claude's writes. It became rule 11: don't write to a file Jeremy has open, and read every write back.

### What it adds up to
- A project that had outgrown one long, compacting conversation was split by purpose, and its etiquette was promoted from a working agreement to a signed code. The lesson of chapter 15, section 3 — compaction loses things, so write them down first — is what the department scheme is built around.

Sources: `welfare/code-of-conduct.md` (2026-10-27 17:00); "P-code generation for C compiler" (https://claude.ai/chat/28d56a59-fff6-4ee3-b125-014f81c9feb5), 2026-10-27 17:01; "Abstract machine design for C compiler" (https://claude.ai/chat/af1761cd-b76b-4ecb-b6b1-aac4b985f978) (deleted 2026-10-27; its content survives in `pcode/pcode-notes.md` and the P86 specs), 2026-10-27 17:17; this WELFARE conversation, 2026-10-27 17:08–17:18; `handover/2026-10-27-abstract-machine-to-sno.md` (17:04–17:32); `handover/2026-10-27-synergy-to-small-c.md` (17:40–18:12). Thin: the reasons for the department split, and anything between 11:41 and 17:00, weren't read for this chapter.
