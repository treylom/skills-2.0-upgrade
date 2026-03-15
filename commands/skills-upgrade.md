---
description: "Skills 2.0 auto-diagnosis and upgrade. /skills-upgrade [--diagnose|--dry-run|--upgrade] [--path <dir>] [skill-name...]"
allowedTools: Bash, Read, Write, Edit, Glob, Grep, AskUserQuestion
---

# /skills-upgrade

Diagnose and upgrade skills to Skills 2.0 compliance through a guided, phase-by-phase workflow. Supports single skill, multiple skills, or full directory scan.

## Usage

```bash
/skills-upgrade                              # Interactive — asks what to upgrade
/skills-upgrade my-skill                     # Single skill by name
/skills-upgrade skill-a skill-b skill-c      # Multiple skills by name
/skills-upgrade --all                        # All skills in default directory
/skills-upgrade --path /custom/skills/       # All skills in custom directory
/skills-upgrade --diagnose                   # Diagnose only (no changes)
/skills-upgrade --dry-run                    # Preview changes
/skills-upgrade --upgrade                    # Interactive guided upgrade
```

Default path: `$HOME/.claude/skills/`

## Execution Flow

### Step 0 — Target Selection (MANDATORY)

Before any diagnosis, determine what the user wants to upgrade.

#### Case A: Arguments provided

Parse `$ARGUMENTS` for skill names, flags, and paths:

| Pattern | Interpretation |
|---------|---------------|
| `my-skill` | Single skill — find `my-skill.md` or `my-skill/SKILL.md` in skills dir |
| `skill-a skill-b` | Multiple skills — find each by name |
| `--all` | All skills in default directory |
| `--path /dir/` | All skills in specified directory |
| `--path /dir/ skill-a` | Specific skill in specified directory |
| (no skill names, no --all) | Go to Case B (interactive) |

Skill name resolution order:
1. `$SKILLS_DIR/{name}.md` (flat file)
2. `$SKILLS_DIR/{name}/SKILL.md` (directory skill)
3. Exact path if absolute path provided

If a skill name is not found, list available skills and ask the user to pick.

#### Case B: No arguments (interactive)

Run full directory scan first to show what's available:

```bash
scripts/diagnose.sh "$SKILLS_DIR" --json
```

Then present the selection:

```
AskUserQuestion:
  "What would you like to upgrade?"
  Options:
    - "Specific skill(s)" — I'll show you the list to pick from
    - "All skills" — Scan and upgrade the entire directory
```

If "Specific skill(s)" selected, show a scored list and let the user pick:

```markdown
## Your Skills (sorted by compliance score)

| # | Skill | Score | Top Issue |
|---|-------|-------|-----------|
| 1 | my-broken-skill | 48% | Missing frontmatter |
| 2 | old-helper | 62% | Body >500 lines |
| 3 | api-guide | 75% | No "Use when..." |
| ... | ... | ... | ... |
| N | perfect-skill | 100% | — |

Skills below 100% compliance: M of N
```

```
AskUserQuestion:
  "Which skills would you like to upgrade? Enter skill names or numbers (comma-separated), or 'all'."
  (free text input)
```

Parse the response:
- Numbers: map to skill names from the table
- Names: validate they exist
- "all": upgrade everything

#### Result of Step 0

Set `TARGET_SKILLS` to one of:
- A list of specific file paths (single or multiple)
- `"all"` (full directory scan)

All subsequent phases operate only on `TARGET_SKILLS`.

### Phase 1 — Scan

Run diagnostics on the selected scope:

```bash
# Single/multiple skills: diagnose each individually
for skill in $TARGET_SKILLS; do
  scripts/diagnose.sh "$skill" --json
done

# All skills: diagnose the directory
scripts/diagnose.sh "$TARGET_PATH" --json
```

Parse JSON into: overall compliance, skill count, issue counts by P1-P7.
If diagnosis fails, stop and show the raw output.

### Phase 2 — Report

Present a clear, friendly report scoped to the selected skills.

#### For specific skill(s):

```markdown
## Compliance Report: {skill-name}

Score: X% (N of 12 checks passed)

| # | Check | Weight | Status | Note |
|---|-------|--------|--------|------|
| 1 | Frontmatter | 20% | PASS/FAIL | ... |
| 2 | Name field | 8% | PASS/FAIL | ... |
| ... | ... | ... | ... | ... |

Issues found: [list]
Suggestions: [list]
```

#### For all skills:

