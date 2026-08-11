# CCA-F Exam Prep — Domain 5: Context Management & Reliability
> **Exam Weight: 15% — ~9 Questions** The lightest domain by weight, but consistently the most common source of careless wrong answers. The exam tests structural fixes — not language tweaks — for context degradation, escalation, error propagation, and information provenance.

---

## Credits & Resources

**Repository owner:** Muhammed Nemathullah Khan

**Author of this domain guide:** [Arun Varadharajalu](https://www.linkedin.com/in/arunv11u/)

**Domain Resources:**
- **Core topics & fundamentals overview:** [CCA-F Study Guide — guide_en.md](https://github.com/paullarionov/claude-certified-architect/blob/main/guide_en.md)
- **Deep dive in each domain:** [Domain 1](./domain-1.md) · [Domain 2](./domain-2.md) · [Domain 3](./domain-3.md) · [Domain 4](./domain-4.md) · [Domain 5](./domain-5.md)
- **Scenarios:** [All 6 exam scenarios](../scenarios/listedScenarios.md)
- **Mock test and sources:** [Mock test bank](./mock-test/) · [Claude Certification Guide — Mock Exam](https://claudecertificationguide.com/mock-exam) · [CyberSkill Practice — CCAF](https://practice.cyberskill.world/practice/ccaf/exam) · [CertSafari — CCAF](https://www.certsafari.com/anthropic/claude-certified-architect-foundations)

---

## Table of Contents

1. [What This Domain Tests](#1-what-this-domain-tests)
2. [The Master Mental Model — Structural vs Linguistic Fixes](#2-the-master-mental-model--structural-vs-linguistic-fixes)
3. [Context Window Fundamentals](#3-context-window-fundamentals)
3b. [The CALM Framework — High-Priority D5 Named Concept](#3b-the-calm-framework--high-priority-d5-named-concept) ⭐ NEW
4. [Progressive Summarization — Risks & Correct Patterns (T5.1)](#4-progressive-summarization--risks--correct-patterns-t51)
5. [The Case Facts Block (D5.2 — Tier A)](#5-the-case-facts-block-d52--tier-a)
6. [Lost-in-the-Middle Effect](#6-lost-in-the-middle-effect)
7. [Context Degradation Management — Scratchpad, /compact, Windowing (T5.1)](#7-context-degradation-management--scratchpad-compact-windowing-t51)
7b. [Conversation Compaction — The Standard CALM Strategy](#7b-conversation-compaction--the-standard-calm-strategy) ⭐ NEW
8. [Error Propagation in Multi-Agent Systems (T5.2)](#8-error-propagation-in-multi-agent-systems-t52)
9. [Escalation Design (T5.3)](#9-escalation-design-t53)
10. [Confidence Scoring & Human Review Routing (T5.5)](#10-confidence-scoring--human-review-routing-t55)
11. [Information Provenance & Source Attribution (T5.6)](#11-information-provenance--source-attribution-t56)
12. [Conflicting Source Data — Annotation, Never Arbitrary Resolution](#12-conflicting-source-data--annotation-never-arbitrary-resolution)
13. [Human Review Workflows — Stratified Sampling & Calibration (T5.5)](#13-human-review-workflows--stratified-sampling--calibration-t55)
14. [Crash Recovery — Structured State Manifests](#14-crash-recovery--structured-state-manifests)
15. [Token Budget Management (T5.4)](#15-token-budget-management-t54)
15b. [RAG Architecture in D5 Context](#15b-rag-architecture-in-d5-context) ⭐ NEW
15c. [Multi-Turn Conversation Design](#15c-multi-turn-conversation-design) ⭐ NEW
15d. [Prompt Caching in D5 (cache_control Breakpoints)](#15d-prompt-caching-in-d5-cachecontrol-breakpoints) ⭐ NEW
16. [Domain 5 in the Exam Scenarios](#16-domain-5-in-the-exam-scenarios)
17. [Anti-Patterns Master List](#17-anti-patterns-master-list)
18. [Key Rules to Memorize](#18-key-rules-to-memorize)
19. [Practice Questions (20 MCQs)](#19-practice-questions-20-mcqs)
20. [Answer Key & Explanations](#20-answer-key--explanations)

---

## 1. What This Domain Tests

| Task Statement | Description |
|---|---|
| T5.1 | Manage context windows across long interactions — progressive summarization, compaction, case facts blocks, and scratchpad persistence |
| T5.2 | Design error propagation patterns in multi-agent systems — structured error context, partial results, coordinator recovery |
| T5.3 | Design escalation workflows — reliable triggers, handoff summaries, HITL patterns |
| T5.4 | Optimize token usage — trim verbose tool outputs, efficient context design |
| T5.5 | Implement human review workflows — confidence calibration, stratified sampling, field-level routing |
| T5.6 | Maintain information provenance — claim-source mapping, temporal data, conflict annotation |

### Domain 5 in the Exam Scenarios

| Scenario | Domain 5 Focus |
|---|---|
| **S1 — Customer Support** | Escalation triggers, case facts block, error propagation, structured handoff summaries |
| **S3 — Multi-Agent Research** | Source attribution, conflict annotation, context degradation, crash recovery |
| **S6 — Structured Data Extraction** | Human review routing, confidence calibration, stratified sampling, temporal provenance |
| **S2 — Code Generation** | Context degradation in long sessions, scratchpad persistence, `/compact` |

**Study priority:** S1 and S3 are the heaviest for Domain 5. The customer support and research scenarios together cover ~80% of D5 exam questions.

---

## 2. The Master Mental Model — Structural vs Linguistic Fixes

Domain 5 follows the same meta-pattern as all other domains:

| Problem | ❌ Linguistic Fix (WRONG) | ✅ Structural Fix (CORRECT) |
|---|---|---|
| Agent forgets key facts in long sessions | "Remember the customer ID throughout" | CASE_FACTS block at top of every prompt |
| Critical context buried mid-prompt | Bold or capitalize it | Move to system prompt (position 0) or top of user message |
| Numeric values distorted by summarization | "Don't lose important numbers" | Structured fact block outside summarized prose |
| Escalation based on gut feel | "Escalate complex cases" | Explicit threshold triggers (amount, retry count, explicit request) |
| Agent picks wrong customer when names match | "Be careful to use the right customer" | Request additional identifiers — never guess |
| Context window fills up | "Be more concise" | Progressive summarization + context windowing |
| Tool output bloats context | "Only use what you need" | PostToolUse hook trimming verbose results |
| Conflicting sources | "Use the most reliable source" | Annotate both values with attribution — never silently pick one |

**The Domain 5 decision shortcut:**
> If a fix requires Claude to *choose* to do the right thing → it's probabilistic (wrong).
> If a fix structurally prevents the wrong thing → it's deterministic (correct).

---

## 3. Context Window Fundamentals

### What Accumulates

In an agentic loop, every turn adds to the context window:
```
Turn 1: [system] + [user message] + [assistant response + tool_use] + [tool_result]
Turn 2: Turn 1 + [assistant] + [tool_result]
Turn 3: Turn 2 + [assistant] + [tool_result]
...
Token count grows O(n²) with number of turns.
```

### Three Categories of Context Content

| Category | Examples | Strategy |
|---|---|---|
| **Immutable** (never changes) | Policies, role, constraints, customer ID | Pin to system prompt or CASE_FACTS block |
| **Semi-stable** (changes rarely) | Prior work summary, key findings so far | Progressive summarization when context nears limit |
| **Ephemeral** (changes every turn) | Tool results, current question, recent messages | Trim verbose results; don't over-retain |

### The Context Nearing Limit — When to Act

Act when context reaches **70-80% of the window limit**, not 100%:
- At 100%: the API starts truncating from the middle (worst position)
- At 70-80%: you have room to summarize and restructure cleanly

---

## 3b. The CALM Framework — High-Priority D5 Named Concept
> ⭐ **Explicitly listed in the official D5 domain description: "CALM framework, prompt caching with cache_control breakpoints, conversation compaction." The claudecertifiedarchitects.com guide names it a high-priority D5 topic.**

### What CALM Is

CALM stands for **Context-Aware LLM Management** — Anthropic's named framework for managing context windows effectively in production Claude systems. It is not a specific API feature; it is a strategic framework for thinking about context decisions.

The CALM exam question (from claudecertifiedarchitects.com): *"An agent is approaching the context limit. What is the standard CALM strategy?"* → **Conversation compaction** (summarising completed steps into a dense state block and continuing).

### The Five CALM Principles

| Principle | What It Means | Exam Application |
|---|---|---|
| **Curate** | Include only high-signal tokens — remove noise | Trim verbose tool outputs; don't retain ephemeral results |
| **Anchor** | Pin critical facts where attention is highest | CASE_FACTS block at top; constraints in system prompt |
| **Layer** | Separate immutable facts from compactable narrative | CASE_FACTS (never compress) + prose summary (compactable) |
| **Manage** | Act proactively at 70-80% — not reactively at 100% | Window at 70%; /compact when needed; scratchpad for long tasks |
| **Minimise** | Use the smallest context that preserves coherence | Delegate to subagents; trim tool outputs; progressive disclosure |

### CALM's Primary Strategy: Conversation Compaction

The single most tested CALM strategy is **conversation compaction** — when context approaches the limit, summarize completed work into a dense state block and continue from there:

```
Standard CALM pattern when context reaches limit:
1. Identify what has been completed (don't lose this)
2. Compact completed turns into dense state block
3. Keep CASE_FACTS block intact (never compress)
4. Continue with pending work from the fresh starting point
→ NOT: clear everything and restart (loses accumulated state)
→ NOT: return an error (task can continue with compaction)
→ NOT: increase max_tokens (controls output, not input window)
```

**The canonical exam question and answer:**
> *"An agent approaching the context window limit still needs to complete 3 more steps. What does the CALM framework recommend?"*
> **Answer: Conversation compaction** — summarise completed steps into a dense state block and continue execution. Clearing history discards accumulated state. Returning an error is premature. `max_tokens` controls output length, not input context size.

### CALM vs. Non-CALM Anti-Patterns

| Situation | ❌ Non-CALM (wrong) | ✅ CALM (correct) |
|---|---|---|
| Context at 100% | Crash or truncate | Acted at 70-80% before truncation |
| Approaching limit | Clear entire history | Compact completed turns; keep CASE_FACTS |
| Verbose tool output | Leave all 40 fields | Trim to 5 relevant fields via PostToolUse hook |
| Long session (hours) | Keep everything in memory | Scratchpad files for persistence; fresh sessions on resume |
| Main agent context full | Keep piling on | Delegate to subagent with fresh context window |
| Critical facts | Buried in history | Anchored at top (CASE_FACTS) or in system prompt |

---

## 4. Progressive Summarization — Risks & Correct Patterns (T5.1)

> ⭐ **The #1 most tested D5 pattern. The exam presents a scenario where summarization loses critical data and asks what the correct fix is.**

### The Core Problem

When you summarize conversation history to save tokens, **prose summarization destroys precision**:

```
Original: "Customer ID: CUST-48291, Order: ORD-9821, Refund: $847.00, Date: 2026-04-12"

After prose summarization:
"A customer requested a refund for an order from April."
→ Customer ID: GONE
→ Order ID: GONE
→ Exact amount: GONE ("approximately $800" is not $847.00)
→ Exact date: GONE
```

### What Gets Lost in Summarization

```
❌ Always lost/distorted in prose summarization:
- Exact dollar amounts → become "approximately", "about", "around"
- Specific dates → become "recently", "earlier", "in April"
- IDs and reference numbers → disappear entirely
- Percentages → become "high" or "low"
- Customer-stated expectations → become vague sentiment
- Ordered sequences → order may be lost
```

### The Correct Pattern — Two-Layer Approach

```python
# ✅ CORRECT — separate structured facts from prose summary

# Layer 1: Structured fact block (NEVER summarized — always exact)
CASE_FACTS = {
    "customer_id": "CUST-48291",
    "order_id": "ORD-9821",
    "refund_amount": 847.00,
    "issue_date": "2026-04-12",
    "account_tier": "premium",
    "retry_count": 2
}

# Layer 2: Prose summary (for narrative context — imprecision is acceptable here)
PROSE_SUMMARY = """
Customer contacted support about a damaged item received in April.
Item was photographed and photos were submitted. Two prior resolution
attempts were unsuccessful. Customer has been cooperative throughout.
"""

# In every prompt: CASE_FACTS injected first, prose summary second
user_message = f"""
CASE_FACTS (authoritative — use exact values):
{json.dumps(CASE_FACTS, indent=2)}

CONVERSATION_SUMMARY:
{PROSE_SUMMARY}

Current question: {current_question}
"""
```

**Why two layers?**
- Structured facts are exact and injected fresh every turn — they never get compressed
- Prose summary handles narrative context where approximate language is acceptable
- Keeping them separate means a summarization operation can never corrupt the numbers

### The Exam Trap

```
Scenario: "After 15 turns, the agent quotes the refund as 'approximately $800'
           when the customer specified $847. What went wrong?"

Wrong answers:
- "The agent made a reasoning error"
- "The temperature was too high"
- "The context window was exceeded"

Correct answer:
"The exact refund amount was stored only in conversation history prose,
where it was compressed during summarization. Extract transactional
facts into a persistent structured CASE_FACTS block injected at every
prompt, outside the summarized history."
```

---

## 5. The Case Facts Block (D5.2 — Tier A)

> ⭐ **Explicitly listed as a D5 concept with full Tier A status on claudearchitectcertification.com. The case-facts block is the canonical fix for numeric precision loss in long sessions.**

### Definition

A case-facts block is a **structured JSON object** containing all transactional, numerical, and reference data that must remain precise throughout a session. It is:
- Injected at the **top** of every user message (high-attention zone)
- **Never passed through summarization** — always reconstructed from the source record
- The single source of truth for all exact values

```json
{
  "CASE_FACTS": {
    "customer_id": "CUST-48291",
    "customer_email": "customer@example.com",
    "account_tier": "premium",
    "order_id": "ORD-9821",
    "order_total": 847.00,
    "refund_requested": 847.00,
    "autonomous_refund_limit": 500.00,
    "issue_date": "2026-04-12",
    "retry_count": 2,
    "escalation_threshold": 3,
    "session_id": "SESSION-7721"
  }
}
```

### What Goes in the Case Facts Block

| ✅ Always include | ❌ Never include |
|---|---|
| Exact dollar amounts | Narrative descriptions |
| Specific dates (ISO format) | Sentiment or tone |
| All IDs (customer, order, session) | Inferred information |
| Account status and tier | Summarized conversation |
| Retry and escalation counts | Vague qualifiers |
| Policy thresholds (e.g., $500 limit) | Changing context |

### The Immutable Fact Block Pattern

The case facts block is sometimes called an **immutable fact block** because its values never change mid-session — they are always injected fresh from the source system, never from memory:

```python
def build_user_message(current_question: str, session: Session) -> str:
    # ALWAYS fetch from source — never from the agent's prior response
    case_facts = fetch_fresh_case_facts(session.customer_id, session.order_id)
    prose_summary = session.get_compressed_history()
    
    return f"""
CASE_FACTS (authoritative — always use exact values from this block):
{json.dumps(case_facts, indent=2)}

PRIOR_CONTEXT:
{prose_summary}

CURRENT_QUESTION:
{current_question}
"""
```

**The exam rule:** CASE_FACTS values always come from the source system — never from the agent's conversation history. If the agent's history says "$800" but the source system says "$847.00", the CASE_FACTS block always wins.

---

## 6. Lost-in-the-Middle Effect

This is a cross-domain concept (also covered in D4 Attention Engineering) but D5 tests specific production scenarios where it causes reliability failures.

### The Problem

Models have a U-shaped attention curve — high at the start and end, low in the middle (40-80% of context). **This is not a bug; it is a property of transformer architectures.** (Documented: Liu et al. 2023 — accuracy drops from 90%+ at position 1 to 40-50% at position 50.)

### D5-Specific Manifestations

| Scenario | What Breaks | Fix |
|---|---|---|
| 20 subagent findings aggregated | Middle findings missed in synthesis | Place key findings at top + section headers |
| Research report from 50 documents | Documents 20-35 underrepresented | Chunked processing per document + integration pass |
| Long customer support conversation | Policy from early turn forgotten by turn 10 | CASE_FACTS block at top of every message |
| Code review across 50 files | Middle files missed | Per-file local passes + cross-file integration pass |

### The Fix — Position-Based, Not Linguistic

```python
# ❌ WRONG — linguistic fix (no effect on attention weights)
aggregated = """
**IMPORTANT: The following findings from document 23 are critical:**
Finding from doc 23 is hidden here in the middle...
"""

# ✅ CORRECT — structural fix: put critical findings at top
aggregated = """
KEY_FINDINGS_SUMMARY (from all sources — read first):
- Doc 23: Revenue discrepancy of $2.3M identified
- Doc 08: CEO resignation confirmed
- Doc 41: Regulatory filing deadline missed

DETAILED_FINDINGS_BY_DOCUMENT:
[Document 1]...
[Document 23] (see KEY_FINDINGS_SUMMARY above)...
"""
```

**The exam rule:** Bold/caps/emphasis has zero effect on transformer attention weights. Position is what matters. Place critical findings at the beginning (or end) of aggregated inputs.

---

## 7. Context Degradation Management — Scratchpad, /compact, Windowing (T5.1)

### Signs of Context Degradation

In extended sessions, the model begins:
- Referencing "typical patterns" instead of specific discovered facts
- Giving answers inconsistent with earlier turns
- "Forgetting" entity names, amounts, or decisions made earlier
- Contradicting its own earlier reasoning

**These are structural failures, not model errors.** The fix is always architectural.

### Three Remediation Strategies

#### Strategy 1: Scratchpad Files

For long-running agentic tasks (hours, multi-day), write key findings to a persistent file on disk. The agent reads the scratchpad at the start of each new session:

```python
# Agent writes to scratchpad as it discovers facts
def update_scratchpad(finding: dict):
    with open("agent_scratchpad.json", "a") as f:
        json.dump({
            "timestamp": datetime.now().isoformat(),
            "finding": finding,
            "source": finding.get("source_url"),
            "session_id": current_session_id
        }, f)
        f.write("\n")

# On session resume — inject scratchpad findings at top
def resume_session(scratchpad_path: str) -> str:
    findings = load_scratchpad(scratchpad_path)
    return f"""
PRIOR_SESSION_FINDINGS (authoritative — do not contradict):
{json.dumps(findings, indent=2)}

Continue from where we left off.
"""
```

#### Strategy 2: /compact Command

In Claude Code, the `/compact` command reduces context usage during extended exploration:
- Compacts the conversation history into a dense summary
- Preserves key decisions and findings
- Frees up context window for continued work

**When to use `/compact`:** when context usage indicator shows high usage (70%+) during an ongoing exploration or code analysis session.

**The exam rule:** `/compact` is a Claude Code feature — it is NOT available in the Claude API directly. The API equivalent is manually summarizing old turns and replacing them with a compact summary.

#### Strategy 3: Context Windowing at Turn N

For agentic loops, implement windowing when the loop hits turn 5 (or when context exceeds 70% of the limit):

```python
MAX_TURNS_BEFORE_WINDOW = 5

async def run_agent_loop(task: str, case_facts: dict):
    messages = []
    turn = 0
    
    while True:
        turn += 1
        
        # Windowing at turn 5: summarize old turns, keep CASE_FACTS fresh
        if turn == MAX_TURNS_BEFORE_WINDOW and len(messages) > 8:
            prior_summary = await summarize_history(messages[:-2])
            messages = [{
                "role": "user",
                "content": f"""
CASE_FACTS (always authoritative):
{json.dumps(case_facts, indent=2)}

PRIOR_WORK_SUMMARY:
{prior_summary}

Continue with the current task.
"""
            }]
        
        response = await call_claude(messages)
        if response.stop_reason == "end_turn":
            return response
        
        # Handle tool_use, append results...
        messages.append(...)
```

### The Subagent Delegation Strategy

When the main agent's context is filling up, spawn a subagent to handle specific sub-tasks:
- Main agent: preserves high-level coordination context
- Subagent: gets a clean context for the specific investigation
- Subagent returns a condensed summary — not raw output

```python
# ✅ CORRECT — delegate to subagent when main context is getting large
if context_usage > 0.70:
    # Spawn fresh subagent with targeted task
    subagent_result = spawn_subagent(
        task=f"Investigate files 23-47. Prior context: {minimal_context_summary}",
        allowed_tools=["Read", "Grep"]
    )
    # Subagent returns condensed finding (1-2K tokens, not 20K)
    main_context.append(f"Subagent finding: {subagent_result['summary']}")
```

---

## 7b. Conversation Compaction — The Standard CALM Strategy
> ⭐ **The claudecertifiedarchitects.com guide uses conversation compaction as the definitive CALM exam example. "Summarising completed steps into a dense state block is the standard CALM strategy."**

### What Conversation Compaction Is

Conversation compaction is the act of replacing verbose conversation history with a dense, information-rich summary — specifically when approaching the context limit — while keeping the agent's task continuity intact.

It is **different** from:
- **Full history clear** (loses all accumulated state)
- **Scratchpad persistence** (for multi-session persistence, not within-session compaction)
- **Subagent delegation** (for offloading work, not reducing the main agent's history)

### The Compaction Pattern

```python
# ✅ CORRECT — compact when approaching context limit
async def compact_and_continue(messages: list, case_facts: dict, pending_steps: list) -> list:
    """
    Standard CALM compaction: summarise completed turns into dense state block.
    Called when context_usage > 70%.
    """
    # Step 1: Summarise completed work (keep it dense — not prose)
    completed_summary = await claude.summarise(
        content=messages,
        instruction="Produce a dense state summary of what has been accomplished. "
                    "Preserve exact values, IDs, and findings. Do not use vague language."
    )
    
    # Step 2: Build fresh context — CASE_FACTS intact, summary at top
    fresh_messages = [{
        "role": "user",
        "content": f"""
CASE_FACTS (never changes — use exact values):
{json.dumps(case_facts, indent=2)}

COMPLETED_WORK_SUMMARY:
{completed_summary}

REMAINING_STEPS:
{json.dumps(pending_steps, indent=2)}

Continue with the first remaining step.
"""
    }]
    
    return fresh_messages  # Continue from here — task continuity preserved
```

### Compaction Decision Tree

```
Context usage < 70%?
→ No action needed — continue normally

Context usage 70-85%?
→ Compact: summarise completed turns, keep CASE_FACTS, rebuild lean context

Context at 85-95%?
→ Urgent compact + delegate verbose sub-tasks to subagent

Context at 95%+?
→ Emergency compact + new session with manifest injection
   (you waited too long — act earlier next time)
```

### Compaction vs. Clearing — The Critical Distinction

```
Clearing:    Delete all history → fresh start → lose all accumulated work
             ❌ ALWAYS WRONG during active task execution

Compaction:  Replace verbose history with dense summary → preserve task state
             ✅ CORRECT — task continuity preserved without losing work
```

**The exam answer:** When a question asks "an agent is approaching the context limit with 3 steps remaining — what should it do?" — the answer is ALWAYS compaction (or the CALM equivalent), not clearing, not erroring, not increasing max_tokens.

---

## 8. Error Propagation in Multi-Agent Systems (T5.2)

> ⭐ **The exam tests structured error propagation — not just "did it fail" but exactly what information the coordinator needs to recover intelligently.**

### The Principle

When a subagent fails, it must NOT:
- Return an empty result and pretend it succeeded
- Return a vague error string ("operation failed")
- Crash silently

It MUST return a **structured error object** with enough context for the coordinator to decide: retry? fallback? escalate? use partial results?

### The Structured Error Object — Required Fields

```json
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
  "suggested_recovery": "Retry with narrower date range",
  "retry_after_ms": 2000
}
```

**The four required fields for coordinator recovery:**
1. `error_category` — what kind of failure (transient/validation/permission/business)
2. `is_retryable` — boolean — can coordinator retry automatically?
3. `attempted_query` — what the subagent was doing when it failed
4. `partial_results` — whatever was retrieved before failure

### The Two-Tier Recovery Pattern

```python
# Tier 1: Subagent handles transient errors locally (don't bother coordinator)
async def subagent_with_local_recovery(query: str) -> dict:
    for attempt in range(3):
        try:
            return await execute_query(query)
        except TransientTimeout:
            if attempt == 2:
                # Exhausted local retries → structured error to coordinator
                return {
                    "status": "error",
                    "error_category": "transient",
                    "is_retryable": True,
                    "attempted_query": query,
                    "partial_results": accumulated_so_far,
                    "message": "Timed out after 3 attempts"
                }
            await asyncio.sleep(2 ** attempt)

# Tier 2: Coordinator makes recovery decision based on structured error
def coordinator_recovery(error: dict, other_subagent_results: list):
    if not error.get("is_retryable"):
        # Validation/permission/business errors — don't retry
        handle_permanent_failure(error)
    elif error["error_category"] == "transient" and retry_count < MAX_RETRIES:
        retry_subagent(error["attempted_query"])
    
    # ALWAYS continue processing independent subagents
    process_successful_results(other_subagent_results)
    
    # ALWAYS include partial results in output
    include_partial_with_annotation(error["partial_results"])
```

### The Key Rules

```
❌ WRONG — terminate all subagents when one fails
✅ CORRECT — log error, continue independent subagents, include partial results

❌ WRONG — silently return empty result on failure (pretend it succeeded)
✅ CORRECT — always return structured error with is_retryable + partial_results

❌ WRONG — retry indefinitely on any error
✅ CORRECT — check is_retryable first; never retry validation/permission errors

❌ WRONG — discard partial results because the query didn't complete
✅ CORRECT — partial results have real value; annotate + include in output
```

---

## 9. Escalation Design (T5.3)

> ⭐ **The exam distinguishes reliable from unreliable escalation triggers. Sentiment analysis and confidence scores are named wrong answers.**

### Reliable vs Unreliable Triggers

| Trigger | Reliable? | Why |
|---|---|---|
| Customer explicitly requests a human | ✅ Reliable | Unambiguous — no inference needed |
| Amount exceeds threshold (>$500) | ✅ Reliable | Programmatic check — deterministic |
| Retry count reaches maximum (≥3) | ✅ Reliable | Programmatic count — deterministic |
| Irreversible action about to execute | ✅ Reliable | Categorical — no ambiguity |
| Policy gap (no rule covers this case) | ✅ Reliable | Agent explicitly recognizes its limit |
| Unable to make meaningful progress | ✅ Reliable | Defined programmatic threshold |
| Sentiment analysis shows frustration | ❌ Unreliable | LLM-based — probabilistic, non-deterministic |
| Model's self-reported confidence score | ❌ Unreliable | Poorly calibrated — not a reliable signal |
| Automatic emotion classifier | ❌ Unreliable | ML classifier — non-zero error rate |
| "Complex case" (vague judgment) | ❌ Unreliable | No clear definition — varies by context |

### The Correct Escalation Implementation

```python
def should_escalate(action: dict, context: AgentContext) -> tuple[bool, str]:
    # Priority 1 — explicit customer request (ALWAYS immediate, no delay)
    if any(phrase in context.customer_message.lower() for phrase in
           ["speak to a human", "talk to a person", "real agent", "supervisor"]):
        return True, "customer_explicit_request"
    
    # Priority 2 — threshold exceeded (programmatic, deterministic)
    if action.get("type") == "refund" and action.get("amount", 0) > 500:
        return True, f"refund_exceeds_autonomous_limit_${action['amount']}"
    
    # Priority 3 — repeated failure
    if context.retry_count >= 3:
        return True, f"max_retries_exceeded_{context.retry_count}"
    
    # Priority 4 — irreversible action
    if action.get("irreversible") and not action.get("pre_approved"):
        return True, "irreversible_action_requires_approval"
    
    # Priority 5 — policy gap
    if context.policy_lookup_result == "no_matching_policy":
        return True, "policy_gap_requires_human_interpretation"
    
    return False, None
```

### What Explicit Customer Request Means

**"Honor explicit customer requests for human agents immediately without first attempting investigation."**

```
❌ WRONG: "Let me try to resolve this for you first — I can usually help!"
   Then investigate, then escalate if can't resolve.
   
✅ CORRECT: Immediately escalate.
   The customer's explicit request overrides the agent's desire to resolve.
   Do not attempt investigation. Do not ask why. Transfer immediately.
```

**The exam trap:** A question describes an agent that "attempts one more resolution before escalating" when the customer requests a human. This is always wrong — explicit request = immediate escalation, no exceptions.

### The Structured Handoff Summary

When escalating, the receiving human agent may not have access to the conversation transcript. The structured handoff summary must give them everything they need to act immediately:

```json
{
  "handoff_summary": {
    "customer_id": "CUST-48291",
    "customer_email": "customer@example.com",
    "issue_type": "refund_request_damaged_item",
    "root_cause": "Order ORD-9821 delivered damaged — customer has photos",
    "refund_amount_requested": 847.00,
    "autonomous_limit": 500.00,
    "reason_for_escalation": "Refund exceeds $500 autonomous limit",
    "recommended_action": "Approve full refund — damage clearly documented",
    "retry_count": 2,
    "conversation_tone": "Patient and cooperative",
    "escalated_at": "2026-05-26T10:32:00Z",
    "conversation_summary": "Customer contacted twice previously. Item confirmed damaged via photos submitted on 2026-04-15."
  }
}
```

**What the exam tests in handoff summaries:** every question about "what should be included" checks for:
- Customer ID + contact info
- Root cause (specific, not vague)
- Exact amounts
- Reason for escalation (the rule that triggered it)
- Recommended action
- Conversation tone/history summary

---

## 10. Confidence Scoring & Human Review Routing (T5.5)

### Field-Level Confidence Scores

Overall confidence scores ("I'm 70% confident") are unreliable for routing decisions. Field-level confidence is more actionable:

```json
{
  "extraction": {
    "invoice_id": {"value": "INV-2026-0042", "confidence": "high"},
    "vendor_name": {"value": "Acme Corp", "confidence": "high"},
    "total_amount": {"value": 2847.50, "confidence": "medium"},
    "tax_rate": {"value": null, "confidence": "low"},
    "line_items": [...]
  },
  "overall_confidence": "medium",
  "requires_human_review": true,
  "review_reason": "tax_rate not found in document; total_amount partially legible"
}
```

### Routing Rules

```python
def route_extraction(result: dict) -> str:
    # Immediate human review
    if result["requires_human_review"]:
        return "human_review_queue"
    
    # Any critical field with low confidence
    critical_fields = ["total_amount", "vendor_name", "invoice_id"]
    for field in critical_fields:
        if result["extraction"].get(field, {}).get("confidence") == "low":
            return "human_review_queue"
    
    # High overall confidence — auto-process
    if result["overall_confidence"] == "high":
        return "auto_process"
    
    # Medium confidence — spot-check sample
    return "spot_check_sample"
```

### Calibration — Why Self-Reported Confidence Fails

The exam explicitly names model self-reported confidence scores as an **unreliable escalation trigger**:
- A model that says "I'm 95% confident" may be wrong on 30% of cases
- Confidence scores are not calibrated to real-world accuracy without explicit calibration
- Use labeled validation sets to calibrate: if model says "high confidence" and is right 95% of the time on the validation set, THEN "high confidence" is a reliable signal

```python
# ✅ CORRECT — calibrate before trusting
def calibrate_confidence_threshold(validation_set: list) -> float:
    """Find the confidence threshold where accuracy meets target."""
    for threshold in [0.9, 0.85, 0.8, 0.75, 0.7]:
        high_confidence = [r for r in validation_set if r["confidence"] >= threshold]
        accuracy = sum(1 for r in high_confidence if r["correct"]) / len(high_confidence)
        if accuracy >= TARGET_ACCURACY:
            return threshold
    return 0.95  # Conservative fallback
```

---

## 11. Information Provenance & Source Attribution (T5.6)

> ⭐ **The exam tests claim-source mapping — every fact in a cited report must trace to a specific source with URL, title, excerpt, and date.**

### What Provenance Means

**Provenance** = the ability to trace every factual claim back to its original source. In multi-agent research systems, provenance is often lost when:
- Subagents return plain text findings
- Summaries are passed without source attribution
- Information is synthesized without claim-source mappings

### Claim-Source Mapping — The Required Schema

```json
{
  "findings": [
    {
      "claim": "The company achieved $240M revenue in FY2025",
      "evidence_excerpt": "Total revenue for fiscal year 2025 was $240 million...",
      "source_url": "https://company.com/investor-relations/annual-report-2025",
      "source_title": "Annual Report FY2025",
      "source_type": "primary",
      "publication_date": "2026-02-14",
      "retrieved_at": "2026-05-26T10:15:00Z",
      "confidence": "high"
    },
    {
      "claim": "CEO transition occurred in March 2026",
      "evidence_excerpt": "Board appoints new CEO effective March 1, 2026...",
      "source_url": "https://company.com/press/ceo-announcement",
      "source_title": "Press Release — Leadership Update",
      "source_type": "primary",
      "publication_date": "2026-02-28",
      "retrieved_at": "2026-05-26T10:18:00Z",
      "confidence": "high"
    }
  ]
}
```

### Why Every Field Matters

| Field | Why It's Required |
|---|---|
| `claim` | The specific assertion being attributed |
| `evidence_excerpt` | The exact text from the source that supports the claim |
| `source_url` | The verifiable location of the original source |
| `source_title` | Identifies the source without requiring URL verification |
| `publication_date` | Prevents temporal confusion (see §12) |
| `retrieved_at` | Documents when the data was accessed |
| `confidence` | Signals when human verification is warranted |

### When Provenance Is Lost

```python
# ❌ WRONG — subagent returns plain text
return f"The company made $240M in revenue last year and the CEO changed."
# Synthesis agent cannot cite this — has no source URL, no evidence excerpt

# ✅ CORRECT — subagent returns structured claim-source mapping
return {
    "findings": [
        {
            "claim": "Company achieved $240M revenue in FY2025",
            "evidence_excerpt": "Total revenue was $240 million...",
            "source_url": "https://...",
            "publication_date": "2026-02-14",
            ...
        }
    ]
}
```

**The exam rule:** A synthesis agent cannot produce a cited report if it only receives plain claims without evidence. Every fact needs: the claim, supporting excerpt, source URL, title, and dates.

---

## 12. Conflicting Source Data — Annotation, Never Arbitrary Resolution

> ⭐ **Silently resolving conflicts is always the wrong answer. Annotating both values with attribution is always the correct answer.**

### The Problem

When multiple sources report different values for the same fact, the naive approach is to pick the "better" source. This is wrong because:
- You may not know which source is correct
- Downstream users lose visibility into the discrepancy
- The downstream report inherits an unacknowledged error

### The Correct Pattern

```json
// ❌ WRONG — coordinator silently picks one value
{
  "revenue_2025": "$240M"   // Source A picked, Source B's $195M silently ignored
}

// ✅ CORRECT — conflict annotated, both sources preserved
{
  "revenue_2025": {
    "value_source_a": "$240M",
    "value_source_b": "$195M",
    "sources": [
      {
        "title": "Annual Report 2025",
        "url": "https://company.com/annual-report",
        "date": "2026-02-14"
      },
      {
        "title": "Industry Analyst Report Q4",
        "url": "https://analyst.com/report",
        "date": "2026-01-20"
      }
    ],
    "conflict": true,
    "annotation": "Values differ — may reflect different accounting periods or revenue definitions",
    "requires_human_resolution": true
  }
}
```

### The Exam Pattern

```
Scenario: "Two subagents both research company revenue. Subagent A finds $240M
           from the annual report. Subagent B finds $195M from an analyst report.
           The coordinator selects $240M because 'primary sources are more reliable.'
           What is wrong?"

Wrong answer: "Nothing — choosing the primary source is correct practice"

Correct answer: "The coordinator should annotate the conflict with both values and
                both sources. Arbitrarily selecting one value hides the discrepancy
                from downstream users who may need to investigate the difference."
```

---

## 13. Human Review Workflows — Stratified Sampling & Calibration (T5.5)

### Why Aggregate Accuracy Is Not Enough

A system with 97% overall accuracy may be 60% accurate on a specific document type. Aggregate metrics mask this:

```python
# ❌ WRONG — measuring only overall accuracy
overall_accuracy = correct / total  # 97%
# Decision: "System is ready for full automation"

# ✅ CORRECT — accuracy by document type AND field segment
accuracy_by_type = {
    "digital_invoice": 99.2%,      # Near-perfect
    "handwritten_form": 61.3%,     # Unacceptably low
    "scanned_pdf_old": 73.8%,      # Below threshold
    "standard_template": 98.7%,   # High
}
# Decision: automate digital_invoice; route handwritten_form to human review
```

### Stratified Random Sampling

For measuring error rates in high-confidence extractions:

```python
# ✅ CORRECT — stratified sample that preserves category proportions
def stratified_sample(results: list, sample_rate: float = 0.05) -> list:
    """Sample 5% from each confidence stratum for human review."""
    strata = {"high": [], "medium": [], "low": []}
    for r in results:
        strata[r["confidence"]].append(r)
    
    sample = []
    for confidence, items in strata.items():
        n = max(1, int(len(items) * sample_rate))
        sample.extend(random.sample(items, min(n, len(items))))
    return sample

# Why stratified vs random: a pure random sample from a 90% high-confidence dataset
# gives very few medium/low examples — you can't measure those error rates accurately.
```

**The exam rule:** Stratified random sampling measures error rates in each confidence stratum separately. A pure random sample from a dataset with mostly high-confidence results undersamples the rare low-confidence cases — precisely the ones most likely to be wrong.

### Human Review Queue Design

```python
def route_to_review_queue(result: dict) -> str:
    """Route extractions to appropriate review queue."""
    
    # Immediate escalation — known high-error categories
    if result.get("document_type") in HIGH_ERROR_DOCUMENT_TYPES:
        return "priority_review_queue"
    
    # Field-level routing — critical fields with uncertainty
    for field in CRITICAL_FIELDS:
        field_result = result.get("extraction", {}).get(field, {})
        if field_result.get("confidence") == "low":
            return "standard_review_queue"
    
    # Conflict detected
    if result.get("conflict_detected"):
        return "conflict_resolution_queue"
    
    # requires_human_review flag set by extraction
    if result.get("requires_human_review"):
        return "standard_review_queue"
    
    # High confidence across all fields
    return "auto_process"
```

---

## 14. Crash Recovery — Structured State Manifests

> ⭐ **Production agents must survive crashes without starting over. The exam tests the correct state manifest design.**

### The Problem

For multi-hour or multi-day agentic tasks, a crash means losing all in-memory context. The agent must be able to resume from where it stopped.

### The State Manifest Pattern

```json
{
  "manifest": {
    "task_id": "TASK-002",
    "checkpoint_at": "2026-05-26T14:32:00Z",
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
    "key_findings": [
      {
        "finding": "Auth service uses JWT RS256, not HS256",
        "discovered_at": "2026-05-26T13:45:00Z",
        "source_file": "src/auth/jwt.ts"
      }
    ],
    "files_processed": ["src/auth.ts", "src/user.service.ts"],
    "files_remaining": ["src/order.service.ts"],
    "session_facts": {
      "authorized_scope": "read-only",
      "task_owner": "eng-team@company.com",
      "hard_deadline": "2026-05-27T09:00:00Z"
    }
  }
}
```

### The Recovery Pattern

```python
# Write manifest at regular checkpoints (every N files, every M minutes)
def write_checkpoint(state: AgentState, path: str = "checkpoint.json"):
    manifest = {
        "task_id": state.task_id,
        "checkpoint_at": datetime.now().isoformat(),
        "completed_steps": state.completed,
        "pending_steps": state.pending,
        "key_findings": state.findings,
        "session_facts": state.immutable_facts
    }
    with open(path, "w") as f:
        json.dump(manifest, f)

# On restart after crash — load manifest and inject into fresh session
def resume_from_checkpoint(path: str) -> str:
    manifest = json.load(open(path))
    return f"""
You are resuming a task that was interrupted. Load this state:

COMPLETED_STEPS:
{json.dumps(manifest["completed_steps"], indent=2)}

PENDING_STEPS (start here):
{json.dumps(manifest["pending_steps"], indent=2)}

KEY_FINDINGS (do not re-discover — use these as authoritative):
{json.dumps(manifest["key_findings"], indent=2)}

SESSION_FACTS (immutable):
{json.dumps(manifest["session_facts"], indent=2)}

Continue with the first item in PENDING_STEPS.
"""
```

**The exam rule:** An agent that crashes and must restart from zero is a design failure. Production agents write state manifests at checkpoints and inject them into fresh sessions on restart.

---

## 15. Token Budget Management (T5.4)

### Trimming Verbose Tool Output

Tools often return far more data than the agent needs. A `lookup_order` tool returning 40 fields when only 5 are relevant bloats the context with every call.

**The fix: PostToolUse hook that trims before Claude sees the result:**

```python
# PostToolUse hook — trim verbose tool output
def trim_order_result(tool_name: str, tool_result: dict) -> dict:
    if tool_name == "lookup_order" and not tool_result.get("isError"):
        content = tool_result.get("content", {})
        # Keep only the 5 fields needed for refund decisions
        tool_result["content"] = {
            "order_id":   content.get("order_id"),
            "status":     content.get("status"),
            "total":      content.get("total"),
            "items":      content.get("items"),
            "created_at": content.get("created_at")
        }
        # Discard: warehouse_id, billing_address, tax_breakdown,
        #          carrier_metadata, internal_flags... (35 other fields)
    return tool_result
```

| Without trimming | With trimming |
|---|---|
| 40 fields × every turn = rapid context growth | 5 relevant fields per turn — minimal footprint |
| Context rot accelerates | Context window stays lean longer |
| Claude processes noise, increasing error rate | Claude sees only what it needs |

### The Decision: Trim vs Summarize vs Delegate

| Context problem | Strategy |
|---|---|
| Verbose tool outputs | PostToolUse trimming (fixes at the tool layer) |
| Long conversation history | Progressive summarization + CASE_FACTS block |
| Single agent doing too much | Delegate to subagent (fresh context window) |
| Context reaching 70%+ | CALM compaction: summarise completed turns, keep CASE_FACTS |
| Need to survive crash | Write state manifest to disk at checkpoint |
| Very long session (hours) | Scratchpad files + periodic session restart |

---

## 15b. RAG Architecture in D5 Context
> ⭐ **The Masterclass D5 syllabus explicitly lists: "RAG architecture — chunking, embedding, and the retrieval pipeline, semantic search, BM25, and hybrid retrieval, multi-index RAG, production hardening, and citation."**

### What RAG Is in the CCA-F Context

Retrieval-Augmented Generation (RAG) is a technique for providing Claude with relevant context from an external knowledge base rather than loading everything into the context window upfront. Instead of front-loading all documents, the agent retrieves only the relevant chunks at query time — this is also called **just-in-time (JIT) retrieval** and is part of the CALM framework's "Minimise" principle.

### When RAG Solves a Context Problem

```
Problem: Knowledge base has 50,000 documents — can't fit in context window
Wrong fix: Increase the context window (exponential cost, context rot)
Right fix: RAG — embed documents, retrieve relevant chunks at query time

Problem: Agent context fills up with reference material
Wrong fix: Summarize the reference docs (loses detail)
Right fix: Progressive disclosure — load specific reference docs only when needed
```

### The Three Retrieval Strategies

| Strategy | How It Works | When to Use |
|---|---|---|
| **Semantic search** (vector/embedding) | Finds conceptually similar chunks using cosine similarity | Natural language queries, conceptual lookups |
| **BM25** (keyword/lexical) | Exact keyword matching with TF-IDF weighting | Known specific terms, codes, IDs, proper nouns |
| **Hybrid retrieval** | Combines semantic + BM25 scores | Production systems — catches both conceptual and exact matches |

**The exam rule:** Hybrid retrieval (semantic + BM25) outperforms either alone in production:
- Semantic alone misses: exact codes, specific IDs, technical jargon
- BM25 alone misses: paraphrased queries, synonyms, conceptual lookups
- Hybrid catches both — use it for production RAG systems

### Citation in RAG Outputs

RAG systems must preserve source attribution through retrieval and synthesis — this connects directly to T5.6 (Information Provenance):

```python
# ✅ CORRECT — RAG retrieval preserves source metadata
retrieved_chunks = vector_store.search(query, top_k=5)
for chunk in retrieved_chunks:
    assert chunk.metadata.get("source_url")       # Required
    assert chunk.metadata.get("source_title")     # Required
    assert chunk.metadata.get("publication_date") # Required for temporal provenance
    assert chunk.metadata.get("chunk_id")         # For deduplication
```

**The exam rule:** RAG chunks must carry source metadata through the retrieval pipeline. If a retrieved chunk doesn't have its source URL and publication date, the final cited output will have citation gaps — the same information provenance failure as plain-text subagent responses.

### Progressive Disclosure — JIT Loading

Progressive disclosure is loading reference content only when it becomes relevant, rather than front-loading everything:

```
Front-loading (wrong): Load all 50 reference documents at session start
                        → Context rot, attention dilution, high cost

Progressive disclosure (right): Load relevant documents when the task reaches them
                                  → Clean context, focused attention, lower cost
```

**The trade-off the exam tests:** Progressive disclosure requires accurate trigger logic. If the trigger fires too late, the agent may not have reference material when it needs it. If trigger logic is wrong, needed content never loads. For most production cases, hybrid retrieval (on-demand via RAG) + CALM compaction is the correct architecture.

---

## 15c. Multi-Turn Conversation Design
> ⭐ **Explicitly listed in the official D5 description: "multi-turn conversation design that maintains quality over many turns."**

### The Quality Degradation Pattern

Multi-turn conversations degrade in specific, predictable ways:

```
Early turns (1-5):  High quality — all context fresh, agent reasoning is sharp
Middle turns (5-15): Quality risk — early facts start to fade, context accumulating
Late turns (15+):   Degradation risk — lost-in-middle, context rot, inconsistent answers
```

### Design Patterns for Quality Across Many Turns

**Pattern 1 — State anchoring every N turns**

```python
# Inject a state anchor every 5 turns to refresh key facts
if turn_number % 5 == 0:
    state_anchor = f"""
STATE_CHECKPOINT (turn {turn_number}):
- Verified: {', '.join(verified_facts)}
- Outstanding: {', '.join(pending_items)}
- Decisions made: {', '.join(decisions)}
CASE_FACTS: {json.dumps(case_facts)}
"""
    prepend_to_next_message(state_anchor)
```

**Pattern 2 — Inject current question at end (recency bias)**

Always place the current question/request at the END of the user message so it benefits from recency bias. Background context goes in the middle, CASE_FACTS at the top:

```
[CASE_FACTS at top — high attention]
[PRIOR_CONTEXT in middle — acceptable for background]
[CURRENT_QUESTION at end — recency bias = high attention]
```

**Pattern 3 — Compact at turn N, not at limit**

Don't wait for the context to fill — compact proactively. The exam tests that compaction should happen at 70-80%, not 95-100%.

### Multi-Turn Anti-Patterns

| Anti-Pattern | Why It Fails | Fix |
|---|---|---|
| No CASE_FACTS block | Amounts/IDs drift via summarization | Inject CASE_FACTS every turn |
| Current question buried in middle | Low attention zone — often partially ignored | Place at end (recency bias) |
| Waiting until context is full to compact | At 100%, truncation is from the middle | Compact at 70-80% |
| Growing message history cached | Cache key changes every turn = cache miss | Cache only system prompt + tool definitions |
| No state anchor in long sessions | Facts forgotten, contradictions increase | State checkpoint every 5 turns |

---

## 15d. Prompt Caching in D5 (cache_control Breakpoints)
> ⭐ **Explicitly listed in the official D5 description: "prompt caching with cache_control breakpoints."** While prompt caching was covered fully in D4, the D5 context is specifically about how caching interacts with **context management strategy**.

### D5's Specific Angle on Prompt Caching

In D4, prompt caching was about cost reduction. In D5, it's about **context architecture**:

| D4 angle (cost) | D5 angle (context management) |
|---|---|
| ~90% cost savings on cached tokens | Which context components to cache vs. compact |
| TTL = 5 minutes | Cached content = high-stability content that doesn't need compaction |
| Cache immutable content | The CASE_FACTS block + system prompt are the ideal cache candidates |

### The D5 Caching + CALM Strategy Interaction

```python
# ✅ CORRECT — cache what's stable; compact what grows
system_prompt = [{
    "type": "text",
    "text": SYSTEM_PROMPT,           # Never changes → cache it
    "cache_control": {"type": "ephemeral"}
}]

tool_definitions = TOOLS            # Stable → cached by SDK automatically

# What NOT to cache — these grow and change:
# ❌ Growing message history → compact instead
# ❌ CASE_FACTS (changes with source system) → inject fresh every turn

# What to DO with CASE_FACTS: inject at top of user message every turn
# (not cached — fetched fresh from source system)
user_message = f"""
CASE_FACTS (fresh from source, not cached):
{json.dumps(fetch_fresh_case_facts(), indent=2)}

CURRENT_QUESTION:
{current_question}
"""
```

### The cache_control Breakpoint Pattern

For long system prompts with multiple sections, use `cache_control` breakpoints strategically:

```python
# Multiple cache breakpoints for a complex system prompt
system_prompt = [
    {
        "type": "text",
        "text": COMPANY_POLICIES,              # Changes quarterly → cache
        "cache_control": {"type": "ephemeral"}
    },
    {
        "type": "text",
        "text": AGENT_ROLE_AND_CONSTRAINTS,    # Never changes → cache
        "cache_control": {"type": "ephemeral"}
    }
    # CASE_FACTS go in user message, NOT system prompt
]
```

**The D5 exam rule:** Prompt caching and CALM compaction work together. Cache what's stable (system prompt, tools). Compact what grows (conversation history). Inject fresh what's transactional (CASE_FACTS). Never cache the growing message history.

---

## 16. Domain 5 in the Exam Scenarios

### Scenario 1 — Customer Support (Primary D5 Scenario)

| Challenge | Correct Approach |
|---|---|
| Exact refund amount quoted vaguely after 15 turns | CASE_FACTS block with exact `refund_amount` outside summarized history |
| Agent attempts more resolution when customer requests human | Immediately escalate — explicit request overrides agent autonomy |
| Escalation based on frustrated tone | Wrong — use explicit request + threshold + retry count triggers only |
| Tool returns 40 fields but only 5 needed | PostToolUse trimming hook |
| Human agent lacks context when receiving handoff | Structured handoff summary: customer ID, root cause, amount, escalation reason, recommendation |
| Multiple customers match the search | Request additional identifiers — never guess based on heuristics |

### Scenario 3 — Multi-Agent Research (Primary D5 Scenario)

| Challenge | Correct Approach |
|---|---|
| Synthesis agent can't cite sources | Subagents must return claim-source mappings, not plain text |
| Two sources report different revenue figures | Annotate conflict with both values + sources — never pick one silently |
| Earlier research findings missed in large synthesis | Key findings summary at top of aggregated input; section headers |
| Agent crashes mid-task after 4 hours | State manifest written at checkpoints; resume by injecting manifest |
| Research report omits entire topic areas | Fix coordinator decomposition — not subagent instructions |
| Dates missing from source citations | Require `publication_date` in every claim-source mapping |

### Scenario 6 — Structured Data Extraction (D5 Crossover)

| Challenge | Correct Approach |
|---|---|
| System shows 97% accuracy but poor on handwritten forms | Analyze accuracy by document type and field segment |
| Hard to tell which extractions to review | Field-level confidence scores + `requires_human_review` flag |
| Measuring error rate in high-confidence pool | Stratified random sampling (5% from each confidence stratum) |
| Confidence scores used as escalation triggers | Calibrate against labeled validation set before trusting |

---

## 17. Anti-Patterns Master List

| Anti-Pattern | Why It's Wrong | Correct Approach |
|---|---|---|
| Storing critical numbers only in conversation history prose | Progressive summarization compresses exact values | CASE_FACTS block with structured JSON injected at every prompt |
| Relying on prose summary for exact amounts/dates | "Approximately $800" ≠ "$847.00" | Extract numerics into structured CASE_FACTS block |
| Using self-reported model confidence as escalation trigger | Poorly calibrated — model says "95% confident" and is often wrong | Calibrate against labeled validation set; use field-level confidence |
| Using sentiment analysis to detect escalation cases | LLM-based — probabilistic, non-deterministic | Explicit threshold triggers: amount > $X, retry >= N, explicit request |
| Attempting more resolution when customer requests human | Customer autonomy — explicit request must be honored immediately | Escalate immediately on explicit request — no delay, no investigation |
| Silently picking one value when sources conflict | Hides discrepancy — downstream inherits hidden error | Annotate both values with sources; set `conflict: true`, route to human |
| Returning plain text findings from subagents | Synthesis agent can't cite what it doesn't have in structured form | Return claim-source mappings with URL, excerpt, date for every finding |
| Aggregate accuracy metric only (97% overall) | Masks poor performance on specific document types or fields | Break down accuracy by document type AND field segment |
| Random sampling instead of stratified sampling | Undersamples rare low-confidence cases — can't measure their error rate | Stratified sampling: N% from each confidence stratum |
| Silently returning empty results as success | Coordinator thinks query succeeded; missing data causes wrong decisions | Return structured error with is_retryable, attempted_query, partial_results |
| Terminating all subagents on one failure | Kills independent work unnecessarily | Log error, continue independent subagents, include partial results |
| No crash recovery for long-running agents | Agent crash = restart from zero | Write state manifest at checkpoints; inject on resume |
| Summarizing without preserving exact values | Dates, amounts, IDs disappear | Two-layer: CASE_FACTS for exact values + prose for narrative context |
| Bold/caps to make critical facts "stand out" | Transformers don't parse Markdown emphasis — position determines attention | Move to system prompt or top of user message (high-attention zones) |
| No claim-source mapping in research output | Can't produce a cited report from uncited claims | Every finding: claim + evidence excerpt + source URL + dates |
| Missing publication dates in source citations | Temporal differences misread as contradictions | Always include `publication_date` in every source citation |
| Heuristic selection when multiple records match | Risk of acting on wrong customer | Request additional identifying information — never guess |
| PostToolUse trimming skipped on verbose tools | 40-field responses accumulate across turns → context rot | Always trim to only relevant fields before Claude sees result |
| /compact confused with API summarization | /compact is a Claude Code command — not available in Claude API | In API: manually summarize old turns and replace with compact version |
| Context windowing only at 100% limit | Truncation from middle (worst position) happens at 100% | Window at 70-80% to retain control over what's kept |
| Structured handoff summary missing key fields | Human agent can't act without customer ID, amounts, root cause, recommendation | Always include: customer ID, root cause, exact amounts, escalation reason, recommended action |
| Escalation based on "case complexity" | Subjective — no clear definition | Only escalate on: explicit request, threshold exceeded, retry max reached, irreversible action, policy gap |
| Information provenance lost in synthesis | Final report can't cite sources | Require subagents to output structured claim-source mappings that pass through all synthesis layers |
| Clearing history instead of compacting | Discards all accumulated task state | CALM compaction: summarise completed turns into dense state block; keep CASE_FACTS intact |
| Increasing max_tokens when context is full | max_tokens controls output length, not input context size | Compact the conversation history; don't try to extend the input window |
| Semantic-only retrieval in RAG | Misses exact codes, IDs, technical jargon | Hybrid retrieval: semantic + BM25 — catches both conceptual and exact matches |
| RAG chunks without source metadata | Retrieved chunks lose provenance; final output can't cite | Every RAG chunk must carry: source_url, source_title, publication_date |
| Front-loading all reference documents | Context rot, attention dilution, high cost | Progressive disclosure (JIT): load relevant docs at query time, not session start |
| Caching the growing message history | Message list changes every turn → cache miss every turn | Cache only stable content (system prompt, tool definitions); message history = compact not cache |
| No state anchor in long multi-turn sessions | Facts forgotten; quality degrades; inconsistent answers | State checkpoint every 5 turns with CASE_FACTS refresh |
| CASE_FACTS in system prompt instead of user message | System prompt for constraints/role; CASE_FACTS are transactional | CASE_FACTS at top of user message (fetched fresh each turn, not cached)

---

## 18. Key Rules to Memorize

```
1.  CASE_FACTS block: structured JSON at top of every user message — never through summarization
2.  CASE_FACTS values come from the source system — never from agent's prior conversation history
3.  What gets lost in summarization: exact amounts, specific dates, all IDs, percentages
4.  Two-layer approach: CASE_FACTS block (exact) + prose summary (narrative) — never mix
5.  Progressive summarization safe for: narrative, tone, sequence of events
6.  Progressive summarization unsafe for: any specific number, ID, date, percentage, threshold
7.  Lost-in-the-middle: accuracy drops from 90%+ at position 1 to 40-50% at position 50
8.  Fix for lost-in-the-middle: position (move to top/system prompt) — NOT bold/caps/emphasis
9.  Bold/caps = zero effect on transformer attention weights
10. Context windowing: act at 70-80% of window limit — not at 100% (too late)
11. /compact = Claude Code command only — not available in the Claude API
12. Scratchpad files: for multi-hour/multi-day tasks; inject findings at start of each new session
13. Subagent delegation: main agent preserves coordination context; subagent gets fresh window
14. Structured error: error_category + is_retryable + attempted_query + partial_results (all four required)
15. is_retryable: false → NEVER retry (validation/permission/business errors)
16. Partial results: always return even if query incomplete — annotate as partial, never discard
17. Two-tier recovery: subagent retries transient errors locally; coordinator only gets unresolvable errors
18. Reliable escalation: explicit customer request | amount > threshold | retry ≥ N | irreversible action | policy gap
19. Unreliable escalation: sentiment analysis | self-reported confidence | "complex case" judgment
20. Explicit customer request for human = IMMEDIATE escalation, NO delay, NO investigation first
21. Structured handoff: customer ID + root cause + exact amounts + escalation reason + recommended action
22. When multiple customers match a query: request more identifiers — NEVER guess based on heuristics
23. Conflicting sources: annotate BOTH values with attribution — NEVER silently pick one
24. Claim-source mapping required fields: claim + evidence excerpt + source URL + source title + publication_date + retrieved_at
25. Missing source dates: temporal differences misread as contradictions — always require publication_date
26. Aggregate accuracy masks poor performance on specific document types — always segment by type AND field
27. Stratified sampling: sample from each confidence stratum separately — random sampling undersamples rare low-confidence cases
28. Confidence calibration: model self-report is unreliable until calibrated against labeled validation set
29. Field-level confidence > overall confidence for routing decisions
30. Crash recovery: state manifest at checkpoints; inject manifest into fresh session on resume
31. State manifest must contain: completed_steps, pending_steps, key_findings, session_facts (immutable facts)
32. PostToolUse trimming: filter verbose tool results (40 fields → 5 relevant) before Claude sees result
33. Context nearing limit: trim tool outputs + progressive summarization + windowing (act at 70-80%)
34. Escalation on policy gap: when policy is ambiguous or silent on the customer's specific case → always escalate
35. Render findings by content type: financial data as tables, news as prose, technical as structured lists
36. Temporal provenance: require publication_date + retrieved_at in every claim-source mapping
37. Domain 5 meta-rule: every wrong answer is a linguistic/behavioral fix; every correct answer is a structural/programmatic fix
38. D5 is lightest domain (15%) but most common source of careless wrong answers — don't underestimate it
39. Error propagation: coordinator continues independent subagents even when one fails
40. Silent suppression (returning empty as success) is ALWAYS wrong — always return structured error
41. CALM = Context-Aware LLM Management — Anthropic's named D5 framework for context window decisions
42. CALM's 5 principles: Curate (only high-signal tokens) | Anchor (critical facts at high-attention zones) | Layer (CASE_FACTS separate from prose) | Manage (act at 70-80%) | Minimise (smallest context that preserves coherence)
43. CALM's primary exam strategy: conversation compaction — summarise completed turns into dense state block; CONTINUE execution
44. CALM wrong answers: clear history (loses state) | return error (premature) | increase max_tokens (doesn't extend input window)
45. Conversation compaction ≠ history clear. Compaction: summarise completed, keep CASE_FACTS, continue. Clear: lose everything.
46. Hybrid retrieval (semantic + BM25) outperforms either alone — catches both conceptual and exact-term queries
47. RAG chunks must preserve source metadata through retrieval: source_url + source_title + publication_date
48. Progressive disclosure / JIT retrieval: load reference content at query time, not session start — prevents front-loading context rot
49. Multi-turn quality: state anchor every 5 turns, CASE_FACTS at top, current question at end (recency)
50. Prompt caching D5 angle: cache stable content (system prompt, tools); compact growing content (history); inject fresh transactional content (CASE_FACTS)
51. cache_control breakpoints: mark each immutable section of system prompt separately for granular caching
52. Never cache CASE_FACTS in the system prompt — CASE_FACTS are transactional (fetched fresh) and belong in user message
```

---

## 19. Practice Questions (20 MCQs)

---

**Q1.** After 15 turns, a customer support agent quotes a refund as "approximately $800" when the customer specified $847.00. The conversation history contains the exact amount. What is the root cause and correct fix?

- A) The model made a rounding error — use temperature=0 for exact arithmetic
- B) The exact amount was stored only in conversation history prose, which was compressed during summarization — extract transactional facts into a persistent CASE_FACTS block injected at every prompt outside summarized history
- C) The context window was exceeded — increase max_tokens
- D) The system prompt doesn't mention preserving exact amounts — add an instruction

---

**Q2.** A customer says "I'd like to speak with a human agent, please." The agent responds: "I understand — let me try to resolve this quickly for you first." What is wrong?

- A) Nothing — attempting resolution first is more efficient and often avoids unnecessary escalation
- B) The agent should ask why the customer wants a human before deciding
- C) Explicit customer requests for human agents must be honored immediately — attempting further resolution is a violation of this principle
- D) The agent should offer the customer a choice: try resolution or escalate

---

**Q3.** A multi-agent research system's subagents return findings as plain text: "The company made $240M in revenue last year." The synthesis agent cannot produce a cited report. What is the root cause?

- A) The synthesis agent lacks web search capability to find the sources
- B) Plain text findings don't preserve source attribution — subagents must return claim-source mappings (claim + evidence excerpt + source URL + publication date)
- C) The coordinator should inject citations during synthesis
- D) The report generator needs access to the original web pages

---

**Q4.** Subagent A reports company revenue as $240M from the annual report. Subagent B reports $195M from an analyst report. The coordinator silently selects $240M as "more reliable." What is wrong?

- A) Nothing — primary sources should always take precedence
- B) The coordinator should average the two values to get a more balanced estimate
- C) The coordinator should annotate both values with their sources and flag the conflict — silently selecting one hides the discrepancy from downstream users
- D) The coordinator should discard the analyst report entirely since primary sources are authoritative

---

**Q5.** A customer support agent uses sentiment analysis to detect frustrated customers and automatically escalates. In production, 23% of escalations are cases where customers were not frustrated — just using formal language. What is the root cause?

- A) The sentiment model needs more training data
- B) Sentiment analysis is a probabilistic, non-deterministic escalation trigger — replace with reliable triggers: explicit customer request, threshold exceeded, retry count reached
- C) The escalation threshold for the sentiment score is too low — raise it
- D) Add a second sentiment classifier as a validation layer

---

**Q6.** An agent that has been running for 6 hours crashes. It had processed 23 of 47 required files. On restart, it begins from file 1 again. What design pattern was missing?

- A) The agent should have used a larger context window
- B) Crash recovery requires periodic state manifests (checkpoints) written to disk — on restart, inject the manifest into a fresh session to resume from pending_steps
- C) The agent should have processed all 47 files in parallel to avoid mid-task crashes
- D) The session should have been paused rather than crashed

---

**Q7.** A lookup_order tool returns 40 fields but the agent only needs 5 for refund decisions. After 10 turns, the context window is filling up rapidly. What is the correct fix?

- A) Switch to a model with a larger context window
- B) Limit the agent to processing a maximum of 5 orders per session
- C) Use a PostToolUse hook to trim the tool result to only the 5 relevant fields before Claude sees it
- D) Summarize the tool output in the system prompt

---

**Q8.** An extraction system reports 97% overall accuracy and is approved for full automation. Six months later, a field audit discovers that handwritten forms have only 61% accuracy. What went wrong with the evaluation?

- A) The field audit is using different accuracy metrics than the original evaluation
- B) Aggregate accuracy masked poor performance on the handwritten form document type — accuracy must always be analyzed by document type AND field segment before automation decisions
- C) The model degraded over time — retrain on recent examples
- D) The 97% overall accuracy is still acceptable — 61% on a rare document type is statistically negligible

