# Handoff Template

> **Read at phase:** Phase 4 (standalone) + Phase 4-sub (dharness-sub) — dharness 인계 발화·yaml 합성 단계

advisor 추천 결과 → dharness factory 입력으로 변환하는 합성 패턴. 두 모드(standalone / dharness-sub) 출력 형식이 다름.

---

## 목차

1. [standalone 모드 합성 1줄 패턴](#1-standalone-모드-합성-1줄-패턴)
2. [dharness-sub 모드 yaml fragment 패턴](#2-dharness-sub-모드-yaml-fragment-패턴)
3. [dharness intent_profile enum 매핑 표](#3-dharness-intent_profile-enum-매핑-표)
4. [T1·T2 시퀀스 다이어그램](#4-t1t2-시퀀스-다이어그램)
5. [실패·예외 처리](#5-실패예외-처리)

---

## 1. standalone 모드 합성 1줄 패턴

**패턴:**
```
{core_feature 한 문장}, {scale}, {확정 방법론 조합}로 진행. {부가 제약 1줄}.
```

**구성 요소:**
- `core_feature`: Phase 0에서 수집한 한 문장 (예: "전자상거래 백엔드 API")
- `scale`: solo / small_team / medium_team / large_team
- `확정 방법론 조합`: 1~3개 결합 (예: "DDD + TDD + Trunk-based")
- `부가 제약 1줄`: timeline, compliance, 도메인 특이사항 (선택)

### 예시 5개

| # | 입력 신호 | 합성 1줄 |
|---|---------|---------|
| E1 | greenfield, product, solo, medium, 추천=TDD + Kanban + GitHub Flow | `"전자상거래 백엔드 API 신규 구축, 솔로 개발, TDD + Kanban + GitHub Flow로 진행. 3개월 내 MVP 목표."` |
| E2 | greenfield, product, small_team, long, compliance=HIPAA, 추천=DDD + BDD + Spec-driven + Hexagonal | `"환자 데이터 관리 SaaS, small team 5명, DDD + BDD + Spec-driven + Hexagonal Architecture로 진행. HIPAA 준수 필수, 12개월 timeline."` |
| E3 | brownfield, product, small_team, churn_hotspots=5, test_coverage=18%, 추천=Characterization + 점진 TDD + Feature Toggle | `"기존 Next.js 커머스 사이트에 추천 시스템 추가, small team 4명, characterization test로 기존 로직 박제 후 점진 TDD + Feature Toggle로 진행. checkout 영역 churn hotspot 우선."` |
| E4 | greenfield, public_oss, small_team, long, 추천=RFC-driven + Trunk-based + Test Pyramid | `"CLI 도구 OSS 신규 개발, small team 3명, RFC-driven + Trunk-based + Test Pyramid로 진행. semantic versioning 박제."` |
| E5 | greenfield, research, solo, medium, 추천=TODO + Notebook + Spec-driven 산출물 | `"머신러닝 데이터 파이프라인 연구 프로토타입, 솔로, TODO 기반 + Jupyter Notebook + Spec-driven 산출물로 진행. 6개월 PoC 후 product 전환 검토."` |

### 합성 룰

1. **방법론 조합 최대 4개** — 4축 doctrine (`methodology-catalog.md §부록 §2.1`): 설계 0~1 + 테스트 0~1 + 워크플로우 0~1 + PM 0~1. 4개 초과 시 핵심 1~3개만 박제, 나머지는 evolve 단계.
2. **축당 주 방법론 1개** — 동축 주 방법론 2개 = 모순 (Kanban + Scrum, GitFlow + Trunk-based, Microservices + Modular Monolith 등 — §2.2 충돌 표 cross-check). 보조 패턴 (Hexagonal·Clean·Event Storming·Characterization 등) 추가 박제 가능 — 단 §2.3 시너지 표 박제 조합에 한함.
3. **부가 제약 = 도메인 한 문장 추가 정보** — timeline, compliance, 핫스팟, stakeholder 등
4. **dharness Phase 2 자동 추론 가능한 신호 우선 박제** — `quality.test_rigor`, `constraints.compliance.regulatory`, `architecture.deployment_target` enum과 직접 매핑되는 키워드 사용
5. **4축 합성 doctrine 격상 (v0.2.0)** — 단일 박제 (1개만) 시 사용자 게이트 발동: "설계·테스트·워크플로우·PM 중 N개 축 박제 생략 의도 맞나요?" — solo·PoC만 1~2개 박제 default 허용, 그 외 ≥3축 박제 권장
6. **시너지 표 박제 조합 우선** — §2.3 검증 조합과 일치 시 합성 1줄에 우선 박제 (반대는 `[unverified-combo]` sigil 박제)

### 사용자 출력 형식

```
=== dharness 인계 발화 ===
"<합성 1줄>"

=== 다음 단계 옵션 ===
[A] 위 발화 복사 → /harness:harness-new <발화> 호출
[B] 자동 인계: /methodology-advisor:advise-and-build (advisor → confirm → dharness 자동 체인)
[C] 박제만 — dharness 호출 없이 종료 (advisor 산출물만 활용)
```

---

## 2. dharness-sub 모드 yaml fragment 패턴

dharness Phase 2 grilling 중 advisor가 sub-skill로 호출됐을 때. 합성 1줄 X, dharness intent_profile에 merge 가능한 yaml fragment 출력.

**패턴:**

```yaml
# methodology-advisor handoff (dharness-sub mode)
# version: 0.1.0
# matrix_hit: R4 | matrix-miss

intent_profile_patch:
  quality:
    test_rigor: <enum>                # none | smoke | unit | integration | tdd
  workflow:
    methodology: [<방법론1>, <방법론2>, ...]   # advisor 추천 조합
    methodology_source: "methodology-advisor v0.1.0"
    methodology_matrix_row: "R4"      # matrix row id 또는 "matrix-miss"

meta:
  user_confirmed_fields:
    - quality.test_rigor
    - workflow.methodology
  open_questions: []                  # advisor가 해결한 질문 박제 (있다면 새 질문 추가)
```

### dharness Phase 2 통합 룰

1. dharness Phase 2가 advisor 호출 시 진행 중 `intent_profile.md` 경로 전달
2. advisor가 Phase 0~3 진행 → Phase 4-sub에서 yaml fragment 생성
3. dharness가 fragment를 `intent_profile.md` frontmatter에 merge:
   - 기존 필드 있으면 advisor 추천 vs 기존 값 비교 → 사용자 게이트
   - 기존 필드 없으면 advisor 값 박제
4. `user_confirmed_fields`에 advisor 처리 필드 박제 — Phase 5 이후 신뢰도 ↑ 표시
5. dharness Phase 2 종료 → Phase 3 진행

### `workflow.methodology` enum 박제 필요 (dharness patch)

현 dharness `intent-profile-schema.md`에 `workflow` 섹션은 있으나 `methodology` 필드 없음. advisor 박제 후 dharness doctrine-only patch 필요:

```yaml
workflow:
  ci_cd: none | lint | test | full_pipeline
  methodology: [string]               # ← 신규 enum (catalog 24개 + 자유 입력)
  methodology_source: string          # ← 신규 (출처 추적)
  methodology_matrix_row: string      # ← 신규 (재현성)
```

dharness patch는 advisor 박제 완료 후 별도 세션에서 수행 (현 결정).

---

## 3. dharness intent_profile enum 매핑 표

advisor 추천 결과를 dharness intent_profile 필드 enum으로 변환하는 매핑 룰.

### 3.1 `quality.test_rigor` 매핑

| advisor 추천 방법론 | `quality.test_rigor` enum |
|------------------|--------------------------|
| TDD | `tdd` |
| BDD + TDD | `tdd` |
| ATDD | `integration` |
| Test Pyramid (default) | `unit` |
| Snapshot only | `smoke` |
| Property-based | `tdd` |
| Contract testing | `integration` |
| 테스트 명시 없음 | `unit` (default) |

### 3.2 `architecture.deployment_target` 보조 매핑

| advisor 추천 방법론 | 보조 매핑 |
|------------------|---------|
| Microservices | `architecture.deployment_target=web` 또는 `api` + 다중 서비스 신호 |
| Modular Monolith | 단일 deployment target |
| MVC·MVP·MVVM | `architecture.deployment_target=web` 또는 `mobile` |
| Hexagonal | deployment target 무관, 내부 구조 |
| Clean Architecture | deployment target 무관, 내부 구조 |

### 3.3 dharness Phase 4 패턴 6개 매핑

| advisor 추천 | Phase 4 패턴 |
|-----------|------------|
| TDD + BDD (생성·검증 명확) | **생성-검증** (Producer-Reviewer) |
| DDD + bounded context 다수 | **계층적 위임** (Hierarchical Delegation) |
| Microservices + Contract testing | **전문가 풀** (Expert Pool) |
| Pipeline 처리 (data, build) | **파이프라인** (Pipeline) |
| 병렬 분석·통합 | **팬아웃·팬인** (Fan-out / Fan-in) |
| 다중 phase 조율 | **감독자** (Supervisor) |

### 3.4 reviewer 에이전트 강제 룰 박제

| advisor 추천 | reviewer 룰 |
|-----------|------------|
| TDD | test 부재 시 fail |
| BDD | Gherkin spec 부재 시 fail |
| ATDD | acceptance criteria 부재 시 fail |
| Spec-driven | OpenAPI/SDL 산출물 부재 시 fail |
| DDD | bounded context 경계 침범 시 fail (writes: 박제 + harness-validate write_path_overlap 회로) |
| Hexagonal · Clean | 의존 방향 위반 시 fail |
| Property-based | 알고리즘 코드에 property test 부재 시 warn |

### 3.5 워크플로우 박제

| advisor 추천 | orchestrator·CI 룰 |
|-----------|------------------|
| Trunk-based | single-branch 강제 + CI 게이트 |
| GitHub Flow | short-lived feature branch + PR 머지 = 배포 |
| GitFlow | 다층 branch 정책 (advisor 추천 빈도 ↓) |
| Release Train | 고정 주기 release window |
| Feature Toggle | toggle 도구 박제 + 점진 rollout 룰 |

### 3.6 PM 방법론 = dharness 매핑 외부

Kanban / Scrum / Shape Up / RFC-driven / Event Storming은 dharness 안 박제 X. handoff 1줄에는 명시되나 dharness intent_profile에는 매핑 안 됨. 사용자 측 외부 도구 (Linear/GitHub Issues/Notion 등)에서 박제.

---

## 4. T1·T2 시퀀스 다이어그램

### 4.1 T1 (standalone, pre-harness-new)

```
사용자
  ↓ "방법론 추천해줘" 발화 또는 /methodology-advisor:advise-and-build
methodology-advisor skill
  ↓ Phase 0 신호 수집 (grilling-loop)
  ↓ Phase 1 매트릭스 분석
  ↓ Phase 2 top-3 + tradeoff 출력
  ↓ Phase 3 사용자 게이트 (확정)
  ↓ Phase 4 합성 1줄 출력 + 옵션 [A][B][C]
사용자
  ↓ [A] 수동 복사 → /harness:harness-new <합성 1줄>
  ↓ [B] 자동 인계 — wrapper command가 dharness 호출
  ↓ [C] 종료
dharness factory
  ↓ Phase 0~8 진행 (Phase 2 grilling 시 advisor 합성 1줄에서 추출된 신호 활용)
  ↓ 산출물 박제
```

### 4.2 T2 (dharness-sub, mid Phase 2)

```
사용자
  ↓ /harness:harness-new <도메인 한 문장> (방법론 명시 없음)
dharness factory
  ↓ Phase 0~1 진행
  ↓ Phase 2 grilling 진입
  ↓ 필수 5필드 grilling 진행
  ↓ quality.test_rigor 또는 workflow.methodology 필드 grilling
  ↓ 사용자 "모름 — 추천해줘" 응답
  ↓ dharness가 methodology-advisor sub-skill 호출 (dharness-sub 모드)
methodology-advisor (sub 모드)
  ↓ Phase 0 (이미 수집된 project_profile.md + 진행 중 intent_profile.md 자동 read)
  ↓ Phase 1 매트릭스 분석
  ↓ Phase 2 top-3 출력 (사용자 게이트는 dharness Phase 2 게이트와 통합)
  ↓ Phase 3 사용자 확정
  ↓ Phase 4-sub yaml fragment 출력
dharness factory
  ↓ yaml fragment → intent_profile.md frontmatter merge
  ↓ Phase 2 종료 → Phase 3 진행
```

---

## 5. 실패·예외 처리

| 상황 | 처리 |
|------|------|
| `[matrix-miss]` 발생 | LLM fallback + sigil 박제. 사용자에게 결정적 룰 부재 명시 |
| 사용자 `다시` 응답 (Phase 3 게이트) | Phase 0 재진입, 신호 재수집 |
| 사용자 `자유` 입력 (catalog 외 방법론) | `[catalog-miss]` sigil 박제 + LLM이 catalog 형식으로 적합/부적합 신호 추론 출력 |
| dharness-sub 호출인데 `project_profile.md` 부재 | greenfield 가정 + Phase 0 신호 사용자 수집 |
| dharness-sub 호출 결과 yaml fragment merge 충돌 (기존 값 ≠ advisor 값) | dharness Phase 2 게이트에 두 값 출력 + 사용자 선택 |
| advisor 추천 vs 사용자 선택 불일치 | 사용자 선택 우선. advisor 추천은 `meta.advisor_recommended_but_overridden` 박제 (학습 신호) |
| dharness `workflow.methodology` 필드 미박제 상태 (dharness patch 전) | yaml fragment의 `workflow.methodology` 필드는 dharness가 무시 + warn. `quality.test_rigor`만 merge 성공 |

---

## 부록: handoff 산출물 보존 정책

| 모드 | 보존 경로 |
|------|---------|
| standalone | `_workspace/_advisor/{ts}_handoff.md` (사용자 프로젝트 루트) |
| dharness-sub | `_workspace/_advisor/{ts}_handoff_sub.yaml` + dharness `_workspace/_baseline/intent_profile.md` merge 이력 |

보존 사유: Phase 9 evolve 시 advisor 추천 vs 실 운영 결과 비교 가능. Matrix row R# 박제로 추천 정확도 누적 분석 가능.
