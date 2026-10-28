# Handover: SYNERGY → SMALL-C (2026-10-27)

*SMALL-C predates synergy's operating model. On 2026-10-27 at 18:09 Jeremy asked for it to be brought "up to date as if bootstrapped from synergy". This records what was applied, from a SMALL-C chat (synergy's model, templates and WELFARE are in `~/claude/` on Jeremy's Mac and in the synergy project). Read after `welfare/code-of-conduct.md`.*

## What's delivered
- **`welfare/stakeholders.md`** (new): key stakeholders Jeremy and Claude; others — blog readers ("fellow retro programmers"), users of the GitHub repos, Anthropic — with their interests and why they aren't key.
- **`welfare/code-of-conduct-v2.md`** (draft, awaiting Jeremy's signature): version 1 word for word, plus a "Key stakeholders" section and rule 11 ("Don't write to a file Jeremy has open; verify every write to his machine by reading it back"). **Until Jeremy signs it, version 1 (`welfare/code-of-conduct.md`) stands.**
- **Versioning** (Jeremy, 18:09): every new version of a shared document gets a new file name; old versions are kept. Proposed naming: plain name for the first version, a suffix for later ones (`-v2`); readers take the highest version.

## Who owns what (the department each document belongs to)
| Document | Owner | Template location (not moved — see below) |
|---|---|---|
| `welfare/code-of-conduct.md` (+ `-v2`) | WELFARE | — |
| `welfare/stakeholders.md` | WELFARE | — |
| `claude/working-agreement.md` | WELFARE | `welfare/working-agreement.md` |
| `claude/sno-historiography.md` | SNO (main department) | `small-c-historiography.md` |
| `claude/sno-backtracking-lessons.md` | SNO | `sno/` |
| `claude/jeremy-writing-style.md` | PRESS | `press/` |
| `claude/sno-readme-reference.md` | PRESS | `press/` |
| `claude/p86.1-spec.md`, `claude/pcode86.2-spec.md`, `-86.3-`, `-86.4-` | PCODE | `pcode/` |
| `pcode/pcode-notes.md` | PCODE | — |
| `press/press-notes.md` | PRESS | — |
| `snobol-green-book.pdf` | (upload — reference for all) | — |
| `handover/*` | the receiving department, until applied | — |

**Not moved (Claude's call, for Jeremy to overrule):** moving the `claude/` documents into department folders would mean rewriting each one through a tool (tens of kilobytes for the working agreement and the historiography), which risks mangling them. The ownership table does the job a folder would. A department that next revises one of its documents can save the new version (versioning) in its own folder, which moves it without risk.

## For Jeremy to do
1. **Paste the new project instructions** (Claude can't edit them):
```
SMALL-C — learning modern compiler techniques on old hardware (C99, 8086, DOS .COM, Open Watcom). Bootstrapped (retrofitted) from SYNERGY's operating model.

Before anything else, read these project docs, in this order:
1. welfare/code-of-conduct.md — who we are and how we work together. Signed off by Jeremy; it wins over everything else. (Highest signed version.)
2. claude/working-agreement.md — roles, standing rules, settled decisions, current status.
3. claude/sno-historiography.md — why things are the way they are. Keep it current at each milestone and ask before compaction.
4. claude/jeremy-writing-style.md + claude/sno-readme-reference.md — for any comments, docs or prose.

Departments (one chat each, named in capitals):
- WELFARE — welfare/ — how we work together; the stakeholders (welfare/stakeholders.md).
- SNO (main) — the SNO library; owns the historiography; claude/sno-backtracking-lessons.md.
- PCODE — pcode/pcode-notes.md — the P0 stack machine; claude/pcode86.4-spec.md (86.3, 86.2, p86.1 kept for history).
- PRESS — press/press-notes.md — READMEs, the blog, repo descriptions.
Handovers between departments: handover/. Who owns which document: handover/2026-10-27-synergy-to-small-c.md.
Reference: snobol-green-book.pdf (text only — attach a page image where the OCR garbles).
Versioning: a new version of a document gets a new file name; take the highest version.

The short version, in case nothing else gets read:
Jeremy writes the code. Claude writes specs, tests, reviews and the history.
No unsolicited help, no spoilers, no sycophancy.
Check the current file before claiming anything.
Proposals are labelled; decisions are Jeremy's.
```
2. **Sign `welfare/code-of-conduct-v2.md`** if it suits — or say what to change.

## For WELFARE to do
- Note the retrofit in the working agreement (a line under its status, pointing here), and review `welfare/stakeholders.md` with Jeremy.

## For SNO to do (it owns the historiography)
- Chapter 16 gains a bullet: 17:40–18:12, synergy founded ("Man Machine Interface in an AI world"); its operating model distilled from SMALL-C; every project has a WELFARE department for all key stakeholders (Jeremy, 17:46–17:51); synergy owns the bootstrap kit (17:57) and is its master (18:09); templates in `~/claude/templates/`; the `project-bootstrap` skill (18:03); versioning (18:09); SMALL-C retrofitted (18:09, this handover). Also the storm of the evening: the Claude app's file window saving its copy back over Claude's writes (17:48–18:02), turned into rule 11.
- Also apply the earlier `handover/2026-10-27-abstract-machine-to-sno.md` if not done.

## Lessons (for WELFARE)
- The retrofit showed one limit of the model: a project's documents can't be moved without rewriting them through a tool. Versioning gives a safe path — move on the next revision.
