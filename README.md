# Vancouver Property Value & Housing Supply Analysis

This project explores Vancouver property assessment values and residential building permit activity to understand how assessed values and housing-supply signals vary over time and across the city. Using two public datasets from the City of Vancouver Open Data portal, it builds a reproducible analytical pipeline from raw data through to validated, documented metrics.

---

## Business Question

> How do Vancouver property assessment values relate to recent residential building permit activity, and what can these public datasets reveal about housing supply pressure and neighbourhood-level change?

---

## Why This Project Matters

Vancouver's housing affordability is one of the most prominent public policy issues in Canada. Assessed values, construction permits, and investment intensity are all publicly available — yet rarely combined in a single, documented analytical pipeline. This project demonstrates that public data can surface meaningful patterns in where and how much new residential supply is being approved, and how those signals align with shifts in assessed property values.

Beyond the subject matter, the project demonstrates a full analytics workflow: data acquisition decisions, cleaning, feature engineering, output validation, and structured documentation — skills directly applicable to data analyst and analytics engineering roles.

---

## Data Sources

| Dataset | Source | Role in Project | Status |
|---|---|---|---|
| Property Tax Report | City of Vancouver Open Data | Property assessment values — land, improvements, year-over-year change | Downloaded; used for feature engineering |
| Issued Building Permits | City of Vancouver Open Data | Residential permit activity and declared project value by year | Downloaded; filtered, aggregated, and exported to `data/processed/` |
| StatCan / CMHC contextual data | Statistics Canada / CMHC | Optional future context (population, mortgage rates, rental vacancy) | Not currently part of core pipeline |

Raw files are excluded from version control via `.gitignore`. Decisions about which datasets to include or exclude are logged in [`docs/dataset_decision_log.md`](docs/dataset_decision_log.md).

---

## Current Analytical Outputs

### Property Tax Features

Derived from `data/raw/property_tax_report_raw.csv` using a validated 1,000-row development sample.

| Feature | Description |
|---|---|
| `current_total_assessed_value` | Sum of current land value and current improvement value for each parcel; null if either component is missing |
| `previous_total_assessed_value` | Sum of prior-year land value and prior-year improvement value; null if either component is missing |
| `absolute_value_change` | Difference between current and previous total assessed value; propagates null when either total is missing |
| `percentage_value_change` | Year-over-year percentage change in total assessed value; computed only for parcels with a positive previous value |

### Residential Building Permit Metrics

Derived from `data/raw/issued_building_permits_raw.csv` using the full dataset, filtered to residential permits (2019–2024).

| Feature | Description |
|---|---|
| `permit_count_by_year` | Number of distinct residential permits issued per year, counted by unique `PermitNumber` |
| `permit_project_value_by_year` | Annual sum, median, mean, and permit count of declared `ProjectValue` for residential permits per year |

---

## Processed Files

The following derived outputs are small enough to commit to version control and are tracked in Git. They will serve as inputs to visualisation notebooks and any future dashboard.

| File | Rows | Description |
|---|---|---|
| `data/processed/permit_count_by_year.csv` | 6 | One row per year (2019–2024): `IssueYear`, `permit_count` |
| `data/processed/permit_project_value_by_year.csv` | 6 | One row per year (2019–2024): `IssueYear`, `total_project_value`, `median_project_value`, `mean_project_value`, `permit_count` |

---

## Initial Visual Outputs

The project currently includes two initial visual outputs based on processed annual residential building permit metrics.

### Annual Residential Permit Count

![Residential building permits issued by year](visuals/permit_count_by_year.png)

This chart shows the annual count of unique residential building permits issued in Vancouver from 2019 to 2024, using permits where `PropertyUse` contains `"Dwelling Uses"`.

The count fluctuates across the period, with the highest permit count in 2022. This metric should be interpreted as permitting activity, not as completed housing units.

### Annual Residential Permit Project Value

![Residential building permit project value by year](visuals/permit_project_value_by_year.png)

