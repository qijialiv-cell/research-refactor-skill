# Project Progress Tracker
### Last Updated: YYYY-MM-DD
### Project: [Dissertation Title / Chapter]

---

## Status Legend

| Symbol | Meaning |
|---|---|
| ✅ | Complete — tested, committed, verified |
| 🔄 | In Progress — actively being worked on |
| ⏳ | Planned — scheduled, not started |
| ⛔ | Blocked — waiting on data, decision, or approval |
| 🔁 | Rolled Back — attempted, reverted, reason documented |
| ⚠️ | Needs Review — complete but output verification pending |

---

## Current Sprint

> What is being worked on **right now**.
> Maximum one item at a time (Rule 4: incremental only).

| ID | Task | File(s) | Risk | Status | Started |
|---|---|---|---|---|---|
| R-007 | Extract year cutoffs to config.yaml | `src/01_clean.do`, `config/config.yaml` | LOW | 🔄 | 2024-XX-XX |

---

## ✅ Completed

> Done, verified, committed. Do not re-open without a new task ID.

| ID | Task | File(s) | Risk | Commit | CI | Verified |
|---|---|---|---|---|---|---|
| R-001 | Add module docstrings to all scripts | `src/*.do` | LOW | `abc1234` | ✅ | ✅ |
| R-002 | Replace hardcoded seed with config.yaml | `src/03_analysis.do` | LOW | `def5678` | ✅ | ✅ |
| R-003 | Add merge assertion after Compustat join | `src/01_clean.do` | LOW | `ghi9012` | ✅ | ✅ |
| R-004 | Rename `x1` → `log_assets` across pipeline | `src/02_construct.do` | MEDIUM | `jkl3456` | ✅ | ✅ |
| R-005 | Extract control variable list to variables.yaml | `config/variables.yaml`, `src/03_analysis.do` | MEDIUM | `mno7890` | ✅ | ✅ |
| R-006 | Add timestamped logging to all modules | `src/*.do` | LOW | `pqr1234` | ✅ | ✅ |

---

## 🔄 In Progress

> Detailed record for active task(s).

### R-007 — Extract year cutoffs to config.yaml

**Objective:**
Remove hardcoded `year >= 2005` and `year <= 2020` from `01_clean.do`.
Move to `config/config.yaml` under `sample.year_start` / `sample.year_end`.

**Affected Files:**
- `src/01_clean.do` (lines 34, 67, 102)
- `config/config.yaml` (new keys)

**Risk Level:** LOW

**Rationale:**
Zen: *Explicit is better than implicit.*
Nelson Ch13: Config/code separation.
Currently this cutoff appears 3× across the script — a config change would require editing 3 lines.

**Validation Checklist:**
- [ ] `data/final/panel.dta` row count unchanged
- [ ] `output/tables/table2_baseline.xlsx` coefficients identical
- [ ] `tests/reference/` snapshot comparison passes

**Rollback:**
`git revert HEAD` or restore from `data/backup/20240101/`

**Checkpoint Commit:** `[pending]`

---

## ⏳ Planned

> Scheduled tasks in refactor order. See `skill.md Part IV` for sequence rules.

| ID | Task | Depends On | Risk | Priority |
|---|---|---|---|---|
| R-008 | Extract FE spec to variables.yaml | R-007 | MEDIUM | High |
| R-009 | Deduplicate regression loops in `03_analysis` | R-008 | MEDIUM | High |
| R-010 | Wrap winsorization in a reusable function/program | R-004 | MEDIUM | Medium |
| R-011 | Add data assertion: no duplicates after panel construction | — | LOW | High |
| R-012 | Add GitHub Actions CI for full pipeline run | R-007 | LOW | Medium |
| R-013 | Modularize `01_clean` into `01a_clean_compustat` + `01b_clean_crsp` | R-011 | HIGH | Low |
| R-014 | Migrate from global macros to local macros (Stata) / function args (R/Python) | R-009 | HIGH | Low |

---

## ⛔ Blocked

| ID | Task | Blocked By | Since | Notes |
|---|---|---|---|---|
| R-015 | Add event study robustness (alternative windows) | Waiting for advisor approval on window spec | 2024-XX-XX | Window = ±3yr vs ±5yr TBD |

---

## 🔁 Rolled Back

> Attempted, failed or reverted. Document what happened so it is not tried again blindly.

| ID | Task | Attempted | Revert Commit | Why Rolled Back |
|---|---|---|---|---|
| R-004b | Rename `roa2` → `roa_lag1` | 2024-XX-XX | `stu5678` | Broke merge key in `03_analysis`; lineage trace was incomplete. Redo after dependency graph update. |

---

## ⚠️ Needs Review

> Technically complete but awaiting output verification or advisor review.

| ID | Task | Complete | Pending |
|---|---|---|---|
| R-005 | Extract control variable list to variables.yaml | Code done | Need to verify Table 3 coefficients match pre-refactor snapshot |

---

## Dependency Map (Summary)

> Which tasks block which. Keep this updated.

```
R-007 (config extraction)
  └── R-008 (FE spec to yaml)
        └── R-009 (deduplicate regression loops)
              └── R-014 (global → local macros)

R-004 (rename roa)
  └── R-010 (winsorization function)

R-011 (panel assertion)
  └── R-013 (modularize 01_clean)

R-012 (CI) — depends on R-007 completing
```

---

## Output Verification Log

> Track which output snapshots have been taken and verified.

| Output File | Snapshot Date | Snapshot Hash / Location | Post-Refactor Match |
|---|---|---|---|
| `output/tables/table2_baseline.xlsx` | 2024-XX-XX | `tests/reference/table2_pre_r007.xlsx` | ⏳ pending R-007 |
| `data/final/panel.dta` | 2024-XX-XX | `data/backup/20240101/panel.dta` | ✅ R-006 verified |

---

## Environment Snapshot

| Language | Lock File | Last Updated | Notes |
|---|---|---|---|
| Stata | `config/environments/stata_ado.txt` | YYYY-MM-DD | reghdfe 6.x, esttab 2.x |
| R | `config/environments/renv.lock` | YYYY-MM-DD | fixest 0.11, tidyverse 2.x |
| Python | `config/environments/requirements.txt` | YYYY-MM-DD | linearmodels 5.x, pandas 2.x |

---

## Tool Use Quick Reference

> For full rules see `skill.md Part X`.

**The one rule that matters most:**
```
Complete change → Verify → git commit    (one per change, immediately)
Verification fails → git reset --hard HEAD~1 or git revert <hash>
```

**Common git commands during a session:**

```bash
git status                          # always start here
git diff                            # review before staging
git diff --staged                   # review before committing
git log --oneline -10               # see recent history
git commit -m "refactor: ..."       # commit after verification
git reset --hard HEAD~1             # rollback last commit (local only)
git revert <hash>                   # rollback a pushed commit safely
gh run list --limit 5               # check CI status
gh pr checks                        # check PR CI
```

---

*This file is updated before and after every work session.*
*It is read by AI assistants as part of Rule Zero (see skill.md).*
