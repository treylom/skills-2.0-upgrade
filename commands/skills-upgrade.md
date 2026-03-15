---
description: "Skills 2.0 auto-diagnosis and upgrade. /skills-upgrade [--diagnose|--dry-run|--upgrade] [--path <dir>]"
allowedTools: Bash, Read, Write, Edit, Glob, Grep, AskUserQuestion
---

# /skills-upgrade

Diagnose and upgrade a skill directory to Skills 2.0 compliance through a guided, phase-by-phase workflow.

## Usage

```bash
/skills-upgrade                          # Diagnose (default)
/skills-upgrade --diagnose               # Same as above
/skills-upgrade --dry-run                # Preview changes without applying
/skills-upgrade --upgrade                # Interactive guided upgrade
/skills-upgrade --path /custom/skills/   # Custom target path
```

Default path: `$HOME/.claude/skills/`

## Execution Flow

### Phase 1 — Scan

1. Parse `$ARGUMENTS` for mode (`--diagnose`, `--dry-run`, `--upgrade`) and `--path <dir>`.
2. Resolve target directory. Default: `$HOME/.claude/skills/`.
3. Run diagnostics:

```bash
scripts/diagnose.sh "$TARGET_PATH" --json
```

4. Parse JSON into a working structure: overall compliance, skill count, issue counts by P1-P7.
5. If diagnosis fails, stop and show the raw output.

### Phase 2 — Report

Present a clear, friendly report with context about what each issue means and how impactful fixing it would be.

#### Compliance Summary

```markdown
## Your Skills 2.0 Compliance Report

| Metric | Value |
|--------|-------|
| Skills scanned | N |
| Overall compliance | X% |

### Issue Breakdown

| Priority | Count | What it means | Impact if fixed |
|----------|-------|---------------|-----------------|
| P1 (frontmatter) | N | Skills without YAML frontmatter can't be discovered by skill loaders | High — frontmatter drives routing |
| P2 (name) | N | Invalid or missing name field breaks skill identification | Medium — needed for stable references |
| P3 (description) | N | Description doesn't start with "Use when..." reducing discoverability | High aggregate — affects CSO across many skills |
| P4 (invocation) | N | Reference files load unnecessary model context | Low per skill — saves tokens |
| P5 (body length) | N | Body exceeds 500 lines, hurting token efficiency | Medium — needs manual Opus splitting |
| P6-P7 (structure) | N | Directory structure or orphan issues | Low — cleanup work |
```

#### Upgrade Path Recommendation

Based on the diagnosis, present the recommended upgrade path:

```markdown
### Recommended Upgrade Path

Based on your current score of X%, here's what each phase would improve:

| Phase | What it fixes | Expected improvement | Automation level |
|-------|--------------|---------------------|-----------------|
| P1-P2 | Frontmatter + name normalization | +10-16%p → ~Y% | Fully automatic (no confirmation needed) |
| P3 | Description "Use when..." pattern | +8-16%p → ~Z% | Semi-automatic (batch with preview) |
| P4 | Invocation control on references | +1-3%p → ~W% | Semi-automatic (requires confirmation) |
| P5+ | File splitting + structure fixes | +2-6%p → ~100% | Manual (Opus model recommended) |

The biggest bang-for-buck is P1-P3: typically takes compliance from ~62% to ~94% with minimal effort.
```

If mode is `--diagnose`, stop here.

### Phase 3 — Plan (--dry-run and --upgrade)

Show the proposed change plan with concrete file counts:

```markdown
## Planned Changes

- **P1**: N files will receive YAML frontmatter
- **P2**: N files will have their `name` field normalized to kebab-case
- **P3**: N descriptions will be transformed to "Use when..." pattern
- **P4**: N reference-type files will get `disable-model-invocation: true`

### P5+ (requires manual follow-up)
- N files exceed 500 lines — splitting needs content understanding (Opus recommended)
- N files have broken references — target files need to be created or links removed
- N files use advisory phrasing instead of imperative commands

Total files affected: N
Backup will be created before any changes.
```

If mode is `--dry-run`, stop here.

### Phase 4 — Execute (--upgrade only)

Guide the user through each phase interactively, asking permission at each step.

#### Step 4.0 — Ask scope

```
AskUserQuestion:
  "How far would you like to upgrade?"
  Options:
    - "P1-P2 only (automatic, safe)" — frontmatter + name fixes
    - "P1-P3 (recommended)" — includes description optimization
    - "P1-P4 (thorough)" — includes invocation control
    - "P1-P4 + P5 guidance" — full upgrade with manual splitting guide
```

