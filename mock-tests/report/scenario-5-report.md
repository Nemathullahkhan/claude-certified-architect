# Scenario 5 Performance Report — Claude Code for CI/CD

> Consolidated performance summary across all practice attempts for Scenario 5. Source detail for every missed question lives in `mock-tests/wrong-answers/wrong-answers-cicd.md` — this report summarizes trends and readiness, not individual question explanations.

---

## 1. Score Summary

| Attempt | Score | Percentage | Notes |
|---|---|---|---|
| Practice Mock | 18/18 | **100%** | Scenario-5 practice question bank (`scenario-5-cicd.md §7`) — all task statements 3.6, 3.1, 4.1, 4.2, 4.3, 4.6 |
| Mock Test 1 | 12/15 | **80%** | First adversarial mock (`mock-test-cicd-1.md`, Q16–Q30) — Batch tool-loop limit, explicit criteria vs few-shot, in-finding triage under no-filter constraint |

**Overall trend: 100% → 80%.** The expected practice-bank-to-adversarial drop occurred (milder than S1/S3's first-mock ~73%). Three distinct miss categories; none recurring yet.

---

## 2. How to Read This Trajectory

The 100% practice-bank score reflected strong grasp of Scenario 5's *underlying concepts* in isolation. Mock Test 1's drop to 80% is the same discrimination-under-pressure pattern seen in Scenarios 1–3: adversarial stems stack a familiar-but-wrong fact (Batch latency; few-shot as default fix; process tiering) next to the stem-specific correct fix.

80% on a first adversarial mock is a solid start — above the historical first-mock floor — but does **not** yet meet the exam-ready bar (≥~90% with zero 2×-recurring categories).

---

## 3. Mistake Categories Identified

Every miss on Mock Test 1 falls into one of these categories (see `wrong-answers-cicd.md` for full per-question detail):

| Category | Mock Tests Affected | Status |
|---|---|---|
| 🔧 Batch Incompatible with Multi-Turn Tools | Mock 1 (Q18) | New, isolated — one occurrence |
| 🏳️ Vague Precision Instruction (criteria vs few-shot fork) | Mock 1 (Q22) | New, isolated — one occurrence |
| ⏱️ Process Redesign vs. In-Finding Triage Info | Mock 1 (Q28) | New, isolated — one occurrence |

**Key insight:** All three misses are distinct axes, not one recurring instinct. None yet meet the "confirmed recurring pattern" bar (2× across mocks). Mock 2 should deliberately re-test the Batch-tool-loop vs latency fork and the criteria-vs-few-shot fork under fresh scenarios.

**Strengths confirmed correct on Mock 1 (among others):**
- `--output-format json` + `--json-schema` over CLAUDE.md/prompt format examples
- Independent review instance for self-review bias
- Sync for blocking PR checks; Batch for nightly/weekly latency-tolerant jobs
- Few-shot for actionable format consistency when instructions alone fail
- Existing test file / prior findings injected into context
- `-p` for headless CI (rejecting invented `--batch` / `CLAUDE_HEADLESS`)
- Per-file + cross-file multi-pass review for large PRs
- Temporarily disable high-FP categories to restore trust

---

## 4. Current Readiness Assessment — STRONG START; NOT YET EXAM-READY

Mock Test 1 **12/15 (80%)** is better than the historical first-adversarial floor, but the three open categories need at least one more adversarial pass (and targeted drill only if any recurs).

**Gaps to watch on the next mock:**
1. **Batch question type discrimination** — latency/SLA matching vs multi-turn tool incompatibility. Same API, two different "why not Batch" answers depending on stem.
2. **4.1 vs 4.2 fork** — vague criterion / wrong-things-flagged → explicit criteria; inconsistent format despite instructions → few-shot. Don't default to few-shot for every quality problem.
3. **Constraint-locked stems** — when filtering is forbidden and the bottleneck is investigation time, enrich findings in-place; don't redesign the review process.

---

## 5. Recommended Next Steps

1. **Run Mock Test 2** — adversarial, ~15 fresh questions. Weight toward: Batch tool-loop vs latency, explicit criteria vs few-shot, and any new Domain 4.1/4.6 blends. Log misses in `wrong-answers-cicd.md`.
2. If any of the three Mock 1 categories recurs → targeted drill (5–8 questions) on that pattern only before calling it closed.
3. Stop rule (same as Scenarios 1/3): **≥90% on an adversarial mock with zero 2×-recurring categories** → exam-ready for Scenario 5.
4. After Scenario 5 is closed, prioritize Scenario 6 (Domain 4's remaining unique statements + 5.5/5.6).

---

## 6. Attempt-by-Attempt Detail Reference

Full explanations for every missed question, including the specific trap and the rule extracted from it, are maintained in:

- `mock-tests/wrong-answers/wrong-answers-cicd.md`
- Mock file: `mock-tests/cicd/mock-test-cicd-1.md`

This report will be updated after each new Scenario 5 mock test attempt.
