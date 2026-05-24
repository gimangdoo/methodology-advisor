# Roadmap

`methodology-advisor` 잔존 작업 트래커. 박제 시점: 2026-05-24 (v0.3.0 release 직후).

세 트랙으로 분리 박제:

- **Track A** — dharness 외부 repo 의존 (블로커: 다른 repo)
- **Track B** — 본 repo 내 즉시 actionable (운영 검증 + 시너지 표 확장)
- **Track C** — v0.3+ deferred (재현성·자동화 개선)

`[STATUS]` sigil: `pending` (미진행) / `in-progress` / `done` / `blocked`.

---

## 현재 상태 (2026-05-24)

- v0.3.0 tag origin 푸시 대기 (commit 박제 완료)
- 플러그인 코드 박제 완료 (skill + 3 references + command + marketplace)
- B1-static 정합성 점검 완료 (tests/static/B1-static-2026-05-24.md)
- B2 시너지 표 +10 셀 박제 완료 (catalog §2.3: 15 → 25)
- 본 시점 이후 실 사용·운영 데이터 0건 — B1-runtime 미수행

---

## Track A: dharness 측 schema patch (외부 의존)

목표: dharness `intent-profile-schema.md`에 `workflow.methodology` enum 필드 추가 → advisor handoff yaml fragment의 `workflow.methodology` 필드 silent ignore 해소.

### A1. `intent-profile-schema.md` 필드 추가 `[STATUS: blocked]`

위치: dharness repo (`gimangdoo/dharness` — 별 repo).

추가 필드 (handoff-template.md §2 박제):

```yaml
workflow:
  ci_cd: none | lint | test | full_pipeline       # 기존
  methodology: [string]                            # 신규
  methodology_source: string                       # 신규 — 출처 추적 (e.g. "methodology-advisor v0.2.0")
  methodology_matrix_row: string                   # 신규 — 재현성 (e.g. "R6" or "matrix-miss")
```

수락 조건:
- dharness schema enum에 catalog 24개 박제 (자유 입력 허용 `[catalog-miss]` sigil)
- dharness `harness-validate` enum 검증 회로 추가
- dharness CHANGELOG·minor version bump

### A2. dharness Phase 2 grilling 분기 박제 `[STATUS: blocked]`

dharness Phase 2가 `workflow.methodology` 미박제 시 → `/methodology-advisor` sub-skill 호출 게이트 추가.

수락 조건:
- dharness `references/grilling-loop.md` 또는 동등 위치에 분기 박제
- "추천해줘" 입력 시 본 plugin sub 모드 호출
- yaml fragment merge 성공 → `meta.user_confirmed_fields`에 `workflow.methodology` 박제

### A3. 본 repo 측 후속 액션 `[STATUS: pending]`

- A1·A2 완료 후 본 repo CHANGELOG에 "dharness vX.Y 호환 박제" 명시
- `handoff-template.md §2 §workflow.methodology enum 박제 필요 (dharness patch)` 박스 제거
- README "Compatibility" 섹션 갱신 (현 "not yet" 문구 제거)

---

## Track B: 본 repo 내 actionable

### B1. e2e 운영 검증 `[STATUS: partial]`

- B1-static: `[done]` (tests/static/B1-static-2026-05-24.md)
- B1.1 자연어 trigger: `[pending]` (사용자 환경 필요)
- B1.2 wrapper command: `[pending]` (사용자 환경 필요)
- B1.3 dharness-sub: `[blocked]` (Track A1·A2 의존)
- B1.4 실패 케이스 4종: `[pending]` (사용자 환경 필요)

본 plugin 코드는 1회도 실 실행 안 됨. 다음 시나리오 1회 dryrun + 결과 박제.

#### B1.1 자연어 trigger → Phase 0~4

- 시나리오: `"전자상거래 백엔드 신규 구축, small team 5명. 방법론 추천해줘"`
- 기대: skill 자동 로드 → Phase 0 grilling → R6 매칭 → top-3 + 합성 1줄
- 검증: `_workspace/_advisor/{ts}_signals.md` / `_recommendation.md` / `_handoff.md` 3 파일 생성
- 박제: 결과 transcript을 `tests/manual/e2e-trigger.md`에 단발성 박제

#### B1.2 `/methodology-advisor:advise-and-build` wrapper

- 시나리오: `/methodology-advisor:advise-and-build "환자 데이터 SaaS, HIPAA"`
- 기대: advisor 완주 → 사용자 confirm → `/harness:harness-new <합성 1줄>` 자동 호출
- 검증: dharness `_workspace/_baseline/` 생성 (별 검증)
- 박제: 결과 transcript을 `tests/manual/e2e-wrapper.md`에 박제