---

**Q9.** Which of the following is a reliable escalation trigger in a customer support agent?

- A) The sentiment classifier detects frustration in the customer's message
- B) The model's self-reported confidence score drops below 60%
- C) The customer uses formal language, which correlates with complex cases
- D) The refund amount requested ($750) exceeds the agent's $500 autonomous limit

---

**Q10.** A long conversation history is summarized. The resulting summary says "the customer requested a refund of approximately $800 from earlier this month." What is wrong with how this session is designed?

- A) The summary is too vague — ask the model to be more precise
- B) Exact values like refund amounts and dates must be stored in a structured CASE_FACTS block outside summarized prose, where precision is always preserved
- C) Summaries should not be used — always keep the full conversation history
- D) The summary format should use bullet points instead of prose

---

**Q11.** A research subagent times out after retrieving 2 of 5 required records. It returns: `{"status": "error", "message": "Operation failed"}`. What is wrong?

- A) Nothing — the error message is clear enough for the coordinator to retry
- B) The error response lacks error_category, is_retryable, attempted_query, and partial_results — the coordinator cannot make an intelligent recovery decision
- C) The subagent should not return an error — it should return empty results instead
- D) The error should be raised as an exception, not returned as a dict

---

**Q12.** When a customer's name matches two records in the system, the agent selects the one with the most recent activity, reasoning "this is probably the right customer." What is wrong?

