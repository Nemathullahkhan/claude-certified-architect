# CCA-F Exam Prep — Domain 1: Agentic Architecture & Orchestration
> **Exam Weight: 27% — The Heaviest Domain**
> Every 4th question on the exam comes from this domain. Master this first.

### Exam Quick Facts (Memorize These)

| Item | Detail |
|---|---|
| Questions | 60 multiple-choice (1 correct + 3 distractors) |
| Duration | **120 minutes** — no breaks, cannot pause |
| Passing Score | **720 / 1000** (scaled) — no per-domain minimums |
| Fee | $125 USD (12-month validity; supersedes launch-period $99 / first-5,000-free terms — see [README Resources](../README.md#resources)) |
| Proctoring | ProctorFree — webcam required, **no external resources** |
| Scenarios | 4 of 6 randomly selected — all questions anchored to those 4 |
| Results | 2 business days — includes domain breakdown + digital badge |
| Resources allowed | **None** — study to internalize, not to look up |

---

## Credits & Resources

**Repository owner:** Muhammed Nemathullah Khan

**Author of this domain guide:** [Arun Varadharajalu](https://www.linkedin.com/in/arunv11u/)

**Domain Resources:**
- **Core topics & fundamentals overview:** [CCA-F Study Guide — guide_en.md](https://github.com/paullarionov/claude-certified-architect/blob/main/guide_en.md)
- **Deep dive in each domain:** [Domain 1](./domain-1.md) · [Domain 2](./domain-2.md) · [Domain 3](./domain-3.md) · [Domain 4](./domain-4.md) · [Domain 5](./domain-5.md)
- **Scenarios:** [All 6 exam scenarios](./listedScenarios.md)
- **Mock test and sources:** [Mock test bank](./mock-test/) · [Claude Certification Guide — Mock Exam](https://claudecertificationguide.com/mock-exam) · [CyberSkill Practice — CCAF](https://practice.cyberskill.world/practice/ccaf/exam) · [CertSafari — CCAF](https://www.certsafari.com/anthropic/claude-certified-architect-foundations)

---

## Table of Contents
1. [What This Domain Tests](#1-what-this-domain-tests)
1b. [The Master Mental Model — Probabilistic vs. Deterministic](#1b-the-master-mental-model--probabilistic-vs-deterministic) ⭐ NEW
1c. [The 6 Exam Scenarios](#1c-the-6-exam-scenarios--know-all-of-them) ⭐ NEW
2. [Agentic vs Workflow vs Conversational Systems](#2-agentic-vs-workflow-vs-conversational-systems)
3. [The Agentic Loop](#3-the-agentic-loop)
4. [Task Decomposition Strategies](#4-task-decomposition-strategies)
5. [Multi-Agent Patterns](#5-multi-agent-patterns)
6. [Subagent Context Isolation & Context Passing](#6-subagent-context-isolation--context-passing)
7. [allowedTools & Subagent Spawning](#7-allowedtools--subagent-spawning)
8. [Hooks & Programmatic Enforcement](#8-hooks--programmatic-enforcement)
8b. [Dynamic Routing vs Fixed Routing](#8b-dynamic-routing-vs-fixed-routing) ⭐ NEW
9. [Session Management](#9-session-management)
10. [Error Classification & Recovery](#10-error-classification--recovery)
11. [Human-in-the-Loop (HITL) Escalation](#11-human-in-the-loop-hitl-escalation)
12. [Minimal Footprint Design](#12-minimal-footprint-design)
13. [Dynamic Planning & Ambiguity Handling](#13-dynamic-planning--ambiguity-handling)
13b. [ReAct Loop — The Formal Pattern](#13b-react-loop--the-formal-pattern) ⭐ NEW
13c. [Context Budget & Context Rot](#13c-context-budget--context-rot) ⭐ NEW
13d. [Coordinator Scope Decomposition — Root Cause of Coverage Gaps](#13d-coordinator-scope-decomposition--root-cause-of-coverage-gaps) ⭐ NEW
13e. [Subagent Context Budget Pattern](#13e-subagent-context-budget-pattern) ⭐ NEW
13f. [MCP in Agentic Systems (Domain 1 Crossover)](#13f-mcp-in-agentic-systems-domain-1-crossover) ⭐ NEW
13g. [Safety Subsystems — Doom Loops & Iteration Caps](#13g-safety-subsystems--doom-loops--iteration-caps) ⭐ NEW
13h. [Reliable vs Unreliable Escalation Triggers](#13h-reliable-vs-unreliable-escalation-triggers) ⭐ NEW
13i. [Coordinator as Synthesis Bottleneck Anti-Pattern](#13i-coordinator-as-synthesis-bottleneck-anti-pattern) ⭐ NEW
13j. [Handoff Chains & Parallel Fan-Out](#13j-handoff-chains--parallel-fan-out) ⭐ NEW
13k. [In-Context vs. External Memory](#13k-in-context-vs-external-memory) ⭐ NEW
13l. [Anthropic's Production Multi-Agent Lessons](#13l-anthropics-production-multi-agent-lessons-first-party-source) ⭐ NEW
14. [Anti-Patterns Master List](#14-anti-patterns-master-list)
15. [Key Rules to Memorize](#15-key-rules-to-memorize)
16. [Practice Questions (20 MCQs)](#16-practice-questions-20-mcqs)
17. [Answer Key & Explanations](#17-answer-key--explanations)

---

## 1. What This Domain Tests

| Task Statement | Description |
|---|---|
| 1.1 | Design and implement agentic loops for autonomous task execution |
| 1.2 | Orchestrate multi-agent systems with coordinator-subagent patterns, handoff schemas, and error propagation |
| 1.3 | Configure subagent invocation, context passing, and spawning |
| 1.4 | Implement multi-step workflows with enforcement and handoff patterns |
| 1.5 | Apply Agent SDK hooks for tool call interception and data normalization |
| 1.6 | Design task decomposition strategies for complex workflows |
| 1.7 | Manage session state, resumption, forking, and crash recovery |
| 1.8 | Handle in-context vs. external memory, agent communication, and error propagation |

---

## 1b. The Master Mental Model — Probabilistic vs. Deterministic

> ⭐ **THE SINGLE MOST IMPORTANT CONCEPT FOR THE EXAM**
> 
> Every wrong answer on the CCA-F exam is a probabilistic mechanism being used where a deterministic one is required.

### The Meta-Pattern

| Task | ❌ Probabilistic (WRONG answer) | ✅ Deterministic (CORRECT answer) |
|---|---|---|
| Enforce tool ordering | Few-shot examples, system prompt instructions | Programmatic prerequisite hooks |
| Escalation routing | Self-reported confidence scores, sentiment analysis | Rule-based triggers (threshold, explicit request, retry count) |
| Schema compliance | Prompt instruction to "output valid JSON" | `strict: true` + JSON schema validation |
| Loop termination | Text content parsing ("Task complete") | `stop_reason === "end_turn"` |
| Error recovery | Retry everything blindly | Check `is_retryable` flag first |
| Safety enforcement | "Never process refunds over $500" in system prompt | PreToolUse hook that blocks programmatically |

### How to Apply It on the Exam

**Step 1:** Read the requirement. Does it say "always", "never", "must", "every time", or "guarantee"?

**Step 2:** If yes — find the deterministic option. Eliminate all probabilistic answers immediately.

**Step 3:** If two options are both deterministic, pick the one that operates at the earliest interception point (e.g., PreToolUse before the tool fires is earlier than PostToolUse after).

### Why This Matters

Prompt instructions are processed by a neural network. Even with perfect phrasing, compliance is ~97–99%. For financial limits, compliance gates, and ordering rules — a 1–3% failure rate means real money lost or real regulatory violations. The exam tests whether you know when "good enough" isn't good enough.

---

## 1c. The 6 Exam Scenarios — Know All of Them

> ⭐ **The exam randomly selects 4 of these 6. You are the architect. Every question is anchored to a specific production system.**

| Scenario | Primary Domains | Core Challenge |
|---|---|---|
| **1 — Customer Support Agent** | D1, D2, D5 | Prerequisite gates, structured escalation, rule-based triggers |
| **2 — Code Generation with Claude Code** | D3, D5 | CLAUDE.md hierarchy, plan mode, session management |
| **3 — Multi-Agent Research System** | D1, D2, D5 | Parallel subagents, structured context, iterative refinement |
| **4 — Developer Productivity Tools** | D1, D2, D3 | Task-scoped tool profiles, dynamic adaptive decomposition |
| **5 — Claude Code for CI/CD** | D3, D4 | Headless mode (`-p` flag), structured output for pipelines |
| **6 — Structured Data Extraction** | D4, D5 | JSON schemas, nullable fields, retry loops, batch processing |

**Study priority:** Scenarios 1, 3, and 4 are the heaviest for Domain 1. They account for ~45% of exam questions combined.

### Scenario-Specific Exam Traps

**Scenario 1 (Customer Support):**
- Using system prompt ordering instead of prerequisite hooks for `process_refund`
- Escalating based on Claude's expressed uncertainty — use rule-based triggers only
- Returning empty results when a tool times out — return structured error with `is_retryable`

**Scenario 3 (Research System):**
- Always routing through all 4 subagents regardless of query type — use dynamic routing
- Passing findings as plain text blobs — use structured JSON with source provenance
- Having the synthesis subagent query the web directly — must go through coordinator

**Scenario 4 (Developer Tools):**
- Giving all available tools regardless of task phase — use task-scoped tool profiles
- Using fixed sequential decomposition for open-ended exploration — use dynamic adaptive
- Over-provisioning tools (Read + Write + Bash + Deploy) when the task only needs Read

---

## 2. Agentic vs Workflow vs Conversational Systems

This is always tested. You must instantly identify which type fits a given scenario.

### The Three Types

| Type | What It Is | Claude's Role | When To Use |
|---|---|---|---|
| **Conversational** | Back-and-forth chat, no tools, no autonomy | Responds to each message | Simple Q&A, customer chat, drafting |
| **Workflow** | Fixed, predetermined sequence of steps | Executes each step in order | Known process with no branching decisions |
| **Agentic** | Dynamic — Claude decides what to do next based on what it finds | Plans, acts, observes, replans | Complex tasks where steps are unknown upfront |

### How to Tell Them Apart on the Exam

**Key question to ask:** Are the steps fixed upfront, or does Claude decide the next step based on what it finds?

- Steps are **fixed and never change** → **Workflow**
- Steps are **decided dynamically** by Claude based on results → **Agentic**
- No tools, no decisions, just conversation → **Conversational**

### Examples

```
"Build a system that always runs: extract → validate → store → notify"
→ WORKFLOW — steps never change

"Build a system that researches a company, decides which sources to check
based on what it finds, and writes a report"
→ AGENTIC — Claude decides next steps based on findings

"Build a customer chat assistant that answers questions"
→ CONVERSATIONAL — no tools, no autonomy
```

### The Exam Trap
The exam will describe a scenario with multiple steps and try to get you to pick **workflow** when the correct answer is **agentic**. The distinction: if Claude must **make decisions** based on intermediate results, it is agentic — not a workflow.

---

## 3. The Agentic Loop

This is the single most tested concept in the entire exam. You must know this precisely.

### The Complete Loop Structure

```
1. Send request to Claude
   (with tool definitions + full conversation history)
2. Receive Claude's response
3. Check stop_reason:
   → "tool_use"    = Claude wants to call a tool
                     → execute the tool
                     → go to step 4
   → "end_turn"    = Claude is done
                     → EXIT the loop
   → "max_tokens"  = hit token limit
                     → handle gracefully, do NOT crash
   → "stop_sequence" = hit configured stop sequence
                     → exit as configured
4. Append assistant message to conversation history
5. Append tool result to conversation history as a user message
6. Go back to step 1
```

### stop_reason — Every Value You Must Know

| stop_reason | Meaning | What To Do |
|---|---|---|
| `"tool_use"` | Claude wants to call a tool | Execute the tool, return result, continue loop |
| `"end_turn"` | Claude is finished | Exit the loop — task is complete |
| `"max_tokens"` | Hit the token limit | Handle gracefully — do NOT just crash |
| `"stop_sequence"` | Hit a configured stop sequence | Exit as configured |
| `"pause_turn"` | Server-side tool loop hit 10-iteration limit | Re-send user message + assistant response; server resumes automatically — do NOT add "Continue." |

> **`pause_turn` exam note:** This applies specifically to **server-side tools** (code execution, web search). The server runs its own internal sampling loop capped at 10 iterations by default. When it hits the cap, `stop_reason` becomes `"pause_turn"`. To resume: re-send the conversation without adding any new user message — the API detects the trailing `server_tool_use` block and resumes from where it stopped. Always set a `max_continuations` limit to prevent infinite loops.

### The #1 Anti-Pattern — Text-Based Loop Termination

```javascript
// ❌ WRONG — fragile, non-deterministic, will fail on the exam
if (response.content[0].text.includes("Task complete")) break;
if (response.content[0].text.includes("DONE")) break;

// ✅ CORRECT — always use the structured API signal
if (response.stop_reason === "end_turn") break;
```

**Why it's wrong:** Claude's phrasing is non-deterministic. It might say "finished", "done", "complete", or nothing at all. `stop_reason` is a reliable, structured API signal — always use it.

### Tool Results Must Go Back Into Conversation History

This is critical. When a tool executes and returns a result, you do NOT start a new conversation. You **append the tool result to the existing history** and send everything back.

```
Conversation history grows like this:
[user message]
[assistant response with tool_use block]
[tool_result block]          ← YOU add this
[assistant next response]    ← Claude now reasons with the result
[tool_result block]          ← YOU add this again
[assistant final response]   ← Claude concludes
```

If you strip history between iterations, Claude loses all context and makes poor decisions.

### Parallel tool_use — Multiple Tool Calls in One Response

Claude can emit **multiple `tool_use` blocks in a single response**. When this happens:

```
// Claude returns two tool_use blocks in one response:
assistant: [
  {type: "tool_use", id: "toolu_01A", name: "get_customer", input: {...}},
  {type: "tool_use", id: "toolu_01B", name: "get_weather", input: {...}}
]

// ❌ WRONG — send each result as a separate user message
user: {tool_result, tool_use_id: "toolu_01A", content: "..."}
user: {tool_result, tool_use_id: "toolu_01B", content: "..."}  // API error

// ✅ CORRECT — collect ALL results and send in ONE user message
user: [
  {type: "tool_result", tool_use_id: "toolu_01A", content: "Customer found..."},
  {type: "tool_result", tool_use_id: "toolu_01B", content: "Weather is 22°C..."}
]
```

**Exam rule:** When Claude returns multiple `tool_use` blocks, execute all tools, then return all results in a **single user message**. Each `tool_result` must include its matching `tool_use_id`. Sending results as separate messages causes a 400 API error.

### tool_choice — Controlling When Claude Uses Tools

| Value | Behavior |
|---|---|
| `{"type": "auto"}` | Default — Claude decides whether to call a tool |
| `{"type": "any"}` | Claude must call at least one tool (guarantees structured output) |
| `{"type": "tool", "name": "X"}` | Claude must call the specific named tool |
| `{"type": "none"}` | Claude cannot use any tools |
| Add `"disable_parallel_tool_use": true` | Forces Claude to use at most one tool per response |

**Exam trap:** `"any"` forces Claude to use a tool but lets it choose which one. `"tool"` forces a specific named tool. They are not the same. `"none"` lets Claude respond with text only despite tool definitions being present.

### Tool Runner vs Manual Loop

| Approach | When to Use |
|---|---|
| **Tool Runner (SDK)** | Most cases — SDK auto-handles the loop: calls API, detects `tool_use`, executes tools, feeds results back, repeats until `end_turn` |
| **Manual Loop** | When you need fine-grained control: custom logging, conditional execution, HITL approval before each tool call |

**Exam rule:** For tools with side effects (financial transactions, sending emails, deleting data) — **use the manual loop** so you can insert human approval before execution. The Tool Runner executes automatically without any interception point.

### The Order-of-Operations Rule — Always Append Assistant Message First

> ⭐ **Exam-tested — the API requires strict role alternation**

The conversation history must always alternate: `user → assistant → user → assistant...`. When handling a tool call, you must append the **assistant message first**, then the tool results as a user message. Never skip the assistant message.

```python
# ✅ CORRECT order — always
messages.append({"role": "assistant", "content": response.content})  # FIRST
messages.append({"role": "user", "content": tool_results})            # SECOND

# ❌ WRONG — skipping the assistant message breaks role alternation
messages.append({"role": "user", "content": tool_results})  # API error
```

**Why:** The Anthropic API enforces strict alternation. If you append tool results without first appending the assistant's `tool_use` response, the API rejects the request with a validation error.

### `max_turns` and `max_budget_usd` — SDK Loop Control Parameters

The Claude Agent SDK (`ClaudeAgentOptions`) provides two parameters for capping loop execution:

```python
options = ClaudeAgentOptions(
    max_turns=10,            # Stop after 10 tool-use turns (secondary safety cap)
    max_budget_usd=0.50,     # Stop if cost exceeds $0.50 (cost cap)
    system_prompt="..."
)
```

| Parameter | Purpose | When It Fires |
|---|---|---|
| `max_turns` | Iteration safety cap | After N tool-use turns with no `end_turn` |
| `max_budget_usd` | Cost safety cap | When accumulated token cost exceeds the budget |

**Exam rule:** Both are **secondary safety nets**, not primary stop mechanisms. The primary stop is always `stop_reason === "end_turn"`. If `max_turns` fires before `end_turn`, the loop was either under-specified or hit a runaway condition.

---

## 4. Task Decomposition Strategies

### Three Patterns — Know All Three

| Pattern | When To Use | Example |
|---|---|---|
| **Prompt Chaining** (Sequential) | Steps are known upfront, each depends on the previous | Get customer → look up order → process refund |
| **Dynamic Adaptive** | Scope is unknown until you explore | "Add tests to a legacy codebase" — you don't know how many files exist until you look |
| **Parallel Decomposition** | Subtasks are independent of each other | Web search agent + document analysis agent running simultaneously |

### Sequential vs Parallel — The Decision Rule

```
Data dependency between steps → SEQUENTIAL
Steps are independent → PARALLEL (for speed)
```

### The Exam Trap
**Dynamic adaptive** is commonly confused with just "agentic". If the exam describes a scenario where the scope or number of steps is **unknown until exploration begins**, the answer is specifically **dynamic adaptive decomposition**.

### Example Decision

```
"System must: (1) fetch customer data, (2) look up their order using their ID,
(3) process a refund based on the order"
→ SEQUENTIAL — step 2 needs step 1's output (customer ID)
              → step 3 needs step 2's output (order details)

"System must: research 5 independent topics and compile a report"
→ PARALLEL — all 5 research tasks are independent of each other

"Refactor a large codebase — we don't know how many files need changes"
→ DYNAMIC ADAPTIVE — scope is unknown until exploration
```

---

## 5. Multi-Agent Patterns

### Single Agent vs Multi-Agent — When to Choose

**Use a single agent when:**
- Task fits within one context window
- Steps are sequential with no parallelism needed
- Low complexity, simple tool use

**Use multi-agent when:**
- Task is too complex for one context window
- Independent subtasks can run in parallel
- You need specialization (different agents for different skills)
- You need independent verification (one agent checks another's work)

### The Three Topology Patterns

#### Hub-and-Spoke (Most Common — Default Answer)

```
         [Coordinator / Orchestrator]
              /         |         \
    [Research      [Analysis    [Writing
     Subagent]      Subagent]   Subagent]
```

**Rules:**
- One central coordinator breaks down tasks and delegates
- Subagents are specialized workers, each does one thing well
- ALL inter-subagent communication routes THROUGH the coordinator
- Subagents never talk to each other directly
- Coordinator handles: decomposition, delegation, aggregation, error routing
- Parallel spawning: emit multiple `Task` tool calls in **one single coordinator response**

**When to use:** Complex tasks with parallel work, most production multi-agent scenarios

#### Pipeline

```
[Agent A] → [Agent B] → [Agent C] → [Output]
```

- Each agent transforms the output of the previous agent
- Sequential by nature — Agent B cannot start until Agent A finishes
- **When to use:** Data transformation pipelines, document processing chains

#### Peer-to-Peer

```
[Agent A] ←→ [Agent B]
     ↕              ↕
[Agent C] ←→ [Agent D]
```

- Agents communicate directly with each other
- Rare pattern — used when agents need to negotiate or collaborate as equals
- **When to use:** Debate/verification scenarios, negotiation systems

### Exam Trap: Hub-and-Spoke vs Pipeline
The exam will try to get you to pick pipeline when the answer is hub-and-spoke. Rule: if there is a **central coordinator making decisions**, it is hub-and-spoke. If data just flows through agents in sequence without a coordinator, it is pipeline.

### Independent Verification Pattern
Using two separate agents to verify work is better than asking one agent to review its own work because:
- The second agent has no knowledge of the first agent's reasoning chain
- It approaches the problem fresh — not anchored by the first agent's assumptions
- It catches errors the first agent's blind spots caused

---

## 5b. Advanced Coordinator Patterns

### Iterative Refinement Loops — Coordinator Keeps Looping Until Quality Met

A one-shot delegation is NOT always enough. The coordinator must evaluate synthesis output and re-delegate if quality criteria are not met.

```
Standard (one-shot) — WRONG for complex research:
Coordinator → delegates to subagents → receives results → synthesizes → done

Iterative refinement — CORRECT:
Coordinator → delegates to subagents → receives results
           → evaluates: "Are there gaps in coverage?"
           → YES: re-delegates to subagents with targeted queries
           → receives new results → re-evaluates
           → NO gaps: synthesizes final output → done
```

**The exam test:** A scenario describes a research system that produces incomplete reports. The fix is not more subagents — it is an iterative refinement loop where the coordinator re-delegates until coverage is sufficient.

### Scope Partitioning — Prevent Subagent Duplication

When multiple subagents research the same topic, token usage doubles without adding coverage. The coordinator must explicitly partition scope upfront.

```
// ❌ WRONG — both agents research "AI trends" broadly
WebSearchAgent: "Research AI trends"
DocAnalysisAgent: "Research AI trends"
→ 80% overlap in findings, double the tokens

// ✅ CORRECT — coordinator partitions scope explicitly
WebSearchAgent: "Research AI trends in healthcare ONLY — sources: news sites, blogs"
DocAnalysisAgent: "Research AI trends in finance ONLY — sources: uploaded PDF reports"
→ Zero overlap, full coverage across both domains
```

**The exam trap:** The wrong answer will say "deduplicate results after both agents finish." The correct answer is always to partition scope BEFORE delegation.

### Coordinator Prompt Design — Goals, Not Steps

```
// ❌ WRONG — over-specified procedural instructions
"Step 1: Search for X. Step 2: Analyze Y. Step 3: Synthesize Z."
→ Removes adaptability — if step 2 yields nothing, agent is stuck

// ✅ CORRECT — goal and quality criteria, not steps
"Research [topic] comprehensively. Your output must cover:
 - Recent developments (last 6 months)
 - Key players and their positions
 - Any conflicting viewpoints
 Ensure all claims are attributed to sources."
→ Subagent decides HOW to achieve the goal, adapts as needed
```

### Structured Handoff Summaries — When Escalating to Humans

When escalating to a human agent, a plain escalation message is not enough. The human agent may not have access to the full conversation transcript.

```
// ❌ WRONG — vague escalation
"Please help this customer. They have a refund issue."

// ✅ CORRECT — structured handoff summary
{
  "customer_id": "CUST-48291",
  "account_email": "customer@example.com",
  "issue_type": "refund_request",
  "root_cause": "Order #ORD-9821 delivered damaged — photos provided",
  "refund_amount_requested": 847.00,
  "autonomous_limit": 500.00,
  "reason_for_escalation": "Refund exceeds $500 autonomous limit",
  "recommended_action": "Approve full refund — damage clearly documented",
  "conversation_summary": "Customer contacted 3 times. Patient and cooperative.",
  "escalated_at": "2026-05-14T10:32:00Z"
}
```

**Why it matters:** The human agent can act immediately without re-reading the full transcript. This is a direct exam question — "what should the structured handoff include?" Know all the fields.

---

## 6. Subagent Context Isolation & Context Passing

### The Most Important Rule About Subagents

> **Subagents NEVER inherit the parent agent's context.**

Every subagent starts with a **completely fresh, empty context window**. This is not a bug — it is by design.

```
// ❌ WRONG assumption
"The subagent will know what the coordinator already found"

// ✅ CORRECT
The coordinator must EXPLICITLY pass every piece of information
the subagent needs to do its job
```

### What Happens If You Don't Pass Context Explicitly

The subagent will:
- Not know what the coordinator found
- Not know the task constraints
- Not know the user's original request
- Make decisions based on only what was passed in its prompt

### Structured Context Passing — Always Use Structured Data

```json
// ❌ WRONG — plain text loses attribution and structure
"Here is what the research agent found: The company was founded in 2010
and has 500 employees and operates in 12 countries..."

// ✅ CORRECT — structured data preserves attribution
{
  "findings": [
    {
      "content": "Company founded in 2010",
      "source_url": "https://example.com/about",
      "source_title": "Company About Page",
      "retrieved_at": "2026-05-14"
    },
    {
      "content": "500 employees, 12 countries",
      "source_url": "https://example.com/careers",
      "source_title": "Careers Page",
      "retrieved_at": "2026-05-14"
    }
  ]
}
```

**Why structured matters:** Plain text blobs lose attribution. When a synthesis agent produces a report, it cannot cite sources it doesn't know about. Structured data preserves the chain of attribution.

### Claim-Source Mapping — For Research & Citation Chains

> ⭐ **NEW — Explicitly tested for multi-agent research pipelines**

When subagents return research findings that will be cited in a final report, each finding must include a **claim-source mapping** — a direct link between every factual claim and the evidence that supports it.

```json
// ✅ CORRECT — claim-source mapped structured output
{
  "findings": [
    {
      "claim": "The company achieved $240M revenue in FY2025",
      "evidence_excerpt": "Total revenue for fiscal year 2025 was $240 million...",
      "source_url": "https://company.com/investor-relations/annual-report-2025",
      "source_title": "Annual Report 2025",
      "publication_date": "2026-02-14",
      "retrieved_at": "2026-05-25",
      "confidence": "high"
    },
    {
      "claim": "CEO transition occurred in March 2026",
      "evidence_excerpt": "Board appoints new CEO effective March 1, 2026...",
      "source_url": "https://company.com/press/ceo-announcement",
      "source_title": "Press Release - Leadership Update",
      "publication_date": "2026-02-28",
      "retrieved_at": "2026-05-25",
      "confidence": "high"
    }
  ]
}
```

**Why it matters:** The synthesis agent cannot produce a cited report if it only receives plain claims without evidence. Every fact needs: the claim, the supporting excerpt, the source URL, the source title, and the retrieval date. Without these, citations are fabricated.

---

## 6b. AgentDefinition Configuration — How Subagents Are Formally Defined

### What AgentDefinition Is

`AgentDefinition` is the actual Claude Agent SDK object used to formally define each subagent — its identity, role, system prompt, and tool restrictions. The exam tests whether you know this is WHERE subagent scope and behavior is configured.

```python
# AgentDefinition — formally defines a subagent in the SDK
agent_def = AgentDefinition(
    name="web-research-agent",
    description="Specializes in searching the web for recent information",
    system_prompt="You are a web research specialist. Search only for factual, \
                   recent information. Use only the WebSearch tool.",
    allowed_tools=["WebSearch"],
    max_tokens=4096
)
```

### Key Fields in AgentDefinition

| Field | Purpose |
|---|---|
| `name` | Unique identifier for this subagent type |
| `description` | What this subagent does — used by coordinator to select it |
| `system_prompt` | Defines the subagent's role, behavior, and constraints |
| `allowed_tools` | Restricts which tools this subagent can access (minimal footprint) |
| `max_tokens` | Limits output size per subagent response |

### Why This Matters on the Exam

The exam will present scenarios where subagents are misbehaving — using wrong tools, going out of scope, or producing inconsistent output. The correct fix is almost always in the `AgentDefinition` — tighten the system_prompt, restrict `allowed_tools`, or improve the description so the coordinator selects the right subagent.

```python
# ❌ WRONG — vague AgentDefinition lets subagent do anything
agent_def = AgentDefinition(
    name="research-agent",
    description="Research agent",
    system_prompt="Research things.",
    allowed_tools=["WebSearch", "Read", "Write", "Bash", "Execute"]
)

# ✅ CORRECT — precise definition with minimal footprint
agent_def = AgentDefinition(
    name="web-research-agent",
    description="Searches the web for recent news and factual information only",
    system_prompt="You are a web research specialist. Only search for factual, \
                   recent information published in the last 6 months. \
                   Never write files or execute code.",
    allowed_tools=["WebSearch"],
)
```

---

## 7. allowedTools & Subagent Spawning

### The Task Tool — Required for Subagent Spawning

To spawn subagents, a coordinator MUST have `"Task"` in its `allowedTools`. Without it, the coordinator cannot create subagents — period.

```python
# ❌ WRONG — coordinator CANNOT spawn subagents
options = ClaudeAgentOptions(
    allowed_tools=["Read", "Grep", "Glob"],
    system_prompt="You are a research coordinator..."
)

# ✅ CORRECT — "Task" enables subagent spawning
options = ClaudeAgentOptions(
    allowed_tools=["Read", "Grep", "Glob", "Task"],
    system_prompt="You are a research coordinator..."
)
```

### Spawning Subagents in Parallel

To run multiple subagents in parallel, the coordinator emits **multiple Task tool calls in a single response** — not one at a time.

```python
# Coordinator response that spawns 3 subagents in parallel (one response, multiple Task calls):
[
  Task(name="research-agent", prompt="Research topic A..."),
  Task(name="analysis-agent", prompt="Analyze dataset B..."),
  Task(name="writing-agent", prompt="Draft section C...")
]
# All three start simultaneously
```

### Common allowedTools by Role

| Agent Role | Typical allowedTools |
|---|---|
| Coordinator | `["Task", "Read"]` |
| Research subagent | `["WebSearch", "Read"]` |
| File analysis subagent | `["Read", "Grep", "Glob"]` |
| Write-only subagent | `["Write"]` |
| Read-only subagent | `["Read", "Grep"]` |

**Minimal footprint applies here too** — only grant the tools each agent actually needs.

---

## 8. Hooks & Programmatic Enforcement

### What Hooks Are

Hooks are Python/TypeScript functions invoked by the **Agent SDK** at specific points in the loop. They are NOT called by Claude — they are called by your code.

### Hook Types

| Hook | When It Fires | Primary Use |
|---|---|---|
| `PreToolUse` | BEFORE the tool executes | Block policy-violating calls, validate inputs |
| `PostToolUse` | AFTER tool executes, BEFORE Claude sees result | Normalize data formats, enrich results |
| Exit hook | When the agent loop ends | Cleanup, logging, notifications |

### PreToolUse — Blocking Policy Violations

```python
# Block refunds over $500 — runs BEFORE the tool executes
def pre_tool_use_hook(tool_name, tool_input):
    if tool_name == "process_refund":
        if tool_input["amount"] > 500:
            return {
                "error": "Refund exceeds $500 limit — escalate to human agent",
                "escalate": True
            }
    return None  # Allow the tool to proceed
```

### PostToolUse — Data Normalization

```python
# Normalize timestamps before Claude processes them
def post_tool_use_hook(tool_name, tool_result):
    if "timestamp" in tool_result:
        tool_result["timestamp"] = to_iso8601(tool_result["timestamp"])
    return tool_result
```

### Hooks vs Prompt Instructions — Know This Cold

| | Hooks (Programmatic) | Prompt Instructions |
|---|---|---|
| **Compliance rate** | 100% — code always runs | ~99% — probabilistic, Claude may not always follow |
| **Use for** | Financial limits, compliance gates, identity checks, safety rules | Style guidance, tone, behavioral preferences |
| **Reliability** | Deterministic | Non-deterministic |
| **Example** | Block refunds > $500 | "Always respond in a professional tone" |

### The Exam Rule
> **Safety-critical or compliance decisions → ALWAYS hooks (programmatic)**
> **Style/behavior guidance → prompts are fine**

### The Hook Keyword Trigger Rule

> ⭐ **Exam decision shortcut — apply this to every question**

If the requirement description contains any of these words, the answer is **always a hook**, never a prompt instruction:

| Trigger word | Example requirement |
|---|---|
| **"always"** | "Always verify the customer before processing refunds" |
| **"never"** | "Never process refunds over $500" |
| **"must"** | "The agent must log every tool call" |
| **"every time"** | "Every time a file is written, check the path" |
| **"guarantee"** | "Guarantee that status codes are normalized" |

**The rule:** Prompt instructions cannot *guarantee* anything — they are probabilistic. Only hooks guarantee behavior.

### Hook Auditability — Why Hooks Beat Prompts for Compliance

| Property | Hooks | Prompt Instructions |
|---|---|---|
| **Compliance rate** | 100% — code runs or errors | ~97–99% — probabilistic |
| **Auditability** | Full — logged as code execution events | None — invisible reasoning inside Claude |
| **Override-ability** | Cannot be circumvented by Claude | Can be "forgotten" in long contexts |
| **Appropriate for** | Business rules, financial gates, policies, data normalization | Style, tone, preferences |

### Complete Hook Use Cases

| Use Case | Hook Type | Example |
|---|---|---|
| Data normalization | PostToolUse | Timestamps, status codes, currency formats |
| Policy enforcement | PreToolUse | Refund limits, write permissions |
| **Audit logging** | PostToolUse | Log all tool calls to observability system |
| **Rate limiting** | PreToolUse | Prevent too many API calls in one session |
| **Path restrictions** | PreToolUse | Block file writes outside `/src` and `/tests` |
| Prerequisite gates | PreToolUse | Require `get_customer` before `process_refund` |
| Translation/enrichment | PostToolUse | Translate foreign-language tool results to English |

---

### Prerequisite Gates — Enforce Step Ordering at Workflow Level

A prerequisite gate is different from a hook. It is a programmatic check that **blocks a downstream tool from running at all** until a required upstream step has successfully completed.

```python
# Prerequisite gate — process_refund CANNOT run until get_customer succeeds
def check_prerequisites(tool_name, workflow_state):
    if tool_name == "process_refund":
        if not workflow_state.get("verified_customer_id"):
            raise PrerequisiteError(
                "process_refund blocked: get_customer must complete \
                 and return a verified customer ID first"
            )
        if not workflow_state.get("order_verified"):
            raise PrerequisiteError(
                "process_refund blocked: lookup_order must complete first"
            )
```

**The difference from hooks:**

| | Prerequisite Gate | Hook (PreToolUse) |
|---|---|---|
| **Purpose** | Enforce ORDERING — step A must finish before step B starts | Intercept a specific tool call to validate or block it |
| **Scope** | Workflow-level ordering enforcement | Individual tool call interception |
| **Example** | Block process_refund until get_customer returns verified ID | Block any refund over $500 |

**The exam pattern:** A scenario describes a financial workflow where `process_refund` sometimes runs before `get_customer` completes. The correct fix is a **prerequisite gate** that blocks `process_refund` until the upstream step returns a verified customer ID — not just a prompt instruction saying "do these in order."

---

## 8b. Dynamic Routing vs Fixed Routing

> ⭐ **Exam anti-pattern: coordinator that always runs all subagents regardless of the query**

### Fixed Routing (Anti-Pattern)

```python
# ❌ WRONG — always invokes all subagents regardless of query type
def research(query):
    web_results = task("web_agent", query)      # always runs
    doc_results = task("doc_agent", query)      # always runs
    synthesis = task("synthesis_agent", web_results, doc_results)
    return task("report_agent", synthesis)
# Simple query still pays for 4 subagent calls
```

### Dynamic Routing (Correct)

```python
# ✅ CORRECT — coordinator analyzes query first, then decides
def research(query):
    plan = coordinator.analyze(query)           # coordinator decides

    results = {}
    if plan.needs_web_search:
        results["web"] = task("web_agent", query)
    if plan.needs_documents:
        results["docs"] = task("doc_agent", query, plan.relevant_docs)

    return task("synthesis_agent", results)
# Simple queries use 1-2 subagents; complex use all
```

**The exam rule:** Fixed routing is a workflow — the coordinator makes no decisions. Dynamic routing is agentic — the coordinator analyzes the query and selects the minimum required subagents. Dynamic routing is almost always the correct answer for coordinator design questions.

---

## 9. Session Management

### Three Session Operations

| Operation | When To Use | What It Does |
|---|---|---|
| `--resume <session-name>` | Prior context is mostly valid, continuing same work | Resumes the exact session state — Claude remembers everything |
| `fork_session` | You want two independent approaches from the same baseline | Creates a branch — changes in each branch don't affect the other |
| **New session + injected summary** | Prior tool results are stale, fundamental assumption changed | Starts fresh with a summary of what matters from the previous session |

### The Critical File Change Trap

> When resuming a session after files were modified, Claude will NOT detect the changes automatically.

```
// ❌ WRONG — developer just resumes without informing Claude
$ claude --resume my-session
# Claude continues reasoning from stale file analysis

// ✅ CORRECT — developer explicitly informs Claude of changes
$ claude --resume my-session
"Before we continue: files auth.ts, user.service.ts, and db.config.ts
have been refactored since our last session. Please re-read these
three files before proceeding."
```

**Why:** Claude's session state contains its previous analysis of those files. Without being told they changed, it will reason from outdated information — silently producing wrong results.

### When to Use Each Operation

```
Scenario: "Continue analyzing the codebase from yesterday — nothing changed"
→ --resume

Scenario: "Try two different architectural approaches starting from the same point"
→ fork_session

Scenario: "We completely changed our database schema since the last session"
→ New session + inject a summary of what's new
```

### Crash Recovery — Agent State Export (Manifest)

> ⭐ **NEW — Tested for long-running agentic tasks that may crash mid-execution**

For multi-hour or multi-day agentic tasks, the agent must be able to **recover from a crash** without starting from scratch. The pattern is to periodically export a **structured state manifest** that captures exactly where the agent was when it stopped.

```json
// Agent State Manifest — written to disk at checkpoints
{
  "task_id": "TASK-002",
  "checkpoint_at": "2026-05-25T14:32:00Z",
  "phase": "analysis",
  "completed_steps": [
    "mapped codebase structure",
    "identified 47 files requiring changes",
    "processed files 1-23"
  ],
  "pending_steps": [
    "process files 24-47",
    "run cross-file integration analysis",
    "generate final report"
  ],
  "key_findings": [...],
  "files_processed": ["src/auth.ts", "src/user.service.ts"],
  "files_remaining": ["src/order.service.ts", "..."],
  "session_facts": {
    "authorized_scope": "read-only",
    "task_owner": "eng-team@company.com"
  }
}
```

**On restart after crash:**
1. Load the manifest from disk
2. Start a new session
3. Inject the manifest as the first message
4. Agent continues from `pending_steps` — not from scratch

**Exam rule:** An agent that crashes and must restart from zero is a design failure. Production agents always write state manifests at checkpoints.

### Structured Summary for Fresh Sessions

When starting a new session after a phase transition (e.g., research → implementation), inject a summary that captures only what's essential going forward:

```
## What we learned in the research phase
- Auth service uses JWT RS256, not HS256 (discovered Day 2)
- Token expiry is 15min access / 7-day refresh
- 3 downstream services depend on the current token format

## Architecture decision made
- Using the Adapter pattern for backwards compatibility
- New JWT service will wrap the existing one during migration

## Starting point for implementation
- Begin with src/auth/jwt-adapter.ts
- Do NOT touch src/auth/legacy-jwt.ts yet — other services depend on it
```

**What makes prior context stale (→ start fresh):**
- Fundamental assumption was wrong
- Major architectural decision changed
- 50%+ of analyzed files were modified
- Task domain changed entirely from prior session

### Multi-Day Session Strategy

```
Day 1: Exploration (named session "explore")
  → understand structure, identify key files

Day 2-3: Design (resume "explore" → fork to "design-a", "design-b")
  → compare two architectural approaches

Day 4+: Implementation (NEW session with injected decision summary)
  → fresh context prevents design-phase dead-ends from polluting implementation
  → summary carries forward only the decisions, not the deliberation
```

### Stateful vs Stateless Agent Design

| Design | What It Is | When to Use |
|---|---|---|
| **Stateful** | Agent retains context across turns within a session | Multi-step tasks where prior tool results inform next steps; most agentic loops |
| **Stateless** | Each invocation starts fresh; state managed externally | High-concurrency tasks, parallel subagents, idempotent operations |

**The exam pattern:** Stateless subagents (each gets a clean context with only what they need) + stateful coordinator (maintains the full picture) is the correct hub-and-spoke design. A fully stateful single agent hits context limits; a fully stateless system loses continuity.

---

## 10. Error Classification & Recovery

### Three Error Types

| Error Type | What It Is | Examples | How to Handle |
|---|---|---|---|
| **Tool errors** | Tool call failed | API down, invalid input, timeout, permission denied | Retry with backoff for transient; report for permanent |
| **Reasoning errors** | Claude made a wrong decision | Wrong tool chosen, misunderstood task | Catch with validation, retry with clearer instructions |
| **Environment errors** | External system unavailable | Database down, network failure, third-party API failure | Graceful degradation, fallback strategy |

### Transient vs Permanent Errors

```
Transient (retry with backoff):
- Network timeout
- Rate limit hit
- Temporary API unavailability

Permanent (report to user / escalate):
- Invalid API key
- Resource not found (404)
- Permission denied (403)
- Malformed input
```

### Error Propagation in Multi-Agent Systems

When a subagent fails, the coordinator must:
1. Catch the error (not crash silently)
2. Decide: retry? fallback? escalate to human?
3. Continue processing other subagents if they are independent
4. Aggregate results including partial failures

```
// ❌ WRONG — terminates everything on one subagent failure
if (subagent_error): terminate_all_subagents()

// ✅ CORRECT — handle gracefully, continue independent work
if (subagent_error):
    log_error(subagent_error)
    continue_other_subagents()
    include_partial_result_in_final_output()
```

### Structured Error Propagation Format

> ⭐ **Exam-tested — subagents must return structured error objects, not plain strings**

When a subagent cannot complete its task, it must return a **structured error object** — not a plain "it failed" message. The coordinator needs enough information to make an intelligent recovery decision.

```json
// ❌ WRONG — vague error gives coordinator nothing to work with
{
  "status": "error",
  "message": "Operation failed"
}

// ✅ CORRECT — structured error enables intelligent coordinator recovery
{
  "status": "error",
  "failure_type": "transient_timeout",
  "error_category": "transient",
  "is_retryable": true,
  "attempted_query": "SELECT * FROM orders WHERE customer_id = 'CUST-48291'",
  "partial_results": [
    {"order_id": "ORD-001", "status": "delivered"},
    {"order_id": "ORD-002", "status": "pending"}
  ],
  "records_retrieved": 2,
  "records_missing": 3,
  "suggested_recovery": "Retry with narrower date range"
}
```

**The four required fields the exam tests:**
1. `failure_type` / `error_category` — what kind of error (transient / validation / permission / business)
2. `is_retryable` — boolean — can the coordinator retry this automatically?
3. `attempted_query` — what the subagent was trying to do when it failed
4. `partial_results` — whatever was successfully retrieved before failure

**Why partial results matter:** A subagent that retrieved 2 of 5 records before timing out still provides useful data. Return it with an annotation — never discard it silently.

### Conflicting Source Data — Annotate, Never Arbitrarily Resolve

When multiple subagents return **conflicting findings** about the same fact, the coordinator must **annotate the conflict** and surface it — not silently pick one version.

```json
// ❌ WRONG — coordinator silently resolves the conflict
{
  "revenue_2025": "$240M"   // Picked from Source A, ignored Source B
}

// ✅ CORRECT — conflict annotated, both sources preserved
{
  "revenue_2025": {
    "value_source_a": "$240M",
    "value_source_b": "$195M",
    "sources": ["Annual Report 2025", "Industry Analyst Report Q4"],
    "conflict": true,
    "note": "Values differ — may reflect different accounting periods"
  }
}
```

**Exam rule:** Silently resolving conflicting data is always wrong. Annotating and surfacing conflicts is always correct.

### Retry Strategies

```
Simple retry: immediately retry (for rare transient errors)
Exponential backoff: wait 1s, 2s, 4s, 8s... (for rate limits / timeouts)
Max retry limit: always set a cap — never retry infinitely
```

---

## 11. Human-in-the-Loop (HITL) Escalation

### When an Agent Must Escalate to a Human

| Trigger | Example |
|---|---|
| Amount/action exceeds threshold | Refund > $500, transaction > $10,000 |
| Customer explicitly requests human | "I want to speak to a person" |
| Agent has failed multiple times | Tried 3 times, still can't resolve |
| Sensitive situation | Legal threat, medical emergency, safety concern |
| Agent genuinely uncertain | Ambiguous edge case with no clear answer |
| Irreversible action about to be taken | Deleting data, sending mass emails |

### Escalation Design Pattern

```python
def check_escalation_needed(action, context):
    # Threshold-based
    if action.type == "refund" and action.amount > 500:
        return escalate("Amount exceeds autonomous limit")

    # Explicit customer request
    if "speak to a human" in context.customer_message.lower():
        return escalate("Customer requested human agent")

    # Repeated failure
    if context.retry_count >= 3:
        return escalate("Max retries exceeded")

    # Irreversible action
    if action.irreversible and not action.pre_approved:
        return escalate("Irreversible action requires human approval")
```

### The Anti-Pattern
```
❌ Agent that tries to handle EVERYTHING autonomously and never escalates
❌ Agent that escalates for EVERY small decision (over-escalation)
✅ Agent that escalates exactly when defined thresholds are crossed
```

---

## 12. Minimal Footprint Design

### The Principle

> An agent should request only the permissions, tools, and data access it actually needs — nothing more.

This is a security and reliability principle. Broader access than necessary:
- Increases the blast radius if something goes wrong
- Creates compliance and audit risks
- Violates the principle of least privilege

### Application to Tools

```python
# ❌ WRONG — "just in case" over-provisioning
options = ClaudeAgentOptions(
    allowed_tools=["Read", "Write", "Delete", "Admin", "Execute"],
    # Agent only needs to read files...
)

# ✅ CORRECT — exactly what's needed, nothing more
options = ClaudeAgentOptions(
    allowed_tools=["Read"],
)
```

### Application to Database Access

```
❌ WRONG: Give the agent admin-level database access "so it never hits permission errors"
✅ CORRECT: Give the agent read-only access to only the tables it needs
```

### Clarify Before Acting

Minimal footprint also applies to **scope of action** — before starting a long autonomous run on a vague task, the agent should clarify ambiguity first.

```
// ❌ WRONG — starts a 2-hour autonomous run on a vague task
User: "Clean up the codebase"
Agent: [immediately starts deleting files, refactoring, reorganizing...]

// ✅ CORRECT — clarifies scope before acting
User: "Clean up the codebase"
Agent: "Before I begin, I have a few questions:
        1. Which directories are in scope?
        2. Should I touch test files?
        3. What does 'clean up' mean — formatting only, or refactoring too?
        4. Are there any files I should not touch?"
```

---

## 13. Dynamic Planning & Ambiguity Handling

### Dynamic Planning (Replanning)

Unlike a workflow with fixed steps, an agentic system can **replan mid-task** based on what it discovers.

```
Initial plan: Research company A
→ Agent searches and finds company A acquired company B
→ Agent REPLANS: now needs to also research company B
→ This was not in the original plan — agent adapted
```

### When Replanning is Needed

- Discovery of unexpected complexity
- A subtask fails and an alternative approach is needed
- New information changes the scope of the original task

### Ambiguity Handling — Clarify Before, Not During

The correct pattern is to identify and resolve ambiguity **before starting** a long autonomous run, not halfway through.

```
Timing of clarification:
✅ CORRECT: Ask all clarifying questions at the START, before any action
❌ WRONG: Start the task, then ask questions mid-run (wastes work done so far)
❌ WRONG: Complete the task with wrong assumptions (must redo everything)
```

### The Minimal Clarification Rule

Ask the **minimum number of clarifying questions** necessary — not every possible question. If something can be reasonably inferred, infer it. Only ask when a wrong assumption would cause significant wasted work.

---

## 13b. ReAct Loop — The Formal Pattern

> ⭐ **NEW — This is explicitly mentioned in the CCA-F exam curriculum as "ReAct loops"**

The CCA-F curriculum names Domain 1 as "Master the design of **ReAct loops** and autonomous agent workflows." ReAct is the formal name for what Section 3 describes as the agentic loop.

### What ReAct Stands For

**ReAct = Reason → Act → Observe** (cyclically)

```
[REASON]   Claude thinks: "I need to look up customer data"
    ↓
[ACT]      Claude emits a tool_use block → tool executes
    ↓
[OBSERVE]  Tool result appended to history as user message
    ↓
[REASON]   Claude reasons: "Customer found. Now I need their order."
    ↓
[ACT]      Claude emits another tool_use block
    ↓
[OBSERVE]  Second result appended
    ↓
[REASON]   Claude concludes → stop_reason: "end_turn"
```

### Why The Exam Uses This Term

Exam questions may use "ReAct loop" or "ReAct pattern" interchangeably with "agentic loop." They are the same thing. If you see either term, apply Section 3 rules.

### Plan-and-Execute Variant

A more advanced form separates planning from execution:
1. **Planner** — generates the full task breakdown upfront
2. **Executor** — works through each subtask in the plan
3. **Re-planner** — adjusts when execution diverges from the original plan

This maps to the **dynamic adaptive decomposition** pattern in Section 4 — the executor replans when discoveries demand it.

| Pattern | Planning Phase | Execution Phase |
|---|---|---|
| Standard ReAct | Interleaved (decide each step as you go) | One loop |
| Plan-and-Execute | Upfront full plan | Separate executor runs steps |

**Exam trap:** Plan-and-execute is NOT a workflow (steps change dynamically). It is agentic — the re-planner kicks in when reality diverges from the plan.

---

## 13c. Context Budget & Context Rot

> ⭐ **NEW — Token accumulation is a tested production concern**

### The Problem: Context Grows Quadratically

In a ReAct loop, every tool result gets appended to conversation history. As the loop runs:

```
Turn 1: [system] + [user] + [assistant + tool_use] + [tool_result]
Turn 2: Turn 1 + [assistant + tool_use] + [tool_result]
Turn 3: Turn 2 + [assistant + tool_use] + [tool_result]
...
```

Token consumption per agent run grows roughly **O(n²)** in the number of steps. A loop that looks cheap in testing (3 steps) silently explodes in production (30+ steps).

### Context Rot

> **Context Rot** = the model becomes *worse* at reasoning as its context window fills up.

Symptoms:
- Earlier tool results are ignored ("lost in the middle" effect)
- The agent starts hallucinating or contradicting prior findings
- Performance degrades even though no error has occurred

### The Fix: Progressive Summarization / Compaction

```
When context approaches 70-80% of the window limit:

1. Summarize completed work phases into a compact summary
2. Store essential structured facts externally (key IDs, amounts, dates)
3. Inject only the summary + structured facts into the fresh context
4. Continue the loop from the new, lean starting point
```

**Critical rule:** Numeric values, IDs, and dates must go into a **structured block** (not prose summary), because summarization compresses precise values into vague language ("approximately", "recently").

```json
// ✅ CORRECT — structured fact block survives summarization
{
  "customer_id": "CUST-48291",
  "order_id": "ORD-9821",
  "refund_amount": 847.00,
  "issue_date": "2026-04-12"
}

// ❌ WRONG — this gets compressed/lost during summarization
"The customer (ID somewhere in the forties thousands) requested a refund
of approximately eight hundred dollars for an order from April."
```

### The Ralph Loop Pattern

A production pattern for long-running agents:
- Agent completes a **phase** of work
- Saves essential state to external storage (filesystem, DB, or structured artifact)
- **Restarts in a fresh context window**
- Loads only the saved state needed to continue

This prevents context rot on multi-hour tasks.

---

## 13d. Coordinator Scope Decomposition — Root Cause of Coverage Gaps

> ⭐ **NEW — Heavily tested: "who is at fault when a report is incomplete?"**

### The Key Insight

> **If the final output is incomplete in SCOPE (not depth), the root cause is almost always the coordinator's task decomposition — not the subagents.**

### The Exam Scenario Pattern

```
Coordinator decomposes "renewable energy technologies" into:
  → SubAgent A: "Research solar panel efficiency"
  → SubAgent B: "Research wind turbine design"

Both subagents produce thorough, well-sourced research.
The final report has no mention of: geothermal, tidal, biomass, nuclear fusion.

Q: What is the root cause?
A: The COORDINATOR — it never assigned those subtopics.
   The subagents did exactly what they were told.
```

### The Depth vs. Scope Distinction

| Problem Type | Root Cause | Fix |
|---|---|---|
| Report is **shallow** on assigned topics | Subagents — didn't research deeply enough | Better subagent instructions / more tool calls |
| Report is **missing whole topic areas** | Coordinator — decomposition too narrow | Fix the coordinator's decomposition logic |

**Wrong answers on the exam:**
- "Give the subagents better search queries" — fixes depth, not scope
- "Add more subagents" — doesn't help if coordinator doesn't assign the new topics
- "Use a better synthesis agent" — synthesis can only work with what it receives

### Coordinator Decomposition Checklist

Before delegating, the coordinator must verify its decomposition:
1. Does the task list cover **all** dimensions of the topic?
2. Are there adjacent topics that are implicitly in scope?
3. Are there any gaps between subtasks?

---

## 13e. Subagent Context Budget Pattern

> ⭐ **NEW — Subagents explore widely but return lean summaries**

### The Core Pattern

Subagents and the coordinator have **asymmetric context needs**:

```
Subagent (during execution):
  → May explore 30,000-80,000+ tokens of content
  → Uses many tool calls, reads many files, searches many pages
  → Has a large internal context during its work

Subagent (when returning to coordinator):
  → Returns a CONDENSED summary — typically 1,000–2,000 tokens
  → Distills only the findings that matter
  → Discards intermediate reasoning and raw tool outputs
```

**Why this matters:** The coordinator's context window accumulates results from ALL subagents. If each subagent returns 20,000 tokens of raw output, the coordinator hits context rot after just 5 subagents.

### The Anti-Pattern: Fat Subagent Returns

```python
# ❌ WRONG — subagent returns everything it found
return {
    "raw_search_results": [...],  # 15,000 tokens of raw API data
    "full_page_text": "...",      # 8,000 tokens
    "all_findings": [...]         # 200 bullet points
}

# ✅ CORRECT — subagent distills before returning
return {
    "summary": "Company founded 2010, 500 employees, 12 countries",
    "key_findings": [
        {"finding": "Revenue: $240M in FY2025", "source": "Annual report"},
        {"finding": "CEO change in March 2026", "source": "Press release"}
    ],
    "sources": ["https://..."]   # For citation chain
}
```

### The Coordinator Bottleneck Trap

A specific anti-pattern where the coordinator unnecessarily spawns a synthesis subagent and passes its entire 80,000-token context to it, when the coordinator already holds all the findings:

```python
# ❌ WRONG — coordinator passes its full context to a synthesis subagent
coordinator_context = 80_000_tokens_of_accumulated_findings
synthesis_subagent_prompt = coordinator_context + "Please synthesize this"
# Result: 40+ second latency, double token cost

# ✅ CORRECT — coordinator synthesizes directly (it already has everything)
# OR passes only the condensed finding objects, not the full context
synthesis_prompt = json.dumps(condensed_finding_objects)  # ~5,000 tokens
```

---

## 13f. MCP in Agentic Systems (Domain 1 Crossover)

> ⭐ **NEW — MCP is tested as part of Domain 1 agentic patterns**

### MCP Basics Within the Agentic Context

The **Model Context Protocol (MCP)** is an open standard that allows agents to discover and connect to external tools and data sources. Within a hub-and-spoke architecture, MCP is particularly useful for **subagents** — it lets them fetch specific resources they need without requiring the coordinator to inject everything upfront.

### Three MCP Primitives (Know These)

| MCP Primitive | What It Is | Example |
|---|---|---|
| **MCP Resources** | Structured data sources agents can read | Documents, database records, API responses |
| **MCP Tools** | Callable actions agents can invoke | Create task, send email, query DB |
| **MCP Prompts** | Reusable prompt templates | Ensures consistent agent behavior across invocations |

### MCP in Hub-and-Spoke

```
Coordinator → delegates task to SubAgent
SubAgent → uses MCP to fetch the specific resource it needs
         → processes just that resource (clean, lean context)
         → returns condensed result to coordinator

vs.

Coordinator → injects ALL possibly-needed context into SubAgent prompt
SubAgent → receives bloated context, much of it irrelevant
         → performance degrades
```

**Exam rule:** MCP reduces coordinator complexity by letting subagents pull only what they need, rather than having the coordinator push everything.

### MCP Configuration Scope (Tested in Domain 2 but appears in Domain 1 scenarios)

| Level | File | Scope |
|---|---|---|
| Project level | `.mcp.json` | Version-controlled, shared with team |
| User/personal level | `~/.claude.json` | Personal only, not committed |

Secrets always use environment variable substitution: `${GITHUB_TOKEN}` — never committed values.

---

## 13g. Safety Subsystems — Doom Loops & Iteration Caps

> ⭐ **NEW — Safety mechanisms within the agentic loop**

### Doom Loop Detection

A **doom loop** occurs when an agent gets stuck retrying the same action repeatedly without making progress. Common causes:
- A tool consistently returns an error, but the agent keeps retrying
- Claude misinterprets a permanent error as transient and never escalates
- A reasoning error causes the agent to cycle through the same steps

**Detection pattern:**
```python
# Track consecutive identical tool calls or consecutive failures
if consecutive_same_tool_calls >= 3:
    raise DoomLoopDetected("Agent is cycling — escalate or terminate")

if consecutive_failures >= max_retry_limit:
    raise MaxRetriesExceeded("Permanent failure — do not retry infinitely")
```

### Iteration Cap

An **iteration cap** is a secondary safety net that terminates the loop after a fixed number of turns, preventing runaway costs.

```python
MAX_ITERATIONS = 50
iteration = 0

while True:
    iteration += 1
    if iteration > MAX_ITERATIONS:
        # Gracefully terminate, report partial results
        return {"status": "max_iterations_reached", "partial_results": ...}

    response = claude.send(messages)
    if response.stop_reason == "end_turn":
        break
    # ... continue loop
```

**Critical exam rule:** The iteration cap is a **secondary safety net** — it must NEVER be the primary loop termination mechanism. `stop_reason === "end_turn"` is always the primary exit. An iteration cap that fires normally (i.e., before `end_turn`) indicates a poorly designed loop.

### Stale Read Detection

When Claude reads a file at turn 1, then 40 turns later acts on that file assuming its content is unchanged — but the file was modified externally — this is a **stale read**. Production agent harnesses detect this by:
- Tracking file read timestamps
- Re-reading files before acting if time since last read exceeds a threshold
- Using checksums to detect external modifications

---

## 13h. Reliable vs Unreliable Escalation Triggers

> ⭐ **NEW — The exam tests which triggers are reliable enough for production**

### Reliable Escalation Triggers (Always escalate on these)

| Trigger | Why It's Reliable |
|---|---|
| Explicit customer request for a human | Unambiguous — no inference needed |
| Amount/action exceeds a defined threshold | Programmatic check — deterministic |
| Repeated failure reaching max retry count | Programmatic count — deterministic |
| Irreversible action about to execute | Categorical — no ambiguity |
| Policy gap the agent cannot interpret | The agent explicitly recognizes its limits |

### Unreliable Escalation Triggers (Do NOT use as primary triggers)

| Trigger | Why It's Unreliable |
|---|---|
| **Sentiment analysis** of customer messages | LLM-based inference — probabilistic, not deterministic |
| **Model self-rated confidence score** ("I'm 60% confident") | Claude's self-assessment is poorly calibrated |
| **Automatic classifiers** for tone or emotion | ML classifier — non-zero error rate |

**The exam trap:** A question will describe an escalation system that uses sentiment analysis or confidence scores. These are wrong answers — they are probabilistic and will fail in edge cases.

```python
# ❌ WRONG — unreliable escalation trigger
if sentiment_score < 0.3:  # "Customer seems frustrated"
    escalate()

# ✅ CORRECT — reliable escalation trigger
if "speak to a human" in customer_message.lower():  # Explicit request
    escalate()
if refund_amount > 500:  # Threshold check
    escalate()
if retry_count >= 3:  # Programmatic count
    escalate()
```

---

## 13i. Coordinator as Synthesis Bottleneck Anti-Pattern

> ⭐ **NEW — Production performance trap tested on the exam**

### The Scenario

A production multi-agent research system experiences 40+ second latency on synthesis queries. Investigation shows:
- The coordinator orchestrates the initial research (accumulates 80,000 tokens)
- For each synthesis query, the coordinator **spawns a new synthesis subagent**
- It passes **its entire 80,000-token context** to the synthesis subagent
- Result: massive latency + double token cost for no gain

### Why This Is Wrong

The coordinator **already holds** all the research findings in its own context. Spawning a synthesis subagent and passing the coordinator's full context is:
1. Redundant — the coordinator can synthesize directly
2. Expensive — 80K tokens × subagent call = enormous cost
3. Slow — subagent initialization + 80K token processing adds 40+ seconds

### The Correct Fix

```python
# ❌ WRONG — coordinator passes its full context to synthesis subagent
synthesis_result = spawn_subagent(
    agent="synthesis-agent",
    prompt=f"Here are all findings: {coordinator_full_context}"  # 80K tokens
)

# ✅ CORRECT — Option A: coordinator synthesizes directly
# (it already has everything — no subagent needed)
synthesis_result = coordinator.synthesize(condensed_findings)

# ✅ CORRECT — Option B: pass only the condensed finding objects
synthesis_result = spawn_subagent(
    agent="synthesis-agent",
    prompt=json.dumps(condensed_structured_findings)  # ~5K tokens
)
```

**The exam rule:** Only spawn a subagent when the coordinator genuinely cannot do the work itself, or when isolation is required. Never spawn a subagent just to hand it your own full context.

---

## 13j. Handoff Chains & Parallel Fan-Out

> ⭐ **NEW — Both are named patterns in the official CCA-F Domain 1 blueprint**

### Handoff Chain

A **handoff chain** is a sequential multi-agent pattern where one agent completes its work and explicitly **hands off** to the next agent with a structured context package. It differs from a pipeline in one key way: the handoff includes deliberate, structured context — not just raw output.

```
[Agent A: Data Extraction]
    ↓ structured handoff
    {
      "extracted_records": [...],
      "extraction_errors": [...],
      "confidence": 0.94,
      "handoff_notes": "3 records were ambiguous — review flagged rows"
    }
[Agent B: Validation]
    ↓ structured handoff
    {
      "validated_records": [...],
      "rejected": [...],
      "handoff_notes": "12 records failed validation — see reasons"
    }
[Agent C: Storage]
```

**Difference from pipeline:**
| | Pipeline | Handoff Chain |
|---|---|---|
| Data passed | Raw output only | Structured context package with metadata |
| Error info | Not explicit | Included in handoff (errors, confidence, notes) |
| Agent awareness | None | Each agent receives deliberate context about prior work |

**Exam trap:** The exam may describe a multi-agent sequence and ask whether it is a pipeline or a handoff chain. If agents pass **structured metadata about their work** (errors encountered, confidence, flags), it is a handoff chain — not a simple pipeline.

### Parallel Fan-Out

**Parallel fan-out** is the specific name for when a coordinator simultaneously delegates to multiple subagents. This is what Section 7 calls "spawning subagents in parallel."

```
                    [Coordinator]
                   /      |      \      \
            [Agent A]  [Agent B] [Agent C] [Agent D]
            (topic 1)  (topic 2) (topic 3) (topic 4)
                   \      |      /      /
                    [Coordinator] ← aggregates all results
```

**Implementation:** Multiple `Task` tool calls emitted in a **single** coordinator response.

**When to use fan-out:**
- Subtasks are independent (no data dependency between them)
- Speed matters — fan-out is ~Nx faster than sequential
- Each subtask fits within one subagent context window

**Fan-out vs. sequential — the decision rule:**
```
SubTask B needs SubTask A's output → SEQUENTIAL (prompt chaining)
SubTask B is independent of SubTask A → PARALLEL FAN-OUT
```

---

## 13k. In-Context vs. External Memory

> ⭐ **NEW — Explicitly listed in the official Domain 1 blueprint under "state and session management"**

### The Two Memory Types

| Memory Type | Where It Lives | Lifespan | Access |
|---|---|---|---|
| **In-context memory** | Inside the conversation history / context window | Current session only — lost when session ends | Immediate — no retrieval needed |
| **External memory** | Outside the model — filesystem, DB, vector store | Persists across sessions | Must be retrieved and injected |

### In-Context Memory

Everything in the current conversation window: system prompt, user messages, tool results, assistant responses. It is **immediately available** to Claude without retrieval — but it is **temporary** and accumulates tokens.

**Limits:**
- Lost when session ends
- Grows with every turn (context rot risk)
- Token cost increases with each iteration

### External Memory

Persistent storage outside the model. Three common forms:

| Form | Example | Best For |
|---|---|---|
| **Structured store** (DB/file) | PostgreSQL, JSON files | Precise facts: IDs, amounts, dates, decisions |
| **Vector store** | Pinecone, pgvector | Semantic retrieval of past findings, documents |
| **Filesystem / Git** | Files written by the agent | Code, reports, artifacts that persist across sessions |

### When to Use Which

```
Task requires memory of THIS session only:
→ In-context memory (just keep it in conversation history)

Task spans MULTIPLE sessions or days:
→ External memory — save state at end of session, reload at start

Task requires PRECISE values that must survive summarization:
→ External structured store (IDs, amounts, dates — not prose)

Task requires finding SIMILAR past work:
→ Vector store with semantic search
```

### The Immutable Fact Block Pattern

For long-running agentic tasks, critical facts must be stored externally AND injected as a **structured block at the top of every prompt** — not buried in the summarized history.

```json
// Injected at top of every continuation prompt — never summarized away
{
  "SESSION_FACTS": {
    "customer_id": "CUST-48291",
    "task_id": "TASK-002",
    "authorized_budget": 50000,
    "hard_deadline": "2026-06-01",
    "approved_actions": ["read", "analyze"],
    "forbidden_actions": ["delete", "deploy"]
  }
}
```

**Why "immutable":** These facts never change during the session. They are injected fresh every turn, bypassing the compaction/summarization process that could distort them. This is sometimes called an **immutable fact block** on the exam.

---

## 13l. Anthropic's Production Multi-Agent Lessons (First-Party Source)

> ⭐ **NEW — Directly from Anthropic's engineering blog on their own Research multi-agent system. This is the authoritative source the exam is based on.**

### Token Cost Reality — Know the Numbers

| System Type | Token Multiplier vs Chat |
|---|---|
| Single agent | ~4× more tokens than chat |
| Multi-agent system | ~15× more tokens than chat |

**Exam implication:** Multi-agent systems are only economically viable when the task value justifies the cost. For simple queries, use fewer agents.

### Effort Scaling Rules — Embed in Coordinator Prompt

Agents cannot judge appropriate effort themselves. You must embed explicit scaling rules in the coordinator's system prompt:

```
Simple fact-finding:   → 1 subagent, 3–10 tool calls
Direct comparison:     → 2–4 subagents, 10–15 tool calls each
Complex research:      → 10+ subagents with clearly divided scope
```

**Exam trap:** A coordinator that spawns 50 subagents for a simple query is a failure mode, not a feature. The correct fix is embedding scaling heuristics in the coordinator prompt.

### Parallel Tool Calling Within a Single Subagent

Parallelism exists at **two levels** — both are tested:

| Level | What It Is | How |
|---|---|---|
| **Coordinator fan-out** | Coordinator spawns multiple subagents simultaneously | Multiple Task calls in one response |
| **Subagent parallel tool calls** | A single subagent calls multiple tools at the same time | Multiple tool_use blocks in one subagent response |

```python
# Subagent emitting 3 tool calls in parallel (one response):
[
  WebSearch(query="semiconductor shortage 2025"),
  WebSearch(query="chip supply chain disruption"),
  Read(file="market_report.pdf")
]
# All 3 execute simultaneously — cut research time by up to 90%
```

**Exam rule:** When a scenario describes a subagent executing searches sequentially and it is slow, the fix is parallel tool calling within the subagent — not spawning more subagents.

### Search Strategy: Start Wide, Then Narrow

Agents default to overly specific queries that return few results. The correct search strategy mirrors expert human research:

```
❌ WRONG — overly specific first query
"semiconductor shortage impact on automotive Q3 2021 chip crisis revenue"
→ Returns few results, agent gets stuck

✅ CORRECT — broad first, then narrow
Step 1: "semiconductor shortage" (broad — assess what's available)
Step 2: "automotive chip crisis 2021" (narrowed based on Step 1 results)
Step 3: "Toyota production cuts chip shortage Q3" (targeted follow-up)
```

**Embed this in subagent prompts:** "Start with short, broad queries. Evaluate what's available. Progressively narrow your focus based on results."

### Extended Thinking in Agentic Systems

**Extended thinking** (where Claude outputs a visible internal reasoning process before responding) is particularly effective in multi-agent systems:

| Agent Role | How Extended Thinking Helps |
|---|---|
| Lead/Coordinator agent | Plans approach, assesses tool fit, determines subagent count and scope |
| Subagents | Interleaved thinking after each tool result — evaluates quality, identifies gaps, refines next query |

**Interleaved thinking** = extended thinking inserted between tool calls within a subagent's loop, not just at the start.

**Exam signal:** If a question asks how to improve a subagent's ability to adapt its search strategy mid-task, extended/interleaved thinking is the answer.

### The CitationAgent Pattern

In production multi-agent research systems, citations are handled by a **dedicated CitationAgent** — a separate specialist agent that runs after the research loop completes:

```
[Lead Research Agent] → coordinates research, synthesizes findings
[Research Subagents] → search, gather, return claim-source mappings
        ↓
[CitationAgent] → processes final report + raw documents
               → identifies exact source locations for each claim
               → inserts properly formatted citations
               → returns final cited report
```

**Why separate?** The lead agent focuses on research quality. The CitationAgent focuses on citation accuracy. Mixing both responsibilities in one agent degrades both.

### LLM-as-Judge Evaluation for Multi-Agent Systems

Standard unit tests cannot evaluate multi-agent systems — different valid paths can reach the same correct answer. The correct evaluation approach is **LLM-as-judge** with a structured rubric:

| Criterion | What It Checks |
|---|---|
| **Factual accuracy** | Do claims match their cited sources? |
| **Citation accuracy** | Do cited sources actually contain the attributed claim? |
| **Completeness** | Are all requested aspects of the topic covered? |
| **Source quality** | Were primary sources preferred over low-quality secondary sources? |
| **Tool efficiency** | Did the agent use the right tools a reasonable number of times? |

**Exam trap:** A question may ask how to evaluate a research agent's output. "Check if it followed the correct steps" is wrong — steps are non-deterministic. Correct: LLM-as-judge scoring completeness, accuracy, and citation quality.

### Memory Tool for Context Overflow Prevention

When a lead agent's context approaches the limit (e.g., 200K tokens), it must **save its plan to an external Memory tool** before context gets truncated:

```python
# Lead agent action when context nears limit:
Memory.save({
    "research_plan": "Investigate semiconductor supply chain across 5 angles...",
    "completed_subtasks": ["angle 1 done", "angle 2 done"],
    "remaining_subtasks": ["angle 3", "angle 4", "angle 5"],
    "key_findings_so_far": [...]
})
# Context is then safely truncated — plan survives in Memory
```

**Why this matters:** If the plan is only in the context window and the context gets truncated, the agent loses its strategy mid-task. Persisting to Memory prevents this.

---

## 14. Anti-Patterns Master List

These are the wrong patterns the exam will present as plausible-looking options:

| Anti-Pattern | Why It's Wrong | Correct Approach |
|---|---|---|
| Text-based loop termination | Non-deterministic — Claude's phrasing varies | Use `stop_reason === "end_turn"` |
| Subagents inheriting parent context | Subagents always start fresh | Explicitly pass all needed context |
| Plain text context passing | Loses attribution and structure | Use structured JSON with metadata |
| "Just in case" over-provisioning tools | Violates minimal footprint | Grant only required permissions |
| Prompt-only safety enforcement | Probabilistic — can fail | Use programmatic hooks for safety rules |
| Terminating all subagents on one failure | Kills independent work unnecessarily | Handle error gracefully, continue others |
| Resuming session without noting file changes | Claude reasons from stale data | Explicitly tell Claude what changed |
| Starting long runs on vague tasks | Wastes effort on wrong assumptions | Clarify scope before acting |
| Agent never escalates to human | Dangerous for sensitive/irreversible actions | Define clear escalation thresholds |
| Agent escalates everything | Defeats the purpose of automation | Escalate only at defined thresholds |
| Missing "Task" in allowedTools | Coordinator cannot spawn subagents | Always include "Task" for coordinators |
| Iteration cap as primary loop exit | Should be secondary safety net only | Primary exit = `stop_reason === "end_turn"` |
| Coordinator scope too narrow | Subagents produce correct but incomplete scope | Fix coordinator decomposition breadth |
| Fat subagent returns (raw output) | Coordinator hits context rot after few subagents | Subagents return condensed 1K-2K token summaries |
| Coordinator passes full context to synthesis subagent | Redundant + massively expensive | Coordinator synthesizes directly or passes condensed objects only |
| Sentinel-based loop termination (iteration count) | Fires before task is complete | Only as backup cap — never primary mechanism |
| Sentiment/confidence-score escalation triggers | Probabilistic — will miss edge cases | Use deterministic programmatic triggers only |
| Ignoring context rot in long-running loops | Model degrades silently as context fills | Apply progressive summarization + structured fact blocks |
| Passing raw output in sequential agent chains | Loses error signals, confidence, and context | Use structured handoff packages with metadata |
| Storing critical facts only in prose history | Summarization compresses/distorts precise values | Use immutable fact blocks injected at every prompt |
| Using in-context memory for multi-session tasks | Context is lost when session ends | Write to external memory; reload at session start |
| Returning plain error strings from subagents | Coordinator can't make intelligent recovery decisions | Return structured error with failure_type, is_retryable, partial_results |
| Silently resolving conflicting source data | Hides discrepancies; downstream reports inherit the error | Annotate conflicts with both values and sources, surface to coordinator |
| No crash recovery mechanism for long-running agents | Agent crashes = restart from zero | Write state manifests at checkpoints; reload on restart |
| Plain text claim passing without source evidence | Synthesis agent fabricates citations | Use claim-source mapping: claim + evidence excerpt + source URL + date |
| Sequential tool calls within a subagent when independent | Slow — up to 90% longer than parallel | Emit multiple tool_use blocks in one subagent response |
| Spawning max subagents regardless of query complexity | Massive unnecessary token cost | Embed effort scaling rules: simple=1 agent, complex=10+ agents |
| Coordinator without effort scaling heuristics | Agent spawns 50 subagents for a simple query | Embed explicit scaling rules in coordinator system prompt |
| Starting with overly specific search queries | Returns few results, agent gets stuck | Start broad, evaluate available results, then progressively narrow |
| Lead agent plan held only in context window | Truncation destroys strategy mid-task | Save plan to Memory tool before context approaches limit |
| Same-session self-review after generation | Agent rationalizes its own decisions | Use independent review instance with no generation context |
| Evaluating agents by checking step sequence | Steps are non-deterministic, valid paths vary | Use LLM-as-judge scoring completeness, accuracy, citation quality |
| Sending each tool result as a separate user message | API 400 error — all results for one response must go in one message | Collect all tool_use results, send together in a single user message |
| Adding "Continue." message after pause_turn | Breaks the conversation — server resumes on its own | Re-send conversation as-is; API detects trailing server_tool_use block |
| Using Tool Runner for side-effect tools without validation | SDK executes automatically — no interception point | Use manual loop to insert HITL approval before destructive tool calls |
| Not setting max_continuations on pause_turn loops | Infinite loop risk on server-side tool chains | Always set a max_continuations cap when handling pause_turn |
| Appending tool results before the assistant message | Breaks role alternation — API returns 400 error | Always append assistant message first, then tool results as user message |
| Using prompt instructions for "always/never/must" requirements | Probabilistic — will miss edge cases | Any "always/never/must" requirement → use a hook |
| Using max_turns as the primary loop exit | Fires before task completes — truncates valid work | max_turns = secondary safety cap only; primary exit = stop_reason "end_turn" |
| Fixed routing (always run all subagents) | Unnecessary token cost; simple queries pay for complex pipeline | Dynamic routing — coordinator analyzes query first, selects minimum required subagents |
| Silently omitting gaps in source coverage | Final report is incomplete with no explanation | Annotate missing topic areas with a gap note; surface to coordinator |
| Using heuristic selection when multiple records match | Risk of acting on the wrong customer/record | Request additional identifying information — never guess |
| Using few-shot examples to enforce tool ordering | Probabilistic — reduces violations but doesn't eliminate them | Programmatic prerequisite hooks — deterministic, 100% enforcement |

---

## 15. Key Rules to Memorize

```
1.  stop_reason === "end_turn" is the ONLY correct loop exit signal
2.  stop_reason === "tool_use" → execute tool, append result, continue
3.  Subagents NEVER inherit parent context — always pass explicitly
4.  Pass context as structured JSON — never plain text blobs
5.  "Task" must be in allowedTools for a coordinator to spawn subagents
6.  Parallel subagents = multiple Task calls in ONE coordinator response
7.  Safety/compliance rules → hooks (programmatic), not prompts
8.  Hooks are invoked by the Agent SDK, NOT by Claude
9.  PostToolUse fires AFTER tool executes, BEFORE Claude sees result
10. PreToolUse fires BEFORE tool executes — use for blocking
11. Hub-and-spoke is the default answer for complex multi-agent tasks
12. Pipeline = data flows through sequential agents, no central coordinator
13. Resume = continue same session | Fork = branch from same baseline
14. After file changes → explicitly tell Claude what changed on resume
15. Minimal footprint = grant only required tools/permissions
16. Clarify ambiguity BEFORE starting a long autonomous run
17. Dynamic adaptive = scope unknown until exploration begins
18. Transient errors → retry with backoff | Permanent errors → report/escalate
19. Independent verification = second agent without first agent's context
20. Irreversible actions always require HITL check before execution
21. ReAct loop = Reason → Act → Observe — the formal name for the agentic loop
22. Iteration cap = SECONDARY safety net only — never the primary exit mechanism
23. Incomplete scope in final output → fix the COORDINATOR's decomposition, not the subagents
24. Subagents explore widely (tens of thousands of tokens) but return condensed summaries (1K-2K tokens)
25. Context Rot = model degrades silently as context fills — apply progressive summarization
26. Structured fact blocks (IDs, amounts, dates) must live OUTSIDE summarized prose — they compress into vague language
27. Escalation triggers: explicit request + threshold + retry count = reliable | sentiment + confidence score = unreliable
28. Doom loops: detect by consecutive identical tool calls or consecutive failures → terminate + escalate
29. Do NOT spawn a synthesis subagent just to hand it the coordinator's own full context
30. MCP lets subagents pull only the resources they need — reduces coordinator complexity
31. Handoff chain = sequential agents passing STRUCTURED metadata (errors, confidence, flags) — not just raw output
32. Parallel fan-out = multiple Task calls in one coordinator response for independent subtasks
33. In-context memory = this session only | External memory = persists across sessions
34. Immutable fact block = structured JSON injected at top of every prompt — bypasses summarization, preserves precise values
35. Structured error propagation = failure_type + is_retryable + attempted_query + partial_results — never a plain string
36. Conflicting source data → annotate BOTH values with sources — never silently pick one
37. Claim-source mapping = every fact needs: claim + evidence excerpt + source URL + publication date + retrieval date
38. Crash recovery = write state manifest (completed/pending steps + key findings) at checkpoints; reload on restart
39. Silently returning empty results as "success" when a query failed is always wrong — return structured error with is_retryable
40. Multi-agent systems use ~15× more tokens than chat — only viable for high-value tasks
41. Effort scaling: simple=1 subagent 3-10 calls | comparison=2-4 subagents | complex=10+ subagents — embed in coordinator prompt
42. Parallel tool calling within ONE subagent = multiple tool_use blocks in one response — cuts research time up to 90%
43. Search strategy: start broad, evaluate results, progressively narrow — never start with overly specific queries
44. Extended/interleaved thinking in subagents = evaluates tool result quality and refines next query between tool calls
45. CitationAgent = separate dedicated agent that inserts citations AFTER research loop completes — not the lead agent's job
46. LLM-as-judge = correct evaluation method for multi-agent output — checks factual accuracy, citation accuracy, completeness, source quality, tool efficiency
47. Save lead agent plan to Memory tool before context limit — truncation destroys strategy if plan is only in context window
48. 5th stop_reason: "pause_turn" = server-side tool loop hit 10-iteration cap — re-send conversation as-is, do NOT add new user message
49. Multiple tool_use blocks in one response → execute ALL tools → return ALL results in ONE user message with matching tool_use_ids
50. tool_choice "auto" = Claude decides | "any" = must call a tool | "tool" = must call specific tool | "none" = no tools allowed
51. disable_parallel_tool_use: true → forces Claude to emit only one tool_use block per response
52. Tool Runner = SDK auto-loop (use for normal tools) | Manual loop = use when you need HITL approval before each tool call
53. is_error: true in tool_result = tell Claude the tool failed; Claude will try a different approach or ask for clarification
54. Order of operations: append assistant message FIRST, then tool results as user message — never skip the assistant turn
55. max_turns and max_budget_usd = SDK safety caps (secondary only); primary loop exit is always stop_reason "end_turn"
56. Hook keyword trigger: if requirement says "always/never/must/every time/guarantee" → hook, never prompt instruction
57. Hooks are fully auditable (logged as code events); prompt compliance is invisible and cannot be audited
58. META-PATTERN: every wrong exam answer substitutes a probabilistic mechanism for a deterministic one
59. Dynamic routing = coordinator analyzes query first then selects subagents; fixed routing = always runs all subagents (anti-pattern)
60. 6 exam scenarios: 4 are randomly selected — Scenarios 1, 3, 4 are highest Domain 1 weight (customer support, research, dev tools)
61. Silently omitting topic areas with no source coverage is wrong — annotate the gap and surface it to the coordinator
62. When multiple customer records match a query, request additional identifiers — never use heuristic selection to pick one
63. Exam format: 60 MCQs, 120 minutes, no breaks, no external resources, passing score 720/1000, results in 2 business days
64. Stateless subagents + stateful coordinator = correct hub-and-spoke — subagents get clean context; coordinator maintains the full picture
65. When starting a fresh session after a phase transition, inject a structured summary of decisions made — not the full deliberation history
66. Prior context becomes stale when: fundamental assumption wrong, major architecture changed, 50%+ files modified, or task domain changed
```

---

## 16. Practice Questions (20 MCQs)

---

**Q1.** An agentic loop receives a response from Claude with `stop_reason: "tool_use"`. What is the correct next action?

- A) Exit the loop — the task is complete
- B) Execute the requested tool, append the result to conversation history, and send the updated history back to Claude
- C) Start a new conversation with the tool result as the first message
- D) Check if the response text says "calling tool" before executing

---

**Q2.** A customer support system has a rule: never process refunds over $1,000. The rule is written in the system prompt. A customer requests a $1,200 refund and the agent processes it. What is the root cause?

- A) The model is too weak — upgrade to a more powerful version
- B) The system prompt was too long and the instruction got diluted
- C) Prompt-based rules are probabilistic and unreliable for safety-critical limits — the rule should be enforced via a PreToolUse hook
- D) The temperature setting was too high causing unpredictable behavior

---

**Q3.** A coordinator agent is configured with `allowed_tools=["Read", "Grep", "Glob"]`. It attempts to spawn three subagents to process document sections in parallel. What happens?

- A) The subagents spawn successfully but run sequentially
- B) The coordinator cannot spawn subagents because "Task" is missing from allowedTools
- C) The coordinator spawns the subagents but they share the coordinator's context
- D) The system crashes due to a token limit from three parallel context windows

---

**Q4.** You are building a research system. The coordinator finds that Company A acquired Company B — information not anticipated in the original task. What should a well-designed agentic system do?

- A) Stop and report to the user that the scope has changed
- B) Ignore the acquisition information and complete the original task
- C) Replan dynamically — add research on Company B to the task plan
- D) Start a new session and pass a summary of findings so far

---

**Q5.** A developer resumes a Claude Code session. Since the last session, two core files were significantly refactored. The developer runs `--resume my-session` without any additional context. What is the most likely problem?

- A) Claude will refuse to resume because file checksums don't match
- B) Claude will automatically re-read all files and update its analysis
- C) Claude will reason from its stale previous analysis of the two changed files, silently producing incorrect results
- D) Claude will ask the developer which files changed before proceeding

---

**Q6.** Which task decomposition pattern is MOST appropriate for: "Refactor a large legacy codebase — we don't know how many files need changes until we explore it"?

- A) Sequential prompt chaining — process files one by one in a fixed order
- B) Parallel decomposition — analyze all files simultaneously
- C) Dynamic adaptive decomposition — explore first, then plan based on what's found
- D) Hub-and-spoke — delegate each file to a separate subagent from the start

---

**Q7.** A coordinator agent needs to run three independent research subagents as fast as possible. What is the correct implementation?

- A) Spawn subagent 1, wait for it to finish, spawn subagent 2, wait, spawn subagent 3
- B) Emit three separate Task tool calls across three consecutive coordinator responses
- C) Emit three Task tool calls in a single coordinator response so they run in parallel
- D) Use fork_session three times to create parallel branches

---

**Q8.** A PostToolUse hook is configured. When exactly does it fire?

- A) Before the tool executes — to validate inputs
- B) After the tool executes and before Claude sees the result — to normalize or enrich the data
- C) After Claude processes the tool result — to log the outcome
- D) When Claude decides which tool to call — to validate the selection

---

**Q9.** A synthesis subagent must produce a report with citations. The coordinator passes context as a plain text summary. What is the problem?

- A) Plain text is too long and will exceed the subagent's context window
- B) Plain text loses source attribution — the subagent cannot cite sources it has no structured record of
- C) Plain text cannot be parsed by the Claude Agent SDK
- D) There is no problem — plain text is the recommended format for coordinator-to-subagent communication

---

**Q10.** Which scenario correctly applies the minimal footprint principle?

- A) A file analysis agent is given Read, Write, Delete, and Admin access so it never hits permission errors
- B) A file analysis agent that only needs to read files is given only Read access
- C) All agents are given identical tool access for consistency across the system
- D) Tool access is determined at runtime based on what the agent requests

