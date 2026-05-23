# Changelog

All notable changes to this plugin are documented in this file. Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/); versioning is [SemVer](https://semver.org/spec/v2.0.0.html).

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
