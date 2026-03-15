# Skills 2.0 Auto-Upgrade

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)]()

> Zero-token diagnostic engine + Claude Code skill for automatically upgrading skills to Skills 2.0 compliance.

## What is Skills 2.0?

Skills 2.0 is Anthropic's 2026 skill framework for making reusable agent instructions easier to discover, load, and maintain.
It formalizes **Progressive Disclosure** as a 3-tier structure: frontmatter for triggering, `SKILL.md` for the working guide, and `references/` for heavy material loaded on demand.
The ecosystem has grown to **351K+ published skills**, which makes search quality, token efficiency, and structural consistency much more important than in early single-file skill setups.
This project helps migrate existing Claude Code skills to that structure with a zero-token bash diagnosis pass and guided upgrade workflow.

## Quick Start

### Install (primary)

```bash
curl -fsSL https://raw.githubusercontent.com/treylom/skills-2.0-upgrade/main/install.sh | bash
```

### Install from local clone (alternative)

```bash
bash install.sh
```

### Basic usage

```bash
/skills-upgrade
/skills-upgrade --dry-run
/skills-upgrade --upgrade
```

## Features

- **Zero-token diagnosis**: Pure bash, no LLM calls needed
- **12-item weighted checklist**: Comprehensive compliance scoring
- **Auto-fix P1-P4**: Automatic frontmatter, name, description fixes
- **Backup before upgrade**: Automatic tar.gz backup
- **Cross-platform**: WSL, macOS, Linux
- **JSON output**: Machine-parseable for CI/CD integration
- **Slash command**: `/skills-upgrade` with 3 modes

## Usage

### Diagnose (default)

```bash
/skills-upgrade
```

Example output:

```text
Skills 2.0 Compliance Report
=============================
Date: 2026-03-15 14:30:22
Path: /home/user/.claude/skills/
Skills scanned: 94
Overall compliance: 78.2%

Top Issues:
1. Missing frontmatter: 12 skills
2. Over 500 lines: 8 skills
3. Missing description: 15 skills
```

### Dry Run

```bash
/skills-upgrade --dry-run
```

Example output:

```text
Planned changes
- P1 Add frontmatter: 12 files
- P2 Normalize name field: 7 files
- P3 Generate description: 15 files
- P4 Add invocation control: 4 files
No files were modified.
```

### Upgrade

```bash
/skills-upgrade --upgrade
```

Example output:

```text
Backup created: ~/.claude/skills-backup-20260315-143022.tar.gz
Re-diagnosing after fixes...
Compliance improved: 62.0% -> 78.2%
```

### Direct Script Usage

```bash
~/.claude/scripts/diagnose.sh ~/.claude/skills/
~/.claude/scripts/diagnose.sh ~/.claude/skills/ --json
~/.claude/scripts/diagnose.sh ~/.claude/skills/ --verbose
~/.claude/scripts/diagnose.sh ~/.claude/skills/my-skill.md
```

## 12-Item Diagnostic Checklist

| # | Check | Weight | Pass Condition |
|---|-------|--------|---------------|
| 1 | Frontmatter exists | 20% | First line is `---` and a closing `---` exists |
| 2 | `name:` field | 8% | Frontmatter contains `name:` and the value uses only hyphens and alphanumeric characters |
| 3 | `description:` field | 10% | Frontmatter contains a non-empty `description:` value |
| 4 | Description `Use when...` pattern | 5% | Description starts with `Use when` (case-insensitive) |
| 5 | Description third-person | 3% | Description does not contain `you` or `your` |
| 6 | Body ≤500 lines | 15% | Body after frontmatter is 500 lines or fewer |
| 7 | Directory structure | 10% | Long skills use `SKILL.md` + `references/`; short skills pass automatically |
| 8 | `disable-model-invocation` | 8% | Reference-type skills include `disable-model-invocation: true`; others pass automatically |
| 9 | No orphan directories | 5% | Skill directories contain `SKILL.md` or at least one `.md` file |
| 10 | No broken references | 5% | `references/` links resolve to existing files |
| 11 | Progressive Disclosure | 5% | Skills with `references/` explicitly point readers to `references/` in the body |
| 12 | Imperative form | 6% | Imperative instructions appear more often than second-person advisory phrasing |

## Model Guidance

| Task | Model | Effort | Why |
|------|-------|--------|-----|
| Diagnosis only | Any (bash) | N/A | No LLM needed |
| P1-P2 auto-fix | Sonnet | medium | Simple patterns |
| P3 file splitting | Opus | high | Content understanding |
| Full upgrade | Opus | high | Includes splitting |

## Cross-Platform Compatibility

| Platform | Support | Notes |
|----------|---------|-------|
| Claude Code | Full | All features |
| Codex CLI | Partial | JSON output mode |
| Antigravity | Partial | Skill read + bash |
| Cursor/Windsurf | Minimal | .md read only |

## How It Works

4-Phase pipeline:
1. **Scan**: Parse arguments, run diagnose.sh
2. **Report**: Generate compliance summary
3. **Plan**: Show proposed changes (dry-run/upgrade)
4. **Execute**: Backup → auto-fix → re-diagnose (upgrade only)

## Case Study

Real-world migration of 94 skills:
- Starting compliance: 62%
- After Phase 1 fixes: 78%
- Top issues: missing frontmatter (12), >500 lines (8), missing description (15)

## Project Structure

```text
skills-2.0-upgrade/
├── commands/
│   └── skills-upgrade.md
├── scripts/
│   ├── backup.sh
│   └── diagnose.sh
├── skills/
│   └── skills-2.0-upgrade/
│       ├── SKILL.md
│       └── references/
│           ├── diagnostic-criteria.md
│           ├── skills-2.0-spec.md
│           ├── trigger-optimization.md
│           └── upgrade-actions.md
├── CHANGELOG.md
├── install.sh
├── LICENSE
├── README-ko.md
└── README.md
```

## Acknowledgments

This project builds on the work of:

- **[superpowers:writing-skills](https://github.com/anthropics/superpowers)** (Anthropic) — TDD-based skill authoring methodology, Claude Search Optimization (CSO), and official SKILL.md frontmatter specification.
- **[fivetaku/skillers-suda](https://github.com/fivetaku/skillers-suda)** (MIT) — 9-item structural validation (`validate-skill.sh`), Auto Eval methodology (`with_skill` vs `without_skill` A/B comparison), and description trigger optimization strategy.

Our 12-item diagnostic checklist merges 8 items from Skills 2.0 with 4 additional items from skillers-suda (third-person description, progressive disclosure, imperative form, and enhanced trigger validation).

## License

MIT — see [LICENSE](LICENSE) for details.
