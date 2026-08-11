# Mock Test: Customer Support Agent (CSA) — Set 1

## Question 46 — Customer Support Agent

While testing, you notice the agent often calls `get_customer` when users ask about order status, even though `lookup_order` would be more appropriate. What should you check first to address this problem?

**What should you check first?**

- **A.** Implement a preprocessing classifier to detect order-related requests and route them directly to `lookup_order`.
- **B.** Reduce the number of tools available to the agent to simplify choice.
- **C.** Add few-shot examples to the system prompt covering all possible order request patterns to improve tool selection.
- **D.** Check the tool descriptions to ensure they clearly differentiate each tool's purpose.

---

## Question 47 — Customer Support Agent

Your agent handles single-issue requests with 94% accuracy (e.g., "I need a refund for order #1234"). But when customers include multiple issues in one message (e.g., "I need a refund for order #1234 and also want to update the shipping address for order #5678"), tool selection accuracy drops to 58%. The agent usually solves only one issue or mixes parameters across requests.

**What approach is most effective?**

- **A.** Implement a preprocessing layer that uses a separate model call to decompose multi-issue messages into separate requests, handle each independently, and merge results.
- **B.** Combine related tools into fewer universal tools.
- **C.** Add few-shot examples to the prompt demonstrating correct reasoning and tool sequencing for multi-issue requests.
- **D.** Implement response validation that detects incomplete answers and automatically reprompts the agent to resolve missed issues.

---

## Question 48 — Customer Support Agent

Production logs show that for simple requests like "refund for order #1234," your agent resolves the issue in 3–4 tool calls with 91% success. But for complex requests like "I was billed twice, my discount didn't apply, and I want to cancel," the agent averages 12+ tool calls with only 54% success—often investigating issues sequentially and fetching redundant customer data for each.

**What change is most effective?**

- **A.** Add explicit verification checkpoints between stages, requiring the agent to record progress after resolving each issue before moving to the next.
- **B.** Reduce the number of tools by combining `get_customer`, `lookup_order`, and billing-related tools into a single `investigate_issue` tool.
- **C.** Decompose the request into separate issues, then investigate each in parallel using shared customer context before synthesizing a final resolution.
- **D.** Add few-shot examples to the system prompt demonstrating ideal tool-call sequences for various multi-faceted billing scenarios.

---

## Question 49 — Customer Support Agent

Your agent achieves 55% first-contact resolution, well below the 80% target. Logs show it escalates simple cases (standard replacements for damaged goods with photo proof) while trying to handle complex situations requiring policy exceptions autonomously.

**What is the most effective way to improve escalation calibration?**

- **A.** Require the agent to self-rate confidence on a 1–10 scale before each response and automatically route to humans when confidence drops below a threshold.
- **B.** Deploy a separate classifier model trained on historical tickets to predict which requests need escalation before the main agent starts processing.
- **C.** Add explicit escalation criteria to the system prompt with few-shot examples showing when to escalate versus resolve autonomously.
- **D.** Implement sentiment analysis to determine customer frustration level and automatically escalate past a negative sentiment threshold.

---

## Question 50 — Customer Support Agent

After calling `get_customer` and `lookup_order`, the agent has all available system data but still faces uncertainty.

**Which situation is most justified for escalation?**

- **A.** A customer wants to cancel an order shipped yesterday and arriving tomorrow. The agent should escalate because the customer might change their mind after receiving the package.
- **B.** A customer claims they didn't receive an order, but tracking shows it was delivered and signed for at their address three days ago. The agent should escalate because presenting contradictory evidence could harm the customer relationship.
- **C.** A customer requests competitor price matching. Your policies allow price adjustments for price drops on your own site within 14 days, but say nothing about competitor prices. The agent should escalate for policy interpretation.
- **D.** A customer message contains both a billing question and a product return. The agent should escalate so a human can coordinate both issues in one interaction.

---

