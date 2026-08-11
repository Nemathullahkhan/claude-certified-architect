# CCAF Mock Exam — Scenario 1: Customer Support Resolution Agent — Set 3

> 19 questions covering the full Scenario 1 domain set (Agentic Architecture, Tool Design & MCP, Context Management & Reliability). Questions are presented in random order without domain labels — identify the underlying issue yourself before picking an answer. Format: single best answer. Answer key with explanations follows the question bank — don't peek early.

---

## Question 1 — Customer Support Agent

Your agentic loop checks `stop_reason`, but a developer also added a safeguard: the loop exits early if the most recent tool result contains a field `"success": true`, regardless of `stop_reason`. You notice the loop sometimes stops right after `get_customer` returns successfully — before `process_refund` is ever called, even though the customer clearly needs a refund.

**What's the correct fix?**

- **A.** Remove the extra `success` field check entirely and terminate solely based on `stop_reason === "end_turn"`
- **B.** Change `get_customer` to return `"success": false` so the loop doesn't misinterpret it as completion
- **C.** Increase `max_tokens` so the model has room to continue after `get_customer` returns
- **D.** Disable prompt caching on tool results so the `success` field isn't reused across turns

---

## Question 2 — Customer Support Agent

Which of the following is a valid signal for terminating the support agent's loop?

**Which signal is valid?**

- **A.** `stop_reason === "end_turn"`
- **B.** The assistant's message contains a closing phrase like "Is there anything else I can help with?"
- **C.** The loop has executed exactly 3 tool calls
- **D.** 30 seconds have passed without a new customer message

---

## Question 3 — Customer Support Agent

Audits show that `escalate_to_human` handoffs consistently include `customer_id`, `issue_summary`, `root_cause`, and `refund_amount` — but `recommended_action` is missing about 20% of the time, because the model sometimes forgets to mention it in its summary.

**What's the most reliable fix?**

- **A.** Add a few-shot example to the system prompt showing a complete handoff summary including `recommended_action`
- **B.** Make `recommended_action` a required parameter in the `escalate_to_human` tool schema, populated by code from the resolved state rather than left to the model's free-form summary
- **C.** Add a bolded instruction: "ALWAYS include a recommended action when escalating"
- **D.** Have a second Claude instance review each handoff summary before it's sent to the human agent

---

## Question 4 — Customer Support Agent

`get_customer` occasionally returns a response where the record was found but `verified_id` is `null` (e.g., an unverified or partial match). Your `process_refund` prerequisite gate currently only checks "was `get_customer` called at some point in this conversation?" — not what it actually returned. As a result, refunds sometimes fire against unverified customers.

**What's the correct fix?**

- **A.** Check the HTTP status code of the `get_customer` call instead of whether it was called
- **B.** Change the prerequisite gate to check `state.verified_id !== null`, not merely that `get_customer` was invoked
- **C.** Call `get_customer` twice to double-check the result
- **D.** Add a system prompt note: "Only proceed if `get_customer` found a real customer"

---

## Question 5 — Customer Support Agent

A customer's message contains two concerns: a billing question (which the agent can resolve using policy) and a request for a price match on a currently-unsupported promotion type (a policy gap).

**What's the correct handling?**

- **A.** Escalate both issues together, since one of them requires escalation
- **B.** Resolve the billing question autonomously using the shared `verified_id`, and escalate only the price-match question with a structured handoff — reusing the same identity context, not re-verifying
- **C.** Resolve the billing question, then ask the customer to open a separate ticket for the price-match request
- **D.** Escalate the price-match question first, then re-run `get_customer` before addressing billing to be safe

---

## Question 6 — Customer Support Agent

Your `PreToolUse` hook blocks `process_refund` when `amount > 500`. In production, some refunds of $650 are still going through. Investigation shows `lookup_order` sometimes returns `amount` as the string `"650.00"` instead of a number, and the hook's comparison silently evaluates as false against a string.

**What's the correct fix?**

- **A.** Move the threshold check into the system prompt so the model catches the type mismatch itself
- **B.** Have the `PreToolUse` hook coerce/validate `amount` to a number before applying the threshold comparison, rejecting the call if the amount can't be reliably parsed
- **C.** Ask `lookup_order` to always return `success: false` when `amount` is a string
- **D.** Increase the threshold to $700 to build in a safety margin for parsing errors

---

## Question 7 — Customer Support Agent