---

**Q11.** An orchestrator spawns four subagents in parallel. One subagent encounters a permission error and fails. What should the orchestrator do?

- A) Immediately terminate all four subagents and return an error to the user
- B) Ignore the failure silently and compile results from the three successful subagents
- C) Log the error, continue processing the three remaining subagents, and include the partial failure in the final result
- D) Retry the failed subagent indefinitely until it succeeds

---

**Q12.** A user says "clean up the codebase" to an agentic system. The agent immediately starts deleting unused files, reformatting code, and refactoring functions. What principle did the agent violate?

- A) Minimal footprint — the agent used more tools than necessary
- B) The agent should have clarified the scope of "clean up" before starting a long autonomous run
- C) The agent should have used parallel decomposition instead of sequential processing
- D) The agent should have created subagents instead of doing the work itself

---

**Q13.** What is the key architectural reason that using two independent agents for verification is better than asking one agent to verify its own work?

- A) Two agents are always faster than one
- B) The second agent has no knowledge of the first agent's reasoning chain, making it more likely to catch blind spots
- C) Two agents reduce token usage by splitting the context window
- D) It complies with the minimal footprint principle

---

**Q14.** A customer explicitly says "I want to speak to a real person." What should a correctly designed agentic customer support system do?

- A) Continue handling the issue autonomously — the agent might resolve it faster
- B) Ask the customer why they want a human before deciding whether to escalate
- C) Immediately escalate to a human agent — explicit customer request is a mandatory escalation trigger
- D) Attempt one more resolution before escalating

