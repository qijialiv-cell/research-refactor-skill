---
name: research-refactor
description: "Research code refactoring framework for R/Stata/Python projects. Enforces safe, incremental, auditable refactoring with dependency tracking and atomic commits."
---

# Research Engineering Skill
### PhD Dissertation Code Quality Framework
### Language: R · Stata · Python (Language-Agnostic Core)

---

## Rule Zero — Prerequisites (AI Must Enforce)

> Before touching any file in this project, an AI assistant **must**:
>
> 1. Read `SKILL.md` (this file) — understand the rules
> 2. Read `Part II — Architecture` of this file — understand the project world
> 3. Read `dependency_graph` (Part III) — understand what depends on what
> 4. Read `progress.md` — understand what is done, in-flight, and pending
>
> **If any of these are absent or unreadable: STOP. Report what is missing. Do not proceed.**

---

## Part I — Philosophy

### 1.1 The Zen of Python (Language-Agnostic Reading)

These principles apply to all research code, regardless of language.

```
Beautiful is better than ugly.
Explicit is better than implicit.          ← No magic numbers. No hidden globals.
Simple is better than complex.             ← The simplest correct solution wins.
Complex is better than complicated.        ← Necessary complexity is allowed.
Flat is better than nested.               ← Max 2 levels of nesting as default.
Sparse is better than dense.              ← One idea per line. White space is free.
Readability counts.                        ← Code is read 10× more than it is written.
Special cases aren't special enough to break the rules.
Although practicality beats purity.        ← If the right abstraction blocks progress, ship first.
Errors should never pass silently.         ← No bare `tryCatch`, no `capture` without logging.
Unless explicitly silenced.               ← Silence must be deliberate and documented.
In the face of ambiguity, refuse the temptation to guess.  ← STOP and ask.
There should be one obvious way to do it. ← Avoid multiple conventions in one project.
Now is better than never.                  ← Ship the refactor. Don't wait for perfection.
Although never is often better than right now.  ← Never refactor mid-analysis.
If the implementation is hard to explain, it's a bad idea.
If the implementation is easy to explain, it may be a good idea.
Namespaces are one honking great idea.    ← Functions, modules, configs. Use them.
```

### 1.2 Nelson Principles — Mapped to Research Engineering

From *Software Engineering for Data Scientists* (Catherine Nelson):

| Nelson Chapter | Research Engineering Mapping |
|---|---|
| Ch1 Good Code | Code must be readable by future collaborators, not just the author |
| Ch2 Maintainable Code | Organize by pipeline stage; measure and manage complexity |
| Ch3 Data Structures | Choose formats intentionally: CSV vs Parquet vs DTA vs RDS |
| Ch4 OOP / FP | **Prefer pure functions for data pipelines** — pure functions = reproducibility |
| Ch5 Errors / Logging | Never let silent failures corrupt downstream statistical outputs |
| Ch6 Formatting / Linting | Enforce consistent style via automated tools (not by hand) |
| Ch7 Testing | Unit-test data assertions; regression-test statistical outputs |
| Ch8 Documentation | Every module has a docstring; every project has a README |
| Ch9 Sharing | Lock environments (renv / conda / requirements.txt); use Git always |
| Ch12 Deploying | CI runs the full pipeline on every commit (GitHub Actions) |
| Ch13 Security | **Config/code separation** — no secrets or hardcoded paths in scripts |

### 1.3 Core Philosophy Statement

> The goal is not clever code.
> The goal is: **readable, maintainable, reproducible, auditable research.**
>
> A reviewer, collaborator, advisor, or your future self in 3 years
> must be able to run this pipeline from scratch and get identical results.
>
> **Only structural changes. Never silent result changes.**

---

## Part II — Architecture (Project Worldview)

### 2.1 Canonical Directory Structure

