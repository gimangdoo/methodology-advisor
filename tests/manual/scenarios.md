# B1-runtime e2e dryrun 시나리오 스크립트

박제 시점: 2026-05-24 (v0.3.0)
대상 버전: v0.3.0+
실행 주체: 사용자 (별 Claude Code 세션에서 plugin install + 발화)
결과 박제 위치: `tests/manual/e2e-{scenario}.md` 신설 (본 스크립트는 빈 templatd)

---

## 사전 준비

1. 본 plugin install 확인
   ```
   /plugin marketplace add gimangdoo/methodology-advisor
   /plugin install methodology-advisor@methodology-advisor-marketplace
   ```
2. 별 빈 작업 디렉토리 진입 (테스트 격리)
3. Claude Code 세션 신규 시작

---

## S1 — 자연어 trigger (T1 golden path)

**목적:** SKILL.md description이 사용자 발화를 자동 매칭 → Phase 0~4 정상 진행 → 산출물 3 파일 박제 확인.

### 입력

```
전자상거래 백엔드 신규 구축, small team 5명. 방법론 추천해줘.
```

### 기대 동작

| 단계 | 기대 |
|----|----|
| skill 자동 로드 | description 매칭 ("방법론 추천해줘") |
| Phase 0 grilling | `purpose`·`timeline`·`compliance` 등 누락 신호만 grilling-loop |
| Phase 1 matrix lookup | `R6` exact match (greenfield/product/small_team/medium) |
| Phase 2 top-3 출력 | `DDD+TDD+GitHub Flow` (1순위) + 2·3 후보 + 시너지·충돌 1줄 박제 |
| Phase 3 사용자 게이트 | enum `1/2/3/자유/다시/상세` 박제 |
| Phase 4 합성 1줄 | `"전자상거래 백엔드 신규 구축, small team, DDD + TDD + GitHub Flow로 진행. ..."` |
| 옵션 `[A][B][C]` | 박제 |

### 검증 체크리스트

- [ ] skill 자동 트리거 (수동 호출 없이)
- [ ] grilling-loop 1q-at-a-time 준수 (한 번에 한 질문)
- [ ] 추천 답안 ≥3개 + 자유 입력 옵션 박제
- [ ] matrix row `R6` 박제 (재현성 신호)
- [ ] 산출물 3 파일 생성:
  - `_workspace/_advisor/{ts}_signals.md`
  - `_workspace/_advisor/{ts}_recommendation.md`
  - `_workspace/_advisor/{ts}_handoff.md`
- [ ] mkdir 자동 (Bash `mkdir -p _workspace/_advisor/`)
- [ ] 시너지 표 §2.3 박제 조합 (DDD+TDD, TDD+GitHub Flow) 출력에 박제
- [ ] `[unverified-combo]` sigil 미출현

### 결과 박제

`tests/manual/e2e-trigger.md` 박제. 양식:

```markdown
# S1 결과 — 자연어 trigger

실행 일시: YYYY-MM-DD HH:MM
plugin version: v0.3.0
Claude Code 버전: <semver>

## transcript

<원본 전체 발화·응답 copy>

## 검증 결과

- [x] skill 자동 트리거
- [ ] grilling-loop 1q-at-a-time — **gap**: ...
- ...

## 발견 gap

| # | 위치 | 내용 | severity |
|---|----|----|--------|
| 1 | ... | ... | high/med/low |

## 다음 액션

- gap #1 → fix commit 박제 또는 issue 박제
```

---

## S2 — wrapper command (T1 자동 체이닝)

**목적:** `/methodology-advisor:advise-and-build` command가 advisor → confirm → `/harness:harness-new` 1회 호출로 정상 체이닝.

### 사전 추가 준비

dharness plugin도 install 필요 — wrapper의 `/harness:harness-new` 호출이 실 동작하려면.
미설치 시 wrapper Step 3에서 fail → 시나리오 절반만 검증 가능.

### 입력

```
/methodology-advisor:advise-and-build "환자 데이터 SaaS, HIPAA 준수 필수"
```

### 기대 동작

| 단계 | 기대 |
|----|----|
| command 인식 | `$ARGUMENTS` = `"환자 데이터 SaaS, HIPAA 준수 필수"` |
| Step 1 advisor 호출 | core_feature 박제, Phase 0~3 진행 |
| matrix lookup | `R7` (greenfield/product/small_team/long+compliance≠none) |
| top-3 출력 | `DDD+BDD+Spec-driven+Hexagonal` (1순위) |
| Step 2 confirm 게이트 | `[Y][N][E]` 박제 |
| Step 3 dharness 호출 | confirm 시 `/harness:harness-new <합성 1줄>` 자동 |
| Step 4 결과 보고 | advisor + dharness 산출물 경로 출력 |

### 검증 체크리스트

- [ ] command frontmatter `allowed-tools`에 `SlashCommand` 포함 → 호출 권한 OK
- [ ] `$ARGUMENTS` 한글·특수문자 escaping 정상 (HIPAA 키워드 보존)
- [ ] confirm 게이트 자동 진행 차단 (Y 응답 전까지 dharness 호출 X)
- [ ] `E` 응답 시 합성 1줄 편집 후 재게이트
- [ ] `N` 응답 시 advisor 산출물만 박제, dharness 호출 X
- [ ] dharness 호출 시 합성 1줄 byte-identical 전달
- [ ] dharness factory Phase 0~8 진입 (별 검증)

### 결과 박제

`tests/manual/e2e-wrapper.md` 박제 (S1과 동일 양식).

---

