# MDR Exchange Tracker

**Version:** `1.4.18` (November 12, 2026)
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

`data.csv` (the Power BI export) is **not** part of the app and must never be committed to the repo. You pick it from your PC with **Load CSV**.

**Version marker:** `const VERSION = '...'` near the top of the script, following standard SemVer (`MAJOR.MINOR.PATCH`):
- **MAJOR** - breaks or redefines something existing (a SharePoint schema change, a field that no longer means what it used to, an auth-method swap, a core status rule rebuilt). Resets MINOR and PATCH to 0.
- **MINOR** - new, additive capability; nothing old stops working. Resets PATCH to 0.
- **PATCH** - a pure bug fix, no new behavior.

It prints in the browser console on load (`[MDR Exchange Tracker] v1.0.0 (2026-11-08)`), alongside `const BUILD_DATE = '...'` for the calendar date of that deploy. Bump VERSION on every deploy, and add a `CHANGELOG` entry (with a matching `version` field) when the change is something users will see.

Versioning switched to SemVer starting at `1.0.0` on November 8, 2026. Older `CHANGELOG` entries below keep their original date-slug `build` tags (e.g. `2026-11-06-qol-batch`) as the historical record of what they were actually called at the time - they were not renumbered retroactively.

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
- Load the sample data with the app's own button, same as always: **Load CSV** → either `offline-sample-csv/data.csv` or `offline-sample-csv/Confirmed___Void_data.csv` (or a combined export with both — see "Load CSV (MDR)" below).
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
| **📥 Load CSV** | MDR | Imports a Power BI CSV export — New, Confirmed, and Void rows can all be in the same file, and each row is routed automatically by its own status (see Load CSV). Refuses the file if it's missing any of the expected columns. |
| **✓ Completed** | MDR | Opens the Completed tab — a separate, read-only comparison of Confirmed/Void jobs (see Completed tab below). |
| Add Record | All | Adds a job by hand. Saving on a Job # that already exists no longer silently creates a duplicate — both records are kept, and a review popup opens (and reopens on reload until resolved) comparing the two so you can merge or keep them separate. |
| **⧉ Find Duplicates** | MDR | Scans every record for likely duplicates — same Job # + same patient, or same Job # + same reference # — and lists each pair to Approve (merge) or Reject (dismiss). Nothing merges without you clicking Approve. |
| Clear Cache | All | Wipes this browser's saved data and tokens and reloads. Fixes most odd behavior. You sign in again after. |
| App Log | MDR | Who did what, and when (sign-ins, loads, exports, refreshes). |
| Updates | All | Full update history. |

Next to the buttons, the **presence bar** shows who else has the tracker open (seen in the last 10 minutes). The **sync light** shows Saved, Syncing, Synced or an error.

Next to the Exchange/Pickup/Mixed/All Done legend, **Last Updated** shows the most recent time the tracker was refreshed by **Load CSV**.

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
| Complete | Confirmed (set by Load CSV, either a Confirmed or a Void row). | Same. |
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
| Closed (Brightree) | MDR's final sign-off (same field that used to show as "Confirmed (Portal)" — renamed, same behavior). It moves the record to Complete for both sides. Checked by hand, or automatically by **Load CSV**. |
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

## Load CSV (MDR)

Starting in `1.4.0`, the old **Load Daily** and **Load Confirmed/Void** buttons were replaced by a single **📥 Load CSV** button. Power BI's **"MDR | SGI PUEX Report"** export can include **New**, **Confirmed**, and **Void** rows together in one file — each row is routed to exactly the same matching/merge logic that "Load Daily" and "Load Confirmed/Void" always used, nothing about how a row resolves onto a record changed, only how many files you have to pick.

