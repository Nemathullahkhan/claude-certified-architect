# Mock Test: Customer Support Agent (CSA) — Set 2

> Built specifically around the gap patterns identified in `wrong-answers/wrong-answer-customer-support-agent.md`: safety-vs-prompting confusion, reasoning-quality vs. architectural overreach, fixed-pattern vs. open-ended quality issues, efficiency fixes that overreach into speculative execution, and stems that lock in a technique. Distractors are intentionally tempting — read every stem twice before answering.

---

## Question 61 — Customer Support Agent

Production data shows that when a customer's message contains a dollar figure (e.g., "I was charged $89 twice"), the agent correctly parses and refunds the amount 99.6% of the time using system-prompt instructions that say "always double-check the charged amount against `lookup_order` before refunding." Leadership wants this to be 100% reliable since it directly affects real money movement, no matter how high current compliance already is.

**What should you do?**

- **A.** Leave it as is — 99.6% compliance is well within acceptable production tolerance for a support agent.
- **B.** Add a second, redundant system-prompt instruction reinforcing the double-check, since one instruction alone left a small gap.
- **C.** Replace the prompt-based double-check with a `PreToolUse` hook that programmatically compares the requested refund amount against `lookup_order` data and blocks the call on mismatch.
- **D.** Add few-shot examples showing the agent correctly cross-referencing charged amounts before refunding.

---

## Question 62 — Customer Support Agent

Your agent's tool-selection accuracy for single-issue requests is 96%. For requests where the customer explicitly says "please have a human look into this, but also can you check my order status," accuracy drops to 61%, with the agent sometimes escalating instead of checking the order, and sometimes checking the order and never escalating.

**What is the most effective fix?**

- **A.** Add a preprocessing classifier that flags any message containing "human" and force-routes it to `escalate_to_human`, skipping other tool calls.
- **B.** Since the customer explicitly requested a human, treat this as a mandatory escalation case and add a hook that forces `escalate_to_human` any time the word "human" appears, before any other tool executes.
- **C.** Add few-shot examples showing that an explicit human request is handled immediately and does not block resolving other stated sub-requests (e.g., checking order status) using existing tools first if quick, before or alongside escalation.
- **D.** Treat this as a multi-issue decomposition problem and add a separate model call that splits the message, sending the escalation portion and the order-status portion to two independent agent instances.

---

## Question 63 — Customer Support Agent

Your agent's post-resolution customer satisfaction scores are inconsistent for subscription-downgrade requests specifically. Sometimes the agent explains proration correctly and mentions the next billing date; other times it gives a technically correct downgrade confirmation but omits proration details or billing date — the omitted detail is different each time, and you cannot identify a consistent missing field.

**What is the most effective fix?**

- **A.** Add 6–8 few-shot examples of subscription downgrades that explicitly demonstrate mentioning proration and billing date every time.
- **B.** Add a self-critique step where the agent checks its draft downgrade confirmation against a completeness checklist (proration, billing date, next steps) before sending it to the customer.
- **C.** Upgrade the system prompt with a bolded instruction: "ALWAYS mention proration and next billing date in downgrade confirmations."
- **D.** Create a `format_downgrade_confirmation` tool that the agent must call, which auto-generates the confirmation text server-side using a template.

---

## Question 64 — Customer Support Agent

Your agent averages 5 tool calls per resolution for straightforward refund requests, most of which involve `get_customer` and `lookup_order` being called in separate turns even though the customer's message contains both the account email and the order ID up front. Latency per resolution is a concern.

**What is the most effective fix?**

- **A.** Modify the agentic loop to always execute `get_customer` and `lookup_order` together automatically whenever either is requested, regardless of whether the model asked for both.
- **B.** Instruct the model in the system prompt that when both an identifier and an order ID are present in the customer's message, it should request both tools within the same turn.
- **C.** Reduce the number of tools by merging `get_customer` and `lookup_order` into a single `get_customer_and_order` tool that always runs both lookups.
- **D.** Increase `max_tokens` so the model has more room to plan both tool calls before responding.

---

## Question 65 — Customer Support Agent

You decide to fix an ambiguous tool-selection problem by rewriting the `get_customer` and `lookup_order` tool descriptions to explicitly state when each should and should not be used. Which revision most effectively achieves this goal?

**Which revision is most effective?**

- **A.** `get_customer`: "Retrieves customer account data." `lookup_order`: "Retrieves order data." Keep descriptions short so the model isn't overloaded with text.
- **B.** `get_customer`: "Looks up customer identity by email/phone; returns `verified_id` required before any order or financial operation. Do not use for order-specific questions — use `lookup_order` instead." `lookup_order`: "Retrieves order status/details by order ID; requires `verified_id` from `get_customer`. Do not use this to verify customer identity."
- **C.** Add a single shared description at the top of the system prompt explaining both tools together, so the model sees the contrast in one place rather than in two separate tool schemas.
- **D.** `get_customer`: "Use this tool first, always." `lookup_order`: "Use this tool second, always." Simple sequencing removes ambiguity entirely.