- A) Nothing — most recent activity is a reasonable heuristic
- B) Heuristic selection risks acting on the wrong customer record — always request additional identifiers (email, order number) to confirm identity
- C) The agent should escalate to a human immediately when there are two matching records
- D) The agent should process both records and let the customer confirm

---

**Q13.** A subagent uses the `/compact` command to reduce context usage during an API-based agentic loop. What is the problem?

- A) /compact is only available in paid tiers of the Claude API
- B) /compact is a Claude Code command — it is not available in the Claude API; the API equivalent is manually summarizing old turns
- C) /compact permanently deletes context — use context windowing instead
- D) /compact requires the extended context window beta header

---

**Q14.** For measuring error rates in high-confidence extractions, a team uses pure random sampling (5% of all results). What is the limitation?

- A) 5% is too small a sample — use at least 10%
- B) Pure random sampling undersamples rare low-confidence cases — use stratified sampling to ensure each confidence stratum is represented proportionally
- C) Random sampling introduces bias — use sequential sampling instead
- D) Sampling should not be used — review all extractions

---

**Q15.** A research report's source citations are missing publication dates. After 3 months, a user notices that an "outdated" figure from 2024 is being cited alongside a 2026 figure for the same metric, making it look like a contradiction. What should have been done?

- A) All sources should have been verified for currency before inclusion
- B) Publication dates should always be required in claim-source mappings — temporal differences are then visible and not misread as contradictions
- C) Only sources from the last 12 months should be cited
- D) The synthesis agent should flag any source older than 6 months