---

**Q15.** Which statement about the hub-and-spoke multi-agent pattern is CORRECT?

- A) Subagents communicate directly with each other to share findings efficiently
- B) All inter-subagent communication routes through the coordinator
- C) The coordinator executes all tool calls directly without using subagents
- D) Hub-and-spoke is only appropriate for sequential workflows

---

**Q16.** A developer wants to try two completely different architectural approaches to solving a problem, starting from the same point in a Claude Code session. Which session operation is correct?

- A) `--resume` twice with different flags
- B) Start two new sessions with identical injected summaries
- C) `fork_session` — creates two independent branches from the same baseline
- D) Create two coordinators that share the same context window

---

**Q17.** A network timeout occurs when a subagent calls an external API. What is the correct retry strategy?

- A) Immediately retry up to 10 times without any delay
- B) Report a permanent error to the user — network errors cannot be recovered
- C) Retry with exponential backoff — wait 1s, 2s, 4s... with a maximum retry limit
- D) Terminate the entire agentic loop and restart from the beginning

---

**Q18.** What is the correct description of how hooks are invoked in the Claude Agent SDK?

- A) Claude calls hooks directly when it decides a tool call needs validation
- B) Hooks are invoked by the Agent SDK at specific loop points — not by Claude
- C) Hooks are triggered by the user's system prompt instructions
- D) Hooks fire only when an error occurs in a tool call

