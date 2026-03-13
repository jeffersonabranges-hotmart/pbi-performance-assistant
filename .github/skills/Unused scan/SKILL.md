---
name: unused-scan
description: Detect unused columns and measures in the currently open Power BI Desktop PBIX with evidence-based analysis from model metadata and mandatory layout parsing. Use when running dashboard diagnostics, cleanup analysis, or as a required sub-step of `dashboard-review`. This skill must save outputs only inside this repository workspace.
---

# Unused Scan

Act as a Power BI Metadata Analysis Assistant.

Detect unused columns and unused measures from the currently open PBIX with high precision, combining:
1. Semantic model evidence
2. Report visual evidence from the PBIX layout

Never assume usage without evidence.
Never label objects as unused when evidence is missing.

## Operating Rules

- This skill is normally called by `dashboard-review`.
- Use the same output language provided by the caller.
- Bind the scan to exactly one open Power BI Desktop PBIX.
- Treat this repository workspace as the only output destination.
- Create `unused_collect/` in the current workspace if it does not exist.
- Save outputs only to `unused_collect/` in this repository.
- Never save outputs next to the `.pbix` file.
- The `.pbix` full path is only for reading the layout and related metadata, never for deciding where to save the report.

## STEP 0 - Output Language

Supported languages:
- Portuguese
- English
- Spanish

If language context is missing, ask once and wait.

## STEP 0.5 - Tool Pre-flight and Execution Policy

Before extraction, validate:
- MCP connection is available for the selected PBIX instance.
- Local file operations are available for reading folders/files and opening the PBIX as ZIP.
- You can create and write output to `unused_collect/` in the current workspace.

Directory rule:
- If `unused_collect/` does not exist, create it before extraction.
- If creation fails, retry up to 3 times and then mark execution as `BLOCKED`.

Execution status:
- `OK`: proceed normally.
- `DEGRADED`: proceed using deterministic fallback chain.
- `BLOCKED`: stop and report a technical blocker.

Run-until-success policy:
- Try each stage with up to 3 attempts.
- If one tool or path fails, move to the next deterministic fallback.
- Stop only after all fallback paths are exhausted.
- Never fabricate data to bypass a blocked stage.

## STEP 1 - Identify the Exact PBIX Context

Bind the scan to one exact PBIX instance.

1. List local Power BI Desktop instances via MCP.
2. Match the target by window title and/or caller context.
3. Connect to the selected instance.
4. Resolve and record:
   - `DashboardName` as PBIX file name without extension
   - full `.pbix` path when available
   - MCP connection name
   - msmdsrv port or instance

PBIX path fallback chain:
- First, use Desktop process command line or instance metadata.
- Then, use caller context, workspace files, and obvious same-folder candidates that match the connected model name.
- If the exact `.pbix` path still cannot be resolved safely, ask the user for the full `.pbix` path.
- Do not skip layout extraction just because the path was not auto-discovered.

Rules:
- Use the `.pbix` path only for reading layout content.
- Do not use the `.pbix` directory as output location.
- If more than one candidate exists and cannot be disambiguated safely, ask the user.

## STEP 2 - Build Full Model Inventory

Extract:
- all tables
- all columns
- all measures
- row count per table when available
- table hidden flag when available

Build normalized keys:
- `all_columns = TableName|ColumnName`
- `all_measures = TableName|MeasureName`

Track:
- `total_columns`
- `total_measures`

Also compute optional clean counters excluding obvious technical artifacts such as:
- `RowNumber-*`
- clearly system-generated helper columns

Do not silently discard raw metrics.

## STEP 3 - Detect Model-Level Usage

Collect evidence from:
- Relationships
- Dependency metadata
- Sort By metadata
- Expressions

### 3.1 Relationships

Mark a column as used if it appears in any active or inactive relationship endpoint.

### 3.2 Dependency Graph

Preferred sources:
- `DISCOVER_CALC_DEPENDENCY`
- model dependency metadata exposed by MCP tools

Mark an object as used when it appears as a referenced dependency.

### 3.3 Sort By

Mark a column as used if referenced by `SortByColumn`.

### 3.4 Expressions

Parse DAX expressions for:
- measures
- calculated columns
- calculation groups when present

Mark references found in patterns:
- `'Table'[Object]`
- `Table[Object]` when resolvable
- unqualified `[Object]` only when resolution is unambiguous

### 3.5 Fallback When Dependency Source Is Incomplete

If dependency metadata is incomplete or unavailable:
- export the model to a TMDL folder,
- parse table, measure, and column definitions directly,
- continue with deterministic parsing,
- retry each dependency path up to 3 attempts before moving to the next path.

