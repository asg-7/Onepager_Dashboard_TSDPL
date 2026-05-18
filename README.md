Kalinganagar Plant — One Pager Dashboard
Browser‑only analytics dashboard for the Kalinganagar plant. Drop your SAP Excel export onto index.html, get an interactive production & inventory view, and download it as PDF. No Python, no server, no install.

Quick Start
Open index.html in Chrome or Edge (double‑click the file).

Drag and drop your SAP .xlsx file onto the drop zone, or click Choose File.

Use the Line tabs, Shift dropdown, and 📅 Select Date Range button to slice the data.

(Optional) Click 📋 PLANNING to enter monthly targets, maintenance, or APU data.

Click ⬇ DOWNLOAD REPORT – the browser’s Save as PDF dialog opens immediately.

Key Features
Zero‑install – a single HTML file, everything runs in the browser.

Drag‑and‑drop SAP processing – automatic cleaning: dedup, numeric coercion, location/category derivation.

Interactive filters – line, shift, custom date range (built‑in calendar, only data‑rich dates are clickable).

KPI strip – Total Actual On‑Date Prod, MTD, Expected (monthly), Quality Hold, Without Order, RM In‑Transit.

Unified production table – plan vs actual, daily and cumulative, expected/actual/asking rates.

FG inventory – age‑bucket breakdown for TSDPL and CPDY locations (Prime coils only).

RM status – age split (≤3 d / >3 d), in‑stock and in‑transit volumes.

Planning panel – manually enter or upload monthly targets, maintenance downtime, and APU data; applies instantly.

PDF export – print‑optimised A3 landscape, charts snapshotted as PNG, all UI chrome hidden.

Offline‑capable – SheetJS and Chart.js are bundled locally; only chart data labels need a CDN connection.

Excel File Requirements
The dashboard looks for these sheets (case‑sensitive):

Sheet name	Contents
RM	Raw material inventory
FG	Finished goods inventory
Production Report(RM)	RM production rows
Production Report(FG)	FG production rows
Fallback: if none of the above are found, it tries One Pager Report.
Monthly ABP targets are read from the One Pager Report sheet (used for plan columns and Expected Rate).

No preprocessing needed – drop the raw SAP export. The dashboard auto‑maps column headers via aliases and derives location/category from the Yard and Status columns.

Offline Use & CDN Dependency
libs/xlsx.min.js & libs/chart.min.js – bundled for offline use. If missing, the page falls back to Cloudflare CDN.

chartjs-plugin-datalabels – always loaded from jsDelivr CDN. Charts still render without it, but value labels will not appear when offline.

Google Fonts (Montserrat, Lato) – degrade gracefully to system fonts if offline.

Customising
Edit the CONFIG object at the top of the <script> block in index.html:

js
const CONFIG = {
  lines: ["WCTL1", "WCTL2", "SLIT"],
  age_buckets: { upper_bounds: [7, 30, 60, 90, 180, null] },
  sheet_names: { fg: "FG", rm: "RM", … },
  // line mappings, aliases, status codes …
};
Add new lines, change age bucket boundaries, or adjust column name aliases there.

Built With
Library	Version	Loaded from
SheetJS (xlsx)	0.18.5	local libs/ / CDN fallback
Chart.js	4.4.1	local libs/ / CDN fallback
chartjs‑plugin‑datalabels	2.2.0	jsDelivr CDN (online only)
No frameworks, no build step.

Notes
Pop‑ups must be allowed for the PDF download button.

FG inventory is automatically derived from the Yard and Status columns – no manual Excel recalculation needed.

If charts don’t appear, check that libs/ folder exists when offline.

Version 4.9 (May 2026) — see the full documentation for recent changes and the complete changelog.