---

**Q19.** A multi-step pipeline requires: (1) extract text from a PDF, (2) translate the text to French using the extracted content, (3) summarize the French translation. What execution pattern is correct?

- A) Parallel — run all three steps simultaneously for maximum speed
- B) Sequential — each step depends on the output of the previous step
- C) Hub-and-spoke — delegate each step to an independent specialized subagent
- D) Dynamic adaptive — explore the PDF first to decide how many steps are needed

---

**Q20.** A coordinator's system prompt says "you are responsible for delegating research tasks to subagents." The coordinator tries to spawn a subagent but nothing happens. The developer checks and finds `allowed_tools=["Read", "WebSearch"]`. What is wrong and how do you fix it?

- A) The system prompt is too vague — rewrite it to be more specific about subagent creation
- B) WebSearch conflicts with the Task tool — remove WebSearch from allowedTools
- C) "Task" is missing from allowedTools — add it so the coordinator can spawn subagents
- D) The coordinator needs admin-level access to spawn subagents — upgrade permissions

---

**Q21.** A multi-agent research system produces reports that frequently miss important subtopics. The coordinator delegates to a web search agent and a synthesis agent in one round. What is the correct architectural fix?

- A) Add more subagents to cover more topics simultaneously
- B) Implement an iterative refinement loop — the coordinator evaluates synthesis output for gaps and re-delegates with targeted queries until coverage is sufficient
- C) Increase the context window size so subagents can process more information per call
- D) Switch from hub-and-spoke to a pipeline pattern for better information flow

