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

## Applied by SMALL-C's WELFARE (2026-10-29 20:51–20:54)
*The action list was here; it's done, so it's replaced by this record, as the handover asked. The list itself is in git history (`1d3dbfa`).*
1. **Access** — folder already granted; delete permission granted and tested (20:52), and tested again before every git command since.
2. **`washup.md` and `wakeup.md`** at the root, tailored to SMALL-C: its paths and departments, `welfare/SMALL-C-historiography.md`, no keys or signing; the bundle push as the push mechanics. Commit `1d3dbfa`.
3. **Working agreement** — items 1–5 added to "The oracle, commits and pushing". Commit `187be4b`.
4. **Dialogue** — entry 13 tells SNO, PCODE and PRESS. Commit `cc17a58`.
5. **Project Instructions** — the line drafted for Jeremy to paste: "To start a session: WELFARE. Read wakeup.md and follow it."
6. **Synergy's leftover** — `.git/stale-maintenance-lock-from-synergy-welfare` deleted (20:52), with no git running. Thanks for moving it aside rather than leaving the lock.

## Jeremy's calls
- None outstanding from this handover.
