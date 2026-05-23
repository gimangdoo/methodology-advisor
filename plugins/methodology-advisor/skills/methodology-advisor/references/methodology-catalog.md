# Methodology Catalog

> **Read at phase:** Phase 1 (신호 분석) + Phase 2 (top-3 tradeoff 출력) + Phase 3 (사용자 `상세` 응답 시)

방법론 24개를 4축으로 분류 박제. 각 항목 = 정의 + 적합/부적합 신호 + dharness 매핑 + 비용 + 보완 방법론.

---

## 목차

1. [설계 방법론 (6)](#1-설계-방법론)
2. [테스트 방법론 (7)](#2-테스트-방법론)
3. [워크플로우 (5)](#3-워크플로우)
4. [PM·진행 방식 (6)](#4-pm진행-방식)

---

## 1. 설계 방법론

### 1.1 DDD (Domain-Driven Design)

- **정의:** 도메인 모델을 중심으로 bounded context 단위로 코드·팀을 분리하는 설계 방법론.
- **적합 신호:** 다도메인 (≥3), 비즈니스 복잡도 ↑, team.size ≥ small, timeline.horizon ≥ medium, compliance 존재.
- **부적합 신호:** solo + PoC, 단일 도메인, ui-중심 prototype.
- **dharness 매핑:** Phase 4 "계층적 위임" 패턴 + 에이전트 `writes:` 필드로 bounded context별 파일 경계 박제 + `harness-validate write_path_overlap` 회로로 침범 결정적 차단.
- **비용:** 학습 곡선 ↑ (Evans 책 또는 Vaughn Vernon Red Book), 초기 속도 ↓, 장기 속도 ↑ (변경 격리).
- **보완:** TDD (도메인 모델 안정성), Event Storming (도메인 발굴), Hexagonal (의존성 격리).

### 1.2 Modular Monolith

- **정의:** 단일 배포 단위 안에서 모듈 경계를 명확히 박제. 마이크로서비스의 운영 비용 회피 + DDD의 경계 이점.
- **적합 신호:** small team, 단일 배포 환경, 도메인 분리 욕구, 향후 microservice 이전 옵션 보존.
- **부적합 신호:** 진짜 분산 요구 (트래픽·팀 분리), 마이크로서비스 운영 인프라 보유.
- **dharness 매핑:** Phase 4 "감독자" 패턴 + 에이전트 1명당 모듈 1개 + 공유 인프라 에이전트 1.
- **비용:** 학습 곡선 중, 초기 속도 ↔, 장기 속도 ↑.
- **보완:** DDD lite, TDD, Trunk-based.

### 1.3 Microservices

- **정의:** 도메인별 독립 배포 가능 서비스로 분리.
- **적합 신호:** large team (20+), 진짜 분산 트래픽, 팀별 release 독립성 필수.
- **부적합 신호:** solo, small team, MVP, 운영 인프라 부족.
- **dharness 매핑:** Phase 4 "전문가 풀" + 서비스당 에이전트 1 + API contract 검증 에이전트 1.
- **비용:** 학습·운영 비용 ↑↑ (orchestration, observability, distributed tracing), 초기·장기 속도 모두 인프라 성숙도에 종속.
- **보완:** Contract testing, Spec-driven (OpenAPI), Release Train.

### 1.4 Hexagonal Architecture (Ports & Adapters)

- **정의:** 도메인 로직과 외부 시스템(DB·API·UI)을 port/adapter로 격리.
- **적합 신호:** 외부 시스템 의존 ↑, 테스트 가능성 우선, 향후 인프라 교체 가능성.
- **부적합 신호:** 단순 CRUD, 외부 의존 적은 도메인.
- **dharness 매핑:** Phase 4 "파이프라인" + adapter 에이전트 + core domain 에이전트 분리 + reviewer가 의존 방향 검증.
- **비용:** 학습 곡선 중, 초기 속도 ↓ (boilerplate), 장기 속도 ↑.
- **보완:** DDD, TDD, Dependency Injection.

### 1.5 Clean Architecture

- **정의:** 의존성 방향을 안→밖 단방향으로 강제. Entities → Use Cases → Adapters → Frameworks.
- **적합 신호:** 장기 유지보수, 프레임워크 교체 가능성, 핵심 비즈니스 로직 보호 필요.
- **부적합 신호:** PoC, 시간 제약 강함, 단순 서비스.
- **dharness 매핑:** Phase 4 "계층적 위임" + layer당 에이전트 1 + 의존 방향 검증 에이전트.
- **비용:** 학습 곡선 ↑, 초기 속도 ↓↓, 장기 속도 ↑↑.
- **보완:** Hexagonal, DDD, TDD.

### 1.6 MVC / MVP / MVVM

- **정의:** UI 중심 패턴. Model-View 분리 + 컨트롤러·프리젠터·뷰모델 매개.
- **적합 신호:** UI 중심 프로젝트, 표준 프레임워크 (Rails/Django/Spring MVC, Android, SwiftUI).
- **부적합 신호:** 백엔드 전용, 도메인 복잡 ↑.
- **dharness 매핑:** Phase 4 "파이프라인" + View / Controller / Model 에이전트 분리.
- **비용:** 학습 곡선 ↓, 초기 속도 ↑, 장기 속도 ↔ (도메인 커지면 한계).
- **보완:** Component-driven (Storybook), Snapshot test.

---

## 2. 테스트 방법론

### 2.1 TDD (Test-Driven Development)

- **정의:** Red → Green → Refactor. 실패 테스트 작성 → 통과 코드 작성 → 리팩토링.
- **적합 신호:** test_rigor 강제, refactor 빈도 ↑, 핵심 도메인 로직, solo·small team.
- **부적합 신호:** PoC, UI prototype, 학습 단계 사용자.
- **dharness 매핑:** `intent_profile.quality.test_rigor=tdd` + Phase 4 "생성-검증" 패턴 + reviewer 에이전트가 test 부재 fail.
- **비용:** 학습 곡선 중, 초기 속도 ↓, 장기 속도 ↑↑.
- **보완:** BDD (스펙 명확화), Trunk-based (CI 빠른 피드백), Property-based (엣지 케이스).

### 2.2 BDD (Behavior-Driven Development)

- **정의:** Given-When-Then 시나리오로 행위 명세. 비즈니스 언어로 테스트 작성.
- **적합 신호:** 비기술 stakeholder 참여, 복잡한 비즈니스 룰, compliance 추적 필수.
- **부적합 신호:** 순수 기술 라이브러리, 비즈니스 stakeholder 없음.
- **dharness 매핑:** Phase 5 에이전트 정의에 Gherkin spec 산출물 박제 의무 + reviewer가 spec-first 강제.
- **비용:** 학습 곡선 ↑ (Cucumber 류 도구), 초기 속도 ↓, 장기 속도 ↑ (의도 추적성).
- **보완:** TDD, ATDD, Spec-driven.

### 2.3 ATDD (Acceptance Test-Driven Development)

- **정의:** 인수 조건을 테스트로 박제 후 구현. BDD의 인수 테스트 변형.
- **적합 신호:** 명확한 인수 조건 보유, 고객 검수 단계 박제.
- **부적합 신호:** 인수 조건 모호, 빠른 iteration.
- **dharness 매핑:** Phase 5 reviewer 에이전트에 acceptance criteria 검증 의무.
- **비용:** 학습 곡선 중, 초기 속도 ↓, 장기 속도 ↑.
- **보완:** BDD, Spec-driven.

### 2.4 Test Pyramid

- **정의:** Unit(많이) > Integration(중간) > E2E(적게) 비율로 테스트 박제.
- **적합 신호:** 모든 프로젝트 default. 특히 medium·large team.
- **부적합 신호:** 거의 없음 (PoC는 unit만 박제 가능).
- **dharness 매핑:** Phase 5 QA 에이전트 가이드 default + `quality.test_rigor=integration` 이상.
- **비용:** 학습 곡선 ↓, 초기 속도 ↔, 장기 속도 ↑.
- **보완:** TDD, Contract testing, Property-based.

### 2.5 Snapshot Testing

- **정의:** 출력 결과를 박제 → 변화 검출.
- **적합 신호:** UI 컴포넌트, JSON API 응답 안정성, 회귀 검출.
- **부적합 신호:** 의도된 변경 빈번, 비결정적 출력 (timestamp, random).
- **dharness 매핑:** Phase 5 UI 에이전트 또는 API 에이전트 test 도구 박제.
- **비용:** 학습 곡선 ↓, 초기 속도 ↑, 장기 속도 ↔ (snapshot bloat 위험).
- **보완:** Visual regression, Component-driven, MVC/MVP/MVVM.

### 2.6 Property-Based Testing

- **정의:** 입력 도메인 명세 → 무작위 입력 생성 → 불변식(invariant) 검증.
- **적합 신호:** 순수 함수, 데이터 변환, 알고리즘, fintech·encryption.
- **부적합 신호:** 부작용 많음, UI, integration 영역.
- **dharness 매핑:** Phase 5 core domain 에이전트에 Hypothesis/fast-check/QuickCheck 도구 박제.
- **비용:** 학습 곡선 ↑, 초기 속도 ↓, 장기 속도 ↑↑ (엣지 케이스 자동 발견).
- **보완:** TDD, Test Pyramid.

### 2.7 Contract Testing

- **정의:** 서비스 간 API contract를 양측이 박제 + 양방향 검증.
- **적합 신호:** Microservices, 외부 API 통합, large team 분산.
- **부적합 신호:** Monolith, 단일 팀.
- **dharness 매핑:** Phase 4 "전문가 풀" + 서비스별 에이전트 + Pact-style contract 에이전트 1.
- **비용:** 학습 곡선 중, 초기 속도 ↓, 장기 속도 ↑↑ (통합 회귀 검출).
- **보완:** Spec-driven (OpenAPI), Microservices, Trunk-based.

---

## 3. 워크플로우

### 3.1 Trunk-Based Development

- **정의:** 모든 변경을 단일 trunk에 직접 merge. feature branch 단명 (≤ 1 day).
- **적합 신호:** CI 인프라 성숙, test 커버리지 ↑, team 신뢰도 ↑.
- **부적합 신호:** test 부재, 긴 리뷰 주기, large 미숙 team.
- **dharness 매핑:** orchestrator-template Phase 4에 single-branch 강제 룰 + CI 검증 게이트.
- **비용:** 학습 곡선 ↓, 초기 속도 ↑ (대형 merge 회피), 장기 속도 ↑↑.
- **보완:** Feature Toggle, TDD, Contract testing.

### 3.2 GitFlow

- **정의:** develop/release/hotfix/feature branch 다층 구조.
- **적합 신호:** 릴리즈 주기 명확, 다중 버전 동시 유지, 엔터프라이즈.
- **부적합 신호:** 연속 배포, small team, 빠른 iteration.
- **dharness 매핑:** orchestrator에 branch 정책 박제 + 에이전트에 PR 룰 박제. 단 dharness 자체는 branch 무지.
- **비용:** 학습 곡선 중, 초기 속도 ↔, 장기 속도 ↓ (merge 충돌·long-lived branch 부담).
- **보완:** Release Train.

### 3.3 GitHub Flow

- **정의:** main + short-lived feature branch. PR 머지 = 배포.
- **적합 신호:** SaaS·연속 배포, small·medium team, OSS.
- **부적합 신호:** 다중 버전 유지, 엔터프라이즈 릴리즈.
- **dharness 매핑:** orchestrator에 PR 게이트 박제 + reviewer 에이전트가 PR 검증.
- **비용:** 학습 곡선 ↓, 초기 속도 ↑, 장기 속도 ↑.
- **보완:** Trunk-based의 lite 버전.

### 3.4 Release Train

- **정의:** 고정 주기 (예: 2주) release. feature 준비 안 됐으면 다음 train 대기.
- **적합 신호:** large team, 다중 팀 동기화 필수, 엔터프라이즈.
- **부적합 신호:** small team, 빠른 iteration.
- **dharness 매핑:** orchestrator에 release window 박제. 다중 팀 시 supervisor 에이전트가 train 조율.
- **비용:** 학습 곡선 중, 초기 속도 ↓, 장기 속도 ↑ (예측성).
- **보완:** Feature Toggle, Contract testing.

### 3.5 Feature Toggle (Feature Flag)

- **정의:** 코드 배포와 기능 활성화를 분리. toggle로 trunk-based + 점진 rollout 가능.
- **적합 신호:** 연속 배포, A/B 테스트, 점진 rollout, dark launch.
- **부적합 신호:** 단순 도구, toggle 관리 인프라 부재.
- **dharness 매핑:** Phase 5 에이전트에 toggle 도구 (LaunchDarkly·Unleash·Flagsmith 또는 자체) 박제.
- **비용:** 학습 곡선 중, 초기 속도 ↓ (인프라), 장기 속도 ↑↑.
- **보완:** Trunk-based, Strangler Fig (legacy 교체).

---

## 4. PM·진행 방식

### 4.1 Kanban

- **정의:** WIP 제한 + 시각화 보드. continuous flow.
- **적합 신호:** support·유지보수, 우선순위 변동 ↑, solo·small team.
- **부적합 신호:** 고정 마일스톤, 강한 timeline 제약.
- **dharness 매핑:** dharness 외부 PM 도구 (Linear/GitHub Projects). dharness 안 박제 X.
- **비용:** 학습 곡선 ↓, 초기 속도 ↑, 장기 속도 ↔.
- **보완:** Trunk-based, Feature Toggle.

### 4.2 Scrum

- **정의:** 2~4주 sprint + 명시적 role (PO, SM, Dev). sprint planning/review/retrospective.
- **적합 신호:** medium·large team, 정형 process, 명확한 stakeholder.
- **부적합 신호:** solo, ad-hoc, OSS.
- **dharness 매핑:** dharness 외부 PM. 단 Phase 4 "감독자" 패턴이 SM 역할 대리 가능.
- **비용:** 학습 곡선 ↑ (CSM 인증 등), 초기 속도 ↓, 장기 속도 ↑ (예측성).
- **보완:** Release Train, TDD.

### 4.3 Shape Up (37signals)

- **정의:** 6주 cycle + 2주 cooldown. shaping → betting → building.
- **적합 신호:** small product team, 자율성 ↑, 명확한 appetite.
- **부적합 신호:** large team, 외부 stakeholder 다수.
- **dharness 매핑:** dharness 외부 PM. 단 도메인 한 문장이 Shape Up "pitch"와 정합.
- **비용:** 학습 곡선 중 (Shape Up 책 1독), 초기 속도 ↔, 장기 속도 ↑.
- **보완:** Trunk-based, Feature Toggle.

### 4.4 Spec-Driven Development

- **정의:** OpenAPI/JSON Schema/GraphQL SDL 등 명세 우선 작성 → 코드 생성·검증.
- **적합 신호:** API 중심, 다중 소비자, contract 안정성 필수, GitHub Spec Kit 활용.
- **부적합 신호:** UI 중심, 순수 라이브러리, 단일 소비자.
- **dharness 매핑:** Phase 5 에이전트 정의에 spec 산출물 박제 의무 + reviewer가 spec-code 정합 검증.
- **비용:** 학습 곡선 중, 초기 속도 ↓, 장기 속도 ↑↑.
- **보완:** Contract testing, BDD, ATDD.

### 4.5 RFC-Driven Development

- **정의:** 큰 변경 전 RFC 문서 박제 → 토론 → 합의 → 구현.
- **적합 신호:** OSS, 분산 team, 큰 아키텍처 결정, 의사결정 추적 필수.
- **부적합 신호:** solo, ad-hoc, 시간 제약 강함.
- **dharness 매핑:** dharness 외부 산출물. 단 `_workspace/_rfc/{ts}_{title}.md` 박제 룰 추가 가능.
- **비용:** 학습 곡선 ↓, 초기 속도 ↓, 장기 속도 ↑.
- **보완:** ADR (Architecture Decision Records), Spec-driven.

### 4.6 Event Storming Workshop

- **정의:** 도메인 발굴 workshop. domain events → commands → aggregates → bounded contexts 순으로 sticky notes 박제.
- **적합 신호:** 복잡 도메인, DDD 도입, 다도메인 분리 필요, 도메인 전문가 참여 가능.
- **부적합 신호:** 단순 도메인, 도메인 전문가 부재, 비대면 분산 team (변형 필요).
- **dharness 매핑:** dharness factory 진입 **전** workshop 결과 박제 → 도메인 한 문장 정확도 ↑. 또는 Phase 3 도메인 분석 보강 input.
- **비용:** 학습 곡선 중, 초기 속도 ↓ (workshop 1~2일), 장기 속도 ↑↑ (도메인 모델 정확도).
- **보완:** DDD, BDD.

---

## 부록: 자유 입력 fallback

위 24개 외 방법론(예: Pair Programming, Mob Programming, Cathedral & Bazaar, Lean Startup, Outside-In TDD, etc.) → catalog 미박제 항목으로 표시 + LLM 자유 추천 + `[catalog-miss]` sigil 박제. 반복 추천 누적 시 catalog 신규 항목 박제 후보.
