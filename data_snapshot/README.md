# CubeReach demonstration: derived data snapshot

This folder holds the derived, pseudonymised datasets behind Section 4.3 (Demonstration) of the CubeReach report by Tommy Kiprillis (FIT5145 Foundations of Data Science, Monash University, Semester 2 2026). The R Markdown file submitted with the report, `CubeReach_Demonstration.Rmd`, reproduces every result from these files. Knitted on its own, it reads them from this repository automatically; to read a local copy instead, knit it with `params = list(data_source = "snapshot", snapshot_path = "<path to this folder>")`. The Rmd and the extraction script described below are part of the submitted analysis and are not stored in this repository.

## WCA attribution notice

This information is based on competition results owned and maintained by the World Cube Association, published at https://worldcubeassociation.org/results as of 13 October 2026.

## Sources and export dates

| Source | Export used | Official download |
|---|---|---|
| WCA Results Export, version 2.0.2 (TSV) | `WCA_export_v2_286_20261013T065024Z.tsv.zip`, exported 2026-10-13 06:50:24 UTC (380,907,657 bytes) | https://www.worldcubeassociation.org/export/results/v2/tsv |
| WCA Developer Database Export (MySQL dump) | `wca-developer-database-dump.zip`, dump completed 2026-10-13 00:35:55 UTC (832,374,626 bytes) | https://www.worldcubeassociation.org/wst/wca-developer-database-dump.zip |

Both official links always serve the latest export, which changes every few days, and the WCA's file server deletes old exports: the exact 13 October 2026 files carry an expiry date of 21 October 2026. This snapshot therefore fixes the 13 October 2026 versions for anyone reproducing the analysis later. Anyone may re-publish the results export under the WCA's "Allowed Use" terms, provided the notice above accompanies it. The developer export is a sanitised dump that the WCA publishes for developers: it fakes every date of birth as 1954-12-04, replaces emails, and exports registration histories empty.

## Governance

* **No names.** The code never read name columns from either export. The Python extractor (`scripts/extract_developer_tables.py`) never writes names, emails, dates of birth, comments or IP addresses.
* **Pseudonymised IDs.** Salted SHA-256 hashes (`person_hash`) replace WCA IDs. The salt stays out of this folder, so nobody can reverse the hashes by recomputing them from public WCA IDs.
* **Small cells suppressed.** In every aggregated table, counts describing fewer than ten people are blank (`NA`), and so are rates that would reveal them by subtraction.
* **Gender** appears only in `cohort_features.csv`, and only for the fairness audit. It is never a model predictor.
* **No age.** The code never reads or derives a date of birth.
* **Re-identification.** `cohort_features.csv` is the only person-level file. It keeps the debut year and month but drops the exact date and the debut competition ID. Even so, someone cross-referencing the public WCA results could still link many rows to named people, because debut month, state, competition size and performance are all public. The file adds nothing beyond what the public export already publishes, but users must not combine it with other data to single people out.

## Files and data dictionary

### `cohort_features.csv` (11,552 rows; one row per Australian debutant, 2012 to 13 September 2025)

