# MDR Exchange Tracker

**Build:** `2026-10-14-export-split-return-tz` (October 14, 2026)
**Owner:** Antwain Jordan, Medical Resources Inc. (MDR)
**Live site:** https://antwainjordan-mdr.github.io/mdr-exchange-tracker/

A shared tracker for equipment exchanges and pickups between MDR and SGI Logistics. Each row is one job: the patient, the pieces SGI should exchange or pick up, and the check-in, closing and confirming steps. Records are stored in SharePoint (site **MDRExchangeTracker**, list **ExchangeRecords**). Everyone sees the same data.

## Files

| File | Purpose |
|---|---|
| `index.html` | The whole app (single file). This is what GitHub Pages serves. |
| `README.md` | This file. |

`data.csv` (the daily Power BI export) is **not** part of the app and must never be committed to the repo. You pick it from your PC with **Load Daily**.

**Version marker:** the build name is `const BUILD = '...'` near the top of the script. It prints in the browser console on load (`[MDR Exchange Tracker] build ...`). Bump it on every deploy, and add a `CHANGELOG` entry when the change is something users will see.

---

## Signing in

- Open the live site and click **Sign in with Microsoft**. Use your Microsoft 365 account.
  - `@mdr.net` accounts get the **MDR** view.
  - SGI accounts get the **SGI** view, branded "SGI Logistics Portal".
  - Any other domain is refused.
- You stay signed in all day. The token refreshes quietly about 5 minutes before it expires. If that fails, a "Session expired" screen asks you to sign in again.
- Click your email (the chip in the header) to **sign out**. Signing out also clears the records cached in that browser.
- **Opening a saved copy from a drive** (for example `Y:\MDR Apps\Exchange Tracker\index.html`) shows a locked screen with an **Open live site** button. Sign-in only works on the live site.
- After a fresh sign-in, a **What's New** box shows the latest update. **Updates** in the header shows the full history.

## The header buttons

| Button | Who | What it does |
|---|---|---|
| SGI View / MDR View | MDR | Preview what the other side sees. Testing only, changes nothing. |
| Light / Dark Mode | All | Switches the theme. Remembered per browser. |
| Flash: On / Off | All | Turns the red pulsing on overdue rows on or off. |
| Refresh | All | Reloads every record from SharePoint. |
| **Export as: CSV** (green) | All | Downloads the rows **on screen** as a CSV (see Exporting). |
| **Export as: PDF** (red) | All | Prints or saves the rows on screen as a PDF report. |
| Load Daily | MDR | Imports today's `data.csv` from Power BI (see Load Daily). |
| Add Record | All | Adds a job by hand. |
| Clear Cache | All | Wipes this browser's saved data and tokens and reloads. Fixes most odd behavior. You sign in again after. |
| App Log | MDR | Who did what, and when (sign-ins, loads, exports, refreshes). |
| Updates | All | Full update history. |

Next to the buttons, the **presence bar** shows who else has the tracker open (seen in the last 10 minutes). The **sync light** shows Saved, Syncing, Synced or an error.

## Filters

- **Month chips:** All, plus the current month and the two before it. **Custom** opens start and end date pickers. Months and ranges filter on **Pickup Date**.
- **Search:** patient name, patient ID, job #, reference numbers, piece names and serial numbers. It searches every matching record, not only the rows already loaded on screen.
- **Status:**

| Status | MDR view | SGI view |
|---|---|---|
| Incomplete | No progress at all: no piece checked, no Date Returned, not closed, verified or confirmed. | (not shown) |
| Pending | Some progress, but not Closed + Verified yet. | Anything MDR has **not** Confirmed yet. |
| Processing | Job Closed **and** Verified, waiting for Confirmed. | (not shown) |
| Complete | Confirmed. | Confirmed. |
| All Active | Everything. | Everything. |

- The **counter pills** (Incomplete, Pending, Partial, Processing, Complete, Memory Download) show totals for the current filters.
- **Partial, Shortage and Overage** only describe how items came back. They never move a record between statuses.
- The table loads 50 rows at a time and loads more as you scroll. Click a column header with ⇅ to sort.

## Reading a row

| Column | Notes |
|---|---|
| Pickup Date | Click to set or change. |
| Patient | Name and patient ID. **NEW** marks rows from a Daily load. 🔒 means the value came from the source file and is locked. |
| Job # | SGI job number. MDR: it links to SGI RapidShip tracking. |
| Pieces / Items | Every piece, with **EXC** (Exchange) or **PKP** (Pickup). A device's serial number shows on its own line under it (`SN: ...`), and ★ MD marks the device that needs a Memory Download (MDR only). |
| Reference | SO and PuEx numbers. MDR: 6-digit numbers starting with 1 open the Brightree Sales Order, and 5-digit numbers starting with 3 open the PuEx. SGI sees plain text. |
| Date Returned | Set automatically when the job is closed, or set by hand. |
| Job Closed (SGI) | Checking it stamps Date Returned with today's date. |
| No Pickup | Marks the job as nothing collected: every piece is checked with qty 0, and Date Returned and Job Closed are stamped. Shows an X. |
| Verified (MDR RCV) | MDR confirms the items were received. SGI can see it but can't change it. |
| Confirmed (Portal) | MDR's final sign-off. It moves the record to Complete for both sides. |
| SGI Notes / MDR Notes | Free text, saved when you leave the box. SGI can read MDR notes but not edit them. |
| Activity (clock icon) | That record's full change history from SharePoint. |

**Row colors:** a blue bar is Exchange, purple is Pickup, yellow is Mixed. A pulsing red row is **overdue**: its Drop-off By date is today or earlier and it isn't Closed or Complete.

## Checking in pieces

