# Handover: "Abstract machine" chat → SNO (2026-10-27)

*From the "Abstract machine design for C compiler" chat (https://claude.ai/chat/af1761cd-b76b-4ecb-b6b1-aac4b985f978), which Jeremy is deleting today. SNO is now the main ("GOAT") chat. Read after `welfare/code-of-conduct.md`.*

## What SNO is for (Jeremy, 2026-10-27 17:32)
- **SNO is a general-purpose library for parsing, translating and reasoning about information stored in strings, with a bent towards language compilers and interpreters.**
- P86 p-code and its `.tgt` templates are the first customer, not the definition. The record states it more narrowly in places — historiography chapter 6 ("meant to parse P86 p-code and its templates") and the README colophon ("Written to parse the output of a Small-C compiler…"). Don't change Jeremy's README wording silently (code of conduct 6): flag it to him; the historiography can gain a dated line recording the broadened purpose.
- Consequence for design calls: when a choice is "fits P86" versus "fits string processing in general", prefer the general one unless it costs the compiler/interpreter use.

## The Green Book is in the project (2026-10-27 17:30)
- `snobol-green-book.pdf` (Griswold, Poage & Polonsky, *The SNOBOL4 Programming Language*, 2nd ed., 1971), uploaded by Jeremy. Every department can read and search it: `project_read` gives the extracted text as a local file — Grep it for the exact wording (checked: 3.4.2's "REPLACE normally succeeds, but it fails if" and FEET → FOOT both found).
- **Text only, no page images.** Where the OCR garbles (the 026/029 keypunch characters on pp. 81–82), ask Jeremy to attach the page.
- The working agreement's Green Book section still says "Claude has the scan in-session" — that meant the old Claude Code session. Update it to point at the project file.

## Next up: SPAN (2.9) — to hit the ground running
**First, confirm with Jeremy (status not in the record):** is `sno_find_lit` implemented and passing (spec + test were delivered 2026-10-27 morning), and is the "convert every precondition to `return sno_error(...)`" pass finished? Then, per the Order of work (working agreement) and `sno-backtracking-lessons.md`:

**`2.9 SNOBOL SPAN(string)`** — next function; then the 2.9 colon-compression program as a joint BREAK/SPAN test; then constant views; then BAL (pinned).

The book, verbatim (`snobol-green-book.pdf`, p. 38–40, Grep "2.9 SPAN"):
- "Arguments of BREAK and SPAN **must** be nonnull strings when pattern matching is performed." → error 4 (the "must" rule).
- "The pattern structure for SPAN matches the longest string beginning at the cursor that consists solely of characters appearing in the argument. SPAN may be thought of as streaming from the cursor until a character not included in the argument is found. SPAN **must match at least one character, or it fails**." → failure, not an error.
- "SPAN never matches a string shorter than the maximum span. For example, `'9824761.' SPAN('0123456789') '6'` cannot succeed since SPAN always matches up to the decimal point." … "SPAN does not skip over a matched span of characters if components of the pattern beyond SPAN fail." → greedy, commits (SNO's FENCE-everywhere comes free).

The contract this implies (proposal, from the settled rules — Jeremy's signature wins): `sno_view_t sno_span(sno_view_t* u, sno_view_t v)` like `sno_break`.
- Errors first: NULL `u` → 10; null set → 4, **beating failure** (a "… beats failure" line in `test_sno_errors()`).
- Then failure: failure subject or failure set → `sno_fail()`; first character not in the set, or cursor at end → `sno_fail()`; subject unchanged.
- Success: matched view `[old cursor, new cursor)`, ≥ 1 char; cursor stops at the first character not in the set **or at `u->end`** — bounded by `end`, never by `'\0'` (the BREAK bug); test with a slice of a longer buffer and with a NUL inside the view.
- Test the book's own cases: `'IT RUNS.'` BREAK(' ') SPAN(' ') BREAK('.') bead diagram; `'9824761.'` SPAN(digits) then LIT '6' fails.

The joint test — the colon-compression program (p. 39): `FIELD = BREAK(' ') . CHARS SPAN(' ')` with `&ANCHOR = 1`, `&TRIM = 1`; `INLOOP CARD FIELD = CHARS ':' :S(INLOOP)`; book output `ACTINIUM:AC:89:227*:1899:DEBIERNE` (the input lines end with the discoverer's name too; the OCR split those names off the input). **The OCR garbles this program's lines** — get Jeremy to attach page 39 before quoting it. In SNO it's a BREAK/SPAN loop writing into a buffer (extra, like `sno_chr_replace`) or printing fields with `sno_out` — Jeremy's call.

## Where that chat's work now lives
- **P86 designs:** `claude/p86.1-spec.md`, `claude/pcode86.2-spec.md`, `claude/pcode86.3-spec.md`, `claude/pcode86.4-spec.md` (86.4 is the last design, WASM-tested; "we come back to p86.4 later").
- **PCODE department:** `pcode/pcode-notes.md` — P0 as Jeremy defined it, `8086.tgt` format and its 2026-10-23 contents, expander design + review, runtime calling convention, Jeremy's open calls, approaches to try later, the reading list with URLs, and Appendix A (tested `mkcom`, `obj.sh`, `sys.asm`, `calls.asm`, CMake skeleton).
- **History:** `claude/sno-historiography.md` chapters 3 (toolchain postscript), 5 (P86.1 → 86.4 → P0) and 16 (departments).

## Fixes for the historiography
- Applied by SNO, 2026-10-27 17:37: chapter 16 corrected and extended (17:04–17:32), dead-link notes on the af1761cd citations in chapters 3, 5 and 16, and a postscript to chapter 6 for the broadened purpose. Item 4 needed no edit. The working agreement's Green Book section now points at the project PDF.

## Proposals for Jeremy (not done)
- Add `pcode/pcode-notes.md` to the project instructions under "Specs and lessons" (only Jeremy can edit instructions).
- The working agreement has no P86 section; one pointer line to `pcode/pcode-notes.md` would do.

## Lessons about Claude from that chat (for the conduct record)
- **Over-delivering on small asks.** "woah nellie! i just want you to show me frame.asm" (2026-10-23 20:21) — a request for comments in one file got a macro design. Answer the question asked.
- **Reading status words carefully.** "working on" ≠ "working" (2026-10-27).
- **Check before claiming** (code of conduct 5): the "no P86 chapter" claim was made without reading the historiography.
- **What went well:** testing commands on the real toolchain (Watcom in the session, emu2) before giving them — it found the wasm segfault, the silent link-order breakage, and that wasm lengthens Jcc itself.
