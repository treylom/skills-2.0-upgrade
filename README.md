# Skills 2.0 Auto-Upgrade

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)]()

> Zero-token diagnostic engine + Claude Code skill for automatically upgrading skills to Skills 2.0 compliance.

**[한국어 버전은 아래를 참고하세요 / Korean version below](#skills-20-자동-업그레이드)**

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
- **Security scan**: Pre-deployment PII, secrets, and hardcoded path detection

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

### Security Scan (pre-deployment)

```bash
~/.claude/scripts/diagnose.sh /path/to/project --security
~/.claude/scripts/diagnose.sh /path/to/project --security --json
```

Detects 13 patterns across 3 severity levels:

| Severity | Detects |
|----------|---------|
| Critical | API keys (`sk-`, `ntn_`, `ghp_`), private key blocks |
| High | Bearer tokens, API key/secret assignments, passwords |
| Medium | Hardcoded user paths (`/home/`, `/mnt/c/Users`, `C:\Users`) |

Run before every `git push` or GitHub deployment to prevent accidental leaks.

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

Real-world migration of 151 skills:

| Phase | Compliance | What changed |
|-------|-----------|--------------|
| Baseline | 62% | No Skills 2.0 structure |
| After P1-P2 | 78% | Frontmatter + name normalization |
| After P1-P3 | **94.3%** | + description "Use when..." pattern |

Remaining gap to 100%: body >500 lines (13 skills, needs Opus splitting), broken references (13), imperative form (some)

### Self-Improvement

This tool was used to diagnose and upgrade itself through 4 iterations:

| Iteration | What happened | Result |
|-----------|--------------|--------|
| 1 | Initial 6-agent team build | 100% compliance on first build |
| 2 | Real-world validation on 151 skills | 62% → 94.3% (batch P1-P3 script created) |
| 3 | UX redesign + false positive fix | Phase-by-phase guide, 100% maintained |
| 4 | Security scan + self-scan | `--security` mode added, clean pass |

See the full [Self-Improvement Report](SELF-IMPROVEMENT-REPORT.md) for details.

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
├── README.md
└── SELF-IMPROVEMENT-REPORT.md
```

## Acknowledgments

This project builds on the work of:

- **[superpowers:writing-skills](https://github.com/anthropics/superpowers)** (Anthropic) — TDD-based skill authoring methodology, Claude Search Optimization (CSO), and official SKILL.md frontmatter specification.
- **[fivetaku/skillers-suda](https://github.com/fivetaku/skillers-suda)** (MIT) — 9-item structural validation (`validate-skill.sh`), Auto Eval methodology (`with_skill` vs `without_skill` A/B comparison), and description trigger optimization strategy.

Our 12-item diagnostic checklist merges 8 items from Skills 2.0 with 4 additional items from skillers-suda (third-person description, progressive disclosure, imperative form, and enhanced trigger validation).

## License

MIT — see [LICENSE](LICENSE) for details.

---

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

## 주요 기능

- **Zero-token 진단**: 순수 bash, LLM 호출 불필요
- **12항목 가중 체크리스트**: 종합 준수율 점수
- **P1-P4 자동 수정**: frontmatter, name, description 자동 수정
- **업그레이드 전 백업**: tar.gz 자동 백업
- **크로스 플랫폼**: WSL, macOS, Linux
- **JSON 출력**: CI/CD 통합용 머신 파싱 가능
- **슬래시 커맨드**: `/skills-upgrade` 3가지 모드
- **보안 스캔**: 배포 전 개인정보, 시크릿, 하드코딩 경로 탐지

## 사용법

### 진단 (기본)

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

### 드라이 런

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

### 업그레이드

```bash
/skills-upgrade --upgrade
```

예시 출력:

```text
Backup created: ~/.claude/skills-backup-20260315-143022.tar.gz
Re-diagnosing after fixes...
Compliance improved: 62.0% -> 78.2%
```

### 스크립트 직접 실행

```bash
~/.claude/scripts/diagnose.sh ~/.claude/skills/
~/.claude/scripts/diagnose.sh ~/.claude/skills/ --json
~/.claude/scripts/diagnose.sh ~/.claude/skills/ --verbose
~/.claude/scripts/diagnose.sh ~/.claude/skills/my-skill.md
```

### 보안 스캔 (배포 전)

```bash
~/.claude/scripts/diagnose.sh /path/to/project --security
~/.claude/scripts/diagnose.sh /path/to/project --security --json
```

3단계 심각도로 13개 패턴을 탐지합니다:

| 심각도 | 탐지 대상 |
|--------|----------|
| Critical | API 키 (`sk-`, `ntn_`, `ghp_`), 개인키 블록 |
| High | Bearer 토큰, API 키/시크릿 할당, 비밀번호 |
| Medium | 하드코딩된 사용자 경로 (`/home/`, `/mnt/c/Users`, `C:\Users`) |

`git push` 또는 GitHub 배포 전 반드시 실행하여 우발적 유출을 방지하세요.

## 12항목 진단 체크리스트

| # | 항목 | 가중치 | 통과 조건 |
|---|------|--------|----------|
| 1 | Frontmatter 존재 | 20% | 첫 줄이 `---`이고 닫는 `---`가 존재 |
| 2 | `name:` 필드 | 8% | frontmatter에 `name:`이 있고 하이픈+영숫자만 사용 |
| 3 | `description:` 필드 | 10% | frontmatter에 비어있지 않은 `description:` 값 존재 |
| 4 | Description `Use when...` 패턴 | 5% | description이 `Use when`으로 시작 (대소문자 무관) |
| 5 | Description 3인칭 | 3% | description에 `you` 또는 `your` 미포함 |
| 6 | 본문 ≤500줄 | 15% | frontmatter 제외 본문이 500줄 이하 |
| 7 | 디렉토리 구조 | 10% | 긴 스킬은 `SKILL.md` + `references/` 사용; 짧은 스킬은 자동 통과 |
| 8 | `disable-model-invocation` | 8% | 레퍼런스형 스킬에 `disable-model-invocation: true` 포함; 그 외 자동 통과 |
| 9 | 고아 디렉토리 없음 | 5% | 스킬 디렉토리에 `SKILL.md` 또는 최소 1개 `.md` 파일 존재 |
| 10 | 깨진 참조 없음 | 5% | `references/` 링크가 실제 파일로 연결됨 |
| 11 | Progressive Disclosure | 5% | `references/`가 있는 스킬은 본문에서 `references/`를 명시적으로 안내 |
| 12 | 명령형 지시문 | 6% | 명령형 지시문이 2인칭 조언 표현보다 많음 |

## 모델 가이드

| 작업 | 모델 | Effort | 이유 |
|------|------|--------|------|
| 진단만 | 아무 모델 (bash) | N/A | LLM 불필요 |
| P1-P2 자동 수정 | Sonnet | medium | 단순 패턴 매칭 |
| P3 파일 분할 | Opus | high | 콘텐츠 이해 필요 |
| 전체 업그레이드 | Opus | high | 분할 포함 |

## 크로스 플랫폼 호환

| 플랫폼 | 지원 수준 | 비고 |
|--------|----------|------|
| Claude Code | Full | 모든 기능 |
| Codex CLI | Partial | JSON 출력 모드 |
| Antigravity | Partial | 스킬 읽기 + bash |
| Cursor/Windsurf | Minimal | .md 읽기만 |

## 작동 원리

4-Phase 파이프라인:
1. **Scan**: 인자 파싱, diagnose.sh 실행
2. **Report**: 준수율 요약 생성
3. **Plan**: 변경 계획 표시 (dry-run/upgrade)
4. **Execute**: 백업 → 자동 수정 → 재진단 (upgrade만)

## 실제 사례

151개 스킬 마이그레이션 실측 결과:

| Phase | 준수율 | 변화 |
|-------|--------|------|
| Baseline | 62% | Skills 2.0 구조 없음 |
| P1-P2 후 | 78% | frontmatter + name 정규화 |
| P1-P3 후 | **94.3%** | + description "Use when..." 패턴 |

잔여 이슈: 500줄 초과 (13개, Opus 분할 필요), 깨진 참조 (13개), 명령형 문체 (일부)

### 자가 개선

이 도구는 자체 진단 도구로 스스로를 4회 반복 개선했습니다:

| 반복 | 내용 | 결과 |
|------|------|------|
| 1 | 6-agent 팀 초기 빌드 | 100% 준수율 초기 달성 |
| 2 | 151개 실제 스킬 검증 | 62% → 94.3% (배치 스크립트 작성) |
| 3 | UX 개선 + false positive 수정 | Phase별 가이드 추가, 100% 유지 |
| 4 | 보안 스캔 + 자체 적용 | `--security` 모드 추가, clean 통과 |

전체 리포트: [Self-Improvement Report](SELF-IMPROVEMENT-REPORT.md)

## 프로젝트 구조

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
└── README.md
```

## Acknowledgments

이 프로젝트는 다음 작업을 기반으로 합니다:

- **[superpowers:writing-skills](https://github.com/anthropics/superpowers)** (Anthropic) — TDD 기반 스킬 작성 방법론, Claude Search Optimization (CSO), 그리고 공식 SKILL.md frontmatter 명세.
- **[fivetaku/skillers-suda](https://github.com/fivetaku/skillers-suda)** (MIT) — 9-item structural validation (`validate-skill.sh`), Auto Eval 방법론 (`with_skill` vs `without_skill` A/B comparison), 그리고 description trigger optimization 전략.

이 프로젝트의 12-item diagnostic checklist는 Skills 2.0의 8개 항목과 skillers-suda의 추가 4개 항목(third-person description, progressive disclosure, imperative form, enhanced trigger validation)을 결합합니다.

## License

MIT — 자세한 내용은 [LICENSE](LICENSE)를 참고하세요.
