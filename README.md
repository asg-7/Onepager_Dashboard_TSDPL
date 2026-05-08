# Kalinganagar Plant — One Pager Dashboard

Browser-only analytics dashboard for the Kalinganagar plant. Drop your SAP Excel export onto `index.html`, get an interactive production & inventory view, download as PDF. No Python, no server, no install.

---

## How to use

1. Open `index.html` in Chrome or Edge (double-click it)
2. Drag and drop your SAP `.xlsx` export onto the drop zone, or click **Choose File**
3. Use the **Line tabs**, **Shift dropdown**, and **📅 Select Date Range** button to slice the data
4. Click **⬇ DOWNLOAD REPORT** — the browser's **Print / Save as PDF** dialog opens immediately

That's it. The dashboard stays open while you save the PDF; no page navigation occurs.

---

## File structure

```
onepager_dashboard_v4/
├── index.html        ← the whole dashboard, self-contained
├── libs/
│   ├── xlsx.min.js                      ← SheetJS 0.18.5  (local copy for offline use)
│   ├── chart.min.js                     ← Chart.js 4.4.1  (local copy for offline use)
│   └── chartjs-plugin-datalabels.min.js ← Chart.js DataLabels plugin 2.2.0
└── README.md
```

> **Note:** Flatpickr has been removed. Date range selection now uses a built-in, zero-dependency inline calendar modal — no external library needed.

The `libs/` folder enables offline use. If it is missing, the page auto-falls back to CDN.

---

## Excel file requirements

The dashboard looks for specific sheet tabs in this priority order:

**Primary (Separate Sheets)**

| Sheet name | Contents |
|---|---|
| `RM` | Raw materials inventory |
| `FG` | Finished goods inventory |
| `Production Report(RM)` | RM production rows |
| `Production Report(FG)` | FG production rows |

If only some sheets exist (e.g. only `RM` and `Production Report(RM)`), the dashboard loads what it can and treats missing sheets as empty.

**Fallback (Single Sheet)**  
If none of the named sheets above are found, the dashboard attempts to read from:

- `One Pager Report`

**Before uploading:** open the file in Excel, hit **Ctrl + Shift + F9** to force-recalculate all formulas, then save. If you skip this, the Location column may still contain formula strings and FG inventory will show zero.

---

## Column mapping

The dashboard auto-maps SAP column names via an `ALIASES` object in `index.html`. If a column isn't being picked up, find `ALIASES` near the top of the `<script>` block and add the variant.

| Canonical name | SAP column names accepted |
|---|---|
| Production line | `PROC Line`, `Proc Line`, `Processing Line`, `PROCLINE` |
| Net weight | `Net Wt`, `NET WT`, `Net Weight`, `MASS`, `Mass` |
| Age (days) | `Bundle Age`, `BUNDLE AGE`, `Age`, `AGE` |
| Location | `Location`, `LOCATION`, `Stor. Location` |
| Quality hold | `Quality Hold`, `QUALITY HOLD`, `QHold` |
| Order ID | `ORDER ID`, `Order ID`, `SO Number` |
| Date | `DATE CREATION`, `DATE Creation`, `Creat.Date` |
| Shift | `Shift`, `SHIFT`, `Work Shift` |
| **Category** *(new)* | `Category`, `CATEGORY`, `Cat` |
| **Mother Coil Mass** *(new)* | `MOTHER COIL MASS`, `Mother Coil Mass`, `MCM`, `Mother Coil Wt` |
| **Place** *(new)* | `Place`, `PLACE`, `Stock Place`, `Stor. Place` |

Line code mapping for FG and RM sheets (edit `CONFIG.fg_line_mapping` / `CONFIG.rm_line_mapping` in `index.html` if yours differ):

| SAP code | Line |
|---|---|
| CTL1 | WCTL1 |
| CTL2 | WCTL2 |
| WCTL1 | WCTL1 |
| WCTL2 | WCTL2 |
| SLIT | SLIT |

---

## Filters

Three filters applied simultaneously:

- **Line tabs** — All Lines / WCTL1 / WCTL2 / SLIT
- **📅 Select Date Range** — opens an inline calendar modal; only dates with data in the file are clickable
- **Shift dropdown** — All Shifts / Morning (A) / Afternoon (B) / Night (C)

### Calendar modal

