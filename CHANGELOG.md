# Changelog

All notable changes to this plugin are documented in this file. Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/); versioning is [SemVer](https://semver.org/spec/v2.0.0.html).

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