```
project_root/
│
├── config/                   ← ALL configuration lives here. Never in scripts.
│   ├── config.yaml           ← Main config: paths, parameters, seeds
│   ├── variables.yaml        ← Variable lists: controls, outcomes, fixed effects
│   └── environments/
│       ├── renv.lock         ← R environment lock
│       ├── requirements.txt  ← Python environment lock
│       └── stata_ado.txt     ← Stata ado packages and versions
│
├── data/
│   ├── raw/                  ← NEVER modified after download. Read-only.
│   ├── interim/              ← Intermediate outputs. Reproducible from raw/.
│   └── final/                ← Analysis-ready datasets.
│
├── src/                      ← All scripts/code lives here.
│   ├── 00_master.(do|R|py)   ← Single entry point. Calls everything else.
│   ├── 01_clean.(do|R|py)
│   ├── 02_construct.(do|R|py)
│   ├── 03_analysis.(do|R|py)
│   └── utils/                ← Shared helper functions/programs.
│
├── output/
│   ├── tables/
│   ├── figures/
│   └── logs/
│
├── tests/                    ← Data assertions and regression tests.
│
├── docs/
│   ├── SKILL.md              ← This file. AI behavior spec + architecture.
│   ├── progress.md           ← Project progress tracker.
│   └── README.md
│
└── .github/
    └── workflows/
        └── ci.yml            ← Automated pipeline run on push.
```

### 2.2 The Master Script Rule

> There must be **exactly one entry point**: `00_master`.
> Running `00_master` from a clean environment must reproduce
> all outputs in `output/` without any manual steps.

### 2.3 Config / Code Separation (Hard Rule)

**This is non-negotiable.**

All of the following must live in `config/config.yaml`, never hardcoded in scripts:

- File paths (absolute or relative)
- Sample selection cutoffs (year ranges, filters)
- Winsorization thresholds
- Random seeds
- Model hyperparameters
- Variable lists (control sets, outcome variables)
- Fixed effect specifications

**Wrong:**
```python
# Python
df = df[df['year'] >= 2005]
model = smf.ols('roa ~ size + lev', data=df)
```

```r
# R
df <- df %>% filter(year >= 2005)
model <- lm(roa ~ size + lev, data = df)
```

```stata
* Stata
keep if year >= 2005
reg roa size lev
```

**Right — config.yaml:**
```yaml
sample:
  year_start: 2005
  year_end: 2020

model:
  outcome: roa
  controls:
    - size
    - lev
  fixed_effects:
    - firm
    - year

winsorize:
  variables: [roa, size, lev]
  lower: 0.01
  upper: 0.99

seed: 42
```

**Right — script:**
```python
# Python
cfg = load_config("config/config.yaml")
df = df[df['year'] >= cfg['sample']['year_start']]
```

```r
# R
cfg <- yaml::read_yaml("config/config.yaml")
df <- df %>% filter(year >= cfg$sample$year_start)
```

```stata
* Stata
import yaml using "config/config.yaml"  // or use a mata parser
local year_start = yaml_get("sample.year_start")
keep if year >= `year_start'
```

> If a parameter appears more than once in your scripts, it belongs in config.

---

## Part III — Dependency Graph

> This section is a **living template**. Fill it in for your project.
> Update it before every refactor session.

### 3.1 Script / Do-file DAG

```
00_master
├── 01_clean          [input: data/raw/]         [output: data/interim/clean.*]
├── 02_construct      [input: data/interim/]     [output: data/final/panel.*]
│   └── utils/winsorize
└── 03_analysis       [input: data/final/]       [output: output/tables/, output/figures/]
    ├── utils/reghdfe_wrapper  (Stata)
    ├── utils/feols_wrapper    (R / fixest)
    └── utils/fe_wrapper       (Python / linearmodels)
```

### 3.2 Data Lineage

```
data/raw/compustat.csv
    ↓ [01_clean: drop duplicates, label vars, handle missing]
data/interim/clean.dta / clean.rds / clean.parquet
    ↓ [02_construct: merge, generate ratios, winsorize]
data/final/panel.dta / panel.rds / panel.parquet
    ↓ [03_analysis: regressions, event studies]
