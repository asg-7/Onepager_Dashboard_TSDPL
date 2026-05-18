# Kalinganagar Plant — One Pager Dashboard

Browser-only analytics dashboard for the Kalinganagar plant. Drop your SAP Excel export onto `index.html`, get an interactive production & inventory view, download as PDF. No Python, no server, no install.

---

## How to use

1. Open `index.html` in Chrome or Edge (double-click it)
2. Drag and drop your SAP `.xlsx` export onto the drop zone, or click **Choose File**
3. Use the **Line tabs**, **Shift dropdown**, and **📅 Select Date Range** button to slice the data
4. Optionally click **📋 PLANNING** to enter or upload monthly targets, maintenance, and APU data
5. Click **⬇ DOWNLOAD REPORT** — the browser's **Print / Save as PDF** dialog opens immediately

That's it. The dashboard stays open while you save the PDF; no page navigation occurs.

---

## File structure

```
onepager_dashboard/
├── index.html        ← the whole dashboard, self-contained
├── libs/
│   ├── xlsx.min.js      ← SheetJS 0.18.5  (local copy for offline use)
│   └── chart.min.js     ← Chart.js 4.4.1  (local copy for offline use)
└── README.md
```

> **Note:** `chartjs-plugin-datalabels` (v2.2.0) is loaded from the jsDelivr CDN — it is **not** bundled locally. An internet connection is required for data labels on charts. The `libs/` folder enables offline use for SheetJS and Chart.js. If `libs/` is missing, the page auto-falls back to Cloudflare CDN for those two libraries.

> **Calendar:** Date range selection uses a built-in, zero-dependency single inline calendar modal — no external library needed.

---

## Excel file requirements

The dashboard looks for specific sheet tabs in this priority order:

**Primary (Separate Sheets)**

| Sheet name              | Contents                 |
| ----------------------- | ------------------------ |
| `RM`                    | Raw materials inventory  |
| `FG`                    | Finished goods inventory |
| `Production Report(RM)` | RM production rows       |
| `Production Report(FG)` | FG production rows       |

If only some sheets exist (e.g. only `RM` and `Production Report(RM)`), the dashboard loads what it can and treats missing sheets as empty.

**Fallback (Single Sheet)**  
If none of the named sheets above are found, the dashboard attempts to read from:

- `One Pager Report`

**For Expected Rate / Plan data** — The dashboard reads monthly ABP targets from the `One Pager Report` sheet (even when actuals come from `RM`/`FG` sheets). These targets power the On Date Plan, To Date Plan, Expected Rate, Actual Rate, and Asking Rate columns. You can also enter or upload targets manually via the **Planning Panel**.

**Before uploading:** No external preprocessing or manual cleaning is needed. The dashboard automatically cleans raw data on upload. It trims whitespace, drops rows missing critical IDs, deduplicates rows based on Batch + Proc Line, coerces numeric strings to floats, and automatically derives `Location` and `Category` from the raw `Yard` and `Status` columns. Just drop the raw SAP export directly.

---

## Column mapping

The dashboard auto-maps SAP column names via an `ALIASES` object in `index.html`. If a column isn't being picked up, find `ALIASES` near the top of the `<script>` block and add the variant.

| Canonical name     | SAP column names accepted                                                                     |
| ------------------ | --------------------------------------------------------------------------------------------- |
| `proc_line`        | `PROC Line`, `Proc Line`, `Proc. Line`, `Processing Line`, `PROCLINE`, `PROC_LINE`            |
| `net_wt`           | `Net Wt`, `NET WT`, `Net Weight`, `NETWT`, `NWT`, `Net_Wt`, `NET_WT`                          |
| `bundle_age`       | `Bundle Age`, `BUNDLE AGE`, `Age`, `AGE`, `BundleAge`, `Bundle_Age`                           |
| `location`         | `Location`, `LOCATION`, `Loc`, `Storage Location`, `StorageLoc`, `Stor. Location`             |
| `quality_hold`     | `Quality Hold`, `QUALITY HOLD`, `QHold`, `Q Hold`, `QH`, `Qual Hold`                          |
| `order_id`         | `ORDER ID`, `Order ID`, `SO Number`, `SalesOrder`, `Sales Order`, `SO No`                     |
| `date_creation`    | `DATE CREATION`, `DATE Creation`, `Creat.Date`, `Creation Date`, `Doc. Date`, `Document Date` |
| `material`         | `Material`, `MATERIAL`, `Mat.`, `Mat No`, `Material No`, `MatNo`, `Material Number`           |
| `status`           | `STATUS`, `Status`, `Stat`, `Stock Status`, `StockStatus`                                     |
| `mass`             | `MASS`, `Mass`, `Weight`, `Gross Wt`, `Gross Weight`, `MASS (MT)`                             |
| `plant`            | `Plant`, `PLANT`, `Plt`, `Plant Code`                                                         |
| `batch`            | `Batch`, `BATCH`, `Batch No`, `Batch Number`                                                  |
| `shift`            | `Shift`, `SHIFT`, `Work Shift`, `WorkShift`, `Shift Code`                                     |
| `sent_to_cpdy`     | `SENT TO CPDY`, `Sent To CPDY`, `SentToCPDY`                                                  |
| `category`         | `Category`, `CATEGORY`, `Cat`, `Material Category`                                            |
| `mother_coil_mass` | `MOTHER COIL MASS`, `Mother Coil Mass`, `MCM`, `Mother Coil Wt`, `Mother Coil Weight`         |
| `place`            | `Place`, `PLACE`, `Stock Place`, `Stor. Place`, `Placement`                                   |
| `yard`             | `Yard`, `YARD`, `Yrd`, `Storage Yard`, `Yard Code`                                            |