---

**Q22.** A coordinator spawns two subagents to research "AI in healthcare." Both agents independently search the same news sources and produce 80% overlapping findings, doubling token costs without improving coverage. What is the correct fix?

- A) Run the agents sequentially so the second agent can see what the first found
- B) Let both agents finish in parallel, then have the coordinator deduplicate results before synthesis
- C) The coordinator must partition scope upfront — assign distinct subtopics or source types to each agent before delegation
- D) Reduce to one research agent to eliminate duplication entirely

---

**Q23.** A customer support agent escalates a $1,200 refund to a human agent with the message: "Customer needs help with a refund — please assist." The human agent has no access to the conversation transcript. What is wrong and what is the correct fix?

- A) Nothing is wrong — the human agent will ask the customer to repeat the issue
- B) The escalation message is too vague — it should include a structured handoff summary with customer ID, root cause, refund amount, reason for escalation, and recommended action
- C) The agent should not escalate — it should keep attempting to resolve the issue autonomously
- D) The escalation message should include the full conversation transcript copy-pasted

---

**Q24.** A coordinator's system prompt says: "Step 1: search the web. Step 2: analyze the documents. Step 3: synthesize the findings." During execution, step 2 returns no useful documents. What problem does this coordinator design create?

- A) No problem — fixed steps ensure consistent, predictable output
- B) The agent follows the steps rigidly and produces a poor synthesis, because procedural instructions remove adaptability when intermediate steps yield nothing
- C) The agent will automatically skip step 2 and proceed to step 3
- D) The agent will ask the user to provide documents for step 2

