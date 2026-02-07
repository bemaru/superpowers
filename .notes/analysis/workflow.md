# 워크플로우 메커니즘 분석

## 1. BRAINSTORMING → PLANNING → IMPLEMENTATION 흐름

### 1단계: **Brainstorming** (`skills/brainstorming/SKILL.md`)
코드 작성 전 체계적인 아이디어 개선으로 워크플로우 시작:

**프로세스:**
- **아이디어 이해** - 에이전트가 프로젝트 컨텍스트(파일, 문서, 커밋) 확인 후 명확화 질문을 한 번에 하나씩
- **접근 방식 탐색** - 트레이드오프가 있는 2-3가지 다른 접근 방식 제안
- **디자인 제시** - 디자인을 200-300단어 섹션으로 나누어 각 섹션 후 검증
- **커버 영역:** 아키텍처, 컴포넌트, 데이터 흐름, 에러 처리, 테스팅

**출력:** 디자인 문서를 `docs/plans/YYYY-MM-DD-<topic>-design.md`에 저장하고 git에 커밋

### 2단계: **Using Git Worktrees** (`skills/using-git-worktrees/SKILL.md`)
전용 작업 공간을 만드는 격리 레이어:

**프로세스:**
1. 디렉토리 선호도 감지 (`.worktrees/` > `worktrees/` > CLAUDE.md 설정 > 사용자 선택)
2. 디렉토리가 git-ignored인지 검증 (중요한 안전 확인)
3. 격리된 worktree 생성: `git worktree add path -b feature-branch`
4. 프로젝트 설정 자동 감지 및 실행 (npm install, cargo build, poetry install, go mod download)
5. 테스트 스위트 실행으로 깨끗한 베이스라인 검증

**안전 기능:**
- worktree 내용의 우발적인 커밋 방지
- 진행 전 깨끗한 테스트 베이스라인 확인
- 디렉토리 선택이 프로젝트 규칙 존중

### 3단계: **Writing Plans** (`skills/writing-plans/SKILL.md`)
디자인을 작은 단위 구현 작업(각 2-5분)으로 변환:

**계획 구조 요구사항:**
- 기능 이름 및 목표 (한 문장)
- 아키텍처 개요 (2-3문장)
- 기술 스택 명세
- 세분화된 단계로 나눈 작업:
  - "실패하는 테스트 작성" (단계)
  - "실패 확인을 위해 실행" (단계)
  - "최소 코드 구현" (단계)
  - "통과 확인을 위해 테스트 실행" (단계)
  - "커밋" (단계)

**각 작업 포함:**
- 정확한 파일 경로
- 완전한 코드 ("검증 추가"가 아닌 실제 코드)
- 예상 출력이 있는 정확한 명령
- 테스트 실행 단계

**출력:** 계획을 `docs/plans/YYYY-MM-DD-<feature-name>.md`에 저장

### 4단계: **구현 실행** (듀얼 경로)

#### 경로 A: Subagent-Driven Development (`skills/subagent-driven-development/SKILL.md`)
작업당 새 서브에이전트로 동일 세션에서 계획 실행:

**작업당 워크플로우:**
1. 전체 작업 텍스트 및 컨텍스트로 **implementer subagent 디스패치**
2. 구현 시작 전 **질문에 답변**
3. **Implementer가 실행** TDD 따름 (RED-GREEN-REFACTOR)
4. 코드가 요구사항과 일치하는지 검증하기 위해 **spec-compliance reviewer 디스패치**
5. 검토자가 불일치를 발견하면 **명세 격차 수정**
6. 구현 품질을 확인하기 위해 **code-quality reviewer 디스패치**
7. 발견되면 **품질 이슈 수정**
8. **작업을 완료로 표시**

**2단계 리뷰 게이트:**
- 명세 준수 (과도/과소 구축 없음)
- 코드 품질 (잘 구조화된 구현)

#### 경로 B: Executing Plans (`skills/executing-plans/SKILL.md`)
배치 체크포인트가 있는 병렬 세션에서 계획 실행:

**프로세스:**
1. 계획을 로드하고 비판적으로 검토
2. 첫 3개 작업을 배치로 실행
3. 계획에 명시된 대로 검증 실행
4. 결과 보고 및 피드백 대기
5. 필요시 변경 적용
6. 다음 배치 실행
7. 완료될 때까지 반복

**핵심 원칙:** 배치 간 사람 리뷰 체크포인트가 있는 배치 실행

---

## 2. HOOKS 시스템

`hooks/` 폴더에 위치한 훅 시스템은 세션 시작 시 자동 컨텍스트 주입을 가능하게 함:

### Hook 설정 (`hooks/hooks.json`)
```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup|resume|clear|compact",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PLUGIN_ROOT}/hooks/session-start.sh",
            "async": true
          }
        ]
      }
    ]
  }
}
```

### Session Start 스크립트 (`hooks/session-start.sh`)
**모든 세션 시작 시, 이 훅은:**

1. **레거시 superpowers 디렉토리 감지** - 사용자에게 `~/.config/superpowers/skills` (이전 위치)가 있으면 경고
2. **using-superpowers 스킬 로드** - `skills/using-superpowers/SKILL.md`의 전체 내용 읽기
3. **JSON 임베딩을 위한 이스케이프** - 내용을 JSON 안전 형식으로 변환
4. **컨텍스트로 주입** - 스킬 소개를 포함하는 `additionalContext`와 함께 JSON 출력

**Hook 출력 형식:**
```json
{
  "hookSpecificOutput": {
    "hookEventName": "SessionStart",
    "additionalContext": "<EXTREMELY_IMPORTANT>\nYou have superpowers.\n\n[Full using-superpowers skill content]\n[Migration warning if needed]\n</EXTREMELY_IMPORTANT>"
  }
}
```

**효과:** 모든 세션이 수동 스킬 호출 없이 자동으로 superpowers 부트스트랩을 포함

---

## 3. COMMANDS 시스템

`commands/` 폴더에 위치한 3개의 명령 정의가 핵심 워크플로우를 트리거:

### 명령: `/brainstorm`
**파일:** `commands/brainstorm.md`
- **설명:** 창의적 작업(피처, 컴포넌트, 수정) 전 필수
- **동작:** `disable-model-invocation: true` - 직접 모델 실행 방지
- **액션:** `superpowers:brainstorming` 스킬 호출 및 정확히 따름

### 명령: `/write-plan`
**파일:** `commands/write-plan.md`
- **설명:** 작은 단위 작업이 있는 상세 구현 계획 생성
- **동작:** `superpowers:writing-plans` 스킬 호출
- **요구사항:** brainstorming 단계에서 승인된 디자인 따름

### 명령: `/execute-plan`
**파일:** `commands/execute-plan.md`
- **설명:** 리뷰 체크포인트가 있는 배치로 계획 실행
- **동작:** `superpowers:executing-plans` 스킬 호출
- **컨텍스트:** 사람 체크포인트가 있는 병렬 세션 실행

---

## 4. 플랫폼 통합

시스템은 3가지 독립적인 플랫폼 아키텍처 지원:

### 플랫폼 1: Claude Code (네이티브 플러그인 시스템)

**설치:**
```bash
/plugin marketplace add obra/superpowers-marketplace
/plugin install superpowers@superpowers-marketplace
```

**설정 파일:**
- `.claude-plugin/plugin.json` - 플러그인 메타데이터 (버전 4.2.0)
- `.claude-plugin/marketplace.json` - 마켓플레이스 등록

**통합 메커니즘:**
- 훅 시스템 (hooks/hooks.json + hooks/session-start.sh)
- SessionStart 훅이 모든 세션 시작 시 컨텍스트 주입
- 명령이 자동으로 인식 및 트리거됨
- `Skill` 도구를 통해 스킬 접근 (Claude Code에 네이티브)

**사용 가능한 도구:**
- `Skill` - 스킬 호출
- `Read`/`Write`/`Edit` - 파일 작업
- `Bash` - 터미널 명령
- `Task` - 서브에이전트 조정 (TodoWrite 동등)
- `Grep` - 내용 검색

### 플랫폼 2: Codex (심볼릭 링크 기반 발견)

