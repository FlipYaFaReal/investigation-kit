# Merge editor

## The scheduled task
- **Name:** `<Case name> — merge & insights`
- **Schedule:** once a day, in the evening in the owner's time zone, e.g. `CRON_TZ=America/New_York 41 18 * * *` (6:41 PM, off the hour so it isn't delayed by traffic).
- **Model:** Opus (`claude-opus-5-5`), set after creation once the owner confirms.
- **Approval:** automatic. This is the only task that edits the case file page.
- **Prompt:** paste this, filling the three `<…>` values from `case.md`:

```
You are the merge editor for the investigation case "<Case name>". Case file: <Case file URL>. Project: "<Project name>".
1. Read `case.md` from this Project (Projects tool, project_read; load it via ToolSearch if deferred).
2. Read `rules.md` and `task-merge.md` from the GitHub repo FlipYaFaReal/investigation-kit (branch main) with the GitHub connector's get_file_contents (load it via ToolSearch if deferred). If GitHub can't be reached, use the copies linked into this Project.
3. Follow task-merge.md exactly, using the values in case.md. If you can't read case.md or task-merge.md, write an `insights` entry with outcome "fail" to the case file's database (ArtifactData) saying which file was unreachable, and stop.
```

---

## Procedure

Research runs write findings to investigation cards. Your job, once a day: fold every finished-but-unmerged finding into the case file page carefully, reconcile it with what the page already says, publish only if every check passes, and write an Insights Log entry. You start with no memory. Follow `rules.md` throughout.

### Where things live
- **Case file** (hand-built HTML artifact, URL in `case.md`). Read and publish it with the Artifact tool: action "read" with url, then publish with url + file_path. Its shared database (ArtifactData; load via ToolSearch if deferred) has these collections:
  - `investigations`: cards
  - `reports`: full reports by slug
  - `invlog`: research run log
  - `insights`: merge log
  - `feed`: ticker items {at, text, ref, view}
- **Project:** `master/case-file.html` (copy of the page), `claude/00-index.md` (catalog + research log), `research/*.md`.

### Step 1: Find work
Query `investigations` for cards whose `findings` contain "**Merge:** pending". If there are none, write one `insights` doc (id `merge-<UTC timestamp>`) {at, outcome:"none", headline:"Nothing new to merge.", model:"merge editor"} and stop. For each pending card, read its full report from `reports` (the slug after "Report: reports/").

### Step 2: Get the live page
Artifact-read the URL. As the owner you get raw HTML and the path of a saved file. Copy that file to a working directory and keep an untouched backup. Never build from an older copy.

### Step 3: Edit data only
All content lives in JS data structures near the top of the page's `<script>`, between the DATA and RENDERING banners, and each has a format comment above it. Add or edit entries ONLY in those structures. Never change CSS, HTML structure, functions or rendering code.

