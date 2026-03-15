# Skills 2.0 자동 업그레이드

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)]()

> Zero-token diagnostic engine + Claude Code skill for automatically upgrading skills to Skills 2.0 compliance.

## Skills 2.0이란?

Skills 2.0은 Anthropic이 2026년에 정리한 스킬 프레임워크로, 재사용 가능한 에이전트 지침을 더 잘 찾고, 더 적은 토큰으로 불러오고, 더 쉽게 유지보수하도록 설계되었습니다.
핵심은 **Progressive Disclosure** 3-Tier 구조입니다. frontmatter는 트리거링용, `SKILL.md`는 실행 가이드용, `references/`는 필요할 때만 불러오는 대용량 참고자료용입니다.
현재 생태계는 **351K+ published skills** 규모까지 커졌기 때문에, 초기의 단일 파일 방식보다 검색 품질, 토큰 효율, 구조 일관성이 훨씬 중요해졌습니다.
이 프로젝트는 zero-token bash 진단과 가이드형 업그레이드 워크플로우로 기존 Claude Code 스킬을 그 구조로 이전할 수 있게 돕습니다.

## 빠른 시작

### 설치 (primary)

```bash
curl -fsSL https://raw.githubusercontent.com/treylom/skills-2.0-upgrade/main/install.sh | bash
```

### 로컬 클론에서 설치 (alternative)

```bash
bash install.sh
```

### 기본 사용법

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

예시 출력:

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

예시 출력:

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

예시 출력:

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

실제 94개 스킬 마이그레이션 사례:
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

이 프로젝트는 다음 작업을 기반으로 합니다:

- **[superpowers:writing-skills](https://github.com/anthropics/superpowers)** (Anthropic) — TDD 기반 스킬 작성 방법론, Claude Search Optimization (CSO), 그리고 공식 SKILL.md frontmatter 명세.
- **[fivetaku/skillers-suda](https://github.com/fivetaku/skillers-suda)** (MIT) — 9-item structural validation (`validate-skill.sh`), Auto Eval 방법론 (`with_skill` vs `without_skill` A/B comparison), 그리고 description trigger optimization 전략.

이 프로젝트의 12-item diagnostic checklist는 Skills 2.0의 8개 항목과 skillers-suda의 추가 4개 항목(third-person description, progressive disclosure, imperative form, enhanced trigger validation)을 결합합니다.

## License

MIT — 자세한 내용은 [LICENSE](LICENSE)를 참고하세요.
