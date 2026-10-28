# SNO — lessons from SNOBOL's backtracking primitives (2026-10-25)

Working through Green Book 2.13 FAIL, 2.14 FENCE, 2.15 ABORT, 2.20 SUCCEED: how each example would be written in SNO, which has no implicit backtracking. None adds a function; together they explain SNO's design. (Status: FENCE done; FAIL MISSISSIPPI done; ABORT done; FAIL A/B/C words and SUCCEED still to go through.)

## Pins and death row (2026-10-27) — read this first
- **`sno_any_lit` renamed `sno_find_lit`** (Jeremy 11:11: "more semantically clear… fits with its return type"). Signature `int sno_find_lit(sno_view_t* u, const char** list)` — the list is now a NULL-terminated list of C strings (as `sno_str_replace`), bound inside; `sno_bind_list` no longer needed to call it (11:24).
- **PINNED: `sno_next_lit` and `sno_next_any_lit`** (11:0x: "not sure about both of the unanchored next versions lets put a pin in them both"). Their specs below stand as drafts; `sno_next_lit` spec + test were written (1.4 C NEXT LIT, strtok token, null literal → error 4). Not in `sno_core.h` until unpinned. Stale references to them remain in LIT's `@warning`/`@see` and BREAK's `@see`.
- **ON DEATH ROW: `sno_bind_list`** (Jeremy 11:41): "its death sentence has not been commuted merely that it waits on death row for a review of its sentence once we hit versions 1 of core and extra." It stays in `sno_core.h` (1.1 C BIND LIST) and its tests stay, but nothing in the lit family depends on it. **Review at sno_core v1.0 / sno_extra v1.0.** Its `@note` still names `sno_any_lit()`/`sno_next_any_lit()` — fix or leave for the review.
- **"Don't over tool too early"** (Jeremy, Oct 27): core principles and core test cases; edge cases ad hoc.
- The lit section is now `sno_lit` (view: what) and `sno_find_lit` (index: which).

## Big idea
- **SNO has an implicit FENCE between every element.** Every SNO function commits on success; nothing can be backed into. That is the PEG property (ordered choice commits — Ford 2004). FENCE is how SNOBOL programmers got PEG behaviour by hand.
- In C: alternation = `||` (no save/restore needed — a failing SNO function leaves the subject unchanged, so each alternative starts at the same cursor); concatenation = `&&`; once `&&` moves on, C can't go back into the `||` → the FENCE.
- Difference: on statement failure SNOBOL restores the subject; in SNO the subject stays advanced past what matched — the caller does `save = u` if it wants it back.
- **Data in the function, control flow in C.** Literal lists are data and can go in a function; alternation of arbitrary patterns, generators and statement-level failure are C control flow.
- **The scanner's control primitives, mapped:** FAIL turns both hidden loops (alternation retry, unanchored advance); FENCE blocks backing up; ABORT stops both loops at once. SNO: loops are C loops, FENCE is everywhere, ABORT is `continue`/`return`.

## 2.14 FENCE (p.49)
- `'BERATES' ('BE' | 'GE' | 'FRE') FENCE ('TS' | 'T')` — SNO gets it **for free**: BE matches, TS and T fail, statement fails; GE/FRE never retried.
- **Catch — overlapping alternatives.** Free only because the alternatives can't compete (different first letters; the book: "it is obvious that GE and FRE should not be tried"). When one alternative is a prefix of another, the implicit FENCE changes the *result*: `'BEATS' ('B' | 'BE') 'ATS'` succeeds in SNOBOL (backs up to BE) but fails in SNO.
- **Porting rule:** alternatives that start differently port unchanged; prefix-overlapping alternatives must be reordered **longest first** (`'BE' | 'B'`). Same rule as `sno_str_replace`'s "list `$10` before `$1`".
- FENCE's anchoring use (`STR (FENCE PAT)`) is unnecessary: SNO functions are always anchored at the cursor.

