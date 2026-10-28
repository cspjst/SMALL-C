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

## Applied by SMALL-C's WELFARE (2026-10-28 19:27–20:20)
*The action list was here; it's done, so it's replaced by this record, as the handover asked. The list itself is in git history (`3fc837c`). The story is in `welfare/SMALL-C-historiography.md`, chapter 17.*
1. **Access** — folder and delete permission granted 19:27. Delete permission lapsed mid-session and was granted again (20:00).
2. **Documents home** — every project document copied into `welfare/`, `sno/`, `pcode/`, `press/` and `handover/`, plus the Green Book's OCR text (Jeremy, 19:30: keep it). Checksummed both sides, and again after a pause. Commit `3fc837c`. The project's copies were replaced by plain-name copies of the oracle after the new instructions went in (20:08); code of conduct v1 bypassed (Jeremy, 20:11).
3. **One historiography** — `welfare/SMALL-C-historiography.md`, owned by WELFARE; the split between `sno/` and `pcode/` dropped.
4. **Code of conduct** — v3, synergy's welfare-first order tailored to SMALL-C, signed by Jeremy 20:03. Commit `322f12e`.
5. **Working agreement** — the oracle and repository rule, commit names, WELFARE as push master, the eggs rule and list (git), the write guards. Commit `65811b2`.
6. **Dialogue** — `dialogue.md` at the root. Commit `a772ad9`.
7. **Pushing** — the first try was refused (Claude's GitHub App had only SYNERGY); Jeremy added SMALL-C. Pushed 20:19 on his word: `main` 9993464 → `a772ad9`, recorded in the dialogue (entry 2, `6a48200`).
8. **Project Instructions** — drafted; pasted by Jeremy 20:08.

## Lessons from synergy
- The Claude app's document window can save its loaded copy over a newer file: never write a file Jeremy has open; check the modification time first; read every write back and compare checksums, once at once and again after a pause.
- The project tool files any top-level document under `claude/`.
- Re-authoring or amending a commit changes its hash: quote hashes only after the last rewrite.
- A renamed GitHub repository has to be added to the session again before Claude can push to it.
