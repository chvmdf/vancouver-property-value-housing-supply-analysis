# Dataset Decision Log

Records every dataset considered for this project — included or excluded — and the reasoning. This creates an audit trail and prevents revisiting decisions already made.

---

## Included

### Property Tax Report
| Field | Detail |
|---|---|
| **File** | `data/raw/property_tax_report_raw.csv` |
| **Source** | City of Vancouver Open Data — Property Tax Report |
| **Accessed** | 2026-06-10 |
| **File size** | 443,785,841 bytes (~423 MB) |
| **Role** | Primary dataset for property value analysis |
| **Used for** | Assessed property values, land value, improvement value, year-over-year value change if supported |
| **Key limitation** | Assessed value reflects BC Assessment's annual valuation, not MLS sale price — does not capture actual transaction prices |

### Issued Building Permits
| Field | Detail |
|---|---|
| **File** | `data/raw/issued_building_permits_raw.csv` |
| **Source** | City of Vancouver Open Data — Issued Building Permits |
| **Accessed** | 2026-06-10 |
| **File size** | 56,671,596 bytes (~54 MB) |
| **Role** | Primary dataset for housing supply signal |
| **Used for** | Permit activity as a proxy for supply, time trends 2019–2024 |
| **Key limitation** | An issued permit is not a completed unit — there is a lag between permit issuance and occupancy, and some permits are never built |

---

## Excluded

### MLS / REBGV Transaction Data
| Field | Detail |
|---|---|
| **Decision** | Excluded |
| **Date** | 2026-06-10 |
| **Reason** | Proprietary data held by the Real Estate Board of Greater Vancouver; not publicly available, not reproducible, and not suitable as a dependency for an open portfolio project |

---

## Deferred / Stretch Goals

### Historical Property Tax Data (2016–2019)
| Field | Detail |
|---|---|
| **Decision** | Deferred |
| **Date** | 2026-06-10 |
| **Reason** | A historical merge extending coverage back to 2016 would strengthen trend analysis but is not required for the first portfolio version. Revisit if time permits or if the 2019–2024 window proves insufficient. |
