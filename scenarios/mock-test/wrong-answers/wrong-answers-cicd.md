# Wrong Answer Tracker — Claude Code for CI/CD (CICD)

> Log every missed question here with the **why**, not just the correct answer. The goal is to catch *patterns* in your mistakes, not just memorize individual answers.

**Problem-category legend:**

- 🏳️ **Vague Precision Instruction** → "be conservative" / "high-confidence only" / "be careful" / vague accuracy checks instead of explicit categorical criteria for what to flag vs. skip
- 🔁 **Self-Review Bias** → same session that generated code reviewing its own output, instead of an independent instance with no generation context
- 📦 **Wrong API for Latency** → Batch API for blocking pre-merge checks (no SLA) instead of Synchronous API; or Synchronous for overnight latency-tolerant jobs
- 🔧 **Batch Incompatible with Multi-Turn Tools** → treating Batch latency as the primary blocker when the real technical limit is no mid-request tool execution / return of results
- 🏷️ **Probabilistic Structured Output** → "please respond in JSON" / `tool_choice: "auto"` / CLAUDE.md format examples instead of forced CLI `--output-format json` + `--json-schema` (or forced `tool_use`)
- 🧭 **Missing Headless Flag** → interactive `claude "..."` in CI (hangs) instead of `claude -p`
- 📋 **Missing Prior Context on Re-run** → re-posting all findings / duplicate tests because prior findings or existing test files weren't injected into context
- ⏱️ **Process Redesign vs. In-Finding Triage Info** → changing how findings are categorized or filtered when the stem's bottleneck is investigation time and filtering is explicitly disallowed — put rationale/confidence *inside* each finding instead

---

## Practice Mock (`scenario-5-cicd.md §7`) — 18/18 (100%)

No misses. Full marks on the first attempt at the scenario's own practice question bank.

**Coverage confirmed correct (all task statements):**

| Task Statement | Questions | Result |
|---|---|---|
| 3.6 — CI/CD Integration (`-p`, `--output-format json`, prior findings, existing tests) | Q1–Q4 | 4/4 |
| 3.1 — CLAUDE.md as Pipeline Context | Q5–Q6 | 2/2 |
| 4.1 — Explicit Criteria to Reduce False Positives | Q7–Q9 | 3/3 |
| 4.2 — Few-Shot Prompting for Consistency | Q10–Q12 | 3/3 |
| 4.3 — Structured Output via Tool Use / JSON Schema | Q13–Q15 | 3/3 |
| 4.6 — Multi-Instance and Multi-Pass Review | Q16–Q18 | 3/3 |

**Your answers:** `BCCCBBBCBCBCBCCBBB` — matched answer key exactly.

---

## Mock Test 1 (`mock-test-cicd-1.md`) — 12/15 (80%)

### Q18 — Iterative tool-calling code review; primary Batch API limitation

- **Your answer:** D — The batch processing latency of up to 24 hours is too slow for pull request feedback, although the workflow would otherwise function.
- **Correct answer:** B — The asynchronous model cannot execute tools mid-request and return results for Claude to continue analysis.
- **Category:** 🔧 Batch Incompatible with Multi-Turn Tools
- **Why you missed it:** Latency *is* a real Batch constraint and a valid reason not to use Batch for PR feedback — but the stem asks for the **primary technical limitation** for *this* workflow specifically. The workflow is multi-turn tool calling (request related files → get results → continue analysis). Batch is fire-and-forget: it cannot intercept a tool call mid-request, execute it, and feed results back for the next reasoning turn. Option D's closing clause ("although the workflow would otherwise function") is the tell — that clause is false. Even with instant Batch completion, iterative tool-calling would still be impossible.
- **Rule to remember:** When a stem describes multi-turn tool use *and* asks about Batch limitations, check **tool-loop incompatibility first** before defaulting to the latency distractor. Latency is the right answer for "blocking vs overnight" matching questions; mid-request tool execution is the right answer when the workflow itself requires iterative tool round-trips. Don't let a familiar, always-true Batch fact (≤24h) override a stem-specific technical blocker.

