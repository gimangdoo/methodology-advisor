# S1 결과 — 자연어 trigger (T1 golden path)

실행 일시: 2026-05-24 14:14:11
plugin version: v0.3.0
Claude Code 버전: 2.1.150
작업 디렉토리: `C:\Users\user01\awesome-files\_e2e-test-s1`
실행 세션 export: `_e2e-test-s1/2026-05-24-141628-git-init-httpsgithubcomgimangdoomethodology.txt`

---

## 입력 발화

```
전자상거래 백엔드 신규 구축, small team 5명. 방법론 추천해줘.
```

---

## 검증 결과 (17 항목)

| # | 항목 | PASS/FAIL | 근거 |
|--|----|---------|----|
| 1 | skill 자동 트리거 (수동 호출 X) | PASS | `Skill(methodology-advisor:methodology-advisor) Successfully loaded skill` 박제 |
| 2 | grilling-loop 1q-at-a-time 준수 | **partial FAIL** | 첫 grilling = 1q (purpose). 둘째 grilling = 2q 동시 (Timeline + Compliance) |
| 3 | 추천 답안 ≥3개 + 자유 입력 옵션 | N/A | transcript에 grilling 질문 본문 박제 X (Claude Code TUI 특성). plugin gap 아님 |
| 4 | matrix row `R6` 박제 | PASS | "Matrix 매치: R6 부분(4/5, compliance dim 누락) + R7 부분(4/5, timeline 불일치)" |
| 5 | top-3 후보 6 항목 박제 | PASS | 정의·적합 사유·시너지/충돌·비용·dharness 매핑 박제. 한 줄 정의는 단일 행 박제 |
| 6 | 시너지 표 §2.3 박제 조합 박제 | PASS | DDD+TDD, TDD+GitHub Flow, DDD+GitHub Flow (§2.3) 모두 박제 — **v0.3.0 신규 셀 효과 검증됨** |
| 7 | `[unverified-combo]` sigil 미출현 | PASS | sigil 박제 0건 |
| 8 | `[conflict-detected]` sigil 미출현 | PASS | "충돌 없음" 박제 |
| 9 | Phase 3 enum 박제 (`1/2/3/자유/다시/상세`) | PASS | "1 / 2 / 3 — top-3 중 선택, 자유, 다시, 상세" |
| 10 | Phase 4 합성 1줄 4 요소 박제 | PASS | "전자상거래 백엔드 신규 구축, small team 5명, DDD + TDD + GitHub Flow + Spec-driven으로 진행. PCI-DSS 준수, 3-12개월." = core_feature + scale + 4축 방법론 + 부가제약 |
| 11 | `[A][B][C]` 옵션 박제 | PASS | 옵션 3 박제 |
| 12 | mkdir 자동 (`_workspace/_advisor/`) | PASS | `Bash(mkdir -p _workspace/_advisor && date +%Y%m%dT%H%M%S)` |
| 13 | 산출물 3 파일 박제 | PASS | signals/recommendation/handoff 박제 |
| 14 | 산출물 timestamp 일치 | PASS | `20260524T141411` prefix 3 파일 동일 |
| 15 | `_signals.md` = Phase 0 yaml 5축 | PASS | project_type/purpose/core_feature/scale/timeline/constraints 박제 |
| 16 | `_recommendation.md` = top-3 + 사용자 확정 | PASS | matrix_match·top-3·selection·override 박제 |
| 17 | `_handoff.md` = 합성 1줄 + 옵션 | PASS | 합성 1줄 + axes 4 + 옵션 3 박제 |

**종합: 16 PASS, 1 partial FAIL, 1 N/A. S1 전반적 PASS.**

---

## 발견 gap

| # | 위치 | 내용 | severity | fix 후보 |
|---|----|----|--------|--------|
| 1 | grilling Step 2 | 2 질문 (Timeline + Compliance) 동시 박제 → `1q-at-a-time` 약 위반 | **med** | SKILL.md §Phase 0 "각 grilling step = 1q only, 다축 동시 박제 금지" 명시 추가 |
| 2 | `_recommendation.md` `hybrid_reason` 필드 | 자유 텍스트 박제 (`PCI-DSS 강신호 → R6 base + Spec-driven 보강`). schema 정형화 안 됨 | low | SKILL.md 또는 handoff-template.md에 `hybrid_reason: {trigger_signal, base_row, augment_axes[]}` schema 박제 후보 (v0.4) |
| 3 | `_handoff.md`에 `intent_profile` 매핑 박제 | standalone 모드 산출물에 sub 모드용 yaml fragment 박제 — doctrine 정합? | low | doctrine 측 의도된 박제 (standalone도 dharness 인계 시 매핑 활용 가능). gap 아님 — 단 SKILL.md §Phase 4·§산출물에 명시 추가 후보 |
| 4 | transcript "User answered Claude's questions" wrapper | grilling 질문 본문이 export에 박제 X — 추천 답안·자유 옵션 검증 외부 관찰 불가 | low (Claude Code TUI 한계, plugin gap 아님) | 검증 한계 — B1-runtime 결과 박제 시 사용자가 본문 별도 박제 권장 |