1. Export the report from Power BI, including whichever of New/Confirmed/Void you want to load — all three at once is fine, so is just one.
2. Click **Load CSV** and pick the file.
3. The app checks the file has the columns the combined report always has (`Patient Name`, `Job #`, `Pieces`, `OtherIDs`, `DropOff By`, `Pickup Date`, `Entry ID`, `Pickup/Exchange Status`). If any are missing, nothing is loaded and you get a message telling you which columns it expected — there's no filename check, so a correctly-headed file loads no matter what it's named.
4. Each row is then split out by its own `Pickup/Exchange Status` and handled exactly as before:
   - **New** rows go through the same logic "Load Daily" always used — matches each row to an existing record by **patient plus at least one shared reference** (or the same Entry ID or job #; patient IDs like `13961` and `13961.00` count as the same), fills in blanks on existing records, adds new references and pieces, updates expected quantities, adds new records for anything it can't match, treats **Return** lines as **Pickups**, and fills each device's serial number from its name. It **never** removes pieces or overwrites MDR's check-in work.
   - **Confirmed** and **Void** rows go through the same logic "Load Confirmed/Void" always used — matches each row to **every** existing record it fits, not just the first one found (so a pre-existing duplicate record doesn't leave one copy stuck in Pending), fills in **Closed (Brightree)** (and **Verified** for Confirmed, or the full **No Pickup** behavior for Void) only for fields that aren't already set, and creates a new record, already resolved, for any row that doesn't match an existing one.
   - **Rebookings/conflicts:** if the same patient and reference come in with a **different** job #, both New and Confirmed/Void rows keep the records separate and list them so you can check them.
   - Any row with a status other than New/Confirmed/Void is skipped and noted in the console.
5. Saves only the new or changed records to SharePoint, and puts a timestamped copy of the CSV in SharePoint Documents under **Daily Uploads**.
6. If SharePoint isn't connected, a red banner says the records were saved locally only. **Don't refresh** until it clears.
7. Once a job is marked Closed (Brightree), it moves to Complete for both MDR and SGI and drops out of the default Pending view and print.

## Completed tab (MDR)

Click **✓ Completed** in the toolbar to open a separate, read-only view of every Confirmed/Void job — this replaces the old "Hide Completed PUEXs" toggle. "All Active" no longer has a way to bring these back into the main table; this tab is where they live now. Click **← Back to Active** to return.

**Columns:** Drop Off Date, Patient, Job #, **SGI**, **MDR Confirmed/Void**, **Actions**, Reference, Total Qty Confirmed/Voided, Match. Everything on this page is plain text — no checkboxes, no editable fields, no Select All; these records are sealed.

- **SGI column** — requested quantity (from the Daily CSV) vs. what SGI actually entered at pickup. Always 2 quantities, never 3. A clean match: ✅ (confirmed) EXC/PKP 🔒 (from source) `[item] x3 🏷️ x3` (exact match). SGI returning less or more than requested is informational only, not an issue: 🔺 (less) / 🔻 (more). A piece SGI added by hand (not from the Daily CSV) always reads ✍🏼 on both sides. A piece SGI never checked in at all reads 🛑 (blank, never verified) and shows up in the Actions column asking to verify with SGI. If every piece on the job is zero or No Pickup is checked, the whole row reads 🚫 instead.
- **MDR Confirmed/Void column** — the same pieces compared against what that Confirmed/Void CSV row itself reported. 🏷️ exact match, ➕ shortage (Brightree confirmed less than requested — needs an inventory correction), ➖ overage (confirmed more — no action needed), 🚫 voided/all-zero, 🛑 "was removed from order" (a piece MDR requested that Brightree's CSV doesn't mention at all), ⁉️ a piece Brightree reported that was never requested (added at confirm time — worth checking for a PURL/PUEX). If SGI's own entered amount disagrees with what Brightree confirmed, a 3rd quantity and an alert icon are added: 🚚 (Brightree confirmed less than SGI said) or ⚠️ (more), plus 🚨 to flag it for investigation.
- **Actions column** spells out, in plain English, what (if anything) needs follow-up for each piece on the row — shortages, voids, manually-added pieces, and SGI/Brightree mismatches each get their own line. A row with nothing to fix reads **🏷️ Exact Match**.
- **Duplicates:** if two (or more) SharePoint records exist for the same patient + Job #, a small **⧉ N records · Merge** button appears next to the Job #, here and on the Active view. Merging combines notes, pieces, and reference numbers onto one record and **permanently deletes** the other one(s) from SharePoint — there's no undo in the app itself (your SharePoint site's recycle bin may still catch it for a while, depending on your retention settings).
- **Total Qty Confirmed/Voided** sums the MDR column.
- **Match:** a record matched normally (patient plus a shared reference, Entry ID, or job #) shows **Fulfilled** — this means the Confirmed/Void CSV row resolved onto an existing record, not that quantities actually matched; a real shortage or overage still shows under Actions regardless of this label. A Confirmed/Void CSV row that didn't match anything existing — so it has no real SGI side to compare against — shows **No Match** in red, with the specific reason(s) underneath: No matching Patient ID, No matching Job #, and/or No matching PUEX/reference.
- **SharePoint setup required:** this tab's MDR column, Actions column, and No Match flag depend on 2 SharePoint columns — `ResolvedPieces` and `MatchReasons` — on the ExchangeRecords list. See "SharePoint lists used" below. Without them, everything still works in the browser you're using, but it goes blank for anyone else (or after a refresh) because there's nowhere to save it.

## Exporting

Both the CSV and PDF buttons use exactly the rows on screen (current month, range, status and search).

- **CSV** columns: #, Pickup Date, Patient, Patient ID, Job #, Pieces / Items, Reference, Notes, Job Closed, No Pickup, Verified. The last three are Yes/No. The Notes text matches the PDF: No Pickup, ★ Memory Download (MDR only), SGI notes, quantity differences and MDR notes. The file is named `MDR_Exchange_Report_YYYY-MM-DD.csv` and opens cleanly in Excel.
- **PDF** opens the print dialog with a one-table report. Rows are colored by type, and overdue rows are marked. Choose **Save as PDF** to keep a file.

## Dates and time zone

All "today" dates use **Central (Alabama) time**, whatever the PC's clock is set to. That covers Date Returned, the overdue check, export file names and the Daily Uploads archive name. Times shown in the App Log, the activity history and the PDF header are in Central time too. Audit timestamps are stored in UTC, which is normal, and converted for display.

## How saving works (for troubleshooting)

- Every change saves to this browser first, then to SharePoint.
- Saves for the same record always go out **one at a time, in order**: single edits, Load CSV batches and refresh retries all use the same queue. An older save can't undo a newer one, and a new record is created on SharePoint only once.
- A record whose SharePoint save hasn't been confirmed is kept on **Refresh** and retried automatically.
- SharePoint throttling (429/503) is retried with back-off. The sync light says "Rate-limited, retrying".
- A record deleted locally but not yet on SharePoint is retried until it succeeds, so it doesn't come back on refresh.

## SharePoint lists used

| List | Used for |
|---|---|
| ExchangeRecords | The records (one item per job). Needs 3 columns the Completed tab depends on — see below. |
| AppAudit | The App Log entries. |
| Presence list | Who's online (heartbeat every 4 minutes). |
| Documents > Daily Uploads | Archived copies of each `data.csv` loaded. |

### Adding the Completed tab's 3 columns (one-time setup)

The Completed tab's MDR column, Actions column, No Match flag, and Pending Review queue are saved to SharePoint through 3 columns on **ExchangeRecords** — `ResolvedPieces`, `MatchReasons`, and `PendingResolved`. Add them once, the same way for all three:

- Open the ExchangeRecords list, click **+ Add column**, choose **Multiple lines of text**
- Name it `ResolvedPieces`, choose **Plain text**, and save
- Repeat for a column named `MatchReasons`, Plain text, save
- Repeat for a third column named `PendingResolved`, Plain text, save

There's no "unlimited length" toggle to look for — that setting only exists for document library columns. A list's Multiple lines of text column already holds roughly 64,000 characters by default, same as the existing `Pieces` column. After all three columns exist, open the tracker and run **Load CSV** once (with a Confirmed/Void export, or a combined one) to backfill every record whose CSV row still matches. A record confirmed before these columns existed, or whose original Confirmed/Void CSV export is no longer available, stays blank on the left — there's no way around that one. `PendingResolved` starts empty for everyone and only fills in going forward (or when you use "Add Pending Correction" to backfill a specific job by hand).

## Deploying an update

1. Edit `index.html`, bump `VERSION` (SemVer - see the note above), and add a `CHANGELOG` entry with a matching `version` field if users will notice the change.
2. Commit and push **only** `index.html` (and this README) to `antwainjordan-mdr/mdr-exchange-tracker`. Never commit any `.csv`.
3. After GitHub Pages updates (a minute or two), open the live site, press Ctrl+F5, and check the console shows the new version.
4. Tag the release in the repo to match (`git tag v1.0.0 && git push --tags`, or a GitHub Release) - this is done outside what's in this file, directly against the GitHub repo.

## Azure setup for sign-in

- App registration redirect URI must be listed under the **Single-page application** platform (it already is): `https://antwainjordan-mdr.github.io/mdr-exchange-tracker/`
- Delegated permissions needed: `User.Read`, `Sites.ReadWrite.All` (admin consent already granted, including for SGI). `offline_access` is requested at sign-in and needs no extra consent step.
- **Implicit grant** (access tokens / ID tokens) is no longer used by the app. Leave it enabled until MDR and an SGI guest account have both signed in on this build, then uncheck both boxes under Authentication.

## Changes in `1.4.18`

- **Merge: Confirmed/Void (MDR) wins.** Once a PUEX is Confirmed or Void in Brightree, a merge can't override that with SGI's side.
  - **Which record is kept:** the Confirmed/Void record first. Then the one with Brightree lines, then the one with a real patient name (not blank or "Unknown"), then the one with the most pieces.
  - **Only the other record is Confirmed/Void:** its Closed (Brightree), Verified, No Pickup, Brightree lines and No Match reasons replace the kept record's.
  - **Both are Confirmed/Void:** the kept record's status stays and every Brightree line from both is kept. The kept record's copy wins only when both have the same item and type, because they describe the same PUEX.
  - **SGI pieces:** pieces from both records are kept, as before.
  - **Filled if missing:** patient name (when blank or "Unknown"), PID, Entry ID, Pickup Date and Shipped By. Never overwritten.
  - **Carried over:** Job Closed, Date Returned and any Pending Review items. Source locks from both records are kept.
  - **References:** combined with no repeats. The old check could add the same numbers again when they were in a different order.
  - **Name warning:** if the patient names don't look like the same person, the merge prompt says so before you confirm.
- **Load CSV: newest Confirmed/Void row wins.** A Confirmed or Void PUEX can't be undone in Brightree, so MDR wins.
  - When a Confirmed/Void row matches a record, its Brightree lines always replace the record's.
  - Its status is applied too: Closed (Brightree) and Job Closed, then Verified with No Pickup unchecked for Confirmed, or No Pickup for Void.
  - This replaces the 1.3.0 rule that sent a second Confirmed/Void row to **Pending Review**. New rows no longer go there. Items already in the queue stay until you Approve or Reject them, and "Add Pending Correction" still works.
  - The tradeoff: if one Job # really has two separate Brightree events, the later row overwrites the earlier one.
- **Name matching** (Ref Cleanup and the merge warning): a single letter matches a word starting with it, and extra words are fine. So "Coolidge, Mary G." matches "Coolidge, Mary Grace".
- **Ref Cleanup overrides** (`index.html`): rows held back because the name didn't match now show their changes and an **Include** checkbox. Tick any that are just a typo, and Apply updates them too (the count is on the Apply button). The completed CSV notes them as "Name override".

## Changes in `1.4.17`

- **No repeated reference numbers on a job.**
  - Each number now appears once per record, in its original order. A repeat is removed when:
    - The tracker loads, which cleans existing records once (same self-healing pass that fixes leftover "/" separators). Only records that actually had a repeat are changed and re-synced.
    - Load CSV runs, on both new records and merges.
    - Add Record or Edit Record saves.
  - Records without repeats keep their exact formatting.
- **TEMPORARY (`index.html` only): 🧹 Ref Cleanup button** (MDR only, next to Load CSV), for a one-time cleanup from Appsmith.
  - It takes `MDR_Reference_Cleanup.csv`, built from Appsmith's Table1Copy export. The columns are Entry ID, Patient Name, Shipped By ("Assigned To" as First L.) and References (Appsmith's SO / PUEX IDs, " / " changed to ", ").
  - It matches **existing records only by Entry ID** and never adds a record.
  - **References:** it keeps the record's own numbers with repeats removed, and adds any Appsmith number the record is missing. Nothing else is removed, so PURL and numbers from the daily stay.
  - **Shipped By:** it fills only a blank Shipped By, then locks it.
  - A record whose name clearly differs from Appsmith's is skipped and listed. Names that share all their words still match, for example two patients on one entry.
  - It shows a preview first, and nothing is saved until Apply. After Apply, it downloads `MDR_Reference_Cleanup_Completed_YYYY-MM-DD.csv` (Patient, Job #, Entry ID, Reference Before, Reference After, Changes) and logs to the App Log.
  - It will be removed in the next release.

## Changes in `1.4.16`

> **Before deploying:** add a **ShippedBy** column to the ExchangeRecords list (**+ Add column**, **Single line of text**, name it `ShippedBy`). The app checks for it and only saves Shipped By once it exists, so nothing breaks if it's added later.
>
> **xMDR Tools:** the patient links send the extension a message, `{ __xmdrOpenPatient: true, pid, page: 'orders' | 'contacts', url }`, where `url` has `{KEY}` in place of the PatientKey. xMDR Tools' `exchange-bridge.js` needs a listener that looks up the PatientKey for `pid` in its CSV, swaps it into `url`, and opens it. If the PID isn't found, it does nothing.

- **Fixed: copy buttons.** Every ⧉ icon stopped its own click, so the copy code never ran. Copying now listens before anything else on the page, and also keeps the click from opening the record.
- **Patient name in the table:** for MDR it opens the patient's **Orders** page in Brightree, through xMDR Tools' PID → PatientKey lookup. For SGI it's plain text and no longer opens the record. The Appsmith link moved to Entry ID in Edit Record.
- **Edit Record layout,** left then right:
  - Patient ID | Reference
  - Patient Name | Job #
  - Entry ID | Pickup Date
  - Shipped By | Drop Off By (renamed from Late By Date)
- **Edit Record links (MDR only):**
  - The **Patient ID** label opens Patient Orders.
  - The **Patient Name** label opens Patient Contacts.
  - The **Entry ID** label opens the Appsmith service entry.
  - These labels are underlined for MDR and plain for SGI.
- **Reference boxes (MDR only):** a gray box above Reference for each number. A **single click copies** it. A **double-click opens** the Sales Order (`SalesOrderKey` = the 6-digit number starting with 1) or the PUEX (`PuExKey` = the 5-digit number starting with 3) in Brightree.
- **Shipped By (new):**
  - Filled from the daily's **Assigned To** column as first name plus last initial (`Lastname, Charleston` or `Charleston Lastname` becomes `Charleston L.`).
  - Fills only when blank, and locks when it comes from the CSV.
  - Records created from Confirmed/Void rows get it too.
- **Serial # box:**
  - Shows only for items in `SERIAL_ITEMS` (currently Apnea Monitor, Oximeter, Concentrator, Suction Machine; the full list is still to come), or for any piece that already has a serial.
  - On those rows, the serial is no longer repeated in the item name.
- **SGI and hand-added records:** once saved, SGI can only edit **Patient ID, Job # and Reference**, unless those are locked from source. Name, Pickup Date, Entry ID and Shipped By are locked for SGI. MDR can edit anything not locked from source.
- **Load CSV onto a hand-added record:**
  - Fills in anything missing (including a blank or "Unknown" name, plus Shipped By) and never overwrites.
  - Only fields the CSV row actually carries get locked.
  - A hand-added piece whose name matches a source piece becomes the source piece: it's locked, and the requested qty comes from the CSV while SGI's received qty and check-in stay.
  - Other hand-added pieces are left alone, and SGI can still add or remove them.
- **GitHub button** (`index.html`, only for antwainjordan@mdr.net): opens the repo's upload page, in the spot where the PUEX Backfill button used to be.

- **Fixed: empty area below the table.** Each date field has a hidden calendar picker behind it. Those pickers had no anchor, so they stretched the page far below the table, and scrolling past the last row scrolled the whole page into empty space. They're now anchored to their date field, so the page is exactly one window tall and the table always reaches the bottom.
- **Removed the temporary 🧾 PUEX Backfill button** (`index.html`). The one-time status catch-up is done. Its description under `1.4.15` is kept as a record.

## Changes in `1.4.15`

- **Active tab piece cards, reworked again:**
  - The lock (🔒 or ✍🏼) sits above the checkbox, outside the card's left edge, both aligned to the top. The ✍🏼 / ✕ remove button for hand-added pieces sits outside on the right, also top-aligned.
  - Inside the card there are three columns. The PKP / EXC / RTN badge sits at the **bottom left**.
  - The item name is 13px and vertically centered in the card, with the serial number on the same line in smaller gray text (it wraps only when there's no room). Short / Over / Match is smaller (10.5px) and sits centered above the SGI qty box. Cards are about 40px tall.
  - The qty boxes are narrow (24px) with no arrows and sit side by side in the **bottom right** in both views: MDR inside, SGI at the card's edge. An empty, editable SGI box is blank, with a blinking caret centered inside it until a number is typed (it no longer shows the requested qty as faded hint text).
  - The MDR box is always `#4f8ef7` blue, in every theme and in SGI view (the SGI theme used to turn it red). The SGI box is always red.
  - The Pieces / Items column is back to 260px to 400px.
- **Locks:**
  - The lock map now decides for every record. A hand-added record starts unlocked. Once Load CSV matches it, its source fields (Patient Name, Patient ID, Pickup Date, Entry ID) lock. Before this, hand-added records stayed fully editable even after a CSV match.
  - Job # now locks when it came from the CSV, including on older records that already have source locks. Reference stays editable.
- **Status:**
  - Confirmed and Void jobs are hidden from SGI's view entirely. MDR still sees them only under **✓ Completed**.
  - For SGI, **Complete** now means **Job Closed**, so a returned job moves from Pending to Complete as soon as SGI closes it.
- **TEMPORARY (`index.html` only): 🧾 PUEX Backfill button** (MDR only, next to Load CSV), for a one-time catch-up of PUEX statuses Brightree has but the tracker never got. One example is jobs 4070471, 4070809 and 4072011, which Brightree shows as Void but which never came through a Load CSV.
  - It takes `MDR_PUEX_Status_Backfill.csv`, built from Brightree's PUEX Line Item Report with each PUEX's patient looked up through the Sales Order it was first placed on. The file has one row per line item: PUEX ID, PUEX Status, Patient ID, Patient Name, Sched Date, Item, Type, Qty.
  - It matches **existing records only**, on the patient ID plus the 5-digit PUEX # in Reference. A record with no PID matches on the PUEX # if its name agrees or is blank. It never adds a record.
  - It fills only what's missing: Patient Name (blank or "Unknown"), Patient ID and Pickup Date, which then lock. On still-open records it also sets Closed (Brightree), Job Closed, Date Returned, Verified (Confirmed) or No Pickup (Void), and saves Brightree's line quantities for the Completed tab if none are saved yet. A record whose PUEX is still New in Brightree only gets its missing fields filled.
  - On records already Confirmed, Brightree's status is applied too. Brightree Void checks No Pickup. Brightree Confirmed on a record marked Void unchecks No Pickup and checks Verified. Records Brightree still shows as **New** are left as is and listed in the preview.
  - After Apply, a CSV of everything it completed downloads automatically (`MDR_PUEX_Backfill_Completed_YYYY-MM-DD.csv`).
  - It shows a preview first. Nothing is saved until you click Apply, and it's logged to the App Log.
  - It will be removed in `1.4.16`. `index-offline.html` doesn't have it.

## Changes in `1.4.14`

- **Active tab: new piece card layout.** The checkbox sits outside the card on the left, vertically centered to it. For pieces added by hand, the ✍🏼 / ✕ remove button sits outside the card on the right, opposite the checkbox.
- **Lock and badge:** the lock (🔒 or ✍🏼) is in the card's top left corner, with the PKP / EXC / RTN badge right under it.
- **Item name:** centered in the space to the right of the lock and badge. The serial number stays on its own line under the name.
- **Short / Over / Match:** sits in line with the item name. When the name wraps to two lines, the word is vertically centered to the name.
- **Qty boxes:** the MDR requested qty and SGI received qty are smaller (spinner arrows hidden so the numbers stay readable) and stacked on the right side of the card, MDR on top. The item name is vertically centered between the lock/badge stack on the left and the qty stack on the right, so each card stays short top to bottom. The MDR box is always blue and the SGI box is always red, whatever the Match / Short / Over state. The card's border and background still show that state.
- **Column width:** the Active tab's Pieces / Items column uses its older width (225px to 280px).
- The Completed tab is unchanged. `index-offline.html` has the same changes, and its SharePoint handling is untouched.

## Changes in `1.4.13`

- **Active tab: piece rows are boxed cards again** — checkbox, badge, lock, name, Short/Over/Match text — with the card's border/background colored green/yellow/red once a source piece is checked in.
- **Active tab: the MDR requested qty is its own box again** (always blue, read-only), sitting right next to the SGI received-qty box, no icon between them. Replaces the inline "×N" tag from `1.4.10`/`1.4.12`.

## Changes in `1.4.12`

- Fixed: the received-qty box on the Active tab's piece rows had no visible border before a piece was checked in — its default border color was almost the same shade as its own background. The border now uses a lighter, clearly visible color in its default state; the green/yellow/red match/short/over borders and the blue/orange read-only borders are unchanged.

## Changes in `1.4.11`

- **📊 Inventory Reports** button in the header (MDR view only, and only when the xMDR Tools extension 1.19.0+ is installed). It opens a window that starts xMDR Tools' Inventory Reports run and shows its progress and the "PUEX status shows None" list. The work happens in the extension (`exchange-bridge.js`); the page only says hello, sends Start / ▶ Run, and draws the state it gets back. See the xMDR Tools README.
- The button is re-checked whenever the MDR/SGI view changes (`applyRoleUI` calls `invShowBtn()`), so the SGI preview hides it too.
- `<meta name="version">` now carries the app version (`1.4.11`) instead of an old date stamp, so every version marker in the file matches.
- `index-offline.html` has the same code (button, window, script, `invShowBtn()` call, version markers) so the two files don't drift apart. The button stays hidden there: xMDR Tools only connects to the live site, so the offline copy never gets an answer.

## Changes in `1.4.10`

- **Active tab: "Pieces / Items" rows have their own flat, single-line style again** — checkbox, badge, lock, name, Short/Over/Match text with a requested-qty `×N` tag, and one color-coded received-qty box. This is no longer the Completed tab's boxed card layout, which had made the column too narrow and clipped the Short/Over text. The Completed/Void tab's cards are untouched; Active and Completed now use fully separate CSS classes, so a change to one tab can't bleed into the other.
- **Active tab: widened the "Pieces / Items" column** (up to 400px) for the flat row style.

## Changes in `1.4.9`

- **Completed tab: loads 25 rows at a time** instead of rendering every completed/void record at once (this is what was making the Completed tab slow to open). The next 25 load automatically once you've scrolled about 2/3 of the way down what's currently loaded. Searching still shows every match at once.
- **Completed & Active: smaller, tighter piece cards** — max width down from 370px to 320px, and internal padding/spacing cut roughly in half.
- **Completed & Active: PKP/EXC/RTN badge moved into the qty row**, next to the MdR Request box, without affecting that box's icon/qty grouping. The header now just shows the lock and the centered item name.
- **Completed tab: Actions column lines up by brackets** — label on the left, bracketed items on the right, so every line's brackets start at the same x position regardless of label length.
- **Completed & Active: outer checkmark/checkbox vertically centered** next to its card.
- **Completed & Active: piece-card icons (lock, eyes, etc.) are about 30% bigger.**

## Changes in `1.4.8`

- **Completed tab: MdR Request qty box gets a 👀 icon**, matching the other boxes in the card (previously it had none).
- **Completed tab: 2-box cards (SGI Returns) now split into a left/right half** — MdR Request centered in the left half, SGI Report/Returns centered in the right half. 3-box cards (MDR Confirmed/Void) keep spreading evenly across the full width.
- **Completed & Active: lock and return-type badge split into a top half / bottom half**, each left-aligned and vertically centered in its own half (previously both were center-aligned as a tight stack).

## Changes in `1.4.7`

- **Completed & Active: evenly-spaced qty boxes.** The qty/icon pairs in each piece card now spread evenly across the card's full width (whether it's 2 boxes or 3), instead of bunching together in the middle with empty margins on both sides.

## Changes in `1.4.6`

- **Active tab: fixed a layout regression from 1.4.5.** The "Pieces / Items" rows had gone to a tall, gappy vertical stack because the shared piece-card style changed in 1.4.5 but these rows weren't rebuilt to match. They now use the same 2-part card as the Completed tab's comparison cards — a header row on top, a centered row of qty boxes underneath — with every checkbox, qty edit, and delete control still working exactly as before.
- **Completed & Active: item names are now centered** in the card's header row (e.g. "D Cylinder Bag"), and the **return-type badge (PKP/EXC/RTN) now stacks under the source lock icon** instead of sitting beside it. Applied to both tabs' piece cards for a consistent look.

## Changes in `1.4.5`

- **Completed tab: reworked both piece cards around the same 2-part layout** — a header row (return type, source lock, item name, free to wrap) on top, and a row of qty boxes centered underneath, each box bound to its own icon in one container so the two move together. This replaces 1.4.4's stacked-with-labels MDR card (dropped — it made the MDR card much taller than its SGI counterpart and threw off the row-for-row alignment between the two columns). Both cards are now the same shape, so SGI Returns and MDR Confirmed/Void line up again piece for piece.

## Changes in `1.4.4`

- **Completed tab: MDR Confirmed/Void card's 3 quantity boxes restacked** — MdR Request, SGI Report, and MdR Confirmed now show as labeled rows stacked vertically instead of side by side, each qty box right-aligned with its icon (when there is one) in the same container so they move together. The SGI Returns card is unchanged (still 2 boxes side by side).

## Changes in `1.4.3`

- **Completed tab: consistent centered alignment** — Drop Off Date, Patient, Job #, Total Qty, and Match are now all centered (previously a mix of left and centered, which read as inconsistent). Reference was already centered.
- **Completed tab: capped the Patient column's width** so a long name wraps onto a second line instead of stretching the column wider than it needs to be.

## Changes in `1.4.2`

- **Completed tab: renamed "1:1 Match" to "Fulfilled"**. It was never actually a quantity check — it just meant the Confirmed/Void CSV row found an unambiguous existing record to resolve onto, so a job could read "1:1 Match" even with a real shortage or overage sitting right next to it in the Actions column. "Fulfilled" reflects what it's actually reporting; a real discrepancy still shows under Actions regardless of this label.
- **Completed tab: "Pickup Date" column is now "Drop Off Date"** (the calculated Late By date), and the row sort now follows it.
- **Completed tab: tightened columns** — Patient, Job #, Reference, Total Qty, and Match are narrower, freeing up room for SGI Returns and MDR Confirmed/Void so a 3-quantity-box card has more space before it needs to wrap onto a second line.

## Changes in `1.4.1`

- **Fixed the Completed tab Actions column overflow**: a piece card with 3 quantity boxes (the MDR Confirmed/Void side) had no width cap matching its column, so it could bleed past the column boundary and crowd into the Actions text next to it. The card now wraps its quantity boxes onto a second line when there isn't room, instead of pushing wider than its column.

## Changes in `1.4.0`

- **Replaced "Load Daily" and "Load Confirmed/Void" with one "📥 Load CSV" button** (MDR only). Power BI's "MDR | SGI PUEX Report" export can now include New, Confirmed, and Void rows together in one file — each row is routed automatically by its own `Pickup/Exchange Status` into exactly the same matching/merge logic the two old buttons always used; nothing about how a row resolves onto a record changed, only how many files you have to pick each time.
- **Locked to matching headers only**: the upload is accepted or rejected purely on whether the file has the columns the combined report always has (`Patient Name`, `Job #`, `Pieces`, `OtherIDs`, `DropOff By`, `Pickup Date`, `Entry ID`, `Pickup/Exchange Status`) — there's no filename check. A file missing any of those columns is rejected with a message naming which ones it expected, instead of guessing from the name.
- See "Load CSV (MDR)" above for the full behavior (unchanged from the old Load Daily/Load Confirmed/Void, just combined into one step).

## Changes in `1.3.1`

- **Fixed a real identity-collision risk**: SGI sometimes leaves patient name as the literal placeholder "Unknown" (a required field with no real name yet), and a blank name could also round-trip through SharePoint's `Title` field and come back as "Unknown" too. The patient-matching logic previously fell back to name-only equality, so two different "Unknown" records could get treated as the same patient if they ever shared a reused job#/reference/entry ID. Fixed: "Unknown" (and a truly blank name) never counts as a name match on its own.
- **Manual "Add Record" duplicate check**: saving a new record for a Job # that already exists no longer silently creates a duplicate. The new record still saves (you can override), but a review popup opens — on save and again on the next load until resolved — comparing it against the existing record, with **Merge Now** or **Keep Both — Resolved**.
- **New "⧉ Find Duplicates" tool** (button near Add Record): scans existing records for likely duplicates — same Job # + patient match, or same Job # + same reference #, whichever flags it — and surfaces each pair to Approve (merge) or Reject (dismiss, won't resurface). Nothing merges without you confirming it.
- **New alert (non-blocking)**: if a Daily or Confirmed/Void row comes in with a reference # that already exists on a *different* Job #, you're notified via toast and the App Log — regardless of whether the patient matches. This usually means an old job was never closed out.
- **Daily/Confirmed-Void import**: a row with a real patient name but no resolvable Patient ID now shows a small "⚠ no PID" flag next to the Patient ID in the table, instead of silently saving with it blank. Computed live off the record's own fields — no SharePoint schema change needed, and it clears itself the moment a PID is filled in.
- Completed tab: the SGI Qty box is now always orange, regardless of match/short/over/issue state (matching how the MDR Confirmed Qty box is always blue).

## Changes in `1.3.0`

- **Fixed a real data-loss bug**: a job # can legitimately have more than one Brightree pickup/exchange event (e.g. two patients shipping to the same address), but "Load Confirmed/Void" matches by job#/reference/Entry ID, not pickup date — so a second event's CSV row was silently overwriting the first event's Confirmed/Void numbers with no trace left behind.
- **New rule: nothing merges without you approving it.** Once a job already has Confirmed/Void data on it, anything new goes to a "Pending Review" queue instead of overwriting it. Every pending item waits for you to Approve (adds its quantities into the job's total) or Reject (discards it) on the Completed tab.
- Completed tab: a job with something waiting shows a "🕐 N Pending" badge next to its Job #; open the row to see each pending item (what it is, where it came from, when it came in) with its own Approve/Reject buttons. New "🕐 Pending Review" toolbar toggle filters the tab down to just those jobs.
- New **"➕ Add Pending Correction"** button (Completed tab, MDR only): manually queue recovered Confirmed/Void numbers for a job — e.g. numbers pulled from the Brightree PUEX Line Item Report for an event this app already overwrote before this queue existed. Goes through the same Approve/Reject review as a real import; nothing is written to the job's totals until you approve it.
- Needs a new SharePoint column, `PendingResolved` — see "Adding the Completed tab's 3 columns" above.

## Changes in `1.2.1`

- **Completed tab — Actions column spacing fixed**: proper padding from the MDR Confirmed/Void column instead of sitting flush against it, each category top-aligned on its own spaced line instead of one packed block, and a job with all 4 categories flagged no longer visually overlaps the row below it.

## Changes in `1.2.0`

- **Completed tab — Actions column rebuilt around 4 categories**: instead of one free-text sentence per piece, the Actions column now lists issues under up to 4 labeled categories: **MdR Request Issues** (original Request vs. what SGI physically returned — 🔻 shortage / 🔺 overage / 🏷️ match), **SGI Reports** (only shown when the Confirmed/Void CSV disagrees with SGI's Daily-entered return — 🚚/⚠️, a paperwork-accuracy flag separate from a real shortage), **SGI Returns** (same Request-vs-Returns comparison as category 1, plus ⁉️ for something Brightree reported that was never on the original Request at all), and **SGI or Patient Missing** (the action step — ➕ add qty back to the patient's account, ➖ remove an Exchange overage, or 🏷️➖ when that overage already has a matching Pickup line in the Confirmed data).
- A piece with no issue on any of the 4 axes is left out of the breakdown entirely. A fully clean job still collapses to a single "🏷️ 100% Exact Match" line; a job with only a reporting mistake (Reports ≠ Returns, but the real-world Request and Returns still match) now lists that piece under the relevant categories instead of hiding it, so reporting mistakes stay tracked even when nothing needed correcting.
- Items sharing the same icon within a category are grouped onto one line instead of one line each.

## Changes in `1.1.0`

- **Completed tab — outer status icon**: every piece card now shows a status icon to its left (same icon drives both the SGI card and the MDR card for a given piece): 🚫 the whole job is No Pickup/all-zero, 🛑 SGI's own entered qty is blank for that piece, ✍🏼 the piece was manually added, ✅ otherwise.
- **Completed tab — qty boxes restructured**: the grey "×N" qty text next to the item name is gone, replaced by a boxed, blue **Requested Qty** as the first box. The SGI card always shows 2 boxes (Requested Qty, SGI Qty); the MDR Confirmed/Void card always shows 3 boxes (Requested Qty, SGI Qty, Confirmed Qty), not only when there's a mismatch. An icon always sits between adjacent boxes — no two qty boxes ever touch.
- **🚨 consolidated**: no longer sits next to individual piece rows or action messages; one 🚨 now appears once, centered, above the Actions text, only when there's something to flag.
- **Active dashboard**: a manually-added piece's lock-icon slot now shows ✍🏼 (was blank); its remove button shows ✍🏼 at rest instead of a plain pencil (still swaps to ✕ on hover) — matching the Completed tab's manual-entry marker.
- **Renames**: the Completed tab's "SGI" column header is now "SGI Returns" (table) / "Returns" (detail popup). The "MDR Confirmed/Void" popup label is now dynamic — "MdR Confirmed" or "MdR Voided" depending on that record's actual status.

## Changes in `1.0.0`

- **Switched version tracking from date-slug build tags to SemVer**, starting fresh at `1.0.0`. Older sections below keep their original tags (e.g. `2026-11-06-qol-batch`) as the historical record of what they were actually called at the time.
- **Fixed a real data-masking bug**: on a job flagged No Pickup / all-zero, the Completed tab's SGI column forced every piece's displayed qty to a flat `0` — even when SGI had actually entered a real, different, non-zero number for that specific piece (confirmed from a real job: 3 of 5 pieces had real qtys of 2, 2, and 3, but all displayed "0"). The real number was only visible secondarily, buried inside the MDR column's 🚚/⚠️ mismatch indicator.
- The SGI column now always shows SGI's own real entered qty per piece. The 🚫 "no pickup" treatment only applies to a piece that is itself genuinely blank or actually `0` — a job-level flag can no longer overwrite one piece's real number with another piece's zero.
- Applied the identical fix to the MDR Confirmed/Void column's matching case, so a voided job with one piece that was actually confirmed with a real qty shows that real qty instead of being relabeled 🚫.
- Genuine all-zero / No Pickup jobs (every piece truly blank or 0) are unaffected — they still show 🚫 across the board, verified against the fix.

## Changes in `2026-11-08-piece-icons-mdr-box`

- **Piece icon convention unified**: every piece checklist (Active dashboard and Completed tab) now uses the same icons for the same thing — 🏷️ = exact 1:1 match, 🔺 = received MORE than requested, 🔻 = received LESS than requested. The Completed tab's SGI column had the over/short arrows backwards before this fix.
- **Active dashboard — manually-added pieces**: pieces added at check-in (not locked to the original order) now show their qty in a plain, non-editable box instead of an editable one. The ✕ remove button is now a pencil icon at rest; hover over it to see the ✕ before clicking.
- **Active dashboard — MDR qty box added**: every piece now shows a second boxed qty — the MDR requested amount — sitting right beside the existing SGI return qty box. Always blue (MDR's color), always non-editable, regardless of match/short/over state.
- **Completed tab restyled to match Active exactly**: the SGI and MDR Confirmed/Void columns now use the same boxed markup as the Active dashboard's checklist (PKP/EXC pill badge, lock icon, boxed qty) instead of a plain text line. The MDR column's qty box is always blue; the SGI column's qty box still color-codes green/yellow/red by match/short/over, same as Active.
- MDR's ➕ (shortage/add to inventory) and ➖ (overage/remove from inventory) icons are intentionally left as-is and not merged with the SGI side's 🔺/🔻 — they answer a different question (what to do to the account) than SGI's icons do (did the return match the request).

## Changes in `2026-11-06-completed-search-fix`

- Fixed: the search bar did nothing on the Completed tab — it filtered the Active view but the Completed tab's render function never checked it. Search now works there the same way it does on Active (patient, patient ID, job #, reference, equipment), including the space/comma-insensitive reference number matching from the QOL batch below.

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