---

**Q25.** A synthesis subagent in a multi-agent research system keeps attempting web searches even though it is only supposed to synthesize findings passed to it by the coordinator. What is the root cause and correct fix?

- A) The coordinator's system prompt is not clear enough — rewrite it to tell the coordinator to stop the subagent from searching
- B) The AgentDefinition for the synthesis subagent has too broad an allowed_tools list and an insufficiently specific system_prompt — restrict allowed_tools to synthesis-only tools and tighten the system_prompt
- C) The synthesis subagent needs more context passed to it so it stops looking for information externally
- D) Switch from hub-and-spoke to pipeline so the synthesis agent receives data in a controlled sequence

---

**Q26.** A customer support agent sometimes calls `process_refund` before `get_customer` has returned a verified customer ID. The system prompt says "always verify the customer before processing refunds." The problem keeps occurring. What is the correct architectural fix?

- A) Rewrite the system prompt with stronger language and add "IMPORTANT:" before the instruction
- B) Implement a prerequisite gate that programmatically blocks process_refund from executing until get_customer has returned a verified customer ID
- C) Add a PostToolUse hook on get_customer to flag when verification completes
- D) Upgrade to a more powerful model that better follows sequential instructions

---

## 17. Answer Key & Explanations

| Q | Answer | Key Reason |
|---|---|---|
| 1 | **B** | Execute the tool, append result to history, continue the loop |
| 2 | **C** | Prompt rules are probabilistic — use PreToolUse hook for safety-critical limits |
| 3 | **B** | "Task" is required in allowedTools for subagent spawning |
| 4 | **C** | Agentic systems replan dynamically based on new discoveries |
| 5 | **C** | Claude doesn't detect file changes — it reasons from stale session analysis |
| 6 | **C** | Unknown scope until exploration = dynamic adaptive decomposition |
| 7 | **C** | Parallel subagents = multiple Task calls in ONE response |
| 8 | **B** | PostToolUse fires after tool executes, before Claude sees the result |
| 9 | **B** | Plain text loses attribution — structured JSON preserves source metadata |
| 10 | **B** | Minimal footprint = only required permissions, nothing more |
| 11 | **C** | Handle gracefully — log error, continue independent subagents, report partial failure |
| 12 | **B** | Clarify ambiguity BEFORE starting a long autonomous run |
| 13 | **B** | Fresh context = no anchoring bias from the first agent's reasoning |
| 14 | **C** | Explicit customer request for human = mandatory escalation trigger |
| 15 | **B** | All inter-subagent communication routes through the coordinator |
| 16 | **C** | fork_session creates independent branches from the same baseline |
| 17 | **C** | Transient errors → exponential backoff with a max retry limit |
| 18 | **B** | Hooks are invoked by the Agent SDK, not by Claude |
| 19 | **B** | Each step depends on the previous output = sequential |
| 20 | **C** | Add "Task" to allowedTools — it is required for subagent spawning |
| 21 | **B** | Iterative refinement loop — coordinator re-delegates until coverage is sufficient |
| 22 | **C** | Partition scope BEFORE delegation — not deduplicate after |
| 23 | **B** | Structured handoff with customer ID, root cause, amount, reason, recommended action |
| 24 | **B** | Procedural step-by-step instructions remove adaptability when steps fail |
| 25 | **B** | AgentDefinition system_prompt + allowed_tools defines subagent scope — fix it there |
| 26 | **B** | Prerequisite gate blocks downstream tool until upstream step completes — prompts are probabilistic |

---

## Quick Cheat Sheet — Domain 1

```
LOOP:        stop_reason "tool_use" → execute → continue
             stop_reason "end_turn" → EXIT
SUBAGENTS:   Always fresh context | Explicit passing | Structured JSON
SPAWNING:    "Task" in allowedTools | Multiple Tasks in ONE response = parallel
AGENT DEF:   AgentDefinition sets name, system_prompt, allowed_tools per subagent
HOOKS:       PreToolUse = block | PostToolUse = normalize | SDK invokes, not Claude
PREREQ:      Prerequisite gates block downstream tools until upstream step completes
PATTERNS:    Hub-and-spoke (default) | Pipeline (sequential transform) | P2P (rare)
DECOMPOSE:   Chaining (known steps) | Parallel (independent) | Dynamic (unknown scope)
REFINEMENT:  Coordinator re-delegates until quality criteria met — not one-shot
PARTITION:   Split scope BEFORE delegation — never deduplicate after
SESSION:     Resume (continue) | Fork (branch) | New+summary (stale context)
SAFETY:      Hooks for compliance | Prompts for style | Clarify before long runs
FOOTPRINT:   Only required tools | Only required permissions | Nothing more
ESCALATE:    Threshold | Customer request | Repeated failure | Irreversible action
HANDOFF:     Structured summary: customer ID + root cause + amount + recommendation
```

---

*CCA-F Domain 1 Study Guide | Prepared for Arun | May 2026*
*Next: Domain 2 — Tool Design & MCP Integration (18%)*

---

---

# 🧠 Extended Practice Question Bank — Domain 1
> **100 Additional Questions** covering every subtopic, anti-pattern, and edge case
> Organized by subtopic for targeted practice

---

## Section A — Agentic Loop & stop_reason (Q27–Q45)

---

**Q27.** Claude returns a response with `stop_reason: "max_tokens"`. What is the correct action?

- A) Treat it as `end_turn` — the task is complete
- B) Crash the loop and restart from the beginning
- C) Handle gracefully — do not crash. Summarize what was done and inform the user the output was truncated
- D) Retry the exact same request immediately without any changes

---

**Q28.** An agentic loop is running. Claude returns a response containing both a text block and a tool_use block in the same response. What does this mean and what should you do?

- A) This is an error — Claude should never return both text and tool_use in the same response
- B) Ignore the text block and only process the tool_use block — execute the tool and continue the loop
- C) Process only the text block — the tool_use block is supplementary
- D) Return an error to the user asking them to clarify

---

**Q29.** A developer sets an arbitrary loop counter: "if loop runs more than 10 times, exit." What is wrong with this approach?

- A) Nothing — 10 iterations is a reasonable safety limit for all use cases
- B) It is fine as a secondary safety net but should never be the PRIMARY stopping mechanism — always use stop_reason first
- C) The limit should be 5, not 10 — 10 iterations always exceeds the context window
- D) Loop counters improve reliability and should be the primary stopping mechanism

---

**Q30.** In an agentic loop, after executing a tool, where exactly do you place the tool result before sending back to Claude?

- A) As a new system message at the top of the conversation
- B) Replacing the previous assistant message entirely
- C) Appended to the conversation history as a user-role message after the assistant's tool_use message
- D) In a separate API call with only the tool result — not the full history

---

**Q31.** Claude calls two tools in a single response (parallel tool calls). How do you handle this correctly?

- A) Execute only the first tool — Claude cannot reliably handle multiple tool results
- B) Execute both tools, append both tool results to conversation history, then send everything back in one API call
- C) Execute the tools one at a time, sending a separate API call for each tool result
- D) Return an error — Claude should not call multiple tools in one response

---

**Q32.** What is the key distinction between model-driven decision-making and a pre-configured decision tree?

- A) Model-driven is faster; decision trees are more accurate
- B) In model-driven, Claude reasons about which tool to call next based on context; in decision trees, the tool sequence is hardcoded regardless of what Claude finds
- C) Decision trees use Claude; model-driven uses traditional rule engines
- D) There is no meaningful difference — both approaches produce the same outcomes

---

**Q33.** An agentic loop exits when Claude returns `stop_reason: "stop_sequence"`. When does this occur?

- A) When Claude detects that the task is logically complete
- B) When Claude hits a specific token or character sequence you configured as a stop trigger
- C) When Claude calls more than the maximum allowed number of tools
- D) When Claude's confidence in its response drops below a threshold

---

**Q34.** A developer strips the conversation history between loop iterations to save tokens. What is the consequence?

- A) No consequence — Claude can infer context from tool results alone
- B) Claude loses all reasoning context and cannot make informed decisions about what to do next
- C) This is the recommended approach for long-running agents to prevent context overflow
- D) Claude will automatically reconstruct its context from the tool results

---

**Q35.** An agent successfully calls `get_customer`, receives the result, appends it to history, and sends back to Claude. Claude then calls `lookup_order`. The tool fails with a timeout. What should happen next?

- A) Exit the loop and report failure to the user
- B) Retry `lookup_order` with exponential backoff — timeouts are transient errors
- C) Skip `lookup_order` and proceed to the next step
- D) Restart the entire loop from the beginning including `get_customer`

---

**Q36.** Which of the following is the MOST reliable way to determine if an agentic task has completed successfully?

- A) Check if Claude's response text length is greater than 100 characters
- B) Count the number of tool calls made and compare to expected count
- C) Check `stop_reason === "end_turn"` — this is the structured API signal for task completion
- D) Ask Claude "are you done?" and check for "yes" in the response

---

**Q37.** An agentic loop has been running for 45 minutes processing a large document. Claude returns `stop_reason: "max_tokens"`. What is the best recovery strategy?

- A) Discard all work and restart from scratch with a smaller document
- B) Summarize the work completed so far, inject that summary into a new session, and continue from where it left off
- C) Increase the max_tokens limit and retry the exact same request
- D) Split the document in half and run two separate unrelated loops

---

**Q38.** A developer checks for `end_turn` but also adds: "if response text contains 'ERROR', exit the loop." Why is this second condition problematic?

- A) It is not problematic — defensive programming is always good practice
- B) "ERROR" might appear in a tool result or legitimate response text unrelated to loop state, causing premature exits
- C) The word "ERROR" is reserved by the Claude API and cannot appear in responses
- D) Text-based checks add latency to the loop

---

**Q39.** Which stop_reason should you NEVER silently ignore?

- A) `"stop_sequence"` — it always means success
- B) `"max_tokens"` — the output was truncated and may be incomplete, which must be handled explicitly
- C) `"end_turn"` — it might be a false positive
- D) `"tool_use"` — it rarely needs to be acted on

---

**Q40.** A loop runs correctly but Claude keeps calling the same tool repeatedly without making progress. What is most likely wrong?

- A) The tool description is too long — shorten it
- B) The tool result is not being appended to the conversation history, so Claude has no memory of already calling the tool
- C) Claude is malfunctioning — restart the process
- D) The model temperature is too high — lower it to 0

---

**Q41.** In a multi-turn agentic loop, what is the role of the "user" message that contains tool results?

- A) It signals to Claude that the user has approved the tool call
- B) It provides Claude with the tool output so Claude can reason about what to do next
- C) It resets the conversation context for the next iteration
- D) It is optional — Claude can proceed without tool results

---

**Q42.** An agentic system has been given a task: "monitor this API endpoint every 5 minutes and alert me if it goes down." How should this be implemented?

- A) A single agentic loop running indefinitely with a sleep timer inside
- B) A scheduled external trigger (cron job, event scheduler) that launches a fresh agentic loop each time — not an infinite loop inside the agent
- C) An agentic loop with `stop_reason` checked against a time limit
- D) A conversational system that asks the user to check in every 5 minutes

---

**Q43.** Claude makes a tool call but the tool returns an empty result (not an error — just no data found). What should a well-designed agent do?

- A) Treat empty results as errors and retry immediately
- B) Append the empty result to history, let Claude reason about it, and continue — Claude should decide what to do next based on context
- C) Exit the loop — no data means the task cannot be completed
- D) Call the same tool again with identical parameters

---

**Q44.** What happens if you send Claude a tool_result without a corresponding tool_use in the conversation history?

- A) Claude ignores it and continues normally
- B) The API returns a validation error — tool results must always match a preceding tool_use block
- C) Claude treats it as a user message
- D) Claude automatically creates a tool_use entry to match it

---

**Q45.** A loop is designed to research 10 topics sequentially. After topic 3, Claude returns `end_turn` instead of calling the next research tool. What is the most likely cause?

- A) Claude hit the maximum tool call limit
- B) Claude's reasoning determined the task was complete — possibly misunderstanding the scope, or the system prompt was ambiguous about the full list of topics
- C) The API rate limit was reached
- D) The loop counter reached its maximum

---

## Section B — Multi-Agent Patterns & Coordinator Design (Q46–Q62)

---

**Q46.** A coordinator agent receives a complex research request. Instead of always routing through all subagents, it analyzes the query first and only invokes the relevant subagents. What principle does this follow?

- A) Minimal footprint — only use tools you need
- B) Dynamic subagent selection — coordinator analyzes requirements and selects which subagents to invoke based on query complexity
- C) Pipeline pattern — agents run in sequence
- D) Peer-to-peer coordination — agents self-select tasks

---

**Q47.** In a hub-and-spoke system, Subagent A finishes and has findings that Subagent B needs. What is the CORRECT way to share this information?

- A) Subagent A directly calls Subagent B and passes the findings
- B) Subagent A writes findings to a shared database that Subagent B reads
- C) Subagent A returns findings to the coordinator, which then explicitly passes them in Subagent B's prompt
- D) Both subagents share a common context window

---

**Q48.** When should you use a pipeline pattern instead of hub-and-spoke?

- A) When you need a central coordinator making decisions
- B) When data needs to be transformed sequentially through agents with no coordination decisions needed between steps
- C) When subagents need to run in parallel
- D) When the task scope is unknown upfront

---

**Q49.** A research system produces reports missing key subtopics. Investigation reveals the coordinator always routes every query through all 4 subagents regardless of the question. What is the problem?

- A) Too many subagents — reduce to 2
- B) The coordinator is not dynamically selecting which subagents to invoke based on what each query actually needs — it is blindly routing through the full pipeline
- C) Subagents have too much context — reduce their token limits
- D) The synthesis agent is missing — add one

---

**Q50.** A multi-agent research system has a web search agent and a document analysis agent both assigned "research AI trends." Both return 80% overlapping results. What is the FIRST thing the coordinator should fix?

- A) Merge both agents into one
- B) Run them sequentially so the second agent can avoid what the first found
- C) Partition the research scope before delegation — assign distinct source types or subtopics to each agent
- D) Add a deduplication agent after both finish

---

**Q51.** Why is routing all subagent communication through the coordinator important for observability?

- A) It reduces token usage by centralizing context
- B) It ensures the coordinator has full visibility into what each subagent found, enabling consistent error handling and controlled information flow across the system
- C) It prevents subagents from calling each other's tools
- D) It is not important — direct subagent communication is more efficient

---

**Q52.** A coordinator delegates a broad research topic to a subagent with the instruction "research everything about AI." The subagent returns shallow, unfocused results. What is the root cause?

- A) The subagent's tools are not powerful enough
- B) The coordinator's task decomposition was too broad — it should have broken the topic into specific, bounded subtasks before delegating
- C) The subagent needs more context window space
- D) The coordinator should have used a pipeline pattern instead

---

**Q53.** You have a task where Agent 1 writes code, and Agent 2 independently reviews it for bugs without seeing Agent 1's reasoning. What pattern is this and why is it effective?

- A) Pipeline — Agent 2 transforms Agent 1's output sequentially
- B) Independent verification — Agent 2 reviews without Agent 1's reasoning bias, making it more likely to catch blind spots
- C) Hub-and-spoke — the coordinator delegates to both agents
- D) Peer-to-peer — both agents collaborate as equals

---

**Q54.** A coordinator spawns Subagent A and Subagent B. Subagent A fails. Subagent B's work is independent of Subagent A. What should the coordinator do?

- A) Terminate Subagent B immediately — partial results are useless
- B) Wait for Subagent A to be retried before allowing Subagent B to continue
- C) Let Subagent B continue and complete, include Subagent B's results in the final output, and report the Subagent A failure separately
- D) Restart both subagents from scratch

---

**Q55.** What is the key risk of "overly narrow task decomposition" by a coordinator?

- A) It uses too many tokens per subagent call
- B) Subtasks are so narrowly defined that the gaps between them leave important parts of the topic unresearched
- C) It forces sequential execution when parallel would be faster
- D) It causes subagents to exceed their context window limits

---

**Q56.** A coordinator needs to aggregate results from 3 subagents. Subagent 1 and 2 succeeded. Subagent 3 partially succeeded (got some data but hit a rate limit). What is the correct aggregation approach?

- A) Discard Subagent 3's partial results — partial data is unreliable
- B) Include Subagent 3's partial results with a clear annotation indicating what was retrieved and what was missed, then synthesize all available data
- C) Retry Subagent 3 indefinitely until it fully succeeds
- D) Only use Subagents 1 and 2 — three-agent results are always inconsistent

---

**Q57.** A peer-to-peer multi-agent pattern is rarely used. When is it genuinely appropriate?

- A) When tasks can be parallelized — use peer-to-peer for speed
- B) When agents need to negotiate or debate as equals — such as two agents presenting opposing arguments for a decision
- C) When you want to avoid using a coordinator to save tokens
- D) When one agent must verify another agent's work

---

**Q58.** A multi-agent system where Agent A preprocesses data, Agent B analyzes it, and Agent C summarizes it — each receiving the output of the previous — is BEST described as:

- A) Hub-and-spoke with three subagents
- B) Parallel decomposition with three independent workers
- C) A pipeline — each agent transforms the output of the previous sequentially
- D) Dynamic adaptive decomposition

---

**Q59.** The coordinator's system prompt says "always invoke all four subagents for every request." A simple factual query comes in that only needs the knowledge-base subagent. What problem does this create?

