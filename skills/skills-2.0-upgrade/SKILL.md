---
name: skills-2-0-upgrade
description: Use when upgrading Claude Code skills to Skills 2.0 compliance, when diagnosing skill health scores, when frontmatter is missing or malformed, when skills exceed 500-line limit, or when skill directories lack SKILL.md structure
---

# Skills 2.0 Upgrade

## Overview
Skills 2.0 is Anthropic's 2026 skill framework for discoverable, token-efficient, composable skills.

Its core pattern is Progressive Disclosure across three tiers:
1. Frontmatter for fast routing
2. `SKILL.md` for the working guide
3. `references/` for heavy detail and reference-only content

This matters because modern skill ecosystems are large. Public catalogs such as agentskills.io expose 351K+ skills, so discovery quality depends on clean frontmatter, searchable trigger wording, and compact bodies.

Upgrading older skills improves:
- compatibility with current skill loaders
- Claude Search Optimization (CSO)
- token efficiency in always-loaded contexts
- maintainability for long or reference-heavy skills
- cross-tool portability across Claude Code and adjacent CLIs

Target state:
- one skill per directory
- one `SKILL.md` entrypoint
- short, searchable body
- heavy detail moved into `references/`
- reference files marked with invocation control when appropriate

## When to Use
Use this skill when any of these symptoms appear:
- `missing frontmatter` — the skill has no YAML frontmatter
- `malformed frontmatter` — fields are missing, extra keys exist, or YAML is invalid
- `skill too long` — body exceeds the 500-line guideline
- `orphan directory` — a skill folder exists without `SKILL.md`
- `broken reference` — `references/` links or targets are invalid
- description does not start with `Use when...`
- description explains workflow instead of triggers
- compliance score is low or trending down
- reference files should use `disable-model-invocation`

Do not use this skill for normal feature implementation inside a skill. Use it for diagnosis, restructuring, and compliance upgrades.

## Quick Reference
| Criterion | Description | Weight |
|-----------|-------------|--------|
| Frontmatter | `name` + `description` YAML block | 20% |
| Body Length | ≤500 lines (excluding frontmatter) | 15% |
| Directory Structure | `SKILL.md` + `references/` for long skills | 10% |
| Description Pattern | Starts with `Use when...` | 5% |
| Model Control | `disable-model-invocation` for references | 8% |

Practical interpretation:
- fix frontmatter first because it drives routing
- shorten oversized skills before polishing prose
- move heavy reference content out of `SKILL.md`
- keep descriptions trigger-focused, not workflow-focused
- add invocation control only to reference files, not the main entrypoint

## Diagnosis
Run the repo diagnostic first:

```bash
scripts/diagnose.sh .claude/skills/
```

Modes:
- `--markdown` (default) — human-readable report
- `--json` — machine-readable output for automation
- `--verbose` — expanded issue details

Examples:

```bash
scripts/diagnose.sh .claude/skills/ --markdown
scripts/diagnose.sh .claude/skills/ --json
scripts/diagnose.sh .claude/skills/ --verbose
```

Typical summary output:

```text
Overall compliance: 78.2% (94 skills scanned)
```

Minimal JSON shape:

```json
{
  "overallCompliance": 78.2,
  "skillsScanned": 94,
  "issues": [{"priority": "P1", "path": "...", "type": "missing_frontmatter"}]
}
```

Use diagnosis before any edits. Upgrade order should follow the reported priorities rather than ad-hoc cleanup.

## Upgrade Actions
| Priority | Action | Automation |
|----------|--------|------------|
| P1 | Add frontmatter | Automatic |
| P2 | Normalize name field | Automatic |
| P3 | Generate description | Semi-auto |
| P4 | Add invocation control | Semi-auto |
| P5 | Split long files | Manual (Opus) |
| P6 | Fix directory structure | Semi-auto |
| P7 | Handle orphan dirs | Semi-auto |

Execution order:
1. repair routing metadata (`P1`, `P2`)
2. improve discoverability (`P3`)
3. control reference loading (`P4`)
4. split oversized files (`P5`)
5. normalize folder layout (`P6`, `P7`)

See `references/upgrade-actions.md` for detailed procedures.

## Model Guide
| Task | Recommended Model | Effort | Why |
|------|-------------------|--------|-----|
| Diagnosis only | Any model (bash) | N/A | No LLM needed, pure bash |
| P1-P2 auto-fix | Sonnet | medium | Simple pattern matching |
| P3 file splitting | Opus | high | Content understanding needed |
| Full upgrade | Opus | high | Includes splitting |
| Codex CLI | `codex exec` | low | JSON output mode |

Rule of thumb:
- use bash for scoring and inventory
- use Sonnet for deterministic metadata fixes
- use Opus when restructuring long content or extracting references

## Cross-Platform
| Platform | Support Level | Notes |
|----------|--------------|-------|
| Claude Code | Full | All features (interactive + auto-fix) |
| Codex CLI | Partial | `codex exec "diagnose.sh .claude/skills/ --json"` |
| Antigravity | Partial | Skill file read + bash execution |
| Cursor/Windsurf | Minimal | `.md` file read only, manual guide |

Keep the main guidance portable, but assume best automation quality in Claude Code.

## Common Mistakes
- Putting workflow summary in `description` instead of triggers only
- Using `you` or `your` in `description` instead of third person
- Editing without diagnosis first
- Skipping backup before upgrade
- Skipping `P3` description work even though CSO impact is high
- Leaving oversized content inline instead of moving it to `references/`
- Forgetting invocation control on reference-only files
- Fixing one file while leaving orphan directories unresolved
