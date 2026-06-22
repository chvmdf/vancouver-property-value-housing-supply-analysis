# Data Dictionary

Documents the key columns used in this project. Only fields relevant to the analysis are included  --  not every column in each raw file.

**Important caveats that apply throughout:**
- Assessed values (land, improvement, levy) are **BC Assessment valuations**, not MLS sale prices or actual transaction prices. They reflect the assessed value at a point in time and may not track market prices closely.
- Building permits represent **authorisation to build**, not completed housing units. A permit can be issued and never built, or take years to complete.

---

## Dataset 1  --  Property Tax Report

**File:** `data/raw/property_tax_report_raw.csv`  
**Source:** City of Vancouver Open Data  
**Observed dtypes based on 1,000-row sample**

| Column | Observed Type | Plain-English Meaning | Project Use | Notes |
|---|---|---|---|---|
| `PID` | str | Parcel Identifier  --  a unique ID assigned to each land parcel by BC Assessment | Primary key for joining rows across years or to other datasets | May contain leading zeros; treat as string, not integer |
| `LEGAL_TYPE` | str | Legal classification of the parcel (e.g., LAND, STRATA, BARE LAND STRATA) | Filtering to specific property classes if needed | Strata parcels represent individual strata lots within a larger building |
| `ZONING_DISTRICT` | str | The zoning district code assigned by the City of Vancouver (e.g., RS-1, RM-4, C-2) | Geographic and policy-level grouping; linking supply signals to zoning context | Codes follow City of Vancouver zoning bylaws; interpretation requires the zoning schedule |
| `ZONING_CLASSIFICATION` | str | A broader classification label derived from the zoning district (e.g., Single Family Residential, Multiple Dwelling) | Aggregating parcels into meaningful land-use categories for analysis | Less granular than `ZONING_DISTRICT`; useful for summary groupings |
| `CURRENT_LAND_VALUE` | float64 | BC Assessment's assessed value of the land only, excluding any structures, for the current assessment year | Core metric for property value analysis; land value trend over time | May load as `object` if formatted with `$` or `,`; will require cleaning before arithmetic |
| `CURRENT_IMPROVEMENT_VALUE` | float64 | BC Assessment's assessed value of structures and improvements on the parcel, excluding land | Differentiating land value appreciation from improvement value changes | Same cleaning caveat as `CURRENT_LAND_VALUE` |
| `PREVIOUS_LAND_VALUE` | float64 | Land value from the prior assessment year | Calculating year-over-year change in land value | Availability depends on whether the dataset includes prior-year columns; verify in full load |
| `PREVIOUS_IMPROVEMENT_VALUE` | float64 | Improvement value from the prior assessment year | Calculating year-over-year change in improvement value | Same caveat as `PREVIOUS_LAND_VALUE` |
| `TAX_ASSESSMENT_YEAR` | float64 | The year for which the tax assessment applies | Time dimension for trend analysis | Distinct from `REPORT_YEAR`; confirm which year each row represents |
| `YEAR_BUILT` | float64 | The year the primary structure on the parcel was built, according to BC Assessment records | Controlling for building age in analysis; identifying older vs. newer stock | May be null for vacant land or parcels where year is unknown; do not use as a reliable source of construction dates |
| `BIG_IMPROVEMENT_YEAR` | float64 | The year of the most significant improvement or renovation recorded for the parcel | Identifying parcels that have been substantially upgraded | Sparsely populated; treat nulls as "no major improvement recorded", not as absence of any improvement |
| `TAX_LEVY` | float64 | The total property tax levied on the parcel for the assessment year | Contextual indicator of tax burden relative to assessed value | Not a value metric  --  reflects tax policy, exemptions, and mill rates, not market value |
| `NEIGHBOURHOOD_CODE` | int64 | A City of Vancouver neighbourhood code identifying the geographic neighbourhood of the parcel | Geographic grouping for spatial analysis and aggregation | Requires a neighbourhood code lookup table to map codes to human-readable names |
| `REPORT_YEAR` | int64 | The year in which this record appears in the open data report | Identifying which snapshot of data a row comes from | May differ from `TAX_ASSESSMENT_YEAR`; clarify the distinction before computing trends |