Label this fallback in Limitations.

Build:
- `used_model_columns`
- `used_model_measures`

## STEP 4 - Detect Visual-Level Usage from PBIX Layout

Use the exact `.pbix` from STEP 1.

1. Open the PBIX as ZIP in read-only mode.
2. Read `Report/Layout` with `utf-16-le`.
3. Parse the top-level JSON.
4. Parse nested JSON payloads inside string fields such as `config` and `filters`.

Scan at minimum:
- `queryRef` patterns
- projection and select nodes
- visual filters
- page filters
- report filters
- drillthrough filters
- slicer state payloads
- bookmarks when present
- conditional formatting bindings when they expose field references
- tooltip bindings when present
- measure references embedded in strings

Normalize to:
- `TableName|ColumnName`
- `TableName|MeasureName`

Mark:
- `used_visual_columns`
- `used_visual_measures`

Rules:
- A measure referenced in layout is used even if missing from dependency graph.
- A column referenced only by layout is still used.
- Do not mark an object as used when table or object resolution is ambiguous.
- Never force-assign ambiguous references.

If `Report/Layout` is missing, unreadable, or malformed:
- retry extraction up to 3 times,
- confirm the `.pbix` path is correct,
- if path resolution is the blocker, ask the user for the full `.pbix` path,
- downgrade confidence only after all layout extraction paths are exhausted.

## STEP 5 - Consolidate Usage

Compute:
- `used_columns_union = used_model_columns ∪ used_visual_columns`
- `used_measures_union = used_model_measures ∪ used_visual_measures`
- `unused_columns = all_columns - used_columns_union`
- `unused_measures = all_measures - used_measures_union`
- `unused_columns_pct`
- `unused_measures_pct`

Per table, compute:
- `table_total_columns`
- `table_unused_columns`
- `table_unused_pct`

## STEP 6 - Confidence and Quality Gates

Before saving, validate:
1. `total_columns > 0` and `total_measures >= 0`
2. Visual scan executed successfully or is explicitly flagged as unavailable
3. Model-level dependency pass executed, primary or fallback
4. No unresolved duplicate matches were force-assigned
5. Layout-derived references were merged whenever layout extraction succeeded

If any gate fails:
- do not produce hard conclusions,
- report partial results with explicit limitations,
- request user validation if needed.

Confidence labels:
- High: model and visual evidence both available
- Medium: model evidence available, visual evidence partial
- Low: dependency evidence incomplete and fallback unavailable

Precision rules:
- Never classify an object as unused until relationships, dependencies, expressions, SortBy, and layout evidence have all been considered or explicitly ruled unavailable.
- Re-check each candidate unused measure against layout strings before finalizing.
- Re-check each candidate unused column against layout bindings, filters, slicers, drillthrough, and bookmarks before finalizing.

Evidence integrity rules:
- Never list a column or measure missing from extracted inventory.
- Separate confirmed metrics from estimated metrics.
- For any estimate, include premise and formula in Technical Explanation.

## STEP 7 - Impact Classification

High Impact:
- rows > 1,000,000 and `table_unused_pct > 30%`

Medium Impact:
- `table_unused_pct` between 10% and 30%
- or high unused volume in a fact-like table with significant rows

Low Impact:
- `table_unused_pct < 10%`
- or very small reference tables

If classification is adjusted by context, explain why.

## STEP 8 - Mandatory Output Format

Use this exact structure:

```md
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
- evidence coverage (model and visual)
- fallback usage if any
- ambiguity handling decisions
```

## STEP 9 - Save Report

1. Ensure `unused_collect/` exists in the current workspace.
2. Save as:
   `unused_collect/<DashboardName> - Unused Analysis.md`
3. Overwrite any existing file.
4. Save using UTF-8.
5. Validate final text for encoding corruption markers and rewrite if needed.
6. Do not print the full report in chat.
7. Confirm only:
   `Unused analysis successfully saved to unused_collect/<DashboardName> - Unused Analysis.md`

Rules:
- Save only inside this repository workspace.
- Never save the output to the PBIX folder or any external path provided by the user.

## Execution Notes

- Prefer deterministic extraction through MCP and model export over heuristic inference.
- Never treat hidden or system tables as automatically unused.
- Keep a short internal trace of evidence sources for each classification.
- If the exact `.pbix` path is required for layout extraction and cannot be discovered automatically, ask the user instead of silently skipping layout analysis.
- Treat layout extraction as core analysis, not an optional embellishment.
- If blocked after all retries and fallbacks, output a technical blocker summary and stop without hard conclusions.
- Do not leave unresolved placeholders such as `Unknown` or `TBD` without explicit limitation context.
