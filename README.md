# MDR Exchange Tracker

**Build:** `2026-11-06-qol-batch` (November 6, 2026)
**Owner:** Antwain Jordan, Medical Resources Inc. (MDR)
**Live site:** https://antwainjordan-mdr.github.io/mdr-exchange-tracker/

A shared tracker for equipment exchanges and pickups between MDR and SGI Logistics. Each row is one job: the patient, the pieces SGI should exchange or pick up, and the check-in, closing and confirming steps. Records are stored in SharePoint (site **MDRExchangeTracker**, list **ExchangeRecords**). Everyone sees the same data.

## Files

| File | Purpose |
|---|---|
| `index.html` | The whole app (single file). This is what GitHub Pages serves. |
| `index-offline.html` | A standalone offline/demo copy — no Microsoft sign-in, no SharePoint. See "Offline demo build" below. |
| `offline-sample-csv/data.csv`, `offline-sample-csv/Confirmed___Void_data.csv` | Anonymized sample data for the offline build — real job/date/piece data with every patient name swapped for a fake one. |
| `README.md` | This file. |

`data.csv` (the daily Power BI export) is **not** part of the app and must never be committed to the repo. You pick it from your PC with **Load Daily**.

**Version marker:** the build name is `const BUILD = '...'` near the top of the script. It prints in the browser console on load (`[MDR Exchange Tracker] build ...`). Bump it on every deploy, and add a `CHANGELOG` entry when the change is something users will see.

---

## Signing in

- Open the live site and click **Sign in with Microsoft**. Use your Microsoft 365 account.
  - `@mdr.net` accounts get the **MDR** view.
  - SGI accounts get the **SGI** view, branded "SGI Logistics Portal".
  - Any other domain is refused.
- You stay signed in all day. Sign-in uses the Microsoft authorization code flow with PKCE, and the app keeps a refresh token. The access token is renewed quietly about 5 minutes before it expires (a plain background request, no hidden iframe). If the refresh is refused (for example after about a day, or if the account was changed), a "Session expired" screen asks you to sign in again.
- Click your email (the chip in the header) to **sign out**. Signing out also clears the records cached in that browser.
- **Opening a saved copy from a drive** (for example `Y:\MDR Apps\Exchange Tracker\index.html`) shows a locked screen with an **Open live site** button. Sign-in only works on the live site.
- After a fresh sign-in, a **What's New** box shows the latest update. **Updates** in the header shows the full history.

## Offline demo build

`index-offline.html` is a separate copy of the app for testing or demoing without Microsoft sign-in or SharePoint. Everything else — the table, filters, stats, CSV import/export, PDF export — works exactly the same as the live site.

- **Open it directly in a browser** — just double-click the file, no server or sign-in needed.
- Instead of "Sign in with Microsoft," you'll see a small **Continue as MDR / Continue as SGI** screen. Pick whichever side you want to test.
- Load the sample data with the app's own buttons, same as always: **Load Daily** → `offline-sample-csv/data.csv`, **Load Confirmed/Void** → `offline-sample-csv/Confirmed___Void_data.csv`.
  - Those two CSVs are your real daily/Confirmed-Void data with **every patient name replaced by a fake one** (job numbers, dates, pieces, and statuses are untouched). Safe to hand to someone else for a demo.
- Nothing in offline mode touches SharePoint. Whatever you load or change stays only in that browser's local storage — close the tab or clear your browser data and it's gone.
- Switching roles (MDR/SGI) means reloading the page — there's no sign-out button, just refresh and pick again.
- `data.csv` now also has **6 synthetic rows at the bottom** simulating SGI's manual Appsmith entries (a patient returning something that wasn't picked up last order, or a malfunctioning unit) — blank Job #, Entry ID, and Pickup Date, just like the real ones. Two reuse an existing sample patient (Davis, Fernando / Lee, Veronica) to test the dateless-return-for-an-existing-patient case; four are brand-new patients for the standalone case.
  - **Tested result:** these load without errors and sit under Pending (no Late By Date, since that's calculated from Pickup Date and there isn't one). The two "reused patient" rows did **not** merge into that patient's existing record — they came in as a second, separate record — because the matching logic requires a shared reference/Job#/Entry ID on top of the name match, and these manual rows have none. Worth knowing if you're trying to track down why a dateless SGI entry shows up as a duplicate instead of combining with the original order.

