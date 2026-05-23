---
name: methodology-advisor
description: "프로젝트 구조·목적·기능·규모를 입력받아 적합한 소프트웨어 개발 방법론(TDD/DDD/BDD/Spec-driven/Kanban/Trunk-based 등)을 추천. 사용자가 방법론을 모를 때 1q-at-a-time + 추천 답안 패턴으로 신호 수집. 산출물 = dharness factory에 그대로 전달 가능한 합성 1줄. 트리거: '방법론 추천', '어떤 방식으로 진행', '뭘 쓸지 모르겠음', '프로젝트 시작 어떻게', 'dharness 시작 전', 'TDD/DDD 골라줘', '개발 방식 선택', '프로세스 정해줘'. should-NOT 트리거: 이미 방법론 명시된 발화, 코드 리뷰, 디버깅, 단일 라이브러리 추천."
---

# Methodology Advisor

프로젝트 신호(구조·목적·기능·규모·제약)를 분석해 적합한 개발 방법론을 추천하는 advisory 스킬. dharness 메타 스킬 팩토리의 **선행 단계**로 호출 → 추천 결과를 dharness 입력 발화에 박제 → factory가 합성한 에이전트 팀이 박제된 방법론으로 작동.

dharness 자체에 박제하지 않은 이유 = dharness doctrine "팩토리" 단일 책임 유지 + PM 영역(to-issues/to-prd) 흡수 제외 결정 정합.

---

## 워크플로우 (Phase 0~4)

### Phase 0: 입력 수집

**실행 모드 분기 (필수, 첫 단계):**

| 모드 | 트리거 | 입력 소스 |
|------|------|---------|
| **standalone** (T1) | 사용자 자연 발화, `/methodology-advisor:advise-and-build` 명시 호출 | 사용자 발화 + grilling |
| **dharness-sub** (T2) | dharness Phase 2 grilling 중 `quality.test_rigor`·`workflow.methodology` 위임 호출 | `_workspace/_baseline/project_profile.md` 자동 read + dharness Phase 2 진행 중 `intent_profile.md` read + 미수집 신호만 grilling |

dharness-sub 모드 진입 신호:
- 호출 시 부모 컨텍스트에 `_workspace/_baseline/project_profile.md` 또는 진행 중 `intent_profile.md` 존재
- 사용자 발화에 "dharness에서 위임", "Phase 2 grilling 위임", "intent_profile.quality.test_rigor 추천 요청" 키워드

**dharness-sub 모드 산출물 차이:**
- Phase 4 합성 1줄 X — 대신 dharness intent_profile 필드 enum 직접 박제 (Phase 4-sub)
- `_workspace/_advisor/{ts}_handoff.md` 대신 dharness intent_profile에 직접 merge 가능한 yaml fragment 출력
- 사용자 confirm 게이트는 dharness Phase 2의 grilling confirm과 통합 (별도 게이트 없음)

**5축 신호 수집:** 사용자 발화에 이미 있는 신호는 스킵, 누락된 신호는 grilling-loop 패턴(1q-at-a-time + 추천 답안 ≥3개 + 선택지 enum + 자유 입력 옵션)으로 수집.

| 축 | 필수 | enum 또는 자유 입력 |
|----|------|-------------------|
| `project_type` | ✓ | greenfield / brownfield |
| `purpose` | ✓ | product / poc / research / education / internal_tool / public_oss |
| `core_feature` | ✓ | 한 문장 (자유 입력) |
| `scale` | ✓ | solo / small_team (2-5) / medium_team (6-20) / large_team (20+) |
| `constraints` | 선택 | timeline (short<3M / medium 3-12M / long>12M), compliance, quality_target |

**brownfield 시 보조 신호 (있으면 정확도 ↑):**
- `_workspace/_baseline/project_profile.md` 존재 여부 read → `churn_hotspots`, `test_coverage`, `module_boundaries`, `business_context.domain`, `compliance.regulatory` 활용
- dharness factory 후 advisor 재호출 시나리오

### Phase 1: 신호 분석

수집된 5축 신호를 `references/decision-matrix.md`의 row와 매칭한다. 매칭 정책:

1. **결정적 매치 우선** — 신호 조합이 matrix row와 정확히 일치 시 해당 row의 추천 채택
2. **부분 매치** — 일치하는 신호 ≥3개 row를 후보로 수집 → top-3 정렬
3. **매치 0** — `[matrix-miss]` sigil 박제 + LLM 자유 추천 fallback (적합 신호·부적합 신호를 catalog와 cross-check)

### Phase 2: top-3 후보 + tradeoff 출력

각 후보에 대해 다음 5개 항목을 1줄씩 출력:

| 항목 | 형식 |
|------|------|
| 방법론 조합 | 1~3개 결합 (예: "DDD + TDD + Trunk-based") |
| 한 줄 정의 | catalog에서 인용 |
| 적합 사유 | 사용자 신호 → 방법론 적합도 매핑 |
| 비용 | 학습 곡선 / 초기 속도 / 장기 속도 |
| dharness 매핑 | intent_profile 필드 + Phase 4 패턴 |

### Phase 3: 사용자 게이트

사용자 응답 enum:
- `1`/`2`/`3` — top-3 중 선택
- `자유` — 자유 입력 (catalog 외 방법론 가능)
- `다시` — Phase 0 재수집
- `상세` — 선택 후보의 catalog 항목 전문 출력

확정 전 종료 금지 — `meta.confirmed: true` 박제 후에만 Phase 4 진행.

### Phase 4: dharness 인계 발화 합성 (standalone 모드)

`references/handoff-template.md` 패턴으로 합성 1줄 생성:

```
{core_feature 한 문장}, {scale}, {확정 방법론}로 진행. {부가 제약 1줄}.
```

