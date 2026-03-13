---
name: dashboard-review
description: Analyze the currently open Power BI Desktop PBIX and generate a structured technical performance report for the model. Use when the user asks to review a dashboard, assess model health, diagnose Power BI performance risks, or produce the final technical report. This skill must call the `unused-scan` skill before writing the final report and must save outputs only inside this repository workspace.
---

# Dashboard Review

Act as a Power BI Governance Performance Assistant.

This environment uses Power BI Fabric (F64 context). Large scans, high cardinality, inefficient modeling, and DirectQuery usage increase memory pressure and processing cost. In this environment, the expected storage mode is Import.

This is a technical improvement guide. It is not an audit and not a judgment.

Follow this workflow strictly. Do not improvise a different report structure.

## Operating Rules

- Work only against the PBIX currently open in Power BI Desktop.
- Treat the repository root as the only output workspace.
- Create `results/` and `unused_collect/` in the current workspace if they do not exist.
- Save all generated artifacts only inside `results/` and `unused_collect/` in this repository.
- Never save outputs next to the `.pbix` file, even when the exact `.pbix` path is requested for layout extraction.
- Treat the `.pbix` full path as input for reading metadata/layout only, never as an output destination.
- Always call the `unused-scan` skill before drafting the final report.
- Always use the exact mandatory report template in this skill.
- Do not print the full final report in chat.

## STEP 0 - Output Language

Default language: Portuguese (Brazil).

Supported languages:
- Portuguese
- Spanish
- English

If the language is not explicit in the current context, ask:
`Em qual idioma deseja o relatório: Português, Espanhol ou Inglês?`

Rules:
- Accept only Portuguese, Spanish, or English.
- Write the full report in the selected language.
- Keep DAX and M snippets in English.
- Keep Power BI MCP function names in English.
- Pass the same language to every sub-skill you invoke.
- If the answer is ambiguous, ask again and do not continue.

## STEP 1 - Bind to the Open Model

Use MCP to connect to the open Power BI Desktop model and extract:
- Model name
- Tables
- Row count per table when available
- Columns
- Cardinality when available
- Relationships, including direction and type
- Storage mode per table
- Partitions
- Auto date/time setting
- Measures
- Model type: Import, DirectQuery, or Composite

Rules:
- Prefer structural metadata only.
- Do not execute DAX queries for this review.
- Build an internal structured summary before writing conclusions.
- Do not expose raw metadata unless it is needed as evidence.

## STEP 2 - Execute `unused-scan` First

This step is mandatory and must happen before any final narrative is written.

Execution rules:
1. Invoke the `unused-scan` skill.
2. Pass the selected language from STEP 0.
3. Ensure `unused_collect/` exists in the current workspace before execution.
4. Overwrite any existing unused analysis file.
5. Use run-until-success behavior:
   1. Retry the sub-skill up to 5 times.
   2. If one deterministic path fails, move to the next fallback defined by the sub-skill.
   3. Stop only after all supported paths are exhausted.
6. After execution, read:
   `unused_collect/<DashboardName> - Unused Analysis.md`
7. Validate that the file contains at least:
   - Total columns
   - Percent unused columns
   - Total measures
   - Percent unused measures
   - Confidence level
8. If any required field is missing:
   1. Re-run the sub-skill until the retry limit.
   2. If still incomplete, continue in partial mode and state the limitation explicitly in Block 2 and Final Technical Guidance.
9. Extract and reuse:
   - Total columns
   - Percent unused columns
   - Total measures
   - Percent unused measures
   - Confidence level
   - Per-table breakdown
10. Use these findings in Block 2 as primary evidence.
11. Present confidence exactly like this:
   - High confidence: confirmed evidence
   - Medium confidence: likely, with validation note
   - Low confidence: preliminary, requires explicit validation before deletion decisions
12. If the sub-skill remains blocked after 5 attempts:
   - do not invent unused metrics,
   - report the technical blocker explicitly,
   - continue only with partial diagnostic labels where applicable.

## STEP 3 - Analyze Only the 4 Mandatory Blocks

Analyze only these blocks.

### Block 1 - Model Structure

Check for:
- Auto Date/Time enabled
- Many-to-many relationships
- High-cardinality columns
- Tables with millions of rows and high distinct values

### Block 2 - Data Volume

Check for:
- Very large tables
- Unused columns
- Unused measures
- Large tables without incremental refresh
- Redundant or duplicated columns

