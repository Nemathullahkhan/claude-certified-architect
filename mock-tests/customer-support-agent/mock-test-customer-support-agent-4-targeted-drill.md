# Targeted Drill: Customer Support Agent (CSA) — Architecture-Overreach & Escalation-Scope

> Purpose: this is NOT a full-domain mock. Use this as an example of a **targeted drill** — a short, focused set built once your own wrong-answer tracker (see `mock-tests/wrong-answers/TEMPLATE-wrong-answer-tracker.md`) shows a *recurring* pattern (2+ mocks), rather than a full domain sweep. This particular drill isolates two of the most common recurring patterns for this scenario:
> 1. **Architecture-overreach** — reaching for a new model call, new agent instance, speculative/automatic tool execution, or new infrastructure when a simpler few-shot/prompting fix is correct.
> 2. **Escalation-scope calibration** — letting one sub-issue's escalation need pull the *entire* multi-issue request into escalation, instead of decomposing first and escalating only what needs it.
>
> All 10 questions below test one of these two patterns, in fresh scenarios/wording. Bar to clear: **9/10 or 10/10** — since these are common systemic gaps, a "passing" score below that bar can mean the pattern is still hiding rather than closed. For each answer, write out *why* you rejected the architectural/all-or-nothing option before picking your final answer — if you can't articulate it, you're pattern-matching, not reasoning.

---

## Question 1

Your agent resolves single-topic shipping questions with 97% tool-selection accuracy. But when a customer writes "my package is late AND I think I was overcharged for shipping," the agent frequently only investigates one of the two threads, or investigates both but merges the findings into one confused response.

**What is the most effective fix?**

- **A.** Add a routing layer that runs a lightweight classifier to detect multi-topic messages and forks them into two parallel sub-agent sessions.
- **B.** Add few-shot examples demonstrating how to identify multiple stated concerns in one message, investigate each with the right tools, and present distinct findings for each.
- **C.** Instruct the agent via system prompt to always ask "is that everything, or do you have another question?" before proceeding, to force single-topic messages.
- **D.** Increase `max_tokens` so the agent has enough room to reason through both topics without truncation.

---

## Question 2

A customer's message: "This is the third time I'm dealing with this shipping delay, I want to file a complaint — and by the way, can you also just confirm my current address on file?"

**What is the correct handling?**

- **A.** Escalate the entire message to a human, since a formal complaint request is present and complaints should always be handled holistically alongside anything else in the same message.
- **B.** Confirm the address using existing tools and shared verified identity, and separately escalate the complaint with a structured handoff noting the repeated-contact history — don't let the complaint block the quick, resolvable address confirmation.
- **C.** Ask the customer to submit the complaint and the address request as two separate messages so they can be processed independently.
- **D.** Spin up a secondary agent instance dedicated to complaint-handling so the primary agent can focus solely on the address confirmation.

---

## Question 3

Your agent's resolution time for cases involving both a refund and a subscription change increased significantly. Analysis shows the agent calls `get_customer`, resolves the refund fully (including a follow-up confirmation exchange), and only *then* begins investigating the subscription change — even though both pieces of information needed for both tasks were available from the first `get_customer`/`lookup_order` calls.

**What is the most effective fix?**

- **A.** Deploy a task-planning sub-agent that decomposes every incoming message into an explicit task list before the main agent begins any tool calls.
- **B.** Add few-shot examples showing that when a message contains multiple independent asks, the agent should gather all needed data up front and address each part before finalizing the response, rather than fully closing one part before starting the next.
- **C.** Combine `get_customer` and `lookup_order` into one composite tool call to reduce the number of round trips.
- **D.** Add a hook that automatically holds the final response until every tool relevant to the customer's account has been called at least once.

---

## Question 4

A customer says: "I'd like a human to review my case because I don't trust the automated process, and also, what's your standard return window?"

Your return-window policy is a fixed, unambiguous 30 days, stated clearly in your policy documentation.

**What should the agent do?**

- **A.** Escalate the entire interaction, since the customer explicitly distrusts automation and any response might reinforce that distrust.
- **B.** Answer the return-window question directly since it's a clear policy fact requiring no judgment, and separately honor the explicit escalation request for the case review — these are two independent asks, not one blocking the other.
- **C.** Decline to answer the return-window question until a human has reviewed the case, to avoid contradicting whatever the human might later say.
- **D.** Use a sentiment-detection step to confirm the customer is indeed frustrated before deciding whether escalation is warranted.

---

## Question 5

Your team wants to reduce cases where the agent handles a multi-part billing question inefficiently — investigating each stated concern in a rigid, fixed order even when the customer's message makes clear that two of the three concerns are related and should be looked at together.

