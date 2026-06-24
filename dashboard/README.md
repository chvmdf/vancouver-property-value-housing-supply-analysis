# Power BI Dashboard Plan

## 1. Dashboard Purpose

This dashboard is the executive presentation layer of the Vancouver Property Value & Housing Supply Analysis project. The Python notebooks handle all data cleaning, feature engineering, validation, and reproducible analysis. Power BI is used to communicate the final descriptive insights to stakeholders in a clear and accessible format, without re-implementing any analytical logic.

## 2. Recommended Data Sources

Load the following validated processed CSV files into Power BI:

- `data/processed/permit_count_by_year.csv`
- `data/processed/permit_project_value_by_year.csv`
- `data/processed/property_value_change_distribution_bins.csv`
- `data/processed/property_value_change_by_neighbourhood.csv`

Power BI should connect to these processed outputs rather than directly to the large raw datasets. The raw files are immutable source data and have not been validated for direct use in a reporting tool.

## 3. Suggested Dashboard Pages

### Page 1: Executive Overview

**Purpose:** Give a high-level view of residential permit activity and assessed property value change patterns across the full dataset.

**Recommended visuals:**

- Card: total number of years covered in permit outputs
- Line or column chart: residential permit count by year
- Line or column chart: declared residential permit project value by year
- Bar chart: distribution of assessed property value change buckets
- Text box: key methodology caveats (see Section 6)

---

### Page 2: BCA-Coded Geography View

**Purpose:** Show descriptive variation in assessed property value change across BC Assessment coded geographies (`neighbourhood_code`).

**Recommended visuals:**

- Bar chart: median percentage value change by `neighbourhood_code`
- Bar chart or table: property count by `neighbourhood_code`
- Table: `neighbourhood_code`, property count, median current assessed value, median percentage value change, share of properties with assessed value increase, share of properties with assessed value decrease
- Text box: geography limitation note explaining that `NEIGHBOURHOOD_CODE` is a BC Assessment coded geography field and does not correspond to readable neighbourhood names

## 4. Recommended KPIs and Measures

The following KPIs and measures can be built using the processed CSV fields. These are conceptual descriptions — do not invent specific numeric results.

- **Total permits:** sum of residential permit counts across all years
- **Total declared project value:** sum of declared project values across all permit records
- **Average annual permit count:** mean permit count per year
- **Median assessed value change:** median of the percentage change in assessed value across properties
- **Share of properties with assessed value increase:** proportion of properties where the assessed value change is positive
- **Share of properties with assessed value decrease:** proportion of properties where the assessed value change is negative

## 5. Visual Design Guidelines

- Use clear, descriptive chart titles on every visual.
- Avoid overcrowding pages — prioritize readability over density.
- Keep caveats visible on every page, not buried in tooltips or footnotes.
- Use "assessed value" consistently — not "price," "market value," or "sale price."
- Use "declared project value" consistently — not "construction cost" or "market value."
- Use `neighbourhood_code` or "BCA-coded geography" as labels — not readable neighbourhood names.
- Apply consistent number formatting: percentages to one decimal place, currency values with dollar signs and thousands separators.
- Keep the dashboard business-readable — avoid overly technical field names or statistical jargon without explanation.

## 6. Required Caveats Panel

The following caveats must appear somewhere visible in the dashboard, preferably on every page:

- Assessed values are administrative valuations set by BC Assessment, not MLS sale prices or market transaction prices.
- Building permits are not the same as completed housing units. A permit represents an application or approval, not a delivered unit.
- Declared project value is the value declared on the permit application. It is not the same as final construction cost or market value.
- `NEIGHBOURHOOD_CODE` is treated as a BC Assessment coded geography field. It does not map to readable neighbourhood names.
- No official public lookup was found to convert BCA neighbourhood codes into readable neighbourhood names.
- The 30 BCA-coded areas in this dataset are not assumed to match Vancouver's 22 Local Areas.
- This dashboard is descriptive. It does not make causal or forecasting claims.

## 7. Suggested Dashboard Title and Subtitle

**Title:** Vancouver Property Value & Residential Permit Activity

**Subtitle:** Descriptive analysis of public assessment and permit data

## 8. Build Sequence

1. Open Power BI Desktop.
2. Import the four processed CSV files listed in Section 2.
3. Check and correct data types for all columns (dates, numbers, text).
4. Rename tables with readable display names (e.g., "Permit Count by Year," "Property Value Change by Neighbourhood").
5. Build Page 1: Executive Overview using the visuals listed in Section 3.
6. Build Page 2: BCA-Coded Geography View using the visuals listed in Section 3.
7. Add a caveats panel to each page using a text box with the language from Section 6.
8. Export page screenshots to `dashboard/powerbi_screenshots/`.
9. Update the main README with dashboard screenshots when ready.

## 9. Files to Export Later

The following files may be added to this directory once the dashboard is built:

- `dashboard/vancouver_property_value_dashboard.pbix` — the Power BI project file
- `dashboard/powerbi_screenshots/executive_overview.png` — screenshot of Page 1
- `dashboard/powerbi_screenshots/bca_coded_geography_view.png` — screenshot of Page 2

The `.pbix` file and screenshots are not included at this stage.