---

## Question 66 — Customer Support Agent

A customer's message states: "Cancel my subscription immediately, I've already spoken to three agents about this and nobody has fixed it." The agent has all the tools needed to process the cancellation itself and policy allows self-service cancellation with no approval needed.

**What should the agent do?**

- **A.** Escalate to a human immediately — a customer citing three prior contacts is a strong signal of an unresolved, complex issue that a human should handle holistically.
- **B.** Process the cancellation using available tools since it's within policy and the agent has everything needed to resolve it; do not escalate solely based on frustration signals or prior-contact count when the customer has not explicitly requested a human.
- **C.** Run a sentiment analysis check on the message; if frustration score exceeds a threshold, escalate regardless of whether the task is otherwise resolvable.
- **D.** Ask the customer to confirm they want to proceed with cancellation given their history, then escalate afterward for a retention specialist to review.

---

## Question 67 — Customer Support Agent

`lookup_order` occasionally returns a response where the `status` field is present but the `estimated_delivery` field is `null`, because some orders (e.g., digital goods) never have a delivery estimate. In other cases, `estimated_delivery` is `null` because a backend timeout prevented the field from populating. The agent currently treats both cases identically, sometimes telling customers "no delivery estimate available" when the real issue was a timeout that should be retried.

**What is the most effective fix?**

- **A.** Update `lookup_order`'s error handling to return a structured error (e.g., `{ isError: true, errorCategory: "transient", isRetryable: true }`) when the timeout occurs, distinct from a valid response where `estimated_delivery` is legitimately `null`.
- **B.** Add a system-prompt instruction telling the agent to ask the customer to try again later whenever `estimated_delivery` is `null`.
- **C.** Modify `lookup_order` to always return a non-null placeholder string like `"N/A"` for `estimated_delivery` so the field is never ambiguous.
- **D.** Add a `PostToolUse` hook that retries `lookup_order` automatically any time `estimated_delivery` is `null`, up to 3 times, before returning the result to the model.

---

## Question 68 — Customer Support Agent

Your agent handles an average support case in 6 tool calls. For a specific subset of cases — customers disputing a charge who also ask "and can you tell me your refund policy in general" — the agent averages 14 tool calls because it repeatedly calls `lookup_order` on the same order ID multiple times while also trying to look up policy information using tools not designed for that purpose.

**What is the most effective first step to diagnose this?**

- **A.** Assume the issue is prompt-based and immediately add few-shot examples showing efficient tool use for this exact case type.
- **B.** Check whether the agent has access to a proper policy-lookup or knowledge-retrieval tool at all — if general policy questions have no dedicated tool, the agent may be misusing `lookup_order` and other tools to compensate.
- **C.** Add a `PreToolUse` hook that blocks repeated calls to `lookup_order` with the same order ID within a single conversation.
- **D.** Increase the iteration cap so the agent has more room to eventually resolve the case, even if inefficiently.

---

## Question 69 — Customer Support Agent

Your `process_refund` tool currently accepts a free-text `reason` parameter. Analysis shows Claude sometimes writes reasons that don't match any internal accounting category, causing downstream reporting errors. You decide the fix should happen at the tool interface level, not through prompting.

**Which tool interface change best fixes this?**

- **A.** Add a system-prompt instruction listing the seven valid reason categories and telling Claude to pick one.
- **B.** Change the `reason` parameter to an enum of the seven valid accounting categories, so Claude can only supply a value the schema allows.
- **C.** Add few-shot examples showing `process_refund` calls with correctly categorized reasons for common scenarios.
- **D.** Keep `reason` as free text but add a `PostToolUse` hook that maps common free-text phrases to categories after the refund has already been processed.

---

## Question 70 — Customer Support Agent

You are deciding how to reduce tool-selection confusion between `get_customer` and `lookup_order` and have committed to improving the tool descriptions rather than adding few-shot examples or hooks, since descriptions are reviewed by a separate team that owns tool schemas.

**Given this constraint, which change is most effective?**

- **A.** Add 8 few-shot examples to the system prompt demonstrating correct tool choice, since this is more reliable than description edits alone.
- **B.** Implement a `PreToolUse` hook that reroutes miscalled tools to the correct one automatically.
- **C.** Expand each tool's description with explicit purpose, required inputs, return values, and an explicit "do not use this for X — use `<other tool>` instead" boundary statement.
- **D.** Rename both tools to more intuitive names, which alone typically resolves ambiguous selection issues without further description changes.

