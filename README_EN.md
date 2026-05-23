# Research Code Refactoring Skill

**[English](README_EN.md)** | **[中文](README_CN.md)**

A structured code refactoring framework for academic research projects (R / Stata / Python). Designed for **Claude Code**.

## Features

- Enforced **refactor order**: formatting -> naming -> docs -> config -> dedup -> modularize -> abstract -> redesign
- **Dependency graph** tracing -- no variable renamed without full lineage check
- **Atomic commits**: one verified change = one commit = one rollback point
- Risk-level tracking (LOW / MEDIUM / HIGH / CRITICAL) in `progress.md`
- R, Stata, Python examples throughout

## Risk Disclaimer

> **Use at your own risk.** The author is NOT responsible for code loss, incorrect results, or any damage.
>
> **You MUST:**
> 1. Back up your project before starting
> 2. Work on a **separate branch** (`git checkout -b refactor/xxx`) -- never refactor on `main`
> 3. Review every AI-suggested change before accepting
> 4. Verify outputs (regression results, data shapes, tables) after each change

## Installation

```bash
git clone https://github.com/qijialiv-cell/research-refactor-skill.git

# Global install (all projects)
mkdir -p ~/.claude/skills/research-refactor
cp research-refactor-skill/SKILL.md research-refactor-skill/progress.md ~/.claude/skills/research-refactor/

# Or per-project install (from your project root)
mkdir -p .claude/skills/research-refactor
cp ~/research-refactor-skill/SKILL.md ~/research-refactor-skill/progress.md .claude/skills/research-refactor/
```

## Usage

```bash
cd /your/project && claude
/research-refactor Extract hardcoded year cutoffs to config.yaml
```

Or in **VS Code**: open Claude panel -> type `/research-refactor <your task>`.

## Key Rules

| Rule | Description |
|---|---|
| Never silently change results | Coefficients, sample sizes, outputs stay identical |
| Config/code separation | All parameters in `config.yaml`, never hardcoded |
| Incremental only | One logical change per commit |
| Dependency graph first | No refactoring without a completed graph |
| Errors must not pass silently | No bare `tryCatch` / `capture` without logging |
| AI must ask before acting | Ambiguity = stop and ask |

## License

MIT
