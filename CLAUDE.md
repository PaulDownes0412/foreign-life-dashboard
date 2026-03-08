# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a single-file static HTML dashboard (`foreign_life_dashboard.html`) that visualizes Q3 2025 OSFI foreign life insurance company data. No build step, no dependencies to install — open the file in a browser to run it.

## Running the Dashboard

Open `foreign_life_dashboard.html` directly in a browser. When served from `file://`, CORS restrictions may block the OSFI API fetch; in that case the page automatically falls back to the embedded `EMBEDDED_TOP20` snapshot baked into the script.

To avoid CORS issues and test live data fetching, serve with a local HTTP server:

```bash
# Python
python -m http.server 8080

# Node (npx)
npx serve .
```

## Architecture

Everything lives in one file with three logical sections:

**Data layer** (`loadData` function, ~line 185)
- Fetches 6 parallel API calls to the OSFI Open Government Canada CKAN datastore (`open.canada.ca/data/en/api/3/action/datastore_search`)
- Two resources: `LF1_RESOURCE` (balance sheet — assets, liabilities) and `LCQ_RESOURCE` (LICAT capital ratios)
- Companies are keyed by `Id` field; the aggregate row (`AGGREGATE_ID = '1000009'`) is excluded
- Insurance contract liabilities = `insContractExclSeg` + `insContractSegFunds` (summed across two DPA codes)
- On any fetch failure, falls back to `EMBEDDED_TOP20` (hardcoded Q3 2025 snapshot)

**Chart layer** (`buildChartAssets`, `buildChartICL`, `buildChartRatios`, ~lines 258–381)
- Uses Chart.js 4.4.4 (CDN)
- All three charts are horizontal bar charts (`indexAxis: 'y'`)
- Data is reversed before passing to Chart.js so the highest-value company appears at the top

**Table layer** (`buildTable`, ~line 383)
- Renders a plain HTML `<table>` from the same `top20` array

## Key Constants

| Constant | Purpose |
|---|---|
| `LF1_RESOURCE` | CKAN resource ID for LF1 (balance sheet) dataset |
| `LCQ_RESOURCE` | CKAN resource ID for LCQ (LICAT ratios) dataset |
| `LF1_DPA` / `LCQ_DPA` | Data Point Address codes that identify specific metrics within each resource |
| `AGGREGATE_ID` | Row ID for the industry aggregate — always excluded from company list |
| `EMBEDDED_TOP20` | Hardcoded fallback data; update this when refreshing to a new reporting quarter |

## Updating for a New Quarter

1. Change `'Fiscal Quarter': 'Q3 - 2025'` in `FILTER_BASE` to the new quarter string
2. Refresh the embedded fallback: run the dashboard with a live server, copy the fetched data, and update `EMBEDDED_TOP20`
3. Update the title, header, and footer quarter references
