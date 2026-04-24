# Session Log: Referee Response Tables and Writing — 2026-04-24

**Status:** In progress (response document under active drafting)

## Goal

Build out the response document for the Management Science R&R, focusing on:
1. R1 Comment 1 (tighter FE structure / shift selection)
2. R2 Comment 1 (reverse causality / collider bias)
3. R2 Comment 3 (specification simplification)
4. R2 Comment 5 (clustering)

Each substantive response includes a referee-only Stata do-file in `_code/referee_only/`, a Stata-generated tabular fragment, a hand-edited wrapper `.tex` in the Overleaf response folder, and substantive prose in `response.tex`.

## What was done this session

### New referee-only do-files

- **`spec_comparison.do`** — within-Lead vs between-spec same-gender effect comparison (3 cols). Outputs `spec_comparison.tex`.
- **`shift_selection.do`** — tests whether physician gender / team composition predicts shift assignment (day vs night) conditional on hospital × time × DOW FE. 3 cols (solo gender, team gender, team-type dummies). Outputs `shift_selection.tex`.
- **`reverse_causality.do`** — three panels:
  - Panel A: Lead × Assist age and experience interactions (3 cols: main spec, continuous interactions, fully interacted quintiles). Outputs `reverse_causality_main.tex`.
  - Panel B: age- and experience-adjusted descriptive on team-formation rate by physician gender (display-only, not tabled).
  - Panel C: SMM × Lead-gender interaction in `is_team` regression, 3 cols (no patient controls, +patient controls, +Lead FE). Outputs `reverse_causality_selection.tex`.

### Wrapper `.tex` files (in `response/referee_tables/`)

Created/updated for each table: `<name>_wrapper.tex` matching the paper's `\floatfoot` style. Used `\begin{spacing}{1}\footnotesize\justify` inside `\floatfoot` to override the response doc's global `\doublespacing` (the response doc applies `\doublespacing` after `\maketitle`, which would otherwise inflate the notes text).

### Substantive responses written in `response.tex`

- **R1.1** (tighter FE / shift selection / within-Lead identification): explains why we use hospital × year × qtr × DOW (not admission-hour, which loses ~30% of sample post-2010 only and creates singletons), shift-selection robustness, and the within-Lead specification as a third defense against shift-margin selection.

- **R2.1(a)** (reverse causality / collider bias): reframes R2's concern as collider bias from selection into the team sample, not strict reverse causality. Walks through the W (unobserved intrapartum signal) chain. Three layers of defense: (i) SMM is post-delivery by clinical definition (cite Main et al. 2016, ACOG 2016), (ii) within-Lead FE absorbs the calling threshold (the very mechanism R2 invokes), (iii) age/experience interactions don't change estimates.

- **R2.1(b)** (selection-into-team test): adjusted descriptive ("women X pp more likely to appear in teams") with caveats, then 3-spec SMM × Lead-gender interaction. Lead-FE column is the most probative test. Honest closing limitation noting we satisfy R2's null-interaction condition but not the equal-rates condition (with alternative explanations: shift changes, practice setting).

- **R2.3** (parameterization): adopts FF leave-out with 3 dummies, cites Salvestrini & Ronchi precedent. Dropped the "parallels within-Lead spec" argument (technically wrong on reflection). Reframed not to claim β₃ is "the central substantive question."

- **R2.5** (clustering): switched all regressions to hospital-level clustering (per Abadie et al. 2023). Noted exceptions: team FE within-pair and challenging-conditions hospital-strain analyses lost significance under hospital clustering — both dropped from the paper given R1/R2 concerns about MF vs FM and streamlining.

## Stata gotchas learned this session

- **`file write` eats `$`**: use `_char(36)` for literal dollar sign. Compound quotes `` `"..."' `` protect `$` inside `local` assignments but not inside `file write`.
- **`\setstretch{1}` doesn't always take effect inside `\floatfoot`** — use `\begin{spacing}{1}...\end{spacing}` instead. Requires `setspace` package (already loaded).
- **`\floatfoot` requires `floatrow` package** (added to response.tex preamble).
- **`\resizebox`** causes width mismatch between tabular and `\floatfoot` notes — drop it for response-doc tables.
- **After `margins, post`**, `e(b)` and `e(V)` are replaced. Compute regression-based scalars BEFORE running margins.
- **For p-values**: use `test <var>` then `r(p)` — more robust than manual `2*ttail(e(df_r), abs(t))` because the latter relies on a scalar `t` that gets reused throughout the file and can give wrong values when blocks are re-run interactively.
- **Continuous interactions in Stata**: `c.x##c.y` (with `##`) gives main effects + interaction; `c.x#c.y` (with `#`) gives just the interaction. Use `#` if main effects are already controlled for via quintile dummies.
- **`i.cat##i.cat2`** works only if both are categorical variables. If they're 5 separate dummies, either reconstruct a categorical (`gen catvar = 1*tile1+2*tile2+...`) or use a single categorical variable directly.

## Workflow conventions reinforced this session

- **Always confirm before editing `response.tex`** or any hand-edited `.tex` in the Overleaf folder. The user has in-flight Overleaf edits that may not have synced to Dropbox.
- **Stata-generated tabulars (`*.tex` in referee_tables/)** are fine to overwrite — they get regenerated.
- **Two-file pattern for tables**: Stata writes only the tabular; wrapper `.tex` has caption/label/notes; main doc does `\input{...wrapper}`.

## Open issues / TODOs

- **R2.1(a) age/experience interactions**: results are stable across specs (interaction coefficients within rounding of baseline). Need to fill in the [PLACEHOLDER] in response text with actual values.
- **R2.1(b) descriptive**: need to fill in the adjusted "X pp" value once Stata runs the new `test`-based p-value version.
- **R2.2** (why MM teams perform worst), **E1** (synchronous vs handoff teams), **E3** (same-gender team differences): not yet drafted.
- **Solo sample selection** (R1#2, AE#6): not addressed.
- **Monte Carlo shuffle test** (R2#4): not implemented.
- **Predict team vs solo** (R1#2.1): not implemented.

## Quality / verification status

- `spec_comparison.tex`, `shift_selection.tex`, `reverse_causality_main.tex`, `reverse_causality_selection.tex` — all generated and rendered cleanly in Overleaf.
- Response document compiles without errors (modulo placeholder text).
- One Stata bug fixed mid-session: scalar `t` was reused throughout `reverse_causality.do` causing wrong p-value display in Panel B; replaced with `test`-based p-value computation.