output/tables/table_*.xlsx
output/figures/fig_*.pdf
```

### 3.3 Variable Lineage

```
roa
├── net_income          [source: Compustat item IB]
└── total_assets        [source: Compustat item AT]
    └── note: winsorized at [config: winsorize.lower / upper]

treatment_post
├── treatment           [source: hand-collected / external dataset]
└── post                [derived: year >= event_year]
    └── event_year      [source: config.yaml → events.year_start]
```

### 3.4 Macro / Parameter Lineage (Critical for Stata)

```
controls_baseline    [defined: config.yaml]  [used: 03_analysis.do lines ~45, ~112]
fe_spec              [defined: config.yaml]  [used: 03_analysis.do lines ~45, ~113]
winsor_p             [defined: config.yaml]  [used: 02_construct.do lines ~30–55]
```

### 3.5 Output Lineage

```
Table 1: Summary Statistics    ← 02_construct output
Table 2: Baseline Regression   ← 03_analysis (controls_baseline)
Table 3: Robustness            ← 03_analysis (controls_extended)
Figure 1: Event Study Plot     ← 03_analysis (event_window config)
```

> **AI Rule:** Never rename a variable, change a control list, or modify a merge key
> without first tracing its full lineage in this graph.

---

## Part IV — Refactor Plan (Decision Audit)

> Every refactor task gets a record here.
> This is the audit trail. It must be updated as work proceeds.

### 4.1 Refactor Order (Mandatory Sequence)

Must proceed in this order. Do not skip ahead.

```
1. Formatting          (whitespace, indentation — zero risk)
2. Naming              (variables, functions — LOW risk with lineage check)
3. Comments/Docstrings (documentation — zero risk)
4. Config extraction   (hardcoded → config.yaml — LOW risk)
5. Deduplication       (extract repeated code — MEDIUM risk)
6. Modularization      (split large scripts — MEDIUM risk)
7. Function/Program abstraction  (MEDIUM–HIGH risk)
8. Workflow redesign   (pipeline restructure — HIGH risk)
```

### 4.2 Risk Classification

| Level | Definition | Required Actions |
|---|---|---|
| LOW | No statistical output affected | Run module, compare logs |
| MEDIUM | Data generation affected | Full output comparison |
| HIGH | Regression results potentially affected | Full regression test + advisor sign-off |
| CRITICAL | Sample selection or merge logic changed | **Stop. Document. Get explicit approval.** |

### 4.3 Decision Record Template

```markdown
## Refactor Task: [ID] — [Short Title]

**Date:** YYYY-MM-DD
**Status:** [ ] Planned  [ ] In Progress  [x] Complete  [ ] Rolled Back

**Objective:**
What this change achieves.

**Affected Files:**
- src/02_construct.do (lines 40–60)
- config/config.yaml (new keys added)

**Risk Level:** LOW / MEDIUM / HIGH / CRITICAL

**Rationale:**
Why this change is necessary. Link to Nelson / Zen principle if applicable.

**Validation Strategy:**
How we confirm outputs are unchanged.
- [ ] Re-run module
- [ ] Compare table_2_baseline.xlsx before/after (md5 or cf)
- [ ] Compare regression coefficients (N, coef, se, p, R²)

**Rollback:**
`git revert <commit_hash>`
or: restore from `data/backup/YYYYMMDD/`

**Result:**
What actually happened. Any deviations documented here.
```

---

## Part V — Core Rules

### Rule 1 — Never Silently Change Results

The following must be **identical** before and after any refactor,
unless a CRITICAL-level change has been explicitly approved and documented:

- Regression coefficients, standard errors, p-values, N, R²
- Sample size at every pipeline stage
- Merge match rates
- Winsorization cutoff values
- Fixed effect specifications
- Random seeds and any stochastic outputs
- Generated datasets (byte-level identical where possible)
- Output tables

> Any modification that affects the above is **HIGH RISK** by definition.

### Rule 2 — Dependency Graph Before Refactor

No refactoring begins without a completed dependency graph (Part III).
If the graph is outdated, update it first. That update is its own commit.

### Rule 3 — Config / Code Separation

See Part II Section 2.3. This rule has no exceptions.

### Rule 4 — Incremental Only

One logical change per commit. Never:
- Rename variables AND change logic in the same commit
- Restructure files AND modify parameters in the same commit
- Refactor AND fix a bug in the same commit

Each commit must be independently revertable.

### Rule 5 — Errors Must Not Pass Silently

```python
# Python — Wrong
try:
    df = pd.read_csv(path)
