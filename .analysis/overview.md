# Superpowers — Overview

## 프로젝트 철학

Superpowers는 AI 코딩 에이전트(Claude Code, Codex, OpenCode)가 **전문적인 소프트웨어 개발 관행**을 체계적으로 따르도록 만드는 **메타-프레임워크**다. 핵심은 코드가 아닌 **문서화된 프로세스(스킬)**로 에이전트 행동을 규범화하는 것이다.

**4대 원칙 (`README.md:113-119`):**

| 원칙 | 의미 |
|------|------|
| Test-Driven Development | 항상 테스트를 먼저 작성 |
| Systematic over Ad-hoc | 추측이 아닌 증거 기반 절차 |
| Complexity Reduction | 단순성을 최우선 목표로 |
| Evidence over Claims | 증명 없는 주장 금지 |

**커뮤니티 가치:**
- 명확한 우선순위: **안정성 → UX → 성능**
- 스킬 자체도 TDD로 검증 — "문서를 코드처럼 테스트"
- 플랫폼 독립성 — Claude Code, Codex, OpenCode 모두 지원

## 해결하는 문제

AI 에이전트는 코딩 능력이 있어도 다음과 같은 문제를 빈번하게 일으킨다:
- 설계 검증 없이 즉시 구현 시작
- TDD 규율 없는 임의 코드 작성
- 요구사항 과도/과소 구축 (YAGNI 위반)
- 코드 리뷰/품질 검증 단계 생략
- 계획 없는 부분 최적화로 인한 편차

**기존 대안 대비 차별점:**

| 관점 | 단순 프롬프트/가이드 | Superpowers |
|------|---------------------|-------------|
| 구조 | 일회성 지침 | 재사용 가능한 15개 스킬 시스템 |
| 트리거 | 수동 호출 | 세션 시작 시 자동 주입 + 상황별 자동 호출 |
| 검증 | 없음 | 2단계 서브에이전트 리뷰 게이트 |
| 규율 강제 | 권장 수준 | Iron Law + 합리화 방지 테이블 + 레드 플래그 체크리스트 |
| 테스트 | 스킬 자체 미검증 | 스킬도 TDD로 검증 (RED→GREEN→REFACTOR) |
| 플랫폼 | 특정 도구 전용 | Claude Code, Codex, OpenCode 통합 |

## 기술 스택과 선택 이유

### 의도적 의존성 제로

Superpowers는 **패키지 의존성이 전혀 없다**. 이는 의도적 설계 결정이다.

| 구성요소 | 기술 | 선택 이유 | 트레이드오프 |
|----------|------|-----------|------------|
| 스킬 문서 | Markdown + YAML frontmatter | 에이전트가 자연스럽게 읽고 해석 가능 | 형식 검증 불가 |
| 세션 훅 | Shell Script (Bash) | 호스트 OS와 무관하게 실행 | Windows 호환성 이슈 (`.cmd` wrapper 필요) |
| 스킬 로더 | JavaScript (Node.js) | 스킬 발견/네임스페이싱/업데이트 확인 | 최소 기능만 구현 (~200 LOC) |
| 메타데이터 | YAML frontmatter | 간단한 추출, 표준화된 포맷 | name, description만 지원 |
| 병렬 개발 | Git Worktrees | 컨텍스트 격리, 표준 git 기능 | 프로젝트 복잡도 증가 |
| 세션 분석 | JSONL transcripts + Python | 토큰 사용량/비용 추적 | 별도 분석 도구 필요 |

### 기술적 최적화 사례

**Bash 문자열 이스케이프 O(n) 최적화** (`hooks/session-start.sh:23-31`):

```bash
# O(n²) 방식 (이전) — 문자 하나씩 처리
for ((i=0; i<${#input}; i++)); do
    char="${input:$i:1}"  # 매번 전체 문자열 복사
done

# O(n) 방식 (현재) — bash parameter substitution
escape_for_json() {
    local s="$1"
    s="${s//\\/\\\\}"    # C-level 단일 pass
    s="${s//\"/\\\"}"    # C-level 단일 pass
    s="${s//$'\n'/\\n}"  # C-level 단일 pass
    s="${s//$'\r'/\\r}"
    s="${s//$'\t'/\\t}"
    printf '%s' "$s"
}
```

macOS에서 7배, Windows Git Bash에서 극적으로 개선 (60초 → 즉시).

---

## 배울 점

1. **"문서를 코드처럼 테스트"하는 접근**: 스킬 없이 에이전트 실행(RED) → 위반 사항 문서화 → 스킬 작성(GREEN) → 우회 방법 차단(REFACTOR). 프로세스 문서의 신뢰성을 TDD로 보장
2. **Iron Law + 합리화 방지 테이블**: 에이전트가 규칙을 우회하려는 시도(변명)를 사전에 예측하고 반박하는 체계. "Too simple to test" → "Simple code breaks. Test takes 30 seconds."
3. **CSO(Claude Search Optimization)**: description에 워크플로우를 요약하면 에이전트가 요약만 따르고 전체 문서를 안 읽는 문제 발견 → 트리거 조건만 기술하는 패턴
4. **의존성 제로 설계**: 모든 것을 Markdown + Shell로 구현하여 어떤 환경에서든 즉시 동작. 기능을 지침과 프로세스에만 의존

## 적용 아이디어

| Superpowers 패턴 | EDR AI 적용 |
|------------------|-------------|
| 2단계 검증 게이트 | AI 분석 결과를 Spec Compliance(보안 정책 준수) → Quality(정확도/신뢰도) 순서로 자동 검증 |
| Iron Law 패턴 | AI가 보안 이벤트 분석 시 "증거 없이 결론 내리기 금지" 같은 절대 규칙 설정 |
| 스킬 시스템 | EDR 분석 패턴(멀웨어 탐지, 이상 행위 분석, 포렌식 등)을 재사용 가능한 AI 스킬로 문서화 |
| Fresh Context Per Task | 대량 보안 이벤트 분석 시 이벤트별 독립 컨텍스트로 교차 오염 방지 |
| 합리화 방지 테이블 | AI가 "정상 트래픽"으로 오분류하려는 시도를 사전 차단하는 체크리스트 |
