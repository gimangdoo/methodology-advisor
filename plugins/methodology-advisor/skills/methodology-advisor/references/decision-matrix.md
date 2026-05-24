# Decision Matrix

> **Read at phase:** Phase 1 (신호 분석) — 결정적 매칭 룰

5축 신호(`project_type`, `purpose`, `scale`, `timeline`, `constraints`) 조합 → 방법론 추천 top-3. 결정적 룰 우선, 미매칭 신호 = `[matrix-miss]` sigil 박제 후 LLM fallback.

---

## 매칭 알고리즘

1. **Exact match** — 신호 조합이 row와 정확히 일치 → 해당 row 추천 채택
2. **Partial match (≥3 신호 일치)** — top-3 후보 정렬, 일치 수 ↑ 우선
3. **No match** — `[matrix-miss]` sigil + LLM fallback (catalog 적합/부적합 신호 cross-check)
4. **충돌 검출 (top-3 출력 전 필수)** — `methodology-catalog.md §부록 §2.2` 상호 배타 표와 cross-check:
   - 후보 내 충돌 쌍 존재 → `[conflict-detected]` sigil 박제 + 회피 룰 적용 (표 §2.2 "회피 룰" 컬럼 따름)
   - 회피 룰 적용 후 후보 = 유효 조합으로 재정렬
   - 사용자 명시 override 시 `[conflict-override]` sigil 박제 후 유지
5. **시너지 강화 (top-3 출력 시 박제)** — 후보 조합이 `§2.3` 시너지 표에 박제된 검증 조합과 일치 → 시너지 사유 1줄 박제

매칭 시 신호 우선순위: `purpose` > `scale` > `project_type` > `constraints` > `timeline`.

---

## Matrix

| # | project_type | purpose | scale | timeline / constraint | 추천 1순위 | 2순위 | 3순위 |
|---|--------------|---------|-------|----------------------|----------|------|------|
| R1 | greenfield | poc | solo | short | TODO + smoke test + Kanban | Spec-driven 1장 | — |
| R2 | greenfield | poc | solo | medium | TODO + TDD core + Spec-driven | Kanban | GitHub Flow |
| R3 | greenfield | product | solo | short | Spec-driven + TDD core + Trunk-based | Kanban | Test Pyramid |
| R4 | greenfield | product | solo | medium | TDD + Kanban + GitHub Flow | DDD-lite | Modular Monolith |
| R5 | greenfield | product | solo | long | DDD-lite + TDD + Trunk-based | Modular Monolith + Hexagonal | Spec-driven |
| R6 | greenfield | product | small_team | medium | DDD + TDD + GitHub Flow | Modular Monolith + Producer-Reviewer | BDD |
| R7 | greenfield | product | small_team | long + compliance≠none | DDD + BDD + Spec-driven + Hexagonal | Clean Architecture + ATDD | Event Storming workshop |
| R8 | greenfield | product | medium_team | long | DDD + Modular Monolith + Scrum + TDD | Trunk-based + Feature Toggle | BDD |
| R9 | greenfield | product | large_team | long + 분산팀 | Microservices + Contract testing + Release Train | Spec-driven (OpenAPI) + Trunk-based | DDD per service |
| R10 | greenfield | internal_tool | small_team | short | MVC + Snapshot + Kanban | GitHub Flow | Test Pyramid |
| R11 | greenfield | research | solo·small | medium | TODO + Notebook + Spec-driven 산출물 | Property-based (알고리즘 시) | RFC-driven |
| R12 | greenfield | education | solo·small | — | TDD katas + Pair Programming | Spec-driven | RFC-driven |
| R13 | greenfield | public_oss | small·medium | long | RFC-driven + Trunk-based + Test Pyramid | Spec-driven + Contract testing | Semantic versioning + Shape Up |
| R14 | brownfield | product | * | * + `pain_points.skipped_tests > 0` 또는 `test_coverage < 30%` | Characterization test 우선 + 점진 TDD + Feature Toggle | Strangler Fig + Trunk-based | Test Pyramid 재구축 |
| R15 | brownfield | product | * | * + `churn_hotspots ≥ 3` | Hotspot 우선 refactor + TDD + Feature Toggle | Modular Monolith 도입 | Trunk-based 전환 |
| R16 | brownfield | product | * | * + `architecture.module_boundaries ≥ 3` | DDD bounded context 강화 + Producer-Reviewer | Hexagonal | Event Storming workshop (도메인 재발굴) |
| R17 | brownfield | product | * | * + `business_context.domain=fintech 또는 health` + `compliance.regulatory ≠ none` | DDD + BDD + Spec-driven + ATDD + RFC-driven | Hexagonal + Contract testing | Event Storming + Snapshot 회귀 |
| R18 | brownfield | product | * | * + `frontend 중심` (UI heavy) | Component-driven + Snapshot test + Visual regression | MVC·MVP·MVVM 정형화 | Storybook + Feature Toggle |
| R19 | brownfield | product | large_team | * + 마이크로서비스 기 보유 | Contract testing + Release Train + Spec-driven | Trunk-based per service + Feature Toggle | DDD per bounded context |
| R20 | * | * | * | `timeline=short` + `purpose∈{poc,research}` 강력 | TODO + smoke test 최소 | Kanban | — |