---

**Q16.** A coordinator agent accumulates 80,000 tokens of research findings, then spawns a synthesis subagent and passes its entire context to the subagent. Users report 40+ second response times and doubled API costs. What is the correct fix?

- A) Upgrade to a model with a larger context window
- B) The coordinator should synthesize directly (it already has everything) or pass only condensed structured finding objects — not its full context
- C) Split the synthesis into 10 separate synthesis subagents
- D) Increase the coordinator's max_tokens budget to handle the large context

---

**Q17.** A state manifest for a crashed agent recovery includes: `"completed_steps"`, `"pending_steps"`, and `"key_findings"`. But after restarting, the agent attempts actions outside its authorized scope. What was missing from the manifest?

- A) The manifest should include the full conversation history
- B) The manifest should include `session_facts` — the immutable scope and authorization constraints that govern the entire session
- C) The manifest should include the original task description
- D) The manifest needs a timestamp field

---

**Q18.** In a multi-agent system, one of four parallel subagents encounters a permission error. The coordinator immediately terminates all four subagents. What is wrong with this approach?

- A) Permission errors are always recoverable — the coordinator should retry automatically
- B) The three successful subagents' work is unnecessarily discarded — log the error, continue processing independent subagents, and include partial failure in the final output
- C) The coordinator should wait for all subagents to complete before checking for errors
- D) A permission error on one subagent indicates a systemic issue requiring all subagents to stop

