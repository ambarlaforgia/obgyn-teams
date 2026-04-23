# CLAUDE.MD — OB-GYN Teams: Gender Mix and Team Performance R&R

**Project:** Management Science R&R — "Gender Mix and Team Performance: Evidence from Obstetrics"
**Authors:** Ambar La Forgia & Manasvini Singh
**Status:** Revise & Resubmit

---

## Core Workflow

- **Language:** Stata (`.do` files). Do NOT write R code unless explicitly asked.
- **Paper:** LaTeX on Overleaf, synced via Dropbox
- **Estimation:** `reghdfe` for fixed effects linear probability models
- **Data:** Florida AHCA inpatient discharge records 2006-2018

---

## Critical File Paths

```
CODE (ONLY work here):
/Users/ambarlaforgia/AL2 Research Dropbox/Ambar La Forgia/Health_equity/_analysis/revision/_code/

OVERLEAF (paper LaTeX):
/Users/ambarlaforgia/AL2 Research Dropbox/Ambar La Forgia/Apps/Overleaf/[PAPER_REVISION] Gender Mix and Team Performance: Evidence from Obstetrics/

DATA:
/Users/ambarlaforgia/AL2 Research Dropbox/FL_data/inp_data/

RESULTS:
/Users/ambarlaforgia/AL2 Research Dropbox/Ambar La Forgia/Health_equity/_analysis/_results/

REFEREE REPORTS:
/Users/ambarlaforgia/obgyn-teams/master_supporting_docs/ManSci/referee_reports/
```

**WARNING:** Do NOT modify files outside the `revision/` code folder. Other folders contain legacy code.

---

## Code Execution Order

1. `1_inpatient_readin_NEW.do` — Data cleaning, physician ID matching, gender/age imputation
2. `2_inpatient_icdxwalk.do` — ICD-9/10 crosswalk
3. `3_inpatient_variables.do` — Construct analysis variables (SMM, risk factors, teams)
4. `4_inpatient_analysis.do` — Master file calling all analysis sub-files:
   - `_locals.do` → global macros
   - `4a_id_patientrisk.do` → predicted complication probability
   - `4a_id_physicianteams.do` → endogenous team formation tests
   - `4b_summary_stats.do` → summary statistics
   - `4c_robustness.do` → robustness checks
   - `4d_heterogeneity.do` → heterogeneity analyses
   - `5a_how_csection.do` → C-section mechanism
   - `5b_how_gender_effect.do` → gender mix effect holding prefs fixed
   - `6a_why_prefmapping.do` → preference incorporation
   - `6b_why_challenges.do` → team resilience
   - `7_response_R1.do` → R1 revision analyses

---

## Key Variables

| Variable | Description |
|----------|-------------|
| `smm` / `MC` | Severe maternal morbidity (25-condition CDC definition) |
| `lead_fem` / `doc_fem_atten` | Lead physician is female |
| `samegender_assist` | Assisting physician same gender as Lead |
| `csection` | C-section delivery |
| `vaginal` | Vaginal delivery |
| `solo` | Single-physician delivery (Lead = Assisting) |
| `team` | Two-physician delivery |

## Team Types
- **MM** (M_L–M_A): Male Lead, Male Assist
- **MF** (M_L–F_A): Male Lead, Female Assist
- **FM** (F_L–M_A): Female Lead, Male Assist
- **FF** (F_L–F_A): Female Lead, Female Assist

---

## Revision Priorities (from referee reports)

1. **Tighter FE as baseline**: hospital×year×quarter×DOW×shift (not just hospital×year + quarter)
2. **Address reverse causality**: Show assisting physician assisted in delivery, not post-complication
3. **4-category specification**: MM as leave-out group (replace samegender_assist coding)
4. **Hospital-level clustering**: Replace lead-physician clustering
5. **Scale back mechanisms**: Focus on establishing performance differences cleanly
6. **Solo sample robustness**: Replicate physician FE in exogenous solo subsamples
7. **Predict team vs solo model**: What drives team formation?
8. **Monte Carlo shuffle test**: Permute lead/assist roles within cases
9. **Streamline paper**: Single baseline spec, clear hierarchy of results

---

## Stata Conventions

- Use `reghdfe` for all fixed effects regressions
- Use `coefplot` for figure coefficient plots
- Store estimates with `eststo <name>`
- Cluster SEs at the **hospital** level (`$cluster = "vce(cluster faclnbr)"`) — per R2 / Abadie et al. (2023)
- Use `$ff_leaveout = "mm fm mf"` for the main 4-category team spec (FF omitted)
- Keep an `$main_int = "i.doc_fem_atten##i.samesex"` version only for margins/coefplot figures
- Always include `cap drop` before `gen` / `predict` to handle re-runs safely
- Globals are defined inline in `4_inpatient_analysis.do` — duplicate them at the top of standalone/referee do-files so they run on their own

---

## Table Generation Pattern (critical — two files per table)

**Every table uses a two-file pattern**:

1. **Stata writes ONLY the tabular** to `<name>.tex`
   - Uses `file write` with `_char(36)` for literal `$` (e.g. `$\times$`, `$R^{2}$`) — Stata eats `$` in `file write` otherwise
   - Output: `\begin{tabular}...\end{tabular}` only. No `\begin{table}`, caption, or notes.

2. **Hand-edited wrapper `.tex`** in Overleaf contains `\begin{table}`, caption, label, and notes — uses `\input{path/<name>}` to pull in the Stata-generated tabular.

3. **Main `.tex` document** does `\input{.../<name>}` (for paper) or `\input{.../<name>_wrapper}` (for response).

### Paths

**Paper tables** (Body.tex / Appendix.tex):
- Stata output: `$overleaf_tables` = `.../[PAPER_REVISION] .../tables/generated/<name>.tex`
- Wrappers: `.../tables/<name>.tex` (hand-edited)
- Source do-files: `_code/code_tables/table_<name>.do`, called via `include` from main analysis files

**Response tables** (response.tex):
- Stata output: `$overleaf_response_tables` = `.../[PAPER_REVISION] .../response/referee_tables/<name>.tex`
- Wrappers: `.../response/referee_tables/<name>_wrapper.tex` (hand-edited)
- Source do-files: `_code/referee_only/<name>.do` (self-contained — each has its own path/globals block and loads data directly)

### Response-doc table notes

- Wrap notes in `\begin{singlespace}...\end{singlespace}` (response doc is double-spaced)
- Use `\scriptsize` for smaller text
- Use `\noindent` to left-align (not centered)

---

## Response Document Conventions

- Main file: `.../response/response.tex`
- Standalone (no biblatex, no xr-hyper) — references to main paper eqs/tables use plain text ("Equation 1 in the manuscript")
- Referee comments: **plain text** (not italicized)
- Response marker: `\textbf{\textcolor{blue}{Response XX: }}`
- Placeholders: `\textcolor{red}{[PLACEHOLDER]}`
- Citations: plain text (e.g. "Abadie et al. (2023)"), not `\citet{}`
- Team-type macros `\PlainMM`, `\PlainMF`, `\PlainFM`, `\PlainFF` defined in preamble