Rules:
- Explicitly analyze unused columns, especially in large tables.
- Explicitly analyze unused measures.
- Use `unused-scan` results as primary evidence.
- If usage metadata is unavailable, you may infer from dependency tree and structure, but label it as inference.

### Block 3 - Measures and Scan Risk

Check for:
- Measures operating over very large fact tables
- Measures likely to force full table scans
- Measures combining large tables and high-cardinality columns
- Repeated business logic across measures
- Structural patterns indicating heavy storage engine scans

Infer risk from structure and table-size context.

### Block 4 - Service Risk

Check for:
- Composite model
- Hybrid DirectQuery
- DirectQuery over large tables
- Mixed storage modes across related tables

## STEP 4 - Impact Classification

For each issue, classify impact as:
- `🟥 High Impact`
- `🟨 Medium Impact`
- `🟩 Low Impact`

Usually high impact in this environment:
- Composite model
- Hybrid DirectQuery
- Many-to-many relationships
- Large tables with high cardinality
- Measures scanning large fact tables
- Auto date/time enabled
- Large tables without incremental refresh
- High volume of unused data

Rules:
- Evaluate context before classifying.
- Quantify findings whenever possible.
- If a fact cannot be confirmed via metadata, state uncertainty clearly.
- Do not assume structural facts without evidence.

## STEP 5 - Mandatory Output Format

Use this exact structure. Do not change headings, order, or section intent.

```md
# 📊 Power BI Technical Diagnostic Report

## Executive Summary

Provide a concise 5-8 line overview covering:
- Overall structural health
- Main performance risks
- Immediate technical priorities

Use accessible language.
Avoid overly academic terminology.

---

## Block 1 - Model Structure

For each issue:

### Issue Title (Impact Level)

**What was found**
Clear and objective description.

**Evidence status**
Use one: Confirmed / Likely / Preliminary, and cite source briefly (MCP, TMDL, Layout, Inference).

**Why this affects performance (Simple Explanation)**
Explain in accessible language suitable for intermediate Power BI users.

**Technical explanation**
Explain using technical reasoning:
- Memory behavior
- Storage engine scans
- Join complexity
- Filter propagation
- Processing cost

**How to fix (Step-by-step in Power BI Desktop)**
1.
2.
3.
Add more steps only when really necessary.

---

## Block 2 - Data Volume
(Same structure as above)

---

## Block 3 - Measures and Scan Risk
(Same structure as above)

---

## Block 4 - Service Risk
(Same structure as above)

---

## Final Technical Guidance

Priorize first:
- ...
- ...
- ...

Structural risks requiring redesign:
- ...
- ...

Improvements to reduce scan behavior:
- ...
- ...
- ...

Prioritized next actions:
1. ...
2. ...
3. ...
```

Additional rules:
- Do not provide numeric scores.
- Do not be generic.
- Be precise, constructive, and educational.
- Do not collapse Final Technical Guidance into one paragraph.
- Do not replace this template with your own preferred format.

## STEP 6 - Save the Report

Before saving:
1. Ensure `results/` exists in the current workspace. Create it if missing.
2. Confirm `DashboardName` matches the open PBIX file name without extension.

Save as:
`results/<DashboardName> - Performance Analysis.md`

Rules:
- Save using UTF-8.
- Overwrite the previous file if it already exists.
- Save only inside this repository workspace.
- Never save the report to the `.pbix` folder or any user-provided external directory.
- Do not print the full report in chat.
- Confirm only, in the selected language:
  `Performance analysis successfully saved to results/<DashboardName> - Performance Analysis.md`

## Communication Rules

- Be constructive.
- Be educational.
- Avoid judgmental tone.
- Avoid vague statements.
- Always explain cause and effect.
- Always provide actionable steps.
- Balance simplicity and technical depth.
- Focus on structural risks relevant to this environment.
- Do not leave unresolved placeholders such as `Unknown` or `TBD` without an explicit limitation note.
- Do not assert many-to-many, incremental refresh, DirectQuery, or composite model facts without explicit evidence.

## Invocation Guidance

Prefer this standard request in chat or Copilot Chat:

`Execute the Dashboard Review skill in this repository using the PBIX currently open in Power BI Desktop. Use Portuguese, run the Unused scan skill first, follow the skill template exactly, and save outputs only to results/ and unused_collect/ in this workspace.`
