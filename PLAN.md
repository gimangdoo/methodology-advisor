# Execution Plan

`methodology-advisor` 잔존 작업 상세 실행 계획. ROADMAP.md = "무엇·왜" / PLAN.md = "어떻게·언제·수락 조건".

박제 시점: 2026-05-24 (v0.2.0 직후).
다음 release target: **v0.3.0** (B1 + B2 완료 시).

각 항목 6 필드 박제:
- **Goal** — 끝났을 때 달라지는 것
- **Steps** — 순서·granularity 박제
- **Acceptance** — 측정 가능한 완료 조건
- **Deliverables** — 박제될 파일·commit·tag
- **Effort** — 작업 시간 추정
- **Deps / Risks** — 선행 의존 + 발생 가능 위험

---

## 우선순위 시퀀스

```
1. B1-static  (정합성 점검, runtime 0)    → 30분, 즉시
2. B1-runtime (e2e dryrun 4 시나리오)     → 1~2시간, 사용자 환경 필요
3. B2         (시너지 표 +7~10셀)         → 30분, B1 완료 후
   └─ v0.3.0 release (tag + push)
4. A1·A2      (dharness schema patch)    → 별 repo, 별 세션
5. A3         (본 repo 후속 정합)         → 10분, A1·A2 완료 후
6. C2         (telemetry 박제)            → v0.4.0 후보
7. C1·C3·C4   (재현성·자동 evolve)        → v0.4+ 후보
```

---

## Track B1 — 운영 검증

### B1-static. 정합성 점검 (runtime 0)

**Goal:** 박제된 코드의 self-consistency 확인. runtime 전 발견 가능한 silent bug 제거.

**Steps:**
1. SKILL.md ↔ references cross-link 점검
   - SKILL.md가 인용한 §·표가 references에 실제 존재하는지
   - references frontmatter `name`이 SKILL.md 인용과 일치하는지
2. `advise-and-build.md` command 점검
   - allowed-tools 적정성 (`SlashCommand`, `Read`, `Write` 필요)
   - `$ARGUMENTS` placeholder escaping
   - `/harness:harness-new` 호출 인자 박제 패턴
3. plugin manifest 일관성
   - `plugin.json` name·version ↔ marketplace.json version
   - skill·command 경로가 manifest와 일치
4. SKILL.md description 트리거 분석
   - should-trigger 10개 키워드 ↔ description 10 키워드 일치
   - should-NOT-trigger 8개 ↔ description "should-NOT" 박제 일치
5. 산출물 경로 정합
   - SKILL.md §산출물 = `_workspace/_advisor/{ts}_*.md` 3 파일
   - handoff-template.md §부록 보존 정책과 일치

**Acceptance:**
- gap report `tests/static/B1-static-{ts}.md` 박제
- gap 0건 또는 발견 gap별 fix commit 박제

**Deliverables:**
- `tests/static/B1-static-{ts}.md` (1 파일)
- 발견 gap fix commit (있다면)

**Effort:** 30분.

**Deps / Risks:**
- 의존: 없음
- 위험: static 점검은 runtime breakage 모두 잡지 못함 — false confidence 주의

---

### B1-runtime. e2e dryrun

**Goal:** 실 사용자 발화로 plugin 1회 도는지 확인. silent breakage 발견 → B2 진행 정당화.

**Steps:**
1. **B1.1 자연어 trigger (T1 golden path)**
   - 새 Claude Code 세션 박제 (테스트 격리)
   - 입력: `"전자상거래 백엔드 신규 구축, small team 5명. 방법론 추천해줘"`
   - 기대 matrix row: `R6` (greenfield/product/small_team/medium)
   - 검증: Phase 0~4 진행 + 산출물 3 파일 + matrix row 박제
2. **B1.2 wrapper command (T1 자동 체이닝)**
   - 입력: `/methodology-advisor:advise-and-build "환자 데이터 SaaS, HIPAA 준수 필수"`
   - 기대 matrix row: `R7`
   - 검증: advisor 완주 → confirm 게이트 → `/harness:harness-new` 자동 호출
3. **B1.3 dharness-sub 시뮬레이션 (T2 부분)**
   - dharness 본체 실행 X (A1·A2 미완)
   - advisor 단독 호출 + 입력에 "dharness Phase 2 위임" 명시 → yaml fragment 출력 확인
4. **B1.4 실패 케이스 4종**
   - `[matrix-miss]`: "게임 서버 24/7 운영, 동접 100K"
   - `[catalog-miss]`: 자유 입력 "Pair Programming"
   - Phase 3 `다시`: top-3 후 재진입
   - 사용자 override: top-3 무시 + 직접 명시

**Acceptance:**
- 4 시나리오 transcript 박제 (`tests/manual/e2e-*.md`)
- 발견 버그 ≥1건 시 issue 또는 fix commit
- README "Usage" 섹션에 실 출력 예시 1개 박제 (현 의사 코드 → 실 transcript 1줄)
- B1-static 통과 후만 진행

