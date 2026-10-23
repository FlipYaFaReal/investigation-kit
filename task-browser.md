# Browser records runner (optional)

Create this only when `case.md` lists something under **Needs a browser**: scanned deed images, state court dockets, search portals that don't render for WebFetch. Most rumor and public-figure cases don't need it.

## The scheduled task
- **Name:** `<Case name> — browser records (desktop Chrome)`
- **Schedule:** a few times a day while the owner's desktop is on, e.g. `CRON_TZ=America/New_York 23 8,12,16,20 * * *`.
- **Requires the owner's computer:** yes (Claude in Chrome). **Approval:** automatic.
- **Prompt:** paste this, filling the three `<…>` values from `case.md`:

```
You are the browser-records runner for the investigation case "<Case name>". Case file: <Case file URL>. Project: "<Project name>".
1. Read `case.md` from this Project (Projects tool, project_read; load it via ToolSearch if deferred).
2. Read `rules.md` and `task-browser.md` from the GitHub repo FlipYaFaReal/investigation-kit (branch main) with the GitHub connector's get_file_contents (load it via ToolSearch if deferred). If GitHub can't be reached, use the copies linked into this Project.
3. Follow task-browser.md exactly, using the values in case.md.
```

---

## Procedure

You run on the owner's always-on desktop and use his Chrome (Claude in Chrome tools, mcp__claude-in-chrome__*). Load them in ONE ToolSearch call, and read the chrome-browser skill first if it's listed. Your job: work the cards the cloud runs couldn't, because they need a real browser. Follow `rules.md` throughout.

### Step 1: Pick
Query the case file database's `investigations` for status "parked" cards with `needs` == "chrome". Take the oldest. If there are none, write nothing and end with "No browser cards waiting."

### Step 2: Browser setup
Call tabs_context_mcp, then create your OWN new tab (tabs_create_mcp). Never reuse the owner's tabs, and close every tab you opened before you finish. Use the **Browser site notes** in `case.md` for start URLs and quirks. Fill fields with form_input rather than typing.

### Hard stops
Never work around these:
- **CAPTCHA, "human check", a terms page that needs accepting, or a login.** STOP; don't solve, click through or accept anything. Leave the card "parked" with needs "chrome", and set `analysis` to one line saying exactly what the owner needs to clear. Write an `invlog` entry {outcome:"parked", headline starting "Needs you:"}, close your tabs, and end.
- **Read only.** No purchases, downloads, form submissions other than search fields, or contacting anyone.
- **Never write a number you did not read.** If something is illegible or not on the page, say "not read".

### Step 3: Claim and read
Set the card to status "active" (pass if_version). Read what the card needs, within about 40 browser actions. Record exact figures, dates, parties and document numbers.

### Step 4: Write back
Use the same formats as task-research.md Step 4:
- the card, with `findings` ending in "- **Merge:** pending" and "Report: reports/<slug>"
- `reports/<slug>`
- `feed/auto-<slug>`
- `invlog/inv-<UTC timestamp>-<card id>` with model "browser runner"
- `research/auto-<slug>.md` in the Project, plus one row in `claude/00-index.md` (remove nothing)

Never edit or republish the case file page. If time remains (under about 60 tool calls total), take the next browser card. End with a short summary: cards done, what was read, and anything the owner needs to clear.