Line code mapping for FG and RM sheets (edit `CONFIG.fg_line_mapping` / `CONFIG.rm_line_mapping` in `index.html` if yours differ):

| SAP code | Line  |
| -------- | ----- |
| CTL1     | WCTL1 |
| CTL2     | WCTL2 |
| WCTL1    | WCTL1 |
| WCTL2    | WCTL2 |
| SLIT     | SLIT  |

---

## Filters

Three filters applied simultaneously:

- **Line tabs** — All Lines / WCTL1 / WCTL2 / SLIT
- **📅 Select Date Range** — opens a single inline calendar modal; only dates with data in the file are clickable
- **Shift dropdown** — All Shifts / Morning (A) / Afternoon (B) / Night (C)

### Calendar modal

Click **📅 Select Date Range** in the filter bar. A compact single-calendar modal appears (`max-width: 380px`). Dates with no data in the uploaded file are greyed out and non-clickable.

**Click flow:**

1. Click a **start date** (hint reads: _CLICK START DATE_)
2. Calendar prompts _NOW CLICK END DATE_ — click the end date
3. Click **Apply Range** to filter all tables and charts
4. Click **Clear** to remove the custom range
5. Press **Escape** or click outside the modal to close without applying

The button label updates to show the active range (e.g. `07/05/2025 → 08/05/2025`) when a custom range is applied.

The badge in the header changes from `● DATA LOADED` to `● FILTERED` when any non-default filter is active. Hit **↻ RESET FILTERS** to clear everything.

---

## KPI Strip

The top strip shows 6 key metrics:

| Card                          | What it shows                                                                                              | Source                                                               |
| ----------------------------- | ---------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------- |
| **Total Actual On-Date Prod** | Total production MT on the most recent date; reads col C of One Pager Report, falls back to raw RM data   | `One Pager Report` col C → `Production Report(RM)` MOTHER COIL MASS |
| **Month-to-Date (MTD)**       | Total RM production MT for the current calendar month                                                      | `Production Report(RM)` — MOTHER COIL MASS                          |
| **Expected (Monthly)**        | Monthly ABP target from One Pager / Planning Panel. Click to override manually; click **↺ auto** to reset | `One Pager Report` or Planning Panel                                 |
| **Quality Hold**              | FG MT with Quality Hold = Y                                                                                | `FG` sheet                                                           |
| **Without Order**             | FG MT with Status = WF (and QH ≠ Y)                                                                       | `FG` sheet                                                           |
| **RM In-Transit**             | RM MT where Place = "RM in Transit"                                                                        | `RM` sheet                                                           |

> **MTD reference date:** "Month-to-date" uses the most recent production date in the file as the reference — not today's calendar date — so the figures match SAP even when the file is opened days later.

---

## Production table

A single unified table under **Production Report** with 8 columns:

| Column             | Description                                                                                                | Source                                                            |
| ------------------ | ---------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------- |
| **Lines**          | WCTL1 / WCTL2 / SLIT                                                                                       | —                                                                 |
| **On Date Plan**   | Daily target = Monthly ABP ÷ days in month                                                                 | One Pager / Planning Panel                                        |
| **On Date Actual** | Production MT on the most recent date; prefers One Pager col C, falls back to raw RM MOTHER COIL MASS     | `One Pager Report` col C → `Production Report(RM)`               |
| **To Date Plan**   | Cumulative target = Daily target × days elapsed                                                            | One Pager / Planning Panel                                        |
| **To Date Actual** | MTD RM production MT (current-month actuals)                                                               | `Production Report(RM)` — MOTHER COIL MASS                       |
| **Expected Rate**  | Monthly ABP target (MT/month)                                                                              | `One Pager Report` or Planning Panel                              |
| **Actual Rate**    | MTD Actual ÷ days elapsed (MT/day)                                                                         | Derived from RM data                                              |
| **Asking Rate**    | (Monthly ABP − MTD Actual) ÷ remaining days (MT/day)                                                      | Derived                                                           |

All actuals come from the **RM/FG sheets**; plan values come from the **One Pager Report** (ABP targets) or the **Planning Panel** if manually entered/uploaded.

---

## Planning Panel

Click **📋 PLANNING** in the header to open the Planning Panel. It has four tabs.

> **All fields in the Planning Panel are optional.** You can fill in as many or as few fields as you like and hit Apply — only the fields you've filled will affect the dashboard. You are never required to complete all rows before saving.

### 📊 Machine Targets

Enter monthly production targets (MT) per line (WCTL1, WCTL2, SLIT). The panel auto-calculates:

- **Daily Target** = Monthly ÷ days in month
- **On-Date Target** = same as Daily Target (displayed for reference)

Click **✔ Apply Targets to Dashboard** to push targets into the production table and charts. Applied targets take priority over values auto-read from the SAP file. Lines left blank retain their SAP-derived values.

**Priority order for Expected Rate:**

1. Planning Panel targets (if applied via the button)
2. ABP targets parsed from the `One Pager Report` sheet in the SAP file
3. Manual override typed into the Expected (Monthly) KPI card

### 🔧 Maintenance

Enter planned maintenance data per line: downtime in minutes, maintenance type (PM / BD / OM), scheduled date, and remarks.

**Availability** is automatically calculated live as you type in the Breakdown field:

```
Availability (%) = ((1440 − Breakdown in minutes) / 1440) × 100
```

Where 1440 = total minutes in a day. The Availability cell updates instantly with colour coding:

- 🟢 **≥ 95%** — green (normal operations)
- 🟡 **80–94%** — amber (moderate downtime)
- 🔴 **< 80%** — red (significant downtime)

Click **✔ Save Maintenance Data** to store values in memory (included in the PDF export).

### ⚡ APU

Enter APU (Asset Performance / Utilisation) target % and actual % per line. Variance (pp) is auto-calculated live. Click **✔ Save APU Data**.

### 📁 Upload Plan File

Drop or browse for a separate planning `.xlsx` file. The file must contain a sheet named `One Pager Report` in the same layout as the SAP export. Monthly targets extracted from this file:

- Fill the Machine Targets inputs
- Override any previously parsed SAP rates
- Immediately re-render the dashboard

> The Planning Panel is hidden in the PDF print output.

---

## FG Inventory tables

Two age-bucket tables (TSDPL and CPDY locations):

- Only **Prime category** coils are counted. The dashboard derives this directly from the `Status` column (e.g. `WB` or `DD`), falling back to the `Category` column if needed.
- Location is automatically mapped from the `Yard` column (e.g. `TS` → TSDPL, `CP`/`K6` → CPDY), falling back to the `Location` column or legacy `SENT TO CPDY` flag.
- Age buckets: ≤7 d / 7–30 d / 30–60 d / 60–90 d / 90–180 d / >180 d

---

## RM Status table

Columns: Line / ≤3d Count / ≤3d MT / >3d Count / >3d MT / In-Stock Count / In-Stock MT / In-Transit MT

- **In-Stock** = rows where `Place` contains `"RM in Stock"` (case-insensitive)
- **In-Transit** = rows where `Place` contains `"RM in Transit"` (case-insensitive)
- Age split (≤3 days vs >3 days) uses the `Bundle Age` column

---

## Date parsing

SAP exports dates as `DD/MM/YYYY HH:MM:SS` strings. The dashboard detects this format via regex and parses it correctly as day/month/year — JavaScript's native `new Date()` would otherwise misread it as month/day (US format).

---

## PDF / Print export

Click **⬇ DOWNLOAD REPORT**. The dashboard:

1. Snapshots all Chart.js canvases to PNG images
2. Pulls the current table data and KPI values
3. Builds a clean, print-ready HTML report and opens it in a new tab
4. The print dialog triggers automatically — set **Destination → Save as PDF** in Chrome/Edge

If Planning targets have been applied, the PDF includes a **Planning Targets & Machine Data** section with Monthly Target, Daily Target, Maintenance summary, APU Target, and APU Actual per line.

Print stylesheet targets **A3 Landscape** with 10 mm margins and forces colour output.

> **Note:** Allow pop-ups for this file. The temporary Blob URL is automatically revoked after printing.

`Ctrl + P` on the dashboard itself also works — the `@media print` stylesheet hides all UI chrome (including the Planning Panel) and reformats cards for paper.

---

## Troubleshooting

**FG inventory shows zero**  
Check two things: (1) force-recalculate in Excel (Ctrl+Shift+F9 → Save) so the Location column has values not formula strings; (2) check the `Category` column — only `Prime` rows are counted.

**All numbers zero after upload**  
Check sheet names match exactly (case-sensitive). Also check the `ALIASES` object — SAP sometimes changes column headers between exports.

**Production shows zero**  
Ensure the `MOTHER COIL MASS` column is present and populated in `Production Report(RM)`. If it's absent, the dashboard falls back to `NET WT` / `MASS`.

**Expected Rate / On Date Plan shows "—"**  
The dashboard reads ABP targets from the `One Pager Report` sheet. If that sheet is missing or the header layout differs, no plan values will be found. Use the **Planning Panel → Machine Targets** tab to enter them manually, or upload a separate plan file.

**RM in-stock / in-transit shows zero**  
Check the `Place` column in the `RM` sheet. Values must contain `"RM in Stock"` or `"RM in Transit"` (the match is case-insensitive and partial, so `"RM in Stock (TSDPL)"` also works).

**Date range calendar shows no selectable dates**  
Dates are sourced from the production sheets only (`Production Report(RM)` and `Production Report(FG)`). If those sheets are empty, no dates will appear.

**Dashboard opens but charts/upload don't work**  
SheetJS or Chart.js didn't load. Check that `libs/xlsx.min.js` and `libs/chart.min.js` exist. If missing and offline, the page won't function. Note: `chartjs-plugin-datalabels` always loads from CDN — chart value labels won't appear without internet access.

**DOWNLOAD REPORT button does nothing**  
Data must be loaded first. Check for a "Pop-up blocked" icon in the browser address bar and allow pop-ups.

**Shift filter returns no data**  
Check what values are in your SHIFT column and add variants to `ALIASES.shift` in `index.html`.

---

## Changing config (lines, buckets, etc.)

Everything lives in the `CONFIG` object near the top of the `<script>` block in `index.html`:

```js
const CONFIG = {
  lines: ["WCTL1", "WCTL2", "SLIT"],
  fg_line_mapping: {
    CTL1: "WCTL1",
    CTL2: "WCTL2",
    WCTL1: "WCTL1",
    WCTL2: "WCTL2",
    SLIT: "SLIT",
  },
  rm_line_mapping: {
    CTL1: "WCTL1",
    CTL2: "WCTL2",
    WCTL1: "WCTL1",
    WCTL2: "WCTL2",
    SLIT: "SLIT",
  },
  status_codes: {
    in_stock: ["IS", "SC", "1C", "2C"],
    in_transit: ["IB", "RQ", "IN"],
  },
  locations: { tsdpl: "TSDPL", cpdy: "CPDY" },
  age_buckets: {
    labels: [
      "<7 Days",
      "7–30 Days",
      "30–60 Days",
      "60–90 Days",
      "90–180 Days",
      ">180 Days",
    ],
    upper_bounds: [7, 30, 60, 90, 180, null],
  },
  sheet_names: {
    fg: "FG",
    rm: "RM",
    prod_fg: "Production Report(FG)",
    prod_rm: "Production Report(RM)",
    one_pager: "One Pager Report",
  },
};
```

- **Add a new line:** add to `CONFIG.lines` and both line mappings. Tabs, filters, and the Planning Panel pick it up automatically.
- **Change age bucket boundaries:** edit `CONFIG.age_buckets.upper_bounds`. Keep `labels` and `upper_bounds` the same length.
- **Change location names:** edit `CONFIG.locations.tsdpl` / `CONFIG.locations.cpdy` to match your `Location` column values.
- **`status_codes`** — legacy STATUS-code arrays kept for backward compatibility; primary RM stock/transit detection now uses the `Place` column (`"RM in Stock"` / `"RM in Transit"`).

