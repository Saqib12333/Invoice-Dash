Invoice-Dash

Overview
- This project will provide a Streamlit dashboard for operational visibility into monthly billable hours for remote medical scribing services.
- Source data is the Excel workbook `Invoice Sheet.xlsx`, specifically the tab named "All Clients". Individual client tabs are not used.

Data mapping ("All Clients" tab)
- Row 1: empty; Row 2: headers; data begins on Row 3.
- Columns:
	- Account (short code)
	- Facility (client/clinic/hospital name)
	- Billing Type (FTP or Hourly)
	- No. of Providers and Assigned Scribes (integer)
	- January … December: monthly billable hours (numbers). As of Aug, Sep–Dec are blank.
	- Average/Provider: latest month hours divided by current providers (precalculated in sheet).
- The last row is a Total aggregation row; it’s excluded from client-level analytics.

Confirmed scope and assumptions
- Only the "All Clients" sheet is considered.
- One row per client for the whole year.
- One provider is associated to exactly one client.
- No revenue estimates will be computed.

Environment
- `.env` contains a OneDrive view-only link to the sheet: `SHEET_LINK=...`
- `.env.example` provides the variable name; copy it to `.env` and set the value if needed.

Planned dashboard content (high level)
- KPIs for the latest month (currently August): total hours, month-over-month change, active clients, total providers, weighted hours per provider, and billing-type mix.
- Trends: total hours by month; split by billing type; client × month heatmap.
- Client leaderboards: top/bottom by latest month hours, YTD hours, and hours per provider.
- Health checks: zero hours in the latest month but earlier activity; large MoM swings.

Getting started (development)
1) Ensure Python 3.9+ is installed.
2) Install dependencies and run the app (to be added in subsequent commits).
3) Place `Invoice Sheet.xlsx` at the project root (or configure the app to read from the OneDrive link when supported).

Notes
- Sep–Dec columns will be empty until months occur; the app treats blanks as 0 for totals and excludes them when detecting the “latest month”.
- Providers may be 0 for some rows; per-provider metrics will guard against divide-by-zero.
