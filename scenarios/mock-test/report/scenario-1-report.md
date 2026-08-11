# Scenario 1 Performance Report — Customer Support Resolution Agent

> Consolidated performance summary across all practice attempts for Scenario 1. Source detail for every missed question lives in `scenarios/mock-test/wrong-answers/wrong-answer-csa.md` — this report summarizes trends and readiness, not individual question explanations.

---

## 1. Score Summary

| Attempt | Score | Percentage | Notes |
|---|---|---|---|
| Practice Mock | 17/17 | **100%** | Scenario-1 practice question bank (`scenario-1-customer-support.md §7`) — all task statements 1.1–5.2 |
| Mock Test 1 | 11/15 | **73.3%** | First cold attempt on mixed, tempting-option mock test |
| Mock Test 2 | 12/15 | **80%** | Custom set targeting gaps found in Mock 1 |
| Mock Test 3 | 17/19 | **89.5%** | Full-domain sweep, no domain labels (blind categorization) |
| Targeted Drill | 9/10 | **90%** | Isolated drill on the two remaining risk patterns only (architecture-overreach, escalation-scope), fresh scenarios not seen in prior tests |

**Overall trend: 73.3% → 80% → 89.5% → 90% (targeted).** The targeted drill confirms both tracked risk patterns are closed: 0 misses on architecture-overreach questions, 0 misses on escalation-scope questions. The single miss was a new, distinct, isolated pattern (over-rigidification — forcing a fixed process where adaptive judgment was needed), not a recurrence of either tracked gap.

---

## 2. How to Read This Trajectory

The practice-mock 100% score reflects strong grasp of the *underlying concepts* in isolation — each question there tests one idea cleanly (e.g., hook vs. prompt, one escalation trigger at a time). The mock-test scores are lower because those tests deliberately blend concepts and use tempting, similarly-worded distractors that require *distinguishing between adjacent categories under pressure* — a harder skill than knowing each concept individually.

The steady climb from 73% to 89% across three increasingly adversarial mock tests indicates the gap was never conceptual knowledge — it was answer-discrimination under ambiguity. That gap is closing.

---

## 3. Mistake Categories Identified

Every miss across the three mock tests falls into one of these categories (see `wrong-answer-csa.md` for full per-question detail):

| Category | Mock Tests Affected | Status |
|---|---|---|
| 🔒 Safety/Financial/Compliance vs. probabilistic prompting | Mock 1 (Q51) | Resolved — 0 misses in this category since Mock 1 |
| 🧠 Reasoning/sequencing quality vs. architectural overreach | Mock 1 (Q47, Q53), Mock 2 (Q62) | **Recurring — highest-frequency error type** |
| 🌀 Open-ended/variable quality vs. fixed-pattern few-shot | Mock 1 (Q52) | Resolved — 0 misses since Mock 1 |
| 🔧 Technique locked by stem (inclusion & exclusion phrasing) | Mock 1 (Q60), Mock 2 (Q69) | Resolved — 0 misses in Mock 3 |
| 🆕 Diagnose-before-prescribing (missing capability check) | Mock 2 (Q68) | Resolved — 0 misses since Mock 2 |
| 🆕 All-or-nothing escalation scope on multi-issue requests | Mock 3 (Q5) | New, isolated — one occurrence |
| 📚 Knowledge gap — full escalation trigger list | Mock 3 (Q18) | New, isolated — one occurrence, easily closed |

**Key insight:** 5 of 7 identified mistake categories have zero recurrences in the most recent attempt. The one confirmed *recurring* systemic pattern — reaching for a bigger architectural fix (extra model calls, new agent instances, speculative execution) when a simpler few-shot/prompting fix is correct — has appeared in 2 of 3 mock tests and remains the top-priority item to drill.

---

## 4. Current Readiness Assessment — EXAM-READY

Based on Mock Test 3 (full domain coverage, no category scaffolding): **17/19 (89.5%)** falls in the "strong readiness" band per the scenario's own scoring guide (17–19 = strong readiness, 13–16 = review needed, below 13 = revisit walkthrough).

That aggregate score alone wasn't sufficient proof, since Mock 3 happened not to test the recurring architecture-overreach pattern at all. The follow-up **targeted drill** (10 fresh questions isolating only architecture-overreach and escalation-scope) closes that gap in the evidence: **9/10, with zero misses in either tracked category.** Combining the full-domain strength (Mock 3) with the isolated-pattern confirmation (targeted drill) is what justifies calling Scenario 1 exam-ready — not either test alone.

**Strengths (consistently correct across all attempts):**
- `stop_reason`-based loop termination logic
- `PreToolUse` vs. `PostToolUse` hook placement and purpose
- Structured error response design (`errorCategory`, `isRetryable`, `description`)
- Tool description differentiation for ambiguous tool pairs
- `tool_choice` configuration (`auto` vs. `any` vs. forced single-turn)
- `CASE_FACTS` context-persistence pattern
- Basic escalation triggers (explicit request, policy gap)

**Gaps closed (confirmed via targeted drill):**
1. **Architecture-overreach instinct** — the tendency to propose a heavier fix (new model call, new agent, speculative/automatic tool execution) for problems that are actually simple reasoning/sequencing issues fixable with few-shot examples. Appeared in Mocks 1 and 2; **0/5 relevant questions missed in the targeted drill — confirmed closed.**
2. **Multi-issue escalation scope calibration** — decomposing a request *before* deciding escalation scope, so only the specific sub-issue requiring escalation gets escalated (not the whole request). **0/4 relevant questions missed in the targeted drill — confirmed closed.**
3. **Complete escalation trigger list** — explicit request OR policy gap OR inability to make progress after reasonable retries. Surfaced in Mock 3, not retested since (pure knowledge-recall item, low risk of resurfacing as a reasoning error).

**New, isolated watch-item (not yet a confirmed pattern):**
- **Over-rigidification** — forcing a fixed/identical process on every case regardless of content, when adaptive judgment is actually needed (the mirror-image failure to architecture-overreach: removing judgment by over-simplifying instead of over-architecting). One occurrence (targeted drill Q5). Not drilled further, since one instance doesn't yet meet the bar for a confirmed recurring pattern — noted in `wrong-answer-csa.md` in case it resurfaces in Scenario 2 or later.

---

## 5. Recommended Next Steps

1. ~~Take a Mock Test 4 weighted toward architecture-overreach and escalation-scope~~ — **done via targeted drill, both patterns confirmed closed (9/10, 0 misses in either category).**
2. **Shift practice time to other scenarios** (e.g., `scenario-2-code-generation.md`, `scenario-5-cicd.md`) using the same mock-test + wrong-answer-tracker + targeted-drill workflow established here. Don't keep re-testing Scenario 1 for marginal gains — the evidence bar for exam-readiness has been met.
3. Keep using `wrong-answers/wrong-answer-csa.md` as the running log for any future Scenario 1 misses, even after reaching exam-ready status — a single new miss post-readiness is still worth categorizing in case it signals a new pattern. In particular, watch for recurrence of the **over-rigidification** pattern (forcing a fixed process where adaptive judgment is needed) across other scenarios, since it may generalize beyond Scenario 1.

---

## 6. Attempt-by-Attempt Detail Reference

Full explanations for every missed question, including the specific trap and the rule extracted from it, are maintained in:

- `scenarios/mock-test/wrong-answers/wrong-answer-csa.md`

This report will be updated after each new Scenario 1 mock test attempt.