**What is the most effective fix?**

- **A.** Introduce a preprocessing model call that outputs a dependency graph of sub-issues before the main agent starts working.
- **B.** Add few-shot examples showing the agent recognizing related sub-issues within one message and investigating them together rather than mechanically addressing each in isolation.
- **C.** Force a fixed sequence tool_choice pattern so every multi-part billing case follows the identical investigation order regardless of content.
- **D.** Build a separate "relatedness classifier" service that flags which sub-issues in a message should be grouped, called before the agent begins.

---

## Question 6

A message reads: "Please cancel order #9981 — and separately, I want to formally escalate the fact that I've never received a response to my email from two weeks ago."

Order #9981 is easily cancellable using existing tools within policy; the "no response to my email" issue has no relevant tool or policy coverage and clearly requires human follow-up.

**What is the correct handling?**

- **A.** Cancel order #9981 using available tools, and escalate only the unaddressed-email complaint with a structured handoff — the two asks are independent and only one needs a human.
- **B.** Escalate everything together since the customer used the word "escalate," which should be treated as an instruction covering the full message.
- **C.** Cancel the order, then tell the customer to re-send their original email so the system can process it before deciding on escalation.
- **D.** Hold the cancellation until a human has addressed the older email complaint, to present one unified resolution.

---

## Question 7

Your agent occasionally produces the correct final answer but takes an inefficient path: for messages where the customer's intent is a simple order-status check, the agent sometimes first calls `get_customer`, then asks the model to plan next steps in a separate follow-up API call, then calls `lookup_order` — three round trips for something that could be two.

**What is the most effective fix?**

- **A.** Insert an intermediate "planning" model call before every customer request, so the agent always reasons about its full plan before executing any tools — even for simple cases.
- **B.** Add a few-shot example showing a simple order-status request resolved in a single turn with both necessary tool calls requested together, without an intermediate planning turn.
- **C.** Build a request-complexity classifier that decides upfront whether a case needs a planning turn or not, adding infrastructure to route accordingly.
- **D.** Automatically skip the planning call whenever fewer than two tools have been used so far in the conversation, regardless of what the model actually requested.

---

## Question 8

A support ticket contains three distinct requests: (1) update the shipping address for a pending order, (2) apply a discount code that failed at checkout, and (3) escalate a complaint about a rude interaction with a previous agent. Requests 1 and 2 are fully resolvable with existing tools and policy; request 3 has no applicable tool and is inherently a human-judgment matter.

**What is the correct handling?**

- **A.** Resolve requests 1 and 2 using available tools and shared verified identity, and escalate only request 3 with a structured handoff summarizing the complaint — don't let the complaint expand into a blanket escalation of the whole ticket.
- **B.** Escalate the full ticket, since a "rude interaction" complaint is a sensitive matter that should be handled by a human in its full original context.
- **C.** Resolve requests 1 and 2, then ask the customer whether they still want to proceed with the complaint before taking further action.
- **D.** Route the entire ticket to a specialized "complaint-handling" agent instance so all three parts are processed under a consistent complaint-aware framing.

---

## Question 9

Your agent handles refund requests efficiently on their own (3 tool calls, 95% success), but when a refund request arrives bundled with a general product question ("Can I get a refund for order #445, and also, does the Model X come in blue?"), the agent's tool-call count nearly doubles because it re-fetches customer data mid-conversation after answering the product question, as if starting a new task.

**What is the most effective fix?**

- **A.** Add few-shot examples demonstrating that once customer/order context has been retrieved for one part of a multi-part message, it should be reused for subsequent parts rather than re-fetched.
- **B.** Cache all tool outputs at the infrastructure level for the duration of the session, transparently intercepting and short-circuiting any duplicate tool call before it reaches the model.
- **C.** Add a hook that blocks any tool call requesting data that's already present in the conversation, regardless of context.
- **D.** Split unrelated topics into separate conversation threads so context never needs to carry over between them.

---

## Question 10

A message reads: "I want a full refund for my last three orders because your service has been consistently disappointing, and I'm also considering canceling my subscription — but first, what would I lose access to if I cancel?"

The "what would I lose access to" question is a simple, factual policy/feature lookup. The refund request for three orders and the cancellation consideration both require more substantive handling, but nothing here is stated as a policy gap or explicit request for a human — the agent has tools to look up each order and process eligible refunds under policy.

**What is the correct handling?**