---

## Question 71 — Customer Support Agent

A 25-turn conversation involves a customer disputing charges across three separate orders. By turn 20, the agent's summarized context describes the situation as "customer has multiple billing disputes across orders," without the specific order IDs or amounts. The customer then asks, "what's the status of the refund for the second order?" and the agent cannot answer correctly.

**What is the most effective structural fix?**

- **A.** Instruct the agent to re-ask the customer for all order IDs and amounts whenever ambiguity arises, to avoid relying on memory.
- **B.** Maintain a `CASE_FACTS` block listing each order ID, amount, and status, updated as new tool results arrive, and injected verbatim at the start of every prompt outside the summarized history.
- **C.** Disable summarization entirely for conversations involving more than one order, to preserve full fidelity.
- **D.** Increase the summarization model's instructions to "be more detailed" so summaries retain more specifics.

---

## Question 72 — Customer Support Agent

Your support agent has `tool_choice` set to `"auto"`. Logs show that for messages like "hi, I have a question," the agent correctly responds conversationally without calling a tool. But for messages like "my order #4521 hasn't arrived," the agent sometimes responds with "Let me check on that for you!" and ends its turn without actually calling `lookup_order`, requiring the customer to prompt it again.

**What is the most effective fix for the second behavior specifically, without breaking the first?**

- **A.** Set `tool_choice` to `"any"` globally so the model must always call a tool, on every single turn including greetings.
- **B.** Set `tool_choice` to force `lookup_order` specifically on every turn, ensuring the delay never happens again.
- **C.** Keep `tool_choice: "auto"` for general conversation, but review whether the loop is terminating on `end_turn` when the assistant's text merely promises future action — the real issue may be the loop stopping too early rather than the model failing to request the tool at all.
- **D.** Add a few-shot example of a greeting-only message with no tool call, since this is purely a prompting gap.

---

## Question 73 — Customer Support Agent

A customer asks for a refund on an order that was placed 95 days ago. Your policy states refunds are only valid within 90 days of purchase, with no stated exceptions. The customer has no special circumstances mentioned (no defect, no shipping error).

**What should the agent do?**

- **A.** Escalate to a human, since any refund-adjacent request carries financial risk and should default to human review.
- **B.** Deny the refund per policy and clearly explain the 90-day window has passed; this is a clear, unambiguous policy application, not a gap requiring escalation.
- **C.** Approve the refund as a goodwill gesture, since the request is only 5 days past the window and this improves customer satisfaction.
- **D.** Ask the customer to explain why they waited so long before deciding whether to approve or deny.

---

## Question 74 — Customer Support Agent

You want to guarantee that every call to `escalate_to_human` includes a non-empty `root_cause` field, since human agents have reported receiving handoffs with this field missing or blank despite system-prompt instructions saying it's required.

**What is the most reliable fix?**

- **A.** Make `root_cause` a required field in the `escalate_to_human` tool's input schema so the API rejects calls missing it, forcing Claude to supply a value.
- **B.** Add a stronger, bolded instruction in the system prompt: "root_cause is MANDATORY and must never be blank."
- **C.** Add few-shot examples of well-formed escalations with detailed root-cause explanations for five common scenarios.
- **D.** Add a `PostToolUse` hook that logs a warning when `root_cause` is blank, so the team can follow up with the customer later.

---

## Question 75 — Customer Support Agent

Your team debates whether to fix a tool-selection issue with few-shot examples or a routing classifier. The issue: for the single specific phrase pattern "check on my stuff," the agent picks the wrong tool nearly every time, but all other phrasings work correctly at 95%+ accuracy. This exact phrase appears in about 0.3% of all conversations.

**What is the most proportionate fix?**

- **A.** Build and deploy a dedicated routing classifier trained on this failure pattern, given how clearly identifiable it is.
- **B.** Add 1–2 few-shot examples covering this specific ambiguous phrasing to the existing examples, since it's a narrow, low-frequency edge case that doesn't warrant new infrastructure.
- **C.** Rewrite both tool descriptions entirely from scratch, since any single failure mode indicates the descriptions are fundamentally flawed.
- **D.** Ignore the issue, since 0.3% frequency is statistically negligible and not worth any engineering effort.

---

## Answer Key

**Q61: C** — Financial/compliance accuracy (money movement) must be deterministic regardless of how high current probabilistic compliance already is. 99.6% is not "always." A redundant prompt instruction (B) or more few-shot examples (D) are still probabilistic and cap out similarly. A `PreToolUse` hook comparing amounts guarantees the check fires every time.

