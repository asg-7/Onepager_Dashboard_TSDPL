# Kalinganagar Plant — One Pager Dashboard

Browser-only analytics dashboard for the Kalinganagar plant. Drop your SAP Excel export onto `index.html`, get an interactive production & inventory view, download as PDF. No Python, no server, no install.

---

## How to use

1. Open `index.html` in Chrome or Edge (double-click it)
2. Drag and drop your SAP `.xlsx` export onto the drop zone, or click **Choose File**
3. Use the line tabs and period pills to slice the data
4. Click **⬇ DOWNLOAD REPORT** — the browser's **Print / Save as PDF** dialog opens immediately

That's it. The dashboard stays open while you save the PDF; no page navigation occurs.

---

## File structure

```
onepager_dashboardv3/
├── index.html        ← the whole dashboard, self-contained
├── libs/
│   ├── xlsx.min.js   ← SheetJS 0.18.5  (local copy for offline use)
│   └── chart.min.js  ← Chart.js 4.4.1  (local copy for offline use)
└── README.md
```

The `libs/` folder is what makes it work offline. If it's missing, the page falls back to loading from CDN automatically — so it still works as long as there's internet.

---

## Excel file requirements

Your SAP export needs to have these four sheet tabs, named exactly like this:

- `FG` — finished goods inventory
- `RM` — raw materials inventory
- `Production Report(FG)` — FG production rows
- `Production Report(RM)` — RM production rows

**Before uploading:** open the file in Excel, hit **Ctrl + Shift + F9** to force-recalculate, then save. If you skip this, the Location column may still contain formula strings instead of values, and FG inventory buckets will show zero.

---

## Column mapping

The dashboard auto-maps SAP column names using an alias list in `index.html`. If a column isn't being picked up, open `index.html`, find the `ALIASES` object near the top of the `<script>` block, and add the variant there.

Common ones that vary between SAP exports:

| What the dashboard looks for | SAP column names it accepts |
|---|---|
| Production line | `PROC Line`, `Proc Line`, `Processing Line`, `PROCLINE` |
| Net weight | `Net Wt`, `NET WT`, `Net Weight`, `MASS`, `Mass` |
| Age (days) | `Bundle Age`, `BUNDLE AGE`, `Age`, `AGE` |
| Location | `Location`, `LOCATION`, `Stor. Location` |
| Quality hold | `Quality Hold`, `QUALITY HOLD`, `QHold` |
| Order ID | `ORDER ID`, `Order ID`, `SO Number` |
| Date | `DATE CREATION`, `DATE Creation`, `Creat.Date` |
| Shift | `Shift`, `SHIFT`, `Work Shift` |

RM line codes get mapped like this (edit `CONFIG.rm_line_mapping` in `index.html` if yours differ):

| SAP code | Line |
|---|---|
| CTL1 | WCTL1 |
| CTL2 | WCTL2 |
| SLIT | SLIT |

---

## Filters

Three filters, all applied simultaneously:

- **Line tabs** — All Lines / WCTL1 / WCTL2 / SLIT
- **Period pills** — All Time / Today / This Month / 📅 Date Range (custom date picker with clickable date chips)
- **Shift dropdown** — All Shifts / Morning (A) / Afternoon (B) / Night (C)

When **📅 Date Range** is selected, a date panel slides open. You can:
- Type or pick a **From** and **To** date
- Click any date chip to build the range interactively
- Press **APPLY** to update all charts and tables

The badge in the header changes from `● DATA LOADED` to `● FILTERED` when any non-default filter is active. Hit **↻ RESET FILTERS** to clear everything.

---

## PDF / Print export

Click **⬇ DOWNLOAD REPORT**. The dashboard:

1. Snapshots all Chart.js canvases to PNG images
2. Pulls the current table data and KPI values
3. Builds a clean, print-ready HTML report
4. Generates a temporary, in-memory Blob URL and opens it in a new tab, automatically triggering the browser's **Print dialog**

In the Print dialog, set the **Destination** to **Save as PDF** (Chrome/Edge). The print stylesheet targets **A3 Landscape** with 10 mm margins and forces colour output.

> **Note:** A new tab will open for the report and the print dialog will appear. Make sure your browser allows pop-ups for this file. The temporary Blob URL is automatically revoked to save memory.

Alternatively, `Ctrl + P` on the dashboard itself also works — the `@media print` stylesheet hides all UI chrome (header buttons, filter bar, line tabs) and reformats cards for paper.

---

## Troubleshooting

**FG inventory shows zero**
Almost always the Location column. Open in Excel → Ctrl+Shift+F9 → Save → re-upload.

**All numbers zero after upload**
Check the sheet names match exactly (case-sensitive). Also check the `ALIASES` object in `index.html` — SAP sometimes changes column headers between exports.

**Dashboard opens but charts/upload don't work**
Means SheetJS or Chart.js didn't load. Check that `libs/xlsx.min.js` and `libs/chart.min.js` exist. If they're missing and you have no internet, the page won't function.

**DOWNLOAD REPORT button does nothing / print dialog doesn't open**
Data must be loaded first. Check your browser's address bar for a "Pop-up blocked" icon. Allow pop-ups for the page and click the button again. The new window must be allowed to open to trigger the print dialog.

**Shift filter returns no data**
Your SAP export probably uses different shift codes. Check what's in the SHIFT column and add those values to the `ALIASES.shift` list in `index.html`.

**Upload doesn't work the second time without resetting**
Click **↺ NEW FILE** in the header first. The processing lock won't release until you reset.

---

## Changing config (lines, buckets, etc.)

Everything lives in the `CONFIG` object at the top of the `<script>` block in `index.html`:

```js
const CONFIG = {
    lines: ["WCTL1", "WCTL2", "SLIT"],
    rm_line_mapping: { CTL1: "WCTL1", CTL2: "WCTL2", SLIT: "SLIT" },
    status_codes: {
        in_stock: ["IS", "SC", "1C", "2C"],
        in_transit: ["IB", "RQ", "IN"]
    },
    age_buckets: {
        labels: ["<7 Days", "7–30 Days", "30–60 Days", "60–90 Days", "90–180 Days", ">180 Days"],
        upper_bounds: [7, 30, 60, 90, 180, null]
    },
    sheet_names: {
        fg: "FG",
        rm: "RM",
        prod_fg: "Production Report(FG)",
        prod_rm: "Production Report(RM)"
    }
}
```

- **Add a new line:** add it to `CONFIG.lines` and (if the RM code differs) to `CONFIG.rm_line_mapping`. Tabs and filters pick it up automatically on next file load.
- **Change age bucket boundaries:** edit `CONFIG.age_buckets.upper_bounds`. Keep labels and bounds the same length and order.
- **Add new RM stock/transit status codes:** edit `CONFIG.status_codes.in_stock` and `CONFIG.status_codes.in_transit`.

---

## Libraries used

| Library | Version | Purpose |
|---|---|---|
| SheetJS | 0.18.5 | Parse the uploaded .xlsx entirely in-browser |
| Chart.js | 4.4.1 | All charts (bar, line, doughnut, radar) |
| Google Fonts | — | Montserrat + Lato (gracefully falls back to system fonts offline) |

No build step, no npm, no dependencies to install. Just the two files in `libs/`.
