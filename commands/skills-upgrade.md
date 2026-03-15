---
description: "Skills 2.0 auto-diagnosis and upgrade. /skills-upgrade [--diagnose|--dry-run|--upgrade] [--path <dir>]"
allowedTools: Bash, Read, Write, Edit, Glob, Grep, AskUserQuestion
---

# /skills-upgrade

Upgrade a skill directory to Skills 2.0 compliance through a four-phase pipeline.

## Usage

```bash
/skills-upgrade
/skills-upgrade --diagnose
/skills-upgrade --dry-run
/skills-upgrade --upgrade
/skills-upgrade --path /custom/skills/dir
```

Default behavior:
- mode: `--diagnose`
- path: `$HOME/.claude/skills/`

## Mode Matrix
| Mode | Flag | Behavior |
|------|------|----------|
| Diagnose | `--diagnose` (default) | Read-only, report only |
| Dry Run | `--dry-run` | Show plan, no changes |
| Upgrade | `--upgrade` | Backup → auto-fix → re-diagnose |

## Phase 1 — Scan
1. Parse `$ARGUMENTS` for:
   - `--diagnose`
   - `--dry-run`
   - `--upgrade`
   - `--path <dir>`
2. Resolve target directory. If `--path` is absent, use `$HOME/.claude/skills/`.
3. Run diagnostics in JSON mode:

```bash
scripts/diagnose.sh "$TARGET_PATH" --json
```

4. Parse the JSON into a working structure such as:
   - overall compliance
   - scanned skill count
   - issue counts by priority
   - affected files by `P1` to `P7`
5. If diagnosis fails, stop and show the raw stderr/output.

## Phase 2 — Report
Present a concise report with three sections:

### Compliance Summary
| Metric | Value |
|--------|-------|
| Skills scanned | `<n>` |
| Overall compliance | `<percent>` |
| P1 issues | `<count>` |
| P2 issues | `<count>` |
| P3+ issues | `<count>` |

### Top Issues
- missing frontmatter
- malformed or non-normalized `name`
- description missing `Use when...`
- missing invocation control on references
- oversized `SKILL.md`
- orphan directories or broken references

### Priority Actions
- `P1`: add frontmatter
- `P2`: normalize `name`
- `P3`: generate trigger-focused description
- `P4`: add invocation control where appropriate
- `P5+`: split files or fix structure manually/semi-automatically

## Phase 3 — Plan
Run this phase for `--dry-run` and `--upgrade`.

Show a proposed change plan before editing:
- target path
- total files affected
- backups to be created
- grouped file lists for `P1`, `P2`, `P3`, and `P4`
- notes for `P5+` items that need manual follow-up

Suggested plan layout:

```markdown
## Planned Changes
- P1: 4 files will receive frontmatter
- P2: 3 files will have `name` normalized
- P3: 5 descriptions will be generated for review
- P4: 2 reference files will get invocation control
```

## Phase 4 — Execute
Run this phase only for `--upgrade`.

### Step 4.1 Backup
Create a backup first:

```bash
scripts/backup.sh "$TARGET_PATH"
```

If backup fails, stop.

### Step 4.2 P1 — Add frontmatter
For each missing-frontmatter file, insert:

```yaml
---
name: {filename-derived}
description: Use when...
---
```

### Step 4.3 P2 — Normalize `name`
Derive from filename or directory name:
- lowercase preferred
- replace spaces and special characters with hyphens
- keep letters, numbers, and hyphens only

### Step 4.4 P3 — Generate description
Generate a candidate `description` from the first `# Heading` and first explanatory paragraph.

Rules:
- start with `Use when...`
- describe triggers only
- avoid workflow summary
- keep third-person tone

Before applying, ask the user to approve each candidate or batch:
- recommended option first
- include original vs proposed text
- allow keep/edit/skip

### Step 4.5 P4 — Add invocation control
Use reference heuristics to identify files that should disable model invocation, then ask for approval before editing.

Heuristics:
- file lives under `references/`
- content is API/reference material rather than workflow guidance
- file is linked from `SKILL.md` as supplemental detail

AskUserQuestion flow:
- show proposed files
- explain why each qualifies
- allow approve all / selective / skip

### Step 4.6 Re-diagnose
Run diagnostics again:

```bash
scripts/diagnose.sh "$TARGET_PATH" --json
```

### Step 4.7 Report before/after
Output a comparison table:

| Metric | Before | After |
|--------|--------|-------|
| Overall compliance | `<before>` | `<after>` |
| P1 issues | `<before>` | `<after>` |
| P2 issues | `<before>` | `<after>` |
| P3 issues | `<before>` | `<after>` |
| P4 issues | `<before>` | `<after>` |

## Auto-fix Rules
- **P1 (frontmatter):** insert missing YAML with `name` and placeholder `description`
- **P2 (name):** derive from filename, convert special chars to hyphens
- **P3 (description):** synthesize from title + first paragraph, then confirm with AskUserQuestion
- **P4 (invocation):** apply reference heuristics, then confirm with AskUserQuestion

## Guardrails
- Never edit files in `--diagnose` mode
- Never run `--upgrade` without a successful backup
- Never auto-apply P3 or P4 silently
- Always re-run diagnosis after edits
- Surface `P5+` items clearly when manual work is still required
