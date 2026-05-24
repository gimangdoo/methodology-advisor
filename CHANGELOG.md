# Changelog

All notable changes to this plugin are documented in this file. Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/); versioning is [SemVer](https://semver.org/spec/v2.0.0.html).

## [0.3.1] — 2026-05-24

B1-runtime S1 dryrun에서 발견된 doctrine gap 2건 박제. 코드 동작 변경 X, 문서·doctrine 명시 patch.

### Fixed

- `SKILL.md §Phase 0` — grilling-loop `1q-at-a-time 엄수 룰` 명시 추가. S1에서 `constraints` 하위(Timeline + Compliance)를 동시 grilling한 사례 재발 방지. 단일 step = 단일 축 단일 질문, 다축 동시 박제 금지. (gap #1, severity=med)
- `SKILL.md §Phase 4` + `§산출물` — standalone 모드의 `_handoff.md`에도 dharness `intent_profile` 매핑 yaml 박제 doctrine 명시. dharness-sub schema와 동일 (`workflow.methodology` + `methodology_source` + `methodology_matrix_row`). [C] 옵션 후 별 세션 호출 시 re-grilling 회피 회로 박제. (gap #3, severity=low)

### Changed

- version literal `v0.3.0` → `v0.3.1` (plugin.json, marketplace.json, SKILL.md Phase 4-sub yaml, handoff-template.md §2, README "Current")

### Known limitations (이월)

- `hybrid_reason` 자유 텍스트 schema 정형화 → v0.4.0 후보 (gap #2)
- `workflow.methodology` enum dharness 측 schema patch 미진행 (Track A 의존)
- B1-runtime S2~S4 dryrun 미수행

## [0.3.0] — 2026-05-24

B1-static 정합성 점검 + B2 시너지 표 확장 + advisory gap 4건 박제. v0.2.0 release 당일 즉시 후속 patch.

### Added

- `methodology-catalog.md §부록 §2.3` 시너지 표 +10 셀 (15 → 25):
  - TDD + GitHub Flow (R4·R6·R10 빈출)
  - DDD + GitHub Flow (R6)
  - Modular Monolith + Hexagonal (R5)
  - DDD + Modular Monolith (R8)
  - TDD + Feature Toggle (R14·R15)
  - TDD + Kanban (R4)
  - Spec-driven + Trunk-based (R9·R13)
  - MVC + Snapshot + Kanban (R10)
  - Hexagonal + Contract testing (R17)
  - Event Storming + Snapshot (R17)
- `advise-and-build.md` frontmatter `allowed-tools: ["SlashCommand", "Read", "Write", "Bash", "Glob", "Grep"]` 박제 (명시적 도구 권한)
- `SKILL.md` 산출물 §에 mkdir 명시 + dharness-sub 산출물명 (`_handoff_sub.yaml`) 박제
- `PLAN.md` 신설 (전 트랙 상세 실행 계획 — Goal/Steps/Acceptance/Deliverables/Effort/Deps/Risks)
- `ROADMAP.md` 신설 (전 트랙 high-level 트래커)
- `tests/static/B1-static-2026-05-24.md` 박제 (정적 정합성 점검 보고)

### Changed

- `SKILL.md` description 트리거 키워드 확장:
  - should-trigger +2: "TDD/DDD/BDD 골라줘" (BDD 추가), "프로세스 가이드"
  - should-NOT-trigger +4: 도구 선택, 코드 검토, 방법론 개념 질문, harness-status 영역
- version literal `v0.2.0` → `v0.3.0` (plugin.json, marketplace.json, SKILL.md, handoff-template.md, README)

### Fixed

- version literal drift `v0.1.0` → `v0.2.0` (SKILL.md L118, handoff-template.md L73·L81) — v0.2.0 release 시점 박제 누락 (commit `55ea3ed`에서 우선 fix 후 본 릴리스에 박제)
- `advise-and-build.md` L41 dangling wikilink `[[project-dharness-handoff-ica-absorption]]` 제거 — published doc에 내부 memory slug leak 차단

### Known limitations

- `workflow.methodology` enum 박제는 dharness 측 schema patch 미진행 (v0.1.0·v0.2.0과 동일, Track A 의존)
- B1-runtime e2e dryrun 미수행 — 사용자 환경에서 진행 필요 (PLAN.md §B1-runtime 박제)
- `[matrix-miss]` 시 LLM fallback 비재현 (Track C1 의존)

## [0.2.0] — 2026-05-24

Hybrid composition doctrine 박제. 실 프로젝트의 단일 방법론 X, 4축 조합 default 명시.

### Added
- `methodology-catalog.md §부록 §2` Hybrid 합성 doctrine 신설:
  - §2.1 4축 합성 doctrine (설계·테스트·워크플로우·PM 각 0~1개, 최대 4개)
  - §2.2 상호 배타 표 (9 충돌 쌍 + 회피 룰)
  - §2.3 시너지 표 (15 검증 조합 + 적합 시나리오)
  - §2.4 합성 출력 형식 (Phase 2·4 박제 예시)
- `decision-matrix.md` 매칭 알고리즘 step 4·5 추가 (충돌 검출 + 시너지 강화)
- `decision-matrix.md` tie-breaking 룰 #5 추가 (시너지 표 박제 조합 우선)
- `SKILL.md` Phase 1 정책 4·5 추가, Phase 2 출력 항목 6번째 "시너지·충돌" 박제
- `SKILL.md` Phase 4 합성 룰에 4축 doctrine 적용 + 단일 박제 시 의도 확인 게이트
- `handoff-template.md` §1 합성 룰 #5·#6 추가 (4축 doctrine 격상 + 시너지 표 우선)
- `[conflict-detected]` / `[conflict-override]` / `[unverified-combo]` sigil 신설

### Changed
- 합성 룰 #1: "방법론 조합 최대 3개" → "최대 4개" (4축 doctrine 정합)
- 합성 룰 #2: "PM 1개만" → "축당 주 방법론 1개 + §2.3 박제 보조 패턴 허용"
- `[matrix-miss]` fallback이 4축 doctrine 명시적 참조 (실은 모든 추천이 4축으로 환원 가능, matrix는 검증 캐시)

### Known limitations
- `workflow.methodology` enum 박제는 dharness 측 schema patch 미진행 (v0.1.0과 동일)
- 시너지 표 §2.3은 sparse (15셀) — catalog 24×24=576 셀의 ~2.6%. 미박제 조합은 default valid 단 `[unverified-combo]` sigil 박제

## [0.1.0] — 2026-05-24

Initial release.

### Added
- `methodology-advisor` skill (Phase 0–4 workflow, standalone + dharness-sub modes).
- `/methodology-advisor:advise-and-build` slash command wrapping `advisor → user confirm → /harness:harness-new`.
- 24-item methodology catalog covering design, testing, workflow, and PM axes.
- 20-row decision matrix for deterministic signal → recommendation matching with `[matrix-miss]` LLM fallback.
- Handoff template documenting the standalone 1-line synthesis pattern, the dharness-sub yaml-fragment pattern, and the dharness `intent_profile` enum mapping table.
- Marketplace manifest at the repository root for `/plugin marketplace add` installation.

### Known limitations
- `workflow.methodology` field is emitted in the dharness-sub handoff but is **not yet** present in the dharness `intent_profile` schema. Until a dharness-side patch lands, that field is ignored with a warning on merge; `quality.test_rigor` merges cleanly.
- Matrix tie-breaking is rule-based, not learned — recommendations against `[matrix-miss]` rely on LLM fallback and are not reproducible across sessions.