---

## Libraries used

| Library         | Version | Loaded from                        | Purpose                                                |
| --------------- | ------- | ---------------------------------- | ------------------------------------------------------ |
| SheetJS         | 0.18.5  | `libs/xlsx.min.js` → CDN fallback  | Parse the uploaded `.xlsx` entirely in-browser         |
| Chart.js        | 4.4.1   | `libs/chart.min.js` → CDN fallback | All charts (bar, line, doughnut, radar)                |
| ChartDataLabels | 2.2.0   | jsDelivr CDN (online only)         | Value labels on top of chart bars                      |
| Google Fonts    | —       | Google Fonts CDN                   | Montserrat + Lato (falls back to system fonts offline) |

No Flatpickr. No build step. No npm. Just open `index.html`.

---

## Changelog

### v4.9 (May 2026)

**Maintenance — Availability column**

- Added a live-computed **Availability (%)** column to the Maintenance tab in the Planning Panel
- Formula: `((1440 − Breakdown mins) / 1440) × 100`
- Updates instantly as you type in the Breakdown field — no need to click Save
- Colour-coded: ≥95% green, 80–94% amber, <80% red
- Default display is 100.00% when no breakdown is entered
- Maintenance grid widened from 5 to 6 columns; `max-width` extended to 980px to accommodate

### v4.8 (May 2026)

**Planning Panel — partial-entry apply**

- Removed the all-or-nothing validation gate from all three Apply/Save buttons (Targets, Maintenance, APU). Previously, every single field in a tab had to be filled before the button would do anything, which blocked the dashboard from updating even if only one or two values needed entering.
- Removed the `required` attribute from all 21 planning inputs across the three tabs.
- Apply now works with any combination of filled fields. Empty fields fall back to 0 / not set, preserving SAP-derived values where nothing was manually entered.
- Removed the now-redundant `.plan-input:invalid` / `.plan-input.touched:invalid` CSS rules.

### v4.7 (May 2026)

**Total Actual On-Date Production — KPI renamed & data source fix**

- KPI card 1 renamed: "Last Day Prod" → **Total Actual On-Date Prod**
- On-Date Actual now reads values from the `One Pager Report` sheet column C ("On Date → Actual") when the sheet is present in the uploaded file. Falls back to computing from `Production Report(RM)` MOTHER COIL MASS if no One Pager sheet is found.
- **Bug fix:** the One Pager parser loop scanned past the production table's "Total" row into the FG inventory section below, where the same line names (WCTL1, WCTL2, SLIT) reappear. Column C in the FG section contains age-bucket values, which were silently overwriting the correct On-Date Actual values. Fixed by breaking the loop at the "Total" row boundary.
- KPI subtitle now shows data source: `(One Pager)` or `(RM)` so the user knows which source is being used.
- PDF export updated with matching KPI label.

### v4.5 (May 2026)

**Smart FG Inventory Derivation (No more Ctrl+Shift+F9)**

- The dashboard now automatically derives the `Location` (TSDPL/CPDY) directly from the raw `Yard` column codes.
- It also derives the `Category` ("Prime") directly from the raw `Status` column (`WB` / `DD`).
- This bypasses stale VLOOKUP formulas entirely, meaning users no longer have to manually force-recalculate the Excel file before uploading just to see their FG inventory. Graceful fallbacks exist for older file structures.
- Added `yard` column aliases.

### v4.4 (May 2026)

**Without-Order & Quality Hold — day-wise age buckets**

- `aggregateWithoutOrder()` and `aggregateQualityHold()` now return a bucketed `{line: {bucket: MT}}` structure — identical to `aggregateFGInventory()` — instead of a flat `{line: MT}` map
- Both tables (**Without-Order WF** and **Quality Hold**) now display the same seven columns as TSDPL/CPDY: `<7d / 7–30d / 30–60d / 60–90d / 90–180d / >180d / Total`
- Both tables now call `fillFGTable()` (the shared FG renderer) instead of a bespoke `fillSimpleTable()`
- Added helper `woLineTotal(woBucketed, line)` — mirrors `qhLineTotal()` — to collapse bucketed WO data to a scalar for KPI badges and charts
- All chart datasets, KPI totals, and Trends panel references updated to use the helpers; no flat `wo[l]` accesses remain

**PDF export fix — To-Date Actual**