**Q62: C** — The customer bundled an explicit human request with a quick informational task. Explicit human requests must be honored, but that doesn't mean *other* stated needs must be blocked or ignored — the reasoning/sequencing gap (handling both correctly) is a few-shot problem, not a keyword-triggered hard override (A, B are keyword-matching traps that would misfire on any message merely containing "human") and not a multi-agent architecture problem (D is overkill for a two-part message).

**Q63: B** — "The omitted detail is different each time" is the open-ended/variable-quality signal — few-shot (A) and stronger wording (C) only cover fixed, anticipated patterns. A self-critique completeness check adapts to whatever is missing in that instance. D removes agent judgment entirely, which is a bigger architecture change than necessary and doesn't fit an LLM-generated confirmation with situational nuance.

**Q64: B** — The model already has both tools available and both needed identifiers were provided; the gap is instructional, not architectural. A (auto-executing tools regardless of request) is the speculative-execution trap. C is an unnecessary architecture change. D doesn't address the root cause (the model isn't token-constrained, it's not being told batching is expected/appropriate here).

**Q65: B** — Directly gives purpose, required inputs, and explicit "do not use for X" boundaries in each tool's own description — this is the standard for differentiating overlapping tools. A is too vague to help. C moves the differentiation out of the tool schema (contradicts the scenario's own constraint in Q70-style questions, and is also just weaker practice generally). D is a sequencing rule dressed up as a description, which breaks down the moment tools legitimately need to be called out of that fixed order.

**Q66: B** — Prior-contact count and expressed frustration are not valid escalation triggers on their own; escalation should only fire on explicit request or policy gap. The task is fully resolvable within policy using existing tools, so proceed. A and C encode the sentiment/frustration-based escalation anti-pattern named directly in scenario-1. D adds unnecessary friction and still escalates without a valid trigger.

**Q67: A** — Distinguishing a legitimate null (no delivery estimate expected) from a transient failure (timeout) requires structured error metadata at the source, exactly like the classic `lookup_order` timeout pattern. B lets a transient failure be silently misreported to the customer. C erases the distinction entirely by making all nulls looks intentional. D silently retries without informing the model of what happened, and could mask a real persistent outage.

**Q68: B** — Before reaching for a prompting or hook-based fix, diagnose whether the actual root cause is a missing capability (no policy-lookup tool exists), which is a tool-design gap, not a tool-selection or efficiency gap. A jumps to a fix without diagnosing the cause. C treats a symptom (repeated calls) without addressing why they're happening. D just tolerates the inefficiency.

**Q69: B** — The question explicitly asks for a tool-interface-level fix, not prompting. An enum constrains the schema itself so invalid categories are structurally impossible, not just discouraged. A and C are prompting-level fixes despite the stem ruling that out. D is reactive (after the bad data has already been recorded) rather than preventive.

**Q70: C** — The stem explicitly locks in "improving tool descriptions" as the chosen technique (analogous to the Q60-style locked-technique trap). A and B propose different techniques entirely (few-shot, hooks) and should be eliminated regardless of merit. D is a description change in name only — renaming without adding usage boundaries doesn't reliably fix ambiguity on its own.

**Q71: B** — A `CASE_FACTS`-style block per order (IDs, amounts, statuses) kept outside summarization is the direct fix for exactly this failure mode — multiple concurrent facts getting flattened into a vague summary. A adds friction and a poor customer experience. C is a blunt, costly overcorrection. D is still probabilistic — the summarization model can still drop specifics under "be more detailed" instructions.

**Q72: C** — This mirrors the classic loop-termination trap: text like "Let me check on that for you!" can be followed immediately by a tool call in the same turn, so the real bug is likely the loop treating any assistant text as a stopping signal rather than checking `stop_reason` correctly. A and B break the correctly-working greeting behavior by forcing tool calls unconditionally. D assumes a prompting gap without first ruling out a loop-logic bug — diagnose before prescribing.

**Q73: B** — A clear, unambiguous policy (90-day window, explicitly stated, no exceptions, no special circumstances present) should be applied directly — this is not a policy gap. Escalating everything refund-adjacent (A) ignores that policy is not silent here. Approving anyway (C) violates explicit policy without authority. Asking for justification (D) implies the decision hinges on something policy doesn't allow for.

**Q74: A** — A required field in the tool's input schema is enforced by the API/tooling layer itself — Claude cannot successfully call the tool without supplying it, which is deterministic. B is the classic probabilistic-strengthening trap. C only covers anticipated scenarios. D detects the problem after the fact instead of preventing it.

**Q75: B** — Proportionate response to a narrow, low-frequency, well-identified edge case: a couple of targeted few-shot examples. A is disproportionate infrastructure for a 0.3% edge case. C overreacts to one failure mode by discarding descriptions that work well for 95%+ of cases. D ignores a known, fixable, if small, reliability gap — "low frequency" isn't the same as "acceptable to ignore" when the fix is cheap.