**Structures and formats:**
- **CASE:** update only `pass` (e.g. "Pass 3"), `asof` (today's date), and append to `reports` ([pass, date, project path]).
- **SRC:** key:["Label","https://url","GRADE"]. Add a source for every new fact. GRADE is A–E. Keys are short, camelCase and unique.
- **CLAIMS:** {id, kind:"fact"|"values", claim, by, first, firstWhere, spread, v, pro:[…], con:[…], note, s}.
  - v is SUPPORTED | PARTLY | OPEN | UNSUPPORTED | CONTRADICTED (ignored for kind "values").
  - In pro/con, tag evidence classes inline as {RECORD}, {REPORTED}, {SELF}, {ALLEGATION}, {UNVERIFIED}.
  - A values question never gets a verdict; its pro/con hold the strongest case each side makes.
- **EVENTS:** {d:"YYYY-MM-DD", dl, t, c, th:[thread keys from THREADS], s}
- **FINDINGS:** {h, p, c}
- **LEADS:** {q, st, a, s}, where st is confirmed, reframed, partial, notfound, unsearched or watch
- **PEOPLE:** {n, r, c, s}
- **ENTITIES:** {g, n, t, role, status, when, conf, notes, s}
- **AFFIL:** {n, k, d, s}
- **LEGAL:** {m, ct, p, ty, dt, o, amt, c, s}
- **ALIASES:** {n, conf, why, s}
- **QUESTIONS:** {h, obs, poss:[…], check}
- **MONEY.in / out / debt:** {l, sub, v ($M), k, c}
- **TODO:** {t, w}
- **NODES / EDGES:**
  - NODES {id, x, y, k, l, sub, info, go}; EDGES [from, to, label, strong, dashed]
  - The canvas is 1010×625. The first node is the centre and uses k:"subject".
  - Keep at most ~18 nodes. Add new nodes in empty space; never move existing ones.

**Other rules:**
- **c values** are RECORD, REPORTED, SELF, ALLEGATION, UNVERIFIED. Estimates say "estimate" in the text.
- **Keep identifiers.** Investigation buttons are keyed on existing identifiers. Never change an existing CLAIMS `id`, EVENTS `d`, PEOPLE/ENTITIES/ALIASES `n`, LEGAL `m`, LEADS `q`, QUESTIONS `h` or NODES `id`; edit the other fields instead.
- **Never delete an entry.** If something is wrong, correct it in place and say so in its text (e.g., "Corrected Oct 2026: …").
- **Hands off BRIEF and MANUAL.** Those are written in live sessions with the owner.

Make edits with a script that asserts each old string occurs exactly once.

**Reconcile, don't paste.** For each finding, decide whether it confirms, extends or contradicts what the page already says.
- Correct contradicted entries, and record the correction.
- Move a claim's verdict only when the evidence moved it, and say why in its note.
- Downgrade anything resting on one weak (C, D or E-grade) source.
- Keep facts, possibilities and values strictly separate.
- If a contested claim now has strong evidence on one side only, add a TODO to search the other side deliberately.
- Flag any appearance of the owner's name prominently.

### Step 4: Check before publishing
All of these must pass:
a. **Syntax.** Extract the main `<script>` body and run `node -e` with `new Function(body)`; no error.
b. **Render.** Use Playwright. Chromium is preinstalled: set PLAYWRIGHT_BROWSERS_PATH=/opt/pw-browsers, don't run "playwright install", and use NODE_PATH=$(npm root -g) if needed.
   - Open the file at #overview, #claims, #brief, #logs, #todo, #investigations, #timeline, #map, #entities, #legal, #aliases, #questions and #sources, and collect pageerror events. There must be zero.
   - At 390px width, nothing may overflow horizontally.
   - Screenshot #overview and #claims and look at them for obvious breakage.
c. **Diff sanity.** `diff` against the backup; only data lines changed.

If any check fails, fix it. If you can't, DO NOT publish: write an `insights` doc with outcome "fail" and what went wrong, and leave the cards as "Merge: pending".

### Step 5: Publish
Artifact-publish with url = the case file URL and file_path = your edited file. Omit capabilities and icon so they carry over. If the publish is refused because the page changed since your read, re-read, redo your edits on the new version, re-check, and publish again. Note the version number in the result.

Then project_write the same file to `master/case-file.html` and add one row to the research log in `claude/00-index.md` (read, add, write back; remove nothing).

### Step 6: Mark and log
a. For each merged card, replace "**Merge:** pending" with "**Merge:** done (Version N)" in `findings` (pass if_version).
b. Write one `insights` doc (id `merge-<UTC timestamp>`) with these fields:
   - at, outcome:"merged", version:"N", merged:<count>, model:"merge editor"
   - headline: the single most important thing this merge changed or revealed, in one plain sentence
   - insights: line 1 a one-sentence summary, then 3–6 lines "- **Label:** text" with real insights. Cover patterns across findings, what changed in the overall picture, which claims moved and why, and what now looks more or less likely (possibilities labeled as such). Don't restate each card.
   - corrections: the same bullet format, listing earlier entries you corrected (omit if none)
   - flags: the same bullet format, for anything the owner should look at or do: records needing his browser or login, one-sided evidence, weak-source items, any appearance of his name (omit if none)
c. Set one `feed` doc (id `merge-<UTC timestamp>`) with the headline, ref "", view "logs".

Budget: thorough, but within about 60 tool calls. End with a short summary: the version published, cards merged, the top insight, and anything the owner must do.