---

## Dataset 2  --  Issued Building Permits

**File:** `data/raw/issued_building_permits_raw.csv`  
**Source:** City of Vancouver Open Data  
**Observed dtypes based on 1,000-row sample**

| Column | Observed Type | Plain-English Meaning | Project Use | Notes |
|---|---|---|---|---|
| `PermitNumber` | str | Unique identifier assigned to each building permit application | Primary key; joining or deduplicating permit records | Format varies (e.g., `BP-XXXXXXX`); treat as string |
| `IssueDate` | str | The calendar date the permit was formally issued by the City | Precise date for time-series analysis | Loads as string; requires conversion to `datetime` using `pd.to_datetime()` during cleaning |
| `IssueYear` | int64 | Calendar year extracted from `IssueDate` | Primary time dimension for trend analysis (2019-2024 scope) | Derived from `IssueDate`; use for annual aggregations |
| `YearMonth` | str | Year and month of permit issuance formatted as a string (e.g., `2022-03`) | Monthly aggregation of permit activity | Useful for plotting seasonal patterns without full date parsing |
| `PermitElapsedDays` | int64 | Number of days between permit application and issuance | Measuring processing time; not directly used in supply analysis | High variance expected; outliers may represent complex projects or data entry anomalies |
| `ProjectValue` | float64 | Applicant-declared estimated value of the construction project in Canadian dollars | Proxy for investment scale of each permit | Self-reported by applicant, not independently verified; large outliers are plausible for major developments |
| `TypeOfWork` | str | Category describing the nature of the permitted work (e.g., New Building, Addition / Alteration, Demolition) | Filtering to new construction only for supply signal analysis | Critical filter: supply analysis should use `New Building` permits; alterations and demolitions should be handled separately |
| `PermitCategory` | str | A classification of the permit type (e.g., Residential, Commercial) | Filtering to residential permits for this project | Verify exact category values in the full dataset before filtering |
| `PropertyUse` | str | How the property is intended to be used after construction (e.g., Dwelling Uses, Retail Uses) | Secondary filter to isolate residential supply | More specific than `PermitCategory`; use in combination for clean filtering |
| `SpecificUseCategory` | str | Granular use classification within the property use (e.g., One-Family Dwelling, Multiple Dwelling) | Distinguishing single-family from multi-family supply signals | Key for disaggregating housing type in trend analysis |
| `GeoLocalArea` | str | The City of Vancouver local planning area where the property is located (e.g., Kitsilano, Mount Pleasant) | Geographic grouping for spatial analysis; linking permits to property tax neighbourhood data | Human-readable area names; check alignment with `NEIGHBOURHOOD_CODE` in the property tax dataset before joining |
| `Geom` | str | GeoJSON geometry object representing the parcel or project location | Mapping and spatial analysis if needed | Requires parsing as JSON; not needed for tabular trend analysis |
| `geo_point_2d` | str | Latitude and longitude as a comma-separated string (e.g., `49.2604,-123.1182`) | Mapping permit locations; spatial joins | Requires splitting into separate latitude and longitude columns before use in any spatial library |

---

## Processed Building Permit Metrics

These files are derived analytical outputs produced by `notebooks/03_building_permits_preparation_and_analysis.ipynb`. Each is a small, validated six-row summary table aggregated from the raw Issued Building Permits dataset. They are saved to `data/processed/` so that downstream notebooks, visualisations, and dashboard scripts can read them directly without re-running the full filter and aggregation pipeline.

---

### File: `data/processed/permit_count_by_year.csv`

**Source raw dataset:** `data/raw/issued_building_permits_raw.csv`  
**Analytical grain:** One row per `IssueYear`  
**Time window:** 2019-2024  
**Residential filter:** `PropertyUse` contains `"Dwelling Uses"`, case-insensitive, including mixed-use records where `"Dwelling Uses"` appears as part of a longer string

| Column | Type | Plain-English Meaning |
|---|---|---|
| `IssueYear` | int | Calendar year when the permit was issued |
| `permit_count` | int | Number of unique `PermitNumber` values for residential permits issued in that year |