---

### Q22 — Comment/docstring accuracy review flags TODOs but misses stale behavior claims

- **Your answer:** B — Add few-shot examples of misleading comments to help the model recognize similar patterns in the codebase.
- **Correct answer:** D — Specify explicit criteria: flag comments only when the behavior they claim contradicts the code's actual behavior.
- **Category:** 🏳️ Vague Precision Instruction
- **Why you missed it:** Reached for few-shot (often correct when *format/consistency* is the problem) when the stem's root cause is a **vague criterion** ("check that comments are accurate and up to date"). That vagueness produces both false positives (TODOs, simple descriptions) and false negatives (comments whose claimed behavior no longer matches the code). Few-shot examples of "misleading comments" help pattern recognition but do not define *what counts as a problem* — explicit categorical criteria do. This is the same Domain 4.1 judgment as "be conservative" failures: replace subjective/vague instructions with a precise flag-vs-skip rule.
- **Rule to remember:** Ask what failed first. If output *format* is inconsistent despite detailed instructions → few-shot. If the model is flagging the wrong *things* (noise) and missing the right ones (true issues) under a vague accuracy/quality instruction → **explicit criteria**, not more examples. Few-shot teaches recognition of patterns you show; criteria define the decision boundary for patterns you haven't enumerated.

---

### Q28 — 40% FP rate; bottleneck is investigation time; stakeholders reject filtering findings

- **Your answer:** C — Categorize findings as "blocking issues" vs "suggestions," with different review requirements by level.
- **Correct answer:** A — Require Claude to include its rationale and confidence estimate directly in each finding.
- **Category:** ⏱️ Process Redesign vs. In-Finding Triage Info
- **Why you missed it:** Blocking-vs-suggestion categorization changes the *review process* (different handling rules by tier) but does not put triage information where the stem says the cost is — inside each finding, so developers don't have to click through to read rationale. The stem also locks two constraints: (1) investigation time is the bottleneck, (2) stakeholders rejected any approach that *filters* findings before developers see them. Options B and D filter; C doesn't filter but also doesn't solve click-to-read overhead. A keeps every finding visible *and* embeds rationale + confidence so triage happens in-place.
- **Rule to remember:** When the stem names a specific bottleneck ("investigation time / click into each finding") *and* forbids filtering, prefer answers that **enrich each finding with triage metadata** (rationale, confidence) over answers that redesign workflow tiers, post-process suppressions, or confidence-based hiding. Match the fix to the named bottleneck and the named constraint.

---

## Recurring Patterns Across Mistakes

1. **Familiar Batch fact overriding stem-specific blocker** (Mock 1: Q18) — defaulting to "Batch is slow / no SLA" when the stem's workflow is multi-turn tool calling. Latency questions and tool-loop questions look similar; screen for "tool call → result → continue" language before picking latency.
2. **Few-shot when the real gap is criteria** (Mock 1: Q22) — reaching for few-shot (strong default from other Scenario 5 questions) when the stem's failure mode is wrong *what* is flagged, not inconsistent *how* findings are formatted. Explicit criteria vs. few-shot is a Domain 4.1 vs 4.2 fork — diagnose which axis failed.
3. **Workflow redesign when the bottleneck is in-finding info** (Mock 1: Q28) — inventing a process/tiering fix for a triage-speed problem under a no-filter constraint. Put the missing info in the finding itself.

## Score Trend

| Mock Test | Score | Missed |
|---|---|---|
| Practice Mock | 18/18 (100%) | — |
| Mock Test 1 | 12/15 (80%) | Q18, Q22, Q28 |

**Trend: 100% → 80%.** Expected practice-bank-to-adversarial drop (smaller than S1/S3's ~73% first-mock drops). Three distinct, non-recurring categories so far — none yet confirmed as a systemic pattern. Next mock should stress-test whether Q18's Batch-tool-loop distinction and Q22's criteria-vs-few-shot fork reappear under fresh wording.
