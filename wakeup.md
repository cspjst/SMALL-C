# Wakeup — starting a chat cold from the oracle

*Owned by SMALL-C's WELFARE. Tailored on 2026-10-29 from synergy's `wakeup.md` (`handover/2026-10-29-synergy-to-small-c.md`); the complement of `washup.md`. A new session remembers nothing: it trusts nothing until it has checked it. Start here when Jeremy says "wakeup", or "Read wakeup.md".*

***Only* WELFARE wakes up and washes up (Jeremy, in synergy, 2026-10-29 19:44: "*ONLY* welfare can wakeup and washup"). SNO, PCODE and PRESS don't run these procedures.**

## The steps
1. **Welfare first.** Check the time against the code of conduct's meal times and bedtime (rule 2), and ask how Jeremy is — before anything else.
2. **Access.** Ask Jeremy for the oracle folder, `~/dosbox/cspjst/SMALLC`, then for delete permission there, and test it: create and remove a scratch file under `.git`. Repeat the test before every git command, `git fetch` included — without it git leaves lock files that break the next command, Jeremy's too.
3. **Read, in order:**
   1. `welfare/code-of-conduct.md` — it wins over everything else;
   2. `welfare/working-agreement.md` — its Status and Order of work first;
   3. the latest washup handover — `handover/<date>-welfare-washup.md`, newest date — and any other handover addressed to WELFARE that isn't yet applied;
   4. `welfare/SMALL-C-historiography.md` — the latest chapter at least;
   5. `dialogue.md` — every entry since WELFARE's last.
   Departments' own notes (`sno/`, `pcode/`, `press/`) are theirs: read them only when the work needs them.
4. **Check git, both ways.** `git fetch` (after the delete test); then what's on GitHub that isn't here (`main..origin/main` — Jeremy commits from other machines), what's here that isn't on GitHub (`origin/main..main`), uncommitted changes, and leftover `.lock` files. Don't touch another department's or Jeremy's uncommitted work.
5. **Check what changed while asleep.** New dialogue entries and commits from SNO, PCODE or PRESS; whether the project's copies and its Instructions still match the oracle.
6. **Check identity.** Recent commits are by Jeremy or `Claude-SMALL-C-<DEPT>`, and the repository's own git config has no `user.name` — Claude's name is set per commit, never there.
7. **Report, then wait.** Tell Jeremy, briefly: what changed since the washup, his open calls, and one suggested first step. Waking up isn't permission to start work.

## The mechanics a woken session needs (SMALL-C WELFARE)
- **Commit** as `Claude-SMALL-C-WELFARE`, `noreply@anthropic.com` (`git -c user.name=… -c user.email=… commit`), with the `Co-Authored-By` and `Claude-Session` trailers; stage named paths only.
- **Push** only on Jeremy's word (a washup is that word). The Mac's shell has no GitHub credentials: bundle on the Mac (`git bundle create .git/smallc.bundle origin/main..main`), stage it into the workspace, fetch it into a clone of `cspjst/SMALL-C` (add the repository to the session with push access first; Claude's GitHub App must include SMALL-C), push, `git fetch` on the Mac, delete the bundle. Record the push in `dialogue.md` with the hash.
- **Write guards:** never write a file Jeremy has open; check the modification time; read every write back and compare checksums, at once and again after a pause; stage each write to the Mac under a fresh path (the copy tool can resend a cached old copy); times come from the clock.
- **The project** holds plain-name copies at the paths its Instructions use (`welfare/`, `sno/`, `pcode/`, `press/`, `handover/`). Its tool won't write at the top level, so these two files' copies live in `welfare/`; `dialogue.md` lives in the oracle only.
- **Eggs:** git — report what happened; don't explain it.