**Caveats:**
- `permit_count` measures issued permit activity, not completed housing units. A permit is approved before construction begins and may never result in a finished dwelling.
- One permit may represent one unit, multiple units, or a non-unit residential alteration such as a renovation or addition. The metric reflects the breadth of permitting activity, not the volume of dwelling units delivered.

---

### File: `data/processed/permit_project_value_by_year.csv`

**Source raw dataset:** `data/raw/issued_building_permits_raw.csv`  
**Analytical grain:** One row per `IssueYear`  
**Time window:** 2019-2024  
**Residential filter:** `PropertyUse` contains `"Dwelling Uses"`, case-insensitive, including mixed-use records

| Column | Type | Plain-English Meaning |
|---|---|---|
| `IssueYear` | int | Calendar year when the permit was issued |
| `total_project_value` | float | Annual sum of declared `ProjectValue` for residential permits in that year |
| `median_project_value` | float | Median declared `ProjectValue` for residential permits in that year |
| `mean_project_value` | float | Average declared `ProjectValue` for residential permits in that year |
| `permit_count` | int | Number of unique `PermitNumber` values for residential permits issued in that year |

**Caveats:**
- `ProjectValue` is the declared construction/project value as submitted by the permit applicant. It is not the BC Assessment assessed value and is not a property market sale price.
- All values are in nominal Canadian dollars and are not adjusted for inflation. Year-over-year comparisons should account for this limitation.
- Large individual projects such as high-rise apartment buildings can significantly skew annual `total_project_value` and `mean_project_value`. The `median_project_value` column is included as a robustness metric that is less sensitive to such outliers.
- Permits with a `ProjectValue` of zero were retained in the dataset. Their presence is documented in the validation output of the source notebook.

---

## Processed Property Value Metrics

These files are derived analytical outputs produced by `notebooks/05_property_value_processed_metrics.ipynb` and `notebooks/06_property_value_distribution_buckets.ipynb`. They are generated from the full City of Vancouver Property Tax Report and are stored in `data/processed/`.

> **Important caveat:** BC Assessment assessed values are administrative property valuations used as the basis for property tax calculations. They are not MLS sale prices, transaction prices, or market values. Year-over-year changes in assessed value should not be interpreted as direct market appreciation or depreciation. No causal claims are made between assessed value changes and housing supply or permitting activity.

### Derived Value-Change Metrics

The following four columns are computed for every property record in the full Property Tax Report. They are defined in `notebooks/05_property_value_processed_metrics.ipynb` via `compute_value_change_features()`.

| Derived Column | Formula | Plain-English Meaning |
|---|---|---|
| `current_total_assessed_value` | `CURRENT_LAND_VALUE + CURRENT_IMPROVEMENT_VALUE` | Total BC Assessment assessed value (land plus improvements) for the current tax year. Null when either component is missing. |
| `previous_total_assessed_value` | `PREVIOUS_LAND_VALUE + PREVIOUS_IMPROVEMENT_VALUE` | Total BC Assessment assessed value (land plus improvements) for the prior tax year. Null when either component is missing. |
| `absolute_value_change` | `current_total_assessed_value - previous_total_assessed_value` | Year-over-year change in assessed value in Canadian dollars. Null when either total is null. |
| `percentage_value_change` | `absolute_value_change / previous_total_assessed_value * 100` | Year-over-year change expressed as a percentage of the prior-year total. Null when the prior-year total is null, zero, or negative. |

---

### File: `data/processed/property_value_change_summary.csv`

**Source notebook:** `notebooks/05_property_value_processed_metrics.ipynb`  
**Source raw dataset:** `data/raw/property_tax_report_raw.csv`  
**Analytical grain:** One row per derived value-change metric (4 rows total)  
**Git tracking:** Tracked  --  small file suitable for version control

