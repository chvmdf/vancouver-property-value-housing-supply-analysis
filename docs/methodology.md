# Methodology

Documents the analytical approach, key decisions, and assumptions behind this project.

## Research Question

_To be defined once datasets are confirmed._

## Analytical Approach

_To be filled in after EDA._

## Key Assumptions

_To be documented as they are made during analysis._

## Limitations

_To be documented as they are identified._

## Tools & Libraries

- Python 3.x
- pandas  --  data manipulation
- matplotlib  --  visualization
- Jupyter  --  interactive analysis environment

---

## Building Permit Metrics Methodology

This section documents the analytical decisions behind the two processed annual permit metrics: `permit_count_by_year` and `permit_project_value_by_year`. Full implementation details, including all filter logic and validation checks, are in `notebooks/03_building_permits_preparation_and_analysis.ipynb`.

### Raw Data Separation

Raw source files in `data/raw/` are treated as immutable inputs and are never modified or overwritten. All filtering, cleaning, and aggregation is performed in notebooks, and the results are written to `data/processed/`. This separation ensures that:

- The original source data is always recoverable.
- Every analytical decision can be traced back to a specific notebook and code cell.
- Processed outputs can be regenerated from scratch if the methodology changes.

### Why Export to `data/processed/`?

The raw permits file is approximately 56 MB. Reloading and re-filtering it in every downstream notebook would be slow and redundant. The two processed annual metrics are six-row summary tables  --  a few hundred bytes each  --  that load instantly. They are small enough to commit to version control alongside the notebooks that produced them, making the project reproducible without requiring access to the large raw file.

### Residential Filter: `PropertyUse` Contains `"Dwelling Uses"`

The `PropertyUse` column is the most consistently populated and semantically direct field for identifying residential permits in this dataset. The filter retains any permit where `PropertyUse` contains the substring `"Dwelling Uses"` (case-insensitive). This approach:

- Captures both pure residential records and mixed-use records where `"Dwelling Uses"` is part of a longer category name.
- Excludes null `PropertyUse` values via `na=False`, so ambiguous records are not silently included.
- Was validated against the full dataset during exploratory analysis in section 6 of the notebook.

An alternative filter based on `PermitCategory` was considered but rejected. The `PermitCategory` column has a substantial null rate in the full dataset (approximately 43% null in the 1,000-row sample), which would have left a large share of potentially residential permits uncategorised and excluded from the metric.

### Time Window: 2019-2024

The analysis covers 2019 through 2024 inclusive for two reasons:

- This window captures the period immediately before and during the substantial post-pandemic housing market movement in Vancouver, which is the primary period of analytical interest for this project.
- Years before 2019 have not been validated for data quality, coverage, or consistency in permit category conventions within this project scope.

The time window is applied using `IssueYear.between(2019, 2024)`, which includes both endpoints and safely handles null year values by returning `False` for them.

### Why `permit_count` Uses Unique `PermitNumber`

A single `PermitNumber` can appear more than once in the raw data if a permit was amended, corrected, or reissued. Counting total rows rather than distinct identifiers would inflate the metric. Using `nunique()` on `PermitNumber` within each year ensures each permit is counted once regardless of how many records it has in the source file. `permit_count` therefore measures distinct approved permits, not administrative record entries.

### `ProjectValue` as a Housing-Supply Investment Signal

`ProjectValue` is the applicant-declared estimated value of the permitted construction work. It is not independently verified at the permit stage and reflects anticipated costs  --  labour, materials, and related work  --  rather than any form of property market value or assessed value. It is used in this project as a proxy for the scale and investment intensity of residential construction activity in a given year.

All `total_project_value` figures are in nominal Canadian dollars. No inflation adjustment has been applied. Year-over-year comparisons of this metric therefore reflect both changes in the volume of construction activity and changes in construction costs over time.

### Why These Metrics Should Not Be Interpreted as Completed Housing Units

A building permit is an authorisation to begin regulated construction. It is issued before work starts and does not confirm that the project was completed, occupied, or added to the housing stock. A single permit may cover a single-family house, an entire apartment building with hundreds of units, or a residential renovation that adds no new units at all.

The permit metrics in this project are leading indicators of housing supply activity. They signal the pace and scale of approved construction, not the number of dwellings delivered to the market.

### Future Use

Both processed files will be used as inputs to visualisation notebooks and dashboard development in later stages of the project. The annual permit count and project value series will be analysed alongside assessed property value trends from the Property Tax dataset to compare residential permitting activity with assessed value trends descriptively over the 2019-2024 period.

---

## Property Tax Full-Dataset Processing Methodology

This section documents the processing approach for the full City of Vancouver Property Tax Report and the derived outputs it produced. Full implementation details, including chunked reading logic and validation checks, are in `notebooks/05_property_value_processed_metrics.ipynb` and `notebooks/06_property_value_distribution_buckets.ipynb`.

### Chunked Processing with `pd.read_csv(..., chunksize=100_000)`

The full Property Tax Report (`property_tax_report_raw.csv`) is approximately 443 MB  --  too large to load into memory comfortably on modest hardware. The full-dataset processing function reads the file in blocks of 100,000 rows using `pd.read_csv(..., chunksize=100_000)`. For each chunk:

1. The four derived value-change metrics are computed using `compute_value_change_features()`, the same function validated on a 1,000-row sample earlier in the notebook.
2. Quality-control counters (missing values, negative totals, infinite values, extreme changes) are accumulated as plain integers. No full-dataset DataFrame is held in memory across chunk boundaries.
3. Only the four derived columns for that chunk are written directly to the distribution file on disk, using append mode after the first chunk.

This approach produces the same result as loading the full file at once, without the memory cost.

### Why the Row-Level Distribution File Is Excluded from Git

The output of chunk processing  --  `property_value_change_distribution.csv`  --  contains one row for every property in the dataset (1,552,663 rows) and is approximately 68.47 MB. Committing a file of this size would bloat the Git history and make cloning slow for anyone without local access to the raw data. The file is therefore excluded from the repository and exists only on the local machine where processing was run.

The distribution file is still useful locally: it is the direct input to `notebooks/06_property_value_distribution_buckets.ipynb` and any future downstream visualisation or aggregation notebooks.

### The Binned Distribution File as the Portfolio-Ready Tracked Output

Rather than tracking the large distribution file, Notebook 06 reads only the `percentage_value_change` column from it and produces `property_value_change_distribution_bins.csv`  --  a 10-row aggregated summary assigning each property record to a percentage change bucket. This file is small enough to commit to the repository alongside the notebook that produced it.

Bucket boundaries use explicit `if` conditions in `assign_percentage_change_bucket()` rather than `pd.cut` defaults, so every boundary is unambiguous. `Below -50%` is strictly `< -50`, matching the `extreme_low_count` definition in `property_value_change_summary.csv`. The binned output preserves the distributional shape of the data for portfolio presentation and downstream visualisation without requiring access to the full distribution file.

### Assessed Values Are Not Market Prices

BC Assessment assessed values are administrative property valuations used as the basis for property tax calculations. They are not MLS sale prices, transaction prices, or appraisals. Year-over-year changes in assessed value reflect changes in the administrative valuation and should not be interpreted as direct measures of market appreciation or depreciation.

Outliers and extreme percentage changes documented in `property_value_change_summary.csv` are retained at this stage. Extreme changes may reflect legitimate events  --  redevelopment, subdivision, or zoning reclassification  --  rather than data errors, and are flagged for future investigation rather than filtered out preemptively.

No causal claims are made between assessed property value changes and housing supply, permit activity, or market prices.