This chart shows the annual sum of declared `ProjectValue` for residential building permits from 2019 to 2024.

The total declared project value increases notably in 2022 and 2023 compared with earlier years. However, `ProjectValue` represents declared or estimated project value, not market value, and large projects may disproportionately affect annual totals.

---

## Repository Structure

```
data/
  raw/            # Original source files — never modified, excluded from Git
  processed/      # Derived analytical outputs — tracked in Git
notebooks/        # Numbered Jupyter notebooks, intended to be run in order
docs/             # Methodology, data dictionary, dataset decision log, business narrative
visuals/          # Exported charts (future)
dashboard/        # Dashboard assets (future)
src/              # Reusable Python scripts extracted from notebooks (future)
```

---

## Methodology Summary

- Raw data files are kept strictly separate from processed outputs. Files in `data/raw/` are immutable — they are never modified or overwritten.
- Large raw files (56 MB+) are excluded from Git via `.gitignore`. Only small, validated derived outputs are committed.
- Derived features are validated with explicit checks (null rate, range, infinity detection) before being considered complete.
- Building permit counts use `nunique()` on `PermitNumber` within each year to avoid inflating counts from amended or reissued permits.
- Residential permits are isolated by filtering `PropertyUse` for records containing the substring `"Dwelling Uses"` (case-insensitive, null-safe). This is more reliable than `PermitCategory`, which has a high null rate in the full dataset.
- The 2019–2024 time window captures the pre- and post-pandemic period of significant housing market movement in Metro Vancouver, and is the scope for which data quality has been validated within this project.

Full methodology is documented in [`docs/methodology.md`](docs/methodology.md).

---

## Key Caveats

- **Assessed value ≠ sale price.** BC Assessment valuations reflect assessed value at a point in time, not MLS transaction prices or market value.
- **Permits ≠ completed housing units.** A building permit is issued before construction begins and does not confirm the project was completed, occupied, or added to the housing stock.
- **`ProjectValue` ≠ property market value.** It is the applicant-declared estimated construction cost and is not independently verified at the time of permit issuance.
- **`ProjectValue` totals are nominal.** No inflation adjustment has been applied. Year-over-year comparisons of total project value reflect both changes in construction volume and changes in construction costs.
- **Large projects can skew totals.** A single high-rise development can materially affect annual `total_project_value` and `mean_project_value`. The `median_project_value` column is included as a more robust alternative.

---

## Current Project Status

**Completed**

- [x] Project scaffold and repo conventions
- [x] Raw data acquisition
- [x] Dataset decision log
- [x] Initial data understanding and EDA (Notebook 01)
- [x] Property value feature engineering with validation (Notebook 02)
- [x] Residential permit metrics with validation (Notebook 03)
- [x] Processed annual permit exports to `data/processed/`
- [x] Data dictionary updated for all datasets and derived outputs
- [x] Methodology documented

**In Progress / Next**

- [ ] Visualisations — annual permit trends, assessed value distributions
- [ ] Neighbourhood-level analysis — spatial breakdown by `GeoLocalArea` and `NEIGHBOURHOOD_CODE`
- [ ] Cross-dataset alignment — joining permit and property tax data by area and year
- [ ] Final business narrative and conclusions
- [ ] Dashboard or portfolio presentation

---

## Skills Demonstrated

`Python` · `pandas` · `data cleaning` · `feature engineering` · `data validation` · `Git / GitHub workflow` · `documentation` · `business analysis` · `public-data storytelling`

---

## Documentation

- [Data Dictionary](docs/data_dictionary.md)
- [Dataset Decision Log](docs/dataset_decision_log.md)
- [Methodology](docs/methodology.md)
- [Business Narrative](docs/business_narrative.md)

---

## Setup

```bash
python -m venv .venv
.venv\Scripts\activate      # Windows
# source .venv/bin/activate  # macOS / Linux
pip install -r requirements.txt
jupyter notebook
```
