# Wrong Answer Tracker — Customer Support Agent (CSA)

> Log every missed question here with the **why**, not just the correct answer. The goal is to catch *patterns* in your mistakes, not just memorize individual answers.

**Problem-category legend** (see `scenarios/mock-test/decision-framework.md` if created):

- 🔒 **Safety/Financial/Compliance** → deterministic gate/hook, never system prompt
- 🧠 **Reasoning/Sequencing Quality** → few-shot examples
- 🌀 **Open-ended/Variable Quality** → self-critique / reflection stage
- ⚡ **Efficiency/Latency** → smallest fix to how existing tools are used
- 🔧 **Technique Already Locked by Stem** → stay inside the named technique

---

## Mock Test 1 (`mock-test-csa-1.md`) — 5/20 missed

### Q47 — Multi-issue tool selection accuracy drop (94% → 58%)

- **Your answer:** A — Preprocessing layer with a separate model call to decompose multi-issue messages
- **Correct answer:** C — Add few-shot examples demonstrating correct reasoning and tool sequencing for multi-issue requests
- **Category:** 🧠 Reasoning/Sequencing Quality
- **Why you missed it:** Treated this like a compliance problem needing a hard architectural fix. This is a reasoning-quality gap, not a safety gap — few-shot examples are the simpler, correct first-line fix. Adding a whole preprocessing layer with an extra model call is overkill for a problem prompting can solve.
- **Rule to remember:** Deterministic/architectural fixes are reserved for compliance-critical actions (money, identity). Reasoning/sequencing issues get fixed with few-shot examples first.

---

### Q51 — Agent skips `get_customer`, calls `lookup_order` directly (12% of cases, misidentified accounts)

- **Your answer:** D — Strengthen the system prompt stating verification is mandatory
- **Correct answer:** C — Add a programmatic precondition that blocks `lookup_order`/`process_refund` until `get_customer` returns a verified identifier
- **Category:** 🔒 Safety/Financial/Compliance
- **Why you missed it:** Chose the exact wrong-answer pattern scenario-1 names explicitly: strengthening a system prompt only raises compliance to ~97–99%, not 100%. With "misidentified accounts" and "incorrect refunds" in the stem, this is a financial/identity risk requiring a hard, code-level gate.
- **Rule to remember:** Any stem mentioning identity verification + financial/account consequences → the answer is a programmatic gate, never prompt wording, no matter how "strengthened."

---

### Q52 — Inconsistent explanation quality on complex billing disputes (gaps vary case by case)

- **Your answer:** D — Few-shot examples for five common complex case types
- **Correct answer:** A — Add a self-critique stage where the agent evaluates its draft for completeness
- **Category:** 🌀 Open-ended/Variable Quality
- **Why you missed it:** Missed the phrase "the specific context gaps vary case by case." Few-shot examples only cover the patterns you enumerate; if gaps are unpredictable, hardcoded examples won't generalize. A self-critique step adapts dynamically to whatever is missing in that specific case.
- **Rule to remember:** "Varies case by case" / "inconsistent, no fixed pattern" → self-critique/reflection, not few-shot.

---

### Q53 — Reduce 4+ API loops per resolution (sequential tool calls that could be combined)

