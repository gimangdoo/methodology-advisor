# dharness Patch Spec — Track A1 + A2

박제 시점: 2026-05-24 (methodology-advisor v0.3.0 release 후)
대상 repo: `gimangdoo/dharness` (별 repo, 본 spec은 본 plugin 측 박제)
대상 dharness 버전: 별도 확인 필요 (PR draft 시 head 기준)

본 spec = dharness repo 측 별 세션에서 참조. 본 plugin 작업 아님 — 본 plugin은 advisor v0.3.0까지 박제 완료, dharness 측 schema·grilling 분기 박제 대기.

---

## 배경

methodology-advisor v0.1.0~v0.3.0이 dharness intent_profile에 yaml fragment merge를 시도하나 dharness `intent-profile-schema.md`에 `workflow.methodology` 필드가 미박제 → dharness가 silent ignore + warning. 결과:

- T2 (dharness-sub) 모드 = 본 advisor 핵심 분기 절반 — 사실상 무력화
- 사용자는 T1 (standalone) 모드의 합성 1줄 수동 복사 패턴만 사용 가능
- README "Two modes" 광고 vs 실 동작 불일치

본 patch가 적용되면 dharness Phase 2 grilling 중 "추천해줘" 응답 → 본 advisor sub 모드 호출 → yaml fragment merge 자동 → 사용자 부담 ↓.

---

## A1 — `intent-profile-schema.md` 필드 추가

### 변경 위치

`dharness/references/intent-profile-schema.md` (정확 경로는 dharness 측 확인)

### 추가 필드

`workflow` 섹션에 3 필드 박제:

```yaml
workflow:
  ci_cd: none | lint | test | full_pipeline       # 기존
  methodology: [string]                            # 신규
  methodology_source: string                       # 신규
  methodology_matrix_row: string                   # 신규
```

### 필드 의미

| 필드 | 타입 | enum / pattern | 박제 의미 |
|----|----|--------------|--------|
| `methodology` | array of string | catalog 24개 + 자유 입력 | advisor 추천 4축 조합 (최대 4개) |
| `methodology_source` | string | `"methodology-advisor v<semver>"` 또는 `"manual"` | 출처 추적 |
| `methodology_matrix_row` | string | `"R1"~"R20"` 또는 `"matrix-miss"` 또는 `"manual"` | 재현성 박제 |

### `methodology` enum 24개

본 advisor `methodology-catalog.md` 박제 항목 (v0.3.0 기준):

**설계 (6):** DDD, Modular Monolith, Microservices, Hexagonal, Clean, MVC·MVP·MVVM
**테스트 (7):** TDD, BDD, ATDD, Test Pyramid, Snapshot, Property-based, Contract testing
**워크플로우 (5):** Trunk-based, GitFlow, GitHub Flow, Release Train, Feature Toggle
**PM (6):** Kanban, Scrum, Shape Up, Spec-driven, RFC-driven, Event Storming

자유 입력 = `[catalog-miss]` sigil + LLM fallback (catalog 외 방법론). dharness validate가 enum 위반 시 warn (block 아님 — `[catalog-miss]` 박제 합법).

### 검증 회로

dharness `harness-validate` 회로에 추가:

```python
def validate_workflow_methodology(intent_profile):
    methodology = intent_profile.get("workflow", {}).get("methodology", [])
    catalog_enum = load_catalog_enum()  # advisor catalog read 또는 dharness 측 mirror
    for m in methodology:
        if m not in catalog_enum:
            warn(f"methodology '{m}' not in catalog — [catalog-miss] sigil expected")
    return True  # block 아님
```

선택: `methodology_matrix_row` 검증 (advisor matrix R1~R20 또는 sentinel).

### backward compat

기존 dharness 사용자 schema 영향:
- `workflow.methodology` 미박제 = 기존 동작 유지 (필드 optional)
- 기존 `workflow.ci_cd`만 박제된 case → 호환

migration 불필요.

### dharness 측 CHANGELOG 박제 예시

```markdown
### Added
- `intent_profile.workflow.methodology` enum 필드 (methodology-advisor v0.3.0+ 호환)
- `intent_profile.workflow.methodology_source` 출처 추적 필드
- `intent_profile.workflow.methodology_matrix_row` 재현성 박제 필드
- `harness-validate` 회로에 methodology enum 검증 추가
```

---

## A2 — Phase 2 grilling sub-skill 호출 분기

### 변경 위치

`dharness/references/grilling-loop.md` 또는 dharness Phase 2 진입점 (정확 경로는 dharness 측 확인)

### 분기 로직

dharness Phase 2 grilling 중 다음 필드 grilling 시점에 분기 박제:
- `quality.test_rigor`
- `workflow.methodology` (A1 적용 후)

분기 트리거 키워드:
- 사용자 응답에 `"추천"`, `"모름"`, `"뭐가 좋은지 모르겠음"`, `"advisor"`, `"방법론 추천"` 박제
- 또는 dharness가 enum 응답 외 자유 텍스트 받음 + 추천 키워드 감지

분기 동작:
1. methodology-advisor sub-skill 호출 (스킬 호출 메커니즘은 dharness 측 결정 — `Skill` tool 또는 inline 호출)
2. sub-skill에 dharness Phase 2 진행 중인 `_workspace/_baseline/project_profile.md` + `_workspace/_baseline/intent_profile.md` 경로 전달
3. advisor 단독 confirm 게이트 X (dharness Phase 2 본 게이트로 통합)
4. advisor가 yaml fragment 출력 시 dharness가 read → frontmatter merge
5. merge 결과를 dharness Phase 2 본 게이트로 사용자에게 confirm

