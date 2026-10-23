# Kickoff runbook

For the first chat in a new investigation Project, when the owner says something like "kick off a case on X". Follow these steps in order. The kit files live in the GitHub repo **FlipYaFaReal/investigation-kit** (branch main) and are linked into the Project. Read `rules.md` and this file before starting.

## 1. Write case.md with the owner
1. Draft `case.md` from `case-template.md`, using what the owner has said. Ask only for what's missing, in one round: the focus questions, the privacy line, and anything under "Needs a browser" or "Needs the owner".
2. Leave **Case file** blank for now (step 2 fills it).
3. Save it to the Project as `case.md` with project_write. It's the only per-case file; never put case details into the kit repo.

## 2. Publish the case file
1. Read `case-file-shell.html` from the kit. Copy it to the working directory.
2. Fill in its `CASE` block: key, title, subject, subjectShort, searchName, watchName (the owner's full name), owner, purpose, mergeTimes ("the daily merge, around 6:40 PM"), pass "Pass 1", asof. Fill in `THREADS` with 4–6 short keys that fit the case. Leave the data arrays empty for now.
3. Load the `artifact-capabilities` skill. Publish as a NEW artifact (no `url`) with `capabilities` `{"db":{},"sample":{},"user":{}}` and a one-word icon. Without `db`, the cards, logs and ticker have nowhere to live.
4. Put the new URL into `case.md` under **Case file**, and save it again.

## 3. First research pass and seed
1. Research the focus questions under `rules.md`: label every fact, grade every source, run both supporter and critic searches on contested claims, and keep values questions apart from facts.
2. Fill FINDINGS, CLAIMS (facts with verdicts; values questions with both sides and no verdict), EVENTS, PEOPLE, ENTITIES, SRC (with grades), QUESTIONS and TODO.
3. Run the checks in `task-merge.md` Step 4, then republish to the same URL.
4. Seed 8–15 open cards in the `investigations` collection (ArtifactData): {ref:"custom-<slug>", title, kind:"Investigation", view, note, analysis:"", findings:"", status:"open", origin:"claude", createdAt, updatedAt}. Each card is one answerable question. Cover both sides of every contested claim.
5. Write to the Project:
   - `research/01-kickoff.md`
   - `master/case-file.html`
   - `claude/00-index.md`, holding the catalog, evidence classes and grades, a research log table with the kickoff row, and the owner-name rule

## 4. Create the scheduled tasks
1. List existing scheduled tasks first. Don't duplicate one, and never change another case's tasks.
2. Create two tasks from THIS chat, using the thin prompts in `task-research.md` and `task-merge.md`:
   - **`<Case name> — research`:** `0 */3 * * *`
   - **`<Case name> — merge & insights`:** `CRON_TZ=<owner's time zone> 41 18 * * *` (once a day, 6:41 PM)

   Fill the three `<…>` values in each prompt from `case.md`, and check that no `<` placeholder is left. Use initiation `human_request`.
3. Create `task-browser.md`'s task only if `case.md` lists something under "Needs a browser". It must require the owner's computer.
4. **Models.** Ask the owner in one line: "Set research to Sonnet and the daily merge to Opus?" Model changes need the owner's own go-ahead, so ask even though this is the kit's default. On a yes, call update_trigger with model `claude-sonnet-5-5` on the research task and `claude-opus-5-5` on the merge task. If a model ID is rejected, pick the current Sonnet or Opus ID from the error or the org's list and tell the owner which you used.
5. **Approval.** Each create result reports its approval setting. Runs only work unattended with automatic approval. If a task didn't get it, tell the owner to switch it on in that task's settings.

## 5. Prove it works
1. Fire the research task once (fire_trigger). Wait, then check three things:
   - an `invlog` entry
   - one card changed to done or parked
   - its report saved in the Project under `research/`

   If the report is in the database but not the Project, the runs can't reach the Project; tell the owner.
2. Fire the merge task once. Check that the `insights` collection got a "merged" or "none" entry. If it merged, check that the page's version number went up.
3. If either fails, read what it logged, fix the cause, and fire it again before handing over.

## 6. Hand over
One short message covering:
- the case file link
- how many cards are queued
- each task's name, schedule, model and approval setting
- anything the owner must do
