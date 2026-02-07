# Superpowers - 저장소 구조 분석

## 개요

**Superpowers**는 Claude AI 에이전트를 위한 종합적인 스킬 라이브러리로, 소프트웨어 개발을 위한 구조화된 워크플로우를 제공합니다.

- **저장소**: https://github.com/obra/superpowers
- **버전**: 4.2.0
- **라이선스**: MIT
- **작성자**: Jesse Vincent

---

## 디렉토리 구조

```
superpowers/
├── .claude-plugin/              # Claude Code 플러그인 설정
│   ├── plugin.json             # 플러그인 메타데이터 (v4.2.0)
│   └── marketplace.json        # 마켓플레이스 설정
├── .codex/                     # Codex 통합 파일
├── .opencode/                  # OpenCode 통합 파일
│   ├── INSTALL.md
│   └── plugins/
├── .github/                    # GitHub 설정
├── .notes/                     # 개인 분석 노트 (gitignore됨)
├── agents/                     # Agent 설정 파일
├── commands/                   # 명령어 정의 (3개)
│   ├── brainstorm.md
│   ├── execute-plan.md
│   └── write-plan.md
├── docs/                       # 문서
│   ├── README.codex.md
│   ├── README.opencode.md
│   ├── testing.md
│   ├── plans/
│   └── windows/
├── hooks/                      # 세션 훅
│   ├── hooks.json             # 훅 설정
│   ├── session-start.sh       # 세션 초기화 스크립트
│   └── run-hook.cmd           # Windows 훅 러너
├── lib/                        # 핵심 라이브러리
│   └── skills-core.js         # 스킬 발견 및 해석 로직
├── skills/                     # 스킬 라이브러리 (15개 핵심 스킬)
│   ├── brainstorming/
│   ├── dispatching-parallel-agents/
│   ├── executing-plans/
│   ├── finishing-a-development-branch/
│   ├── receiving-code-review/
│   ├── requesting-code-review/
│   ├── subagent-driven-development/
│   ├── systematic-debugging/
│   ├── test-driven-development/
│   ├── using-git-worktrees/
│   ├── using-superpowers/
│   ├── verification-before-completion/
│   ├── writing-plans/
│   ├── writing-skills/
│   │   └── examples/           # 스킬 생성 예제
│   └── [각 스킬은 SKILL.md 포함]
├── tests/                      # 테스트 스위트
│   ├── claude-code/
│   ├── explicit-skill-requests/
│   ├── opencode/
│   ├── skill-triggering/
│   └── subagent-driven-dev/
├── README.md                   # 메인 문서
├── RELEASE-NOTES.md           # 버전 히스토리
└── LICENSE                    # MIT 라이선스
```

---

## 기술 스택

### 언어 및 런타임
- **JavaScript/Node.js** - 핵심 라이브러리 및 도구
- **TypeScript** - 예제에 사용 (e.g., `condition-based-waiting-example.ts`)
- **Bash/Shell** - 훅 스크립트 (`session-start.sh`, `run-hook.cmd`)
- **Markdown** - 모든 스킬 문서 및 명령어

### 주요 의존성/통합
- **Claude Code** - 주요 플랫폼 (내장 플러그인 시스템)
- **Codex** - 대체 AI 플랫폼 지원
- **OpenCode** - 대체 AI 플랫폼 지원
- **Git** - 버전 관리 통합 (worktrees, branching)
- **Node.js 내장 모듈**: `fs`, `path`, `child_process`, `execSync`

---

## 주요 진입점

### 1. 플러그인 시스템 (Claude Code)
- **플러그인 설정**: `.claude-plugin/plugin.json`
  - 이름: "superpowers"
  - 설명: "Core skills library for Claude Code"
  - 버전: 4.2.0

### 2. 훅 시스템
- **설정**: `hooks/hooks.json`
  - 트리거: `startup`, `resume`, `clear`, `compact`
  - 동작: `hooks/session-start.sh` 비동기 실행
  - 목적: 세션 시작 시 superpowers 초기화

### 3. 세션 초기화
- **스크립트**: `hooks/session-start.sh`
  - `using-superpowers` 스킬 내용 읽기
  - Claude Code 컨텍스트에 주입
  - 레거시 스킬 디렉토리 확인 및 경고

### 4. 스킬 라이브러리
- **핵심 라이브러리**: `lib/skills-core.js`
  - **함수**:
    - `extractFrontmatter()` - SKILL.md 파일에서 YAML 메타데이터 파싱
    - `findSkillsInDir()` - 최대 깊이 3으로 재귀적 스킬 발견
    - `resolveSkillPath()` - 스킬 이름 해석, 섀도잉 처리 (개인 스킬이 superpowers 오버라이드)
    - `checkForUpdates()` - git 저장소에서 업데이트 확인
    - `stripFrontmatter()` - 스킬 내용에서 YAML frontmatter 제거

---

## 핵심 스킬 (15개)

### 테스팅 & 디버깅
- `test-driven-development` - RED-GREEN-REFACTOR 사이클
- `systematic-debugging` - 4단계 근본 원인 분석
- `verification-before-completion` - 수정 검증 확인

### 협업 & 계획
- `brainstorming` - 소크라테스식 디자인 개선
- `writing-plans` - 상세 구현 계획 (2-5분 작업)
- `executing-plans` - 체크포인트가 있는 배치 실행
- `subagent-driven-development` - 2단계 리뷰 프로세스
- `dispatching-parallel-agents` - 동시 서브에이전트 워크플로우

### 코드 리뷰 & 워크플로우
- `requesting-code-review` - 사전 리뷰 체크리스트
- `receiving-code-review` - 피드백 응답
- `using-git-worktrees` - 격리된 병렬 브랜치
- `finishing-a-development-branch` - Merge/PR 결정

### 메타/인프라
- `writing-skills` - 베스트 프랙티스 스킬 생성
- `using-superpowers` - 스킬 시스템 소개

---

## 스킬 포맷

각 스킬은 표준 구조를 따릅니다:

```
skills/[skill-name]/
├── SKILL.md                    # 메인 스킬 명령어 (YAML frontmatter 포함)
├── [선택적 지원 파일]
└── [examples/]
```

**SKILL.md Frontmatter 예시:**
```yaml
---
name: brainstorming
description: "Use when creating features... Explores user intent, requirements and design before implementation."
---
```

---

## 핵심 워크플로우

### 기본 개발 흐름
1. **Brainstorming** (디자인 단계) → 디자인 문서 저장
2. **Using Git Worktrees** (격리) → 피처 브랜치 생성
3. **Writing Plans** (작업 분해) → 명세가 있는 2-5분 작업
4. **Subagent-Driven Development** (구현) → 2단계 리뷰
5. **Test-Driven Development** (검증) → RED-GREEN-REFACTOR
6. **Requesting Code Review** (품질 체크)
7. **Finishing Development Branch** (merge/PR 결정)

---

## 철학 & 원칙

- **Test-Driven Development (TDD)** - 테스트 우선 작성
- **Systematic over ad-hoc** - 프로세스 중심 접근
- **Complexity reduction** - 주요 목표로서의 단순성
- **Evidence over claims** - 성공 선언 전 검증
- **YAGNI** - You Aren't Gonna Need It
- **DRY** - Don't Repeat Yourself

---

## 요약

Superpowers는 AI 에이전트가 전문적인 소프트웨어 개발 관행을 따르도록 안내하는 잘 구조화된 모듈식 스킬 프레임워크입니다. 컨텍스트 기반 자동 트리거되는 구성 가능한 명령어 세트를 통해 체계적인 프로세스, 협업, 테스팅, 코드 품질을 강조합니다.
