# Irish Rental Market Data (2020–2025) & Power BI Dashboard

This repository contains half-yearly average rent data for towns, cities, and counties across Ireland, along with a Power BI dashboard built on top of it.

## Contents

| File | Description |
|---|---|
| `irish_rent_full.csv` | Raw dataset — 50,208 rows of average monthly rent (€), by area, property type, and bedroom count, for each half-year from 2020H1 to 2025H1 |
| `Dashboard_1__1_.png` | Screenshot of the Power BI report built from this dataset (4 visuals) |

## About the Data

Each row is an **average monthly rent figure (€)** for a specific combination of:
- a geographic area (town/area, county, province)
- a property type (e.g. apartment, semi-detached house)
- a bedroom category (e.g. one bed, two bed)
- a half-year reporting period (H1 = Jan–Jun, H2 = Jul–Dec)

The structure — half-yearly periods, county/area breakdowns, property type × bedroom-count cross-tabs — closely mirrors the **RTB (Residential Tenancies Board) Rent Index**, Ireland's official report on registered tenancy rents. It's likely this file was built from that source, though it isn't stated in the data itself.

### Column reference

| Column | Type | Description |
|---|---|---|
| `rent_euro` | number | Average monthly rent in euro for this row's combination of area/type/bedrooms/period |
| `year` | integer | Calendar year (2020–2025) |
| `half` | integer | 1 = H1 (Jan–Jun), 2 = H2 (Jul–Dec) |
| `half_year` | text | Combined period label, e.g. `2023H2` |
| `time_period` | integer | Sequential period index (1 = 2020H1 … 11 = 2025H1) |
| `county` | text | One of Ireland's 26 counties |
| `province` | text | Leinster, Munster, Connacht, or Ulster (only the 3 Republic-of-Ireland Ulster counties: Cavan, Donegal, Monaghan) |
| `area` | text | Town/area name (e.g. `Terenure`) |
| `location` | text | Fuller location label, often `area, county` or `area, Dublin postcode` |
| `property_type` | text | All property types, Detached house, Semi detached house, Terrace house, Apartment, Other flats |
| `bedrooms` | text | Bedroom category as reported (see mapping below) |
| `bedrooms_num` | number | Numeric bedroom bucket derived from `bedrooms`; blank when `bedrooms` = "All bedrooms" |
| `is_dublin` | boolean | Whether the area falls within Co. Dublin |
| `is_city` | boolean | Whether the area is classified as urban/city |
| `is_county_aggregate` | boolean | `True` for the county-level roll-up row; `False` for individual towns/areas within that county |

**Bedroom category → numeric bucket:**

| `bedrooms` | `bedrooms_num` | Rows |
|---|---|---|
| One bed | 1.0 | 3,600 |
| 1 to 2 bed | 1.5 | 7,708 |
| Two bed | 2.0 | 6,363 |
| 1 to 3 bed | 2.0 | 11,085 |
| Three bed | 3.0 | 6,080 |
| Four plus bed | 4.0 | 3,033 |
| All bedrooms | *(blank)* | 12,339 |

Note that **"Two bed" and "1 to 3 bed" both map to 2.0** — so the "2.0" bucket in any chart combines two different original categories, which is why it's by far the largest.

### Coverage at a glance

- **50,208 rows**, covering **11 half-year periods**: 2020H1 through 2025H1
- **2025 is a partial year** — only H1 has been reported so far, H2 isn't in the data yet
- **4 provinces**, **26 counties**, **403 distinct areas** / 416 location labels
- **6 property types**, **7 bedroom categories**
- Rent ranges from **€401.58** (Donegal, one-bed flats, 2020H2) to **€5,372.87** (Ballsbridge, Dublin 4, four-plus-bed, 2025H1)

One important trend to know before reading the dashboard: **the number of rows reported per period shrinks steadily over time** — from 11,851 rows in 2020 down to 2,993 in the H1-only 2025 — as fewer areas/breakdowns get published in the more recent periods. This matters for how the "Rent Cost Over Year" chart should be read (see below).

## The Power BI Dashboard

The dashboard (`Dashboard_1__1_.png`) has four visuals, all built on the `rent_euro` field summed (aggregated with SUM, not averaged):

### 1. Rent Cost per Bedrooms
A bar chart of summed rent across all rows, grouped by `bedrooms_num`. The 2.0 bucket dominates at **~22.1M**, because — as noted above — it combines both "Two bed" and "1 to 3 bed" rows.

### 2. Rent Cost per Province
A bar chart of summed rent by `province`. **Leinster** dwarfs the other three provinces (**~45.5M**, vs. ~11.9M Munster, ~5.8M Connacht, ~2.0M Ulster), largely because it contains Dublin.

### 3. Rent Cost Over Year
A line chart of summed rent by `year`, falling from **14.6M in 2020 to 4.6M in 2025**. At a glance this looks like rents are collapsing — but see the caveat below, because that's not what's actually happening.

### 4. Rent Cost per Province per County
A bar chart of summed rent by county, faceted by province. **Dublin alone accounts for ~31.5M** — more than every other Leinster county combined, and more than Munster, Connacht, and Ulster combined.

## ⚠️ How to read these charts correctly

The dashboard's visuals **sum** an already-averaged figure (`rent_euro`), so the totals shown (e.g. "45.5M") aren't a real amount of money anyone paid — they're a sum of hundreds of average-rent figures, useful only for comparing categories *relative to each other*, not as a literal euro total.

This matters most for the **"Rent Cost Over Year"** chart. Its steady decline doesn't mean rents fell — it's mostly a side effect of two things:
1. **2025 has only one half-year of data** (H1), so it's naturally about half the size of a full year like 2024.
2. **Fewer rows are reported in later years** (11,851 rows in 2020 vs. 2,993 in 2025H1), which shrinks the sum regardless of the rent level.

If you instead look at the **average** `rent_euro` per row by year — a fairer measure of the actual rent trend — the picture reverses completely:

| Year | Average rent (€/month) |
|---|---|
| 2020 | 1,232.52 |
| 2021 | 1,249.60 |
| 2022 | 1,260.96 |
| 2023 | 1,330.72 |
| 2024 | 1,445.42 |
| 2025 (H1 only) | 1,536.86 |

Rents actually **rose every year** across this dataset. If this dashboard is developed further, switching the "Rent Cost Over Year" measure from `SUM(rent_euro)` to `AVERAGE(rent_euro)` would give a much more accurate trend line.

A similar caution applies to `is_county_aggregate`: each county has both a single roll-up row and separate rows for its individual towns, so summing without filtering on this flag will double-count rent for many counties.

## Suggested next steps

- Switch chart measures to `AVERAGE(rent_euro)` (or a properly weighted average) instead of `SUM` for anything meant to represent typical rent levels.
- Filter to `is_county_aggregate = TRUE` when comparing counties, to avoid double-counting county totals against their own sub-areas.
- Exclude `bedrooms = "All bedrooms"` and `property_type = "All property types"` rows from breakdown charts, since those are themselves summaries of the other rows.
- Treat 2025 as a partial year in any year-over-year comparison until H2 data is available.

## Source note

The underlying data isn't accompanied by source metadata, but its structure (half-yearly periods, county/area geography, property type × bedroom cross-tabs, actual-vs-asking framing implied by "average rent") is consistent with Ireland's **RTB Rent Index / Average Rent Report**, published via the Residential Tenancies Board and data.gov.ie. If you know the exact source/export date, it's worth adding here for future reference.