**설치:** (`.codex/INSTALL.md`)
```bash
git clone https://github.com/obra/superpowers.git ~/.codex/superpowers
mkdir -p ~/.agents/skills
ln -s ~/.codex/superpowers/skills ~/.agents/skills/superpowers
```

**발견 메커니즘:**
- `~/.agents/skills/superpowers`로의 네이티브 심볼릭 링크를 통한 스킬 발견
- 부트스트랩 컨텍스트 주입 없음 (훅 지원 없음)
- 수동 스킬 호출 필요

**도구 매핑:**
- `Skill` 도구 → 네이티브 스킬 도구
- `TodoWrite` → (사용 불가, 수동 작업 추적 사용)
- `Task` → `@mention` 서브에이전트 구문
- File/Bash 도구 → 네이티브 동등물

**업데이트:** `~/.codex/superpowers`에서 `git pull`을 통해

### 플랫폼 3: OpenCode (플러그인 + 심볼릭 링크 시스템)

**설치:** (`.opencode/INSTALL.md`)
```bash
git clone https://github.com/obra/superpowers.git ~/.config/opencode/superpowers
ln -s ~/.config/opencode/superpowers/.opencode/plugins/superpowers.js ~/.config/opencode/plugins/
ln -s ~/.config/opencode/superpowers/skills ~/.config/opencode/skills/superpowers
```

