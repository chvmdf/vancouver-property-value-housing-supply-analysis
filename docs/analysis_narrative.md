# Analysis Narrative

## 1. Project Purpose

This project explores Vancouver property assessment values and residential building permit activity using two public datasets from the City of Vancouver Open Data portal. The goal is to build a reproducible, documented analytical pipeline that surfaces descriptive patterns in assessed property values and residential construction permitting over time.

The project is intentionally scoped as a descriptive data analytics exercise. It does not claim to identify causes of property value change or to model future housing market behaviour. It demonstrates what public data can reveal when cleaned, processed, and summarized in a structured, transparent workflow.

---

## 2. Business Question

> How can public property assessment and building permit data be used to describe patterns in assessed property value changes and residential permit activity in Vancouver?

This framing reflects the nature of the available data: both datasets are administrative records, not market transaction data. The question is answerable with the public sources in scope and avoids overstating what the data can support.

---

## 3. What the Data Shows

### Residential building permit activity

The processed permit output summarizes the annual count of distinct residential building permits issued in Vancouver from 2019 to 2024. The data shows variation in permit activity across this period, with permit counts shifting from year to year. This output measures approved permit activity, not completed construction.

### Declared permit project value

The annual declared project value for residential permits was also summarized. The processed outputs indicate that the total declared value shifted across the 2019-2024 window. Declared project value reflects applicant-estimated construction costs at the time of permit issuance and is not independently verified. Year-over-year comparisons reflect both changes in construction volume and changes in construction costs, with no inflation adjustment applied.

### Property assessment value change distribution

The full City of Vancouver Property Tax Report — covering all property records in the dataset — was processed using chunked reading to compute year-over-year assessed value change metrics for each record. The resulting distribution was summarized into 10 percentage-change buckets ranging from below -50% to above 50%, with a separate bucket for records where a change percentage could not be calculated.

The processed outputs indicate that the large majority of property records fell within modest positive or negative change ranges. This suggests descriptive variation across the assessed value distribution, consistent with a broad, administrative annual revaluation cycle. This does not establish causality between any external factor and assessed value patterns.

### Neighbourhood-code-level property value summary

Property value change metrics were also aggregated by `neighbourhood_code`, which is treated as a BC Assessment coded geography field. This produced a 30-row summary covering all property records in the dataset, including median assessed values, median and mean percentage changes, and the share of records with increases, decreases, or missing values within each coded area. This does not establish causality between neighbourhood-level variation and any external factor such as permit activity.

---

## 4. Geography Limitation

`NEIGHBOURHOOD_CODE` is a coded geography field sourced directly from the City of Vancouver Property Tax Report. It is a 3-digit identifier assigned by BC Assessment (BCA) to organise properties into appraisal areas. The City of Vancouver publishes the numeric code in the open data file but does not publish a corresponding readable neighbourhood name field. According to the City's open data documentation, BCA does not supply the City with the referencing neighbourhood name information.

No official public lookup was found during research that maps these codes to readable neighbourhood names. A code-to-name mapping may exist in BC Assessment restricted or commercial data products, but those are not part of the public City of Vancouver open-data files used in this project.

As a result, the project keeps this field as `neighbourhood_code` rather than substituting inferred or manually guessed labels. The 30 BCA-coded areas are not assumed to match the City of Vancouver's 22 Local Areas. These are distinct geography systems, and treating them as equivalent without a verified official crosswalk would introduce unverifiable geographic labelling.

Two future paths exist for resolving this limitation: obtaining an official BC Assessment code-to-name mapping (the path recommended by the City's open data documentation), or building a separate City Local Area analysis using an appropriate spatial assignment method. Either would be a distinct analytical step.

---

## 5. Interpretation Limits

The following caveats apply to all outputs and any conclusions drawn from this project:

- **Assessed values are not sale prices.** BC Assessment valuations are administrative estimates used as the basis for property tax calculations. They are not MLS transaction prices, market values, or appraisals. Year-over-year changes in assessed value reflect changes in the administrative valuation and should not be interpreted as direct measures of market appreciation or depreciation.
- **Building permits are not completed housing units.** A permit is an authorisation to begin regulated construction. It does not confirm that the project was completed, occupied, or added to the housing stock. A single permit may cover a small renovation, a single-family house, or a large multi-unit building.
- **Declared project value is not final construction cost or market value.** It is an applicant-estimated figure provided at the time of permit issuance and is not independently verified.
- **The analysis is descriptive only.** This project does not claim that permit activity caused property value changes, or that any observable pattern has a specific causal driver. All findings describe patterns in administrative records.
- **No forecasting.** This project does not forecast future housing prices, permit volumes, or assessed value trends.

---

## 6. Skills Demonstrated

This project demonstrates the following analytical and technical skills:

- **Public data analysis** — sourcing, evaluating, and combining open government datasets
- **Data cleaning** — handling nulls, type coercion, and ambiguous categorical fields across large files
- **Feature engineering** — computing derived metrics with explicit null propagation and boundary rules
- **Chunked processing of a large dataset** — reading a 443 MB CSV in 100,000-row blocks using `pd.read_csv(..., chunksize=100_000)` to stay within memory constraints
- **Reproducible Python workflows** — notebooks structured as documented, ordered pipelines with explicit validation checks and safety flags
- **Data validation** — assertion-based checks at every processing stage to confirm row counts, column presence, and output integrity before committing results
- **Descriptive visualization** — portfolio-ready charts exported at 300 dpi with proper axis labels, caveats, and no unsupported interpretive claims
- **Methodology documentation** — decision log, data dictionary, and methodology notes maintained throughout the project
- **Business-oriented communication** — findings presented with appropriate caveats, written for a recruiter or non-technical stakeholder audience

---

## 7. Recommended Next Step

The most impactful next step for this project is to build a Power BI dashboard using the processed CSV outputs as data sources. The processed files are small, clean, and already validated — they are well suited for direct import without any further transformation.

Connecting the dashboard to the processed outputs rather than the raw files keeps the workflow fast, avoids re-running the full processing pipeline, and ensures the dashboard reflects validated, committed data.

The dashboard could include the following views:

- **Permit count by year** — bar or line chart showing annual residential permit volumes from 2019 to 2024
- **Declared permit project value by year** — bar or line chart showing annual totals, with a median series included to reduce the influence of large single projects
- **Property value change distribution** — horizontal bar chart of the 10 percentage-change buckets across all property records
- **Median assessed value change by BCA-coded neighbourhood code** — horizontal bar chart of the 30 coded geography areas, clearly labelled as BCA codes rather than readable neighbourhood names
- **Caveats and methodology panel** — a dedicated text panel noting that assessed values are not MLS sale prices, permits are not completed units, and neighbourhood codes are BCA-coded geography fields with no public name mapping

This dashboard would make the analytical outputs accessible to a non-technical audience while keeping the underlying data clean and the methodology transparent.
