
## General logic used for scenario construction

### PV vs PV+ESS

For most cases:

- `include_ess = FALSE` means ESS-related rows are excluded
- `include_ess = TRUE` means ESS-related rows are included

ESS-related rows commonly include:

- the ESS sheet
- ESS Mount in SBOS
- ESS Labor in Fieldwork
- ESS Acquisition in Officework
- ESS Management in Other
- ESS replacement items in O&M

### Indirect cost logic

When `include_indirect = TRUE`, indirect costs are included.

These usually include:

- Officework
- Other
- indirect parts of some sheets
- indirect rows flagged by the model

When `include_indirect = FALSE`, those rows may be excluded depending on the workbook configuration.

---

## Common sheet interpretation

### System
Summary-level component totals and scenario toggles.

### Module
Module manufacturing and delivery cost elements.

### Inverter
Inverter manufacturing and delivery cost elements.

### ESS
Battery / storage manufacturing and delivery cost elements.

### SBOS
Structural balance-of-system hardware.

### EBOS
Electrical balance-of-system hardware.

### Fieldwork
Field installation labor and related activities.

### Officework
Permits, design, logistics, and customer acquisition.

### Other
Business overhead, taxes, distributor markup, and profit.

### O_M
Annual operation and maintenance costs.

### Factors
Shared assumptions, conversion factors, and model parameters.

---

## Why this design is useful

This schema makes the data suitable for:

- filtering by system type
- comparing PV to PV+ESS
- comparing MSP to MMP
- analyzing direct vs indirect costs
- grouping hardware vs soft cost drivers
- computing total system costs
- building dashboards and charts
- feeding cost-benefit or techno-economic analysis workflows

Because everything is in one normalized table, the same dataset can support both descriptive analysis and quantitative modeling.

---

## Recommended conventions

To keep the dataset clean, use these conventions consistently:

### `scenario_name`
- PV
- PV+ESS
- ESS

### `price_type`
- MSP
- MMP

### `cost_group`
- Hardware
- Soft Cost
- Labor
- Overhead
- O&M
- Summary

### `cost_scope`
- Direct
- Indirect
- Mixed
- Annual_O&M
- System_Summary

---

## Data quality notes

When building rows from screenshots or manual extraction:

- preserve values exactly as shown
- do not silently reconcile mismatches
- if a system-summary value and sheet-detail value differ, keep both as captured and flag later in QA
- preserve comments where possible
- document any derived rows clearly

This is important because some workbook values may differ across sheets depending on toggles, distortions, rounding, or summary logic.

---

## Recommended next files

It is useful to maintain the following companion files:

- `pvscm_master_dataset.csv` — main analysis table
- `pvscm_data_dictionary.md` — this guide
- `pvscm_qa_checks.csv` — optional validation results
- `pvscm_build_notes.md` — optional notes on extraction decisions

---

## Final summary

This master dataset is a normalized, traceable representation of PVSCM cost-model outputs.

It is designed so that:

- one row = one cost item
- one scenario = one exact system configuration
- one schema works across RPV, UPV, and APV
- the dataset remains auditable, comparable, and analysis-ready

This structure supports energy analysis, techno-economic evaluation, and cost-benefit analysis with much less manual cleanup than working directly from multiple workbook sheets.