### merge 룰

advisor yaml fragment의 `intent_profile_patch`를 dharness `intent_profile.md` frontmatter에 merge:

```yaml
# advisor 출력
intent_profile_patch:
  quality:
    test_rigor: tdd
  workflow:
    methodology: ["DDD", "TDD", "GitHub Flow"]
    methodology_source: "methodology-advisor v0.3.0"
    methodology_matrix_row: "R6"
```

merge 시 충돌 처리:
- 기존 필드 미박제 → advisor 값 박제
- 기존 필드 박제 + advisor 값과 일치 → 무동작
- 기존 필드 박제 + advisor 값과 불일치 → dharness 본 게이트에 두 값 출력 + 사용자 선택

### `meta.user_confirmed_fields` 박제

advisor가 처리한 필드는 dharness `meta.user_confirmed_fields`에 박제:
```yaml
meta:
  user_confirmed_fields:
    - quality.test_rigor       # advisor 박제
    - workflow.methodology     # advisor 박제
```

Phase 5 이후 dharness가 이 박제를 신뢰도 신호로 활용 (재 grilling 회피).

### Phase 2 doctrine 정합

dharness Phase 2 = "5필수 + 5선택" 박제 doctrine. methodology가 어느 그룹인지 결정:
- 권장: **5필수**의 `quality.test_rigor`가 advisor 호출 트리거 (이미 5필수)
- `workflow.methodology`는 신규 enum → 5선택 또는 신설 그룹. dharness doctrine 측 결정 필요.

---

## A3 — 본 advisor repo 측 후속 (A1·A2 완료 후)

본 spec은 dharness 측 patch 후 본 repo 진행. 별 commit, 본 spec과 동일 세션 X.

### 작업

1. `README.md` "Compatibility" 섹션 갱신
   - 현 "not yet" 문구 제거
   - "requires dharness vX.Y+" 박제 (A1·A2 patch가 박제한 dharness 버전)
2. `handoff-template.md` §2 §workflow.methodology enum 박제 필요 박스 제거
3. `CHANGELOG.md` `[0.3.x]` 또는 `[0.4.0]` 박제:
   - `Changed`: dharness vX.Y 호환 박제 완료
4. `ROADMAP.md` A1·A2·A3 `[STATUS: done]`
5. `PLAN.md` Track A 박제 (retrospective)

### 검증

- B1.3 dharness-sub 시나리오 (`tests/manual/scenarios.md §S3`) 재실행
- yaml fragment merge 성공 + `intent_profile.md` frontmatter 갱신 확인

---

## 검증·롤백

### dharness 측 검증

1. A1 적용 후 `harness-validate` 회로 통과 — `workflow.methodology` 박제된 sample intent_profile 통과
2. A2 적용 후 dharness `harness-new` 시뮬레이션 — Phase 2 grilling 중 "추천해줘" 응답 → advisor sub 호출 → yaml merge 성공
3. 기존 사용자 intent_profile (methodology 필드 없는 case) → backward compat 유지

### 롤백

A1·A2 모두 backward compatible. 롤백 시:
- A1: schema 필드 제거 → 기존 dharness validate 동작 복귀, advisor sub yaml은 다시 silent ignore (이전 상태로 복귀)
- A2: 분기 로직 제거 → Phase 2 grilling이 사용자 enum 응답만 처리 (advisor 호출 X)

본 advisor repo 측은 dharness 롤백 시 영향 없음 — handoff-template.md의 "dharness patch 미완" 박스만 다시 명시.

---

## 의존 관계

```
A1 (schema 필드)          ──┐
                            ├──→ A2 (Phase 2 분기) ──→ B1.3 시나리오 검증 ──→ A3 (본 repo 정리)
methodology-advisor v0.3.0 ──┘                          │
                                                        └──→ dharness vX.Y release
```

- A1 + 본 advisor v0.3.0 = A2 진입 조건
- A2 완료 = B1.3 실 검증 가능
- B1.3 통과 = A3 진입 조건

---

## 박제 위치 (본 repo 측)

| 파일 | 박제 |
|------|----|
| `docs/dharness-patch-spec.md` | 본 spec |
| `ROADMAP.md` Track A | A1·A2·A3 status 추적 |
| `PLAN.md` Track A | 상세 실행 계획 |
| `plugins/methodology-advisor/skills/methodology-advisor/references/handoff-template.md` §2 | dharness patch 미완 박스 (A3 후 제거) |
| `README.md` "Compatibility" | 호환 상태 (A3 후 갱신) |
| `CHANGELOG.md` `Known limitations` | dharness patch 미진행 박제 (A3 후 제거) |

---

## 다음 액션 (사용자)

1. dharness repo 별 클론 + 별 Claude Code 세션 진입
2. 본 spec read + dharness 측 schema·grilling 진입점 위치 확인
3. A1 patch 박제 → dharness 측 commit + version bump
4. A2 patch 박제 → dharness 측 commit
5. dharness release tag 박제 + push
6. 본 advisor repo 측 A3 박제 (별 commit) + v0.3.x patch 또는 v0.4.0 release