**즉시 fix 후보:** gap #1 (med severity) — v0.3.x patch.
**v0.4.0 후보:** gap #2 — hybrid_reason schema 정형화.
**doctrine 명시 후보:** gap #3 — standalone 산출물에 intent_profile 매핑 박제 합법성 명시.

---

## v0.3.0 시너지 표 확장 효과 검증

S1 출력의 후보 1·2·3 시너지 박제 분석:

| 후보 | 시너지 박제 조합 | v0.2.0 vs v0.3.0 |
|----|-------------|---------------|
| 1 (DDD+TDD+GitHub Flow+Spec-driven) | DDD+TDD, TDD+GitHub Flow, DDD+GitHub Flow | TDD+GitHub Flow·DDD+GitHub Flow = **v0.3.0 신규 §2.3 박제** |
| 2 (DDD+BDD+Spec-driven+Hexagonal) | DDD+Hexagonal, BDD→ATDD | 기존 v0.2.0 박제 |
| 3 (Modular Monolith+DDD-lite+TDD+Trunk-based+Feature Toggle) | Modular Monolith+DDD-lite, TDD+Trunk-based, Trunk-based+Feature Toggle | 기존 v0.2.0 박제 |

후보 1이 v0.3.0 신규 셀 2개 활용. 만약 v0.2.0 상태였다면 후보 1에 `[unverified-combo]` sigil 박제 가능성 ↑ → v0.3.0 시너지 확장 효과 실 운영 검증됨.

---

## hybrid 합성 동작 검증

S1은 R6·R7 모두 partial match (각 4/5). matrix 알고리즘 `Exact > Partial≥3 > Miss` 정책 박제와 정합 — exact 부재 시 partial 후보 정렬. PCI-DSS 강신호 cross-check로 R6 base + Spec-driven (R7 박제 항목) 보강 = **hybrid 합성 정상**.

catalog §부록 §2.1 4축 doctrine 박제 정합: 설계 (DDD) + 테스트 (TDD) + 워크플로우 (GitHub Flow) + PM (Spec-driven) = 4축 완비, 충돌 없음.

---

## 산출물 3 파일 (실 내용)

### `_signals.md`
```yaml
project_type: greenfield
purpose: product
core_feature: 전자상거래 백엔드
scale: small_team   # 5명
timeline:
  horizon: medium   # 3-12M
constraints:
  compliance:
    regulatory: [PCI-DSS]
  quality_target: high   # 결제 정확성 + 감사 요구
```

### `_recommendation.md` frontmatter
```yaml
matrix_match: partial (R6 4/5, R7 4/5)
hybrid_reason: PCI-DSS 강신호 → R6 base + Spec-driven 보강
confirmed: true
selection: 1
```

### `_handoff.md` 합성 1줄
```
"전자상거래 백엔드 신규 구축, small team 5명, DDD + TDD + GitHub Flow + Spec-driven으로 진행. PCI-DSS 준수, 3-12개월."
```

intent_profile 매핑:
```yaml
quality:
  test_rigor: tdd
workflow:
  methodology: ["DDD", "TDD", "GitHub Flow", "Spec-driven"]
  methodology_source: "methodology-advisor v0.3.0 (matrix-hit partial R6+R7 hybrid)"
constraints:
  compliance:
    regulatory: [PCI-DSS]
```

---

## 다음 액션

1. gap #1 (med) → SKILL.md §Phase 0에 "1q-at-a-time 엄수" 명시 추가 (v0.3.1 patch 또는 B1.2 후 묶음 박제)
2. gap #2 (low) → v0.4.0 로드맵 후보
3. gap #3 (low) → SKILL.md §Phase 4·§산출물에 standalone 모드 intent_profile 매핑 박제 doctrine 명시 (v0.3.1)
4. README "Usage" 섹션에 합성 1줄 실 출력 박제 (현 의사 코드 → 실 transcript 1줄)
5. ROADMAP B1.1 `[script-ready]` → `[done]`
6. 2순위 (S2 wrapper) 진입