- **Your answer:** A — Speculative execution: auto-call likely-needed tools regardless of what was requested
- **Correct answer:** D — Instruct Claude in the prompt to bundle tool requests into one turn
- **Category:** ⚡ Efficiency/Latency
- **Why you missed it:** Picked an answer that executes tools the model never requested — risky in a system with `process_refund` present. The actual root cause is a prompting gap (Claude isn't told it *can* batch requests), not a missing capability requiring speculative/automatic execution.
- **Rule to remember:** Reject any answer where tools fire "automatically" or "regardless of what was requested" — especially when financial tools are in scope. Prefer the smallest prompt-level fix that changes *how* existing tools are used.

---

### Q60 — Ambiguous tool selection; "you decide to add few-shot examples" (stem locks the technique)

- **Your answer:** A — Add explicit "use when" / "don't use when" guidance in tool descriptions
- **Correct answer:** C — Add 4–6 examples targeted at ambiguous scenarios, each with rationale for why one tool was chosen over the other
- **Category:** 🔧 Technique Already Locked by Stem
- **Why you missed it:** Pattern-matched to Q57 (where expanding tool descriptions was correct) instead of reading this stem, which already committed to "add few-shot examples." A is not even a few-shot technique, so it can't be the answer regardless of how good it sounds.
- **Rule to remember:** When the stem says "you decide to do X, which approach to X is best?" — eliminate any option that isn't a version of X, even if it's normally a great answer in a different framing.

---

## Mock Test 2 (`mock-test-csa-2.md`) — 12/15 (80%)

### Q62 — Explicit human request bundled with a quick order-status check

- **Your answer:** D — Separate model call to split the message across two independent agent instances
- **Correct answer:** C — Few-shot examples showing the human request is honored without blocking resolution of the other stated sub-request
- **Category:** 🧠 Reasoning/Sequencing Quality
- **Why you missed it:** Identical failure mode to Q47 in Mock Test 1 — reached for a heavier architectural fix (extra model call, multi-instance split) when the problem was a reasoning/sequencing gap fixable with few-shot examples. This is now a confirmed **recurring** pattern, not a one-off.
- **Rule to remember:** When a fix involves "add a separate model call" or "spin up another agent instance" for what's fundamentally a sequencing/reasoning issue on a single message, pause — few-shot is almost always the correct first-line answer before adding architecture.

### Q69 — `process_refund` reason parameter causing downstream reporting errors

- **Your answer:** C — Few-shot examples showing correctly categorized reasons
- **Correct answer:** B — Change the `reason` parameter to an enum of valid categories in the tool schema
- **Category:** 🔧 Technique Already Locked by Stem (exclusion variant)
- **Why you missed it:** The stem explicitly said "the fix should happen at the tool interface level, not through prompting." Few-shot examples are a prompting technique, so they were ruled out before evaluating options. This is the same family as the Q60 mistake, but the lock was an *exclusion* ("not through prompting") rather than an *inclusion* ("add few-shot examples"), which made it easier to miss.
- **Rule to remember:** Locked-technique stems can rule a technique **in** or **out**. Scan for both phrasings — "you decide to do X" and "should happen at Y level, not through Z" are the same trap in different words.

### Q68 — Agent misusing `lookup_order` to answer general policy questions

- **Your answer:** A — Assume it's prompt-based, add few-shot examples immediately
- **Correct answer:** B — First check whether a dedicated policy-lookup tool exists at all
- **Category:** 🆕 Diagnose-Before-Prescribing (new pattern, not previously logged)
- **Why you missed it:** Jumped straight to a fix-type without first diagnosing root cause. If the agent has no proper tool for the task, no amount of prompting or few-shot tuning fixes a missing capability — it can only make the misuse marginally less bad.
- **Rule to remember:** Before choosing *how* to fix a tool-selection/efficiency problem, first ask *whether the agent has the right tool for the job at all*. A missing-capability gap looks like a prompting problem on the surface but isn't one.

---

## Mock Test 3 (`mock-test-csa-3.md`) — 17/19 (89%)

### Q5 — Multi-issue message: resolvable billing question + policy-gap price match

- **Your answer:** A — Escalate both issues together, since one of them requires escalation
- **Correct answer:** B — Resolve the billing question autonomously using the shared `verified_id`; escalate only the price-match question with a structured handoff
- **Category:** 🆕 All-or-Nothing Escalation Scope (new pattern)
- **Why you missed it:** Let one sub-issue's escalation need "contaminate" the whole request instead of decomposing and resolving what's resolvable. This is a scope-calibration cousin of your earlier architecture-overreach pattern — same instinct ("when part of it needs a stronger response, apply the stronger response to everything") but showing up in escalation decisions instead of tool/architecture choices.
- **Rule to remember:** Multi-issue requests should be decomposed *before* deciding on escalation. Escalate only the specific sub-issue that needs it; resolve the rest using shared context (same `verified_id`, no re-verification, no blanket escalation).

### Q18 — Retrying a transient failure 5 times with no customer request and clear policy

- **Your answer:** A — Yes, keep retrying since policy is clear and the customer hasn't asked for a human
- **Correct answer:** B — No; inability to make meaningful progress after reasonable retries is itself a valid escalation trigger
- **Category:** 📚 Knowledge Gap — Escalation Trigger List
- **Why you missed it:** Not a reasoning trap this time — a genuine gap in the mental checklist of valid escalation triggers. You had "explicit customer request" and "policy gap" covered, but "repeated failure / no progress after reasonable retries" is a third, independent trigger that doesn't require either of the other two conditions.
- **Rule to remember:** Escalation triggers = (1) explicit customer request, (2) policy is silent/ambiguous, (3) the agent cannot make progress after reasonable retries — even if policy is clear and the customer hasn't asked. Any one of the three is sufficient on its own.

---

## Targeted Drill (`mock-test-csa-4-targeted-drill.md`) — 9/10 (90%)

> Isolated drill testing only architecture-overreach and escalation-scope, per `scenario-1-report.md` §5 recommendation.

### Q5 — Multi-part billing question with related sub-issues investigated rigidly in isolation

- **Your answer:** C — Force a fixed-sequence `tool_choice` pattern so every multi-part case follows an identical order regardless of content
- **Correct answer:** B — Add few-shot examples showing the agent recognizing related sub-issues and investigating them together
- **Category:** 🆕 Over-Rigidification (new pattern, distinct from tracked ones)
- **Why you missed it:** Neither architecture-overreach nor escalation-scope — this is the opposite failure: eliminating adaptive flexibility by forcing one mechanical order for all cases, when the actual fix is teaching the model to *recognize* relatedness dynamically via few-shot.
- **Rule to remember:** "Force a fixed/identical sequence regardless of content" is as much a red flag as "add a new model call" — both remove judgment from a place that needs it, just in opposite directions (over-rigid vs. over-architected). Isolated single occurrence — not yet a confirmed pattern; watch for recurrence.

**Result: architecture-overreach (0/5 relevant questions missed) and escalation-scope (0/4 relevant questions missed) both fully clean. Both patterns confirmed closed per the drill's 9-10/10 threshold.**

---

## Recurring Patterns Across Mistakes

1. **Reaching for a bigger/architectural fix when a simpler prompting fix is correct** (Mock 1: Q47, Q53 · Mock 2: Q62) — **confirmed recurring across two mock tests.** This is your #1 systemic gap. Check if the problem is actually safety-critical before assuming it needs a hard architectural solution.
2. **Missing qualifier phrases in the stem** like "varies case by case" (Mock 1: Q52) that change the correct category entirely.
3. **Pattern-matching to a previous question's answer-shape** instead of re-reading the current stem fresh (Mock 1: Q60, Q51 · Mock 2: Q69 — locked-technique traps, both inclusion and exclusion phrasing).
4. **Prescribing a fix before diagnosing root cause** (Mock 2: Q68) — new pattern. Jumping to "add few-shot examples" or "add a hook" without first confirming the agent has the right tool/capability for the task.
5. **All-or-nothing scope on multi-part problems** (Mock 3: Q5) — a scope-calibration variant of pattern #1: when one sub-issue needs a stronger response (escalation), the stronger response gets over-applied to the whole request instead of just the sub-issue that needs it.

## Score Trend

| Mock Test | Score | Missed |
|---|---|---|
| Mock 1 | 15/20 (75%) | Q47, Q51, Q52, Q53, Q60 |
| Mock 2 | 12/15 (80%) | Q62, Q68, Q69 |
| Mock 3 | 17/19 (89%) | Q5, Q18 |
| Targeted Drill | 9/10 (90%) | Q5 (new, isolated pattern) |

**Trend: 75% → 80% → 89% → 90% (isolated drill).** Consistent upward trajectory. Mock 3's misses were milder than prior rounds — one scope-calibration slip (a variant of a known pattern) and one straightforward knowledge gap (easily closed), not the deeper reasoning-vs-architecture confusion that dominated Mocks 1–2. The targeted drill confirms **both tracked patterns (architecture-overreach, escalation-scope) are now clean** — the single miss was a new, distinct, isolated pattern (over-rigidification), not a recurrence of either tracked gap.

## Action Items Before Next Mock Test

- [ ] Before picking an answer, name the problem category out loud (🔒/🧠/🌀/⚡/🔧)
- [ ] When a fix involves adding a new model call, new agent instance, or new piece of infrastructure — stop and check if a much simpler few-shot/prompting fix solves it first.
- [ ] Before selecting a fix-type, ask: "does the agent even have the right tool/capability for this task?" — diagnose before prescribing.
- [ ] Re-read the stem once more after choosing an answer to check for locked-in techniques stated as either inclusions ("add X") or exclusions ("not through Y")
- [ ] **New:** On multi-issue requests, decompose first — resolve what's resolvable, escalate only the specific sub-issue that needs it. Don't let one part's escalation need pull the whole request into escalation.
- [ ] **New:** Escalation triggers = explicit request OR policy gap OR inability to make progress after reasonable retries. Any one alone is sufficient — none require the others to also be true.
- [ ] Don't let the previous question's correct answer bias the current one
- [x] Architecture-overreach and escalation-scope patterns confirmed closed via targeted drill (9/10, both patterns 0 misses) — Scenario 1 status: **exam-ready**
- [ ] **Watch (new, isolated):** "Force a fixed/identical sequence regardless of content" is an over-rigidification trap — same family of error as architecture-overreach (removing needed judgment) but in the opposite direction. Only one occurrence so far; not yet a confirmed pattern.
