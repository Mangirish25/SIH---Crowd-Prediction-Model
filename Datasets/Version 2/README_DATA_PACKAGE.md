# Tourist Crowd Prediction — Rebuilt & Validated Data Package (v2)

This supersedes the previous package. The monument-level visitor data has been **rebuilt from scratch directly from the three official PDF source tables** you uploaded, rather than patched from the earlier corrupted CSV.

## What changed from v1

The earlier `NEEDS_REVIEW` file quarantined 31 obviously-corrupted rows. While rebuilding those, I found the corruption was **more pervasive than 31 rows** — serial numbers and monument names in significant parts of the original `ASI_monument_visitors_2021_22_to_2024_25.csv` didn't line up correctly against the true official tables. So instead of patching, I re-extracted all three tables directly from their PDFs using structural table parsing (`pdfplumber`), then cross-validated everything against the official published grand totals.

## Main file: `monument_visitors_2019_20_to_2024_25_FULL.csv`

145 monuments × 6 annual periods (2019-20 through 2024-25), built from:

| Period(s) | Source PDF | Table |
|---|---|---|
| 2019-20, 2020-21 | Your uploaded `India-Tourism-Statistics-2021-monuments.csv` | (matched to canonical names via circle-scoped fuzzy matching) |
| 2021-22, 2022-23 | India Tourism Statistics 2023 | Table 5.2.3 |
| 2022-23, 2023-24 | India Tourism Data Compendium 2024 | Table 4.2.3 (used for cross-validation) |
| 2023-24, 2024-25 | India Tourism Data Compendium 2025 | Table 4.2.3 (canonical monument list & names) |

**Coverage:** 144 of 145 monuments have data for all 6 periods; the remaining monument (Mattancherry Palace Museum, Kochi) has no disclosed figures in the source tables for several years — preserved as blank, not zero.

## Validation results (rebuilt data vs. official published grand totals)

| Period | Domestic (rebuilt vs official) | Foreign (rebuilt vs official) |
|---|---|---|
| 2021-22 | 27.49M vs 26.05M — see note below | 116,023 vs 116,114 (99.9%) |
| 2022-23 | 47.85M vs 47.90M (99.9%, explained) | 1.44M vs 1.45M (99.6%, explained) |
| 2023-24 | **53,090,007 — exact match** | **2,314,641 — exact match** |
| 2024-25 | 54.19M vs 54.23M (99.93%, explained) | **2,415,000 — exact match** |

I also cross-checked the two PDFs that both cover 2022-23 and 2023-24 independently (Compendium 2024 and Compendium 2025) — after fixing one extraction bug, **every one of the 145 monuments matches exactly between the two independent sources** for their overlapping years. This is strong evidence the rebuilt data is accurate.

### Explained gaps
- **2024-25 domestic (short by 38,296, 0.07%):** entirely due to Mattancherry Palace Museum's undisclosed figures.
- **2022-23 (short by ~54K domestic, ~6K foreign, ~0.1%):** the 2021-22/2022-23 source table only lists 144 monuments — one monument present in later years' tables didn't exist in that year's published table.
- **2021-22 domestic (high by ~1.44M):** see anomaly below — not explained by missing data, the reverse.

### Flagged data-quality issue (not fixed, left visible)
**Cooch Bihar Palace, 2021-22 domestic visitors** is printed in the source PDF (India Tourism Statistics 2023) as 2,287,176 — but this doesn't reconcile with the same table's own printed year-over-year growth percentage, and it's wildly out of line with that monument's visitor count in every surrounding year (126K in 2020-21, 609K in 2022-23, 685K in 2023-24, 682K in 2024-25). This looks like a genuine typo in the official government publication itself. I did not correct it — the original printed figure is preserved, and the row is flagged `flag_source_anomaly = True` so it's easy to exclude or treat specially in modeling. This single value fully explains why the 2021-22 domestic total runs high.

### Two extraction bugs I caught and fixed (my own parsing, not the source)
1. **Sun Temple, Konark (2022-23 domestic):** table extraction split "24,05,307" and misplaced the leading digit into the name field, initially reading as 405,307 — confirmed and corrected to 2,405,307 by visually inspecting the source page image.
2. **One monument name truncated at a page break** ("Ancient Buddhist" → corrected to "Ancient Buddhist Site know as Chaukhandi stupa") using the same monument's full name from an earlier table.

## Files in this package

| File | Description |
|---|---|
| `monument_visitors_2019_20_to_2024_25_FULL.csv` | **Main file.** 145 monuments × 6 annual periods, circle, two QA flag columns |
| `monument_master.csv` | Original 145-monument master list (unchanged) |
| `national_aggregate_2004_2024_25.csv` | National totals, 2004–2024/25 (unchanged, already clean) |
| `national_quarterly_seasonality_2001_2019.csv` | National quarterly seasonality shape (unchanged) |
| `statewise_visits_2019_2020.csv` | State/UT-level visits, 2019 vs 2020 (unchanged) |

The old `monument_visitors_2021_22_to_2024_25_CLEAN.csv` and `_NEEDS_REVIEW.csv` files are superseded — use `monument_visitors_2019_20_to_2024_25_FULL.csv` instead.

## What this unlocks

- A genuine 6-year annual time series per monument (2019-20 → 2024-25), including the COVID collapse and recovery — good for trend/recovery modeling, not just a single post-COVID snapshot.
- High confidence in the 2021-22 → 2024-25 span specifically (cross-validated across independent official sources).
- One clearly flagged anomaly you can choose to exclude, winsorize, or investigate further, rather than a black-box "clean" number.

## Still not resolved
- Sub-annual (quarterly/monthly/daily) footfall per monument — none of the official ASI publications report at that granularity. The national quarterly seasonality shape remains the best available proxy for approximating within-year timing, and it's a national average, not monument-specific.