## Question 51 — Customer Support Agent

Production logs show that in 12% of cases your agent skips `get_customer` and calls `lookup_order` directly using only the customer-provided name, sometimes leading to misidentified accounts and incorrect refunds.

**What change is most effective?**

- **A.** Add few-shot examples showing that the agent always calls `get_customer` first, even when customers voluntarily provide order details.
- **B.** Implement a routing classifier that analyzes each request and enables only a subset of tools appropriate for that request type.
- **C.** Add a programmatic precondition that blocks `lookup_order` and `process_refund` until `get_customer` returns a verified customer identifier.
- **D.** Strengthen the system prompt stating that customer verification via `get_customer` is mandatory before any order operations.

---

## Question 52 — Customer Support Agent

Production metrics show that when resolving complex billing disputes or multi-order returns, customer satisfaction scores are 15% lower than for simple cases—even when the resolution is technically correct. Root-cause analysis shows the agent provides accurate solutions but inconsistently explains rationale: sometimes omitting relevant policy details, sometimes missing timeline info or next steps. The specific context gaps vary case by case. You want to improve solution quality without adding human oversight.

**What approach is most effective?**

- **A.** Add a self-critique stage where the agent evaluates a draft response for completeness—ensuring it resolves the customer's issue, includes relevant context, and anticipates follow-up questions.
- **B.** Add a confirmation stage where the agent asks "Does this fully resolve your issue?" before closing, allowing customers to request additional information if needed.
- **C.** Upgrade the model from Haiku to Sonnet for complex cases, routing based on a defined complexity metric.
- **D.** Implement few-shot examples in the system prompt showing complete explanations for five common complex case types, demonstrating how to include policy context, timelines, and next steps.

---

## Question 53 — Customer Support Agent

Production metrics show your agent averages 4+ API loops per resolution. Analysis reveals Claude often requests `get_customer` and `lookup_order` in separate sequential turns even when both are needed initially.

**What is the most effective way to reduce loops?**

- **A.** Implement speculative execution that automatically calls likely-needed tools in parallel with any requested tool and returns all results regardless of what was requested.
- **B.** Increase `max_tokens` to give Claude more room to plan and naturally combine tool requests.
- **C.** Create composite tools like `get_customer_with_orders` that bundle common lookup combinations into single calls.
- **D.** Instruct Claude in the prompt to bundle tool requests into one turn and return all results together before the next API call.

---

## Question 54 — Customer Support Agent

Production logs show a pattern: customers reference specific amounts (e.g., "the 15% discount I mentioned"), but the agent responds with incorrect values. Investigation shows these details were mentioned 20+ turns ago and condensed into vague summaries like "promotional pricing was discussed."

**What fix is most effective?**

- **A.** Increase the summarization threshold from 70% to 85% so conversations have more room before summarization triggers.
- **B.** Store full conversation history in external storage and implement retrieval when the agent detects references like "as I mentioned."
- **C.** Extract transactional facts (amounts, dates, order numbers) into a persistent "case facts" block included in every prompt outside the summarized history.
- **D.** Revise the summarization prompt to explicitly preserve all numbers, percentages, dates, and customer-stated expectations verbatim.

---

## Question 55 — Customer Support Agent

Your `get_customer` tool returns all matches when searching by name. Currently, when there are multiple results, Claude picks the customer with the most recent order, but production data shows this selects the wrong account 15% of the time for ambiguous matches.

**How should you address this?**

- **A.** Implement a confidence scoring system that acts autonomously above 85% confidence and requests clarification below the threshold.
- **B.** Instruct Claude to request an additional identifier (email, phone, or order number) when `get_customer` returns multiple matches before taking any customer-specific action.
- **C.** Modify `get_customer` to return only a single most-likely match based on a ranking algorithm, eliminating ambiguity.
- **D.** Add few-shot examples to the prompt demonstrating correct reasoning and tool sequencing for ambiguous matches.