## S3 — dharness-sub 시뮬레이션 (T2 부분)

**목적:** dharness 본체 없이 advisor 단독으로 yaml fragment 출력 + schema 정합 확인. A1·A2 patch 전 사전 검증.

### 사전 준비

dummy `_workspace/_baseline/project_profile.md` 박제 (임의 신호 박제):

```yaml
project_type: greenfield
purpose: product
core_feature: "결제 시스템"
scale: small_team
constraints:
  compliance:
    regulatory: PCI-DSS
  timeline:
    horizon: medium
```

### 입력

```
dharness Phase 2 grilling 위임. intent_profile.quality.test_rigor + workflow.methodology 추천 요청. project_profile.md read 후 진행.
```

### 기대 동작

| 단계 | 기대 |
|----|----|
| 모드 분기 | dharness-sub 모드 진입 ("위임" 키워드 + project_profile 존재) |
| Phase 0 | project_profile.md 자동 read, 미수집 신호만 grilling |
| Phase 1·2·3 | standalone과 동일 |
| Phase 4-sub | yaml fragment 출력 (합성 1줄 X) |

### 기대 yaml fragment 형식

```yaml
# methodology-advisor handoff (dharness-sub mode)
# version: 0.3.0
# matrix_hit: R8 | R6 | matrix-miss

intent_profile_patch:
  quality:
    test_rigor: tdd
  workflow:
    methodology: ["DDD", "TDD", "GitHub Flow"]
    methodology_source: "methodology-advisor v0.3.0"
    methodology_matrix_row: "R6"

meta:
  user_confirmed_fields:
    - quality.test_rigor
    - workflow.methodology
  open_questions: []
```

### 검증 체크리스트

- [ ] dharness-sub 모드 진입 (standalone과 다른 출력 형식)
- [ ] project_profile.md 자동 read (수동 read 호출 없음)
- [ ] 합성 1줄 출력 안 함 (standalone과 분기)
- [ ] yaml `# version: 0.3.0` 박제 (v0.2.0 drift 회피)
- [ ] `methodology_source: "methodology-advisor v0.3.0"` 박제
- [ ] `methodology_matrix_row` 박제 (재현성)
- [ ] 별도 confirm 게이트 미박제 (dharness Phase 2 게이트로 통합 — sub 모드 doctrine)
- [ ] 산출물 `_workspace/_advisor/{ts}_handoff_sub.yaml` 박제

### 결과 박제

`tests/manual/e2e-sub-sim.md` 박제.

---

## S4 — 실패 케이스 4종

### S4.1 `[matrix-miss]` 강제

**입력:** `"게임 서버 24/7 운영, 동접 10만, 방법론 추천해줘"`

**기대:**
- matrix row 미매칭 → `[matrix-miss]` sigil 박제
- 4축 fallback (catalog §부록 §2.1) 적용 — 설계·테스트·워크플로우·PM 4축 LLM 합성
- 출력에 sigil 명시 ("결정적 룰 부재 — LLM 자유 추천")

### S4.2 `[catalog-miss]` 자유 입력

**입력:** Phase 3 사용자 응답 `자유` → `Pair Programming 위주로 가고 싶음`

**기대:**
- `[catalog-miss]` sigil 박제
- LLM이 catalog 형식 4 필드(정의·적합·부적합·dharness 매핑·비용·보완) draft 출력
- 합성 1줄에 "Pair Programming" 박제

### S4.3 Phase 3 `다시` 재진입

**입력:** Phase 3 게이트 응답 `다시`

**기대:**
- Phase 0 재진입
- 기존 박제 신호 reset 또는 확인 게이트
- 산출물 timestamp 갱신 (이전 박제 보존)

### S4.4 사용자 override

**입력:** Phase 3 게이트 후 추천 top-1 무시 + `자유` → `"BDD만 쓸래"` 단일 박제

**기대:**
- 4축 doctrine 단일 박제 게이트 발동: "설계·테스트·워크플로우·PM 중 3축 박제 생략 의도 맞나요?" (catalog §2.1 룰 3)
- 사용자 재확인 시 합성 1줄 박제
- `meta.advisor_recommended_but_overridden` 박제 (학습 신호)

### 검증 체크리스트 (4종 공통)

- [ ] 해당 sigil 정확히 박제
- [ ] 사용자 응답 enum 처리 정상
- [ ] 산출물 박제 (signals·recommendation·handoff)
- [ ] override 경우 `meta.advisor_recommended_but_overridden` 필드 박제

### 결과 박제

`tests/manual/e2e-failures.md` 박제 (4 케이스 합본).

---

## 종합 박제

4 시나리오 (S1·S2·S3·S4) 완주 후:

1. `tests/manual/e2e-{trigger,wrapper,sub-sim,failures}.md` 4 파일 박제
2. 발견 gap 합본 → `tests/manual/B1-runtime-summary-{ts}.md` 박제
3. severity high gap → 즉시 fix commit + patch bump (v0.3.x)
4. severity med·low gap → ROADMAP B1 갱신 + v0.4.0 후보 박제
5. README "Usage" 섹션에 S1 transcript 발췌 1줄 박제 (현 의사 코드 → 실 출력)

---

## 비실행 우회 (가능 한도)

본 plugin install 또는 dharness install 불가 시:
- S1 발화를 advisor SKILL.md를 직접 read한 LLM에게 던져 phase 흐름 모의 가능
- 단 실 산출물 박제·trigger 자동 매칭은 검증 불가 — silent breakage 위험 잔존
- 우회는 fallback, 실 install 후 dryrun이 정식 검증
