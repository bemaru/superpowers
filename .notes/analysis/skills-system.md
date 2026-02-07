# Skills 시스템 상세 분석

## 1. 구조 개요

Superpowers 시스템은 `D:\repos\bemaru\fork\superpowers\skills\`에 14개의 독립적인 스킬 모듈을 포함한 **플러그인 기반 스킬 라이브러리**로 구성되어 있습니다.

### 핵심 특징
- 플랫 네임스페이스 (모든 스킬이 단일 디렉토리에)
- 각 스킬은 필수 `SKILL.md` 파일을 포함하는 폴더
- 선택적으로 지원 파일 포함 (프롬프트, 도구, 참조)
- Claude Code, Codex, OpenCode의 스킬 프레임워크를 통해 트리거

---

## 2. 사용 가능한 스킬 & 목적

### 핵심 개발 워크플로우

| 스킬 | 목적 | 트리거 조건 |
|------|------|-------------|
| **brainstorming** | 소크라테스식 대화를 통한 요구사항 개선, 디자인 대안 탐색, 디자인 문서 저장 | 창의적 작업 전 (피처, 컴포넌트, 기능) |
| **writing-plans** | 승인된 디자인을 작은 단위 작업(2-5분)으로 분해, 정확한 파일 경로와 완전한 코드 포함 | 디자인 승인 후, 구현 전 |
| **using-git-worktrees** | 스마트 디렉토리 선택과 깨끗한 테스트 베이스라인 검증으로 격리된 git worktrees 생성 | 피처 작업 또는 계획 실행 전 |
| **test-driven-development** | RED-GREEN-REFACTOR 사이클: 실패하는 테스트 → 최소 코드 → 통과 테스트 → 리팩터링 | 모든 피처 또는 버그 수정 구현 중 |
| **verification-before-completion** | 성공 주장 전 검증 명령 실행; "항상 주장 전 증거" | 완료/성공 주장 전 |

### 실행 & 협업

| 스킬 | 목적 | 트리거 조건 |
|------|------|-------------|
| **executing-plans** | 사람 리뷰 체크포인트가 있는 배치 실행 | 별도 세션의 다단계 계획, 사람이 참여하는 루프 |
| **subagent-driven-development** | 작업당 새 서브에이전트 + 2단계 리뷰 (명세 준수 → 코드 품질) | 동일 세션에서 연속 반복으로 계획 실행 |
| **dispatching-parallel-agents** | 병렬 작업을 위한 동시 서브에이전트 워크플로우 | 여러 독립적인 서브에이전트 동시 실행 |
| **requesting-code-review** | 사전 코드 품질을 위한 리뷰 체크리스트 | 개발 중 작업 간 |
| **receiving-code-review** | 코드 리뷰 피드백 응답을 위한 구조화된 워크플로우 | 코드 리뷰 피드백 수신 후 |
| **finishing-a-development-branch** | 테스트 검증 → merge/PR/keep/discard 옵션 제시 → 정리 | 모든 구현 작업 완료 후 |

### 문제 해결

| 스킬 | 목적 | 트리거 조건 |
|------|------|-------------|
| **systematic-debugging** | 수정 전 4단계 근본 원인 조사 (에러 읽기 → 재현 → 변경 확인 → 데이터 흐름 추적) | 모든 버그, 테스트 실패, 예상치 못한 동작 (특히 시간 압박 하에) |

### 메타/거버넌스

| 스킬 | 목적 | 트리거 조건 |
|------|------|-------------|
| **using-superpowers** | 스킬 시스템 소개; "1% 확률로 스킬 적용 = 반드시 호출" | 모든 세션 시작 시 |
| **writing-skills** | TDD 적응 사이클을 사용한 새 스킬 생성 (베이스라인 테스트 → 스킬 작성 → 허점 보완) | 배포 전 스킬 생성 또는 편집 |

---

## 3. 스킬 파일 포맷 & 규칙

### 파일 구조

```
skill-name/
  SKILL.md              # 필수: 메인 스킬 참조 (일반적으로 1-2 KB)
  supporting-file.*     # 선택: 프롬프트, 도구, 중요 참조