**통합 메커니즘:**
- **플러그인 시스템** (`.opencode/plugins/superpowers.js`) - 다음을 수행하는 JavaScript 플러그인:
  - `using-superpowers` 스킬 내용 추출
  - 시스템 프롬프트 변환을 통해 주입 (에이전트 재설정 버그 #226 회피)
  - 도구 매핑 문서 제공

- **스킬 발견** - `~/.config/opencode/skills/superpowers`로의 심볼릭 링크
- **부트스트랩 주입** - `experimental.chat.system.transform` 훅을 통해

**플러그인의 도구 매핑:**
```javascript
- TodoWrite → update_plan
- Task with subagents → @mention 구문
- Skill tool → OpenCode의 네이티브 스킬 도구
- Read/Write/Edit/Bash → 네이티브 도구
```

**핵심 기능:** 스킬 우선순위 = 프로젝트 스킬 > 개인 스킬 > Superpowers 스킬

---

## 5. 완전한 개발 워크플로우 다이어그램

```
┌─────────────────────────────────────────────────────────────┐
│ 세션 시작                                                    │
│ 훅이 SessionStart 이벤트를 통해 using-superpowers 스킬 주입  │
└─────────────────┬───────────────────────────────────────────┘
                  ↓
         ┌────────────────┐
         │ BRAINSTORMING  │ (명령: /brainstorm)
         ├────────────────┤
         │ - 질문         │
         │ - 대안         │
         │ - 디자인 명세  │
         └────────┬───────┘
                  ↓
         ┌────────────────────┐
         │ 사용자 승인         │
         │ (읽기 & 검증)      │
         └────────┬───────────┘
                  ↓
    ┌─────────────────────────────┐
    │ GIT WORKTREE 설정           │
    ├─────────────────────────────┤
    │ - 디렉토리 감지             │
    │ - .gitignore 검증           │
    │ - 브랜치 생성               │
    │ - 프로젝트 설정 실행        │
    │ - 테스트 베이스라인 검증    │
    └──────────┬──────────────────┘
               ↓
    ┌──────────────────────────┐
    │ WRITE PLAN               │ (명령: /write-plan)
    ├──────────────────────────┤
    │ - 작업으로 분해          │
    │ - 작업당 2-5분           │
    │ - 완전한 코드            │
    │ - 정확한 명령            │
    └──────────┬───────────────┘
               ↓
    ┌──────────────────────────────────────────┐
    │ 실행 경로 선택                            │
    └─────────┬──────────────────────┬─────────┘
              ↓                      ↓
    ┌─────────────────┐   ┌──────────────────┐
    │ SUBAGENT-DRIVEN │   │ EXECUTING-PLANS  │
    │ (동일 세션)     │   │ (병렬)           │
    ├─────────────────┤   ├──────────────────┤
    │작업당:          │   │배치당 (3개 작업):│
    │- impl 디스패치  │   │- 실행            │
    │- Q&A            │   │- 검증            │
    │- TDD 구현       │   │- 보고            │
    │- 명세 리뷰      │   │- 피드백 대기     │
    │- 품질 리뷰      │   │- 반복            │
    │- 승인           │   └──────────────────┘
    └────────┬────────┘
             ↓
    ┌──────────────────────────────┐
    │ 개발 완료                     │
    ├──────────────────────────────┤
    │ 1. 모든 테스트 통과 검증      │
    │ 2. 4가지 옵션 제시:           │
    │    - 로컬에서 merge           │
    │    - PR 생성 & push           │
    │    - 그대로 유지              │
    │    - 작업 폐기                │
    │ 3. 선택 실행                  │
    │ 4. Worktree 정리              │
    └──────────┬───────────────────┘
               ↓
    ┌─────────────────────────────┐
    │ 통합 완료                    │
    │ (Merge/PR/Archive/Delete)    │
    └─────────────────────────────┘
```

---

## 6. TEST-DRIVEN DEVELOPMENT 통합

모든 구현 작업은 엄격한 TDD를 따름 (`skills/test-driven-development/SKILL.md`):

**RED-GREEN-REFACTOR 사이클:**
1. **RED** - 실패하는 테스트 작성 (하나의 동작)
2. **RED 검증** - 테스트가 올바르게 실패하는지 확인
3. **GREEN** - 최소 코드 구현
4. **GREEN 검증** - 테스트가 통과하는지 확인
5. **REFACTOR** - 정리 (테스트를 녹색으로 유지)

**철칙:** "실패하는 테스트 없이 프로덕션 코드 작성 금지"

**테스트 전에 코드를 작성했다면:** 삭제. 다시 시작.

---

## 7. 코드 리뷰 통합

작업 간, 코드는 `skills/requesting-code-review/SKILL.md`를 사용하여 리뷰됨:

**2단계 리뷰 (subagent-driven development에서):**
1. **명세 준수** - 코드가 요구사항과 정확히 일치하는지 검증
2. **코드 품질** - 구현 품질 확인

**Critical + Important 이슈는 진행을 차단** - 진행 전에 수정되어야 함

---

## 8. 핵심 워크플로우 원칙

1. **Ad-hoc보다 체계적** - 모든 단계가 정의된 프로세스를 가짐
2. **격리 우선** - Worktrees가 main 브랜치에서 작업 분리
3. **디자인 검증** - 코드 전에 명세 승인
4. **TDD 필수** - 테스트가 동작을 증명하고 회귀 방지
5. **작은 단위 작업** - 명확한 검증이 있는 각 2-5분
6. **다중 리뷰 게이트** - 명세 준수 + 코드 품질
7. **자율 실행** - 에이전트가 편차 없이 2시간 이상 작업 가능
8. **증거 기반** - 모든 주장이 테스트 출력으로 검증됨

---

## 9. 요약

**Superpowers** 시스템은 다음을 수행하는 완전한 개발 오케스트레이션 프레임워크입니다:

1. **자동으로 시작** - 훅이 세션 시작 시 컨텍스트 주입
2. **디자인 단계 안내** - Brainstorming → Plan → Implementation
3. **작업 격리** - Git worktrees가 main 브랜치 오염 방지
4. **TDD 강제** - 모든 코드가 테스트 우선 필요
5. **디자인 검증** - 코드 품질 전 명세 준수
6. **병렬화 가능** - 빠른 반복을 위한 서브에이전트 주도 개발
7. **체크포인트 제공** - 계획 검증, 테스트 검증, 리뷰 게이트
8. **3개 플랫폼 지원** - 네이티브 통합으로 Claude Code, Codex, OpenCode
9. **일반 작업 자동화** - 프로젝트 설정, 의존성 설치, 정리
10. **출력 아티팩트 생성** - 완전한 추적 가능성이 있는 디자인 문서, 계획, 구현

이것은 중요한 게이트에서 사람 감독과 함께 자율 에이전트 개발을 위해 설계된 **프로덕션급 워크플로우 엔진**입니다.