**Deliverables:**
- `tests/manual/e2e-trigger.md` (B1.1)
- `tests/manual/e2e-wrapper.md` (B1.2)
- `tests/manual/e2e-sub-sim.md` (B1.3)
- `tests/manual/e2e-failures.md` (B1.4)
- README 업데이트
- (선택) bug fix commit

**Effort:** 1~2시간 (사용자 환경에서).

**Deps / Risks:**
- 의존: plugin이 사용자 Claude Code에 install되어 있어야 함
- 위험: SKILL.md description weak match → 자동 트리거 실패 (수동 호출로 우회)
- 위험: `/harness:harness-new` 미설치 시 B1.2 절반만 검증
- 위험: 한글·따옴표 escaping 이슈 — wrapper command `$ARGUMENTS` 박제 확인

---

## Track B2 — 시너지 표 확장 (v0.3.0)

**Goal:** `methodology-catalog.md §부록 §2.3` 셀 15 → ≥22. matrix R# 빈출 조합 `[unverified-combo]` sigil 박제 빈도 ↓.

**Steps:**
1. B1-runtime 결과로 후보 목록 갱신
   - 실 transcript에서 `[unverified-combo]` 출현 조합 우선 박제
   - ROADMAP B2 10 후보 + 추가 발견 후보 통합
2. §2.3 표에 7~10셀 박제
   - 형식 = 기존 박제 패턴 유지 (조합 / 시너지 사유 / 적합 시나리오 3열)
   - tone = catalog 기존 항목과 일치 (단문 박제, 한국어)
3. CHANGELOG `[0.3.0]` 박제
   - `Added`: §2.3 신규 셀 N개 (목록)
   - `Changed`: 박제 사유 1줄
4. plugin manifest version bump
   - `plugins/methodology-advisor/.claude-plugin/plugin.json` 0.2.0 → 0.3.0
   - `.claude-plugin/marketplace.json` 0.2.0 → 0.3.0
   - SKILL.md `methodology_source: "methodology-advisor v0.1.0"` 박제 위치 → v0.3.0 갱신
   - handoff-template.md `# version: 0.1.0` 박제 위치 → v0.3.0
5. git tag `v0.3.0`
6. push origin main + tag (사용자 직접)

**Acceptance:**
- §2.3 셀 ≥22
- B1-runtime 결과 R6·R7·R10 조합에서 `[unverified-combo]` sigil 미출현
- v0.3.0 tag 박제 + origin push
- CHANGELOG `[0.3.0]` 박제

**Deliverables:**
- `methodology-catalog.md` (수정)
- `CHANGELOG.md` (수정)
- `plugin.json` + `marketplace.json` + `SKILL.md` + `handoff-template.md` version bump
- commit `feat(catalog): synergy table +N cells — v0.3.0`
- tag `v0.3.0`

**Effort:** 30분 (B1 결과 반영 포함).

**Deps / Risks:**
- 의존: B1-runtime 완료 권장 (실 후보 발견 위함)
- 위험: 시너지 사유 박제가 LLM 추정 — 실 운영 검증 없음. 향후 telemetry로 검증 필요
- 위험: 사용자 게이트 없이 catalog 직접 patch → user review 필수

---

## Track A — dharness 측 schema patch

### A1. `intent-profile-schema.md` 필드 추가

**Goal:** dharness `intent_profile`에 `workflow.methodology` enum 박제. advisor handoff yaml fragment의 `workflow.methodology` 필드 silent ignore 해소.

**Steps:** (dharness repo 측)
1. `references/intent-profile-schema.md` `workflow` 섹션에 3 필드 추가
   - `methodology: [string]` (catalog 24 enum + 자유 입력)
   - `methodology_source: string`
   - `methodology_matrix_row: string`
2. `harness-validate` 회로에 enum 검증 추가 (catalog 박제 목록과 cross-check)
3. CHANGELOG + minor version bump

**Acceptance:**
- dharness schema 3 필드 박제
- `harness-validate` 회로 통과 (enum 위반 차단 확인)
- dharness CHANGELOG `Added` 박제

**Deliverables:** (dharness repo)
- `references/intent-profile-schema.md` (수정)
- `harness-validate` 회로 (수정)
- dharness CHANGELOG (수정)
- dharness version bump commit

**Effort:** 30분~1시간 (dharness 측).

**Deps / Risks:**
- 의존: 본 plugin v0.2.0+ 박제 (필드 enum 박제용 catalog 24 source 필요)
- 위험: dharness 측 기존 사용자 schema 위반 가능 — backward compat patch 필요할 수 있음

---

### A2. dharness Phase 2 sub-skill 호출 분기

**Goal:** dharness Phase 2 grilling 중 사용자가 "추천해줘" 응답 시 → 본 plugin sub 모드 호출 → yaml fragment merge.

**Steps:** (dharness repo 측)
1. `references/grilling-loop.md` 또는 동등 위치에 분기 박제
2. `quality.test_rigor` 또는 `workflow.methodology` 필드 grilling 중 "추천" 키워드 감지
3. sub-skill 호출 → yaml fragment 수신 → `intent_profile.md` frontmatter merge
4. `meta.user_confirmed_fields`에 박제

