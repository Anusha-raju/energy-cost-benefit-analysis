# PVSCM Master Dataset Guide

## Purpose

This document explains the structure, meaning, and construction logic of the PVSCM master dataset used for energy cost analysis and cost-benefit analysis.

The goal of the master dataset is to convert PVSCM workbook outputs into a single analysis-ready table where:

- one row represents one cost item
- each row belongs to one exact system configuration
- each value can be traced back to a specific sheet and scenario

This makes it easier to perform:

- MSP vs MMP comparisons
- PV vs PV+ESS comparisons
- direct vs indirect cost comparisons
- residential vs utility vs aggregate comparisons
- cost breakdown charts
- cost-benefit analysis
- sensitivity analysis

---

## What this dataset represents

The source model is the **Photovoltaic System Cost Model (PVSCM)**.

The master dataset is designed to store outputs from:

- **RPV** = Residential PV
- **UPV** = Utility PV
- **APV** = Aggregate / All PV benchmark or reference case

It can capture the following scenario combinations:

- PV only
- PV + ESS
- ESS-only, if such a case is explicitly modeled
- indirect costs included
- indirect costs excluded
- MSP values
- MMP values

The dataset is normalized so that the same schema works for all system types and scenario combinations.

---

## How the master dataset is organized

The dataset is designed in **long format**.

That means:

- each row is one line item
- each line item has one price type
- each line item belongs to one scenario
- the same component can appear multiple times across different scenarios

Example:

- `Module | Cells | MSP | PV`
- `Module | Cells | MMP | PV`
- `Module | Cells | MSP | PV+ESS`
- `Module | Cells | MMP | PV+ESS`

This structure is better than a wide spreadsheet because it supports filtering, grouping, pivoting, aggregation, and plotting.

---

## Column groups

The dataset is organized into five logical groups:

1. Core identity columns
2. Cost hierarchy columns
3. Value columns
4. Unit / scaling columns
5. Traceability / quality columns

---

## Core identity columns

These tell you what system or scenario the row belongs to.

| Column | Type | Example | Why it exists |
|---|---|---|---|
| dataset | text | PVSCM | Source model family |
| version | text | 2024Q1 | Model version |
| system_type | text | RPV, UPV, APV | Residential / Utility / Aggregate |
| scenario_name | text | PV, PV+ESS, ESS | Human-readable scenario |
| include_ess | boolean | TRUE | Whether ESS is included |
| include_indirect | boolean | TRUE | Whether indirect costs are included |

### Notes

- `scenario_name` is the human-readable scenario label.
- `include_ess` is the machine-readable switch used to identify whether storage is part of the system.
- `include_indirect` shows whether indirect costs are included in the modeled scenario.
- In most cases:
  - `include_ess = FALSE` maps to `scenario_name = PV`
  - `include_ess = TRUE` maps to `scenario_name = PV+ESS`

---

## Cost hierarchy columns

These tell you where the number sits in the cost structure.

| Column | Type | Example | Why it exists |
|---|---|---|---|
| component | text | Module, Inverter, Officework | Main category |
| subcomponent | text | Cells, Labor, Permits | Line item |
| cost_group | text | Hardware, Soft Cost, Labor, Overhead, O&M, Summary | Analysis grouping |
| cost_scope | text | Direct, Indirect, Mixed, Annual_O&M, System_Summary | Useful for filtering |

### Notes

- `component` is usually aligned with the workbook sheet or main model category.
- `subcomponent` is the specific row or cost element within that component.
- `cost_group` is used for high-level analytics, such as grouping soft costs or hardware costs together.
- `cost_scope` helps distinguish whether a value is direct, indirect, annual O&M, or a system-level summary row.

### Suggested standard values for `cost_group`

- Hardware
- Soft Cost
- Labor
- Overhead
- O&M
- Summary

### Suggested standard values for `cost_scope`

- Direct
- Indirect
- Mixed
- Annual_O&M
- System_Summary

---

## Value columns

These are the actual numeric values.

| Column | Type | Example | Why it exists |
|---|---|---|---|
| price_type | text | MSP, MMP | Sustainable vs market |
| value | numeric | 321.47 | The actual number |
| currency | text | USD | Good practice |
| currency_year | text | 2023 | Important for inflation consistency |

### Notes

- `price_type = MSP` means **Minimum Sustainable Price**
- `price_type = MMP` means **Modeled Market Price**
- `value` stores the actual numeric amount shown or derived from the model
- `currency_year` should reflect the dollar-year used in the workbook assumptions

---

## Unit / scaling columns

These explain what the number is measured against.

| Column | Type | Example | Why it exists |
|---|---|---|---|
| basis_unit | text | kWdc, kWac, kWh-cap, m2, annual | Measurement basis |
| basis_value | numeric | 8, 7, 13.5, 38, 1 | Scenario basis |
| line_item_unit | text | optional | If you later store intrinsic units |
| normalized_to | text | optional | e.g. per_kWdc, per_system, annual |

### Notes

- `basis_unit` and `basis_value` describe the system size or production basis shown on the sheet.
- Examples:
  - system size of 8 kWdc
  - annual production of 1,500,000 kWdc
  - ESS basis of 13.5 kWh-cap
  - mounting area of 38 m2
- `line_item_unit` can store the intrinsic unit used in the sheet, such as hour, m2, kg, kWdc, or kWh-cap.
- `normalized_to` indicates how the value should be interpreted analytically, such as:
  - `per_kWdc`
  - `per_kWac`
  - `per_kWh`
  - `per_m2`
  - `annual`

---

## Traceability / quality columns

These make the dataset auditable and research-safe.

| Column | Type | Example | Why it exists |
|---|---|---|---|
| source_sheet | text | Module, System, O_M | Where it came from |
| comments | text | Cells from SE Asia... | Captures model notes, assumptions, or distortions |

### Notes

- `source_sheet` preserves the origin of each row
- `comments` stores the descriptive note from the workbook when available
- these fields are especially important when validating values or interpreting distortions such as:
  - tariffs
  - subsidies
  - passthrough credits
  - fixed cost assumptions
  - replacement-cost assumptions

---

## How we built the master dataset

The master dataset was built from PVSCM workbook screenshots and organized into a standardized row-based format.

### Step 1: Identify the scenario

Each batch starts by identifying:

- dataset
- version
- system type
- scenario name
- include_ess
- include_indirect

This information is usually read from the **System** sheet and scenario toggles.

### Step 2: Capture sheet-level basis information

For each sheet, the following are recorded when visible:

- system size
- annual production
- annual installations
- area basis
- storage basis

These become `basis_unit` and `basis_value`.

### Step 3: Extract cost rows

For each sheet:

- each visible cost element becomes one or more rows
- each cost element is stored separately for MSP and MMP where applicable
- row names are standardized into `component` and `subcomponent`

### Step 4: Assign analytical categories

Each row is categorized into:

- a cost group
- a cost scope

This supports later grouping and reporting.

### Step 5: Preserve traceability

Each row retains:

- the sheet it came from
- comments when visible
- the exact scenario settings used

This allows the dataset to be checked later against the workbook.

---

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