except:
    pass

# Python — Right
try:
    df = pd.read_csv(path)
except FileNotFoundError as e:
    logger.error(f"Data file not found: {path}")
    raise
```

```r
# R — Wrong
tryCatch(df <- read.csv(path), error = function(e) NULL)

# R — Right
tryCatch(
  df <- read.csv(path),
  error = function(e) {
    log_error(glue("Failed to read {path}: {e$message}"))
    stop(e)
  }
)
```

```stata
* Stata — Wrong
capture use "data/panel.dta"

* Stata — Right
capture use "data/panel.dta"
if _rc != 0 {
    display as error "Failed to load panel.dta — rc = `_rc'"
    exit `_rc'
}
```

---

## Part VI — Testing & Validation

### 6.1 Levels of Testing

| Type | What | When |
|---|---|---|
| Data assertions | Row counts, no missing keys, value ranges | After every merge/clean step |
| Output regression test | Coefficient tables identical before/after | After every refactor |
| Reproducibility test | Full pipeline from raw data | Before every commit to main |
| Environment test | Pipeline runs in clean environment | CI/CD on every push |

### 6.2 Data Assertions (by language)

```python
# Python (with pandas / great_expectations)
assert df['firm_id'].notna().all(), "Merge introduced missing firm IDs"
assert df.shape[0] == expected_n, f"Row count changed: got {df.shape[0]}, expected {expected_n}"
assert df['roa'].between(-1, 1).all(), "ROA out of expected range after winsorization"
```

```r
# R (with assertr / checkmate)
df %>%
  assert(not_na, firm_id) %>%
  assert(within_bounds(-1, 1), roa) %>%
  verify(nrow(.) == expected_n)
```

```stata
* Stata
assert !missing(firm_id)
assert roa >= -1 & roa <= 1
count
assert r(N) == `expected_n'
```

### 6.3 Regression Output Comparison

Before refactor: save a reference snapshot.

```python
# Python
results_before.to_csv("tests/reference/table2_before.csv")
# After refactor:
pd.testing.assert_frame_equal(results_after, results_before, rtol=1e-10)
```

```r
# R
saveRDS(results_before, "tests/reference/table2_before.rds")
# After refactor:
expect_equal(results_after$coefficients, results_before$coefficients, tolerance = 1e-10)
```

```stata
* Stata
cf old_results.dta new_results.dta
```

> Tolerance rule: **exact match preferred**. Float tolerance only with documented justification.

---

## Part VII — Documentation Standards

### 7.1 Module Docstring (every script)

```python
# Python
"""
Module: 03_analysis.py
Purpose: Run baseline and robustness regressions.
Inputs:  data/final/panel.parquet
Outputs: output/tables/table2_baseline.xlsx
         output/tables/table3_robustness.xlsx
Config:  config/config.yaml (model, sample, fixed_effects)
Dependencies: linearmodels, pandas, yaml
Author:  [Name]
Last Modified: YYYY-MM-DD
"""
```

```r
# R
#' Module: 03_analysis.R
#' Purpose: Run baseline and robustness regressions.
#' Inputs:  data/final/panel.rds
#' Outputs: output/tables/table2_baseline.xlsx
#' Config:  config/config.yaml
#' Dependencies: fixest, readr, yaml
#' Author: [Name]
#' Last Modified: YYYY-MM-DD
```

```stata
/******************************************************************
Module:       03_analysis.do
Purpose:      Run baseline and robustness regressions.
Inputs:       data/final/panel.dta
Outputs:      output/tables/table2_baseline.xlsx
Config:       config/config.yaml
Dependencies: reghdfe, esttab
Author:       [Name]
Last Modified: YYYY-MM-DD
******************************************************************/
```

### 7.2 Function / Program Docstring

```python
def winsorize_variable(series: pd.Series, lower: float, upper: float) -> pd.Series:
    """
    Winsorize a series at specified quantile bounds.

    Parameters
    ----------
    series : pd.Series
        Variable to winsorize.
    lower : float
        Lower quantile bound (e.g., 0.01).
    upper : float
        Upper quantile bound (e.g., 0.99).

    Returns
    -------
    pd.Series
        Winsorized series. Original index preserved.

    Side Effects
    ------------
    None. Pure function.
    """
```

---

## Part VIII — Naming Standards

### 8.1 Universal Rules

- All variable/column names: `snake_case`, lowercase
- Explicit and descriptive — no `x1`, `temp`, `aa`, `var`
- Boolean variables: prefix with `is_`, `has_`, `flag_`
- Date variables: suffix with `_year`, `_date`, `_quarter`
- Log-transformed variables: prefix with `log_`
- Winsorized variables: suffix with `_w` (and document in variable lineage)

### 8.2 Good Names vs Bad Names

| Bad | Good | Why |
|---|---|---|
| `x1` | `log_assets` | Explicit |
| `temp` | `firm_year_panel` | Descriptive |
| `d` | `treatment_indicator` | No abbreviations |
| `yr` | `fiscal_year` | No truncation |
| `roa2` | `roa_w` | Suffix explains transformation |

### 8.3 Script Naming

Scripts are numbered to reflect pipeline order:
```
00_master
01_clean
02_construct
03_analysis
04_tables
```
Utility helpers live in `src/utils/` without numeric prefix.

---

## Part IX — Logging & Git Discipline

### 9.1 Logging Rules

Every script execution produces a timestamped log.

```python
# Python
import logging
logging.basicConfig(
    filename=f"output/logs/03_analysis_{datetime.now():%Y%m%d_%H%M%S}.log",
    level=logging.INFO
)
```

```r
# R
log_file <- glue("output/logs/03_analysis_{format(Sys.time(), '%Y%m%d_%H%M%S')}.log")
```

```stata
log using "output/logs/03_analysis_$S_DATE.log", replace
```

Logs must never be silently overwritten without version tracking.

### 9.2 Git Commit Types (Conventional Commits)

```
refactor: replace hardcoded year cutoffs with config.yaml
fix:      correct merge key causing duplicate observations
feat:     add event study specification to 03_analysis
docs:     add module docstrings to 02_construct
test:     add data assertions for panel merge step
config:   extract variable lists to variables.yaml
```

One logical change per commit. Commits are the rollback unit.

### 9.3 Rollback Safety

Before any MEDIUM or HIGH risk change:
- Create a `data/backup/YYYYMMDD/` snapshot
- `git commit` current state with `[CHECKPOINT]` tag
- Document the checkpoint hash in refactor_plan (Part IV)

---

## Part X — Tool Use Protocol

> This section governs how AI assistants and researchers use the toolchain:
> `bash` · `Claude Code` · `VS Code` · `git` · `gh` (GitHub CLI)

### 10.1 The Atomic Commit Rule

> **One completed, verified change = one immediate git commit.**
> No exceptions.

This is the central discipline of this toolchain. It means:

- Complete a change → verify outputs → commit → only then move to the next task
- **Never batch** two logical changes into one commit
- **Never commit** before verification passes
- If verification fails → rollback immediately, do not patch forward

```
Change cycle (mandatory):

  PLAN → IMPLEMENT → VERIFY → COMMIT
                         ↓
                    (if fail)
                    ROLLBACK → re-PLAN
```

The commit is the checkpoint. Between commits, nothing is safe.

### 10.2 Tool Roles

| Tool | Role | When to Use |
|---|---|---|
| `bash` | Execute scripts, run tests, inspect data | Any terminal operation |
| `Claude Code` | AI-assisted editing in terminal | Refactoring, generation, explanation |
| `VS Code` | Editor + diff viewer | Code editing, reviewing changes visually |
| `git` | Version control — the rollback backbone | After every verified change |
| `gh` | GitHub CLI — CI status, PRs, issues | Checking CI, opening PRs, tracking issues |

### 10.3 Bash Rules

```bash
# Always run from project root
cd /your/project/root   # replace with your actual project path

# Always echo what you are about to do before doing it
echo "=== Running 03_analysis.do ==="
stata -b do src/03_analysis.do

# Never pipe into destructive operations without confirmation
# Wrong:
rm -rf data/interim/*

# Right:
echo "Files to delete:" && ls data/interim/ && read -p "Confirm? [y/N] " yn && [[ $yn == y ]] && rm -rf data/interim/*
```

Bash output must be logged. If the script does not log itself, capture it:

```bash
bash src/01_clean.sh 2>&1 | tee output/logs/01_clean_$(date +%Y%m%d_%H%M%S).log
```

### 10.4 Git Workflow — Step by Step

Every change follows this exact sequence:

```bash
# Step 1 — Check current state before touching anything
git status
git diff

# Step 2 — Make the change (one logical unit only)
# ... edit files ...

# Step 3 — Verify
# Run the affected module and compare outputs

# Step 4 — Stage and review what you are about to commit
git diff --staged        # Review every line
git status               # Confirm no unintended files

# Step 5 — Commit with a meaningful message
git commit -m "refactor: extract year cutoffs to config.yaml"

# Step 6 — Confirm the commit landed cleanly
git log --oneline -5
```

**Commit message format (Conventional Commits):**

```
<type>: <short imperative description>

Types:
  refactor  structural change, no output change
  fix       corrects a bug or wrong output
  feat      adds new functionality
  docs      documentation only
  test      adds or fixes tests / assertions
  config    config.yaml / variables.yaml changes only
  chore     tooling, CI, environment changes
```

### 10.5 Rollback Procedures

**Scenario A — Last commit was wrong, nothing else committed since:**

```bash
# Undo the last commit, keep changes staged
git reset --soft HEAD~1

# Or: undo the last commit and discard changes entirely
git reset --hard HEAD~1
```

**Scenario B — Need to revert a specific past commit (safe, keeps history):**

```bash
# Find the commit hash
git log --oneline

# Revert it (creates a new revert commit)
git revert <commit_hash>
```

**Scenario C — Working directory is broken, nothing committed yet:**

```bash
# Discard all uncommitted changes and return to last commit
git checkout -- .

# Or: nuclear reset to last commit
git reset --hard HEAD
```

**Scenario D — Need to roll back a data file that was overwritten:**

```bash
# Restore a specific file from a past commit
git checkout <commit_hash> -- data/interim/clean.dta
```

> Rule: always prefer `git revert` over `git reset` when commits have been pushed.
> `git revert` preserves history. `git reset` rewrites it.

### 10.6 Claude Code Rules

When using Claude Code in the terminal:

1. **State the task explicitly** before asking Claude Code to act
   - Wrong: "Fix this"
   - Right: "Refactor lines 40–55 of `02_construct.do` to load `year_start` from `config.yaml` instead of hardcoding 2005. Do not change any other logic."

2. **Review every suggested change** in VS Code diff view before accepting

3. **Claude Code must not commit autonomously** — the researcher reviews and commits manually

4. **After Claude Code edits**: run `git diff` to confirm the change is exactly what was intended before committing

5. **One task per Claude Code session** — do not chain multiple changes in one prompt

### 10.7 gh (GitHub CLI) — CI and Review

```bash
# Check CI status after pushing
gh run list --limit 5
gh run view <run_id>

# Open a PR from current branch
gh pr create --title "refactor: extract config parameters" --body "See progress.md R-007"

# Check if CI passed on the PR
gh pr checks

# View open issues (pending refactor tasks)
gh issue list --label "refactor"

# Create an issue for a blocked task
gh issue create --title "R-015: event study window spec" --label "blocked"
```

CI must pass before any branch is merged. A failed CI run = the commit is not clean.

### 10.8 VS Code Recommended Setup

Extensions for this workflow:

| Extension | Purpose |
|---|---|
| GitLens | Commit history inline, blame, rollback |
| Git Graph | Visual DAG of branch/commit history |
| YAML | Config file validation |
| Stata Enhanced / stataRun | Stata syntax + run from editor |
| R (Posit) | R language server |
| Python (Pylance) | Python language server |
| EditorConfig | Consistent formatting across languages |

Key shortcuts:
- `Ctrl+Shift+G` — Source Control panel (stage, commit, diff)
- `Ctrl+Shift+P → GitLens: Compare` — compare file across commits

---

## Part XI — AI Safety Rules

An AI assistant working on this codebase **must**:

1. **Never infer variable meaning** from name alone — check variable lineage (Part III)
2. **Never rename a variable** without completing a full lineage trace
3. **Never modify a regression specification** without explicit instruction
4. **Never assume merge keys are unique** — verify with assertions
5. **Never delete code** without documenting what it did and why it was removed
6. **Never proceed past an ambiguity** — STOP, state the ambiguity, ask
7. **Always state risk level** before proposing any change
8. **Always show before/after** for any proposed modification
9. **Config changes require explicit confirmation** — never silently update config.yaml
10. **If the dependency graph is missing: refuse to refactor**

> Uncertainty is not a reason to guess. It is a reason to stop.

---

## Part XII — Dangerous Patterns (Flag These)

| Pattern | Risk | Action |
|---|---|---|
| Hardcoded numbers in scripts | HIGH | Extract to config.yaml |
| `capture` / `tryCatch` without logging | HIGH | Add error logging |
| `save, replace` / overwrite without backup | HIGH | Backup first |
| Global macros (Stata) / global env vars | MEDIUM | Refactor to local/function-scoped |
| Nested loops > 2 levels | MEDIUM | Extract inner loop to function |
| Duplicate regression blocks | MEDIUM | Extract to parameterized function |
| Implicit sort assumptions before merge | HIGH | Add explicit `sort` + assertion |
| `set.seed` / `set seed` missing | CRITICAL | Add to config.yaml + script top |
| Magic strings (variable names in regression formula) | MEDIUM | Extract to config variables.yaml |
| Missing `assert` after merge | HIGH | Add match-rate assertion |

---

## Appendix — Three-Language Quick Reference

### Environment Management

| Task | R | Stata | Python |
|---|---|---|---|
| Lock environment | `renv::snapshot()` | `stata_ado.txt` (manual) | `pip freeze > requirements.txt` |
| Restore environment | `renv::restore()` | `ssc install` list | `pip install -r requirements.txt` |
| Config loading | `yaml::read_yaml()` | mata yaml parser / `import delimited` | `yaml.safe_load()` |

### Data Comparison

| Task | R | Stata | Python |
|---|---|---|---|
| Compare datasets | `all.equal(df1, df2)` | `cf old.dta new.dta` | `pd.testing.assert_frame_equal()` |
| Row count assert | `nrow(df) == n` | `assert r(N) == n` | `assert len(df) == n` |
| No missing assert | `assert(not_na, var)` | `assert !missing(var)` | `assert df['var'].notna().all()` |

### Regression Packages (with FE + Clustering)

| Task | R | Stata | Python |
|---|---|---|---|
| FE regression | `fixest::feols()` | `reghdfe` | `linearmodels.PanelOLS` |
| Cluster SE | `vcov = ~cluster_var` | `, cluster(var)` | `cov_type='clustered'` |
| Export table | `modelsummary()` | `esttab` | `stargazer` / `statsmodels` |

### Logging

| Task | R | Stata | Python |
|---|---|---|---|
| Start log | `sink(logfile)` | `log using file, replace` | `logging.basicConfig(filename=...)` |
| Timestamped name | `glue("{f}_{Sys.time()}")` | `log using "f_$S_DATE.log"` | `f"{f}_{datetime.now():%Y%m%d}"` |

---

*This SKILL.md is both the AI behavior specification and the project architecture document.*
*It must be read before any AI-assisted work on this codebase.*
*Last reviewed: YYYY-MM-DD*