## The header buttons

| Button | Who | What it does |
|---|---|---|
| SGI View / MDR View | MDR | Preview what the other side sees. Testing only, changes nothing. |
| Light / Dark Mode | All | Switches the theme. Remembered per browser. |
| Flash: On / Off | All | Turns the red pulsing on overdue rows on or off. |
| Refresh | All | Reloads every record from SharePoint. |
| **CSV** (green) | All | Downloads the rows **on screen** as a CSV (see Exporting). |
| **PDF** (red) | All | Prints or saves the rows on screen as a PDF report. MDR also gets a brief "SGI approximate arrival" notice when clicking either export button, logged to the App Log. |
| Load Daily | MDR | Imports today's `data.csv` from Power BI (see Load Daily). Refuses the file if it contains anything other than "New" rows. |
| Load Confirmed/Void | MDR | Imports Brightree's `Confirmed & Void_data.csv` and resolves matching jobs (see Load Confirmed/Void). |
| **✓ Completed** | MDR | Opens the Completed tab — a separate, read-only comparison of Confirmed/Void jobs (see Completed tab below). |
| Add Record | All | Adds a job by hand. |
| Clear Cache | All | Wipes this browser's saved data and tokens and reloads. Fixes most odd behavior. You sign in again after. |
| App Log | MDR | Who did what, and when (sign-ins, loads, exports, refreshes). |
| Updates | All | Full update history. |

Next to the buttons, the **presence bar** shows who else has the tracker open (seen in the last 10 minutes). The **sync light** shows Saved, Syncing, Synced or an error.

Next to the Exchange/Pickup/Mixed/All Done legend, **Last Updated** shows the most recent time the tracker was refreshed by either Load Daily or Load Confirmed/Void.

**MDR only:** on every page load, a reminder pops up not to print the report before SGI's data is back in. Close it with the X or by clicking outside it — that's logged to the App Log, and it doesn't block anything else from loading. (This reminder does not show for antwainjordan@mdr.net.)

## Filters

- **Month chips:** All, plus the current month and the two before it. **Custom** opens start and end date pickers. Months and ranges filter on **Pickup Date**. On a fresh load, the filter now defaults to **Custom: 30 days ago through 7 days from today** instead of All.
- **Search:** patient name, patient ID, job #, reference numbers, piece names and serial numbers. It searches every matching record, not only the rows already loaded on screen.
- **Status:**

