# Data Dictionary

Documents the key columns used in this project. Only fields relevant to the analysis are included — not every column in each raw file.

**Important caveats that apply throughout:**
- Assessed values (land, improvement, levy) are **BC Assessment valuations**, not MLS sale prices or actual transaction prices. They reflect the assessed value at a point in time and may not track market prices closely.
- Building permits represent **authorisation to build**, not completed housing units. A permit can be issued and never built, or take years to complete.

---

## Dataset 1 — Property Tax Report

**File:** `data/raw/property_tax_report_raw.csv`  
**Source:** City of Vancouver Open Data  
**Observed dtypes based on 1,000-row sample**

| Column | Observed Type | Plain-English Meaning | Project Use | Notes |
|---|---|---|---|---|
| `PID` | str | Parcel Identifier — a unique ID assigned to each land parcel by BC Assessment | Primary key for joining rows across years or to other datasets | May contain leading zeros; treat as string, not integer |
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
| `TAX_LEVY` | float64 | The total property tax levied on the parcel for the assessment year | Contextual indicator of tax burden relative to assessed value | Not a value metric — reflects tax policy, exemptions, and mill rates, not market value |
| `NEIGHBOURHOOD_CODE` | int64 | A City of Vancouver neighbourhood code identifying the geographic neighbourhood of the parcel | Geographic grouping for spatial analysis and aggregation | Requires a neighbourhood code lookup table to map codes to human-readable names |
| `REPORT_YEAR` | int64 | The year in which this record appears in the open data report | Identifying which snapshot of data a row comes from | May differ from `TAX_ASSESSMENT_YEAR`; clarify the distinction before computing trends |

---

## Dataset 2 — Issued Building Permits

**File:** `data/raw/issued_building_permits_raw.csv`  
**Source:** City of Vancouver Open Data  
**Observed dtypes based on 1,000-row sample**

| Column | Observed Type | Plain-English Meaning | Project Use | Notes |
|---|---|---|---|---|
| `PermitNumber` | str | Unique identifier assigned to each building permit application | Primary key; joining or deduplicating permit records | Format varies (e.g., `BP-XXXXXXX`); treat as string |
| `IssueDate` | str | The calendar date the permit was formally issued by the City | Precise date for time-series analysis | Loads as string; requires conversion to `datetime` using `pd.to_datetime()` during cleaning |
| `IssueYear` | int64 | Calendar year extracted from `IssueDate` | Primary time dimension for trend analysis (2019–2024 scope) | Derived from `IssueDate`; use for annual aggregations |
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

*Last updated: 2026-06-10. Update this file whenever a new column is introduced into analysis.*