- Root cause: the PDF builder called `buildProdTableHTML('prodToDateBody', …)` but `prodToDateBody` was removed when the dashboard was unified into a single table; the element did not exist so To-Date columns were blank in the PDF
- `buildProdTableHTML()` rewritten as a no-arg function that clones the live `prodOnDateBody` tbody and `prodOnDateTot` tfoot from the DOM
- PDF WO and QH table headers updated from the old `Line | MT` to the new 7-column bucket format

### v4.3 (May 2026)

**Planning Panel**

- New **📋 PLANNING** button in the header opens/closes a collapsible planning panel
- **Machine Targets tab** — enter monthly MT targets per line; auto-calculates Daily Target and On-Date Target; apply to dashboard with one click
- **Maintenance tab** — log planned downtime minutes, type (PM/BD/OM), scheduled date, and remarks per line
- **APU tab** — enter APU target % and actual %; variance (pp) auto-calculated live
- **Upload Plan File tab** — drag-and-drop or browse for a separate planning `.xlsx`; targets extracted from the `One Pager Report` sheet override SAP-parsed values and update the Machine Targets inputs
- Planning Panel is excluded from PDF/print output
- Planning target priority: manually applied > SAP `One Pager Report` > KPI card manual override

**Production table redesign**

- Two separate 4-column tables (On-Date / To-Date) replaced by a **single unified 8-column table**
- Columns: Lines / On Date Plan / On Date Actual / To Date Plan / To Date Actual / Expected Rate / Actual Rate / Asking Rate
- All actuals sourced from `RM` / `Production Report(RM)` sheets
- Plan/Expected Rate sourced from `One Pager Report` ABP targets (or Planning Panel)
- Actual Rate = MTD Actual ÷ days elapsed; Asking Rate = (ABP − MTD) ÷ remaining days

**Calendar modal — single-calendar redesign**

- Two side-by-side calendars replaced with a single compact calendar (`max-width: 380px`)
- Sequential click flow: first click sets Start, second click sets End (hint text guides the user)

**PDF export**

- Planning Targets & Machine Data section injected into PDF when targets have been applied
- Includes: Monthly Target, Daily Target, Maintenance summary, APU Target, APU Actual per line

### v4.2 (May 2026)

**Expected Rate fix**

- Expected Rate KPI and production table plan columns now read ABP monthly targets from the `One Pager Report` sheet header block (WCTL1 / WCTL2 / SLIT rows), instead of deriving it as MTD ÷ days elapsed (which produced the same value as Actual Rate)
- Chart daily production reference line (`Expected/day`) now uses the ABP-derived daily rate

### v4 (May 2026)

**Data correctness fixes**

- **Date parsing:** SAP `DD/MM/YYYY HH:MM:SS` strings are now parsed correctly (was being misread as MM/DD by `new Date()`)
- **FG inventory:** now filters `Category = Prime` only; reads `Location` column directly instead of using the `SENT TO CPDY` flag
- **Without-Order formula:** changed to `Status = WF AND QualityHold ≠ Y`
- **RM in-stock/transit:** now reads the `Place` column (`"RM in Stock"` / `"RM in Transit"`) instead of deprecated STATUS codes
- **Production weight:** to-date and MTD figures now use `MOTHER COIL MASS`; on-date uses `NET WT`

**KPI strip changes**

- Card 1: "Total FG" → "Last Day Prod"
- Card 2: "Total RM" → "Month-to-Date (MTD)"
- Card 3: "Today's Production" → **Expected (Monthly)** (editable MT/month KPI auto-read from SAP file)
- KPI strip is a fixed 6-column grid (responsive: 3 columns on tablet, 2 on mobile)

**Date filter — calendar modal**

- Replaced Flatpickr with zero-dependency built-in inline calendar modal
- Only dates with production data are selectable; others are greyed out
- Apply / Clear / Escape controls; button label updates to show active range

**Layout & charts**

- Added RM Coverage section: RM Age Breakdown (stacked bar) + RM In-Stock vs In-Transit
- Daily Production trend chart (line, per line + total + expected rate reference line)
- Visual Analytics: FG Split donut / FG Age Radar / Quality Risk doughnut / Production Trend line chart
- Trends Analysis section: Production Output / Shift Efficiency / Cumulative vs Target / Inventory Health / Quality & Hold Trends

**New ALIASES**

- `category`, `mother_coil_mass`, `place` added; full alias list expanded across all fields

**Sheet loading**

- Partial sheet sets load gracefully
- Available dates derived from production sheets only
- Header subtitle shows "Latest data: DD/MM/YYYY" after file load