- **A.** Escalate the entire message because "consistently disappointing" signals a retention risk that should default to human handling.
- **B.** Answer the factual cancellation-impact question directly, and proceed to evaluate each of the three orders for refund eligibility using existing tools and policy — escalating only if a specific order's refund actually falls outside policy or another genuine trigger (explicit request, policy gap, or repeated failure) applies, not merely because the customer expressed dissatisfaction.
- **C.** Immediately escalate due to the mention of canceling a subscription, since retention decisions should never be automated.
- **D.** Refuse to process any refunds until the customer confirms whether they are canceling their subscription or not, to avoid conflicting actions.

---

## Answer Key

**Q1: B** — This is a reasoning/sequencing gap (investigating multiple stated concerns correctly within one message), not a missing-capability or architecture problem. (A) is the overreach trap — parallel sub-agent sessions for what few-shot can teach directly. (C) suppresses legitimate multi-topic messages instead of handling them. (D) misdiagnoses it as a token-budget issue.

**Q2: B** — Decompose first: the address confirmation is quick and resolvable now: do it. The complaint is a genuine escalation (explicit history + no tool/policy path for "complaint resolution" as a category) — escalate only that part. (A) is the all-or-nothing trap. (C) adds friction with no benefit. (D) spins up unnecessary architecture for what's a same-message, same-context split.

**Q3: B** — The fix is behavioral/sequencing (gather what's needed and address both asks without needlessly closing one out first) — a few-shot problem. (A) and (D) are architecture-overreach: a planning sub-agent or a blanket "hold until all tools called" hook are heavier than the problem requires. (C) doesn't address the sequencing behavior at all, only a tool count.

**Q4: B** — A clear, stated, unambiguous policy fact (30-day window) should be answered directly — it's not a judgment call requiring a human, regardless of what else is in the message. The escalation request is honored separately, in parallel, not by blocking the factual answer. (A) and (C) let the escalation request contaminate an unrelated, easily-answerable question. (D) is an unnecessary sentiment-gating layer.

**Q5: B** — Recognizing relatedness between sub-issues within a single message and adjusting investigation order is a reasoning-quality skill, teachable via few-shot with worked examples. (A) and (D) build new infrastructure/services for something prompting can solve. (C) is the opposite problem — force-fitting a rigid order actively prevents the desired adaptive behavior.

**Q6: A** — Two genuinely independent asks: one fully resolvable now, one requiring a human with no applicable tool/policy. Handle each on its own merits. (B) treats the word "escalate" as a magic keyword covering the whole message — a keyword-triggered trap. (C) and (D) add unnecessary friction or make the resolvable action needlessly dependent on the unrelated complaint.

**Q7: B** — The fix is teaching the model, via example, that it can and should request multiple needed tools in one turn without an intervening planning-only call — a prompting/few-shot fix. (A) makes the inefficiency worse by mandating planning calls universally, including for genuinely simple cases. (C) is unnecessary new infrastructure for a prompting gap. (D) is a blunt heuristic disconnected from the model's actual reasoning.

**Q8: A** — Decompose the ticket: resolve the two tool/policy-coverable requests, escalate only the one requiring human judgment (no tool exists for "rude interaction" complaints, and it's a genuine judgment call). (B) is the all-or-nothing trap driven by the emotionally weighty language of one sub-issue. (C) adds unneeded friction on already-resolvable items. (D) is architecture-overreach — a dedicated complaint agent instance for a single sub-request within a mixed ticket.

**Q9: A** — The core issue is the model not recognizing it can reuse already-fetched context for a later part of the same conversation — a reasoning/instruction gap, fixable with few-shot examples showing context reuse across sub-topics. (B) and (C) are infrastructure-level interception fixes that are heavier than necessary and risk incorrectly blocking legitimate repeat calls (e.g., if data actually changed). (D) fragments a single coherent conversation unnecessarily.

**Q10: B** — Separate the factual question (answer directly) from the refund evaluations (proceed under normal tool/policy process, escalating only if a genuine trigger — explicit request, policy gap, or repeated failure — actually applies to a specific order). Emotional language ("consistently disappointing") and the mere mention of considering cancellation are not, by themselves, valid escalation triggers. (A) and (C) escalate based on sentiment/retention-risk framing alone, which is the named anti-pattern from the underlying scenario material. (D) introduces an unnecessary blocking condition unrelated to actual policy requirements.

---

**Scoring guide for this drill:** 9-10/10 = pattern confirmed closed, safe to move to Scenario 2. 7-8/10 = one more isolated drill before moving on. Below 7 = the pattern is still active; revisit your own wrong-answer tracker and "Recommended Next Steps" section in your scenario report before attempting Scenario 2.