```markdown
## Your Skills 2.0 Compliance Report

| Metric | Value |
|--------|-------|
| Skills scanned | N |
| Overall compliance | X% |

### Issue Breakdown

| Priority | Count | What it means | Impact if fixed |
|----------|-------|---------------|-----------------|
| P1 (frontmatter) | N | Skills can't be discovered by skill loaders | High |
| P2 (name) | N | Invalid name breaks skill identification | Medium |
| P3 (description) | N | Missing "Use when..." reduces discoverability | High aggregate |
| P4 (invocation) | N | Reference files load unnecessary context | Low per skill |
| P5 (body length) | N | Body >500 lines hurts token efficiency | Medium |
| P6-P7 (structure) | N | Directory or orphan issues | Low |
```

#### Upgrade Path (always show)

```markdown
### Recommended Upgrade Path

| Phase | What it fixes | Expected improvement | Automation |
|-------|--------------|---------------------|------------|
| P1-P2 | Frontmatter + name | +10-16%p | Fully automatic |
| P3 | Description "Use when..." | +8-16%p | Semi-auto (batch) |
| P4 | Invocation control | +1-3%p | Semi-auto (confirm) |
| P5+ | File splitting + structure | +2-6%p | Manual (Opus) |

The biggest bang-for-buck is P1-P3: typically 62% → 94% with minimal effort.
```

If mode is `--diagnose`, stop here.

### Phase 3 — Plan (--dry-run and --upgrade)

Show proposed changes scoped to selected skills:

```markdown
## Planned Changes ({scope description})

- **P1**: N files will receive frontmatter
- **P2**: N files will have `name` normalized
- **P3**: N descriptions will be transformed to "Use when..."
- **P4**: N reference files will get invocation control

### P5+ (manual follow-up)
- N files exceed 500 lines
- N files have broken references

Backup will be created before any changes.
```

If mode is `--dry-run`, stop here.

### Phase 4 — Execute (--upgrade only)

#### Step 4.0 — Ask upgrade scope

```
AskUserQuestion:
  "How far would you like to upgrade?"
  Options:
    - "P1-P2 only (automatic, safe)" — frontmatter + name fixes
    - "P1-P3 (recommended)" — includes description optimization
    - "P1-P4 (thorough)" — includes invocation control
    - "P1-P4 + P5 guidance" — full upgrade with splitting guide
```

#### Step 4.1 — Backup

```bash
scripts/backup.sh "$TARGET_PATH"
```

If backup fails, stop. Show path and size.

#### Step 4.2 — P1-P2 (Automatic)

For specific skills, edit each file directly using Read + Edit tools.
For all skills, run the batch script:

```bash
python3 scripts/batch-p1p2p3.py "$TARGET_PATH" --dry-run  # preview
python3 scripts/batch-p1p2p3.py "$TARGET_PATH"             # apply
```

For single/multiple skills, apply P1-P2 fixes inline:
1. Read the skill file
2. Check frontmatter presence → add if missing
3. Check name field → normalize if invalid
4. Write the fixed file

Report: "P1-P2 complete: N frontmatter added, M names normalized."

#### Step 4.3 — P3 (Semi-automatic)

Show description transformation previews:

```markdown
### Description Changes

| Skill | Before | After |
|-------|--------|-------|
| {name} | "Analyzes code quality" | "Use when analyzing code quality" |

Proceed with these changes?
```

Wait for confirmation. Apply or skip.

#### Step 4.4 — P4 (Confirmation required)

Identify reference-type skills, present candidates, ask for approval.

#### Step 4.5 — Re-diagnose

Re-run diagnosis on the same scope (selected skills or full directory).

#### Step 4.6 — Before/After Report

```markdown
## Upgrade Complete!

| Metric | Before | After | Change |
|--------|--------|-------|--------|
| Compliance | X% | Y% | +Z%p |
| P1 issues | N | 0 | -N |
| P2 issues | N | 0 | -N |
| P3 issues | N | M | -K |
| P4 issues | N | M | -K |
| P5+ issues | N | N | (manual) |
```

#### Step 4.7 — Beyond P4 Guidance

```markdown
### What's Left?

**1. Body >500 lines (P5) — N skills**
Split into SKILL.md + references/. Use Opus model.
Longest: [list top 3]

**2. Broken references — N skills**
Create missing reference files or remove broken links.

**3. Imperative form — N skills**
Rewrite "you should" → direct commands ("Run", "Check").

Would you like help with any of these?
```

## Auto-fix Rules

- **P1 (frontmatter):** insert YAML with auto-derived `name` and `description`
- **P2 (name):** derive from filename, convert to kebab-case
- **P3 (description):** transform to "Use when..." pattern, preserve Korean/proper nouns
- **P4 (invocation):** apply reference heuristics, confirm with user

## Guardrails

- Never edit files in `--diagnose` mode
- Never run `--upgrade` without a successful backup
- Never auto-apply P3 or P4 without showing the user what will change
- Always re-run diagnosis after edits and show before/after comparison
- Surface P5+ items with clear guidance on how to resolve them manually
- Present expected impact at every decision point
- Respect target scope: if user selected specific skills, do not touch other files
