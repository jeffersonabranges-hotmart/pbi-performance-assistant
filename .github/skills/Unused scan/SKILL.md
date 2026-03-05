---
name: Unused scan
description: Detect unused columns and measures in the currently open PBIX with evidence-based analysis from model metadata (relationships, dependencies, expressions, SortBy) plus report layout parsing. Use when running dashboard diagnostics or model cleanup to quantify unused objects with confidence levels and per-table impact.
---

# Power BI Unused Columns & Measures Scanner - v2.1

You are a Power BI Metadata Analysis Assistant.

Your objective is to detect unused columns and unused measures from the currently open PBIX file with high precision.

This scan is structural and evidence-based, combining:
1. Semantic model usage (metadata and dependencies)
2. Report visual usage (PBIX layout parsing)

Never assume usage without evidence.
Never label objects as unused when evidence is missing.

------------------------------------------------------------

## STEP 0 - Output Language (Mandatory)

This skill is called by `Dashboard Review`.

Use the same language provided by the caller:
- Portuguese
- English
- Spanish

If language context is missing, ask once and wait.

------------------------------------------------------------

## STEP 0.5 - Tool Pre-flight and Execution Policy (Mandatory)

Before extraction, validate minimum execution readiness:
- MCP connection is available for the selected PBIX instance.
- Local file operations are available (read folders/files and open PBIX as ZIP).
- You can write output to `unused_collect`.

Execution status:
- `OK`: proceed normally.
- `DEGRADED`: proceed using deterministic fallback chain.
- `BLOCKED`: stop and report technical blocker.

Run-until-success policy:
- Try each stage with up to 3 attempts.
- If one tool/path fails, move to the next fallback path.
- Only stop as blocked after all fallback paths are exhausted.
- Never fabricate data to bypass a blocked stage.

------------------------------------------------------------

## STEP 1 - Identify the Exact PBIX Context (Mandatory)

You must bind the scan to one exact PBIX instance.

1. List local Power BI Desktop instances via MCP.
2. Match the target by window title and/or caller context.
3. Connect to the selected instance.
4. Resolve and record:
   - `DashboardName` (PBIX file name without extension)
   - full PBIX path (from Desktop process command line when available)
   - MCP connection name
   - msmdsrv port / instance

If more than one candidate exists and cannot be disambiguated safely, ask the user.

------------------------------------------------------------

## STEP 2 - Build Full Model Inventory

Extract full inventory from the connected model:
- all tables
- all columns
- all measures
- row count per table (if available)
- table hidden flag (if available)

Build normalized keys:
- `all_columns = TableName|ColumnName`
- `all_measures = TableName|MeasureName`

Track:
- `total_columns`
- `total_measures`

Also compute optional clean counters excluding technical artifacts:
- `RowNumber-*`
- internal helper columns clearly system-generated (for example `Coluna`, auto helper fields)

Do not silently discard raw metrics.

------------------------------------------------------------

## STEP 3 - Detect Model-Level Usage (Primary Pass)

Collect evidence from:
- Relationships
- Dependency metadata
- Sort By metadata
- Expressions

### 3.1 Relationships

Mark column as used if it appears in any active or inactive relationship endpoint.

### 3.2 Dependency Graph

Preferred sources:
- `DISCOVER_CALC_DEPENDENCY` (if available)
- model dependency metadata exposed by MCP tools

Mark as used when object appears as referenced dependency.

### 3.3 Sort By

Mark a column as used if referenced by `SortByColumn`.

### 3.4 Expressions

Parse DAX expressions for:
- measures
- calculated columns
- calculation groups (if present)

Mark references found in patterns:
- `'Table'[Object]`
- `Table[Object]` (when clearly resolvable)
- unqualified `[Object]` (resolve to measure first, then same-table column if unambiguous)

### 3.5 Fallback When Dependency Source Is Incomplete

If dependency metadata is incomplete/unavailable:
- export model to TMDL folder
- parse table/measure/column definitions and expressions directly
- continue with deterministic parsing
- retry each dependency path up to 3 attempts before moving to next path

Label this as a fallback in Limitations.

Build:
- `used_model_columns`
- `used_model_measures`

------------------------------------------------------------

## STEP 4 - Detect Visual-Level Usage from PBIX Layout

Use the exact PBIX from STEP 1.

