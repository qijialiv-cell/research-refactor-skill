# Research Code Refactoring Skill

**[English](README_EN.md)** | **[中文](README_CN.md)**

---

A structured code refactoring framework for academic research projects (R / Stata / Python). Designed to be used with **Claude Code** as an AI-assisted refactoring skill that enforces safe, incremental, and auditable code changes.

## What It Does

- Enforces a **refactor order** (formatting -> naming -> docs -> config -> dedup -> modularize -> abstract -> redesign)
- Maintains a **dependency graph** so no variable or merge key is renamed without full lineage tracing
- Requires **atomic commits**: one verified change = one commit = one rollback point
- Tracks progress in `progress.md` with risk levels (LOW / MEDIUM / HIGH / CRITICAL)
- Supports R, Stata, and Python with language-specific examples throughout

## Risk Disclaimer

> **IMPORTANT: Use at your own risk.**
>
> This skill is provided "as is" without any warranty. The author is **NOT responsible** for:
> - Code loss, file corruption, or unintended modifications
> - Incorrect analysis results caused by AI-assisted refactoring
> - Data corruption or loss during the refactoring process
> - Any consequences arising from the use of this tool
>
> **Before using this skill, you MUST:**
> 1. **Back up your project** (full directory copy or `git stash` / commit all pending work)
> 2. **Work on a separate branch** -- never refactor on `main` or any branch without a backup
> 3. **Review every AI-suggested change** before accepting -- do not blindly trust AI output
> 4. **Verify outputs after every change** -- compare regression results, data shapes, and tables
>
> AI-assisted refactoring can introduce subtle bugs that are hard to detect. Always maintain
> independent verification and the ability to roll back.

## Best Practices

1. **Always create an independent branch before refactoring:**
   ```bash
   git checkout -b refactor/extract-config
   ```
   Never refactor directly on `main` or any unprotected branch.

2. **Commit early and often** -- each verified change gets its own commit, so you can always `git revert` a single bad change without losing the rest.

3. **Take output snapshots before starting** -- save reference tables/data so you can compare after refactoring.

4. **One change at a time** -- the skill enforces this, but it is worth repeating: never batch multiple logical changes.

---

## Installation

### Option 1: Install as a Claude Code Skill (Recommended)

This makes the skill available via a slash command (e.g., `/research-refactor`) across all your projects.

#### Via Terminal (macOS / Linux)

```bash
# 1. Clone this repo
git clone https://github.com/qijialiv-cell/research-refactor-skill.git

# 2. Copy the skill directory to your personal Claude Code skills folder
mkdir -p ~/.claude/skills/research-refactor
cp research-refactor-skill/SKILL.md ~/.claude/skills/research-refactor/SKILL.md
cp research-refactor-skill/progress.md ~/.claude/skills/research-refactor/progress.md
```

After installation, the skill is immediately available in Claude Code as `/research-refactor`.

#### Per-Project Installation

If you want the skill scoped to a single project (and committed to version control):

```bash
# From your project root
mkdir -p .claude/skills/research-refactor
cp /path/to/research-refactor-skill/SKILL.md .claude/skills/research-refactor/SKILL.md
cp /path/to/research-refactor-skill/progress.md .claude/skills/research-refactor/progress.md

# Commit the skill to your project repo
git add .claude/skills/research-refactor/
git commit -m "chore: add research refactoring skill"
```

### Option 2: Install as a Claude Code Plugin

For distribution and team sharing via a plugin marketplace:

```bash
# 1. Add the marketplace (if published)
# In Claude Code:
/plugin marketplace add YOUR_USERNAME/research-refactor-skill

# 2. Install the plugin
/plugin install research-refactor@marketplace-name
```

### Option 3: Use in VS Code Claude Extension

1. Install the Claude extension in VS Code
2. Type `/` in the Claude prompt box to see available skills
3. If the skill is installed via Option 1 or 2, it will appear in the list
4. Alternatively, manage plugins via `/plugins` in the VS Code Claude panel

---

## Usage

### In Claude Code (Terminal)

```bash
# Start Claude Code in your project directory
cd /path/to/your/research/project
claude

# Invoke the skill
/research-refactor Extract hardcoded year cutoffs to config.yaml
```

The skill will:
1. Read `SKILL.md` and `progress.md` to understand the project state
2. Analyze the dependency graph
3. Propose changes following the mandatory refactor order
4. Wait for your review before making any modifications

### In VS Code

1. Open your project in VS Code
2. Open the Claude panel (sidebar or `Cmd+Shift+P` -> "Claude: Open")
3. Type `/research-refactor` followed by your task description
4. Review the suggested changes in the diff view before accepting

### Quick Start Checklist

```bash
# 1. Back up your work
git stash  # or: cp -r /project /project_backup

# 2. Create a refactor branch
git checkout -b refactor/initial-setup

# 3. Set up the skill files in your project
mkdir -p docs
cp /path/to/SKILL.md docs/skill.md
cp /path/to/progress.md docs/progress.md

# 4. Customize for your project
# Edit docs/progress.md to reflect your actual project structure

# 5. Start Claude Code and invoke the skill
claude
# Then type: /research-refactor Review my project structure
```

---

## File Structure

```
research-refactor-skill/
├── README.md           # Language selector
├── README_EN.md        # English documentation
├── README_CN.md        # Chinese documentation
├── SKILL.md            # Main skill instructions (with frontmatter)
└── progress.md         # Progress tracker template
```

In your research project, the recommended structure becomes:

```
your-project/
├── config/
│   ├── config.yaml
│   └── variables.yaml
├── data/
│   ├── raw/
│   ├── interim/
│   └── final/
├── src/
│   ├── 00_master.R
│   ├── 01_clean.R
│   ├── 02_construct.R
│   └── 03_analysis.R
├── output/
│   ├── tables/
│   ├── figures/
│   └── logs/
├── tests/
├── docs/
│   ├── skill.md        # Refactoring rules (this skill)
│   └── progress.md     # Refactoring progress tracker
└── .claude/
    └── skills/
        └── research-refactor/
            └── SKILL.md
```

---

## Key Rules Enforced by This Skill

| Rule | Description |
|---|---|
| Never silently change results | Regression coefficients, sample sizes, and outputs must remain identical |
| Config/code separation | All parameters go in `config.yaml`, never hardcoded in scripts |
| Incremental only | One logical change per commit, each independently revertable |
| Dependency graph first | No refactoring without a completed dependency graph |
| Errors must not pass silently | No bare `tryCatch`, no `capture` without logging |
| AI must ask before acting | Ambiguity = stop and ask, never guess |

---

## License

MIT License. See [LICENSE](LICENSE) for details.

## Contributing

Contributions welcome. Please open an issue or pull request on GitHub.