- A) No problem — using all subagents ensures comprehensive coverage
- B) Unnecessary token usage, increased latency, and potential for conflicting results from irrelevant subagents
- C) The coordinator will error because not all subagents will have relevant data
- D) Subagents will compete for the same tools

---

**Q60.** Why must the coordinator handle error routing centrally rather than letting subagents handle their own errors independently?

- A) Subagents cannot access error handling libraries
- B) Centralized error routing gives the coordinator full visibility, enables consistent retry/escalation decisions, and prevents silent failures that corrupt the final output
- C) It is more efficient — one error handler is faster than four
- D) Subagents should handle their own errors — the coordinator only aggregates successes

---

**Q61.** A coordinator is designed to research topics. For a simple lookup query, it spawns all 5 research subagents. For a complex multi-dimensional analysis, it spawns only 2. What is wrong?

- A) The coordinator should always spawn the same number of subagents for consistency
- B) The coordinator's subagent selection logic is inverted — simple queries need fewer subagents, complex queries need more. The coordinator is not dynamically selecting based on actual query requirements
- C) 5 subagents is always too many — reduce to 3 maximum
- D) Nothing is wrong — the coordinator is adapting appropriately

---

**Q62.** When a synthesis subagent produces a report with gaps, what is the coordinator's correct next action in an iterative refinement loop?

- A) Accept the report as-is — the subagent did its best
- B) Evaluate the gaps, formulate targeted re-queries, re-delegate to the search and analysis subagents with specific instructions to fill those gaps, then re-invoke synthesis
- C) Add more subagents to compensate for the gaps
- D) Start a completely new session with fresh subagents

---

## Section C — Subagent Spawning, Context Passing & AgentDefinition (Q63–Q75)

---

**Q63.** A coordinator's `allowed_tools` includes `["Read", "WebSearch", "Task", "Write"]`. The coordinator is a research coordinator that should never write files. What principle is violated and what should be done?

- A) No violation — extra tools don't cause harm
- B) Minimal footprint is violated — remove "Write" from the coordinator's allowed_tools since it has no need to write files
- C) The coordinator needs Write access to save research findings
- D) Task tool conflicts with Write — remove Task instead

---

**Q64.** What is the AgentDefinition `description` field used for?

- A) It is displayed to the end user to explain what the agent does
- B) It is used by the coordinator to select which subagent to invoke for a given task — a vague description leads to wrong subagent selection
- C) It sets the maximum response length for the subagent
- D) It controls which MCP servers the subagent can access

---

**Q65.** A coordinator must pass web search results AND document analysis results to the synthesis subagent. What is the correct format?

- A) Concatenate all results into one long string and pass as plain text
- B) Pass each result set as a separate structured object with source metadata (URL, document name, page number, retrieved_at) so attribution is preserved
- C) Summarize all results into a short paragraph before passing to save tokens
- D) Pass only the most recent result — the synthesis agent will request more if needed

---

**Q66.** Why should coordinator prompts specify goals and quality criteria rather than step-by-step instructions?

- A) Step-by-step instructions make the coordinator more predictable and reliable
- B) Goals and quality criteria allow the coordinator to adapt its approach when intermediate steps fail or yield unexpected results — procedural instructions create brittle systems
- C) Step-by-step instructions use more tokens
- D) Goals-based prompts are easier to write — step-by-step is unnecessarily complex

---

**Q67.** A subagent is supposed to only analyze uploaded PDF documents but keeps trying to search the web. Where is the CORRECT place to fix this?

- A) In the coordinator's system prompt — tell it to instruct the subagent better
- B) In the AgentDefinition for this subagent — restrict `allowed_tools` to document-reading tools only, and tighten the `system_prompt` to explicitly exclude web search
- C) Add a PostToolUse hook to block web search results after they're returned
- D) Increase the subagent's max_tokens so it has enough space to read documents without needing web search

---

**Q68.** A coordinator needs to spawn 4 subagents. The developer writes: spawn subagent 1, wait for result, spawn subagent 2, wait, spawn subagent 3, wait, spawn subagent 4. Total time: 40 seconds. What is the optimization?

- A) Reduce to 2 subagents — 4 is always too many
- B) Emit all 4 Task tool calls in a single coordinator response so they run in parallel — total time drops to ~10 seconds
- C) Use fork_session 4 times to create parallel branches
- D) Cache the results of subagent 1 and reuse for subagents 2, 3, and 4

---

**Q69.** A subagent's AgentDefinition has `system_prompt: "You are a helpful assistant."` and `allowed_tools: ["Read", "Write", "Bash", "WebSearch", "Execute"]`. What are the two problems?

- A) The system prompt is too short; the tool list is too long — both violate minimal footprint and the vague prompt gives the subagent no role clarity
- B) The system prompt should be in JSON format; tool names are case-sensitive
- C) No problems — flexible agents are better for production systems
- D) The system prompt needs a persona name; Execute is not a valid tool

---

**Q70.** When passing context from a web search subagent to a synthesis subagent, which fields are ESSENTIAL to include for proper attribution?

- A) Only the text content — synthesis agents don't need metadata
- B) Content, source URL, source title, and retrieval timestamp — these are required for proper citation in the final report
- C) Only the URL — the synthesis agent can fetch the content itself
- D) Content and word count — to help the synthesis agent prioritize

---

**Q71.** A coordinator spawns subagents by emitting Task tool calls across 3 separate turns (one per turn). What is wrong?

- A) Nothing — sequential spawning ensures ordered execution
- B) Subagents spawned across separate turns run sequentially, not in parallel — emit all Task calls in a single response for true parallel execution
- C) The Task tool can only be called once per session
- D) Separate turns cause context pollution between subagents

---

**Q72.** A subagent receives this context from the coordinator: "Research company X and find their revenue." The subagent has no other information. What critical information is missing from the coordinator's delegation?

- A) Nothing — the task is clear
- B) The coordinator's prior findings, the scope of research (which sources to check), quality criteria (what level of detail is needed), and any constraints (time period, geography)
- C) The user's name and session ID
- D) The maximum number of search queries allowed

---

**Q73.** What is fork_session used for in the context of subagent management?

- A) Creating a subagent that runs faster than normal subagents
- B) Creating independent branches from a shared analysis baseline to explore divergent approaches — changes in each branch don't affect the other
- C) Forking the coordinator's context to multiple subagents simultaneously
- D) Creating a backup session in case the primary session fails

---

**Q74.** A developer wants two different synthesis strategies applied to the same research findings. Both strategies should start from the same analysis baseline. Which approach is correct?

- A) Run Strategy A, then pass its output to Strategy B sequentially
- B) Create two independent parallel subagents — both will independently develop their approaches from scratch
- C) Use fork_session to create two independent branches from the shared baseline, apply Strategy A in one branch and Strategy B in the other
- D) Run Strategy A and B in the same subagent prompt separated by "---"

---

**Q75.** A coordinator passes context to a synthesis subagent as a raw string: `"Google was founded in 1998 by Sergey Brin and Larry Page. Microsoft revenue is $200B. Apple market cap is $3T."` What is the problem with this approach?

- A) The string is too short — synthesis agents need more context
- B) Plain text concatenation loses source attribution — the synthesis agent cannot cite which source each fact came from, making citations in the report impossible or inaccurate
- C) The string contains competitor information which may confuse the agent
- D) There is no problem — plain text is the standard format for inter-agent communication

---

## Section D — Hooks, Enforcement & Prerequisite Gates (Q76–Q88)

---

**Q76.** A PostToolUse hook receives a tool result where `timestamp` is in Unix epoch format (e.g., `1715000000`). The Claude model expects ISO 8601 format. What should the hook do?

- A) Pass the Unix timestamp directly — Claude can interpret both formats
- B) Convert the Unix timestamp to ISO 8601 format before returning the result so Claude always receives a consistent format
- C) Log the format mismatch and return an error to the coordinator
- D) Strip the timestamp field entirely — timestamps are rarely needed

---

**Q77.** An interception hook blocks a `process_refund` call where the amount exceeds $500. What should the hook return to the agent?

- A) Nothing — just silently block the call
- B) A structured error with the reason for blocking and a redirect instruction (e.g., "Refund of $750 exceeds autonomous limit — escalate to human agent")
- C) A success response with the amount reduced to $500
- D) An unstructured error string: "blocked"

---

**Q78.** Why are hooks described as providing "deterministic guarantees" while prompt instructions only provide "probabilistic compliance"?

- A) Hooks run faster than prompt evaluation
- B) Hooks are code — they always execute with the same outcome for the same input. Prompt instructions rely on the LLM's judgment, which has a non-zero chance of being ignored or misinterpreted
- C) Hooks use a separate AI model that is more reliable
- D) Prompts are non-deterministic because they use random seeds

---

**Q79.** A workflow requires identity verification before any financial transaction. The system prompt says "always call get_customer before process_refund." The agent occasionally calls process_refund first. What is the ONLY reliable fix?

- A) Rewrite the system prompt with clearer language
- B) Add "IMPORTANT: ALWAYS verify customer first" to the system prompt
- C) Implement a prerequisite gate that programmatically checks whether get_customer has returned a verified customer ID before allowing process_refund to execute
- D) Upgrade to a more capable model that better follows instructions

---

**Q80.** A PreToolUse hook checks if a customer is on a "do not contact" list before allowing send_email to execute. This hook fires BEFORE the email tool runs. What is this an example of?

- A) PostToolUse data normalization
- B) Programmatic prerequisite enforcement — blocking a tool call based on a compliance check before it executes
- C) Session management
- D) Minimal footprint design

---

**Q81.** Which scenario is BEST handled by a PostToolUse hook rather than a PreToolUse hook?

- A) Blocking a refund that exceeds a financial limit
- B) Verifying customer identity before processing a transaction
- C) Normalizing a tool result that returns dates in 5 different formats from 5 different MCP servers into a single consistent ISO 8601 format
- D) Checking if a user has admin permissions before allowing a delete operation

---

**Q82.** A developer argues: "We don't need hooks — our system prompt clearly says never process refunds over $500 and Claude always follows it." What is the risk?

- A) No risk — if Claude always follows it, hooks are unnecessary overhead
- B) LLM compliance is probabilistic. In edge cases, unusual phrasings, long context, or adversarial inputs, Claude may not follow the instruction. For financial compliance, a non-zero failure rate is unacceptable — hooks eliminate this risk
- C) The risk is only theoretical — it has never happened in production
- D) The risk is performance — hooks are faster than prompts for compliance

---

**Q83.** A hook needs to block tool calls that would delete more than 100 records at once. Which hook type and which firing point is correct?

- A) PostToolUse — intercept after the deletion to log it
- B) PreToolUse — intercept before the deletion executes to check the record count and block if over 100
- C) Exit hook — check record counts at the end of the session
- D) No hook needed — use a prompt instruction instead

---

**Q84.** An agent calls `lookup_order` which returns numeric status codes (1=pending, 2=shipped, 3=delivered). Claude keeps misinterpreting these codes. Where should the fix be applied?

- A) Rewrite the system prompt to include a code legend
- B) PostToolUse hook — intercept the tool result and convert numeric codes to human-readable strings (1→"pending", 2→"shipped", 3→"delivered") before Claude processes them
- C) Rename the tool to include status descriptions
- D) Add few-shot examples of correct code interpretation to the prompt

---

**Q85.** A prerequisite gate checks that `get_customer` has run successfully before allowing `process_refund`. The developer also wants to require `lookup_order` to run first. How should the gate be updated?

- A) Add a second system prompt instruction specifying the order
- B) Update the prerequisite gate to check BOTH conditions: verified_customer_id must exist AND order_verified must be true before process_refund is allowed to execute
- C) Create two separate prerequisite gates that run independently
- D) Use a PostToolUse hook on lookup_order to set a flag

---

**Q86.** Hooks are invoked by ______, not by ______.

- A) Claude / the Agent SDK
- B) The Agent SDK / Claude
- C) The user / the system prompt
- D) MCP servers / the coordinator

---

**Q87.** An exit hook fires when the agent loop ends. What are valid use cases for an exit hook? (Choose the BEST answer)

- A) Blocking tool calls that violate policy
- B) Normalizing data formats from tool results
- C) Cleanup operations, logging final session state, sending completion notifications, or triggering downstream workflows after the agent finishes
- D) Verifying customer identity before financial operations

---

**Q88.** A developer wants to ensure that whenever the `web_search` tool returns results, any non-English content is automatically translated to English before Claude processes it. Which hook and timing is correct?

- A) PreToolUse — translate inputs before web_search runs
- B) PostToolUse — intercept web_search results after execution and before Claude sees them, translate non-English content, then return the translated result
- C) Exit hook — translate all results at the end of the session
- D) No hook needed — Claude can translate inline

---

## Section E — Session Management (Q89–Q96)

---

**Q89.** A developer is comparing two refactoring approaches for a legacy codebase. They want both approaches to start from the same codebase analysis without affecting each other. Which session operation is correct?

- A) Run approach 1 to completion, then start a new session for approach 2
- B) Use `fork_session` to create two independent branches from the shared baseline analysis — neither branch affects the other
- C) Use `--resume` twice with different task descriptions
- D) Run both approaches in the same session with clear section separators

---

**Q90.** After a Claude Code session analyzed a codebase, the team rewrote the authentication module entirely. The developer resumes the old session. What MUST they do before asking Claude to continue work on auth?

- A) Nothing — Claude will automatically detect the rewrite through its file monitoring
- B) Explicitly tell Claude that the authentication module has been completely rewritten and ask it to re-read the relevant files before proceeding
- C) Start a completely new session — rewrites always require fresh sessions
- D) Provide Claude with a diff of the changes

---

**Q91.** When is starting a new session with an injected summary BETTER than using `--resume`?

- A) When the session name is forgotten
- B) When prior tool results are stale — for example, when the codebase has changed significantly, external APIs have updated, or fundamental assumptions from the previous session no longer hold
- C) New sessions are always better — resume is rarely appropriate
- D) When the session is older than 24 hours

---

**Q92.** A team is exploring whether to use PostgreSQL or MongoDB for a new project. They have a shared baseline analysis of requirements. What is the IDEAL session management approach?

- A) Run PostgreSQL analysis first, then MongoDB analysis sequentially in the same session
- B) Use fork_session — branch one for PostgreSQL exploration, branch two for MongoDB exploration, both starting from the identical requirements analysis baseline
- C) Create two completely separate new sessions with manually duplicated context
- D) Use `--resume` and add a flag indicating which database to analyze

---

**Q93.** A developer resumes a session with `--resume project-analysis` and continues working. What does Claude retain from the previous session?

- A) Only the system prompt — conversation history is cleared on resume
- B) The full conversation history including all tool calls, results, and reasoning from the previous session
- C) Only the last 10 messages — older context is dropped on resume
- D) The file contents Claude read, but not the conversation history

---

**Q94.** Which scenario correctly requires a NEW session with injected summary rather than `--resume`?

- A) "I want to continue yesterday's codebase analysis — nothing has changed"
- B) "We switched from REST to GraphQL since the last session — all the API analysis from before is now outdated"
- C) "I want to add one more file to the analysis from last session"
- D) "I forgot to ask Claude to check the test files — resume and add them"

---

**Q95.** What is the key advantage of using named sessions with `--resume <session-name>` over unnamed sessions?

- A) Named sessions have higher token limits
- B) Named sessions can be resumed by name across work sessions, enabling persistent investigation threads that can be picked up at any time
- C) Named sessions are stored in the cloud while unnamed sessions are local only
- D) Named sessions have faster API response times

---

**Q96.** A developer uses `fork_session` to explore two approaches. Approach A modifies 15 files. Approach B modifies 10 different files. Do the changes in Approach A affect Approach B?

- A) Yes — fork_session shares file system changes between branches
- B) No — fork_session creates completely independent branches; changes in one branch do not affect the other
- C) Only if the same files are modified in both branches
- D) Yes — the coordinator synchronizes changes across all forks

---

## Section F — Error Handling, HITL & Minimal Footprint (Q97–Q114)

---

**Q97.** A subagent gets a 403 Forbidden error when calling a file system tool. How should this be classified and handled?

- A) Transient error — retry with exponential backoff
- B) Permission error — non-retryable. Report to the coordinator with error details. Retrying will not fix a permission issue
- C) Environment error — wait 60 seconds and retry
- D) Reasoning error — Claude chose the wrong tool

---

**Q98.** An agent repeatedly tries to fix a bug but makes no progress after 3 attempts. What should a well-designed system do?

- A) Continue trying indefinitely — persistence is key to autonomous systems
- B) Escalate to a human after a defined retry threshold — repeated failure is a mandatory HITL escalation trigger
- C) Try a completely different approach automatically
- D) Report success to the user anyway

---

**Q99.** A customer says "I've been dealing with this billing issue for 3 weeks and I'm very frustrated." Should the agent escalate?

- A) No — frustration is not a technical trigger for escalation
- B) Yes — while frustration alone is not always a trigger, 3 weeks of unresolved issues combined with expressed frustration is a signal the agent may not be able to resolve this autonomously. Escalation should be considered
- C) Only escalate if the customer explicitly asks for a human
- D) Escalate only if the billing amount exceeds the threshold

---

**Q100.** An agent is about to permanently delete 50,000 customer records as part of a data cleanup task. What should happen before the deletion executes?

- A) Proceed — the task was assigned by an authorized user
- B) The agent should pause and require explicit human confirmation before executing any irreversible action of this magnitude
- C) Create a backup first, then proceed with deletion autonomously
- D) Check if the record count exceeds the system prompt limit before deleting

---

**Q101.** What is a "reasoning error" and how is it different from a "tool error"?

- A) They are the same — all errors in agentic systems are tool errors
- B) A reasoning error is when Claude makes a wrong decision (wrong tool chosen, misunderstood task, incorrect inference). A tool error is when a tool call fails technically (timeout, invalid input, permission denied)
- C) Reasoning errors only occur in multi-agent systems; tool errors occur in single-agent systems
- D) Tool errors are recoverable; reasoning errors always require human intervention

---

**Q102.** An agent is given tools: `read_file`, `write_file`, `delete_file`, `execute_bash`, `send_email`, `query_database`, `admin_database`, `create_user`, `delete_user`. Its actual task is only to read log files and summarize them. What is wrong?

- A) Nothing — extra tools provide flexibility
- B) This violates minimal footprint — the agent only needs `read_file`. All other tools increase blast radius if something goes wrong and should be removed
- C) The agent has too many tools causing selection confusion — reduce to 5 maximum
- D) `admin_database` should be replaced with `query_database`

---

**Q103.** A legal team asks for an agent to review contracts. The task is clearly defined. The developer gives the agent broad file system access "in case it needs to reference other documents." What principle does this violate?

