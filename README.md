# 🏭 Kalinganagar Plant — One Pager Dashboard

> A zero-install, browser-only analytics dashboard for SAP production & inventory data.

---

## Problem Statement

Plant operations teams at Kalinganagar needed a fast way to visualise daily SAP exports — production output, raw material status, and finished goods inventory — without relying on a server, Python environment, or IT setup. Existing workflows required manual copy-pasting into Excel and produced static, hard-to-share snapshots with no filtering or trend visibility.

**Key pain points:**
- SAP exports raw `.xlsx` files with no built-in analytics view
- No lightweight tool for shift/line/date-level slicing
- PDF reports had to be assembled manually every day
- Offline use was a hard requirement on the shop floor

---

## Solution

A fully self-contained `index.html` dashboard that runs entirely in the browser. Drop your SAP Excel export onto the page and instantly get interactive charts, KPI cards, production tables, and inventory age-buckets — then export a clean A3 PDF in one click.

```
onepager_dashboard_v4/
├── index.html        ← entire dashboard, self-contained
├── libs/
│   ├── xlsx.min.js                       SheetJS 0.18.5
│   ├── chart.min.js                      Chart.js 4.4.1
│   └── chartjs-plugin-datalabels.min.js  DataLabels plugin 2.2.0
└── README.md
```

### How it works

1. **Open** `index.html` in Chrome or Edge (double-click, no server needed)
2. **Drop** your SAP `.xlsx` export onto the upload zone
3. **Filter** by Line tab, Shift, or custom Date Range using the built-in calendar modal
4. **Export** — click ⬇ DOWNLOAD REPORT to get a print-ready A3 PDF

---

## Tech Stack

| Layer | Technology | Why |
|---|---|---|
| **Parsing** | [SheetJS 0.18.5](https://sheetjs.com/) | Parse `.xlsx` entirely client-side, no server upload |
| **Charts** | [Chart.js 4.4.1](https://www.chartjs.org/) | Bar, line, doughnut, and radar charts |
| **Data labels** | ChartDataLabels 2.2.0 | Value annotations on chart bars |
| **Date picker** | Custom zero-dependency modal | Replaced Flatpickr; only dates with data are selectable |
| **Export** | Browser Print API + Blob URL | Snapshot canvases → print-ready HTML → PDF via browser |
| **Fonts** | Google Fonts (Montserrat + Lato) | Falls back to system fonts offline |
| **Runtime** | Vanilla JS — no build step, no npm | Works offline on any machine with Chrome/Edge |

---

## Key Features

**KPI Strip** — 6 live metrics: Last Day Production, MTD, Expected Rate (editable), Quality Hold MT, Without-Order MT, RM In-Transit MT

**Production Tables** — On-Date and To-Date actuals with Expected and vs-Expected delta columns

**FG Inventory** — Age-bucket breakdown (≤7 d / 7–30 d / 30–60 d / 60–90 d / 90–180 d / >180 d) for TSDPL and CPDY locations; Prime category only

**RM Status** — In-Stock vs In-Transit split per line, with ≤3 day / >3 day age breakdown

**Charts** — Daily production trend, RM age stacked bar, FG split donut, quality risk doughnut, FG age radar

**Smart date parsing** — Handles SAP's `DD/MM/YYYY HH:MM:SS` format correctly (avoids JS `new Date()` MM/DD misread)

**Offline-first** — All libraries bundled in `libs/`; CDN fallback if folder is missing

---

## Screenshot

> *Drop your SAP file → filter by line/shift/date → export PDF. No install. No server.*

---

## Quick Start

```bash
# No install required — just open the file
open index.html          # macOS
start index.html         # Windows
xdg-open index.html      # Linux
```

**Before uploading your SAP file:** open it in Excel and press `Ctrl + Shift + F9` to force-recalculate formulas, then save. This ensures the `Location` column is populated.

---

## Configuration

All tuneable settings live in the `CONFIG` object at the top of the `<script>` block in `index.html`:

```js
const CONFIG = {
  lines: ["WCTL1", "WCTL2", "SLIT"],
  age_buckets: {
    labels: ["<7 Days", "7–30 Days", "30–60 Days", "60–90 Days", "90–180 Days", ">180 Days"],
    upper_bounds: [7, 30, 60, 90, 180, null],
  },
  locations: { tsdpl: "TSDPL", cpdy: "CPDY" },
  // ...
};
```

Column name variants (e.g. `NET WT` vs `Net Weight`) are handled via an `ALIASES` object in the same file.

---

## Changelog — v4 (May 2026)

- Fixed SAP date parsing (`DD/MM/YYYY` misread as `MM/DD`)
- FG inventory now filters `Category = Prime` only; reads `Location` column directly
- RM status now uses `Place` column (`"RM in Stock"` / `"RM in Transit"`)
- Production weight switched to `MOTHER COIL MASS`
- New KPIs: Last Day Prod, MTD, Expected Rate (replaces old Total FG / Total RM cards)
- Replaced Flatpickr with zero-dependency built-in calendar modal
- Added RM Coverage section (age stacked bar + in-stock vs in-transit chart)
- Added Daily Production trend line chart with expected rate reference line
- Partial sheet sets now load gracefully instead of erroring

---

## License

Internal tool — Kalinganagar Plant Operations.
