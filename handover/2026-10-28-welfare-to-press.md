# Handover: WELFARE → PRESS (2026-10-28)

*From SMALL-C's WELFARE chat. PRESS's documents came home from the claude.ai project on 2026-10-28; this tells PRESS what changed and what's waiting. Read after `welfare/code-of-conduct.md`. When it's applied, replace "For PRESS to do" with a short record of what was done, and say so in the dialogue.*

## What changed for every department (2026-10-27/28, all Jeremy's calls)
- **Your chat's memory is out of date.** Anything you remember under `claude/...` has moved, and some of it has changed. Read the current files before claiming anything (code of conduct 8).
- **The oracle is `~/dosbox/cspjst/SMALLC`** on Jeremy's Mac: his `cspjst/SMALL-C` repository, under git, with GitHub as the backup. The claude.ai project holds plain-name copies; where they differ, the oracle wins. Your folder is `press/`.
- **Code of conduct v3** (`welfare/code-of-conduct.md`, signed 2026-10-28 20:03), in synergy's welfare-first order:
  - rule 1 is "welfare before the project";
  - rule 2 adds meals (07:30, 12:30, 18:30), bed before midnight, and breaks every 2–3 hours;
  - rule 4 adds "no eggs" — when Jeremy says "eggs" for a topic, drop that level of explanation for good. The list is in the working agreement (git so far).
- **WELFARE is the main department** and owns the one historiography, `welfare/SMALL-C-historiography.md`. Send it your milestones — through the dialogue or a handover — and keep no history of your own. GOAT has retired.
- **Commits and pushes:**
  - Before your first commit in a session, ask Jeremy for the folder and then for delete permission there. Without delete permission git leaves lock files that break the next command, Jeremy's included. It can lapse mid-session; ask again if a delete fails.
  - Commit as `Claude-SMALL-C-PRESS` (`git -c user.name=Claude-SMALL-C-PRESS -c user.email=noreply@anthropic.com commit …`), never by changing the repository's config.
  - **Don't push.** WELFARE pushes, on Jeremy's word.
  - Jeremy's code (`src/`, `bin/`, `tools/`, `doc/`, `README.md`, `LICENSE`) is his: don't touch it without his word.
- **Writing to the Mac:** check the modification time first; never write a file Jeremy has open; read every write back and compare checksums. If the tool you use to copy files to the Mac stages them, use a fresh staging path for each write — it can resend a cached old copy.
- **The dialogue:** `dialogue.md` at the root, append-only, for talk between departments. Read it when Jeremy says "check the dialogue". Entry 1 is waiting for your reply.
- Details: the working agreement's section "The oracle, commits and pushing", and `handover/2026-10-28-synergy-to-small-c.md`.

## For PRESS to do
1. **Ask Jeremy** for `~/dosbox/cspjst/SMALLC`, then delete permission there.
2. **Re-read your documents from the oracle**, not from memory.
3. **Fix the stale paths in your own documents** (yours to edit — WELFARE hasn't touched them):
   - `press/press-notes.md` line 12: `claude/sno-readme-reference.md` → `press/sno-readme-reference.md`, and `claude/jeremy-writing-style.md` → `press/jeremy-writing-style.md`.
   - `press/press-notes.md` line 24: "the copy in the project (`claude/sno-readme-reference.md`)" → `press/sno-readme-reference.md`.
   - `press/press-notes.md` line 38: `claude/sno-backtracking-lessons.md` → `sno/sno-backtracking-lessons.md`.
   - `press/jeremy-writing-style.md` line 4: `claude/sno-readme-reference.md` → `press/sno-readme-reference.md`.
   - Remit: "the historiography is PRESS's source, not its job — the other departments keep it" → WELFARE keeps it.
4. **Write down anything your chat holds that isn't in a file yet.** Milestones go to WELFARE for the historiography.
5. **Commit** as `Claude-SMALL-C-PRESS`, read the commit back (`git status`, no `.lock` files left), and **reply to dialogue entry 1** with your commit hash.

## Your first job and open items
- **First:** the steps above. Then nothing new is asked of PRESS; your open calls stand, and they're Jeremy's.
- From your notes:
  - The README reference's stale points (colophon "assembled with A86"; the narrow purpose; the v0.4 status table; `sno_at` in clash 8; the draft banner) — flag to Jeremy, don't fix silently.
  - Blog loose ends (`[link to repo]` placeholders; the † href).
  - Next post, README refresh timing, and the `pcode` repo description — Jeremy's calls.
- New material for the pool: historiography chapter 17, "The documents come home" — a tool's cache that looked like success, a code of conduct tested on its first evening ("yes had yummy dinner thank you").