#### Step 4.1 — Backup

```bash
scripts/backup.sh "$TARGET_PATH"
```

If backup fails, stop. Show the backup path and size.

#### Step 4.2 — P1-P2 (Automatic)

Run batch fix for P1 and P2:

```bash
python3 scripts/batch-p1p2p3.py "$TARGET_PATH" --dry-run  # preview first
```

Show a summary of what will change, then apply:

```bash
python3 scripts/batch-p1p2p3.py "$TARGET_PATH"
```

Report: "P1-P2 complete: N frontmatter added, M names normalized."

#### Step 4.3 — P3 (Semi-automatic)

The batch script also handles P3. Show a sample of 5 description transformations:

```markdown
### Sample Description Changes (5 of N)

| Skill | Before | After |
|-------|--------|-------|
| my-skill | "Analyzes code quality" | "Use when analyzing code quality" |
| ... | ... | ... |

N total descriptions will be updated. Proceed?
```

Wait for user confirmation before applying. If user says no, skip P3.

Report: "P3 complete: N descriptions updated to 'Use when...' pattern."

#### Step 4.4 — P4 (Confirmation required)

Identify reference-type skills using heuristics:
- name contains: guide, reference, spec, examples, templates, schema, collection, strategies, checklist, encyclopedia, glossary
- file lives under `references/` directory

Present candidates:

```markdown
### Invocation Control Candidates

These files appear to be reference material and should have `disable-model-invocation: true`:

| # | File | Reason |
|---|------|--------|
| 1 | references/api-guide.md | Name contains "guide" |
| 2 | references/schema.md | Name contains "schema" |

Apply invocation control to these files?
```

AskUserQuestion: approve all / selective / skip.

#### Step 4.5 — Re-diagnose

```bash
scripts/diagnose.sh "$TARGET_PATH" --json
```

#### Step 4.6 — Before/After Report

```markdown
## Upgrade Complete!

| Metric | Before | After | Change |
|--------|--------|-------|--------|
| Overall compliance | X% | Y% | +Z%p |
| P1 issues | N | 0 | -N |
| P2 issues | N | 0 | -N |
| P3 issues | N | M | -K |
| P4 issues | N | M | -K |
| P5+ issues | N | N | (manual) |

Backup: ~/.claude/.claude/skills-backup-TIMESTAMP.tar.gz
```

#### Step 4.7 — Beyond P4 Guidance

After the automated phases, explain what remains and how to address it:

```markdown
### What's Left? A Guide to 100% Compliance

Your skills are now at Y% — great progress! Here's what would get you to 100%:

**1. Body >500 lines (P5) — N skills**
These skills are too long for efficient token loading. Each needs to be split into
a compact SKILL.md (core guide) + references/ (detailed material).

How to proceed:
- Use Opus model with high effort for content understanding
- Process one skill at a time: read → identify sections → propose split → confirm → apply
- Add "See references/..." pointers in SKILL.md for progressive disclosure
- Longest skills to prioritize: [list top 3 by line count]

**2. Broken references — N skills**
Some skills reference files in references/ that don't exist yet.
Options: create the missing reference files, or remove the broken links.
Files with broken references: [list]

**3. Imperative form — N skills**
These skills use advisory phrasing ("you should check") instead of
direct commands ("Check", "Run", "Verify"). Rewrite instructions as commands.

**4. Progressive Disclosure — N skills**
Skills with references/ directories should include explicit "See references/..."
pointers in their body to guide readers to detailed material.

Would you like help with any of these? I can start with P5 splitting for your
longest skills, or fix broken references.
```

## Auto-fix Rules

- **P1 (frontmatter):** insert missing YAML with auto-derived `name` and `description`
- **P2 (name):** derive from filename, convert to kebab-case
- **P3 (description):** transform to "Use when..." pattern, preserve Korean/proper nouns
- **P4 (invocation):** apply reference heuristics, confirm with user

## Guardrails

- Never edit files in `--diagnose` mode
- Never run `--upgrade` without a successful backup
- Never auto-apply P3 or P4 without showing the user what will change
- Always re-run diagnosis after edits and show before/after comparison
- Surface P5+ items with clear guidance on how to resolve them manually
- Present expected impact at every decision point so the user can make informed choices
