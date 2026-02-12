# Superpowers 분석

## 메타 정보

| 항목 | 내용 |
|------|------|
| 저장소 | [obra/superpowers](https://github.com/obra/superpowers) |
| 주요 언어 | Shell, Markdown, JavaScript |
| 라이선스 | MIT — 완전 자유 사용, 수정, 재배포 가능 |
| Stars | 50K+ |
| 분석일 | 2026-02-12 |
| 성격 | AI 코딩 에이전트용 워크플로우/스킬 프레임워크 |

## 핵심 인사이트

1. **메타-프레임워크 설계** — 코드 라이브러리가 아닌, AI 에이전트의 행동 규범을 Markdown 문서(스킬)로 체계화. 의존성 제로로 모든 환경에서 즉시 동작
2. **2단계 검증 게이트** — Spec Compliance → Code Quality 순서의 서브에이전트 리뷰. 요구사항 충족을 먼저 보장한 후에만 코드 품질을 검토하는 절대 규칙
3. **Claude Search Optimization (CSO)** — 스킬 description에 "무엇을 하는가"가 아닌 "언제 사용할 것인가"만 기술. Claude가 요약만 따르고 전체 스킬을 읽지 않는 문제를 해결
4. **스킬도 TDD로 작성** — "문서를 코드처럼 테스트"하는 혁신적 접근. 스킬 없이 에이전트 실행(RED) → 스킬 작성(GREEN) → 우회 방법 차단(REFACTOR)
5. **Fresh Context Per Task** — 서브에이전트마다 새 컨텍스트를 부여하여 이전 작업의 피로/혼란/오염을 원천 차단

## 문서 구성

| 문서 | 내용 |
|------|------|
| [overview.md](./overview.md) | 프로젝트 철학, 차별점, 기술 스택 선택 이유 |
| [core-logic.md](./core-logic.md) | 실행 흐름, 핵심 알고리즘/패턴 상세 분석 |
| [architecture.md](./architecture.md) | 스킬 시스템 구조, 플랫폼 통합, 설계 패턴 |
