# Vancouver Property Value & Housing Supply Analysis

An exploratory analysis of the relationship between housing supply and property values across Metro Vancouver municipalities.

## Project Goal

Understand whether and how new housing supply affects assessed property values over time, and identify geographic or policy-driven patterns across the region.

## Repo Structure

```
data/
  raw/          # Original source files — never modified
  processed/    # Cleaned and joined datasets ready for analysis
notebooks/      # Numbered Jupyter notebooks (follow in order)
src/            # Reusable Python scripts and helpers
visuals/        # Exported charts and maps
dashboard/      # Interactive dashboard app
docs/           # Methodology, data dictionary, and narrative docs
```

## Setup

```bash
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter notebook
```

## Documentation

- [Data Dictionary](docs/data_dictionary.md)
- [Dataset Decision Log](docs/dataset_decision_log.md)
- [Methodology](docs/methodology.md)
- [Business Narrative](docs/business_narrative.md)

## Status

Scaffolded — data sourcing and EDA in progress.
