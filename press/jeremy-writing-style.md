# Jeremy's writing style — for comments, specs and docs

## ⭐ Reference document
**`press/sno-readme-reference.md`** (the SNO README, 2026-10-24, approved by Jeremy) is THE reference for his prose style and document layout. Compare all future writing against it — both drafts Claude writes and pieces Jeremy sends for feedback.

### Layout conventions it establishes
- Title `# NAME - one-line description`, then an italic personal tagline (`*I've wanted to do this since 1988!*`), then one plain paragraph saying what it is and crediting the source (full citation).
- Draft/status notice as a blockquote.
- `##` sections in narrative order: Where it started → A little history → themed main section (`## When Worlds Collide!` with a plain `###` subtitle) → Putting it all together → short hedged question section → The epiphany → Status → Colophon → sign-off line → author + blog link → licence.
- Numbered subsections as `#### Clash N - Declarative title` (spaced hyphen, the title states the conclusion).
- Cross-references name the number *and* the topic: "clash 3 (null versus failure)".
- Tables captioned `##### Table N. Caption`.
- Code fenced with a language tag; types named inline in backticks.
- "NB"/"N.B." paragraphs for consequences; principle stated in the first line of a section.

### How to give feedback on his writing
1. Compare against the reference: voice, layout conventions above, and the lessons below.
2. Say what's working (briefly) and what departs from the reference.
3. Flag typos, factual slips (e.g. SNOBOL `INPUT` is a variable, not a function) and broken cross-references, with line numbers.
4. Don't rewrite unless asked; when asked to correct, change only what was flagged.
5. Record anything new learned here.

Sources: his `sno_core.h` comments, blog https://libfbp.blogspot.com/ (esp. "Applying Design by Contract to C", 2025; "Getting Zed and Clangd to Be Useful for the Retro Programmer", 2026; his two most popular narrative posts, "C++ Mellor-Crummey Scott (MCS) lock", 2018, and "Tic-Tac-Toe Bitboards", 2017); his rewrites of Claude's SNO README draft (2026-10-24).

## Code comments (doc blocks and //)
- **Lead with the verb or the noun**, not a sentence: "Bind a null-terminated C string to parsing context", "Unbind the view - copy a view into buf as a C string".
- **Short, plain statements.** No chains of "which…" clauses, no connective padding ("Note that", "In order to", "This function").
- **Spaced hyphen as separator:** "Fails at end of subject - there is no character to match".
- **"ie" without dots** in code comments ("subject is empty ie cursor at end"). (Prose uses "i.e.".)
- **Emphasis with asterisks:** "*first* character", "*never* includes".
- **Say the consequence, not the mechanics:** "// there is no return if the post-condition is not met".
- **Labels for rules:** `// Precondition`, `// INVARIANT: 0 <= size <= capacity` (caps tag, rule stated directly).
- `//` one-liners often have no full stop.
- **Occasional dry aside is welcome**: "// Schrödinger's error: intentional or bug?" — one, not a running joke.
- Dislikes comments that only restate a rule ("NOTE: caller must hold mutex!") — prefers a runtime check plus a comment saying what it guards.

## Prose (blog, file headers)
- **British spelling and idiom**: colour, organise, behaviour, modernised; "Noughts & Crosses", "draughts", "Blighty".
- First person, conversational, enthusiastic about the niche, self-deprecating about it ("which is a lie but a lie we can live with").
- Pattern: **concrete problem → snippet → cause → fix**. Motivation stated plainly: "I want to…".
- Mixed sentence length: long explanatory sentences with clauses, broken by short punchlines. Semicolons in explanations; ellipses for rhythm in taglines.
- Recurring: "retro", "fellow retro programmers", "best of both worlds", "desiderata", "N.B."/"NB", "Okay, so", "Yes of course".

## Lessons from his README rewrites (2026-10-24) — what he changed in Claude's draft

### Section 1 (Where it started)
- **Personal history over practical motive.** Lead with the *personal* timeline ("since 1988", "decades of ad-hoc… ersatz and ultimately unsatisfying snobol-esque functions - the time has come to do the job 'properly'").
- **Honest tagline beats pastiche.** Don't imitate his blog tagline; let him supply the line.
- **Chatty bracketed asides, informal abbreviations**: "(I still do love C btw warts and all!)".
- **Long, comma-led sentences with hedges** are his. Don't tidy them into short punchy sentences.
- **Plain over clever.** "the resolutions *are* the library" → "the resolutions became the library".
- **Platform/build details out of the opening** — colophon.
- **Kept**: "So the naive idea was… job done. It was not job done."

### Section 2 (A little history) — kept unchanged
- "Uncanny reading it." Factual history with a light opinion and the "once you look" observation, credit + link.

### Section 3 (the clashes) — rewritten, most of Claude's paragraphs kept verbatim
- **Themed playful heading** from retro pop culture ("When Worlds Collide!") with a plain subtitle; **carries the metaphor** ("Moving between worlds").
- **Declarative headings that state the conclusion**, not reader interruptions ("Errors are not failures", not "What happens when the programmer gets it wrong?").
- **Numbered, report-like structure**; captioned tables.
- **The rule goes first** in a section.
- **Personal confessions**: "I used to like ARB but I probably over and inappropriately used it."
- **Reasoning signposts**: "Therefore", "Ergo", "However...", "When in doubt fall back to the ideas behind SNOBOL", "Yet, I still managed to…".
- **Suspense pauses**: "- the answer?...", "However..." on its own line.
- **Problem framing before solution** ("an anachronism… a problem… a tricky one at that").
- **Names the type inline**: "the SNO view as `sno_view_t`".
- **Softer on bold claims**: don't oversell against other tools (regex section cut to one hedged line ending in "?").
- **Earnest over arch**: "The amazing discovery" → "The epiphany"; dropped the patent-language joke.

## Narrative shape (from the MCS lock and bitboard posts, adjusted by the rewrites)
- **Open where he started**, in his own history, not a project rationale.
- **History and credit get real space**; place the idea in its lineage before explaining it.
- **Teach by building up in small concrete steps**.
- **Headings**: one playful themed heading for a big section; subsection headings state the conclusion. Question headings belong in tutorial posts, not READMEs.
- **Speaks to the reader** where natural.
- **Diagrams/tables do the explaining**.
- **Ends with a Colophon** and a modest sign-off.
- **Avoid his two weak spots**: don't stop abruptly; **link the citations**.

## His design philosophy (relevant to SNO)
- From the DbC post (2025): `assert()` is "ephemeral" — vanishes in release builds and lacks semantic power. Contract violations are **intentionally fatal** and should be **specific**. DbC is not for recoverable errors.
- SNO's `sno_error()` with Green Book error numbers is the same idea.

## For Claude
- Header comments in his voice: terse, verb-first, hyphen separators, "ie", asterisk emphasis, British spelling, consequence over mechanics.
- Don't add explanatory sentences he wouldn't write.
- README/blog drafts: match the reference document; draft for him to rewrite, never final.
- When he feeds back a rewrite, diff it against the draft and record what changed here; record what he kept unchanged too.