`*` = wildcard (모든 값 매칭).

---

## Edge cases / Tie-breaking

**같은 점수 후보 다수 시 tie-breaking 룰:**

1. **사용자 부담 ↓ 우선** — TDD vs BDD 동점 → TDD 우선 (BDD 도구 학습 비용 ↑)
2. **유지비 ↓ 우선** — Microservices vs Modular Monolith 동점 → Modular Monolith 우선 (운영 비용 ↓)
3. **dharness 매핑 명확도 우선** — Phase 4 6패턴과 직결되는 방법론 우선 (DDD ↔ 계층적 위임, TDD ↔ 생성-검증)
4. **catalog 적합 신호 일치 수 우선** — Phase 0 수집 신호와 catalog 적합 신호 일치 ↑ 우선
5. **시너지 표 박제 조합 우선** — 동점 시 `methodology-catalog.md §부록 §2.3` 시너지 표 박제 조합이 미박제 조합보다 우선

---

## `[matrix-miss]` fallback 룰

매칭 row 없음 시 4축 합성 doctrine (`methodology-catalog.md §부록 §2.1`) 적용:

1. 사용자 신호를 catalog 4축에 직접 매핑:
   - 설계 1개 선택 (project 규모·도메인 복잡도 기반, solo·PoC = 생략 가능)
   - 테스트 1개 선택 (test_rigor 신호 기반, 기본 = Test Pyramid)
   - 워크플로우 1개 선택 (team.size + CI 성숙도 기반, 기본 = GitHub Flow)
   - PM 1개 선택 (timeline.horizon + stakeholder 수 기반, 기본 = Kanban)
2. 4축 조합 = top-1 추천
3. 1개 축을 다른 옵션으로 교체 = top-2, top-3
4. **충돌 검출 step 4 실행** — top-3에 §2.2 충돌 쌍 없는지 cross-check
5. 출력에 `[matrix-miss]` sigil 박제 — 사용자가 결정적 룰 부재 인지

**도크트린:** 4축 합성 = matrix-hit 케이스에도 동일 적용 — matrix row 추천도 결국 4축 조합으로 환원 가능. matrix는 단지 검증된 4축 조합 캐시.

---

## Matrix 유지 정책

- 신규 row 박제 트리거: 동일 신호 조합으로 `[matrix-miss]` 발생 ≥3회 누적
- 기존 row 삭제 트리거: ≥6개월간 매칭 0회
- 매트릭스 변경 시 advisor version bump + dharness intent_profile enum 매핑 갱신 검토 (handoff-template §3)

---

## 신호 enum 정의 (Phase 0 수집 형식과 일치)

```yaml
project_type: greenfield | brownfield
purpose: product | poc | research | education | internal_tool | public_oss
scale: solo | small_team | medium_team | large_team
timeline.horizon: short (<3M) | medium (3-12M) | long (>12M)
constraints.compliance.regulatory: [GDPR | HIPAA | PCI-DSS | SOX | SOC2 | ISO27001 | KISA | GxP | none | unknown]
constraints.quality_target: minimal | standard | high | extreme
```

brownfield 보조 신호 (project_profile.md read 시 활용):

```yaml
maturity.test_coverage.line_coverage_percent: float
pain_points.churn_hotspots.length: int
pain_points.skipped_tests.total_count: int
architecture.module_boundaries.length: int
business_context.domain: enum
compliance.regulatory: enum
```