`lookup_order` returns order status as inconsistent representations depending on backend region: some return `2` (numeric code), others return `"shipped"` (string).

**Where should this be reconciled into a single consistent format before the model reasons about it?**

- **A.** In the system prompt, by explaining both possible formats
- **B.** In a `PreToolUse` hook, before `lookup_order` is called
- **C.** In a `PostToolUse` hook, which normalizes the result before it's added to the conversation
- **D.** In `escalate_to_human`'s description, so the human agent knows to expect either format

---

## Question 8 — Customer Support Agent

Mid-investigation into a billing dispute, the agent discovers the customer also has an open dispute case with a different order that shares the same charge date — something the standard four-step billing pipeline didn't anticipate.

**What should happen next?**

- **A.** Continue the fixed pipeline exactly as designed, since deviating risks inconsistency
- **B.** Restart the fixed pipeline from step one now that new information has emerged
- **C.** Adapt the investigation dynamically — e.g., pull the second order's details and check whether the two are related before continuing the original billing steps
- **D.** Escalate immediately, since the fixed pipeline no longer covers this case

---

## Question 9 — Customer Support Agent

A teammate suggests: "Since policy occasionally changes, we should replace the fixed `get_customer` → `lookup_order` → `process_refund` pipeline with fully dynamic decomposition everywhere, just to be safe."

**Is this a good idea?**

- **A.** Yes — dynamic decomposition is always safer than fixed pipelines
- **B.** No — the sequence is stable and predictable for routine cases; fixed pipelines should be kept for those, with dynamic decomposition reserved for cases where a mid-investigation finding changes what needs to be checked
- **C.** Yes, but only if the refund amount exceeds $500
- **D.** No — dynamic decomposition should never be used in a customer support context

---

## Question 10 — Customer Support Agent

The agent sometimes calls `escalate_to_human` for a routine $200 refund that should have been auto-processed via `process_refund`, and other times calls `process_refund` for a $600 refund that should have been escalated. Both tools have accurate but generic descriptions ("processes a refund" / "hands off to a human agent").

**What's the most effective fix?**

- **A.** Add a routing classifier in front of the agent to pre-select the correct tool
- **B.** Rewrite both descriptions to explicitly state the boundary condition — e.g., "use `process_refund` only for verified refunds ≤ $500; use `escalate_to_human` for anything above that or when policy doesn't cover the case"
- **C.** Merge both tools into a single `handle_refund` tool that internally decides
- **D.** Add a system prompt instruction listing dollar thresholds in prose

---

## Question 11 — Customer Support Agent

Your system prompt includes: "If in doubt, escalate to a human." Logs show the agent now escalates routine order-status lookups it could easily answer using `lookup_order`, because "if in doubt" is triggering broadly.

**What's the root cause and fix?**

- **A.** `lookup_order`'s description is missing — add one
- **B.** The phrase "if in doubt" is a keyword-sensitive instruction creating an unintended bias toward escalation; rephrase it to apply only when policy is genuinely silent or ambiguous, not general uncertainty
- **C.** The agent's context window is full, causing it to default to escalation
- **D.** `tool_choice` is misconfigured and should be set to `"any"`

---

## Question 12 — Customer Support Agent

A refund request is well-formed, the amount is under $500, and identity is verified — but `process_refund` fails because the customer's account has an active chargeback flag, a business rule unrelated to identity or amount.

**How should the tool communicate this failure?**

- **A.** `{ "isError": true }` — a minimal flag is sufficient since the agent already has the context
- **B.** `{ "isError": true, "errorCategory": "permission", "isRetryable": false }` — since the customer isn't authorized for this action
- **C.** `{ "isError": true, "errorCategory": "business", "isRetryable": false, "description": "Refund blocked: account has an active chargeback flag" }`
- **D.** `{}` — an empty result signaling the refund didn't go through

---

## Question 13 — Customer Support Agent

`lookup_order` intermittently fails with malformed JSON due to a known downstream service issue that historically resolves itself on retry within seconds.

**How should this be classified?**

- **A.** `errorCategory: "validation", isRetryable: false` — the data itself is malformed, so retrying won't help
- **B.** `errorCategory: "transient", isRetryable: true` — the underlying cause is an intermittent service condition, not a genuine data or input problem
- **C.** `errorCategory: "permission", isRetryable: false`
- **D.** No error category needed — the agent should simply retry blindly without structured metadata

---

## Question 14 — Customer Support Agent