- A) No principle — defensive access is good practice
- B) Minimal footprint — grant only the specific access needed for the defined task. If the agent needs to reference other documents, those specific documents should be explicitly identified and scoped
- C) HITL escalation — legal tasks always require human oversight
- D) Session management — legal tasks should always use fresh sessions

---

**Q104.** A transient database timeout occurs during an agent's tool call. What is the correct retry sequence?

- A) Retry immediately 10 times, then give up
- B) Retry with exponential backoff: wait 1 second, retry; if fails wait 2 seconds, retry; if fails wait 4 seconds, retry — with a maximum retry limit
- C) Report permanent failure to the user immediately — database issues are never transient
- D) Restart the entire agentic loop from the beginning

---

**Q105.** A customer support agent successfully resolves 85% of issues autonomously. For the remaining 15%, it escalates to humans. Is this a well-designed system?

- A) No — a well-designed agent should resolve 100% of issues autonomously
- B) Yes — defining clear escalation thresholds and escalating when appropriate is correct design. 80%+ first-contact resolution with proper escalation is the target
- C) No — the escalation rate is too high. Reduce HITL triggers to achieve 95% autonomous resolution
- D) Only if the escalation adds latency of less than 5 minutes

---

**Q106.** What should a well-designed escalation handoff include when passing a customer issue to a human agent?

- A) Just the customer's name and issue type
- B) Customer ID, account details, root cause analysis, what was already attempted, refund/action amount, reason for escalation, and recommended next action — everything the human needs to act immediately
- C) The full raw conversation transcript copy-pasted
- D) Only the error code that triggered the escalation

---

**Q107.** An agent is configured with `allowed_tools=["Read", "Write", "Bash", "Admin"]` for a task that only requires reading configuration files. The agent accidentally deletes a production config file using the Write tool. What design failure caused this?

- A) The model was not smart enough to avoid the deletion
- B) Minimal footprint was violated — the agent was given Write and Admin access it didn't need. Read-only access would have prevented the accidental deletion entirely
- C) The system prompt didn't say "don't delete files"
- D) The agent should have used fork_session to avoid affecting production

---

**Q108.** An agent's task is to "improve the codebase." Before starting, it should do what?

- A) Immediately begin reading all files to understand the scope
- B) Ask clarifying questions: which files are in scope, what does "improve" mean (formatting? refactoring? performance?), are there files to avoid, what is the definition of done
- C) Create a plan based on reasonable assumptions and execute
- D) Fork the session to safely experiment without affecting the main codebase

---

**Q109.** A customer explicitly says "please just fix this automatically, I trust you completely." Should the agent disable all HITL escalation triggers?

- A) Yes — the customer has explicitly consented to full autonomous operation
- B) No — HITL triggers exist for compliance, safety, and irreversible action protection. Customer preference cannot override escalation for actions above defined financial thresholds or irreversible operations
- C) Only disable escalation for this specific session
- D) Reduce escalation triggers by 50% as a compromise

---

**Q110.** An agent encounters an error it cannot classify as transient, permanent, or environment-related. What is the correct action?

- A) Retry three times and then give up
- B) Treat it as a permanent error and report to the user
- C) Escalate to a human — unclassifiable errors with unknown recovery paths are appropriate HITL triggers
- D) Log the error silently and continue

---

**Q111.** Which of the following is NOT a valid HITL escalation trigger?

- A) Customer explicitly requests a human agent
- B) Refund amount exceeds the autonomous processing limit
- C) The agent's response takes longer than 3 seconds
- D) The agent is about to execute an irreversible action

---

**Q112.** An agent is designed to "never escalate — always resolve autonomously." A customer threatens legal action. What is the design flaw?

- A) No flaw — autonomous resolution is always preferable
- B) Legal threats are a sensitive situation requiring human judgment. A system that never escalates will handle this poorly, potentially creating legal liability. Escalation triggers must include sensitive situations regardless of the "always autonomous" mandate
- C) The agent should ask for legal advice from an external API
- D) Legal situations should be redirected to a FAQ page

---

**Q113.** Error propagation in a multi-agent system: Subagent A fails. The coordinator catches the error. What information should the coordinator propagate upward in its error report?

- A) Just the error code
- B) What the subagent was trying to do, what error occurred, what was successfully completed before the failure, whether retry was attempted, and what partial results are available
- C) The full conversation history of the failed subagent
- D) Nothing — errors should be silently suppressed to avoid alarming users

---

**Q114.** An agent is given the task "send a promotional email to all 500,000 users." Before executing, what should a well-designed system require?

- A) Proceed immediately — the task was authorized
- B) Require explicit human confirmation before sending — mass emails are irreversible actions with large blast radius that must always require HITL approval
- C) Send to 100 users first as a test, then proceed with the rest autonomously
- D) Check if the email content exceeds 1,000 characters before sending

---

## Section G — Dynamic Planning, Ambiguity & Mixed Scenarios (Q115–Q126)

---

**Q115.** An agent begins a task and discovers mid-execution that the scope is significantly larger than anticipated. What should a well-designed agentic system do?

- A) Complete only the original anticipated scope and stop
- B) Dynamically replan — update the task plan based on the new discovery, potentially spawning additional subagents or re-scoping the work
- C) Restart from scratch with the new scope understanding
- D) Ask the user whether to continue every time the scope expands

---

**Q116.** A user asks an agent: "clean up the test files." The agent has no additional context. Before starting a potentially hour-long cleanup, what MUST the agent do?

- A) Start immediately — "clean up test files" is self-explanatory
- B) Clarify scope: which directories contain test files, what does "clean up" mean (delete failing tests? reformat? rename?), are there any files to preserve, what is the target state
- C) Read all test files first, then ask if the proposed plan is correct
- D) Fork the session and try cleaning up a small subset first

---

**Q117.** Dynamic adaptive decomposition is BEST described as:

- A) Running all possible subtasks in parallel from the start
- B) Breaking a task into fixed sequential steps known upfront
- C) Generating subtasks adaptively based on what is discovered at each step — the full task plan is unknown until exploration begins
- D) Delegating the entire task to a single specialized subagent

---

**Q118.** An agent is asked to "add comprehensive test coverage to a legacy codebase." Using dynamic adaptive decomposition, what should the agent do FIRST?

- A) Immediately start writing tests for every file found
- B) Map the codebase structure first — identify modules, dependencies, existing coverage, and high-impact areas — then create a prioritized test plan that adapts as dependencies are discovered
- C) Ask the user for a list of files to test
- D) Spawn 10 subagents immediately to test different files in parallel

---

**Q119.** A task is described as: "analyze customer feedback from last month." The agent needs to decide whether to use prompt chaining or dynamic adaptive decomposition. Which and why?

- A) Prompt chaining — the steps are unknown until exploration
- B) Dynamic adaptive — the volume and content of feedback is unknown until the agent starts reading it, requiring an adaptive approach to determine how many categories exist and what analysis is needed
- C) Prompt chaining — always use fixed steps for data analysis tasks
- D) Neither — this is a workflow, not an agentic task

---

**Q120.** Prompt chaining is MOST appropriate when:

- A) The task scope is unknown and must be explored first
- B) Steps are well-defined and predictable upfront, each building on the previous — for example, extract data → validate → format → store
- C) Subagents need to run in parallel for maximum speed
- D) The coordinator needs to adapt based on intermediate findings

---

**Q121.** An agent is halfway through a 2-hour autonomous task when it discovers the underlying data it was given is corrupted. What should it do?

- A) Complete the task with the corrupted data and note the issue in the output
- B) Pause, report the discovery to the user with what has been completed so far, and await guidance before continuing — mid-task discovery of fundamental data issues is an appropriate pause point
- C) Continue — the agent should be resilient to data quality issues
- D) Restart the entire task with fresh data automatically

---

**Q122.** A developer is designing an agent for a customer support system. The agent's task is ambiguous: "handle customer issues." Before deploying, what should the developer ensure?

- A) Deploy as-is — the agent will figure out edge cases over time
- B) Define clear scope boundaries: what types of issues the agent handles autonomously, what triggers escalation, what tools are available, and what the escalation handoff protocol looks like
- C) Give the agent maximum tool access so it can handle anything
- D) Ensure the agent never escalates so it appears fully capable

---

**Q123.** An agent is replanning mid-task. It was researching Company A and discovered Company A was acquired by Company B. The original task only mentioned Company A. What is the correct behavior?

- A) Ignore Company B — it was not in the original task
- B) Dynamically replan to include Company B in the research scope — the acquisition is directly relevant to understanding Company A
- C) Stop the task and ask the user if they want to include Company B
- D) Complete the Company A research and submit a separate request for Company B

---

**Q124.** Which scenario is an example of correct minimal clarification before a long autonomous run?

- A) Asking 20 clarifying questions covering every possible edge case before starting
- B) Starting immediately to avoid delays, then asking questions if problems arise
- C) Asking only the 2-3 clarifying questions where a wrong assumption would cause significant wasted work — not every possible question
- D) Asking one clarifying question per tool call throughout execution

---

**Q125.** A task says "review the codebase for security issues." The agent has read access to 50,000 files. Using dynamic adaptive decomposition, what is the correct FIRST step?

- A) Read all 50,000 files immediately to get the full picture
- B) Map the codebase structure first — identify entry points, authentication code, data handling, external integrations — then prioritize high-risk areas for deep analysis
- C) Spawn 50 subagents to each read 1,000 files in parallel
- D) Search for known vulnerability patterns across all files simultaneously

---

**Q126.** FINAL CHALLENGE — A customer support agent receives: "I want a refund of $1,500 for my order #12345 because it arrived damaged. I've been waiting 3 weeks. I want to speak to a manager." Identify ALL escalation triggers present in this message.

- A) Only the refund amount — $1,500 exceeds the autonomous limit
- B) Refund amount exceeds autonomous limit ($1,500 > $500 threshold) AND explicit request for a human ("I want to speak to a manager") AND repeated unresolved issue (3 weeks waiting) — all three are present and any one alone would be sufficient to trigger escalation
- C) Only the explicit human request — amount and time don't matter
- D) None — the agent should attempt autonomous resolution first before escalating

---

## Extended Answer Key (Q27–Q126)

| Q | Answer | Key Reason |
|---|---|---|
| 27 | **C** | max_tokens = truncated output — handle gracefully, inform user |
| 28 | **B** | When both text and tool_use present — execute the tool, continue loop |
| 29 | **B** | Iteration caps are fine as secondary safety nets — never the primary stop mechanism |
| 30 | **C** | Tool result goes as user-role message after assistant's tool_use message |
| 31 | **B** | Execute both tools, append both results, send in one API call |
| 32 | **B** | Model-driven = Claude decides dynamically; decision tree = hardcoded sequence |
| 33 | **B** | stop_sequence fires when a configured stop trigger string is hit |
| 34 | **B** | Stripping history loses all context — Claude cannot reason about what it already did |
| 35 | **B** | Timeout = transient error — retry with exponential backoff |
| 36 | **C** | stop_reason "end_turn" is the only reliable completion signal |
| 37 | **B** | Summarize completed work, inject summary into new session, continue from there |
| 38 | **B** | "ERROR" may appear in legitimate content — text checks cause false exits |
| 39 | **B** | max_tokens = output truncated — must be handled explicitly, never ignored |
| 40 | **B** | Tool result not appended to history — Claude has no memory of calling it |
| 41 | **B** | User message with tool result gives Claude the output to reason about next action |
| 42 | **B** | External scheduler triggers fresh loops — infinite internal loops are anti-pattern |
| 43 | **B** | Append empty result, let Claude reason — empty ≠ error |
| 44 | **B** | API validation error — tool_result must match a preceding tool_use block |
| 45 | **B** | Claude misunderstood the scope or system prompt was ambiguous about full task list |
| 46 | **B** | Dynamic subagent selection — coordinator analyzes needs before routing |
| 47 | **C** | All inter-subagent info routes through coordinator — A never contacts B directly |
| 48 | **B** | Pipeline = sequential data transformation with no coordination decisions |
| 49 | **B** | Coordinator not dynamically selecting — blindly routing through full pipeline |
| 50 | **C** | Partition scope BEFORE delegation — not deduplicate after |
| 51 | **B** | Central routing = full coordinator visibility, consistent error handling |
| 52 | **B** | Task decomposition too broad — coordinator must break into specific bounded subtasks |
| 53 | **B** | Independent verification — second agent has no reasoning bias from first |
| 54 | **C** | Continue independent subagent, include results, report failure separately |
| 55 | **B** | Gaps between narrowly defined subtasks leave important areas unresearched |
| 56 | **B** | Include partial results with annotation — partial data is better than none |
| 57 | **B** | Peer-to-peer for negotiation/debate as equals — rarely used |
| 58 | **C** | Sequential transform through agents = pipeline pattern |
| 59 | **B** | Unnecessary tokens, latency, and conflicting results from irrelevant subagents |
| 60 | **B** | Central error routing = visibility, consistency, prevents silent failures |
| 61 | **B** | Selection logic is inverted — complex queries need MORE subagents, not fewer |
| 62 | **B** | Evaluate gaps, targeted re-queries, re-delegate, re-invoke synthesis |
| 63 | **B** | Remove Write — minimal footprint, coordinator has no need for file writing |
| 64 | **B** | Description is used by coordinator for subagent selection — vague = wrong selection |
| 65 | **B** | Structured objects with source metadata preserve attribution for citations |
| 66 | **B** | Goals enable adaptability when steps fail — procedural = brittle |
| 67 | **B** | Fix in AgentDefinition — restrict allowed_tools and tighten system_prompt |
| 68 | **B** | Emit all 4 Task calls in one response = parallel execution, ~4x faster |
| 69 | **A** | Vague system_prompt gives no role clarity; overly broad tools violate minimal footprint |
| 70 | **B** | Content + source URL + source title + retrieval timestamp = proper citation |
| 71 | **B** | Separate turns = sequential, not parallel — emit all Tasks in ONE response |
| 72 | **B** | Missing: prior findings, research scope, quality criteria, constraints |
| 73 | **B** | fork_session = independent branches from shared baseline |
| 74 | **C** | fork_session — both branches start from identical baseline, explore independently |
| 75 | **B** | Plain text loses attribution — structured JSON preserves source for citations |
| 76 | **B** | Convert Unix timestamp to ISO 8601 in PostToolUse before Claude sees it |
| 77 | **B** | Return structured error with reason and redirect instruction — not silent block |
| 78 | **B** | Code is deterministic; LLM judgment has non-zero failure rate |
| 79 | **C** | Only a prerequisite gate provides 100% reliable ordering enforcement |
| 80 | **B** | PreToolUse compliance check blocking tool based on business rule |
| 81 | **C** | PostToolUse = normalize data AFTER tool runs, BEFORE Claude sees it |
| 82 | **B** | LLM compliance is probabilistic — for financial compliance, non-zero failure = unacceptable |
| 83 | **B** | PreToolUse — block BEFORE deletion executes, not after |
| 84 | **B** | PostToolUse hook converts numeric codes to strings before Claude processes them |
| 85 | **B** | Update gate to check both conditions: verified_customer_id AND order_verified |
| 86 | **B** | Agent SDK invokes hooks, not Claude |
| 87 | **C** | Exit hooks for cleanup, logging, notifications, downstream triggers |
| 88 | **B** | PostToolUse — intercept after web_search, translate, return to Claude |
| 89 | **B** | fork_session creates independent branches from same baseline |
| 90 | **B** | Explicitly tell Claude which files changed and ask it to re-read them |
| 91 | **B** | New session when tool results are stale — fundamentals have changed |
| 92 | **B** | fork_session — one branch per database option from shared baseline |
| 93 | **B** | Full conversation history retained including tool calls and reasoning |
| 94 | **B** | API architecture changed — all prior API analysis is now outdated/stale |
| 95 | **B** | Named sessions enable resumption across work sessions by name |
| 96 | **B** | fork_session branches are fully independent — no cross-branch contamination |
| 97 | **B** | 403 = permission error — non-retryable, report to coordinator |
| 98 | **B** | Escalate after defined retry threshold — repeated failure = mandatory HITL trigger |
| 99 | **B** | 3 weeks unresolved + expressed frustration = strong escalation signal |
| 100 | **B** | Irreversible action at scale requires explicit human confirmation before executing |
| 101 | **B** | Reasoning error = wrong decision by Claude; tool error = technical call failure |
| 102 | **B** | Minimal footprint violated — agent only needs Read, all others should be removed |
| 103 | **B** | Minimal footprint — only explicitly needed access should be granted |
| 104 | **B** | Exponential backoff with max retry limit for transient errors |
| 105 | **B** | 80%+ autonomous resolution with proper escalation is correct target design |
| 106 | **B** | Full structured handoff: ID, details, root cause, attempted, amount, recommendation |
| 107 | **B** | Minimal footprint violated — Read-only would have prevented the accidental deletion |
| 108 | **B** | Clarify scope before starting — "improve" is too vague for autonomous action |
| 109 | **B** | Customer preference cannot override compliance thresholds and irreversibility gates |
| 110 | **C** | Unclassifiable errors with unknown recovery = escalate to human |
| 111 | **C** | Response latency is not a HITL trigger — the others are all valid triggers |
| 112 | **B** | Legal threats are sensitive situations requiring human judgment — always escalate |
| 113 | **B** | Full error report: what was attempted, error, completed work, retry status, partials |
| 114 | **B** | Mass irreversible action requires human confirmation — never autonomous |
| 115 | **B** | Dynamically replan based on new discovery — agentic systems adapt |
| 116 | **B** | Clarify scope before starting — ambiguous task + long run = must clarify first |
| 117 | **C** | Dynamic adaptive = subtasks generated from discoveries, plan unknown until started |
| 118 | **B** | Map structure first, identify high-impact areas, then create adaptive plan |
| 119 | **B** | Volume and categories unknown until exploration = dynamic adaptive |
| 120 | **B** | Prompt chaining = well-defined predictable steps known upfront |
| 121 | **B** | Pause, report discovery with completed work, await guidance |
| 122 | **B** | Define scope, escalation triggers, tools, and handoff protocol before deploying |
| 123 | **B** | Replan to include Company B — acquisition is directly relevant |
| 124 | **C** | Ask only questions where wrong assumptions cause significant wasted work |
| 125 | **B** | Map structure first, identify high-risk areas, then prioritize for deep analysis |
| 126 | **B** | All three triggers present: amount > threshold + explicit human request + 3 weeks unresolved |