| Column | Meaning |
|---|---|
| `person_hash` | Salted SHA-256 hash of the WCA ID |
| `split` | `train` (debuts 2015 to 2019 and 2022), `test` (2023 and 2024), `descriptive only` (2012 to 2014 and 2025), or `excluded (COVID-19 overlap)` (12-month follow-up overlapped 16 March 2020 to 31 December 2021) |
| `debut_year`, `debut_month`, `debut_quarter` | When the debut competition started |
| `returned_12m` | 1 if the person competed again, anywhere, at a competition starting after the debut ended and within 365 days of its start |
| `return_day` | Days from the debut's start to that first return (blank if none) |
| `n_events` | Events entered at the debut |
| `debut_comp_size` | Number of competitors at the debut competition |
| `perf_pct_333` | Percentile of the best 3x3x3 average at debut among same-year Australian debutants (1 = fastest; blank if no valid average) |
| `debut_state` | State or territory of the debut venue (`ozmaps` boundaries) |
| `debut_regional` | 1 if the debut venue lies more than 100 km from the nearest state or territory capital |
| `debut_dist_capital_km` | Great-circle distance from the debut venue to the nearest capital (km) |
| `opps_150km_12m`, `opps_100km_12m`, `opps_200km_12m` | Australian competitions within 150, 100 or 200 km of the debut venue starting after the debut ended and within 365 days of its start |
| `opps_known_150km_12m` | As `opps_150km_12m`, counting only competitions already announced when the debut ended (developer export `announced_at`) |
| `wait_days_150km` | Days from the debut's end to the first competition within 150 km (365 if none) |
| `none_150km_12m` | 1 if no competition fell within 150 km in that window |
| `dist_next_km` | Distance to the nearest Australian competition starting within 180 days after the debut (km) |
| `access_index` | Simplified enhanced two-step floating catchment area (E2SFCA) index: capacity per local active competitor, weighted by Gaussian distance decay (100 km bandwidth, 150 km catchment) and a 90-day waiting-time half-life |
| `access_index_100`, `access_index_200` | The same index with 100 km or 200 km catchments |
| `access_index_known` | The 150 km index counting only competitions announced by the debut's end |
| `local_comps_m01` to `local_comps_m12` | Competitions within 150 km starting in each 30-day month after the debut (month 12 covers days 331 to 365) |
| `gender` | `m`, `f`, `o` or `unknown`, as recorded in the WCA export; fairness audit only |

### `competitions_au.csv` (563 rows; Australian competitions held by the export date, not cancelled, with results)

`competition_id`, `city_name`, `state`, `start_date`, `end_date`, `latitude`, `longitude` (decimal degrees, WGS84), `nearest_capital`, `dist_capital_km`, `regional` (as above), `n_competitors`, `e2sfca_ratio_150` (step-1 E2SFCA supply-to-demand ratio), `announced_lead_days` (days from announcement to start). All fields describe public events, not people.

### `registrations_by_competition.csv` (469 rows; capped Australian competitions using WCA registration, 2019 to the export date)

`competition_id`, `start_date`, `region_type` (capital-city region or regional), `nearest_capital`, `competitor_limit`, `n_competing_registrations` (registrations with `is_competing = 1`, any final status), `oversubscription_ratio` (registrations divided by the limit), `reached_cap` (registrations at least equal to the limit), `hours_to_cap` (from `registration_open` to the registration that reached the limit; blank if the cap was not reached or the opening time was invalid), `invalid_opening_time` (cap reached before the recorded opening time), `registrations_after_cap` (registrations created after the cap was reached; blank if fewer than ten).

### Aggregated tables

| File | Contents |
|---|---|
| `registrations_summary.csv` | Share of competitions reaching their cap, median hours to cap, oversubscription, registrations after the cap and the newcomer share of them, overall, by region type and by capital-city region |
| `registrations_newcomer_followup.csv` | Newcomers (no WCA result before the competition) by whether they registered before or after the cap was reached and whether they were accepted, with the share who competed anywhere within 12 months |
| `registration_timing_global.csv` | Time to reach the cap for capped competitions since 2019: Australia, the rest of the world and the world |
| `registrations_checks.csv` | Registration row counts (1,241,868 worldwide; 63,500 for Australian competitions) and timing checks |
| `cohort_flow.csv` | Cohort construction and row-count reconciliation |
| `data_quality_checks.csv` | Data-quality checks on the results export and the action taken |
| `table2_verification.csv` | Verified values for the size figures in the report's Table 2 |
| `raw_structure.csv` | Rows and columns read from each raw table, and the personal columns never read |
| `provenance.csv` | Export file names, dates, format versions, sizes and URLs |

## Derivation

1. `CubeReach_Demonstration.Rmd` (raw mode) reads the competitions, persons and results tables of the results export, keeping only the columns it needs and never the name columns.
2. `scripts/extract_developer_tables.py` streams the developer dump out of its zip and writes Australian competitions, registrations and minimal user fields to `data_raw/` (never published), plus one aggregated timing row per competition worldwide.
3. The Rmd builds the cohort (Section 4), the access features (Section 5) and the registration measures (Section 8), hashes IDs, suppresses small cells and writes this folder. Every later section of the Rmd analyses only these files, so knitting in snapshot mode reproduces the report's numbers exactly.

Prepared on 13 October 2026. Software: R 4.6.1 with tidyverse, sf, geosphere, ozmaps, rpart, ranger and pROC; Python 3.13 for the extraction step only.
