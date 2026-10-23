# Research runner

## The scheduled task
- **Name:** `<Case name> — research`
- **Schedule:** every 3 hours: `0 */3 * * *`.
- **Model:** Sonnet (`claude-sonnet-5-5`), set after creation once the owner confirms.
- **Approval:** automatic (runs can't wait for a click).
- **Prompt:** paste this, filling the three `<…>` values from `case.md`:

```
You are the research runner for the investigation case "<Case name>". Case file: <Case file URL>. Project: "<Project name>".
1. Read `case.md` from this Project (Projects tool, project_read; load it via ToolSearch if deferred).
2. Read `rules.md` and `task-research.md` from the GitHub repo FlipYaFaReal/investigation-kit (branch main) with the GitHub connector's get_file_contents (load it via ToolSearch if deferred). If GitHub can't be reached, use the copies linked into this Project.
3. Follow task-research.md exactly, using the values in case.md. If you can't read case.md or task-research.md, write an `invlog` entry with outcome "fail" to the case file's database (ArtifactData) saying which file was unreachable, and stop.
```

---

## Procedure

Each run processes EXACTLY ONE open investigation card, writes the result back, logs it, and stops. You start with no memory; the case file's database and the Project are your memory. Follow `rules.md` throughout. "The owner" means the Owner in `case.md`.

### Where things live
- **Case file** (the URL in `case.md`). Its shared database (ArtifactData tool; load via ToolSearch if deferred) has these collections:
  - `investigations`: cards with ref, title, kind, view, note (the owner's own note; read it, it often says exactly what he wants), analysis, findings, status (open | active | parked | done), needs, origin, createdAt, updatedAt.
  - `feed`: ticker items {at (ISO time), text (one line, ≤160 chars), ref, view}.
  - `reports`: full reports; doc id = report slug; {title, at, body (markdown), card}.
  - `invlog`: the Investigation Log on the Run Logs tab: {at, card, ref, title (≤120 chars), outcome ("done" | "parked" | "none" | "fail"), headline (one or two plain sentences), report (slug, or omit), model}.
- **Project:** `claude/00-index.md` is the catalog and research log; `research/*.md` are prior reports. Read the index first and use project_search to see what's already known about your card's subject before researching.

### Step 1: Pick
Query `investigations` where status == "open". Some cards can't be done from the cloud:
- If a card needs something listed under **Needs a browser** in `case.md`: set status "parked", needs "chrome", and a one-line `analysis` naming exactly what to read.
- If a card needs something listed under **Needs the owner**, or the owner's own records: set status "parked", needs "paul", and a one-line `analysis` saying exactly what he should do.

For each parked card, write an `invlog` entry with outcome "parked" (id `inv-<UTC timestamp>-<card id>`), then move on. Of the remaining cards, take the OLDEST by createdAt. If none are left, write one `invlog` entry with outcome "none" and headline "No open cards." (but ONLY if the newest existing invlog entry isn't already "none"), then end the run.

If partway through you find the answer sits in a record you can't open, finish what you can, then park the card with the right `needs` and say exactly what to read.

### Step 2: Claim
Update the card to status "active". Always pass if_version from your read.

### Step 3: Research
Use WebSearch / WebFetch and the Project docs. Prefer the **Primary sources** in `case.md`. Budget about 40 tool calls; finish with what you have rather than running long.
- For a contested claim, run supporter-style and critic-style searches, and report the best evidence on both sides.
- If the card is really a values question, say so. Give each side's strongest case and check the facts inside it.

### Step 4: Write back
a. **Card.** Set status "done" (or "parked" with needs), updatedAt = now, and `analysis` = one or two sentences on what you checked. Write `findings` in this exact format:
   - line 1: a one-sentence summary
   - one fact per line: "- **Label:** text", with the evidence class and source grade in the text, e.g. "(RECORD, A)"
   - if the card tests a claim: "- **Verdict:** SUPPORTED | PARTLY | OPEN | UNSUPPORTED | CONTRADICTED | VALUES"
   - then the line "- **Merge:** pending"
   - last line: "Report: reports/<slug>"

b. **Report.** Set a doc in `reports` (id = short slug). Its markdown body has:
   - a title
   - an "Answer" paragraph
   - a table: Item | Class | Grade | Date | Source URL
   - for contested claims: "Strongest case for" and "Strongest case against"
   - "Possibilities (not conclusions)"
   - "Gaps / what would settle it"
   - one line on the owner's name: found (where) or not found

   Also project_write it to `research/auto-<slug>.md`. Then append one row to the research log table in `claude/00-index.md`: read it, add the row, write it back, and remove nothing.

c. **Ticker.** Set one `feed` doc (id `auto-<slug>`) with the headline finding.

d. **Log.** Set one `invlog` doc (id `inv-<UTC timestamp>-<card id>`) with outcome, title, headline, report slug and model "research runner". If the run fails partway, still write an invlog entry with outcome "fail" and what went wrong, and set the card back to "open".

Never edit or republish the case file page; the merge run does that. Use batch writes where possible. End with a three-line summary: which card, the headline finding, and anything the owner needs to do.