| Status | MDR view | SGI view |
|---|---|---|
| Incomplete | Not Job Closed, and the Late By Date has already passed — **or** Partial and not yet Confirmed (see below). | Same. |
| Pending | **Every still-open job, including late/Incomplete ones** — so MDR warehouse staff watching Pending (or 1:1 Match) never miss something SGI physically returned without updating the system. Use **Hide Late & Incomplete** to actually hide the late ones from view when wanted. | Job Closed but not yet Confirmed, **or** still open but the Late By Date hasn't passed yet — **or** Partial and not yet Confirmed. |
| Partial | Multi-piece job with at least one piece checked but not all of them — Job Closed or not. A Partial, not-yet-Confirmed job shows under **both** Pending and Incomplete at once (deliberately — so it can't be missed under either filter), regardless of Job Closed or the Late By Date, until every piece is explicitly checked or the job is Confirmed. Leaving a piece unchecked is never treated as "not picked up" — that's a separate, explicit checkbox (No Pickup). | Same. |
| Processing | ⚠️ Deprecated — logic removed. Chip and dropdown option stay visible but always show/match 0 records until it's rebuilt. | (not shown) |
| Complete | Confirmed (set by Load Confirmed/Void, either a Confirmed or a Void row). | Same. |
| No Pick up | No Pickup is checked, regardless of other status. | Same. |
| All Active | Everything **except** Confirmed/Void jobs — those are sealed onto the separate **✓ Completed** tab instead (see below). | Everything. |

**Every stat badge now counts only what's actually in the table right now** — including the Status dropdown and whichever chip filter is active, not just search/Hide-toggle/date-range. Selecting a Status (e.g. Complete) drops every other badge to match what's really on screen for that selection. **1:1 Match** and **Memory Download** both exclude jobs already pulled off the Confirmed/Void CSV.

- **Hide Late & Incomplete** (button next to Status, was "Hide Incomplete"): hides whatever rows the table is currently flashing red for — late by date passed, not complete, not closed — on top of whatever else is filtered. Click again to bring them back. View-only, changes no data, and resets on reload.
- **Hide Returned Jobs** (button next to Hide Late & Incomplete, SGI view only): hides any job already Job Closed — i.e. already returned to MDR — so SGI can see what still needs attention instead of scrolling past jobs just waiting on MDR to confirm. Click again to bring them back. View-only, never affects MDR's view.
- The **counter pills** (Incomplete, Pending, Partial, Processing, Complete, No Pick up, 1:1 Match, Memory Download) now only count the records in the current search/Hide-toggle/date-range scope, not every record in the tracker. Click **Partial**, **1:1 Match**, or **Memory Download** to filter the table down to just those records, sorted by Pickup Date — click again to flip the sort direction, a third click clears the filter. **1:1 Match** means every piece on the job came back checked at exactly the expected quantity (no shortage, no overage).
- **Partial, Shortage and Overage** chips on each row only describe how items came back; they never move a record between statuses. (The Partial *Status filter and counter* above is a separate, stricter rule.)
- The table loads 50 rows at a time and loads more as you scroll. Click a column header with ⇅ to sort.
- **Top banner (SGI only):** a static, always-visible banner across the very top of the page reports whether Late & Incomplete and/or Returned Jobs are currently hidden or shown, for the active date range. It cannot be hidden. **MDR** sees its own separate banner tied to the same Hide/Show Late & Incomplete button, about SGI's incomplete PUEXs — informational text only.

## Reading a row

| Column | Notes |
|---|---|
| # (Job #) | A plain row number — the first visible row is always 1. Changes as filters/sorting change which rows are visible. |
| Pickup Date | Click to set or change. |
| Patient | Name and patient ID. **NEW** marks rows from a Daily load. 🔒 means the value came from the source file and is locked. |
| Job # | SGI job number. MDR: it links to SGI RapidShip tracking. |
| Pieces / Items | Every piece, with **EXC** (Exchange) or **PKP** (Pickup). A device's serial number shows on its own line under it (`SN: ...`), and ★ MD marks the device that needs a Memory Download (MDR only). |
| Reference | SO and PuEx numbers. MDR: 6-digit numbers starting with 1 open the Brightree Sales Order, and 5-digit numbers starting with 3 open the PuEx. SGI sees plain text. |
| Date Returned | Set automatically when the job is closed, or set by hand. |
| Job Closed (SGI) | Checking it stamps Date Returned with today's date. |
| No Pickup | Marks the job as nothing collected: every piece is checked with qty 0, and Date Returned and Job Closed are stamped. Shows an X. |
| Verified (MDR RCV) | MDR confirms the items were received. SGI can see it but can't change it. |
| Closed (Brightree) | MDR's final sign-off (same field that used to show as "Confirmed (Portal)" — renamed, same behavior). It moves the record to Complete for both sides. Checked by hand, or automatically by **Load Confirmed/Void**. |
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
6. **Wrong-file guard:** if `data.csv` contains any row whose status isn't "New" (because Brightree has already resolved it), the import is refused with a warning instead of silently loading an already-resolved job back into the queue. Download the correct report and try again.

## Load Confirmed/Void (MDR)

Brightree's second daily export — jobs it has already resolved — lists each one as **Confirmed** or **Void**. Loading it is what actually gets a resolved job off of SGI's Pending queue and print.

1. Export the Confirmed & Void report from Brightree.
2. Click **Load Confirmed/Void** and pick it. The filename just needs to contain "Confirmed" and "Void" and end in `.csv` — small naming differences (spacing, a re-download's `(1)`, etc.) are fine.
3. The app:
   - matches each row to **every** existing record it fits (patient plus at least one shared reference, Entry ID, or job #) — not just the first one found. If a patient/job has a pre-existing duplicate record (for example a stray hand-added row sitting alongside the real Daily-sourced one), both get resolved, so neither one is left stuck in Pending.
   - on each match, fills in **Closed (Brightree)** (and **Verified** for a Confirmed row, or the full **No Pickup** behavior — every piece checked, qty 0, Date Returned stamped — for a Void row), but **only for fields that aren't already set**. Nothing you or SGI already marked by hand gets overwritten or undone, and piece quantities SGI already checked in are left exactly as they are.
   - creates a new record, already resolved, for any row that doesn't match an existing one.
   - a row that matches a patient but carries a conflicting, different job # is kept separate (same rebooking-style safety as Load Daily) and listed in the console.
   - if a row matched more than one existing record, it's listed in the console (and a toast warns you) — worth a look to clean up the duplicate, even though both copies are now resolved.
4. Once a job is marked Closed (Brightree), it moves to Complete for both MDR and SGI and drops out of the default Pending view and print.

## Completed tab (MDR)

Click **✓ Completed** in the toolbar to open a separate, read-only view of every Confirmed/Void job — this replaces the old "Hide Completed PUEXs" toggle. "All Active" no longer has a way to bring these back into the main table; this tab is where they live now. Click **← Back to Active** to return.

**Columns:** Pickup Date, Patient, Job #, **SGI**, **MDR Confirmed/Void**, **Actions**, Reference, Total Qty Confirmed/Voided, Match. Everything on this page is plain text — no checkboxes, no editable fields, no Select All; these records are sealed.

- **SGI column** — requested quantity (from the Daily CSV) vs. what SGI actually entered at pickup. Always 2 quantities, never 3. A clean match: ✅ (confirmed) EXC/PKP 🔒 (from source) `[item] x3 🏷️ x3` (exact match). SGI returning less or more than requested is informational only, not an issue: 🔺 (less) / 🔻 (more). A piece SGI added by hand (not from the Daily CSV) always reads ✍🏼 on both sides. A piece SGI never checked in at all reads 🛑 (blank, never verified) and shows up in the Actions column asking to verify with SGI. If every piece on the job is zero or No Pickup is checked, the whole row reads 🚫 instead.
- **MDR Confirmed/Void column** — the same pieces compared against what that Confirmed/Void CSV row itself reported. 🏷️ exact match, ➕ shortage (Brightree confirmed less than requested — needs an inventory correction), ➖ overage (confirmed more — no action needed), 🚫 voided/all-zero, 🛑 "was removed from order" (a piece MDR requested that Brightree's CSV doesn't mention at all), ⁉️ a piece Brightree reported that was never requested (added at confirm time — worth checking for a PURL/PUEX). If SGI's own entered amount disagrees with what Brightree confirmed, a 3rd quantity and an alert icon are added: 🚚 (Brightree confirmed less than SGI said) or ⚠️ (more), plus 🚨 to flag it for investigation.
- **Actions column** spells out, in plain English, what (if anything) needs follow-up for each piece on the row — shortages, voids, manually-added pieces, and SGI/Brightree mismatches each get their own line. A row with nothing to fix reads **🏷️ Exact Match**.
- **Duplicates:** if two (or more) SharePoint records exist for the same patient + Job #, a small **⧉ N records · Merge** button appears next to the Job #, here and on the Active view. Merging combines notes, pieces, and reference numbers onto one record and **permanently deletes** the other one(s) from SharePoint — there's no undo in the app itself (your SharePoint site's recycle bin may still catch it for a while, depending on your retention settings).
- **Total Qty Confirmed/Voided** sums the MDR column.
- **Match:** a record matched normally (patient plus a shared reference, Entry ID, or job #) shows **1:1 Match**. A Confirmed/Void CSV row that didn't match anything existing — so it has no real SGI side to compare against — shows **No Match** in red, with the specific reason(s) underneath: No matching Patient ID, No matching Job #, and/or No matching PUEX/reference.
- **SharePoint setup required:** this tab's MDR column, Actions column, and No Match flag depend on 2 SharePoint columns — `ResolvedPieces` and `MatchReasons` — on the ExchangeRecords list. See "SharePoint lists used" below. Without them, everything still works in the browser you're using, but it goes blank for anyone else (or after a refresh) because there's nowhere to save it.

## Exporting

Both the CSV and PDF buttons use exactly the rows on screen (current month, range, status and search).

- **CSV** columns: #, Pickup Date, Patient, Patient ID, Job #, Pieces / Items, Reference, Notes, Job Closed, No Pickup, Verified. The last three are Yes/No. The Notes text matches the PDF: No Pickup, ★ Memory Download (MDR only), SGI notes, quantity differences and MDR notes. The file is named `MDR_Exchange_Report_YYYY-MM-DD.csv` and opens cleanly in Excel.
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
| ExchangeRecords | The records (one item per job). Needs 2 columns the Completed tab depends on — see below. |
| AppAudit | The App Log entries. |
| Presence list | Who's online (heartbeat every 4 minutes). |
| Documents > Daily Uploads | Archived copies of each `data.csv` loaded. |

### Adding the Completed tab's 2 columns (one-time setup)

The Completed tab's MDR column, Actions column, and No Match flag are saved to SharePoint through 2 columns on **ExchangeRecords** — `ResolvedPieces` and `MatchReasons`. Add them once, the same way for both:

- Open the ExchangeRecords list, click **+ Add column**, choose **Multiple lines of text**
- Name it `ResolvedPieces`, choose **Plain text**, and save
- Repeat for a second column named `MatchReasons`, Plain text, save

There's no "unlimited length" toggle to look for — that setting only exists for document library columns. A list's Multiple lines of text column already holds roughly 64,000 characters by default, same as the existing `Pieces` column. After both columns exist, open the tracker and run **Load Confirmed/Void** once to backfill every record whose CSV row still matches. A record confirmed before these columns existed, or whose original Confirmed/Void CSV export is no longer available, stays blank on the left — there's no way around that one.

## Deploying an update

1. Edit `index.html`, bump `BUILD`, and add a `CHANGELOG` entry if users will notice the change.
2. Commit and push **only** `index.html` (and this README) to `antwainjordan-mdr/mdr-exchange-tracker`. Never commit any `.csv`.
3. After GitHub Pages updates (a minute or two), open the live site, press Ctrl+F5, and check the console shows the new build.

## Azure setup for sign-in

- App registration redirect URI must be listed under the **Single-page application** platform (it already is): `https://antwainjordan-mdr.github.io/mdr-exchange-tracker/`
- Delegated permissions needed: `User.Read`, `Sites.ReadWrite.All` (admin consent already granted, including for SGI). `offline_access` is requested at sign-in and needs no extra consent step.
- **Implicit grant** (access tokens / ID tokens) is no longer used by the app. Leave it enabled until MDR and an SGI guest account have both signed in on this build, then uncheck both boxes under Authentication.

## Changes in `2026-11-06-qol-batch`

- **Copy buttons** (live site only, not in the offline demo): hover over Patient ID, patient name, Job #, or any reference/SO number — in both MDR and SGI views — to see a small copy icon. Click it to copy that value, with a brief "Copied" confirmation. Locked fields are click-to-copy too, since clicking them no longer opens the record.
- **Typeable date fields**: Pickup Date, Date Returned, and the Custom date range's Start/End fields are now text boxes you can type or paste into — accepts `M/D/YYYY`, `MM/DD/YY`, or `YYYY-MM-DD`, always displays as `MM/DD/YYYY`. The calendar icon is still there to pick a date the old way. Typing something that isn't a valid date reverts the field to its last good value on blur instead of saving. Calculated fields like Late By Date only recheck once you click/tab out of the field, never while you're still typing.
- **Reference number spacing**: reference numbers now display with a space after each comma (`126244, 35645` instead of `126244,35645`) everywhere they're shown as a single line, including CSV exports. The stored data, and Load Daily/Confirmed-Void matching, are unchanged. Search now also ignores spaces and commas in reference numbers, so searching `126244 35645` or `126244, 35645` still finds a record stored as `126244,35645`.
- **Collapsible Notes columns**: SGI Notes and MDR Notes can now be collapsed independently, or both at once, using the ◀ / ⬤ / ▶ buttons in the table header (left closes SGI, right closes MDR, center toggles both). Both always start expanded on page load/reload — the collapsed state is never remembered between sessions. Exports always include full note text regardless of what's collapsed on screen.
- **Zoom lag investigated and fixed**: browser zoom (Ctrl +/- or pinch) fires a burst of resize events while it animates, and the app was recalculating the sticky table header's position on every single one of those events instead of once per zoom gesture — the fix coalesces that burst down to once per actual animation frame. If zooming still feels slow on your machine after this, let me know and I'll keep digging.
- **App version label**: a small version label now sits next to the logo/app title (top-left), showing the current build date at a glance.
- **"Last Updated" moved**: the "Last Updated" timestamp now sits next to the SGI/MDR view toggle button, in larger text, instead of being tucked into the legend row below the table.
- **Oximeter quantity bug — investigated, not yet resolved**: a report that a record showed an expected oximeter quantity of 3 when the PUEX stated 1. Traced the daily-import parsing, the merge/dedup logic, and the Confirmed/Void round-trip and found no code path that sums piece quantities together — so the cause is still unconfirmed. If you can send a sample record (or the specific daily/Confirmed-Void CSV row) that shows the bad quantity, that'll make it much easier to pin down.

## Changes in `2026-11-06-completed-tab-icons`

- Fixed: the Completed tab's MDR column and Total Qty went blank (and every row read "1:1 Match") after any reload. `ResolvedPieces` and `MatchReasons` were only ever saved to the browser's own cache, never to SharePoint — they now round-trip through 2 new SharePoint columns. See "Adding the Completed tab's 2 columns" above for the one-time setup.
- Completed tab redesigned: the two plain-text piece columns are now icon-based and clearly labeled SGI vs. MDR Confirmed/Void, plus a new Actions column spelling out what (if anything) needs follow-up. A clean row reads "🏷️ Exact Match". See "Completed tab (MDR)" above for the full icon legend.
- Duplicate SharePoint records (same patient + Job #) are now flagged with a "⧉ N records · Merge" button, on both the Completed tab and the Active view. Merging combines notes, pieces, and reference numbers onto one record and permanently deletes the other(s) from SharePoint.

## Changes in `2026-11-04-completed-tab`

- MDR: new **✓ Completed** tab replaces the old "Hide Completed PUEXs" toggle — a separate, read-only view of every Confirmed/Void record. Left "Pieces / Items" column is what SGI originally selected, right is what the Confirmed/Void CSV itself reported (0 for Void, the confirmed qty for Confirmed), plus a Total Qty Confirmed/Voided column.
- "All Active" now always excludes Confirmed/Void jobs — no toggle needed, they live on the Completed tab instead.
- A Confirmed/Void CSV row with no matching Patient ID, Job #, or reference now shows **No Match** on the Completed tab (same red styling as Late & Incomplete), with the specific reason(s) listed underneath.
- Fixed: importing a Void status used to overwrite the record's own piece quantities with 0, permanently losing what SGI had originally entered on the next re-import. That data is now preserved untouched — it's the left column on the Completed tab.

## Changes in `2026-11-03-pkce-sign-in`

- Sign-in rebuilt on the authorization code flow with PKCE (plain `fetch`, no library). Tokens no longer travel in the URL.
- Token renewal now uses a refresh token instead of a hidden iframe, so browsers that block third-party cookies no longer break it.
- Same "Session expired" fallback, same roles and domains, no change to how the app looks.
- `index-offline.html` is not affected (it has no Microsoft sign-in).

## Changes in `2026-11-03-sticky-header-fix`

Fixes the sticky header added in `2026-11-03-mdr-code-review-fixes` just below — it was pinning a few rows down into the table instead of staying at the top of the screen. Root cause: the table's horizontal-scroll wrapper (`.twrap`) was silently also capturing vertical scroll (setting only `overflow-x` makes the browser compute `overflow-y` the same way), so the header was anchoring to that wrapper's own position instead of the page. The table area now has its own bounded scroll pane with the header genuinely pinned to its top, both directions, with columns guaranteed to line up with the data since the header and rows share the exact same table. Full writeup in section 5 of the Code Review Findings doc.

## Changes in `2026-11-03-mdr-code-review-fixes`

From a full code review pass (dead code, confusing/overlapping logic, cosmetic consistency). Full findings and your answers are in the "MDR Exchange Tracker — Code Review Findings" doc.

- Table column headers now stay stuck to the top of the screen while scrolling, instead of disappearing.
- MDR: "All Active" now excludes Confirmed/Void jobs by default — new **Show Completed PUEXs** button brings them back; **Hide Completed PUEXs** is the MDR-side equivalent of SGI's Hide Returned Jobs.
- A single-piece job not yet checked in now matches the "Partial" Status filter and Pending+Incomplete dual-bucket rule, same as its own row chip already did.
- The quick "Add Items" popup now logs to the App Log, and its piece picker now works the same as the full Add/Edit Record screen (quantity box, can re-add the same item).
- Manually adding a piece now fills in the received quantity with what you typed instead of leaving it blank.
- Removed 5 unused CSS rules — no visible effect.

Held back for separate scoping (real new features, not fixes): the "🔕None" button click/double-click spec, and the SGI-vs-Brightree side-by-side comparison tab. The duplicate-merge matching question is still open, per your note to circle back.

## New: offline demo build (November 3, 2026)

- Added `index-offline.html` — a standalone copy with no Microsoft sign-in or SharePoint dependency, for local testing/demo use. See "Offline demo build" above.
- Added anonymized sample CSVs (`offline-sample-csv/`) — real job data with every patient name swapped for a fake one, for use with the offline build.

## Changes in `2026-11-03-mdr-pending-counts-match-view`

- **MDR**: every still-open job (including late/Incomplete ones) now shows under Pending too, so MDR warehouse staff watching Pending or 1:1 Match never miss something SGI physically returned without updating the system. "Hide Late & Incomplete" still hides those from view when wanted; Incomplete is still there as the narrower, late-only lens.
- **1:1 Match** now excludes jobs already pulled off the Confirmed/Void CSV.
- **Memory Download** now excludes jobs already pulled off the Confirmed/Void CSV too (fixes it counting one extra that was already resolved).
- Every stat badge now counts only what's actually in the table right now, including the Status dropdown and chip filter — not just search/Hide-toggle/date-range.

## Changes in `2026-11-03-partial-dual-bucket`

- A Partial, not-yet-Confirmed job now shows under **both** the Pending and Incomplete filters/counters at once, regardless of Job Closed or the Late By Date — so it can't be missed under either view until every piece is explicitly checked or the job is Confirmed.
- Leaving a piece unchecked is still just "not yet verified" — never treated as "not picked up." That stays its own explicit checkbox (No Pickup).

## Changes in `2026-11-03-status-bucket-rebuild`

- Rebuilt how Complete/Pending/Incomplete are decided, for both MDR and SGI: Complete is Confirmed; Job Closed but not yet Confirmed is Pending; everything still open splits on the Late By Date (not yet due = Pending, past due = Incomplete).
- **Incomplete is no longer MDR-only** — SGI sees it too now, with the same rule.
- Partial and No Pick up stay as separate, overlapping counters/filters — a record can be Partial AND Incomplete (or Partial AND Pending) at once, without being double-counted.
- **Processing** (MDR) is deprecated — logic removed, chip and dropdown option stay visible (marked ⚠️) but always show/match 0 records until it's rebuilt.

## Changes in `2026-11-03-static-banner-11match`

- The SGI top banner is now a static strip (no scrolling) with twice-as-large text.
- The MDR "don't print before SGI returns" reminder no longer shows for antwainjordan@mdr.net.
- New **1:1 Match** stat chip (both sides): jobs where every piece came back checked at exactly the expected quantity. Click to filter + sort by Pickup Date, same behavior as Partial and Memory Download.

## Changes in `2026-11-03-marquee-partial-nopickup`

- New **Last Updated** timestamp next to the Exchange/Pickup/Mixed/All Done legend — the most recent of Load Daily or Load Confirmed/Void.
- The Month/Custom date filter now defaults to **Custom: 30 days ago through 7 days from today** on a fresh load, instead of All.
- New always-on scrolling banner across the very top of the page, **SGI only**: reports whether Late & Incomplete and/or Returned Jobs are hidden or shown, for the active date range. Cannot be hidden.
- The **Pending/Partial/Complete/No Pick up** counters now reflect only the records in the current search/Hide-toggle/date-range scope, not every record.
- New **Partial** Status dropdown option (both sides): a multi-piece job with at least one but not all pieces checked, Job Closed or not. The **Partial** stat chip is now clickable — filters the table to Partial, sorted by Pickup Date; click again to flip the sort, a third click clears it. The **Memory Download** chip got the same behavior.
- New **No Pick up** status, counter, and Status dropdown option for any record with No Pickup checked.
- Added a row number to the Job # column — table, CSV export, and PDF export. The first visible row is always 1.
- Removed the "Export as" label above the CSV/PDF buttons.
- **MDR**: a reminder now shows on every page load not to print the report before SGI's data is back in, until closed (X or outside click) — logged to the App Log. Clicking CSV or PDF also shows a brief "SGI approximate arrival" notice, logged to the App Log only.
- **"Hide Incomplete"** is now **"Hide Late & Incomplete"** / **"Show Late & Incomplete"** (same button, both sides). MDR also now sees its own 2-state banner about SGI's incomplete PUEXs tied to that same button — informational text only, no Teams integration is actually wired up.

## Changes in `2026-10-30-late-by-calculated`

- Late by Date is now calculated by the tracker (Pickup Date + 3 business days, weekends never count) instead of coming from the report. No Pickup Date means no Late By Date.
- No longer editable by anyone, MDR or SGI — it's a plain calculated value in the table and the Add/Edit form.
- One-time cleanup: every existing record's Late By Date gets recalculated from its Pickup Date on the next load.

## Changes in `2026-10-30-reference-slash-cleanup`

- One-time cleanup: existing records with a leftover "/" in the Reference field now get it converted to a comma automatically the next time the app loads.

## Changes in `2026-10-30-reference-slash-normalize`

- Reference field: a "/" typed between reference numbers (only happens when SGI manually adds a record) is now converted to a comma on save.

## Changes in `2026-10-30-confirmed-void-btn-fix`

- Fixed **Load Confirmed/Void** showing as an oversized button with a stray "Choose File / No file chosen" control inside it — the file input just wasn't hidden like the other upload buttons. Icon changed to ⛳ to match **Load Daily**'s style.

## Changes in `2026-10-30-hide-returned-jobs-toolbar`

- Moved **Hide Returned Jobs** (SGI view only) out of the Job Closed column header and onto the toolbar row next to Hide Incomplete — it was getting squeezed/clipped in the old spot, the likely reason some SGI users couldn't see it.

## Changes in `2026-10-30-hide-returned-jobs-toggle`

- New **Hide Returned Jobs** button above the Job Closed column, SGI view only: hides jobs already Job Closed (already returned to MDR) so SGI can focus on what's still outstanding. Click again to bring them back. View toggle only — doesn't change any data, never affects the MDR view.

## Changes in `2026-10-30-hide-incomplete-toggle`

- New **Hide Incomplete** button next to the Status filter: hides whatever rows the table is currently flashing red for (late by date passed, not complete, not closed). Click again to bring them back. View toggle only — doesn't change any data, and resets on reload.

## Changes in `2026-10-30-confirmed-void-dupe-match-fix`

- Fixed: **Load Confirmed/Void** only updated the first matching record it found for a job. Where a pre-existing duplicate record already existed, that left whichever duplicate it didn't pick stuck in Pending. It now resolves every matching record for a job, and warns (console + toast) when a patient had duplicates so you know which ones are worth cleaning up.

## Changes in `2026-10-30-confirmed-void-filename-fix`

- Fixed: **Load Confirmed/Void** was rejecting the real exported file because its name had to match one exact string. It now just checks the filename contains "Confirmed" and "Void" and ends in `.csv`.

## Changes in `2026-10-30-confirmed-void-import`

- New **Load Confirmed/Void** button (MDR only): imports Brightree's `Confirmed & Void_data.csv` and matches it onto existing records the same way Load Daily does.
- **Confirmed (Portal)** is now called **Closed (Brightree)** everywhere it appears (header, legend, audit log). Same field, same behavior — just renamed to match where it's now set from.
- Loading Confirmed/Void auto-fills Closed (Brightree) on a match (plus Verified for a Confirmed row, or the full No Pickup behavior for a Void row) — only for fields not already set, so nothing already marked by hand is overwritten. A row with no match gets a new record created, already resolved.
- **Load Daily now refuses to import** `data.csv` if it contains anything other than "New" rows, with a warning telling you to download the correct report instead.

## Changes in `2026-10-14-export-split-return-tz`

- New **Export as** split button (CSV and PDF) for MDR and SGI. It replaces the separate Export PDF and Export CSV buttons.
- Daily loads keep **Return** pieces as **Pickups**. They used to be dropped.
- **Serial numbers follow their device:** filled from the device name into the SN field and shown under the device.
- **Memory Download stays on its device:** shown as ★ MD on that piece. The Edit Record checkbox now flags the oximeter or apnea monitor, not just the first piece.
- **Central time** everywhere: Date Returned, overdue and file names no longer jump to tomorrow after 7 PM.
- **Save queue for every save**, including batch and retry saves. That fixes possible duplicate SharePoint rows and older-save-wins races.
- A **saved copy opened from a drive** shows a locked screen instead of running without sign-in, and clears any records it had cached.