**출력 형식 (사용자에게 제시):**

```
=== dharness 인계 발화 ===
"<합성 1줄>"

=== 다음 단계 옵션 ===
[A] 위 발화를 복사 → /harness:harness-new <발화> 호출
[B] 자동 인계: /methodology-advisor:advise-and-build 다시 호출 (advisor → confirm → dharness 자동 체인)
[C] 박제만 — dharness 호출 없이 진행 (수동 사용 위함)
```

### Phase 4-sub: dharness intent_profile 필드 직접 박제 (dharness-sub 모드)

standalone 모드의 Phase 4 대신 실행. dharness Phase 2 grilling에서 호출됐으므로 합성 1줄이 아니라 intent_profile 필드 yaml fragment를 출력한다.

**출력 형식:**

```yaml
# methodology-advisor handoff (dharness-sub mode)
intent_profile_patch:
  quality:
    test_rigor: tdd                    # advisor 추천 enum
  workflow:
    methodology: ["DDD", "TDD", "Trunk-based"]   # 추천 조합
    methodology_source: "methodology-advisor v0.1.0 (matrix-hit / matrix-miss)"
meta:
  open_questions: []                   # advisor가 해결한 질문 박제
  user_confirmed_fields:
    - quality.test_rigor
    - workflow.methodology
```

dharness Phase 2가 이 yaml을 read → `intent_profile.md` frontmatter에 merge → 사용자 confirm은 dharness Phase 2 본 게이트로 통합.

**핵심 제약:** `workflow.methodology` 필드는 현 dharness intent_profile schema에 없음 — dharness doctrine-only patch로 신규 enum 박제 필요 (advisor 박제 후 별도 패치 세션에서 수행).

---

## 산출물

| 경로 | 내용 |
|------|------|
| `_workspace/_advisor/{ts}_signals.md` | Phase 0 수집 신호 5축 yaml |
| `_workspace/_advisor/{ts}_recommendation.md` | Phase 2 top-3 + Phase 3 사용자 확정 |
| `_workspace/_advisor/{ts}_handoff.md` | Phase 4 합성 1줄 + 옵션 A/B/C |

**워크스페이스 디렉토리 미존재 시:** 사용자 프로젝트 루트에 `_workspace/_advisor/` 생성. dharness factory가 같은 `_workspace/` 사용 → 공존 안전.

---

## dharness 정합 매핑

advisor 출력이 dharness intent_profile enum과 1:1 매핑되어야 handoff 정확도 보장:

| advisor 출력 | dharness 매핑 |
|------------|--------------|
| TDD 박제 | `intent_profile.quality.test_rigor=tdd` + Phase 4 "생성-검증" 패턴 |
| DDD 박제 | Phase 4 "계층적 위임" 패턴 + 에이전트 `writes:` 필드로 bounded context 박제 |
| BDD 박제 | reviewer 에이전트에 spec-first 강제 룰 추가 |
| Spec-driven | Phase 5 에이전트 정의에 spec 산출물 박제 의무화 |
| Trunk-based | orchestrator-template Phase 4에 single-branch 강제 룰 |
| Kanban | 외부 PM 도구 (Linear/GitHub Issues) — dharness 매핑 없음, 사용자 측 박제 |

상세 매핑 표는 `references/handoff-template.md` §3 참조.

---

## 트리거 should/should-NOT 분리

**should-trigger (자동 발동 OK):**
1. "방법론 추천해줘"
2. "어떤 방식으로 진행하면 좋을까"
3. "프로젝트 시작 전 뭘 정해야 하지"
4. "TDD가 좋을지 BDD가 좋을지"
5. "dharness 호출 전에 정하고 싶어"
6. "프로세스 가이드 필요"
7. "개발 방법 선택"
8. "뭘 쓸지 모르겠음 — 추천"
9. dharness Phase 2 grilling 위임 호출 (dharness-sub 모드, T2 진입)
10. "intent_profile.quality.test_rigor 추천 요청"

**should-NOT-trigger (발동 거부):**
1. "TDD로 진행해줘" → 이미 명시. dharness 직접 호출.
2. "이 코드 리뷰해줘" → 코드 리뷰 영역.
3. "버그 고쳐줘" → 디버깅 영역.
4. "lodash 쓸까 ramda 쓸까" → 라이브러리 선택, 방법론 X.
5. "Slack 대신 Discord?" → 도구 선택, 방법론 X.
6. "dharness 진행 상황 확인" → harness-status 영역.
7. "지금 작성한 코드가 DDD 맞아?" → 코드 검토 영역.
8. "TDD가 뭐야" → 단순 개념 질문, advisor 진입 불필요.

---

## 비용·한계

| 항목 | 내용 |
|------|------|
| 학습 곡선 | 사용자 측 0 — advisor가 추천 + tradeoff 1줄로 제시 |
| 토큰 비용 | catalog + matrix LLM 1회 read (~3K tokens) + Phase 0~4 대화 (~5K tokens) |
| 추천 정확도 한계 | matrix row 부재 신호 조합 = `[matrix-miss]` LLM fallback (재현성 ↓) |
| dharness 의존도 | handoff 매핑은 dharness intent_profile schema에 종속 — schema 변경 시 advisor handoff 갱신 필요 |

---

## 참고

- `references/methodology-catalog.md` — 방법론 24개 정의·신호·dharness 매핑·비용·보완
- `references/decision-matrix.md` — 신호 → 추천 결정 룰 매트릭스
- `references/handoff-template.md` — dharness 인계 발화 합성 패턴 + intent_profile enum 매핑 표
- dharness `references/grilling-loop.md` — 1q-at-a-time + 추천 답안 패턴 원본 (Phase 0에서 차용)
- dharness `references/intent-profile-schema.md` — handoff 매핑 대상 schema
