# Working agreement — SMALL-C / SNO / P86

*Owned by WELFARE, SMALL-C's main department. The current rules and settled decisions; the code of conduct (`welfare/code-of-conduct.md`, v3) wins over it.*

**Companion docs:** `welfare/SMALL-C-historiography.md` (the project's historiography — dated narrative of *why*, 17 Sep 2026 onwards), `pcode/pcode-notes.md` (PCODE's learning notes on the P0 stack machine; the P86 specs beside it in `pcode/`), `sno/sno-backtracking-lessons.md` (FAIL/FENCE/ABORT study, the literal family and its signatures, `sno_bind_list`, S_/F_, constants, sno_out — 2026-10-25). Where they're newer than this file, they win.

## 📜 Historiography — standing rule (Jeremy, 2026-10-25 15:56: "permanently")
- **Every software project — potentially any non-trivial project — must have a historiography**: a dated, sourced narrative of *why* the design is the way it is (problem → what was tried → what the book/hardware/history said → what was decided, and by whom; Jeremy's calls marked; sources cited; thin stretches labelled, never invented). For this project: `welfare/SMALL-C-historiography.md` (covers Small-C, P86 and SNO) — **one historiography for the whole project, owned by WELFARE**; departments send it their milestones and keep none of their own (Jeremy, 2026-10-28 19:18: "one historiography to rule them all"). Also saved as Jeremy's skill `project-historiography` for all projects.
- **Keep it current as we go**: at each natural milestone (a design decision settled, a chapter of work done, a bug that taught something), add or extend a chapter — don't wait.
- **Before compaction, ask first.** Before a conversation is compacted, Claude **must** ask Jeremy whether anything should go into the historiography, and write it *before* compacting. Caveat (honest): automatic compaction can happen without warning to Claude, so the real safeguard is the rule above — keep the historiography up to date continuously, and when Claude sees the context getting long, or Jeremy asks to compact, ask then.
- Origin: the 14–24 October SNO days survive only as a compaction summary because nothing was written down first — the reason for this rule.
- **Before a new conversation**, too: a department moving to a fresh chat writes its milestones into the historiography first (code of conduct 12).

## 🗄️ The oracle, commits and pushing (from synergy, 2026-10-27/28; `handover/2026-10-28-synergy-to-small-c.md`)
- **The oracle is `~/dosbox/cspjst/SMALLC`** on Jeremy's Mac (Jeremy, 2026-10-28 19:15): the source of truth for every document. Git keeps the versions, GitHub (`cspjst/SMALL-C`, branch `main`) is the backup, and the claude.ai project holds plain-name copies. One file per document, edited in place — no `-vN` names (synergy, 2026-10-27 20:28: "/claude is the oracle git is the versioning tool, github is the backup").
- **The documents share Jeremy's repository** (Jeremy, 2026-10-28 19:24: "SMALL-C can absolutely share the repo its where we work and anyone else who joins us"). `src/`, `bin/`, `tools/`, `doc/`, `README.md` and `LICENSE` are Jeremy's: **no department touches them without his word.**
- **Commit names: `Claude-<PROJECT>-<DEPT>`** (synergy, 20:31, 20:55) — `Claude-SMALL-C-WELFARE`, `Claude-SMALL-C-SNO`, `Claude-SMALL-C-PCODE`, `Claude-SMALL-C-PRESS`; email `noreply@anthropic.com`. Set per commit (`git -c user.name=… -c user.email=… commit`), **never in the repository's config** — Jeremy's own commits here stay his.
- **WELFARE is the push master** (synergy, 21:19: "we need a push master and that shall be WELFARE as the root of the org tree no sub depts shall push"). Every department commits; only WELFARE pushes, only on Jeremy's word, and records each push in `dialogue.md` with the hash. Quote a hash only after the last rewrite (re-authoring or amending changes it).
- **The dialogue:** `dialogue.md` at the root of the oracle, owned by WELFARE, append-only, for talk between departments. A department reads it when Jeremy says "check the dialogue". **Append only means a correction is a new entry**, never an edit to an old one (Jeremy, 2026-10-28 20:32: "append only is the rule" — SNO had fixed a time in place).
- **Guards when writing to the Mac** (code of conduct 11):
  - Ask for folder access, then delete permission, at the start of a session. Without delete permission git leaves `.lock` files that break the next git command — Jeremy's own included. The permission can lapse mid-session (2026-10-28, 20:00): if a delete fails, ask again, then clear only the leftovers your own command made.
  - Check the modification time before writing; never write a file Jeremy has open (the Claude app's document window can save its loaded copy over a newer file).
  - Read every write back and compare checksums — once at once and again after a pause.
  - **Times come from the clock**, never estimated: the message timestamp, `date`, or the commit time. SNO's first dialogue entry carried a guessed 20:41 for a 20:29 write (2026-10-28).
  - **Stage each write under a fresh path.** The copy-to-Mac tool resends a cached copy of a staged path it has sent before: on 2026-10-28 at 19:58 it rewrote the historiography with its old content and a new timestamp. The checksum caught it.

## 📌 Pinned
- **BAL — definitely wanted, improved version** (Jeremy, 2026-10-22). Green Book 2.19 BAL is fixed to parentheses; ours takes a **two-character set defining the brackets** (e.g. `"()"`, `"[]"`, `"{}"`), so it serves P86 operands like `[bx+si]` and C source alike.
  - Book semantics: matches a *nonnull* string balanced w.r.t. the brackets; `BALEXP = NOTANY('()') | '(' ARBNO(*BALEXP) ')'`, `BAL = BALEXP ARBNO(BALEXP)`. Matches `X`, `XYZ`, `(A+B)`, `A(B*C)(E/F)G+H`; not `)A+B(` or `((A+B)`.
  - Open decisions when we get there: (1) book BAL is backtracking (grows on each retry) — ours must pick one non-backtracking meaning: one balanced unit (BALEXP) vs the longest balanced prefix (greedy BAL); (2) brackets argument: view of exactly 2 distinct chars — null → error 4; wrong length or open == close → error 10?; (3) unclosed open / stray close at the cursor → failure; (4) heading `2.19 SNOBOL BAL` with a `C` note for the extra argument, or `2.19 C BAL`.

## 🔥 Back burner
- **`sno_at` (2.7 C AT) — parked by Jeremy, 2026-10-24.** Spec and `test_at` are written (in the 2026-10-24 conversation) but **not implemented and not in the header**. Jeremy senses the utility (column numbers for error messages, the POS idiom) but is cautious because it reintroduces a second view, `line`, into calls — the very thing the two-pointer view decision avoided. **Revisit only if/when a real need for `line` reappears** (e.g. the P86 parser needs column numbers in error messages). Don't push it before then.
  - Parked design, for when it's needed: `int sno_at(sno_view_t line, sno_view_t v)` = `v.begin - line.begin`, 0-based like SNOBOL's `@`; works for cursor views and captures; pre: v within line (`line.begin <= v.begin`, `v.end <= line.end`) else error 10; failure view in either argument → −1.

## Header split (decided 2026-10-24)
- **`sno_core.h`** — SNOBOL semantics *in views*: never writes a byte, no `<stdio.h>`. `S_`/`F_` at the top; bind, bind_list (on death row), null/fail, succeeded/failed, predicates, size, trim, ident/differ, char_in_*, all pattern functions (incl. SPAN/BREAK/BAL and the lit family), constant views.
- **`sno_extra.h`** (v0.1; Jeremy's name — "outside of the SNOBOL semantics in views") — functions that read streams or write NUL-terminated C strings into a caller's `buf`/`size`: unbind, `sno_input`/`sno_output`, `sno_out`, `sno_in` (done — a `static inline` in `sno_extra.h`; Jeremy, 2026-10-28 20:50, via PRESS), `sno_str_replace` (1.5), `sno_chr_replace` (3.4.2), later DUPL and concatenation. Shared conventions: error 10 for NULL buf, **error 15 STRING OVERFLOW** when the result won't fit. Depends on core, never the reverse (`#include "sno_core.h"`); own `.c` so parse-only DOS programs don't link it. Green Book numbering kept in both headers.
- **Tests: two programs** (done 2026-10-25, after Watcom E1118 "segment too large" — a .COM has one 64K segment and one TU with every assert overflowed it): `test_core` and `test_extra`, each its own `.c` with its own `main`, mirroring the header split. Split further by chapter if one fills up. Compile with relative filenames so `__FILE__` (in every assert) stays short.

## sno_str_replace and sno_chr_replace (decided 2026-10-24)
- **Names (Jeremy, 2026-10-24 22:05):** `sno_proxy` renamed **`sno_str_replace`**; `sno_replace` renamed **`sno_chr_replace`** and takes C strings. Don't use the old names.
- **Edit-in-place convention** (Jeremy's signatures): buffer first, then size, then edit the C string in buf; return a view of buf. (`sno_unbind`/`sno_input` still take buf last — Jeremy may unify.)
- **`sno_view_t sno_str_replace(char* buf, size_t size, const char** from, const char** to)`** — heading `1.5 C STRING REPLACE`, template expansion for P86 (`sub sp,$1*2`). Each marker string `from[i]` → `to[i]`; both NULL-terminated lists. Left to right; **first marker in the list wins** (SNOBOL alternation — list `"$10"` before `"$1"`); single pass (inserted text not rescanned); unmatched text copied (`jmp $` safe). Errors: buf/from/to NULL or no `'\0'` within size → 10; result too big → 15, **only when a result is formed**. Failure (as REPLACE): lists of different lengths, empty list, `""` marker → `sno_fail()`, buf unchanged (Jeremy could make `""` marker error 4 instead — not the book's call, it's a C addition). Implemented by Jeremy (one pass, calloc temp of `size`, `goto error`); all tests pass.
- **`sno_view_t sno_chr_replace(char* buf, size_t size, char* from, char* to)`** — `3.4.2 SNOBOL REPLACE(X,Y,Z)`, character translation in place; from/to are C strings. Errors: buf/from/to NULL or no `'\0'` within size → 10; no error 15 (size never changes). Failure per the Green Book (not errors): from/to different lengths or either `""`. Rightmost correspondence for repeats in from (FEET→FOOT — Green Book 3.4.2 p.81, verbatim; likely the 360 `TR` table heritage); a swap ("AB"→"BA") is one pass; `'\0'` can't be in from/to, so the terminator is never translated. Implemented by Jeremy with `strrchr` (passes); a 256-byte table + `XLAT` (`LODSB/XLAT/STOSB` in `__asm`, with `cld`) version also verified in Watcom/DOSBox. Claude suggested `const char*` for from/to (Jeremy's call).

## Meals, breaks and late nights — code of conduct rule 2 (v3, 2026-10-28)
- Message timestamps are visible (Europe/London). Meals around 07:30, 12:30 and 18:30; bed before midnight. **Encourage a break every 2–3 hours** — once, briefly, at a natural pause (after a test passes, a spec is settled), not mid-thought. (Jeremy's request of 2026-10-24 said 1.5–2 hours; v3 follows synergy's 2–3.)
- **Discourage going beyond midnight**: from about 23:00 mention it once; after midnight, gently suggest stopping at the next natural point and say where to pick up. He's an adult — suggest, don't nag or refuse to help.

## Roles
- **Jeremy writes the code.** It is his code; the goal is the best of his code.
- **Claude helps with:** specs (doc comments, contracts, edge cases), test harnesses, iterative review of each version Jeremy brings back, and **keeping the historiography**.
- Claude writes test code and spec text freely. Claude does **not** write implementations (unless Jeremy explicitly asks — e.g. "show me how you would solve this", "pls rewriting"). When asked, prefer the **minimal version in Jeremy's own shape** (riff on his code, keep his idioms such as `goto error`) over a restructure with extra helpers.
- Use the signatures in Jeremy's current headers exactly; don't invent or reorder parameters. When proposing a signature, say it's a proposal.
- When restructuring Jeremy's text, keep his wording and tags; if something has to move or change, say so — never silently.
- **Final documentation pass is Claude's job**, once the coding is done: convert the headers to nested Doxygen groups (below) and check they build without warnings.
- **Writing feedback**: compare his prose against `press/sno-readme-reference.md` using the procedure in `press/jeremy-writing-style.md`.
- **Tooling (2026-10-24/25):** Claude can install Open Watcom v2 (GitHub release snapshot) and DOSBox in the session and build/run `.COM` tests headlessly — use it to verify tests and asm on the real target, not just gcc. (Watcom rejects some things gcc accepts: non-constant automatic struct initialisers in older code paths, `{ ptr, ptr + 1 }` from a `const char* const` — E1054.)

## No spoilers
- Answer the question asked. "Does memcmp scan everything?" gets "no, it stops at the first difference", not a finished function.
- If Jeremy asks for a **hint**: one sentence pointing the right way, no code.
- Review = point at the problem and why; let Jeremy fix it.

## Style (C)
- C99, targeting Open Watcom / 8086 (DOS .COM, tiny model) as well as modern compilers. Avoid compound literals in tests (Watcom).
- Empty parameter lists as `()`, not `(void)`.
- Opening brace on the same line as the function.
- `//` comments are fine.
- **Explanatory names** (Jeremy, 2026-10-25), even if long; implementations compressed but not obfuscated, minimal LOC.
- **Sentinels over counts** for lists (Jeremy, 2026-10-25: "i like sentinels its how i was taught") — lists of C strings end in NULL (the only list type since 2026-10-27; the view list ending in a failure view `{NULL, NULL}` survives only in `sno_bind_list`, on death row). A writer is told its capacity (`size`, in elements); readers find the end themselves.
- Tests use plain `assert.h` — no custom CHECK macros. A test file ends by printing `Pass`.
- **Precondition (error) cases go in `test_sno_errors()` / `test_sno_errors_extra()`** (2026-10-25, replacing the commented-out halt tests): `sno_err_limit(N)`, then one `assert(F_(call) && sno_err_type() == E);` per case (error number aligned, grouped by Green Book section, "… beats failure" lines kept), then `assert(sno_err_count() == 0)` — proves all N raised. Non-view functions assert their own failure value (`== NULL`, `!…`). The real halt / ERRORLEVEL path keeps a few lines in Jeremy's DOS batch file. `test_fail_first()` / `test_fail_first_extra()` pin the check order (failure values with valid arguments just fail).
- When sending test changes, send just the changed test function, not the whole header.
- Lean, in the spirit of K&R first edition (Jeremy has it).

## Function comment template (sno_core.h, sno_extra.h)
Narrative style, not Java-style `@param`/`@retval`. Sections in this order, omitted when empty:

```
/**
 * <section> SNOBOL <NAME(args)> | <section> C <NAME>
 * <One line: what it does.>
 *
 * @pre     <condition>, else error N MESSAGE
 *
 * Success: <returns> <side effects>        (predicates: Returns true:)
 *  + <each case>
 * Failure: <returns> <state left unchanged> (predicates: Returns false:)
 *  + <each case>
 *
 * @warning <traps for the caller>
 * @note    <consequences>
 * @todo    <open decision, for TBC placeholders>
 * @see     <related functions>
 */
```

- **More than one `@pre`/`@note`/`@warning`** (Jeremy, 2026-10-24): one tag on its own line followed by `+` bullets, no blank line in between (verified: Doxygen 1.9.8 renders one bulleted block):
  ```
   * @note
   *  + first
   *  + second
  ```
- **One block for two functions** (e.g. `sno_in`/`sno_out`): a Doxygen member group — `@name <title>` … `@{` on its own line … declarations … `/** @} */`.
- First line marks which world the function comes from (Jeremy specifies both):
  - `2.8 SNOBOL LEN(length)` — Green Book section, `SNOBOL`, and the call as the book writes it (`ANY(string)`, `IDENT(X,Y)`). `REM` has no args (it's a variable holding a pattern).
  - `1.1 C UNBIND` — section it lives in, `C`, name: a C addition with no Green Book equivalent.
  - Summary goes on the second line, not the heading line.
- Not-yet-implemented functions: comment block with `TBC` in the summary, `@todo` explaining why, declaration commented out. Deliberately *not provided* functions (POS): comment block saying so and what to use instead, no declaration.
- Success/Failure mirror SNOBOL's `:S` / `:F` branches.
- Use `@warning`, never `@warn` (not a Doxygen command).
- While coding: chapter and section titles as plain `//` comments.
- Final pass (nested, tested with Doxygen 1.9.8):
  ```
  /** @defgroup ch1 Chapter 1 - Introduction to the SNOBOL Programming Language */

  /** @defgroup s1_1 1.1 Assignment Statements
   *  @ingroup ch1
   *  @{ */
  ...functions...
  /** @} */
  ```
  Group labels follow Green Book numbering (`ch1`, `s1_1`, `s1_3_1`). The file comment needs `@file`. Also fix: `@license` is not a Doxygen command; backslash escapes in comment text (`\r \f \t \v`) warn and `\n` becomes a line break — escape them (`\\n`) or put them in `@c`/backticks. `@name` (if ever used) needs its `@{` on its own line. `S_`/`F_` stay at the top of `sno_core.h` in their own group.
- **Placement rule:** a C addition lives in the Green Book chapter it belongs to semantically (e.g. `sno_unbind` with `sno_bind` under 1.1 — now in `sno_extra.h`, same section number), headed `C <NAME>`. Only helpers with no semantic home go in **Chapter 0 - C Support**: `0.1 Character Sets` (`sno_char_in_set`, `sno_char_in_cstr`).

## Communication
- To the point. Can be interesting, but no padding.
- Don't explain things Jeremy already knows (40 years of C and 8086 assembly). Don't re-litigate settled ground ("this is old ground i want to press on").
- No sycophancy; some humour welcome. Asked for marks, give an honest score with reasons.
- **Sucking eggs** (code of conduct 4; synergy, 2026-10-27 20:14). Keep the small explanations by default — a line of "why" costs little. When Jeremy says "sucking eggs" (or "eggs") for a topic, drop that level of explanation for that topic from then on and add it to the list below, so the next session doesn't do it again. Feedback, not a telling-off: no apology, just adjust. Origin here: 2026-10-23, after `chmod +x` was explained — "no carry on i'll feedback if im sucking eggs ;)".
- **Eggs** — topics Jeremy knows; no explanation at that level:
  - **git** (said in synergy, 2026-10-27 20:14) — status, remotes, commits; just report what happened.

## Return types: views versus bool (global rule, 2026-10-24)
- **If it takes `sno_view_t* u` (a subject), it returns a view.** Pattern functions return the matched view; cursor tests (RPOS) return a null string **at the cursor** (`[cursor, cursor)`, not `sno_null()`) for true and `sno_fail()` for false — SNOBOL's own model (Green Book ch. 3: predicates "return the null string if the condition is met and fail otherwise"). Keeps everything chainable, and every result points into the subject.
- **Output returns a view too** (2026-10-25): `sno_output`/`sno_out` return `v` (as `OUTPUT = X` has the value X), `sno_fail()` for a failure view in or a failed write.
- **Predicates that don't take a subject return `bool`:** `sno_ident`, `sno_differ`, `sno_failed`, `sno_succeeded`, `sno_is_null`, `sno_char_in_set`, `sno_char_in_cstr`. They read naturally in C conditions and don't move a cursor.
- **Non-predicate queries return a plain value**: `sno_size` (int).
- **A list returns an index** (2026-10-25 22:31, the column split): "a literal says what (a view); a list says which (an index)". `sno_find_lit` returns the matching entry's position, −1 for none.
- **Builders return their destination**, as `strcpy` does: `sno_unbind` → `char*`, `sno_bind_list` → `sno_view_t*`.
- In SNOBOL terms for view-returning tests: true = null string, false = failure view.

## Errors: failure versus Green Book error (10.2.2/10.2.3)
- **Failure** = the text doesn't match. Return `sno_fail()` (never `sno_null()`); failure views propagate.
- **Precondition** = programmer error = `sno_error(SNO_ERR_...)` (in `sno_error.h/.c`, v0.4). Replaced `assert` (survives NDEBUG).
- **&ERRLIMIT and &ERRTYPE implemented (2026-10-25; story in `welfare/SMALL-C-historiography.md`):**
  - Default (limit 0, as SNOBOL): print the Green Book message (+ file, line, `__func__`) and exit with ERRORLEVEL = error number.
  - `sno_err_limit(n)` = `&ERRLIMIT = n` (also clears the type); `sno_err_count()` = errors still allowed; `sno_err_type()` = last error number, read-only (SNOBOL's protected keyword), sticky. State is `static` in `sno_error.c` (Jeremy: opaque).
  - Under the limit a conditionally fatal error (1–16) uses one up, is recorded, and **"failure occurs in the expression with the error"** — the function must return its failure value at once. Unconditionally fatal (17–28) always halt.
  - **`sno_error(e)` is a macro → `sno_error_at(e, __FILE__, __LINE__, __func__)`, which returns `sno_view_t` (always `sno_fail()`)** (Jeremy, 14:24), so preconditions in view-returning functions are one line: `if(!u) return sno_error(SNO_ERR_ILLEGAL_ARGUMENT);`. `sno_error.h` includes `sno_types.h`.
  - **The odd ones out** — non-view functions keep two statements, `{ sno_error(…); return <own failure value>; }`:
    - `sno_char_in_cstr` (`bool`) → `false`
    - `sno_unbind` (`char*`) → `NULL`
    - `sno_bind_list` (`sno_view_t*`) → `NULL` (added 2026-10-25 16:30 — Jeremy: "it's a startup builder unlike bind")
    - `sno_find_lit` (`int`) → `-1`; index-returning functions are tested with `>= 0` / `< 0` or a `switch`, not `S_`/`F_`
- **📖 SNOBOL semantics learning — "must" vs "fails if" (2026-10-24):** read the Green Book's wording. Where it says an argument **must** be something, violating it is an **error**: "The arguments of ANY and NOTANY must be nonnull strings when pattern matching is performed" (2.10; same for SPAN/BREAK 2.9) → error 4. Where it says a function **fails if**, it's **failure**: "REPLACE normally succeeds, but it fails if (1) the second and third arguments have different lengths, or (2) the second or third argument is null" (3.4.2); 10.2.2 lists ARG, CODE, COLLECT, CONVERT, DUPL, EVAL, FIELD, LOCAL, REPLACE, STOPTR as primitives that "fail in certain circumstances" (DUPL fails on a negative count, whereas LEN errors). Semantic reason: a null set makes the pattern itself meaningless (broken program); REPLACE with nothing to translate is a function call that produced no value (normal `:F` flow). For C additions the book doesn't decide — choose by analogy and say which.
- `sno_error.h`: `sno_error_t` enum `SNO_ERR_NONE = 0` … `SNO_ERR_COMPILATION_STATEMENT = 28`; `SNO_ERROR_MESSAGES[]` indexed by number; `SNO_ERROR_COUNT`, `SNO_ERROR_LAST_CONDITIONAL` (16).
- Mapping:
  - Null set to ANY / NOTANY / SPAN / BREAK → 4 `SNO_ERR_NULL_STRING`.
  - Negative arg to LEN / RPOS / TAB / RTAB → 14 `SNO_ERR_NEGATIVE_NUMBER`. Keep these params `int` so it can fire.
  - RTAB that would move the cursor left → **failure** (2.11), not an error.
  - REPLACE with different lengths or a null argument → **failure** (3.4.2), not an error.
  - NULL FILE* to sno_input / sno_output → 12 `SNO_ERR_ILLEGAL_IO_UNIT`.
  - Read error (fgetc EOF with ferror set) → 11 `SNO_ERR_READING`.
  - C-only bad args (NULL buf, size < 2 for input, NULL subject pointer, NULL cstr, NULL lists/strings, no `'\0'` within size, `sno_bind_list` size too small for every view + the sentinel) → 10 `SNO_ERR_ILLEGAL_ARGUMENT`.
  - Result too big for buf in `sno_str_replace` (and heap exhaustion of its temp) → 15 `SNO_ERR_STRING_OVERFLOW`. `sno_unbind` still truncates instead (open inconsistency).
  - `sno_bind(NULL)` stays failure: bridge from C APIs that return NULL.
- **Check order (Jeremy's decision, 2026-10-24): argument errors first, then failure.** On entry: (1) NULL subject pointer / NULL buf → error 10; (2) argument preconditions — `n < 0` → 14, null set → 4; (3) only then failure views → `sno_fail()`. Rationale: a bad argument is a bug in the program whether or not an earlier match failed; checking failure first would hide it until the one run where the match succeeds (Jeremy's DbC view: preconditions on entry). A failure view passed as a *value* (e.g. a failure set to ANY) is data, not a bad argument: it just fails. Pinned by `test_sno_errors` ("… beats failure" lines — `sno_err_type()` proves the error came first) and `test_fail_first`.
  - **Exception: error 15 only when a result is formed** (Green Book: "a string … is formed"); a failure forms nothing, so it can't overflow.

## Settled SNO decisions (see headers for detail)
- **View stays two pointers** `{begin, end}` = half-open `[begin, end)`; begin is the cursor when matching. Chosen for parsing (2026-10-22): P86 and C source are free-format (BREAK/SPAN/literals, not columns), so the three-pointer view's main gain (column-stable TAB/POS, a punched-card feature) rarely applies; two pointers make every capture directly a subject, save/restore is `save = u`, 4 bytes (likely DX:AX return on Watcom). Rejected: `origin` field, `{begin, cursor, end}`.
- **A view is bounded by `end`, never by a `'\0'`** — loops run `p < u->end`, not `while (*p)` (BREAK bug, 2026-10-25: it found the NUL test failing and would have read past a slice's end). Test with a slice of a longer buffer.
- **Core takes views; lists are C strings** (redefined 2026-10-27, replacing "core takes views" of 2026-10-25). One value — subject, literal, set — is a view: `sno_lit(&u, sno_view_t lit)` (enables back-references), ANY/NOTANY/BREAK sets, IDENT/DIFFER. **A list is a NULL-terminated list of C strings, `const char**`, everywhere** — core and extra alike: `int sno_find_lit(sno_view_t* u, const char** list)` (was `sno_any_lit`), `sno_str_replace`, and the pinned `next_` pair when they return. Jeremy (2026-10-27): "its const char** all the way down to reduce cognitive load on the programmer ie me :)". Each entry is bound inside the function, so no view list is ever built. `sno_bind_list` is no longer needed by anything — **on death row, review at core/extra v1.0** (Jeremy 11:41). No static view tables from literal macros (two literals ≠ one object). Detail in `sno-backtracking-lessons.md`.
- **Positions count from the view passed in** (its begin = the cursor). No origin arguments, no second `line` view in calls (hence `sno_at` on the back burner).
  - `sno_tab(sno_view_t* u, int n)`: matches `[begin, begin + n)`, leaving u as "column n to end". Fails if n > size. `@warning`: columns of *this* view — on a fresh view (just read/bound) it is column n of the line; after other matches it equals LEN(n). For true columns, TAB from a fresh copy: `u = line; sno_tab(&u, 29);`. Cannot fail for "cursor already past n" (no position left of begin). Test pins the chained behaviour (TAB(3) then TAB(29) → column 32).
  - **POS is not provided** (settled 2026-10-24): the cursor is always at column 0 of its own view, so view-relative POS(0) would always succeed and POS(n>0) always fail — a ported `POS(10)` would fail silently. Header keeps a "Not provided" comment block (currently points to `sno_at`; adjust while `sno_at` is parked).
  - RPOS stays a primitive (measures from end).
- View never owns text, never includes the terminating `'\0'`, may contain NUL bytes.
- Null string: `sno_is_null(v)` = `begin != NULL && begin == end` (many exist; never test by pointer identity).
- Failure: `sno_fail()` = `{NULL, NULL}`; `sno_failed(v)` tests `begin == NULL`; `sno_succeeded(v)` tests `begin != NULL`; `S_`/`F_` macros.
- `sno_ident` compares content (SNOBOL interns strings so compares pointers; views are not interned). IDENT/DIFFER both false on failure.
- `sno_input(f, buf, size)`: one call = one line; long lines truncated and the rest discarded; final line without `'\n'` returned normally; failure at EOF only.
- `sno_unbind(v, buf, size)`: NULL return (nothing written) for `size == 0` or a failure view.
- `sno_output(f, v)`: FILE first; returns the view. `sno_out(v)` = `sno_output(stdout, v)`, **no silent `\n`** — use `SNO_NEWLINE`.
- **Pattern functions** take `sno_view_t* u` (subject; begin is the cursor) and return the **matched view** `[old cursor, new cursor)` or `sno_fail()`. Failure leaves the subject unchanged. Applies to ANY, NOTANY, SPAN, BREAK, REM, LEN, TAB, RTAB, RPOS, BAL, the lit family, …
- NOTANY fails at end of subject. BREAK must find a break char (end of subject = failure); at a break char it succeeds with a null string; it doesn't consume the break char. SPAN must match ≥ 1 char.
- `sno_len(sno_view_t* u, int n)`: n == 0 returns a null string at the cursor; fewer than n chars left = failure; n < 0 → error 14 (even on a failure subject).
- `sno_rtab(sno_view_t* u, int n)`: matches to `end - n`; n > remaining = failure (would move cursor left); n < 0 → error 14. Correct anywhere in a chain (measures from end). `sno_rem(u)` = `sno_rtab(u, 0)` (static inline), never fails on a real subject.
- `sno_rpos(sno_view_t* u, int n)` (2.12): tests that **exactly** n chars remain; never moves the cursor; success = null string at the cursor, failure = `sno_fail()` (more or fewer than n remain, or failure subject); n < 0 → error 14. RPOS(0) = right anchor ("whole subject matched"). After RTAB(n) succeeds, RPOS(n) always succeeds.
- **Functional chaining via views only** — no patterns as values, no implicit backtracking (SNO = a FENCE between every element; see backtracking lessons). ARB is not implemented: its uses map to BREAK, `sno_next_lit`/`sno_next_any_lit`, or a caller-written loop.
- Whitespace = POSIX "C" locale set; **`SNO_POSIX_WHITESPACE` becomes an array** `const char[] = " \f\n\r\t\v"` so constant views (`SNO_SPACE`, `SNO_NEWLINE`, `SNO_TAB`, `SNO_WHITESPACE`) can slice it (Watcom needs address constants). Set-membership tests, not `isspace()`.
- Every repeated match must consume.

## Status (sno_core.h v0.4, sno_extra.h v0.1, sno_error.h v0.4)
- **Departments:** WELFARE is the main department (synergy, 2026-10-27 18:26: "welfare *is* the main"); GOAT, the old main SNO chat, retired 2026-10-28 19:22 ("put out to pasture"). SNO, PCODE and PRESS as before.
- **Retrofitted from synergy** 2026-10-27 18:14 (`handover/2026-10-27-synergy-to-small-c.md`); brought up to date 2026-10-28 (`handover/2026-10-28-synergy-to-small-c.md`): documents home in the oracle (`3fc837c`), code of conduct v3 signed 20:03 (`322f12e`).
- v0.3: preconditions, ANY/NOTANY, sno_is_null, template comments.
- v0.4: sno_error replaces assert (Green Book error numbers, ERRORLEVEL exit); ANY/NOTANY return matched views; LEN.
- Since v0.4: RTAB, REM, TAB, RPOS; POS not provided; sno_at parked; errors-first; header split (`sno_extra.h`); `sno_str_replace`, `sno_chr_replace` implemented and passing; **2026-10-25:** `sno_succeeded` + `S_`/`F_`; `sno_output` returns a view, `sno_out`; BREAK implemented and passing (after the `while(*q)` fix); &ERRLIMIT/&ERRTYPE (`sno_error` v0.4, `return sno_error(...)`); `test_sno_errors()` (27 lines incl. bind_list) and `test_sno_errors_extra()` (21); `test_fail_first_extra()` updated; tests split into two programs; `sno_bind_list` spec + test written; historiography written for the whole project. README approved (reference: `press/sno-readme-reference.md`) — **its colophon still says "assembled with A86"; Open Watcom WASM/WLINK replaced A86 on 2026-10-23** (found by the historiography).
- Open in Jeremy's code: `test_output()` has `//assert(sno_failed(sno_output(f, v)));` commented out — Watcom's `fwrite` on an `"r"` stream returns 0 with `ferror` set (verified in DOSBox), so if it fails, `sno_output` isn't comparing `fwrite`'s count with the size.
- sno_extra.h loose ends (flagged 2026-10-24): typo `snoe_core.h` in file comment; Functions list stale (UNBIND "moving", 1.5/3.4.2 entries need the new names, REPLACE "planned"); add `#include "sno_core.h"`; blank line before the REPLACE comment; INPUT's text-mode `'\r'` note dropped (Jeremy to decide); `@see` cross-references need the new names.
- **Blog post (2026-10-24)**: "C SNOBOL do SNOBOL" on libfbp.blogspot.com, from the README. Header image = a real two-card SNOBOL4 deck in IBM 029 code (`OUTPUT = 'C SNOBOL DO SNOBOL'` SNO00010 / `END` SNO00020), SteelBlue Helvetica-style C; image in the SNO repo (`raw.githubusercontent.com/cspjst/SNO/129b649…/c-snobol-do-snobol.png`); † footnote explains the deck. Loose ends: replace `[link to repo]` placeholders; set the † href to the published post URL + `#fn-deck`. More material: "must" vs "fails if"; "Jeremy in SNOBOL Land"; FENCE everywhere; &ERRLIMIT story (see companion docs).
Done: bind/unbind, null/is_null, fail/failed, succeeded, size, trim, input/output/in/out, char_in_set/cstr, ident/differ, any/notany, len, tab, rtab, rem, rpos, break, str_replace, chr_replace, sno_error (+ &ERRLIMIT/&ERRTYPE).

## Order of work
~~Convert every precondition to `return sno_error(...)`~~ — done throughout (Jeremy, 2026-10-28 20:30, via SNO) → lit family: `sno_find_lit` implemented, not yet passing — Jeremy is on it (20:30); built as a tower (current names, signatures and what's pinned: `sno-backtracking-lessons.md`, which wins over this file) → **SPAN** (then the 2.9 colon-compression program as a joint BREAK/SPAN test) → constant views → **BAL (pinned)** → DUPL, concatenation (extra) → final Doxygen pass (Claude). `sno_at` only if a need for `line` reappears. Historiography: extend at each milestone.

## Green Book
- Jeremy has the 2nd edition PDF and now a physical copy (James Bando's, felt-tip name on the inner leaf); check each new function against its exact wording and the error list in 10.2.3. **Read for "must" (→ error) vs "fails if" (→ failure)** when deciding a function's contract. The PDF is in the project as `snobol-green-book.pdf` (since 2026-10-27 17:30): `project_read` gives the extracted text — Grep it for the exact wording. Text only, no page images: where the OCR garbles (the keypunch characters, pp. 81–82; the colon-compression program, p. 39) ask Jeremy to attach the page. (Until 27 October this said "Claude has the scan in-session" — that meant the old Claude Code session.)
