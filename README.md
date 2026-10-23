# Investigation kit

Rules, procedures and an empty case-file page for running public-record investigations on any subject: a person, a company, a political rumor, a public controversy. Built from the Von Canon case file. Each case runs in its own claude.ai Project, with its own case file and its own scheduled runs. This repo holds the parts every case shares.

## Start a new case
1. **Create a Project** in claude.ai, named after the case.
2. **Paste the instructions** from `PROJECT-INSTRUCTIONS.md` into the Project. They're the same every time.
3. **Link this repo** to the Project.
4. **Open a chat** in the Project and say *"Kick off a case on <subject>. <why you're asking>."*

The chat follows `kickoff.md`:
- writes `case.md` with you
- publishes the case file and seeds the first findings, claims and investigation cards
- creates the scheduled tasks
- asks you to confirm their models
- test-runs both tasks before handing over

## How a case runs
```
 kickoff chat seeds investigation cards
        │
 research runner (Sonnet, every 3 hours) takes the oldest open card,
 researches it, writes findings + a report, marks it "Merge: pending"
        │
 merge editor (Opus, once a day ~6:40 PM) folds pending findings into the
 case file's data, checks it renders, publishes, writes the Insights Log
        │
 case file ◀── you read it and add cards with any Investigate button
```

## Files
| File | Generic or per case | Purpose |
|---|---|---|
| `PROJECT-INSTRUCTIONS.md` | generic | Paste into every investigation Project |
| `rules.md` | generic | Evidence labels, source grades, facts vs values, steelmanning, privacy, owner-name rule |
| `kickoff.md` | generic | Runbook for the first chat in a new Project |
| `case-template.md` | template | Becomes the Project's `case.md`, the only per-case file |
| `task-research.md` | generic | Research runner: thin task prompt + full procedure |
| `task-merge.md` | generic | Merge editor: thin task prompt + full procedure |
| `task-browser.md` | generic, optional | Desktop-Chrome runner for records that need a real browser |
| `case-file-shell.html` | generic | The case-file page with all data empty |

## Why the tasks read from this repo
Each scheduled task's prompt is three lines:
1. read `case.md` from the Project
2. read the procedure from this repo
3. follow it

Improving a procedure here changes every case's next run. No task needs editing. Case details never go into this repo; they stay in each Project's `case.md`.

## The case file page
- **Views:** Overview, Claims Ledger, Decision Brief, Run Logs, To-Dos, Investigations, Timeline, Connections, Entities & People, Legal & Money, Names, Open Questions, Sources.
- **Claims Ledger:** one card per claim with who says it, where it started, how it spread, evidence for and against, and a verdict (Supported, Partly supported, Can't tell yet, Unsupported, Contradicted).
- **Values questions:** get the strongest case from each side and no verdict.
- **Source grades:** A–E beside every citation.
- **Your-name flag:** the Overview lists every place the owner's name appears.
- **Config:** a `CASE` block at the top of the data holds the case details; the scheduled merge edits only the data.

## Running costs
- Research every 3 hours is 8 runs a day; the merge adds 1.
- Pause a case's tasks when it goes quiet. Pausing loses nothing.
- Each case's tasks only touch that case's case file.
