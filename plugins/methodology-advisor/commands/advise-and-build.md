---
description: "methodology-advisor → 사용자 confirm → dharness factory 자동 인계 wrapper. 방법론 추천 + dharness factory 1회 호출로 일괄 박제."
argument-hint: "<도메인 한 문장>"
---

# /methodology-advisor:advise-and-build

방법론 추천(methodology-advisor)과 하네스 합성(dharness `/harness:harness-new`)을 한 명령으로 체인. 사용자 부담 ↓ — advisor 출력을 수동 복사할 필요 없음.

---

## 워크플로우

### Step 1: methodology-advisor 호출

사용자가 명령에 제공한 `<도메인 한 문장>` (있다면)을 advisor Phase 0의 `core_feature` 초기값으로 박제. 나머지 신호 4축은 grilling-loop로 수집.

**입력 인자:**
- `$ARGUMENTS` = 도메인 한 문장 (선택). 빈 값 시 advisor Phase 0가 처음부터 수집.

**advisor 실행:**
1. methodology-advisor skill의 SKILL.md 본문 그대로 진행
2. Phase 0~3 진행 후 Phase 4 합성 1줄 출력
3. 산출물 = `_workspace/_advisor/{ts}_handoff.md` 박제

### Step 2: 사용자 confirm 게이트 (필수)

advisor Phase 4 출력 후 자동 진행 금지. 명시적 사용자 confirm 게이트 박제:

```
=== advisor 추천 합성 1줄 ===
"<합성 1줄>"

=== confirm 게이트 ===
이 발화로 dharness factory를 자동 호출할까요?
[Y] 진행 — /harness:harness-new <합성 1줄> 자동 실행
[N] 중단 — advisor 산출물만 보존, dharness 호출 안 함
[E] 편집 — 합성 1줄을 사용자가 수정 후 진행
```

**doctrine 정합:** dharness "제안 + 승인" 모델 + Phase 10 자동 적용 금지 doctrine과 정합 ([[project-dharness-handoff-ica-absorption]] L37의 사용자 게이트 doctrine 유지).

사용자 응답 enum:
- `Y` → Step 3 진행
- `N` → 종료. advisor 산출물 경로 출력 + `/harness:harness-new` 수동 호출 안내
- `E` → 사용자 입력 받아 합성 1줄 수정 → 다시 confirm 게이트 (재진입)

### Step 3: dharness factory 자동 호출

confirm 완료 시 `/harness:harness-new <합성 1줄>` 자동 실행. dharness factory가 Phase 0~8 진행.

**중요:** dharness Phase 2 grilling이 advisor 합성 1줄에서 신호를 자동 추출하므로 grilling 횟수 ↓ 기대. 단 `meta.confidence_low` 필드는 여전히 사용자 확인 게이트 발동 가능.

### Step 4: 결과 보고

dharness factory 완료 후:

```
=== 완료 보고 ===
advisor handoff: _workspace/_advisor/{ts}_handoff.md
dharness baseline: _workspace/_baseline/{project,intent}_profile.md
합성된 에이전트: .claude/agents/*.md (N개)
합성된 스킬: .claude/skills/*/SKILL.md (N개)
orchestrator: .claude/skills/{domain}-orchestrator/SKILL.md

=== 다음 단계 ===
[1] 합성된 orchestrator skill description의 트리거 키워드 확인
[2] 트리거 키워드를 발화로 사용해 실 프로젝트 작업 시작
[3] 주 1회 /harness:harness-status로 drift 점검
[4] 방법론·플로우 변경 시 /harness:harness-evolve로 반영
```

---

## 실패·예외 처리

| 상황 | 처리 |
|------|------|
| advisor Phase 0~3 중 사용자 중단 | dharness 호출 안 함. advisor 산출물만 보존 |
| confirm 게이트 `N` | dharness 호출 안 함. advisor 산출물 경로 출력 |
| dharness factory 실행 중 실패 | advisor 산출물 보존 + dharness `_workspace/` 보존 → 사용자 수동 재진입 가능 |
| advisor 출력이 dharness intent_profile enum과 매핑 안 됨 | warn 후 사용자 확인 게이트. 매핑 누락 필드는 dharness Phase 2 grilling이 채움 |

---

## 사용 예시

```
/methodology-advisor:advise-and-build "전자상거래 백엔드 API 신규 구축"

[advisor Phase 0~3 진행]
  scale? → small_team
  timeline? → 6개월 (medium)
  compliance? → none
  ...

[advisor Phase 4 출력]
=== 추천 ===
1. DDD + TDD + GitHub Flow (matrix row R6)
2. Modular Monolith + TDD + Trunk-based
3. BDD + Hexagonal + Spec-driven

[사용자 선택: 1]

[advisor 합성]
"전자상거래 백엔드 API 신규 구축, small team, DDD + TDD + GitHub Flow로 진행. 6개월 timeline."

=== confirm 게이트 ===
[Y]

[dharness /harness:harness-new 자동 호출 → Phase 0~8 진행 → 산출물 박제]

=== 완료 ===
```

---

## 비단축 경로 (수동)

wrapper 명령 안 쓰고 직접 진행 시:

```
1. methodology-advisor skill 자연 발동 ("방법론 추천해줘")
2. advisor Phase 0~4 진행 → 합성 1줄 출력
3. 사용자가 합성 1줄 복사
4. /harness:harness-new <합성 1줄> 수동 호출
```

wrapper와 동일한 결과. wrapper는 단지 3·4 단계 자동화.

---

## 관련 명령

| 명령 | 역할 |
|------|------|
| `methodology-advisor` skill (자연 발동) | advisor 단독 호출 |
| `/harness:harness-new` | dharness factory 단독 호출 |
| `/harness:harness-grill` | dharness Phase 2 grilling 강제 진입 |
| `/harness:harness-evolve` | factory 후 사후 방법론 변경 (advisor 재호출 가능) |