- Tick a piece's checkbox when it comes back. The received qty fills in with the expected qty (shown as ×N). Change it if a different number came back, and **SHORT** or **OVER** appears.
- **Select All / Deselect All** ticks or clears every piece on the row. When the last piece is checked, the job closes automatically and Date Returned is set to today.
- **+ Add item** adds a piece that wasn't on the source list, from the item library or by typing a new name. Pieces added this way can be removed (✕). Pieces from the source are locked.
- **Serial numbers follow their device.** When a device name ends in a serial (for example `Oximeter MBH2601549` or `Concentrator B238300208DS`), that serial fills the piece's **SN** field automatically. It shows under the device and in Edit Record. A serial someone already typed is never overwritten. Accessories and model words (bags, cables, `Ped Reg Drive MAX4LPM`) don't get one.
- **Memory Download stays with its device.** The daily marks it on the device's own line (`Exchange - Memory Download -- Oximeter MBH2311043`), so that exact piece is flagged ★ MD. It is counted in the Memory Download pill and printed on the PDF/CSV notes (MDR only).
- The **Memory Download** checkbox in **Edit Record** is for the whole order. If you tick it and no piece is flagged yet, the app flags the device (the oximeter or apnea monitor, otherwise the piece with a serial). Unticking it clears the flag from every piece.

## Load Daily (MDR)

1. Export the Power BI daily report as **`data.csv`**. The file must be named exactly that.
2. Click **Load Daily** and pick it.
3. The app:
   - matches each row to an existing record by **patient plus at least one shared reference** (or the same Entry ID or job #). Patient IDs like `13961` and `13961.00` count as the same.
   - fills in blanks on existing records (job #, pickup date, Entry ID and so on), adds new references and pieces, and updates expected quantities. It **never** removes pieces or overwrites MDR's check-in work.
   - adds new records for anything it can't match.
   - treats **Return** lines (for example `Return - Memory Download -- Oximeter ...`) as **Pickups**, keeping the Memory Download on that device.
   - fills each device's serial number from its name.
   - saves only the new or changed records to SharePoint, and puts a timestamped copy of the CSV in SharePoint Documents under **Daily Uploads**.
4. **Rebookings:** if the same patient and reference come in with a **different** job #, the app keeps them as two separate records and shows a list so you can check them.
5. If SharePoint isn't connected, a red banner says the records were saved locally only. **Don't refresh** until it clears.

## Exporting

Both halves of **Export as** use exactly the rows on screen (current month, range, status and search).

- **CSV** columns: Pickup Date, Patient, Patient ID, Job #, Pieces / Items, Reference, Notes, Job Closed, No Pickup, Verified. The last three are Yes/No. The Notes text matches the PDF: No Pickup, ★ Memory Download (MDR only), SGI notes, quantity differences and MDR notes. The file is named `MDR_Exchange_Report_YYYY-MM-DD.csv` and opens cleanly in Excel.
- **PDF** opens the print dialog with a one-table report. Rows are colored by type, and overdue rows are marked. Choose **Save as PDF** to keep a file.

## Dates and time zone

All "today" dates use **Central (Alabama) time**, whatever the PC's clock is set to. That covers Date Returned, the overdue check, export file names and the Daily Uploads archive name. Times shown in the App Log, the activity history and the PDF header are in Central time too. Audit timestamps are stored in UTC, which is normal, and converted for display.

## How saving works (for troubleshooting)

- Every change saves to this browser first, then to SharePoint.
- Saves for the same record always go out **one at a time, in order**: single edits, Load Daily batches and refresh retries all use the same queue. An older save can't undo a newer one, and a new record is created on SharePoint only once.
- A record whose SharePoint save hasn't been confirmed is kept on **Refresh** and retried automatically.
- SharePoint throttling (429/503) is retried with back-off. The sync light says "Rate-limited, retrying".
- A record deleted locally but not yet on SharePoint is retried until it succeeds, so it doesn't come back on refresh.

## SharePoint lists used

| List | Used for |
|---|---|
| ExchangeRecords | The records (one item per job). |
| AppAudit | The App Log entries. |
| Presence list | Who's online (heartbeat every 4 minutes). |
| Documents > Daily Uploads | Archived copies of each `data.csv` loaded. |

## Deploying an update

1. Edit `index.html`, bump `BUILD`, and add a `CHANGELOG` entry if users will notice the change.
2. Commit and push **only** `index.html` (and this README) to `antwainjordan-mdr/mdr-exchange-tracker`. Never commit any `.csv`.
3. After GitHub Pages updates (a minute or two), open the live site, press Ctrl+F5, and check the console shows the new build.

## Later (not urgent)

Sign-in uses the older "implicit" flow. It works, but Microsoft now recommends MSAL with the authorization code flow (PKCE) for single-page apps:
1. In Azure (app registration), add the live address as a **Single-page application** redirect URI.
2. Swap the hand-written sign-in code for Microsoft's MSAL browser library.
3. That removes the hidden-iframe token refresh, which can break when a browser blocks third-party cookies.

## Changes in `2026-10-14-export-split-return-tz`

- New **Export as** split button (CSV and PDF) for MDR and SGI. It replaces the separate Export PDF and Export CSV buttons.
- Daily loads keep **Return** pieces as **Pickups**. They used to be dropped.
- **Serial numbers follow their device:** filled from the device name into the SN field and shown under the device.
- **Memory Download stays on its device:** shown as ★ MD on that piece. The Edit Record checkbox now flags the oximeter or apnea monitor, not just the first piece.
- **Central time** everywhere: Date Returned, overdue and file names no longer jump to tomorrow after 7 PM.
- **Save queue for every save**, including batch and retry saves. That fixes possible duplicate SharePoint rows and older-save-wins races.
- A **saved copy opened from a drive** shows a locked screen instead of running without sign-in, and clears any records it had cached.