---

**Q19.** What is the correct content for a structured handoff summary when escalating a customer support case to a human agent?

- A) The full conversation transcript copy-pasted
- B) Customer ID, root cause, exact refund amount, reason for escalation (which rule triggered it), and recommended action
- C) A brief note: "Customer needs help with a refund issue"
- D) The system prompt used by the AI agent

---

**Q20.** A research coordinator receives findings from 4 subagents and detects that subagent 3's output conflicts with subagent 1's on the same metric. How should the coordinator handle this?

- A) Discard subagent 3's finding — it likely has lower quality data
- B) Average the two values to produce a balanced estimate
- C) Use the most recent finding since it reflects the latest information
- D) Annotate the conflict: include both values with their sources, set `conflict: true`, and route to human resolution or surface in the final report with both attributions visible

---

## 20. Answer Key & Explanations

| Q | Answer | Key Reason |
|---|---|---|
| 1 | **B** | Prose summarization compresses exact amounts. CASE_FACTS block (structured JSON outside summarized history) preserves precision |
| 2 | **C** | Explicit customer request = immediate escalation, no investigation, no delay |
| 3 | **B** | Plain text loses source attribution. Subagents must return structured claim-source mappings |
| 4 | **C** | Silently picking one value hides the conflict. Annotate both with sources; flag conflict |
| 5 | **B** | Sentiment analysis = probabilistic trigger. Use deterministic triggers: explicit request, threshold, retry count |
| 6 | **B** | No state manifest = restart from zero. Checkpoints + manifest injection = correct recovery |
| 7 | **C** | PostToolUse hook trimming fixes at the tool layer. Model upgrade/limits don't solve the root cause |
| 8 | **B** | Aggregate accuracy masks subgroup failures. Must segment by document type AND field |
| 9 | **D** | Amount > threshold = programmatic/deterministic = reliable. Sentiment, confidence scores = unreliable |
| 10 | **B** | Exact values must be in structured CASE_FACTS outside prose. Prose summarization always loses precision |
| 11 | **B** | Error response missing error_category, is_retryable, attempted_query, partial_results — coordinator can't recover intelligently |
| 12 | **B** | Heuristic selection = wrong customer risk. Always request additional identifiers |
| 13 | **B** | /compact is Claude Code only. API equivalent: manually summarize and replace old turns |
| 14 | **B** | Random sampling undersamples low-confidence cases. Stratified sampling preserves each stratum |
| 15 | **B** | Missing publication dates make temporal differences look like contradictions. Always require dates |
| 16 | **B** | Passing full 80K-token context to synthesis subagent is redundant and expensive. Coordinator synthesizes directly or passes only condensed objects |
| 17 | **B** | session_facts (scope, authorization) missing → agent operates outside authorized boundaries |
| 18 | **B** | Other subagents' work is independent and should continue. Log error, include partial results |
| 19 | **B** | Handoff summary needs: customer ID, root cause, exact amount, escalation reason, recommended action |
| 20 | **D** | Conflicting sources must be annotated with both values + sources. Never silently resolve |