---

## Question 56 — Customer Support Agent

Production logs show a consistent pattern: when customers include the word "account" in their message (e.g., "I want to check my account for an order I made yesterday"), the agent calls `get_customer` first 78% of the time. When customers phrase similar requests without "account" (e.g., "I want to check an order I made yesterday"), it calls `lookup_order` first 93% of the time. Tool descriptions are clear and unambiguous.

**What is the most likely root cause?**

- **A.** The system prompt contains keyword-sensitive instructions that steer behavior based on terms like "account," creating unintended tool-selection patterns.
- **B.** The model's base training creates associations between "account" terminology and customer-related operations that override tool descriptions.
- **C.** The model needs more training data on multi-concept messages and should be fine-tuned on examples containing both account and order terminology.
- **D.** Tool descriptions need additional negative examples specifying when NOT to use each tool to prevent this keyword-induced confusion.

---

## Question 57 — Customer Support Agent

Production logs show the agent often calls `get_customer` when users ask about orders (e.g., "check my order #12345") instead of calling `lookup_order`. Both tools have minimal descriptions ("Gets customer information" / "Gets order details") and accept similar-looking identifier formats.

**What is the most effective first step?**

- **A.** Implement a routing layer that analyzes user input before each turn and preselects the correct tool based on detected keywords and ID patterns.
- **B.** Combine both tools into a single `lookup_entity` that accepts any identifier and internally decides which backend to query.
- **C.** Add few-shot examples to the system prompt demonstrating correct tool selection patterns, with 5–8 examples routing order-related queries to `lookup_order`.
- **D.** Expand each tool's description to include input formats, example queries, edge cases, and boundaries explaining when to use it versus similar tools.

---

## Question 58 — Customer Support Agent

You are implementing the agent loop for your support agent. After each Claude API call, you must decide whether to continue the loop (run requested tools and call Claude again) or stop (present the final answer to the customer).

**What determines this decision?**

- **A.** Check the `stop_reason` field in Claude's response—continue if it is `tool_use` and stop if it is `end_turn`.
- **B.** Parse Claude's text for phrases like "I'm done" or "Can I help with anything else?"—natural language signals indicate task completion.
- **C.** Set a maximum iteration count (e.g., 10 calls) and stop when reached, regardless of whether Claude indicates more work is needed.
- **D.** Check whether the response contains assistant text content—if Claude generated explanatory text, the loop should terminate.

---

## Question 59 — Customer Support Agent

Production logs show the agent misinterprets outputs from your MCP tools: Unix timestamps from `get_customer`, ISO 8601 dates from `lookup_order`, and numeric status codes (1=pending, 2=shipped). Some tools are third-party MCP servers you cannot modify.

**Which approach to data format normalization is most maintainable?**

- **A.** Use a `PostToolUse` hook to intercept tool outputs and apply formatting transformations before the agent processes them.
- **B.** Modify tools you control to return human-readable formats and create wrappers for third-party tools.
- **C.** Create a `normalize_data` tool that the agent calls after every data retrieval to transform values.
- **D.** Add detailed format documentation to the system prompt explaining each tool's data conventions.

---

## Question 60 — Customer Support Agent

Production logs show the agent sometimes chooses `get_customer` when `lookup_order` would be more appropriate, especially for ambiguous queries like "I need help with my recent purchase." You decide to add few-shot examples to the system prompt to improve tool selection.

**Which approach is most effective?**

- **A.** Add explicit "use when" and "don't use when" guidance in each tool description covering ambiguous cases.
- **B.** Add examples grouped by tool—all `get_customer` scenarios together, then all `lookup_order` scenarios.
- **C.** Add 4–6 examples targeted at ambiguous scenarios, each with rationale for why one tool was chosen over plausible alternatives.
- **D.** Add 10–15 examples of clear, unambiguous requests demonstrating correct tool choice for typical scenarios for each tool.