#### B1.3 dharness-sub 모드 (Track A 완료 후 의존)

- A1·A2 완료 전까지 `[blocked]`
- 시나리오: dharness Phase 2 grilling 중 "추천해줘" → sub 모드 호출
- 기대: yaml fragment 생성 → dharness intent_profile.md merge

#### B1.4 실패 케이스 1회씩

- `[matrix-miss]` 강제 발화 (catalog 외 도메인)
- `[catalog-miss]` 자유 입력
- 사용자 `다시` 응답 (Phase 3 게이트)
- 사용자 override (advisor 추천 무시)

수락 조건:
- 4 시나리오 각 1회 dryrun + transcript 박제
- 발견 버그 issue로 박제
- README "Usage" 섹션에 실 출력 예시 1개 박제

### B2. 시너지 표 확장 `[STATUS: done]` (v0.3.0)

현재 `methodology-catalog.md §부록 §2.3` = 15셀. matrix 20행에 등장하나 시너지 표 미박제 조합 cross-check:

| 신규 시너지 후보 | matrix row 출처 | 사유 박제 후보 |
|--------------|-------------|-------------|
| TDD + GitHub Flow | R4·R6·R10 | 단위 안정 + PR 게이트 검증 (small team default) |
| DDD + GitHub Flow | R6 | bounded context 변경 단위 = PR 단위 |
| Modular Monolith + Hexagonal | R5 | 모듈 경계 + 외부 의존 격리 (운영 비용 ↓) |
| DDD + Modular Monolith | R8 | bounded context 박제 + 단일 배포 (medium team 정석) |
| TDD + Feature Toggle | R14·R15 | 회귀 안전망 + 점진 rollout (brownfield) |
| TDD + Kanban | R4 | 단위 안정 + continuous flow (solo·small) |
| Spec-driven + Trunk-based | R9·R13 | 명세 박제 + CI 게이트 (OpenAPI 정합) |
| MVC + Snapshot + Kanban | R10 | 정형 패턴 + UI 회귀 + 가벼운 PM (internal_tool 정석) |
| Hexagonal + Contract testing | R17 | 외부 의존 격리 + 통합 회귀 검출 |
| Event Storming + Snapshot | R17 | 도메인 발굴 + UI 회귀 (workshop 직후 UI 안정화) |

수락 조건:
- 위 10셀 중 ≥7셀 catalog §2.3 박제
- 각 셀 = 시너지 사유 1줄 + 적합 시나리오 1줄
- minor bump v0.3.0
- CHANGELOG `Added` 항목 박제

### B3. ROADMAP.md `[STATUS: done]`

본 파일.

---

## Track C: v0.3+ deferred (재현성·자동화 개선)

지금 박제 X. v0.3+ 후보로 박제만.

### C1. matrix tie-breaking 학습화

- 현: 5 룰 (사용자 부담 / 유지비 / dharness 매핑 명확도 / catalog 신호 일치 / 시너지 박제) deterministic
- 한계: `[matrix-miss]` 시 LLM fallback 비재현
- 후보: telemetry 기반 weight 학습 — Phase 3 사용자 선택 vs advisor top-1 일치율 누적
- 의존: telemetry 박제 (C2 선행)

### C2. usage telemetry 박제

- 위치: `_workspace/_advisor/_telemetry.jsonl` (append-only)
- 박제 필드: `{ts, matrix_row, top_3, user_confirmed, sigils, dharness_handoff_taken}`
- 회로: `harness-adapt` 패턴 차용 (사용자 승인 후만 적용, 자동 적용 없음)
- 한계: PII·도메인 정보 마스킹 필요 (단순 hashing 검토)

### C3. catalog auto-evolve

- 트리거: `[catalog-miss]` 누적 ≥3회 동일 키워드 → 신규 항목 박제 후보 출력
- 출력: catalog 형식 4 필드(정의·적합·부적합·dharness 매핑·비용·보완) draft + 사용자 confirm 게이트
- 의존: C2 telemetry 선행

### C4. `[unverified-combo]` sigil 누적 분석

- 시너지 표 §2.3 미박제 조합 출현 시 sigil 박제 중
- ≥3회 동일 조합 누적 → §2.3 신규 셀 박제 후보 자동 추출
- B2와 통합 가능 (B2 = 수동, C4 = 자동)

---

## 진척 룰

- Track B 항목 완료 시 ROADMAP `[STATUS]` 갱신
- Track A 항목은 dharness repo 측 PR/commit 박제 시 link 박제
- Track C 항목은 v0.3.0 release 직전 우선순위 재검토
- ROADMAP 자체 = 매 minor release마다 갱신 (단 source of truth는 CHANGELOG)