Click **📅 Select Date Range** in the filter bar. The modal shows two side-by-side calendars (Start Date / End Date). Dates that have no data in the uploaded file are greyed out and non-clickable. After selecting both dates, click **Apply Range** to filter all tables and charts. Click **Clear** to remove the custom range. Press **Escape** or click outside the modal to close without applying.

The button label updates to show the active range (e.g. `07/05/2025 → 08/05/2025`) when a custom range is applied.

The badge in the header changes from `● DATA LOADED` to `● FILTERED` when any non-default filter is active. Hit **↻ RESET FILTERS** to clear everything.

---

## KPI Strip

The top strip shows 6 key metrics:

| Card | What it shows | Source |
|---|---|---|
| **Last Day Prod** | RM production MT on the most recent date in the file | `Production Report(RM)` — MOTHER COIL MASS |
| **Month-to-Date (MTD)** | Total RM production MT for the current calendar month | `Production Report(RM)` — MOTHER COIL MASS |
| **Expected Rate** | Auto-calculated MTD ÷ days elapsed. Click the value to override manually; click **↺ auto** to reset | Derived |
| **Quality Hold** | FG MT with Quality Hold = Y | `FG` sheet |
| **Without Order** | FG MT with Status = WF (and QH ≠ Y) | `FG` sheet |
| **RM In-Transit** | RM MT where Place = "RM in Transit" | `RM` sheet |

> **MTD reference date:** "Month-to-date" uses the most recent production date in the file as the reference — not today's calendar date — so the figures match SAP even when the file is opened days later.

---

## Production tables

Two tables under **Production Report**:

- **On-Date Actual** — shows RM production MT for the most recent date in the file (or the applied custom range), plus an Expected column (from Expected Rate KPI) and a vs-Expected delta column
- **To-Date Actual** — shows cumulative RM production MT, with the same Expected and delta columns

Production uses the **MOTHER COIL MASS** column for weight. PROC Line codes from `Production Report(RM)` are used directly (not remapped).

---

## FG Inventory tables

Two age-bucket tables (TSDPL and CPDY locations):

- Only **Prime category** coils are counted (rows where `Category = Prime`)
- Location is read directly from the `Location` column — not inferred from the `SENT TO CPDY` flag
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
2. Pulls the current table data and KPI values (including Expected Rate)
3. Builds a clean, print-ready HTML report and opens it in a new tab
4. The print dialog triggers automatically — set **Destination → Save as PDF** in Chrome/Edge

Print stylesheet targets **A3 Landscape** with 10 mm margins and forces colour output.

> **Note:** Allow pop-ups for this file. The temporary Blob URL is automatically revoked after printing.

`Ctrl + P` on the dashboard itself also works — the `@media print` stylesheet hides all UI chrome and reformats cards for paper.

---

## Troubleshooting

**FG inventory shows zero**  
Check two things: (1) force-recalculate in Excel (Ctrl+Shift+F9 → Save) so the Location column has values not formula strings; (2) check the `Category` column — only `Prime` rows are counted.

**All numbers zero after upload**  
Check sheet names match exactly (case-sensitive). Also check the `ALIASES` object — SAP sometimes changes column headers between exports.

**Production shows zero**  
Ensure the `MOTHER COIL MASS` column is present and populated in `Production Report(RM)`. If it's absent, the dashboard falls back to `NET WT` / `MASS`.

**RM in-stock / in-transit shows zero**  
Check the `Place` column in the `RM` sheet. Values must contain `"RM in Stock"` or `"RM in Transit"` (the match is case-insensitive and partial, so `"RM in Stock (TSDPL)"` also works).

**Date range calendar shows no selectable dates**  
Dates are sourced from the production sheets only (`Production Report(RM)` and `Production Report(FG)`). If those sheets are empty, no dates will appear.