**Acceptance:**
- dharness Phase 2 시뮬레이션에서 sub 모드 진입 + yaml merge 성공
- merge 충돌 (기존 값 ≠ advisor 값) 시 dharness 본 게이트로 사용자 선택

**Deliverables:** (dharness repo)
- `references/grilling-loop.md` (수정)
- Phase 2 분기 회로 (수정)
- 통합 테스트 transcript 박제

**Effort:** 1~2시간 (dharness 측).

**Deps / Risks:**
- 의존: A1 선행 (schema 박제 후 merge 가능)
- 위험: dharness Phase 2 doctrine "5필수 + 5선택" 박제와 충돌 가능 — methodology가 어느 그룹에 속하는지 결정 필요

---

### A3. 본 repo 측 후속 정합

**Goal:** A1·A2 완료 후 본 repo의 "dharness patch 미완" 경고 박제 제거.

**Steps:**
1. README "Compatibility" 섹션 갱신
   - 현 "not yet" 문구 제거
   - dharness 호환 버전 박제 (e.g. "requires dharness vX.Y+")
2. `handoff-template.md §2 §workflow.methodology enum 박제 필요` 박스 제거
3. CHANGELOG `[0.3.x or 0.4.0]` 박제
   - `Changed`: dharness 호환 박제 완료
4. ROADMAP A1·A2·A3 `[STATUS: done]` 갱신

**Acceptance:**
- README·CHANGELOG·handoff-template·ROADMAP 4 파일 갱신
- single commit `docs: dharness vX.Y compatibility confirmed`

**Deliverables:**
- README.md / CHANGELOG.md / handoff-template.md / ROADMAP.md (수정)
- commit + tag (필요 시 patch bump)

**Effort:** 10~15분.

**Deps / Risks:**
- 의존: A1·A2 완료
- 위험: dharness 측 patch 도중 schema 변경 시 본 repo 박제와 drift 가능 — A1·A2 완료 immediately 후 진행

---

## Track C — v0.3+ deferred (개요만)

### C1. matrix tie-breaking 학습화

**Goal:** `[matrix-miss]` 시 LLM fallback 비재현 해소. 동일 신호 → 동일 추천 ≥90% 재현.

**Approach:** telemetry 누적 → 5 룰 weight 학습 → tie-break 재현성 ↑.

**Acceptance:** 동일 신호 조합 ≥10회 누적 시 동일 추천 출력.

**Effort:** 1~2일 (algo 박제 + sparse data 처리).

**Deps:** C2 telemetry 선행 필수.

**Target:** v0.4+ 후보.

---

### C2. usage telemetry 박제

**Goal:** advisor 추천 vs 사용자 확정 신호 누적. C1·C3·C4 base.

**Approach:**
- 위치: `_workspace/_advisor/_telemetry.jsonl` (append-only)
- 레코드: `{ts, matrix_row, top_3, user_confirmed, sigils, dharness_handoff_taken}`
- 회로: `harness-adapt` 패턴 차용 (자동 적용 X, 사용자 승인 후 weight 갱신)

**Acceptance:**
- Phase 4 완료 시 1줄 append
- 사용자 opt-in 게이트 (PII 위험 회피)
- 30일 보관 + prune 옵션

**Effort:** 4~8시간.

**Deps:** opt-in 정책·masking 정책 선행 박제.

**Target:** v0.4.0.

---

### C3. catalog auto-evolve

**Goal:** `[catalog-miss]` 누적 → catalog 24 항목 자동 신규 후보 draft + 사용자 게이트.

**Approach:** C2 telemetry sigil 카운터 ≥3회 → catalog 형식 draft + confirm.

**Acceptance:** 누적 ≥3회 동일 키워드 시 draft 출력. 승인 시 patch.

**Effort:** 2~4시간 (C2 위에서).

**Deps:** C2.

**Target:** v0.4+ 후보.

---

### C4. `[unverified-combo]` 자동 분석

**Goal:** §2.3 시너지 표 stale 회피. B2 수동 작업 자동화.

**Approach:** C2 telemetry sigil 카운터 ≥3회 → §2.3 신규 셀 draft + confirm.

**Acceptance:** 누적 ≥3회 동일 조합 시 draft 출력. 승인 시 patch.

**Effort:** 1~2시간 (C2·C3 위에서, 코드 공유).

**Deps:** C2.

**Target:** v0.4+ 후보 (C3와 통합 가능).

---

## 변경 박제 규칙

- 각 항목 완료 시 ROADMAP `[STATUS]` 갱신 + PLAN 해당 섹션 박제
- v0.x.0 release마다 CHANGELOG `Added`/`Changed`/`Fixed` 박제
- breaking 변경 시 major bump + dharness 측 알림 박제
- PLAN 자체 = 매 release마다 retrospective 박제 후 다음 release 계획 갱신