```

### SKILL.md 포맷

**YAML Frontmatter (필수, 정확히 2개 필드):**
```yaml
---
name: skill-name-with-hyphens
description: Use when [특정 트리거 조건 및 증상]
---
```

**Description 규칙 (발견에 중요):**
- "Use when..."으로 시작하여 트리거/증상에 집중 (프로세스 아님)
- 3인칭 시점
- 가능하면 최대 500자
- **절대 워크플로우 요약 안 함** (Claude가 전체 스킬 대신 설명을 따르게 됨)
- 키워드 포함: 에러 메시지, 증상, 도구 이름

**본문 섹션 (일반적):**
1. Overview - 이것이 무엇인가? 핵심 원칙 1-2문장
2. When to Use - 트리거 조건, 증상, 사용 사례
3. Core Pattern - 프로세스/기법 (플로우차트 포함 가능)
4. Quick Reference - 스캔을 위한 테이블 또는 체크리스트
5. Implementation - 코드 예제 또는 단계별 안내
6. Common Mistakes - 잘못되는 것 + 수정
7. Red Flags - STOP 조건 (자가 점검 목록)
8. Rationalization Prevention - 변명 테이블 + 현실
9. Integration - 필수/관련 다른 스킬 참조

---

## 4. 스킬 로딩 & 트리거 방법

### 로딩 메커니즘

스킬은 **Claude Code 플러그인 시스템**을 통해 발견 및 로드됩니다:

1. **플러그인 등록** (`.claude-plugin/plugin.json`):
   - 플러그인 이름: `superpowers` (v4.2.0)
   - `/plugin install superpowers@superpowers-marketplace`를 통해 자동 로드

2. **스킬 발견**:
   - 프레임워크가 `skills/` 디렉토리 스캔
   - `SKILL.md`가 있는 각 폴더가 사용 가능해짐
   - frontmatter의 메타데이터가 검색/매칭에 사용됨

3. **호출**:
   - 스킬은 **필수** (선택적 제안 아님)
   - `1% 확률로 스킬이 적용될 수 있음` 시 `Skill` 도구를 통해 트리거
   - 형식: `Invoke Skill: skill-name` 또는 동등한 형식

### 스킬 활성화 시점

**using-superpowers** 스킬에 따르면:

```
사용자 메시지 → "스킬 적용될 수 있나?" [다이아몬드 결정]
                ↓ (yes, 1%라도)
             Skill 도구 호출
                ↓
             알림: "[스킬] 사용하여 [목적]"
                ↓
             체크리스트 있음? → TaskCreate todos 생성 (필요시)
                ↓
             스킬 정확히 따름
```

**스킬을 적용해야 함을 나타내는 레드 플래그:**
- "이건 그냥 간단한 질문이에요" → 질문도 작업, 스킬 확인
- "먼저 더 많은 컨텍스트 필요" → 스킬 확인이 명확화 질문보다 먼저
- "먼저 탐색할게요" → 스킬이 탐색 방법을 알려줌
- "X를 빠르게 확인할 수 있어요" → 스킬이 무질서한 탐색 방지

---

## 5. 스킬 프레임워크 디자인 원칙

### 발견 최적화 (Claude Search Optimization - CSO)

Frontmatter description이 **중요**한 이유:
- Claude가 어떤 스킬을 로드할지 결정하기 위해 description 읽음
- description이 워크플로우를 요약하면 → Claude가 전체 스킬 대신 description을 따를 수 있음
- 예: "작업 간 코드 리뷰"라고 설명하면 Claude가 2번이 아닌 1번 리뷰 수행

**해결책:** Description = 트리거 조건만, 워크플로우 요약 없음

### 토큰 효율성

목표 단어 수:
- 시작 워크플로우: < 150단어
- 자주 로드되는 스킬: < 200단어 총
- 기타 스킬: < 500단어

기법:
- 모든 플래그 문서화 대신 `--help` 참조
- 반복 대신 다른 스킬 교차 참조
- 일반적인 패턴을 압축하는 예제 사용

### Iron Law 패턴

규율을 강제하는 스킬은 "Iron Law"의 명시적 버전을 포함:

test-driven-development에서:
```
실패하는 테스트 없이 프로덕션 코드 작성 금지
```

verification-before-completion에서:
```
새로운 검증 증거 없이 완료 주장 금지
```

**허점 차단 전략:**
1. 규칙을 명확히 명시
2. 에이전트가 시도할 수 있는 특정 우회 방법 나열 (및 금지)
3. 레드 플래그 체크리스트 생성
4. 반대 주장이 있는 합리화 테이블 구축
5. "문자 vs 정신" 원칙 강조

---

## 6. 스킬 의존성 그래프

```
brainstorming
  └→ [디자인 승인]
      └→ using-git-worktrees
          └→ [격리된 작업 공간 생성]
              └→ writing-plans
                  └→ [작은 단위 작업 생성]
                      ├→ subagent-driven-development (동일 세션)
                      │   ├→ 각 작업: test-driven-development
                      │   ├→ requesting-code-review (작업 간)
                      │   └→ finishing-a-development-branch (모든 작업 후)
                      │
                      └→ executing-plans (별도 세션)
                          ├→ 각 배치: test-driven-development
                          ├→ 사람 리뷰 체크포인트
                          └→ finishing-a-development-branch (모든 배치 후)