## 2.13 FAIL (p.47) — MISSISSIPPI
- `&ANCHOR = 0` / `'MISSISSIPPI' ('IS' | 'SI' | 'IP' | 'PI') $ OUTPUT FAIL` prints IS SI IS SI IP PI.
- **The statement hides two loops and FAIL drives both:** inner = alternation retry (FAIL backs into `|`); outer = the unanchored scanner moving the start position on (the implicit ARB in front of every unanchored pattern). In SNO both are plain C loops, so FAIL has nothing to do. `sno_fail()` already *is* FAIL's value.
- Faithful SNO version (verified, prints the book's output): `do { for each pat[i]: t = u; if (S_(m = sno_lit(&t, pat[i]))) output m; } while (S_(sno_len(&u, 1)));` — LEN(1) failing at the end terminates the scan (SNO failure as loop control); also tries once at the end of the subject, as SNOBOL does.
- A generator ≠ alternation: `sno_find_lit` stops at the first match (FENCE); FAIL wants every match. With `('I' | 'IS')` SNOBOL prints both from one position — only the hand-written inner loop does that.
- **No `sno_generate`** (callback = inverted control, state through `void*`); the C loop *is* the generator. What's worth a function is the step — the unanchored search.
- Possible: head `sno_fail()`'s comment `2.13 SNOBOL FAIL` (currently `1.8 C FAIL`) — Jeremy's call.

## 2.13 FAIL — A/B/C words (p.48) — first look
- Uses FAIL to force all alternatives for `$` side effects (SNOBOL alternation short-circuits like `||`) → logical AND. In C: three independent searches on copies. If it ever became a function, `sno_all_lit` ("all of these occur") is its natural name — keep that name free.

## 2.15 ABORT (p.49)
- **CARDFORM = '*' ABORT | CARD** (skip comment cards) — Jeremy's favourite; directly useful for P86 templates (`; build the stack frame` → `";"`). Works with today's functions:
  ```c
  while (S_(card = sno_input(stdin, buf, sizeof buf))) {
      t = card;
      if (S_(sno_any(&t, sno_bind("*")))) continue;   // '*' ABORT: comment card
      u = card;
      // ... match CARD on u                          // | CARD
  }
  ```
  (For one character, `ANY('*')` = the literal `'*'`.)
- **Lessons:**
  1. **ABORT is non-consuming lookahead.** `'*'` matches, then the whole match is discarded — CARD would have started at column 1. SNO: test on the copy `t`, `card` untouched (the two-pointer view's `save = u` payoff). = PEG not-predicate `!'*' CARD`. Not `NOTANY('*') CARD` — NOTANY consumes, CARD would start at column 2.
  2. **ABORT stops both hidden loops** (alternation and the unanchored advance — even with `&ANCHOR = 0` the scanner doesn't move on to column 2). SNO: `continue` — statement-level control, no function needed.
  3. **SNO separates "skip" from "no match"; SNOBOL can't.** A comment card and a card CARD rejects both just fail the statement and take the same `:F`. In SNO they're two paths: `continue` for the comment, an error report for a malformed card. Matters for the P86 parser (skip comments silently, report bad lines).
  4. **SHORTPAT = LEN(12) ABORT | PAT** — SNOBOL must express the predicate "≥ 12 characters" as a pattern; SNO asks directly: `if (sno_size(u) >= 12) fail; else PAT`. (Lookahead `t = u; sno_len(&t, 12)` also works; `sno_size` says what's meant.)
  5. **PNOTQ = Q ABORT | P** ("P but not Q") → test Q on a copy; if it succeeds stop, else match P. Three lines of C; a function would need Q as a value — no.
- Two levels of failure: function failure (`sno_fail()`, try next alternative) vs statement failure (caller's `return`/`continue`). ABORT is the latter.

## 2.20 SUCCEED (p.61) — first look
- Always matches null, even on backtrack (book: "practical uses … seem limited"; SAWTOOTH never terminates). In SNO it's `sno_len(&u, 0)`; with no backing up it can't loop.

## Decisions — the literal family (2026-10-25; final signatures 22:31 — the column split, option B; **amended 2026-10-27, see top**)

| | one literal → **view** | any of a list (first in list wins) → **index** |
|---|---|---|
| **anchored** — match at the cursor | `sno_view_t sno_lit(&u, lit)` ✅ | `int sno_find_lit(&u, list)` (was `sno_any_lit`) |
| **unanchored** — skip ahead to the next occurrence | 📌 `sno_view_t sno_next_lit(&u, lit)` | 📌 `int sno_next_any_lit(&u, list)` |

📌 = pinned 2026-10-27.

**Rule: "a literal says *what* (a view); a list says *which* (an index)."** Decided by Jeremy at 22:31 after comparing dispatch options side by side on a one-line p-code example (A: row split — search then check position; B: column split — one anchored call; C: `int*` out-parameter). B makes anchored dispatch one call: `switch (sno_find_lit(&u, MNEMONICS)) { case PUSHI: … default: /* -1 */ }` — enum in the same order as the list (test_find_lit does exactly this).

- **`sno_view_t sno_lit(sno_view_t* u, sno_view_t lit)`** — done. LEN on a copy + IDENT, commit the copy. Only error: NULL u → 10. Failure view as `lit` just fails; `sno_null()` matches the null string at the cursor; back-references work.
- **`int sno_find_lit(sno_view_t* u, const char** list)`** — anchored: the index of the first entry that matches at the cursor; cursor moves past it; −1 if none (subject unchanged). Loop over `sno_lit(u, sno_bind(list[i]))`. Errors: NULL u → 10, NULL list → 10 (beats failure). `@warning`: longest first (PUSHI before PUSH); a whole-token check after (`PUSHX` matches PUSH) is the caller's. Spec + test (with a `switch`) delivered 2026-10-27.
- 📌 **`sno_view_t sno_next_lit(sno_view_t* u, sno_view_t lit)`** — unanchored, strtok semantics (Jeremy 21:55: "i like the equivalence with strtok and sno_next_lit"): returns the **token** — the text skipped before the next occurrence `[old cursor, start)`; cursor moves past the literal; `sno_fail()` if none (subject unchanged). The literal found is `[tok.end, u.begin)`.
- 📌 **`int sno_next_any_lit(sno_view_t* u, list)`** — unanchored: skip ahead to the next occurrence of any listed literal; returns its **index**; cursor moves past it; −1 if none. Earliest position wins; at one position, first in the list wins. Token = `[saved.begin, u.begin − size(list[i]))`. (List type would follow `sno_find_lit`: C strings.)
- **Odd ones out (errors):** the index-returning functions use `{ sno_error(…); return -1; }`; the view-returning ones keep `return sno_error(…)`. Index-returning functions are tested with `>= 0` / `< 0` (or a `switch`), not `S_`/`F_`.
- **Errors for the `next_` pair (proposed, pinned):** a null-string literal → error 4 (would match at the cursor forever — "every repeated match must consume").
- **strtok equivalence** (Jeremy's canonical example, `"Hello, World! Welcome to C programming."` with delimiters `" ,.!"` → Hello / World / Welcome / to / C / programming): `while ((d = sno_next_any_lit(&u, delims)) >= 0)`, token from the saved cursor, skip null tokens, last token via `sno_rem` when −1 and the subject isn't empty. Better than strtok: re-entrant, non-destructive, delimiters can be whole strings, and `d` says which delimiter ended each token. (Pinned with its function.)
- **Rejected (21:34–22:31):** `int* index` out-parameter ("dump optional parameter it smells"); `sno_index_of(list, v)` lookup ("nope dont like it"); the row split (anchored → view, unanchored → index; 22:03–22:27). The 22:00–22:18 translator demos are superseded.
- **Built as a tower:** `sno_lit` = LEN + IDENT; `sno_find_lit` = loop over `sno_lit` with a counter; `sno_next_any_lit` = loop: try each literal at the cursor (on a copy), else LEN(1); `sno_next_lit` = the same search for one literal, returning the token. Later accelerator: BREAK to the literal's first character.
- **Dispatch must be anchored:** a search on `pop ax ; was PUSH` finds PUSH in the comment — searching is for templates/tokenising; parsing at the cursor is `sno_lit`/`sno_find_lit`.
- **Trap — no static view tables from literals** (`{ s, s + sizeof(s) - 1 }` = two literals, not one object). Was `sno_bind_list`'s reason to exist; `sno_find_lit` taking C strings sidesteps it.
- **Boundary:** only literals. Alternation of arbitrary patterns stays C `||`.

## Other decisions (2026-10-25)
- **`sno_bind_list(const char** cstrs, sno_view_t* views, size_t size)`** — done by Jeremy; **on death row since 2026-10-27, review at v1.0** (see top). Count first, then write (nothing written on error); size in views (`sizeof ops / sizeof *ops`); error 10 if too small; one of the odd ones out (`return NULL`).
- **`sno_break(sno_view_t* u, sno_view_t v)`** — done (after fixing `while(*q)` → bound by `u->end`). Doesn't consume the break char. Errors: NULL u → 10, null set → 4 (beats failure). SPAN next; then the 2.9 colon-compression program as a joint test.
- **`sno_succeeded(v)`** + **`S_`/`F_`** macros (top of `sno_core.h`; trailing underscore — `_S`/`_F` reserved, C99 7.1.3).
- **`sno_output` returns a view**; **`sno_out(v)`** = `sno_output(stdout, v)`, no silent `\n` (`sno_out(SNO_NEWLINE)`).
- **Constant views** from the whitespace array (`SNO_POSIX_WHITESPACE` must become `const char[]`): `SNO_SPACE`, `SNO_NEWLINE`, `SNO_TAB`, `SNO_WHITESPACE`.
- **&ERRLIMIT / &ERRTYPE** — see `welfare/SMALL-C-historiography.md` (chapters 12–13 + postscript). Tracing: Jeremy kept his own on/off trace in `sno_error_at` (preferred over Claude's &TRACE-style countdown). `test_sno_errors()` has per-section `assert(sno_err_count() == N - k)` checks.
- **Naming preference (Jeremy):** explanatory function names, even if long; implementations compressed but not obfuscated, minimal LOC.

## Blog / README material
- **"Jeremy in SNOBOL Land"** — the Alice in Wonderland frame: "tea!" said the Dormouse; at the Mad Tea-Party it's always tea-time; FAIL, FENCE and ABORT turn out to be C loops in disguise — "curiouser and curiouser".
- "SNO has a FENCE between every element."
- "A SNOBOL statement hides two loops; FAIL turns them; in SNO they're just C."
- "SNOBOL can't tell a comment from a mistake; SNO can" (ABORT and `:F`).
- strtok, re-entrant and non-destructive, from `sno_next_any_lit`.
- "A literal says what; a list says which."
- `sno_bind_list` on death row: "its sentence has not been commuted" — a week-old library already has a legacy.
- The keypunch program (3.4.2, 026 → 029) in SNO as a second README example; James Bando's copy of the Green Book.
