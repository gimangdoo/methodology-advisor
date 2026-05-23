# methodology-advisor

A Claude Code plugin that recommends a software development methodology (TDD / DDD / BDD / Spec-driven / Trunk-based / Kanban, etc.) based on project structure, purpose, features, and scale.

Pairs with the [`dharness`](https://github.com/gimangdoo/dharness) factory: emits a single handoff sentence that can be fed straight into `/harness:harness-new`.

---

## What's inside

| Component | Path | Purpose |
|-----------|------|---------|
| Skill | `plugins/methodology-advisor/skills/methodology-advisor/SKILL.md` | The advisor itself. Auto-triggers on phrases like "방법론 추천해줘", "어떤 방식으로 진행", "TDD가 좋을까 BDD가 좋을까". |
| Command | `plugins/methodology-advisor/commands/advise-and-build.md` | `/methodology-advisor:advise-and-build` — wrapper that chains `advisor → user confirm → /harness:harness-new`. |
| Reference: catalog | `.../references/methodology-catalog.md` | 24 methodologies × (definition, fit signals, dharness mapping, cost, complements). |
| Reference: matrix | `.../references/decision-matrix.md` | Deterministic rules mapping 5-axis signals → top-3 recommendations. |
| Reference: handoff | `.../references/handoff-template.md` | Synthesis patterns for the standalone 1-line sentence and the dharness-sub yaml fragment. |

---

## Two modes

### `standalone` (T1) — pre-`harness-new`

1. User asks for a methodology recommendation.
2. Advisor runs Phase 0–4: gather 5-axis signals → match decision matrix → output top-3 with tradeoffs → user confirms → synthesize a single sentence.
3. User feeds the sentence to `/harness:harness-new` (manually copy, or via the `/methodology-advisor:advise-and-build` wrapper for one-shot chaining).

### `dharness-sub` (T2) — invoked from inside dharness Phase 2

1. dharness Phase 2 grilling hits `quality.test_rigor` / `workflow.methodology` and the user says "추천해줘".
2. dharness delegates to this advisor as a sub-skill.
3. Advisor outputs a yaml fragment that merges directly into `intent_profile.md` — no separate confirm gate (dharness owns the gate).

---

## Install

This repo is itself a [marketplace](https://docs.claude.com/en/docs/claude-code/plugin-marketplaces). Add it to Claude Code:

```
/plugin marketplace add gimangdoo/methodology-advisor
/plugin install methodology-advisor@methodology-advisor-marketplace
```

Or point at a local clone:

```
/plugin marketplace add /path/to/methodology-advisor-project
/plugin install methodology-advisor@methodology-advisor-marketplace
```

---

## Usage

### Natural-language trigger (recommended)

```
"전자상거래 백엔드 신규 구축, small team 5명. 방법론 추천해줘."
```

The skill auto-loads, runs Phase 0–4, and prints a synthesized sentence ready for dharness.

### Slash command (one-shot chain)

```
/methodology-advisor:advise-and-build "환자 데이터 관리 SaaS, HIPAA 준수 필수"
```

Runs the advisor, prompts for confirmation, then automatically calls `/harness:harness-new` with the synthesized sentence.

---

## Triggers

**Should trigger:**
- "방법론 추천해줘", "어떤 방식으로 진행할까", "프로세스 가이드 필요"
- "TDD가 좋을지 BDD가 좋을지", "개발 방법 선택"
- "dharness 호출 전에 정하고 싶어"
- dharness Phase 2 delegated call (`dharness-sub` mode)

**Should NOT trigger:**
- "TDD로 진행해줘" (already specified — call dharness directly)
- Code review, debugging, library selection ("lodash vs ramda")
- Tool selection ("Slack vs Discord")
- "TDD가 뭐야" (concept question — answer directly without entering the advisor flow)

---

## Outputs (standalone mode)

| Path | Contents |
|------|----------|
| `_workspace/_advisor/{ts}_signals.md` | The 5-axis signals collected in Phase 0 (yaml). |
| `_workspace/_advisor/{ts}_recommendation.md` | Top-3 candidates from Phase 2 + the user-confirmed choice from Phase 3. |
| `_workspace/_advisor/{ts}_handoff.md` | The synthesized sentence + the [A]/[B]/[C] next-step options. |

`_workspace/_advisor/` lives at the user's project root and is safe to coexist with `_workspace/_baseline/` (dharness factory) and `_workspace/_intent/`.

---

## Design notes

- **Why this is a separate plugin and not part of dharness:** dharness has a "factory" single-responsibility doctrine. Methodology recommendation is an advisory step that *precedes* the factory; embedding it would expand dharness scope and conflict with the PM-domain exclusion decision (see `dharness` project memory on handoff/ICA absorption).
- **Why grilling-loop:** users who already know what they want shouldn't need to enter the advisor. Users who don't shouldn't be asked all five signals at once. The grilling-loop pattern (one question at a time, ≥3 suggested answers, free-input fallback) is borrowed from dharness `references/grilling-loop.md`.
- **Why a decision matrix:** deterministic matches are reproducible across sessions and make recommendation drift observable (matrix row R# is stamped in the handoff). LLM fallback applies only on `[matrix-miss]`.
- **Why output a sentence, not a profile dump:** the dharness factory's Phase 2 already extracts signals from a sentence. Re-marshalling into a profile would double the parsing surface and create a sync hazard.

---

## Compatibility

- **Claude Code:** any version that supports plugins + skills + slash commands.
- **dharness:** the `workflow.methodology` field referenced in `handoff-template.md §2` is **not yet** in the dharness `intent-profile-schema.md`. Until that patch lands, `quality.test_rigor` merges cleanly; `workflow.methodology` is emitted but ignored by dharness with a warning. Track the patch in the dharness project.

---

## Versioning

Semantic versioning. Catalog or matrix edits that change recommendation behavior bump the minor version. Schema-breaking handoff format changes bump the major version (and require a dharness-side update).

Current: `0.1.0`.

---

## License

MIT — see [LICENSE](./LICENSE).