You want to guarantee the agent always calls `get_customer` as the very first action of a new conversation, but want it to reason freely about which tool to call after that.

**What's the correct configuration?**

- **A.** Set `tool_choice: {"type": "tool", "name": "get_customer"}` for every turn of the entire conversation
- **B.** Force `tool_choice: {"type": "tool", "name": "get_customer"}` on the first turn only, then switch to `"auto"` for subsequent turns
- **C.** Set `tool_choice: "any"` for the whole conversation
- **D.** Add "always call `get_customer` first" to the system prompt and leave `tool_choice` as `"auto"` throughout

---

## Question 15 — Customer Support Agent

A stakeholder proposes adding the company's internal fraud-detection tool to the support agent's toolset "in case it's ever useful for edge cases," bringing the total from 4 to 5 tools.

**What's the most likely consequence, even with just one extra tool?**

- **A.** No meaningful effect, since one additional tool is a small increase
- **B.** Tool selection reliability begins to degrade as decision complexity increases, and the agent may start misusing or misrouting to the fraud tool outside its intended scope
- **C.** The agent will automatically ignore the tool unless explicitly instructed to use it
- **D.** Response latency increases, but tool selection accuracy is unaffected

---

## Question 16 — Customer Support Agent

A conversation involves two separate issues (a billing dispute and a missing order), each with its own amount, date, and status.

**How should this be represented in context to survive long conversations without being lost?**

- **A.** One combined `CASE_FACTS` block with a single vague summary like "customer has a couple of issues, one about billing"
- **B.** A persistent `CASE_FACTS` block at the beginning of each prompt containing structured, per-issue facts (separate amounts, dates, order IDs for each issue), rather than merging them into one summary
- **C.** Two entirely separate conversations, one per issue
- **D.** No structured block — rely on the model to remember both issues from the raw conversation history

---

## Question 17 — Customer Support Agent

`lookup_order` returns 40+ fields, but only 5 are relevant to resolving the customer's issue.

**What's the correct mechanism to prevent the irrelevant fields from consuming context across a long conversation?**

- **A.** Instruct the model in the system prompt to "only remember the important fields"
- **B.** A `PostToolUse` hook that trims the result to the 5 relevant fields before it's added to conversation history
- **C.** Ask the customer to specify which fields they care about
- **D.** Summarize the full 40-field result into one sentence at the end of each turn

---

## Question 18 — Customer Support Agent

The agent has retried a transient `lookup_order` failure 5 times over several turns without success. The customer hasn't asked for a human, and policy clearly covers the situation once the lookup succeeds.

**Should the agent keep retrying indefinitely?**

- **A.** Yes — since policy is clear and the customer hasn't asked for escalation, retrying is always correct
- **B.** No — inability to make meaningful progress after reasonable retries is itself a valid escalation trigger, separate from customer request or policy gaps
- **C.** No — but only because 5 retries specifically exceeds a hard-coded limit; any other number would be fine to continue
- **D.** Yes, but only if the agent apologizes to the customer first

---

## Question 19 — Customer Support Agent

A search for a customer by phone number returns two matching records.

**What's the correct next step?**

- **A.** Escalate to a human immediately, since any ambiguity should be handled by a person
- **B.** Select the record with the most recent account activity, since it's statistically most likely to be correct
- **C.** Ask the customer for an additional identifier (e.g., order ID or last 4 digits of the card) before proceeding — never guess between matches
- **D.** Process the request using the first record returned by the search

---

## Answer Key

**Q1: A**
The extra field-based check is exactly the anti-pattern the exam tests: parsing tool result content instead of relying on `stop_reason`. Remove it. (B) doesn't address the loop logic bug. (C) and (D) are unrelated to the actual cause.

**Q2: A**
`stop_reason === "end_turn"` is the only reliable termination signal. (B) is text-content parsing — the classic trap. (C) is an arbitrary iteration cap. (D) is a timing heuristic, not a loop-lifecycle signal.

**Q3: B**
Structured fields that are required and code-compiled guarantee completeness. (A) and (C) are probabilistic — they improve but don't guarantee compliance. (D) adds a review step but doesn't fix the root cause (the tool schema not enforcing the field).

**Q4: B**
The gate must check the actual value returned (`verified_id`), not merely that the tool was called — calling a tool that returns nothing useful still leaves you unverified. (A) checks the wrong signal (HTTP status, not business state). (C) doesn't fix the gate logic. (D) is probabilistic.

