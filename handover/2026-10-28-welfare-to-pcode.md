# Handover: WELFARE → PCODE (2026-10-28)

*From SMALL-C's WELFARE chat. PCODE's documents came home from the claude.ai project on 2026-10-28; this tells PCODE what changed and what's waiting. Read after `welfare/code-of-conduct.md`. When it's applied, replace "For PCODE to do" with a short record of what was done, and say so in the dialogue.*

## What changed for every department (2026-10-27/28, all Jeremy's calls)
- **Your chat's memory is out of date.** Anything you remember under `claude/...` has moved, and some of it has changed. Read the current files before claiming anything (code of conduct 8).
- **The oracle is `~/dosbox/cspjst/SMALLC`** on Jeremy's Mac: his `cspjst/SMALL-C` repository, under git, with GitHub as the backup. The claude.ai project holds plain-name copies; where they differ, the oracle wins. Your folder is `pcode/`.
- **Code of conduct v3** (`welfare/code-of-conduct.md`, signed 2026-10-28 20:03), in synergy's welfare-first order:
  - rule 1 is "welfare before the project";
  - rule 2 adds meals (07:30, 12:30, 18:30), bed before midnight, and breaks every 2–3 hours;
  - rule 4 adds "no eggs" — when Jeremy says "eggs" for a topic, drop that level of explanation for good. The list is in the working agreement (git so far).
- **WELFARE is the main department** and owns the one historiography, `welfare/SMALL-C-historiography.md`. Send it your milestones — through the dialogue or a handover — and keep no history of your own. GOAT has retired.
- **Commits and pushes:**
  - Before your first commit in a session, ask Jeremy for the folder and then for delete permission there. Without delete permission git leaves lock files that break the next command, Jeremy's included. It can lapse mid-session; ask again if a delete fails.
  - Commit as `Claude-SMALL-C-PCODE` (`git -c user.name=Claude-SMALL-C-PCODE -c user.email=noreply@anthropic.com commit …`), never by changing the repository's config.
  - **Don't push.** WELFARE pushes, on Jeremy's word.
  - Jeremy's code (`src/`, `bin/`, `tools/`, `doc/`, `README.md`, `LICENSE`) is his: don't touch it without his word.
- **Writing to the Mac:** check the modification time first; never write a file Jeremy has open; read every write back and compare checksums. If the tool you use to copy files to the Mac stages them, use a fresh staging path for each write — it can resend a cached old copy.
- **The dialogue:** `dialogue.md` at the root, append-only, for talk between departments. Read it when Jeremy says "check the dialogue". Entry 1 is waiting for your reply.
- Details: the working agreement's section "The oracle, commits and pushing", and `handover/2026-10-28-synergy-to-small-c.md`.

## Done by PCODE (2026-10-28, 20:35–20:36)
1. Jeremy granted `~/dosbox/cspjst/SMALLC`, then delete permission there.
2. Re-read from the oracle: the code of conduct v3, the working agreement, `pcode/pcode-notes.md` (the oracle's copy is newer than the claude.ai project's: approaches, references, Appendix A), the historiography's PCODE chapters (3, 5, 16) and the dialogue.
3. Stale paths fixed in `pcode/pcode-notes.md`: line 3 now points to `welfare/SMALL-C-historiography.md`, owned by WELFARE, with milestones sent to WELFARE; lines 94 and 107–109 now say `pcode/…`.
4. Nothing new to write down: this PCODE chat has only read since 2026-10-27; no PCODE milestones are waiting for the historiography.
5. Committed as `Claude-SMALL-C-PCODE`, not pushed; reply in the dialogue (entry 5).

## Your first job and open items
- **First:** the steps above. Then back to P0, where Jeremy left it.
- From your notes (Jeremy's calls): LEAVE (what it does, and how it differs from EXIT); SYS's `.tgt` entry; constants (`PUSH 5` or a separate `LIT`); SUB/DIV operand order; divide by zero; values versus addresses (`ADDR`/`LOAD`/`STORE`). Next listed: ADDR, LOAD and STORE in `8086.tgt`, then the first pointer program.
- Status as last recorded (2026-10-27): v0.1 in progress — ENTER, LEAVE, EXIT and SYS. Jeremy's files win over the notes.