**Dashboard opens but charts/upload don't work**  
SheetJS or Chart.js didn't load. Check that `libs/xlsx.min.js` and `libs/chart.min.js` exist. If missing and offline, the page won't function.

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
  fg_line_mapping: { CTL1:"WCTL1", CTL2:"WCTL2", WCTL1:"WCTL1", WCTL2:"WCTL2", SLIT:"SLIT" },
  rm_line_mapping: { CTL1:"WCTL1", CTL2:"WCTL2", WCTL1:"WCTL1", WCTL2:"WCTL2", SLIT:"SLIT" },
  locations: { tsdpl: "TSDPL", cpdy: "CPDY" },
  age_buckets: {
    labels: ["<7 Days","7–30 Days","30–60 Days","60–90 Days","90–180 Days",">180 Days"],
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

- **Add a new line:** add to `CONFIG.lines` and both line mappings. Tabs and filters pick it up automatically.
- **Change age bucket boundaries:** edit `CONFIG.age_buckets.upper_bounds`. Keep `labels` and `upper_bounds` the same length.
- **Change location names:** edit `CONFIG.locations.tsdpl` / `CONFIG.locations.cpdy` to match your `Location` column values.

---

## Libraries used

| Library | Version | Purpose |
|---|---|---|
| SheetJS | 0.18.5 | Parse the uploaded `.xlsx` entirely in-browser |
| Chart.js | 4.4.1 | All charts (bar, line, doughnut, radar) |
| ChartDataLabels | 2.2.0 | Value labels on top of chart bars |
| Google Fonts | — | Montserrat + Lato (falls back to system fonts offline) |

No Flatpickr. No build step. No npm. Just open `index.html`.

---

## Changelog

### v4 (May 2026)

**Data correctness fixes**
- **Date parsing:** SAP `DD/MM/YYYY HH:MM:SS` strings are now parsed correctly (was being misread as MM/DD by `new Date()`)
- **FG inventory:** now filters `Category = Prime` only; reads `Location` column directly instead of using the `SENT TO CPDY` flag
- **Without-Order formula:** changed to `Status = WF AND QualityHold ≠ Y` (was incorrectly using `orderId = '' OR status = WF`)
- **RM in-stock/transit:** now reads the `Place` column (`"RM in Stock"` / `"RM in Transit"`) instead of deprecated STATUS codes
- **Production weight:** to-date and MTD figures now use `MOTHER COIL MASS`; on-date uses `NET WT`
- **Production line mapping:** RM production rows (`Production Report(RM)`) no longer apply `rm_line_mapping` — PROC Line is used directly

**KPI strip changes**
- **Card 1:** "Total FG" → "Last Day Prod" (RM production MT on the most recent date in the file)
- **Card 2:** "Total RM" → "Month-to-Date (MTD)" (RM production MT for the current month)
- **Card 3:** "Today's Production" replaced by **Expected Rate** (editable MT/day KPI; auto-calculated as MTD ÷ days elapsed, overrideable manually)
- Removed the duplicate "Month-to-Date" card (was showing the same value twice)
- KPI strip is now a fixed 6-column grid (responsive: 3 columns on tablet, 2 on mobile)

**Date filter — calendar modal**
- Replaced Flatpickr with a zero-dependency built-in inline calendar modal
- Two-calendar layout (Start / End) with side-by-side month navigation
- Only dates with production data in the file are selectable; others are greyed out
- Range summary panel shows selected dates and count of data days in range
- Apply / Clear / Escape controls; button label updates to show active range
- Flatpickr library files (`flatpickr.min.js`, `flatpickr.min.css`) removed from `libs/`

**Production tables**
- On-Date table now shows the actual date in its title (e.g. `On-Date Actual (07/05/2025)`)
- Both On-Date and To-Date tables now include **Expected (MT)** and **vs Expected** delta columns
- "On-Date" uses the latest date in the file (not today's calendar date)

**Layout & charts**
- Added **RM Coverage** section above Production: RM Age Breakdown (stacked bar) + RM In-Stock vs In-Transit
- Production section now includes a **Daily Production trend chart** (line chart by date, per line + total + expected rate reference line)
- Quality Hold table relocated next to the Daily Production chart
- Removed the old "Charts Row 1" three-panel grid (WO+QH chart, RM chart, Prod chart)
- Visual Analytics section now shows: FG Split donut / FG Age Radar / Quality Risk doughnut + Production Trend line chart
- Chart title updated: "Production Trend — Last Day vs MTD (RM)"

**New ALIASES**
- `category` → `Category`, `CATEGORY`, `Cat`, etc.
- `mother_coil_mass` → `MOTHER COIL MASS`, `MCM`, `Mother Coil Wt`, etc.
- `place` → `Place`, `PLACE`, `Stock Place`, `Stor. Place`, etc.

**Sheet loading**
- Partial sheet sets now load gracefully (e.g. file has `RM` but not `FG` — loads what is available instead of erroring)
- Available dates derived from production sheets only (not FG/RM inventory sheets)
- Header subtitle shows "Latest data: DD/MM/YYYY" after file load