| Column | Type | Plain-English Meaning |
|---|---|---|
| `metric` | str | Name of the derived column being described (one of the four metrics above) |
| `total_rows` | int | Total number of property records in the full raw dataset |
| `non_null` | int | Number of records where this metric was successfully computed (non-null) |
| `missing` | int | Number of records where this metric could not be computed (null) |
| `negative_total_count` | int | Number of records where `current_total_assessed_value` or `previous_total_assessed_value` is negative |
| `inf_count` | int | Number of records where this metric is infinite (should be zero if division-by-zero is handled correctly) |
| `extreme_high_count` | int | Number of records where `percentage_value_change > 100` |
| `extreme_low_count` | int | Number of records where `percentage_value_change < -50` |

**Caveats:**
- `negative_total_count`, `inf_count`, `extreme_high_count`, and `extreme_low_count` are quality-control flags, not filters. No records are removed on the basis of these counts.
- Extreme percentage changes may reflect legitimate events such as redevelopment, subdivision, or zoning reclassification rather than data errors.

---

### File: `data/processed/property_value_change_distribution.csv`

**Source notebook:** `notebooks/05_property_value_processed_metrics.ipynb`  
**Source raw dataset:** `data/raw/property_tax_report_raw.csv`  
**Analytical grain:** One row per property record (1,552,663 rows)  
**Approximate size:** 68.47 MB  
**Git tracking:** Not tracked  --  excluded from the repository due to file size; exists locally only

| Column | Type | Plain-English Meaning |
|---|---|---|
| `current_total_assessed_value` | float | Total BC Assessment assessed value for the current tax year (see derived metric definition above) |
| `previous_total_assessed_value` | float | Total BC Assessment assessed value for the prior tax year (see derived metric definition above) |
| `absolute_value_change` | float | Year-over-year change in assessed value in Canadian dollars (see derived metric definition above) |
| `percentage_value_change` | float | Year-over-year change expressed as a percentage of the prior-year total (see derived metric definition above) |

**Caveats:**
- This file is used locally for downstream aggregation and visualisation. It is not tracked in the repository.
- Outliers and extreme percentage changes are present. They are documented in `property_value_change_summary.csv` but not filtered out at this stage.

---

### File: `data/processed/property_value_change_distribution_bins.csv`

**Source notebook:** `notebooks/06_property_value_distribution_buckets.ipynb`  
**Source file:** `data/processed/property_value_change_distribution.csv` (local)  
**Analytical grain:** One row per percentage-change bucket (10 rows total)  
**Git tracking:** Tracked  --  small aggregated file suitable for version control

| Column | Type | Plain-English Meaning |
|---|---|---|
| `bucket` | str | Percentage change bucket label (see bucket definitions below) |
| `property_count` | int | Number of property records assigned to this bucket |
| `percentage_of_records` | float | Share of all records in this bucket, rounded to 4 decimal places |

**Bucket definitions for `percentage_value_change`:**

| Bucket | Condition |
|---|---|
| `Below -50%` | `percentage_value_change < -50` |
| `-50% to -20%` | `percentage_value_change >= -50` and `< -20` |
| `-20% to -10%` | `percentage_value_change >= -20` and `< -10` |
| `-10% to 0%` | `percentage_value_change >= -10` and `< 0` |
| `0% to 5%` | `percentage_value_change >= 0` and `< 5` |
| `5% to 10%` | `percentage_value_change >= 5` and `< 10` |
| `10% to 20%` | `percentage_value_change >= 10` and `< 20` |
| `20% to 50%` | `percentage_value_change >= 20` and `<= 50` |
| `Above 50%` | `percentage_value_change > 50` |
| `Missing / not calculable` | `percentage_value_change` is null |

**Boundary note:** `Below -50%` is strictly less than -50. A value of exactly -50 falls into `-50% to -20%`. This is consistent with the `extreme_low_count` quality-control flag in `property_value_change_summary.csv`, which counts records where `percentage_value_change < -50`.

**Caveats:**
- `property_count` values sum to the total number of rows in `property_value_change_distribution.csv` (1,552,663).
- No records are excluded before bucketing. Extreme-value buckets are included in the full distribution.
- No causal claims are made between assessed value changes and housing supply or permitting activity.

---

*Last updated: 2026-06-22. Update this file whenever a new column is introduced into analysis.*
