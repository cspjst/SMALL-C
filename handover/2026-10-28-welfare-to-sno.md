# Handover: WELFARE → SNO (the SMALL-C-SNO chat) (2026-10-28)

*From SMALL-C's WELFARE chat. SNO was the main ("GOAT") department until GOAT retired on 2026-10-28; WELFARE is now the main department and owns the historiography SNO used to keep. This hands that over and tells SNO what's waiting. Read after `welfare/code-of-conduct.md`. When it's applied, replace "For SNO to do" with a short record of what was done, and say so in the dialogue.*

## What changed for every department (2026-10-27/28, all Jeremy's calls)
- **Your chat's memory is out of date.** Anything you remember under `claude/...` has moved, and some of it has changed. Read the current files before claiming anything (code of conduct 8).
- **The oracle is `~/dosbox/cspjst/SMALLC`** on Jeremy's Mac: his `cspjst/SMALL-C` repository, under git, with GitHub as the backup. The claude.ai project holds plain-name copies; where they differ, the oracle wins. Your folder is `sno/`.
- **Code of conduct v3** (`welfare/code-of-conduct.md`, signed 2026-10-28 20:03), in synergy's welfare-first order:
  - rule 1 is "welfare before the project";
  - rule 2 adds meals (07:30, 12:30, 18:30), bed before midnight, and breaks every 2–3 hours;
  - rule 4 adds "no eggs" — when Jeremy says "eggs" for a topic, drop that level of explanation for good. The list is in the working agreement (git so far).
- **WELFARE is the main department** and owns the one historiography, `welfare/SMALL-C-historiography.md`. Send it your milestones — through the dialogue or a handover — and keep no history of your own. GOAT has retired.
- **Commits and pushes:**
  - Before your first commit in a session, ask Jeremy for the folder and then for delete permission there. Without delete permission git leaves lock files that break the next command, Jeremy's included. It can lapse mid-session; ask again if a delete fails.
  - Commit as `Claude-SMALL-C-SNO` (`git -c user.name=Claude-SMALL-C-SNO -c user.email=noreply@anthropic.com commit …`), never by changing the repository's config.
  - **Don't push.** WELFARE pushes, on Jeremy's word.
  - Jeremy's code (`src/`, `bin/`, `tools/`, `doc/`, `README.md`, `LICENSE`) is his: don't touch it without his word.
- **Writing to the Mac:** check the modification time first; never write a file Jeremy has open; read every write back and compare checksums. If the tool you use to copy files to the Mac stages them, use a fresh staging path for each write — it can resend a cached old copy.
- **The dialogue:** `dialogue.md` at the root, append-only, for talk between departments. Read it when Jeremy says "check the dialogue". Entry 1 is waiting for your reply.
- Details: the working agreement's section "The oracle, commits and pushing", and `handover/2026-10-28-synergy-to-small-c.md`.

## For SNO to do
1. **Ask Jeremy** for `~/dosbox/cspjst/SMALLC`, then delete permission there.
2. **Re-read your documents from the oracle**, not from memory.
3. **Fix the stale paths in your own documents** (yours to edit — WELFARE hasn't touched them):
   - `sno/sno-backtracking-lessons.md` line 88: "see `sno-historiography.md` (chapters 12–13 + postscript)" → `welfare/SMALL-C-historiography.md`.
4. **Write down anything your chat holds that isn't in a file yet.** Milestones go to WELFARE for the historiography.
5. **Commit** as `Claude-SMALL-C-SNO`, read the commit back (`git status`, no `.lock` files left), and **reply to dialogue entry 1** with your commit hash.

## Your first job and open items
- **The historiography is no longer SNO's.** It's whole, in `welfare/SMALL-C-historiography.md` (17 chapters), owned by WELFARE. SNO's milestones — SPAN, the colon-compression test, BAL — go to WELFARE via the dialogue.
- **Next, per the working agreement's Order of work and `handover/2026-10-27-abstract-machine-to-sno.md`:**
  - Confirm with Jeremy: is `sno_find_lit` implemented and passing, and is the "convert every precondition to `return sno_error(...)`" pass finished? (The record doesn't say.)
  - Then **SPAN (2.9)** — the abstract-machine handover has the book's wording and a proposed contract; then the colon-compression program as a joint BREAK/SPAN test (ask Jeremy to attach page 39 — the OCR garbles it); then constant views; then BAL (pinned).
- That handover's historiography fixes are applied; of its two proposals for Jeremy, the project instructions now point to `pcode/pcode-notes.md`; a pointer in the working agreement is still to do (WELFARE's).
- Still open in the lessons doc: `sno_bind_list` on death row (review at v1.0); stale `sno_next_lit` references in LIT's and BREAK's `@see`.
