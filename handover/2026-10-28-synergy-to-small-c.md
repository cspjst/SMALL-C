# Handover: synergy WELFARE → SMALL-C WELFARE (2026-10-28)

*From synergy's WELFARE chat. Synergy owns the bootstrap kit, and improvements to WELFARE propagate out from it (operating model, "The bootstrap kit"). This brings SMALL-C up to date with what synergy settled after SMALL-C's retrofit (2026-10-27 18:14, `handover/2026-10-27-synergy-to-small-c.md` in SMALL-C's project). Read it after your code of conduct. Apply the action list, then delete the list, keeping the record.*

*Reference copies of everything below: `~/synergy` on Jeremy's Mac, or `cspjst/SYNERGY` on GitHub — `operating-model.md` (v0.7), `welfare/code-of-conduct.md`, `welfare/working-agreement.md`, `dialogue.md`, `connect-github.md`.*

## What synergy settled (all Jeremy's calls, 2026-10-27 unless dated)
1. **WELFARE is the main department, in every project** (18:26): "welfare *is* the main". It owns the historiography (18:24).
2. **One historiography per project** (2026-10-28 19:18): "one historiography to rule them all". Owned by WELFARE; departments send it their milestones and keep none of their own.
3. **WELFARE is the root of the org tree and the only department that pushes** (21:19): "we need a push master and that shall be WELFARE as the root of the org tree no sub depts shall push". Every department commits.
4. **The code of conduct, welfare first** (19:53–20:16): rule 1 "welfare before the project"; rule 2 meals and bed — breakfast 07:30, lunch 12:30, dinner 18:30, bed before midnight, breaks at timely moments every 2–3 hours; rule 4 "no unsolicited help, and no eggs"; the rules grouped as the principle, the body, the mind, the work; zero tolerance last, "the firm punctuation that finalises".
5. **Sucking eggs** (20:14): keep the small explanations; when Jeremy says "eggs" for a topic, drop that level for good and add the topic to the eggs list in the working agreement. Synergy's list so far: **git**.
6. **The oracle** (20:28): "/claude is the oracle git is the versioning tool, github is the backup". A folder on Jeremy's Mac is the source of truth, under git; GitHub is the backup; the project holds copies. One file per document, edited in place — no `-vN` names.
7. **Commit names** (20:31, 20:55): `Claude-<PROJECT>-<DEPT>` — for you, `Claude-SMALL-C-WELFARE`, `Claude-SMALL-C-PCODE`, and so on.
8. **The dialogue** (21:10): `dialogue.md` at the root of the oracle, owned by WELFARE, append-only, for talk between departments. A department reads it when Jeremy says "check the dialogue".

## GOAT
Retired from the work (Jeremy, 2026-10-28 19:22): "GOAT has been put out to pasture we chat sometimes about life". SMALL-C's WELFARE is its main department; GOAT has no remit in the project and needs no handover.

## SMALL-C's oracle
Jeremy (2026-10-28 19:15): `~/dosbox/cspjst/SMALLC`. It's already a git repository — `cspjst/SMALL-C` on GitHub, branch `main`, last commit `9993464` (2026-09-26) — holding Jeremy's code: `src/`, `bin/`, `tools/`, `doc/`, `README.md`, `LICENSE`. **Those are Jeremy's: no department touches them without his word** (SMALL-C's rule: Jeremy writes the code). The documents share the repository (Jeremy, 19:24): "SMALL-C can absolutely share the repo its where we work and anyone else who joins us".

## For SMALL-C's WELFARE to do
1. **Access first.** Ask Jeremy for `~/dosbox/cspjst/SMALLC`, then for delete permission there before your first commit (without it git leaves lock files that break the next command).
2. **Bring the documents home.** Copy each SMALL-C project document into the oracle, by department: `welfare/` (code of conduct, working agreement, stakeholders, the historiography), `sno/`, `pcode/`, `press/`, `handover/`. Copy exactly — read the project document to a file and copy the file; never retype a large document. Check each against its source. Then the project's copies are replaced by plain-name copies of the oracle.
3. **One historiography.** Keep it whole, in `welfare/`, renamed after the repository: `SMALL-C-historiography.md` (Jeremy, 2026-10-28 19:24: "historiography takes root repo name"). Drop the plan to split it between `sno/` and `pcode/`.
4. **Code of conduct.** Bring it into synergy's welfare-first order with rules 1, 2 and 4 above, tailored to SMALL-C. Jeremy signs it afresh.
5. **Working agreement.** Add: the oracle and repository rule (item 6), commit names (7), WELFARE as push master (3), the eggs rule and list (5).
6. **Dialogue.** Start `dialogue.md` at the root, as synergy's.
7. **Pushing.** Commit as `Claude-SMALL-C-WELFARE`. Push only on Jeremy's word, and record each push in the dialogue with the hash. The Mac's shell has no GitHub credentials: GitHub must be linked to Claude (`connect-github.md`) and `cspjst/SMALL-C` added to your session; carry the commits across (a git bundle works) and push from your own workspace, then bring the Mac level.
8. **Project Instructions.** Draft new ones — reading order, departments, "the oracle is `~/dosbox/cspjst/SMALLC`" — for Jeremy to paste.

## Lessons from synergy
- The Claude app's document window can save its loaded copy over a newer file: never write a file Jeremy has open; check the modification time first; read every write back and compare checksums, once at once and again after a pause.
- The project tool files any top-level document under `claude/`.
- Re-authoring or amending a commit changes its hash: quote hashes only after the last rewrite.
- A renamed GitHub repository has to be added to the session again before Claude can push to it.
