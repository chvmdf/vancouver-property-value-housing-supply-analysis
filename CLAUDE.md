# CLAUDE.md — Project Conventions

## Project

Exploratory analysis of housing supply vs. property values in Metro Vancouver.

## Workflow Rules

- **Never modify files in `data/raw/`** — treat as immutable source of truth.
- All cleaning and transformation outputs go to `data/processed/`.
- Notebooks are numbered and meant to be run in order.
- Exported charts go to `visuals/`. Do not embed large images in notebooks.
- `src/` is for functions extracted from notebooks once they stabilize — no premature abstraction.

## Code Style

- Python only. No R.
- Pandas for tabular data. Matplotlib for charts (no Seaborn unless added to requirements).
- No inline magic numbers — assign constants with descriptive names.
- Keep notebooks narrative: each code cell should be preceded by a markdown cell explaining intent.

## Documentation

- Update `docs/data_dictionary.md` whenever a new column is used in analysis.
- Log every dataset decision (include or exclude) in `docs/dataset_decision_log.md`.

## Do Not

- Do not assume column names before inspecting actual data.
- Do not run analysis before documenting the plan in the relevant notebook's markdown cells.
- Do not commit files from `data/raw/`, `.venv/`, or any `__pycache__/`.