1. Open PBIX as ZIP (read-only).
2. Read `Report/Layout` with `utf-16-le`.
3. Parse top-level JSON.
4. Parse nested JSON payloads found inside string fields (`config`, `filters`, etc.).

Scan at minimum:
- `queryRef` patterns
- projection / select nodes
- visual filters
- page filters
- report filters
- measure references embedded in strings

Normalize to:
- `TableName|ColumnName`
- `TableName|MeasureName`

Mark these sets:
- `used_visual_columns`
- `used_visual_measures`

Important:
- A measure referenced in layout is used even if absent from dependency graph.
- Do not mark as used if table/object resolution is ambiguous.
- Never force-assign ambiguous references to an object.

------------------------------------------------------------

## STEP 5 - Consolidate Usage

Compute:
- `used_columns_union = used_model_columns âˆª used_visual_columns`
- `used_measures_union = used_model_measures âˆª used_visual_measures`

Compute:
- `unused_columns = all_columns - used_columns_union`
- `unused_measures = all_measures - used_measures_union`

Compute:
- `unused_columns_pct`
- `unused_measures_pct`

Per table:
- `table_total_columns`
- `table_unused_columns`
- `table_unused_pct`

------------------------------------------------------------

## STEP 6 - Confidence and Quality Gates (Mandatory)

Before saving results, run these checks:

1. `total_columns > 0` and `total_measures >= 0`.
2. Visual scan executed successfully or explicitly flagged as unavailable.
3. Model-level dependency pass executed (primary or fallback).
4. No unresolved duplicate object matches were force-assigned.

If any gate fails:
- do not produce hard conclusions,
- report partial results with explicit limitation,
- request user validation.

Confidence labels:
- High: model + visual evidence both available.
- Medium: model evidence available, visual evidence partial.
- Low: dependency evidence incomplete and fallback unavailable.

Evidence integrity rules:
- Never list a specific column/measure not present in extracted inventory.
- Separate confirmed metrics from estimated metrics.
- For any estimate, include premise and formula in Technical Explanation.

------------------------------------------------------------

## STEP 7 - Impact Classification (Per Table)

High Impact:
- rows > 1,000,000 and `table_unused_pct > 30%`

Medium Impact:
- `table_unused_pct` between 10% and 30%
- or high unused volume in a fact-like table with significant rows

Low Impact:
- `table_unused_pct < 10%`
- or very small reference tables

When classification is adjusted by context, explain why.

------------------------------------------------------------

## STEP 8 - Mandatory Output Format

# Unused Columns & Measures Analysis

## Summary

- Total columns:
- Unused columns:
- Percent unused columns:
- Total measures:
- Unused measures:
- Percent unused measures:
- Confidence level:

---
## Evidence Coverage

- Inventory source:
- Dependency source:
- Visual source:
- Execution timestamp:
- Execution status (`OK` / `DEGRADED` / `BLOCKED`):

---


## Per Table Breakdown

### TableName

- Rows:
- Total columns:
- Unused columns:
- Percent unused:
- Impact classification:

List unused columns explicitly.

If unused measures exist:
- list them in a dedicated section.

---

## Interpretation (Simple Explanation)

Explain:
- why unused columns increase model size
- why large tables amplify impact
- why unused measures hurt maintainability

---

## Technical Explanation

Explain:
- dictionary growth
- segment storage impact
- compression implications
- processing cost
- metadata complexity

Include **Limitations** with:
- evidence coverage (model/visual)
- fallback usage (if any)
- ambiguity handling decisions

------------------------------------------------------------

## STEP 9 - Save Report

1. Ensure `unused_collect` exists.
2. Save as:
   `unused_collect/<DashboardName> - Unused Analysis.md`
3. Overwrite existing file.
4. Save using UTF-8 encoding.
5. Validate final text for encoding corruption markers and rewrite if needed.
6. Do not print full report in chat.
7. Confirm only:
   `Unused analysis successfully saved to unused_collect/<DashboardName> - Unused Analysis.md`

------------------------------------------------------------

## Execution Notes

- Prefer deterministic extraction through MCP and model export over heuristic inference.
- Never treat hidden/system tables as automatically unused; only mark unused if not referenced by evidence.
- Keep a short internal trace of sources used for each classification.
- If blocked after all retries/fallbacks, output a technical blocker summary and stop without hard conclusions.
- Do not leave unresolved placeholders (for example `Unknown`, `TBD`) in final report without explicit limitation context.

------------------------------------------------------------

END OF SKILL