---

## Quick Cheat Sheet — Domain 5

```
META-RULE (D5)
  → Every wrong answer is linguistic (tell Claude to behave better)
  → Every correct answer is structural (inject it, trim it, annotate it, checkpoint it)

CASE_FACTS BLOCK
  → Structured JSON injected at TOP of every user message — never through summarization
  → Contents: all IDs, exact amounts, specific dates, counts, thresholds
  → Always fetched from source system — never from agent's prior history
  → Two-layer: CASE_FACTS (exact) + prose summary (narrative)

PROGRESSIVE SUMMARIZATION
  → SAFE for: narrative, tone, event sequence
  → UNSAFE for: any number, ID, date, percentage, threshold
  → Fix: extract unsafe values into CASE_FACTS block BEFORE summarizing

LOST-IN-THE-MIDDLE
  → Accuracy: 90%+ at position 1 → 40-50% at position 50 → recovery at end
  → Fix: position (system prompt or top of user message) — NOT bold/caps/emphasis
  → Bold/caps = zero effect on transformer attention weights

CONTEXT WINDOWING
  → Act at 70-80% — not 100% (truncation at 100% is from the middle = worst zone)
  → /compact = Claude Code only; API equivalent = manual summarization of old turns
  → Subagent delegation: main agent keeps coordination context; fresh window for subtask

ERROR PROPAGATION
  → Required fields: error_category + is_retryable + attempted_query + partial_results
  → is_retryable: false → NEVER retry (validation/permission/business)
  → Partial results: always return; annotate as partial; never discard
  → Never terminate all subagents for one failure — continue independent work
  → Silent empty = always wrong; always return structured error

ESCALATION
  → Reliable: explicit customer request | amount > threshold | retry ≥ N | irreversible | policy gap
  → Unreliable: sentiment analysis | model confidence score | "complex case"
  → Explicit request = IMMEDIATE escalation; no delay; no investigation first
  → Handoff: customer ID + root cause + exact amounts + escalation reason + recommendation

INFORMATION PROVENANCE
  → Every claim needs: claim + evidence_excerpt + source_url + source_title + publication_date + retrieved_at
  → Missing publication_date = temporal differences misread as contradictions
  → Conflicting sources: annotate BOTH with attribution — never silently pick one

HUMAN REVIEW ROUTING
  → Field-level confidence > overall confidence for routing decisions
  → Stratified sampling: sample from each confidence stratum — random undersamples low-confidence
  → Calibrate confidence against labeled validation set before trusting
  → 97% overall accuracy: segment by document type AND field segment before automation

CRASH RECOVERY
  → State manifest: completed_steps + pending_steps + key_findings + session_facts
  → Write at checkpoints (every N steps or M minutes)
  → On resume: inject manifest into fresh session

TOKEN BUDGET
  → PostToolUse trimming: 40 fields → 5 relevant; fixes at tool layer
  → Trim verbose tool outputs before Claude sees them
  → Large coordinator context → synthesize directly; pass condensed objects only

CALM FRAMEWORK (high-priority D5 named concept)
  → Context-Aware LLM Management — Anthropic's named D5 framework
  → 5 principles: Curate | Anchor | Layer | Manage | Minimise
  → Standard CALM strategy: conversation compaction at 70-80% limit
  → Compaction: summarise completed turns into dense state block; keep CASE_FACTS; CONTINUE
  → Not compaction: clear history (loses state) | return error | increase max_tokens (wrong lever)

CONVERSATION COMPACTION
  → Compact = summarise completed + keep CASE_FACTS + continue execution (preserves state)
  → Clear = wipe everything (destroys accumulated work) — ALWAYS WRONG during active task
  → max_tokens controls OUTPUT length; has zero effect on INPUT context window size

RAG IN D5
  → Semantic search: conceptual similarity | BM25: exact keyword matching | Hybrid: both (production default)
  → RAG chunks must carry: source_url + source_title + publication_date (provenance)
  → Progressive disclosure (JIT): load reference docs at query time; not session start
  → Front-loading all docs = context rot + attention dilution

MULTI-TURN DESIGN
  → State anchor every 5 turns: CASE_FACTS refresh + checkpoint
  → CASE_FACTS at top (high attention) | current question at end (recency) | context in middle
  → Compact proactively at 70-80% — not reactively at 95%

PROMPT CACHING IN D5
  → D5 angle: which content to cache vs compact vs inject fresh
  → Cache: system prompt + tool definitions (stable, immutable)
  → Compact: growing message history (changes every turn)
  → Inject fresh: CASE_FACTS (transactional — from source system, not cached)
  → cache_control breakpoints: separate sections of system prompt for granular caching
```

---

## Bonus Practice Question — CALM Framework

**Q21.** An orchestrator agent has been running for 20 turns and is now approaching the context window limit. It still has 3 more tool calls to complete the task. What is the standard CALM strategy?

- A) Clear the entire conversation history and restart from the original system prompt
- B) Summarise completed steps into a dense state block, keep the CASE_FACTS block intact, and continue execution from the compact starting point
- C) Increase `max_tokens` to extend the available context window
- D) Return a context-limit error to the user and ask them to restart the task

**Answer: B** — Conversation compaction is the standard CALM strategy. It preserves accumulated task state while freeing context space. Clearing history (A) discards all accumulated work. `max_tokens` (C) controls output length, not input context size. Returning an error (D) is premature when compaction makes continuation possible.

---

*CCA-F Domain 5 Study Guide | Prepared for Arun | May 2026*
*All 5 domains now complete: D1 (27%) → D2 (18%) → D3 (20%) → D4 (20%) → D5 (15%)*