모든 작업/버그:
  └→ systematic-debugging (4단계 근본 원인 분석)
      └→ [근본 원인 발견]
          └→ test-driven-development (실패하는 테스트 작성)

모든 완료 주장 전:
  └→ verification-before-completion (검증 명령 실행)

스킬 생성/편집:
  └→ writing-skills (문서를 위한 TDD 적응)
```

---

## 7. 강제 메커니즘

### 규율 강제 스킬

TDD 및 디버깅 같은 스킬은 명시적인 **합리화 방지**를 포함:

**일반적인 변명: "나중에 테스트할게요"**
- 현실: 즉시 통과하는 테스트는 아무것도 증명하지 못함
- 반대: 테스트-후 = "이게 뭐 하는 거지?" vs 테스트-먼저 = "이게 뭘 해야 하지?"
- 강제: 테스트 실패 관찰 요구사항, 합리화 테이블, 레드 플래그 목록

**일반적인 변명: "이제 작동할 거예요"**
- 현실: 검증 명령 실행
- 반대: 자신감 ≠ 증거
- 강제: 게이트 함수, 검증 명령 요구사항, 완료 주장 차단

---

## 8. 규칙 & 베스트 프랙티스

### 네이밍
- 문자, 숫자, 하이픈만 (괄호, 특수 문자 없음)
- 동사 우선, 능동태: `condition-based-waiting` not `async-test-helpers`
- 동명사 (-ing)가 프로세스에 적합: `creating-skills`, `testing-skills`

### 예제
- 하나의 훌륭한 예제가 여러 평범한 예제보다 나음
- 가장 관련성 높은 언어 선택 (테스팅은 TypeScript, 데이터는 Python 등)
- 완전하고 실행 가능, WHY를 잘 주석 처리, 실제 시나리오에서

### 참조
- 중요한 참조 (100+ 줄) → 별도 파일
- 재사용 가능한 도구/코드 → 별도 파일
- 그 외 모든 것 → 인라인

### 교차 참조
```markdown
# ✅ 좋음
**REQUIRED SUB-SKILL:** Use superpowers:test-driven-development

# ❌ 나쁨 - 파일 강제 로드
See @path/to/skill/SKILL.md
```

---

## 요약

Superpowers 시스템은 다음을 수행하는 **종합적인 스킬 프레임워크**입니다:

1. **개발 워크플로우 구조화** - 검증된 관행 중심 (TDD, 체계적 디버깅, 코드 리뷰)
2. **자동 스킬 트리거** - 상황 키워드 및 증상 기반
3. **규율 강제** - 합리화 방지 테이블 및 레드 플래그 체크리스트
4. **서브에이전트 조정 가능** - 상세한 프롬프트 및 리뷰 워크플로우
5. **발견 최적화** - 검색 가능한 설명 및 키워드가 풍부한 내용
6. **코드 품질 유지** - 다단계 리뷰 및 검증 게이트
7. **일반적인 실수 방지** - 명시적 허점 차단 및 "정신 vs 문자" 원칙

시스템은 **프로세스 문서를 코드처럼 취급**하여 (TDD 원칙 적용), 스킬을 프로덕션 코드만큼 테스트 가능하고 신뢰할 수 있게 만듭니다.
