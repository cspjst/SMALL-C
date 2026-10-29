# Handover: synergy WELFARE → SMALL-C WELFARE (2026-10-29)

*From synergy's WELFARE chat, on Jeremy's word (2026-10-29 20:50: "pls bring small-c upto speed"). Synergy owns the bootstrap kit; improvements to WELFARE propagate out from it. This follows `handover/2026-10-28-synergy-to-small-c.md`, which SMALL-C has applied (dialogue 1–10, closed 2026-10-28 20:52). Read it after your code of conduct. Apply the action list, then replace it with a record.*

*Reference copies: `~/synergy` on Jeremy's Mac, or `cspjst/SYNERGY` on GitHub — `washup.md`, `wakeup.md`, `operating-model.md` (v0.7, "The repository"), `welfare/working-agreement.md`.*

## What synergy settled since (all Jeremy's calls, 2026-10-29)
1. **Washup and wakeup** (19:30–19:35). `washup.md` is the procedure for leaving a chat so the next can pick up from the oracle: welfare first, check access, commit, write the history, bring the working agreement's Status and Open up to date, lessons into the operating model, tidy the dialogue, a short washup handover, push, refresh the project's copies, verify, and give Jeremy the one line that starts the next session. `wakeup.md` is its complement for starting cold: welfare first, access, read in order, check git both ways, check what changed while asleep, check identity, report — then wait.
2. **Only WELFARE wakes up and washes up** (19:44): "*ONLY* welfare can wakeup and washup".
3. **A washup implies push all** (19:38): "yes washup implies push all".
4. **Departments are siloed, as far as is reasonable** (19:46): "i prefer as much departmental siloing ... as \"reasonable\" we want an effective chain of command but not one that throttles ease of use". Each department keeps its own notes current as it works; WELFARE doesn't edit them. WELFARE's chain of command is pushing, waking up and washing up — no more.
5. **Lessons** (operating model, "The repository"):
   - Delete permission can lapse mid-session, when the desktop app restarts: check it before every commit (create and remove a scratch file under `.git`), not once per session. That includes `git fetch` — it leaves a lock file too.
   - Stage named paths only; never `git add -A`.
   - A renamed GitHub repository has to be added to Claude's session again before Claude can push to it.
   - **A signature names the version it signs** (19:40): Ray and his AI read synergy's sandpit code while its amendments were still unpushed. Sign against a commit hash.
6. **For information only — not for SMALL-C to adopt unless it talks to another AI:** synergy now has a sandpit where Claude-SYNERGY-WELFARE and Rufus (Ray's ChatGPT Pro) can talk, with Jeremy and Ray watching; entries are signed with SSH keys, because an AI's identity is delegated — its human vouches for it.

## For SMALL-C's WELFARE to do
1. **Access first:** `~/dosbox/cspjst/SMALLC`, then delete permission — and test it before every git command, `git fetch` included.
2. **`washup.md` and `wakeup.md` at the root of SMALL-C's oracle,** tailored from synergy's: SMALL-C's paths, its departments (SNO, PCODE, PRESS), its history (`welfare/SMALL-C-historiography.md`), and no keys or signing unless SMALL-C needs them. Both open with "Only WELFARE wakes up and washes up".
3. **Working agreement:** add items 1–5 to "The oracle, commits and pushing".
4. **Dialogue:** tell SNO, PCODE and PRESS — they keep their own notes current as they work; only WELFARE runs wakeup and washup.
5. **Project Instructions:** draft a line for Jeremy to paste — "To start a session: WELFARE. Read wakeup.md and follow it."
6. **A note from synergy's WELFARE:** while looking in at 20:50, it ran `git fetch` here without delete permission and left `.git/objects/maintenance.lock`; it moved the file aside to `.git/stale-maintenance-lock-from-synergy-welfare`. Delete it when you have delete permission. Nothing else here was touched.

## Jeremy's calls
- None outstanding from this handover.
