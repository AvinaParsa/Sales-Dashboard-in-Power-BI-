# Advertising Production Management Dashboard (Power BI)

An end-to-end business intelligence project built for a signage and advertising production company (signs, stickers, mesh banners, and related materials). The company had been tracking every job in a Google Sheet with no real reporting layer on top of it — just rows of data and manual status checks. This project turns that raw operational log into a management dashboard the business owner can actually use to run the company.

Company and client names in this repository have been anonymized (Company A, Company B, etc.) for confidentiality. The underlying data is real production data from an active business.

## Why this project is different from a typical "dashboard tutorial" project

Most portfolio dashboards start from a clean CSV. This one didn't. The source data came from a spreadsheet that had been filled in manually by different people over more than a year, and it showed. A large part of the actual work here wasn't chart design — it was figuring out what the data was actually telling me versus what it looked like it was telling me at first glance. A few examples:

- **Persian dates stored as fake ISO timestamps.** Every date field looked like `1404-04-21T20:34:16.000Z` — a Jalali (Persian) calendar date wrapped in a Gregorian-looking format. Power BI has no native Jalali calendar support, so I wrote and validated a Jalali-to-Gregorian conversion function from scratch in Power Query M, cross-checked it against a Python reference implementation before trusting it in the model.
- **One card number, multiple physical items.** A single job (e.g. four storefront stickers installed at four different spots) shares one order number but is split across multiple rows (`404042201`, `404042201-2`, `404042201-3`...). Counting rows would have overcounted projects by a wide margin, so the model distinguishes between "row" and "project" throughout — every project-level metric filters down to the primary row only.
- **A self-inflicted bug that skewed an entire metric.** While patching one bad date entry, a blanket `Replace Values` operation accidentally back-filled hundreds of legitimately empty shipping dates with a placeholder date. The average "days to shipping" metric came out wildly negative before I traced it back to that step and removed it. Left as a reminder in the query steps of how easy it is to silently corrupt a metric with a well-intentioned quick fix.
- **Same province, five different spellings.** Iran's provinces were entered inconsistently across the sheet — Arabic vs. Persian character variants (`ي` vs `ی`, `ك` vs `ک`), trailing spaces, and missing spaces in compound names. This alone was breaking every geographic breakdown, including the map, until it got normalized in Power Query.
- **Boolean fields that silently became text.** Several `TRUE`/`FALSE` fields turned into text after value replacement, which DAX only surfaced as a cryptic comparison error much later — a useful lesson in checking column types after every transformation, not just at the end.
- **Data quality as a reporting output, not just a cleanup step.** Some inconsistencies were real signals, not just typos — for example, projects marked as "installed" with no shipping or delivery date logged. Rather than quietly fixing those, they're surfaced as their own metric so the business can see where its own data entry process is breaking down.

## What the dashboard covers

**Page 1 — Overview**
Total projects, cancellation rate, average time to installation, current project status breakdown, monthly trend of new projects vs. cancellation rate, and a province-level map of Iran (including a manually reconstructed boundary for Alborz province, which is missing from most open-source Iran GeoJSON datasets since it split from Tehran province in 2010).

**Page 2 — Process & Sales Performance**
A funnel of how many projects reach each approval stage (measurement → pre-invoice approval → design → mockup → finance → production → shipping → delivery → installation), a breakdown of at which stage cancelled projects actually dropped off, stage-by-stage lead time analysis, and per-salesperson performance (project volume, cancellation rate, install rate).

**Page 3 — Data Quality & Market Mix**
The data quality findings described above, a treemap of order volume by client with material mix on hover, and material share of total production volume (by square meterage, not just order count, since a single order can range from one square meter to a hundred).

## Approach

- **Power Query / M** for all data cleaning: custom Jalali date conversion, project-vs-row grain handling, text normalization, error-safe transformations (`try...otherwise`), and a helper table anonymizing client names by rank.
- **DAX** for the analytical layer: `CALCULATE` with multiple filter conditions, `ALLEXCEPT` for share-of-parent calculations, `SWITCH(TRUE(), ...)` for multi-condition classification (e.g. determining which approval stage a cancelled project last reached), and defensive patterns like `DIVIDE` and `COALESCE` to keep visuals from breaking on edge cases.
- **Data modeling** decisions made explicit rather than assumed — most measures exist in a "row-level" and "project-level" pair, and the report is deliberately built so a viewer can't accidentally double-count a multi-item order.
- **Custom theme and layout**: a dark navy/burgundy corporate theme built as a Power BI theme JSON file, a custom gradient page background, and manually positioned KPI cards and navigation buttons rather than relying on default auto-layout.

## Tools

Power BI Desktop (Power Query, DAX, Data Modeling), Python (for validating the date conversion logic and initial data checks).

## Screenshots

See the `/screenshots` folder for full-resolution images of all three report pages.

---

*Company and client identities have been anonymized. No raw source data or `.pbix` file is included in this repository — see the note in the repo description for details.*