**Q5: B**
Multi-concern decomposition: resolve what's resolvable, escalate only what requires it, and reuse the same verified identity across both — no re-verification, no blanket escalation, no telling the customer to split their request.

**Q6: B**
This is a data-normalization problem that must be handled deterministically at the interception point — a `PreToolUse` hook should coerce and validate the type before the threshold check runs. (A) is probabilistic and doesn't fix a type bug. (C) doesn't address root cause. (D) is a workaround that doesn't fix the underlying defect and weakens the policy.

**Q7: C**
Normalizing inbound tool results is a `PostToolUse` job. (B) is wrong because `PreToolUse` fires before the call, not after the result returns. (A) and (D) don't guarantee consistent handling.

**Q8: C**
A mid-investigation discovery that reshapes the case calls for dynamic adaptation — check the related finding, then decide how it affects the remaining steps. (A) ignores new information. (B) restarts unnecessarily; identity and other early findings are still valid. (D) is premature — the agent can still investigate with available tools.

**Q9: B**
Fixed pipelines remain the right choice for predictable, repeatable sequences. Dynamic decomposition should be reserved for cases where something discovered mid-investigation changes what needs to happen next — it isn't a blanket "safer" default.

**Q10: B**
Clear, boundary-explicit tool descriptions are the primary mechanism for correct tool selection. (A) and (C) are more invasive architectural changes not warranted as a first step. (D) is probabilistic and less reliable than fixing the descriptions themselves.

**Q11: B**
"If in doubt" is a keyword-sensitive phrase that's over-triggering escalation broadly, rather than only for genuine policy gaps. Rewording it to be conditional (e.g., "escalate when policy doesn't address the request") fixes the root cause. (A), (C), (D) misdiagnose the issue.

**Q12: C**
This is a business-rule violation — not a permission error (the customer is authorized to interact; the block is a policy condition) and not a generic minimal flag. `errorCategory: "business"` with `isRetryable: false` and a clear description lets the agent explain the reason to the customer rather than guessing or retrying uselessly. (B) mislabels the category — permission errors are about authorization to access the resource, not business-policy conditions on an authorized action. (A) and (D) hide the reason entirely.

**Q13: B**
The failure originates from an intermittent, self-resolving service condition — that's the definition of transient, and it's retryable. (A) is a tempting trap: "malformed JSON" sounds like a validation problem, but the root cause described is a transient service issue, not bad input data. (C) mislabels it. (D) throws away useful structured signal.

**Q14: B**
Forced tool selection can be scoped to a single turn and then relaxed — this guarantees the first action while preserving flexibility afterward. (A) would force `get_customer` on every turn, causing unnecessary re-verification. (C) doesn't guarantee which tool is called first. (D) is probabilistic, not guaranteed.

**Q15: B**
Even one additional tool increases decision complexity and can degrade selection reliability, especially for a tool outside the agent's core scope — this is a matter of degree, not a hard cutoff at some large number. (A) underestimates the effect. (C) and (D) are incorrect — added tools remain available for selection and do affect accuracy.

**Q16: B**
Structured, per-issue facts in a persistent block avoid the "vague summary" failure mode where numbers and details get compressed and lost. (A) is the exact anti-pattern the exam targets. (C) is impractical for a live multi-issue conversation. (D) risks the lost-in-the-middle effect.

**Q17: B**
A `PostToolUse` hook is the deterministic mechanism for trimming verbose results before they accumulate in context. (A) and (D) are probabilistic and still let the full result occupy context during the turn it's returned. (C) puts unnecessary burden on the customer.

**Q18: B**
"Inability to make meaningful progress" is a distinct, valid escalation trigger in its own right — separate from explicit customer requests or policy gaps. Retrying forever isn't correct just because policy is clear and the customer hasn't asked for a human. (A) and (D) miss this trigger. (C) is a partially-right distractor — the number isn't the point; the principle is that repeated failed attempts justify escalation regardless of the exact retry count chosen.

**Q19: C**
Multiple matches should always be resolved by requesting an additional identifier — never by heuristic selection (B, D) and not by escalating before even trying the straightforward clarification step (A), which would be premature and hurt first-contact resolution rate.

---

**Scoring guide:** 17–19 correct = strong readiness. 13–16 = review the missed task statements before the real exam. Below 13 = revisit the Domain Task-Statement Walkthrough section of the scenario guide.
