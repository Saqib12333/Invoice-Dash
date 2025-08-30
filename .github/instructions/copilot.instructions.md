# Invoice-Dash — Project Working Notes (for developers & AI agents)

Authoritative scope: Only the worksheet/tab named "All Clients" in `Invoice Sheet.xlsx`.

Data source
- Primary file: `Invoice Sheet.xlsx` (local copy). There is also a OneDrive view-only link in `.env` → `SHEET_LINK`.
- The dashboard must load only the "All Clients" sheet.
- Row 1 is empty (visual spacing); Row 2 has headers; data starts from Row 3.
- Column A is intentionally blank (visual spacing); headers begin at column B.
- There is a final "Total" row (sums) — exclude from client-level analytics.
- Currently 17 client rows; more may be added (the app must be dynamic).

Column mapping (confirmed from screenshot)
- A: (blank column — ignore)
- B: Account (short code)
- C: Facility (client/clinic/hospital name)
- D: Billing Type (values: `FTP` or `Hourly`)
- E: No. of Providers and Assigned Scribes (integer; providers are 1-to-1 associated with one client)
- F–Q: Monthly billable hours (Jan–Dec). From Jan to Aug have values; Sep–Dec currently blank as of Aug.
  - Month columns by header text: January, February, March, April, May, June, July, August, September, October, November, December.
  - Columns after August (Sep–Dec) may be empty but still exist.
- R: Average/Provider (formula, equals most recent month billed hours ÷ current providers). For now, this is effectively August/E if August is the latest non-empty month.

Important ingestion rules
- Use header names to locate columns (not absolute letters); be resilient to column re-ordering.
- Treat numeric blanks as 0 for monthly hours; preserve true 0s.
- Drop the "Total" aggregation row from client-level analysis; keep a separate aggregate computed in code.
- Providers (D) must be > 0 to compute per-provider metrics; when 0 or NaN, set per-provider to NaN and don’t divide by zero.
- Billing Type is categorical with two values: `Hourly` and `FTP`.
- Data may expand with more clients; code should handle arbitrary row counts.

Definitions
- Active client (YTD): any hours > 0 in months with data (Jan–Aug currently).
- YTD hours (client): sum of monthly hours from first month with data through latest month populated.
- Latest month: the rightmost month column with any non-null data; currently expected to be "August".
- MoM change (client): Latest − Previous (by month); MoM% = (Latest − Previous) / max(|Previous|, 1e-9).
- Weighted avg hours per provider (month): sum(hours) / sum(providers) among clients with providers > 0.

Core metrics for the dashboard (planning)
- KPIs (latest month): total hours; MoM abs and %; active clients; total providers; weighted hours/provider; billing type mix by hours and clients.
- Time-series: total hours by month; split by billing type; heatmap client × month; per-client MoM deltas (top movers).
- Client leaderboard: top/bottom by latest month hours, YTD hours, and hours/provider (latest month).
- Provider-normalized views: distribution of hours/provider; scatter providers vs hours/provider; group by billing type.
- Health flags: zero hours in latest month but non-zero before; ±30% MoM changes; persistently low hours/provider (no target currently).

App constraints & assumptions
- No revenue calculations (no rates provided).
- One provider is associated with one client (no overlap across clients).
- One row per client for the whole year.
- Only the "All Clients" sheet is used; ignore individual client tabs.

Environment
- `.env` contains `SHEET_LINK` (view-only). For local development, read from the local Excel file; for deployment, consider reading from a URL if direct download is enabled. If not downloadable, require a local file.

Data loading strategy (Python/Streamlit)
- Use `pandas.read_excel(file, sheet_name="All Clients", header=1, usecols=lambda c: not str(c).startswith('Unnamed'))` to skip the blank first column (header row index 1 because row 2 is headers, zero-based indexing). Alternatively, read then drop columns matching `^Unnamed`.
- Normalize column names (strip, title case), e.g., `Facility`, `Billing Type`, `No. Of Providers And Assigned Scribes`, etc.
- Identify month columns via a fixed ordered list of month names, intersected with the dataframe columns.
- Detect latest month as the last month with any non-null values across clients.
- Exclude the row where `Facility == 'Total'` (or where `Account` is blank and Facility equals 'Total'). Use robust logic: if a row’s Facility equals 'Total' and numeric month sums equals the column sums, drop it.

Edge cases & data hygiene
- Providers = 0 or NaN: skip per-provider metrics; guard against divide-by-zero.
- Mixed types in month columns: coerce to numeric with `errors='coerce'` then fillna(0) for hours.
- Duplicate clients: currently none; if encountered, aggregate by Facility for rollups but show duplicates in detailed table.
- Leading/trailing spaces: trim all string columns.

Repo layout (current)
- Root: `Invoice Sheet.xlsx`, `.env`, `.env.example`, `README.md`
- `.github/instructions/copilot.instructions.md` (this file)
- Future code: `app/` for Streamlit app and `requirements.txt`.

Security & privacy
- No PII beyond facility names.
- Do not commit credentials. `.env` example should not have real URLs if private.

Next steps (after planning acceptance)
- Implement Streamlit app scaffolding with data loader and cached reading of Excel.
- Add metrics calculations and visualizations (Plotly/Altair).
- Add filters (billing type, client search) and downloads.
- Write basic tests for data parsing and month detection.
