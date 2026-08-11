# CCA-F Exam Prep — Domain 2: Tool Design & MCP Integration
> **Exam Weight: 18% — ~11 Questions**
> The domain most candidates underestimate. The exam presents broken or suboptimal tool schemas and asks you to fix them. You must know MCP architecture cold.

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
2. [The Master Mental Model — Tool Descriptions as the Routing Layer](#2-the-master-mental-model--tool-descriptions-as-the-routing-layer)
3. [Anatomy of an Effective Tool Description](#3-anatomy-of-an-effective-tool-description)
4. [Tool Splitting — When One Tool Should Become Three](#4-tool-splitting--when-one-tool-should-become-three)
5. [MCP Architecture — The Three Primitives](#5-mcp-architecture--the-three-primitives)
6. [MCP Transport Layers — stdio vs SSE vs Streamable HTTP](#6-mcp-transport-layers--stdio-vs-sse-vs-streamable-http)
7. [MCP Configuration — Project vs User Level](#7-mcp-configuration--project-vs-user-level)
8. [Structured Error Responses](#8-structured-error-responses)
9. [Empty Result vs Error — The Critical Distinction](#9-empty-result-vs-error--the-critical-distinction)
10. [Tool Scoping — Least Privilege for Tools](#10-tool-scoping--least-privilege-for-tools)
11. [task_scoped Tool Profiles](#11-task-scoped-tool-profiles)
12. [tool_choice Configuration](#12-tool_choice-configuration)
13. [strict: true — Schema Validation](#13-strict-true--schema-validation)
14. [Built-in Tools — Grep vs Glob vs Read vs Edit vs Write vs Bash](#14-built-in-tools--grep-vs-glob-vs-read-vs-edit-vs-write-vs-bash)
15. [MCP Resources as Content Catalogs](#15-mcp-resources-as-content-catalogs)
16. [System Prompt Keyword Conflicts](#16-system-prompt-keyword-conflicts)
17. [Authentication Patterns for MCP](#17-authentication-patterns-for-mcp)
17b. [PostToolUse for Tool Output Trimming](#17b-posttooluse-for-tool-output-trimming)
18. [Anti-Patterns Master List](#18-anti-patterns-master-list)
19. [Key Rules to Memorize](#19-key-rules-to-memorize)
20. [Practice Questions (20 MCQs)](#20-practice-questions-20-mcqs)
21. [Answer Key & Explanations](#21-answer-key--explanations)

---

## 1. What This Domain Tests

| Task Statement | Description |
|---|---|
| 2.1 | Design effective tool interfaces with clear descriptions and boundaries |
| 2.2 | Implement structured error responses for MCP tools |
| 2.3 | Distribute tools appropriately across agents and configure tool choice |
| 2.4 | Configure MCP servers at the correct scope (project vs user) with secure credential handling |
| 2.5 | Select and apply built-in tools (Read, Write, Edit, Bash, Grep, Glob) effectively |

### Domain 2 in the Exam Scenarios

| Scenario | Domain 2 Focus |
|---|---|
| Customer Support Agent | Scoped tool access, structured error responses |
| Multi-Agent Research System | Tool description disambiguation, coordinator tool profiles |
| Developer Productivity Tools | Built-in tool selection (Grep/Glob/Read/Edit), MCP server integration |
| Enterprise Integration Platform | MCP server config, transport selection, authentication |

---

## 2. The Master Mental Model — Tool Descriptions as the Routing Layer

> **The single most tested concept in Domain 2.**

Claude has **no internal routing logic beyond tool descriptions**. When Claude decides which tool to call, it reads each tool's description and picks the closest match to what it needs. This means:

```
Minimal descriptions    → misrouting (Claude guesses)
Overlapping descriptions → unpredictable selection (random between similar tools)
Well-differentiated     → reliable routing (Claude selects correctly every time)
```

### The Exam Pattern

The exam will show you a scenario where an agent is calling the wrong tool — e.g., calling `get_customer` for order queries 45% of the time. The wrong answer options will be:
- Add a routing classifier
- Merge the two tools into one
- Upgrade to a more powerful model

**The correct answer is always: improve the tool descriptions first.** Fix the root cause before adding infrastructure.

---

## 3. Anatomy of an Effective Tool Description

Every tool description must answer **5 questions** — and the official exam guide adds two more specifics: **example queries** and **edge cases**:

1. **What does it do** — specifically, not generically
2. **What input does it take** — format, type, constraints, examples
3. **What does it return** — fields, types, what "empty" looks like
4. **When to use it** — the exact trigger condition, with example queries
5. **When NOT to use it** — disambiguation from similar tools (boundary explanations)
6. **Edge cases** — e.g., "Returns empty array if no records found — this is NOT an error"

> **Official guide wording (Task 2.1):** "The importance of including input formats, **example queries, edge cases, and boundary explanations** in tool descriptions."

### Bad Descriptions (Cause Misrouting)

```python
# ❌ WRONG — identical descriptions, Claude cannot distinguish
tools = [
    {"name": "get_customer",  "description": "Gets customer information"},
    {"name": "lookup_order",  "description": "Gets order information"},
]
# Result: Claude calls get_customer for order queries 45% of the time
```

### Good Descriptions (Route Correctly Every Time)

```python
# ✅ CORRECT — each description is fully differentiated
tools = [
    {
        "name": "get_customer",
        "description": """Retrieve a customer's profile, verification status, account tier,
        and contact information from the customer database.

        INPUT: customer_id (string, format: CUS-XXXXXXXX) OR email (string).
        At least one identifier required.

        RETURNS: {customer_id, name, email, tier, verified, open_cases, account_since}.
        Returns empty content with isError:false if no customer matches — this is NOT an error.

        USE THIS for: customer identity, verification status, account tier, contact details.
        DO NOT USE for: order history or transaction data — use lookup_order instead."""
    },
    {
        "name": "lookup_order",
        "description": """Retrieve a specific order's details, status, items, and shipping
        information from the orders database.

        INPUT: order_id (string, format: ORD-XXXXXXXX) OR customer_id to list recent orders.

        RETURNS: {order_id, customer_id, status, items[], total, shipping, created_at}.
        Returns empty array if no orders found for the customer — this is a valid result.

        USE THIS for: order status, item details, shipping tracking, refund eligibility.
        DO NOT USE for: customer profile or verification — use get_customer instead."""
    }
]
```

### The 6-Element Checklist for Every Tool Description (Official Guide)

| ✓ | Element | Example |
|---|---|---|
| ☐ | **Specific action** | "Searches internal knowledge base for policy documents" |
| ☐ | **Input format** | "Input: customer_id (string, format: CUS-XXXXXXXX)" |
| ☐ | **Return structure** | "Returns: {order_id, status, items[], total}" |
| ☐ | **Trigger condition + example queries** | "Use when: customer asks about order status. E.g. 'where is my order', 'track my shipment'" |
| ☐ | **Boundary/disambiguation** | "DO NOT USE for customer profile — use get_customer" |
| ☐ | **Edge cases** | "Returns empty array if no orders found — this is NOT an error, do not retry" |

---

## 4. Tool Splitting — When One Tool Should Become Three

### Signs a Tool Needs to Be Split

- It takes a `type` parameter that changes its behavior entirely
- Its description has multiple "if X, then Y" branches
- Misrouting frequency is above ~5%
- Different agent roles need different subsets of its functionality

### The Split Pattern

```python
# ❌ WRONG — overloaded tool with type parameter
{
    "name": "analyze_document",
    "description": "Analyzes a document. If type='web', extracts web content.
                    If type='pdf', extracts PDF sections. If type='db', queries database."
}
# Claude can't reliably choose the right type — descriptions are internal to the tool

# ✅ CORRECT — three focused tools, each unambiguous
{
    "name": "extract_web_content",
    "description": """Extracts structured data from web search results or fetched pages.
    Input: raw HTML or URL. Returns: {title, url, summary, published_date}.
    Use for web content only. DO NOT USE for PDFs or database queries."""
},
{
    "name": "extract_pdf_sections",
    "description": """Extracts text sections and metadata from PDF documents.
    Input: PDF file path or bytes. Returns: {sections[], page_count, metadata}.
    Use for PDF files only. DO NOT USE for web content."""
},
{
    "name": "query_database_record",
    "description": """Queries the structured customer/transaction database by ID.
    Input: {record_id: string}. Returns typed fields per schema.
    Use when you have a specific ID to look up. DO NOT USE for documents."""
}
```

**Exam trap:** The wrong answer will suggest "add a `type` parameter so the tool can handle all three cases." The correct answer is always to split into focused, unambiguous tools.

### Official Exam Split-Tool Names (from the Exam Guide)

The official exam guide uses these specific names as the canonical example of splitting a generic tool. Recognize them if they appear:

```
Generic tool:        analyze_document
Split into:
  → extract_data_points      — extracts specific data fields from a document
  → summarize_content        — produces a narrative summary
  → verify_claim_against_source — checks a claim against source material

Also:
  analyze_content  →  extract_web_results  (renamed with web-specific description)
```

**Exam rule:** When you see `analyze_document`, `extract_data_points`, `summarize_content`, or `verify_claim_against_source` in a question, you're looking at a tool-splitting scenario.

---

## 5. MCP Architecture — The Three Primitives

The Model Context Protocol defines three primitives. **Know all three — the exam tests them directly.**

| Primitive | What It Is | Agent Interaction | Example |
|---|---|---|---|
| **Tools** | Callable actions that affect external systems | Agent invokes → side effects possible | `create_task`, `send_email`, `query_db` |
| **Resources** | Read-only structured data sources | Agent reads → no side effects | Issue summaries, DB schemas, document catalogs |
| **Prompts** | Reusable prompt templates exposed by the server | Agent retrieves → ensures consistent behavior | Standard analysis prompt, review template |

### The Primitive Decision Rule

```
Agent needs to DO something (write, create, delete, send) → Tool
Agent needs to READ something (documents, schemas, catalogs)  → Resource
Agent needs a consistent behavior template               → Prompt
```

### MCP Architecture Components

```
┌─────────────────────────────────────────────────┐
│  HOST (e.g., Claude Desktop, Claude Code, App)  │
│  ┌─────────┐     ┌─────────┐                    │
│  │ Client  │     │ Client  │  ← one per server  │
│  └────┬────┘     └────┬────┘                    │
└───────┼───────────────┼─────────────────────────┘
        │               │  JSON-RPC 2.0
   ┌────▼────┐     ┌────▼────────────┐
   │  MCP    │     │  MCP Server B   │
   │ Server A│     │ (remote, SSE)   │
   │ (local, │     │                 │
   │  stdio) │     └─────────────────┘
   └─────────┘
```

**Key facts:**
- One MCP Client per connected server
- Communication: JSON-RPC 2.0 over the transport layer
- Servers can be local (subprocess) or remote (network)

---

## 6. MCP Transport Layers — stdio vs SSE vs Streamable HTTP

> ⭐ **This is directly exam-tested. The SSE question appears in official sample questions.**

### The Three Transports

| Transport | Deployment | Network Crossing | Multi-Client | When to Use |
|---|---|---|---|---|
| **stdio** | Local only | ❌ No | ❌ Single client | Local tools on the same machine as Claude |
| **SSE** | Remote | ✅ Yes | ✅ Yes | Legacy remote servers; being deprecated |
| **Streamable HTTP** | Remote | ✅ Yes | ✅ Yes | ✅ **Recommended for new remote servers** |

### The Decision Rule

```
Server runs on the SAME machine as Claude?  → stdio
Server is REMOTE / cloud-hosted?            → Streamable HTTP (new) or SSE (legacy)
Need OAuth authentication?                  → Streamable HTTP (native support)
WebSocket?                                  → NOT a supported MCP transport — WRONG ANSWER
```

### The Exam Question Pattern

**Scenario:** "You are building an MCP server that streams file contents to Claude during a long-running analysis task in a cloud environment. Which transport?"

```
A) stdio — lower latency for subprocess communication
B) SSE — supports real-time streaming over HTTP across network boundaries  ← CORRECT
C) WebSocket — provides bidirectional streaming
D) stdio — simpler to implement
```

**Why B:** SSE crosses network boundaries. stdio is local-only — it cannot reach a cloud deployment. WebSocket is NOT a supported MCP transport.

> **Note on deprecation:** SSE is technically being phased out in favor of Streamable HTTP, but the exam uses "SSE" as the remote transport answer. For exam purposes: **local = stdio, remote = SSE**.

---

## 7. MCP Configuration — Project vs User Level

This is tested directly in the exam. Know which file goes where and why.

| Level | File | Scope | Committed? | Use For |
|---|---|---|---|---|
| **Project** | `.mcp.json` (project root) | All team members who clone the repo | ✅ Yes | Shared team MCP servers (Jira, Slack, internal APIs) |
| **User/Personal** | `~/.claude.json` | This machine only | ❌ No | Personal tools, experimental servers |

### The Critical Secrets Rule

**Never commit secrets.** Use environment variable substitution:

```json
// ✅ CORRECT — .mcp.json committed to version control
{
  "mcpServers": {
    "jira": {
      "command": "npx",
      "args": ["-y", "@anthropic/jira-mcp"],
      "env": {
        "JIRA_API_TOKEN": "${JIRA_API_TOKEN}",
        "JIRA_BASE_URL": "${JIRA_BASE_URL}"
      }
    }
  }
}

// ❌ WRONG — secret value hardcoded, gets committed to git
{
  "JIRA_API_TOKEN": "sk-jira-actual-token-here-abc123"
}
```

### The Exam Trap

**Scenario:** "A new developer joins the team. They clone the repo but Claude Code doesn't follow the project's established MCP conventions."

**Wrong answers:** Add to `.claude/settings.local.json` | Add to `~/.claude.json` | Tell the developer to add it manually.

**Correct answer:** The MCP server configuration belongs in `.mcp.json` at project level, committed to version control — so every developer gets it automatically on clone.

### Community vs Custom MCP Servers

| Choice | When |
|---|---|
| Use existing **community MCP server** | Standard integrations (Jira, GitHub, Slack, Notion) — battle-tested, maintained |
| Build **custom MCP server** | Team-specific workflows with proprietary APIs not covered by community servers |

**Exam rule:** Prefer community MCP servers for standard tools. Reserve custom server development for truly team-specific needs. Don't build what already exists.

### Tools From All MCP Servers Are Available Simultaneously

> ⭐ **Official PDF Knowledge item under Task 2.4 — directly tested**

A key fact the exam tests: when multiple MCP servers are configured, tools from **all of them** are discovered at connection time and available to the agent simultaneously. There is no manual loading or activation step per server.

```
# Two MCP servers configured in .mcp.json:
#   - github-mcp   (tools: create_pr, review_code, list_issues)
#   - jira-mcp     (tools: create_ticket, update_status, list_sprints)
#
# The agent sees ALL six tools immediately — no extra steps needed.
# Claude can call create_pr and create_ticket in the same conversation.
```

**Exam trap:** A question may describe an agent that "can only use one MCP server at a time." This is wrong — tools from all configured servers are simultaneously available. The agent routes between them using tool descriptions just like any other tools.

**Why this matters:** It means description disambiguation becomes even more critical when you have tools from multiple servers with overlapping functionality (e.g., both GitHub and Jira have "create issue" style tools). Each tool needs a distinct description even across servers.

---

## 8. Structured Error Responses

> ⭐ **The exact field names are exam-tested. Memorize the schema.**

Every MCP tool failure must return a structured error object — not a plain string. The coordinator needs these fields to make an intelligent recovery decision.

### The Required Schema

```typescript
interface MCPErrorResponse {
  isError: true;
  errorCategory: 'transient' | 'validation' | 'business' | 'permission';
  isRetryable: boolean;
  message: string;          // Specific, human-readable description
  retryAfterMs?: number;    // Optional: hint for retry timing
}
```

### The Four Error Categories — Know Every One

| Category | What It Means | `isRetryable` | Agent Action |
|---|---|---|---|
| `transient` | Temporary failure — DB timeout, service down | `true` | Retry with exponential backoff |
| `transient` (rate limit) | Rate limit hit — too many requests | `true` | Retry **only after backoff delay** — not immediately |
| `validation` | Bad input — wrong format, missing required field | `false` | Fix input, do NOT retry |
| `business` | Policy violation — refund exceeds limit, not allowed | `false` | Escalate to human or communicate to user with **customer-friendly explanation** |
| `permission` | Access denied — no auth, wrong scope, expired token | `false` | Rotate credentials or escalate |

> **Rate limit nuance (exam-tested):** Rate limit errors are `transient` and `isRetryable: true` — but the agent must wait for the backoff delay before retrying. Immediate retry on a rate limit will reproduce the same error. This is distinct from a network timeout, which may be retried more quickly.

> **Business error nuance (exam-tested):** Business rule violations require a **customer-friendly explanation** in the message — not just an internal flag. The agent needs to relay the reason to the user (e.g., "Your refund of $750 exceeds our $500 automated limit. A support agent will follow up within 24 hours."). A generic "Policy violation" message is insufficient.

### Code Examples

```python
# ✅ CORRECT — structured error enables intelligent coordinator recovery
def get_customer(customer_id: str) -> dict:
    try:
        result = db.query(customer_id)
        if result is None:
            # NOT an error — valid query, no matching record
            return {"isError": False, "content": [], "message": f"No customer found: {customer_id}"}
        return {"isError": False, "content": [result]}

    except DatabaseTimeout:
        return {
            "isError": True,
            "errorCategory": "transient",
            "isRetryable": True,
            "message": "Database connection timeout. Retry appropriate.",
            "retryAfterMs": 2000
        }
    except PermissionDenied:
        return {
            "isError": True,
            "errorCategory": "permission",
            "isRetryable": False,
            "message": "Agent lacks permission to access customer records."
        }
    except InvalidInput as e:
        return {
            "isError": True,
            "errorCategory": "validation",
            "isRetryable": False,
            "message": f"Invalid customer_id format: {e}. Expected CUS-XXXXXXXX."
        }
    except RefundLimitExceeded:
        return {
            "isError": True,
            "errorCategory": "business",
            "isRetryable": False,
            "message": "Refund of $750 exceeds the $500 autonomous limit. Escalate to human."
        }
```

### Agent-Side Error Handling

```python
def handle_tool_result(tool_name: str, result: dict) -> None:
    if not result.get("isError"):
        process_content(result.get("content", []))  # empty is fine
        return

    if result.get("isRetryable") and retry_count < MAX_RETRIES:
        wait_ms = result.get("retryAfterMs", 1000 * (2 ** retry_count))
        schedule_retry(tool_name, wait_ms)
    elif result["errorCategory"] == "business":
        escalate_to_human(reason=result["message"])
    elif result["errorCategory"] == "permission":
        alert_operations(f"Permission error on {tool_name}")
    else:
        propagate_structured_error_to_coordinator(result)
```

### What the Exam Tests

- "A tool returns `'Operation failed'` when it times out. What is wrong?"
  → **Wrong:** No `errorCategory`, no `isRetryable` flag — coordinator cannot make a recovery decision.
- "How should the coordinator respond to `isRetryable: false`?"
  → **Never retry** — fix the input, escalate, or rotate credentials based on category.

### Local Recovery Before Propagation — The Correct Subagent Pattern

> ⭐ **From the official exam guide subdomain 2.2 — tested directly**

Subagents should attempt **local recovery** for transient failures before escalating to the coordinator. They should only propagate errors they genuinely cannot resolve locally — and when they do, they must include what was attempted and any partial results.

```python
# ✅ CORRECT — subagent attempts local recovery first
async def web_search_subagent(query: str) -> dict:
    for attempt in range(3):          # local retry for transient errors
        try:
            return await search(query)
        except ServiceTimeout:
            if attempt == 2:
                # Exhausted local retries — propagate structured error to coordinator
                return {
                    "isError": True,
                    "errorCategory": "transient",
                    "isRetryable": True,
                    "attempted": f"Searched for: {query} (3 attempts)",
                    "partial_results": accumulated_results_so_far,
                    "message": "Search timed out after 3 attempts. Coordinator may retry."
                }
            await asyncio.sleep(2 ** attempt)  # exponential backoff

# ❌ WRONG — immediately escalates every error to coordinator
async def web_search_subagent_bad(query: str) -> dict:
    try:
        return await search(query)
    except ServiceTimeout:
        return {"isError": True, "message": "Failed"}  # no local retry, no detail
```

**The two-tier error handling rule:**
1. **Subagent level:** Handle transient errors locally with retries + backoff
2. **Coordinator level:** Only receive errors that couldn't be resolved locally, with full context (what was attempted, partial results)

---

## 9. Empty Result vs Error — The Critical Distinction

> ⭐ **The most common source of infinite retry loops — directly tested.**

```json
// ✅ Successful query, no records found — NOT an error
// isError: false — agent accepts this and moves on
{
  "isError": false,
  "content": [],
  "message": "No orders found for customer CUS-48291 in the last 90 days"
}

// ❌ Tool execution failed — IS an error
// isError: true — agent must handle the failure
{
  "isError": true,
  "errorCategory": "transient",
  "isRetryable": true,
  "message": "Database connection timeout after 30 seconds"
}
```

### Why This Distinction Matters

| Mistake | Consequence |
|---|---|
| Treat **empty results as errors** | Agent retries a successful query endlessly — infinite loop |
| Treat **errors as empty results** | Agent silently misses data — produces incorrect output without warning |

**Exam rule:** Empty content with `isError: false` = success. `isError: true` = actual failure. These are mutually exclusive and must never be confused.

---

## 10. Tool Scoping — Least Privilege for Tools

> Same principle as Domain 1's minimal footprint — applied specifically to tools.

**Why fewer tools = better selection:**
- With 4–5 tools, each description is distinct and salient to Claude
- With 18+ tools, descriptions compete and selection reliability degrades
- Out-of-role tools get misused (synthesis agent with web search will sometimes search instead of synthesize)

**The 4–5 tool maximum rule:**
```
Research coordinator:    ["Task", "search_web"]
Web search subagent:     ["search_web", "fetch_page"]
Document analysis agent: ["read_pdf", "extract_tables", "search_document_text"]
Synthesis agent:         ["format_citations", "check_consistency"]  ← no web search!
Report agent:            ["format_report", "write_file"]
```

**Exam trap:** A question describes an agent that misuses tools (synthesis agent querying the web). The wrong answer is to add instructions telling it not to. The correct answer is to remove the tool from its `allowedTools` list — deterministic enforcement, not probabilistic instruction.

### Scoped Cross-Role Tools — The Exception

Occasionally a secondary role needs limited access to a tool from another specialization. The pattern is to provide a **constrained version** of the tool rather than the full version:

```python
# ❌ WRONG — give synthesis agent full web search access
synthesis_agent_tools = ["format_citations", "check_consistency", "search_web"]
# Synthesis agent starts searching when it should be synthesizing

# ✅ CORRECT — provide a constrained alternative for high-frequency cross-role needs
synthesis_agent_tools = [
    "format_citations",
    "check_consistency",
    "verify_fact"          # ← constrained tool: only verifies a specific claim
                           #   against a known source; cannot do open-ended search
]
# Complex searches still route through the coordinator
```

**The pattern:**
- Give the agent a **constrained tool** (e.g., `verify_fact`) for the occasional cross-role need
- Route all **complex cross-role requests** through the coordinator
- Never give the full original tool (e.g., `search_web`) to an out-of-role agent

**Another example — replacing generic tools with constrained alternatives:**
```python
# ❌ fetch_url — agent can fetch any URL including external ones
# ✅ load_document — validates that URLs are internal documentation only
# Constraining the tool prevents out-of-scope behavior without relying on prompts

# ❌ fetch_url — can access any external URL
# ✅ load_internal_docs — validates URLs against internal documentation domain
#    before fetching; rejects requests to external domains
# This is the exam's canonical example of a scope-constraining replacement tool
```

---

## 11. Task-Scoped Tool Profiles

For agents that change roles across phases, define **per-phase tool sets** rather than granting all tools upfront.

```python
# Exploration phase — read only, no write access
exploration_options = ClaudeAgentOptions(
    allowed_tools=["Read", "Glob", "Grep"],
    permission_mode="default"
)

# Development phase — add write capabilities
development_options = ClaudeAgentOptions(
    allowed_tools=["Read", "Glob", "Grep", "Write", "Bash"],
    permission_mode="acceptEdits"
)

# Deployment phase — add deployment tools
deployment_options = ClaudeAgentOptions(
    allowed_tools=["Read", "Glob", "Grep", "Write", "Bash",
                   "run_tests", "deploy_staging", "notify_team"],
    permission_mode="acceptEdits"
)
```

**Why:** An exploration agent with Write access might modify files when it should only be reading. Giving Write only during the development phase prevents accidental writes during exploration.

**Exam pattern:** A scenario describes an agent that accidentally deletes or modifies files during a read-only exploration task. The fix is always a task-scoped tool profile — not a prompt instruction saying "don't write files."

---

## 12. tool_choice Configuration

| Value | Behavior | When to Use |
|---|---|---|
| `{"type": "auto"}` | Claude decides whether to use any tool | Default — most conversations |
| `{"type": "any"}` | Claude must call at least one tool | When you need guaranteed structured output |
| `{"type": "tool", "name": "X"}` | Claude must call this specific named tool | Fixed pipeline step that always uses the same tool |
| `{"type": "none"}` | Claude cannot use any tools | Text-only response needed despite tool definitions |
| Add `"disable_parallel_tool_use": true` | Forces one tool call per response | Step-by-step sequential tool execution |

### Forced tool_choice for Pipeline Steps

```python
# ✅ Use forced tool_choice when a pipeline step always needs a specific tool
extraction_response = client.messages.create(
    model="claude-sonnet-4-6",
    tools=[extraction_tool],
    tool_choice={"type": "tool", "name": "extract_invoice_data"},
    messages=[{"role": "user", "content": invoice_text}]
)
# Claude MUST call extract_invoice_data — no ambiguity, no chance of skipping
```

### Forced tool_choice to Sequence Steps

Use forced tool_choice to **guarantee a specific tool fires first**, then let subsequent steps proceed normally in follow-up turns:

```python
# Step 1 — Force metadata extraction first (guaranteed ordering)
step1 = client.messages.create(
    tool_choice={"type": "tool", "name": "extract_metadata"},
    tools=[extract_metadata, enrich_data, validate_record],
    messages=[{"role": "user", "content": record}]
)

# Step 2 — Now let Claude decide which enrichment/validation tool to call
step2 = client.messages.create(
    tool_choice={"type": "auto"},  # Claude picks from remaining tools
    tools=[enrich_data, validate_record],
    messages=[...step1_history...]
)
```

**Exam rule:** When you need a specific tool to always run before others in a pipeline, force it in the first turn, then use `"auto"` for follow-up steps. This is deterministic ordering without using hooks.

**Exam trap:** `"any"` forces Claude to use a tool but lets it choose which one. `"tool"` forces a specific tool. These are NOT the same.

---

## 13. strict: true — Schema Validation

Adding `strict: true` to a tool definition enables **Structured Outputs mode** — guaranteeing Claude's tool call input always matches your JSON schema exactly.

```python
tools = [{
    "name": "classify_ticket",
    "description": "Classify a support ticket by category and priority",
    "input_schema": {
        "type": "object",
        "properties": {
            "category": {
                "type": "string",
                "enum": ["billing", "technical", "account", "other"]
            },
            "priority": {
                "type": "integer",
                "minimum": 1,
                "maximum": 5
            },
            "summary": {"type": "string"}
        },
        "required": ["category", "priority"]  # summary is optional
    },
    "strict": True   # ← guarantees schema compliance on Claude's tool call
}]
```

### What strict: true Guarantees vs Doesn't

| Guarantees | Does NOT Guarantee |
|---|---|
| Tool call input always matches the schema | Semantic correctness (e.g., line items summing to total) |
| No missing required fields | Values being factually accurate |
| No invalid enum values | The right tool being called |
| No type mismatches | Sensible values for optional fields |

**The exam rule:** `strict: true` eliminates syntax errors in tool calls. It does NOT eliminate semantic errors.

### Schema Design for Nullable Fields

```python
# ❌ WRONG — required field with no guaranteed source data
# Forces Claude to hallucinate a value when data is missing
"required": ["category", "priority", "customer_sentiment"]

# ✅ CORRECT — optional field with null allowed
"customer_sentiment": {
    "type": ["string", "null"],  # can be null when not determinable
    "enum": ["positive", "neutral", "negative", null]
}
# And removed from "required" list
```

**The exam rule:** Mark fields as `required` only when the data is always present. Optional/nullable fields prevent hallucination when source data is absent.

### Enum + "other" + "unclear" — Extensible Categorization

When designing enum fields for classification tools, always include escape-hatch values so Claude doesn't force a bad category fit:

```python
# ❌ WRONG — closed enum forces hallucination when nothing fits
"category": {
    "type": "string",
    "enum": ["billing", "technical", "account"]
}
# If customer asks about "shipping damage" → Claude forces it into a wrong category

# ✅ CORRECT — open enum with escape hatches
"category": {
    "type": "string",
    "enum": ["billing", "technical", "account", "other", "unclear"]
}
# "other" = something else | "unclear" = ambiguous input, needs human review
```

**Exam rule:** Always include `"other"` and/or `"unclear"` in classification enums. A closed enum with no escape hatch forces Claude to hallucinate a category fit when the real answer doesn't match any option.

### `additionalProperties: false` — Strict Schema Boundary

For strict schemas (especially with `strict: true`), add `"additionalProperties": false` to prevent Claude from adding extra fields that aren't in the schema:

```python
"input_schema": {
    "type": "object",
    "properties": {
        "customer_id": {"type": "string"},
        "refund_amount": {"type": "number"}
    },
    "required": ["customer_id", "refund_amount"],
    "additionalProperties": false   # ← Claude cannot add unexpected fields
}
```

**Why it matters:** Without `additionalProperties: false`, Claude can include extra fields in the tool call input that your server doesn't expect, potentially causing downstream errors.

---

## 14. Built-in Tools — Grep vs Glob vs Read vs Edit vs Write vs Bash

> ⭐ **"Grep vs Glob distinction that trips everyone up" — directly called out in exam curriculum.**

Claude Code ships with these built-in tools available in every session. The exam tests when to use each one.

### The Decision Table

| Tool | What It Does | Best For | NOT For |
|---|---|---|---|
| **Grep** | Searches file **contents** for patterns (text, regex) | Finding where a function is defined; finding all uses of a variable; searching for error messages | Finding files by name/path |
| **Glob** | Finds **files by name or path pattern** | Finding all `*.test.tsx` files; finding all files in a directory; listing files matching a pattern | Searching inside file contents |
| **Read** | Opens and reads a **full file** | Reading a specific file you already know; loading config files | Searching across multiple files |
| **Edit** | Makes **targeted in-place modifications** using unique text matching | Replacing a specific line; fixing a specific function without rewriting the whole file | When the target text isn't unique in the file |
| **Write** | Creates or **completely overwrites** a file | Creating new files; overwriting when full rewrite is needed | Small targeted changes (use Edit instead) |
| **Bash** | Runs **shell commands** | Running tests; git operations; any CLI command; compiling | Direct file reading or searching (use Read/Grep instead) |

### The Grep vs Glob Exam Question

**Scenario:** "An agent needs to find all files in the codebase that import from `@/auth`."

```
A) Glob — finds all files matching the pattern **/@/auth**
B) Grep — searches file contents for the import statement
C) Read — reads each file to check for the import
D) Bash — runs a shell find command
```

**Correct answer: B — Grep.** You're searching *inside* file contents for a string pattern. Glob finds files by *name/path*, not by what's inside them.

**Scenario:** "An agent needs to find all TypeScript test files in the project."

```
A) Grep — searches for 'test' in file contents
B) Glob — finds all files matching **/*.test.ts pattern
C) Read — reads each file to check if it's a test
D) Bash — runs find command
```

**Correct answer: B — Glob.** You're finding files by name pattern, not searching their contents.

### When Edit Fails — Fall Back to Read + Write

```python
# Edit uses unique text matching — fails if target text isn't unique
try:
    edit_result = Edit(
        file_path="src/auth.ts",
        old_text="const token = getToken();",
        new_text="const token = await getToken();"
    )
except NonUniqueMatch:
    # Fall back: Read the full file, modify in memory, Write back
    content = Read("src/auth.ts")
    modified = content.replace(...)
    Write("src/auth.ts", modified)
```

### Codebase Exploration Strategy (Incrementally, Not All at Once)

```
Step 1: Grep for entry points → find main.ts, index.ts, app.ts
Step 2: Read those files → understand import structure
Step 3: Glob for specific file types → find all *.service.ts
Step 4: Grep for specific patterns → find all uses of deprecated API
```

**Anti-pattern:** Reading all 50,000 files upfront to "get the full picture." Use Grep + Glob to narrow scope first, then Read selectively.

### Tracing Function Usage Across Wrapper Modules

> ⭐ **Official exam guide skill 2.5 — specific pattern tested**

When a function is re-exported through multiple wrapper modules, finding all callers requires a two-step Grep approach:

```
Step 1: Grep for all exported names
        Grep("export.*authService\|export.*getToken") → finds wrapper.ts, index.ts

Step 2: Grep for each exported name across the codebase
        Grep("authService\|getToken") → finds every caller in every file

vs. ❌ WRONG: Reading all files one by one to find callers
```

**Why this matters:** A function exported as `authService.getToken` might be called as `getToken` after destructuring, or as `auth.getToken` after aliasing. Grep for the function name first, then the wrapper export names — don't assume you know the import path.

### Built-in Tools vs MCP Tools — The Preference Problem

> ⭐ **Exam-tested: agents may prefer built-in tools over custom MCP tools**

Claude Code ships with Grep, Glob, Read, etc. as familiar built-in tools. When you add a custom MCP tool that performs a similar function (e.g., a specialized code search MCP tool), Claude may still default to the built-in Grep because it "knows" Grep from training.

**The problem:**
```
You add an MCP tool: "search_internal_codebase" (specialized, indexed, faster)
Claude keeps using Grep instead — ignoring your better MCP tool
```

**The fix — enhance the MCP tool description to explicitly contrast with built-ins:**
```python
{
    "name": "search_internal_codebase",
    "description": """Search the indexed internal codebase using semantic and exact-match
    search across all repositories.

    PREFER THIS OVER Grep for internal code searches — this tool has pre-built indexes
    that are 10x faster and support fuzzy matching across 50+ repositories simultaneously.
    Use Grep only for single-file pattern matching when you already know the file path.

    INPUT: query string (supports regex and natural language).
    RETURNS: [{file_path, line_number, match_context, repository}]"""
}
```

**Exam rule:** If an agent ignores a custom MCP tool and uses a built-in instead, the fix is to **enhance the MCP tool's description** to explicitly explain why it's better than the built-in alternative — not to remove the built-in tool.

### The Tool-Testing Agent — Anthropic's Production Pattern

> ⭐ **From Anthropic's own engineering blog — the "40% improvement" pattern**

Anthropic's production multi-agent research system used a **tool-testing agent** to iteratively improve MCP tool descriptions:

1. The tool-testing agent is given a flawed MCP tool
2. It attempts to use the tool dozens of times
3. It identifies failure modes and nuances
4. It **rewrites the tool description** to prevent those failures

**Result:** 40% decrease in task completion time for agents using the improved description.

**Exam implication:** Tool description quality is not set once and forgotten — it's an iterative process. The correct approach when tools fail is to analyze failure modes and improve descriptions, not to build routing infrastructure or retrain the model.

---

## 15. MCP Resources as Content Catalogs

**MCP Resources** expose read-only data catalogs to agents — reducing the need for exploratory tool calls.

### What Resources Enable

```
Without MCP Resources:
Agent → (tool call) "what Jira issues exist?" → scans everything → expensive
Agent → (tool call) "what databases are available?" → exploratory tool call needed

With MCP Resources:
Agent → reads resource catalog immediately → knows what exists without a tool call
→ No exploratory scanning needed → leaner, faster, fewer tokens
```

### Example Resource Structure

```json
// MCP Resource: Jira Issues Catalog
{
  "uri": "jira://issues/backlog",
  "name": "Backlog Issues",
  "description": "All open backlog issues with status, assignee, and priority",
  "mimeType": "application/json",
  "content": [
    {"id": "PROJ-123", "title": "Fix auth bug", "status": "open", "priority": "high"},
    {"id": "PROJ-124", "title": "Add dark mode", "status": "open", "priority": "low"}
  ]
}
```

### The Exam Rule

When an agent needs to know **what data is available** (schema, catalog, list of entities) before deciding what to query — use MCP Resources. Resources are read-only, no side effects, and don't count as tool calls that could trigger misrouting.

**Use Resources for:**
- Issue summaries and catalogs
- Database schemas
- Document hierarchies
- Configuration catalogs

**Use Tools for:**
- Creating, updating, or deleting records
- Running queries with specific parameters
- Sending messages or notifications

---

## 16. System Prompt Keyword Conflicts

System prompt phrasing can accidentally create tool selection bias.

```
System prompt: "Always analyze the content from each URL before summarizing."
Tool name:      analyze_content

→ The phrase "analyze content" in the system prompt creates a strong association
  with the analyze_content tool, causing it to fire even when a different tool
  (e.g., fetch_url) is more appropriate.
```

### How to Fix It

```
// ❌ WRONG system prompt — keyword matches tool name
"You must analyze content from each source document."

// ✅ CORRECT — rephrase to avoid keyword collision
"You must examine each source document thoroughly before summarizing."
```

**Exam rule:** If an agent is over-calling a specific tool in situations where it shouldn't be, check the system prompt for phrases that match that tool's name. Rename the tool or rephrase the system prompt to break the association.

### The Audit Checklist

Before deploying:
1. List all tool names and key words in their descriptions
2. Scan the system prompt for phrases that appear in tool names
3. Rephrase any matching system prompt phrases or rename the conflicting tool

---

## 17. Authentication Patterns for MCP

### Environment Variable Pattern (Always Correct)

```json
// .mcp.json — committed to version control safely
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@anthropic/github-mcp"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}",
        "GITHUB_ORG": "${GITHUB_ORG}"
      }
    }
  }
}
```

The `${GITHUB_TOKEN}` syntax is resolved at runtime from the environment — the actual token never appears in the file.

### Transport-Level Auth

| Transport | Native Auth Support |
|---|---|
| stdio | No built-in auth — relies on process environment |
| SSE | HTTP headers; tokens in env vars |
| Streamable HTTP | ✅ **Native OAuth support** — recommended for production remote auth |

**Exam rule:** For production remote MCP servers requiring OAuth — use Streamable HTTP. It's the only transport with native OAuth support.

---

## 17b. PostToolUse for Tool Output Trimming

> ⭐ **Exam-tested: trimming verbose tool outputs before they hit Claude's context**

MCP tools often return far more fields than Claude needs. A `lookup_order` tool might return 40 fields when only 5 are relevant to the current task. Without trimming, every tool call bloats the context window — accelerating context rot.

**The fix: PostToolUse hook that filters tool results to only relevant fields.**

```python
# PostToolUse hook — trim verbose tool output before Claude sees it
def trim_order_result(tool_name: str, tool_result: dict) -> dict:
    if tool_name == "lookup_order" and not tool_result.get("isError"):
        content = tool_result.get("content", {})
        # Keep only the 5 fields Claude actually needs for refund decisions
        tool_result["content"] = {
            "order_id":    content.get("order_id"),
            "status":      content.get("status"),
            "total":       content.get("total"),
            "items":       content.get("items"),
            "created_at":  content.get("created_at")
        }
        # Discard: internal_id, warehouse_id, billing_address, tax_breakdown,
        #          shipping_manifest, carrier_metadata, ... (35 other fields)
    return tool_result

options = ClaudeAgentOptions(
    post_tool_use_hooks=[trim_order_result]
)
```

### Why This Matters

| Without trimming | With trimming |
|---|---|
| 40 fields × every tool call × multiple turns = rapid context growth | Only 5 relevant fields per call — minimal context footprint |
| Claude processes irrelevant data, increasing noise | Claude sees only what it needs — better reasoning, lower cost |
| Context rot sets in faster | Context window stays lean longer |

### The Exam Pattern

**Scenario:** "An agent's context window fills up quickly during a customer support session because `lookup_order` returns 40 fields but the agent only uses 5. What is the correct fix?"

```
A) Switch to a model with a larger context window
B) Limit the agent to a maximum of three order lookups per session
C) Trim the tool output to include only return-relevant fields before appending to context  ← CORRECT
D) Summarize the entire conversation history every 5 turns
```

**Why C:** PostToolUse trimming is the correct tool-layer fix. Larger context windows (A) don't reduce cost or prevent rot — they just delay it. Arbitrary lookup limits (B) break functionality. Periodic summarization (D) is a context management technique, not a tool design fix.

**Exam rule:** When tool results are verbose, the fix is PostToolUse trimming — not model upgrades, context window increases, or lookup limits. This is a tool design decision, not a reliability workaround.

---

## 18. Anti-Patterns Master List

| Anti-Pattern | Why It's Wrong | Correct Approach |
|---|---|---|
| Identical or minimal tool descriptions | Claude cannot distinguish → misrouting | Write full 5-element descriptions with DO/DON'T disambiguators |
| Type-parameter overloaded tools | Internal branching is invisible to Claude | Split into separate focused tools |
| Treating empty results as errors | Infinite retry loop on successful queries | `isError: false` for empty results; `isError: true` only for failures |
| Vague error strings ("Operation failed") | Coordinator can't decide: retry? escalate? fix input? | Return structured error with `errorCategory` + `isRetryable` + message |
| Retrying on `isRetryable: false` | Validation/business/permission errors won't fix themselves | Check `isRetryable` before retrying — false means don't |
| 18+ tools per agent | Selection reliability degrades significantly | 4–5 tools max per agent role |
| Synthesis agent given web search tools | Agent searches instead of synthesizing | Remove out-of-role tools from `allowedTools` |
| Using prompt instruction to restrict tool use | Probabilistic — "don't use X" sometimes ignored | Remove the tool from `allowedTools` — deterministic |
| Committing secrets to `.mcp.json` | Security breach — credentials in version control | `${ENV_VAR}` substitution for all credentials |
| Personal MCP config in project-level file | Applies to all users — overly broad | Personal tools → `~/.claude.json`; shared tools → `.mcp.json` |
| Project MCP config in user-level file | New developers miss it on clone | Shared tools → `.mcp.json` committed to version control |
| Using stdio for remote/cloud MCP servers | stdio is local-only — cannot cross network | Use SSE or Streamable HTTP for remote servers |
| Using WebSocket as MCP transport | WebSocket is not a supported MCP transport | Use SSE (legacy) or Streamable HTTP (new) for remote |
| Building custom MCP server for standard tools | Reinventing the wheel — maintenance burden | Use community MCP server (Jira, GitHub, Slack already exist) |
| Making all schema fields required | Forces hallucination when data is absent | Optional/nullable fields for data not always present |
| Assuming strict: true catches semantic errors | strict validates syntax only | Add semantic validation in tool result processing |
| Using Glob to search file contents | Glob finds files by name, not contents | Use Grep for content search |
| Using Grep to find files by name | Grep searches contents, not filenames | Use Glob for filename/path pattern matching |
| Reading all files upfront to explore codebase | Token explosion — 50,000 files is millions of tokens | Grep for entry points → Read selectively → Glob for file types |
| System prompt phrase matching tool name | Biases Claude to over-call that tool | Rephrase system prompt or rename the tool |
| Closed enum with no escape hatches | Forces hallucination when no category fits the real answer | Add "other" and "unclear" to every classification enum |
| Missing `additionalProperties: false` in strict schemas | Claude may add unexpected extra fields causing server errors | Always include `"additionalProperties": false` in strict schemas |
| Ignoring custom MCP tool in favor of built-in (Grep) | Built-in is familiar; MCP tool description doesn't explain its advantage | Enhance MCP description to explicitly contrast with the built-in alternative |
| Retrying rate limit errors immediately | Reproduces the same error instantly — rate limits need time to expire | Rate limits are `transient` + `isRetryable: true` but require backoff delay |
| Treating all transient errors with same retry speed | Rate limits need delay; timeouts may be immediate | Check `retryAfterMs` hint; use category-appropriate strategy |
| Not improving tool descriptions after misrouting | Adds routing infrastructure on top of a broken foundation | Analyze failure modes → rewrite descriptions → test iteratively |
| Giving full out-of-role tool instead of constrained alternative | Synthesis agent misuses full web search | Provide constrained tool (verify_fact) for cross-role needs; route complex cases through coordinator |
| Generic tools instead of constrained alternatives | Agent can do more than intended (security/reliability risk) | Replace fetch_url with load_document that validates input scope |
| Subagent immediately escalates every error to coordinator | Floods coordinator with recoverable errors | Subagents retry transient errors locally first; escalate only what cannot be resolved |
| Escalating to coordinator without partial results and context | Coordinator can't make intelligent recovery decision | Always include attempted action + partial results when propagating errors |
| Appending verbose tool results directly to context | 40-field responses bloat context rapidly — accelerates context rot | Use PostToolUse hook to trim to only relevant fields before Claude sees result |
| Using model upgrade or larger context window to handle verbose tools | Delays the problem, increases cost | Fix at the tool layer: PostToolUse trim to relevant fields |

---

## 19. Key Rules to Memorize

```
1.  Tool descriptions ARE the routing layer — Claude has no other signal for tool selection
2.  6-element good description: what it does | input format | output structure | trigger + example queries | when NOT to use (boundary) | edge cases
3.  If misrouting occurs → fix descriptions first, before adding routing infrastructure
4.  Type-parameter overloaded tools → split them into separate focused tools
5.  isError: false + empty content = successful query with no results (NOT an error)
6.  isError: true = actual tool failure requiring error handling
7.  4 error categories: transient (retry) | validation (fix input) | business (escalate) | permission (rotate creds)
8.  isRetryable: true → exponential backoff | isRetryable: false → never retry
9.  4–5 tools max per agent — more tools = worse selection reliability
10. Remove out-of-role tools from allowedTools — don't rely on prompt instructions to restrict tool use
11. MCP primitives: Tools (actions) | Resources (read-only data) | Prompts (reusable templates)
12. stdio = local only (same machine) | SSE = remote over HTTP | Streamable HTTP = recommended new remote
13. WebSocket is NOT a supported MCP transport — always wrong answer
14. Project MCP config: .mcp.json (version controlled, shared)
15. Personal MCP config: ~/.claude.json (not shared, machine-local)
16. Secrets always via ${ENV_VAR} substitution — never hardcoded values in .mcp.json
17. Community MCP server for standard tools (Jira, GitHub) | Custom for team-specific workflows only
18. strict: true guarantees syntax compliance; does NOT guarantee semantic correctness
19. Nullable fields (type: ["string", "null"]) prevent hallucination on absent optional data
20. tool_choice "any" = must use a tool (your choice) | "tool" = must use THIS specific tool
21. Forced tool_choice for pipeline steps that always require the same tool
22. Grep = search file CONTENTS | Glob = find files by NAME/PATH pattern
23. Edit = targeted modification (unique text matching) | Write = full file overwrite
24. Edit fails on non-unique text → fall back to Read + Write
25. System prompt keywords that match tool names bias Claude to over-call that tool — audit and rephrase
26. MCP Resources = content catalogs agents read without tool calls (no side effects, reduces exploratory scanning)
27. Streamable HTTP = only transport with native OAuth for production remote auth
28. Treat empty results as success — they represent valid queries with no matching data
29. Return partial results with structured error when tool partially fails before timing out
30. Prefer existing community MCP servers — don't rebuild standard integrations from scratch
31. Rate limit errors = transient + isRetryable: true BUT require backoff delay — never retry immediately
32. "Always retry" = wrong exam answer for auth failures and schema validation errors
33. Enum fields must include "other" and "unclear" escape hatches — closed enums force hallucination
34. additionalProperties: false in strict schemas prevents Claude from adding unexpected fields
35. Built-in tool preference over MCP: fix by enhancing MCP tool description to explain why it's better
36. Tool-testing agent pattern: test tool dozens of times → identify failure modes → rewrite description → 40% faster completion
37. Cross-role tools: use a constrained alternative tool (e.g., verify_fact) for occasional cross-role needs — never give the full out-of-role tool
38. Replace generic tools with constrained alternatives (fetch_url → load_document) to enforce scope without prompt instructions
39. Forced tool_choice sequencing: force specific tool in turn 1, use "auto" in turn 2 — deterministic first step without hooks
40. Local error recovery: subagents retry transient errors locally first; only propagate to coordinator errors they cannot resolve, with attempted context + partial results
41. PostToolUse trimming: filter verbose tool results (40 fields → 5 relevant) before appending to context — prevents rapid context bloat at the tool layer
42. Official split-tool names: analyze_document → extract_data_points + summarize_content + verify_claim_against_source | analyze_content → extract_web_results
43. Wrapper module tracing: Grep exported names first, then Grep each name across codebase — don't assume you know the import path or alias used by callers
44. Business errors require customer-friendly message, not just an internal flag — agent must relay reason to user ("refund exceeds $500 limit, a support agent will follow up")
45. Tools from ALL configured MCP servers are discovered at connection time and available simultaneously — no per-server activation needed; description disambiguation applies across servers
```

---

## 20. Practice Questions (20 MCQs)

---

**Q1.** A customer support agent is calling `get_customer` for order-related queries 40% of the time. Logs show both `get_customer` and `lookup_order` have nearly identical descriptions: "Gets customer information" and "Gets order information." What is the correct first fix?

- A) Add a routing classifier that inspects the query before tool selection
- B) Merge the two tools into a single `get_entity` tool with a type parameter
- C) Expand both tool descriptions to include input format, output structure, and explicit DO/DON'T disambiguation
- D) Upgrade to a more powerful Claude model with better tool selection

---

**Q2.** An MCP tool returns `{"message": "Database error"}` when the database times out. What is wrong with this response and what is the correct fix?

- A) Nothing is wrong — the message is clear enough for the agent to decide what to do
- B) The response is missing `isError`, `errorCategory`, and `isRetryable` — the agent cannot determine whether to retry or escalate
- C) The response should return an HTTP 500 status code instead of a message field
- D) The tool should throw an exception rather than returning a structured response

---

**Q3.** A tool returns `{"isError": false, "content": [], "message": "No orders found for CUS-48291"}`. What should the agent do?

- A) Retry the query — empty content indicates the tool failed
- B) Treat it as a successful query result and continue processing — empty content is valid
- C) Return an error to the user — the customer has no order history
- D) Escalate to a human — missing data requires human verification

---

**Q4.** You are building an MCP server that needs to stream large file contents to Claude during a long-running task in a cloud environment. Which transport should you use?

- A) stdio — lower latency for subprocess communication
- B) SSE — supports real-time streaming over HTTP and crosses network boundaries
- C) WebSocket — provides full bidirectional streaming
- D) stdio — simpler to implement and avoids HTTP server overhead

---

**Q5.** A developer wants team-wide MCP server configuration for a Jira integration to be available to all developers who clone the repository. Where should this be configured?

- A) `~/.claude.json` on each developer's machine
- B) `.mcp.json` in the project root, committed to version control
- C) `.claude/settings.local.json` with a note in the README to copy it
- D) The system prompt in CLAUDE.md with the Jira credentials included

---

**Q6.** A synthesis agent in a multi-agent research system keeps performing web searches even though it should only be synthesizing findings passed by the coordinator. What is the correct fix?

- A) Add "Do not search the web" to the synthesis agent's system prompt
- B) Remove the web search tool from the synthesis agent's `allowedTools` list
- C) Add a PostToolUse hook that cancels web search results from the synthesis agent
- D) Instruct the coordinator to include a "no web searching" reminder in every subagent prompt

---

**Q7.** What does `strict: true` in a tool definition guarantee?

- A) Claude will always call this tool when it's appropriate for the query
- B) Claude's tool call input will always match the declared JSON schema
- C) The tool will validate that its output matches the expected response schema
- D) Claude will not call this tool in parallel with other tools

---

**Q8.** An agent needs to find all files in a TypeScript codebase that import from `@/auth/tokens`. Which built-in tool is correct?

- A) Glob — searches for files by the `@/auth/tokens` path pattern
- B) Grep — searches file contents for the import string
- C) Read — reads each file to check for the import statement
- D) Bash — runs a shell find command

---

**Q9.** An agent needs to find all `*.test.ts` files anywhere in a large monorepo. Which built-in tool is correct?

- A) Grep — searches for "test" in file contents
- B) Bash — runs `find . -name "*.test.ts"`
- C) Glob — finds files matching the `**/*.test.ts` pattern
- D) Read — reads directory listings recursively

---

**Q10.** A tool returns `isError: true` with `errorCategory: "validation"` and `isRetryable: false`. What should the agent do?

- A) Retry with exponential backoff — transient errors often resolve on retry
- B) Do not retry — fix the invalid input format before attempting the call again
- C) Escalate to a human agent — validation errors require manual intervention
- D) Retry once immediately, then escalate if it fails again

---

**Q11.** Which schema design prevents Claude from hallucinating a value for a field when the source data doesn't contain that information?

- A) Mark the field as required with a default value of `"unknown"`
- B) Make the field optional with `"type": ["string", "null"]` and exclude it from `required`
- C) Add strict: true to ensure Claude fills in all fields
- D) Add a description saying "leave this field blank if unknown"

---

**Q12.** A tool description says: "Analyzes documents. If type='web', processes web content. If type='pdf', processes PDF files." What is the architectural problem?

- A) The tool name is too generic and should be renamed
- B) The type parameter creates invisible internal branching — Claude cannot distinguish the behaviors; the tool should be split into separate focused tools
- C) The description is too long and should be shortened
- D) The tool should accept both web and PDF input without a type parameter

---

**Q13.** A coordinator MCP server needs OAuth authentication for a production cloud deployment. Which transport supports this natively?

- A) stdio with environment variable token injection
- B) SSE with OAuth headers in the HTTP request
- C) Streamable HTTP with native OAuth support
- D) WebSocket with JWT tokens in the handshake

---

**Q14.** An agent's system prompt contains the phrase "analyze and process content from each source." The agent has a tool named `process_content`. What problem does this create?

- A) No problem — system prompt phrasing doesn't affect tool selection
- B) The phrase "process content" in the system prompt biases Claude to call `process_content` even when a different tool is more appropriate
- C) The tool name is too generic and will cause the agent to fail
- D) The system prompt should explicitly tell Claude not to use `process_content`

---

**Q15.** An exploration agent running in phase 1 of a codebase analysis task accidentally modifies a file. What is the root cause and correct fix?

- A) The agent misunderstood its role — add a stronger system prompt instruction
- B) The agent was given Write access during the exploration phase — use a task-scoped tool profile with Read/Grep/Glob only for exploration
- C) The Edit tool should be removed from all agents during analysis
- D) The agent should ask for confirmation before every file operation

---

**Q16.** A team uses Jira for issue tracking. They need Claude agents to query Jira for ticket status. What is the correct approach?

- A) Build a custom MCP server from scratch to integrate with the Jira API
- B) Write a system prompt that tells Claude to simulate Jira queries using its training knowledge
- C) Use an existing community Jira MCP server and configure it in `.mcp.json`
- D) Give Claude direct API credentials and let it call the Jira REST API with the Bash tool

---

**Q17.** An agent receives `isError: true` with `errorCategory: "permission"`. What is the correct agent action?

- A) Retry with exponential backoff — permission errors are often transient
- B) Do not retry — alert operations or escalate; permission errors require credential rotation or scope changes
- C) Ask the user to grant permission through the interface
- D) Continue with a fallback tool that performs a similar function

---

**Q18.** A tool description for `search_internal_kb` says "Searches for information." A tool description for `search_web` also says "Searches for information." Logs show misrouting at 35%. What is the minimum change that fixes this?

- A) Add a routing classifier layer between Claude and the tools
- B) Merge both tools into one `search` tool
- C) Rewrite both descriptions to specify the data source, input format, return structure, and explicit disambiguation (DO/DON'T USE)
- D) Remove one of the tools so Claude only has one search option

---

**Q19.** Which statement about MCP Resources is correct?

- A) Resources are callable actions that agents invoke to modify external systems
- B) Resources are read-only data catalogs that agents access without making tool calls, reducing exploratory scanning
- C) Resources are reusable prompt templates that ensure consistent agent behavior
- D) Resources require the same error handling as tool calls since they can fail

---

**Q20.** A developer uses `tool_choice: {"type": "any"}` for a structured data extraction pipeline. A teammate suggests changing to `tool_choice: {"type": "tool", "name": "extract_invoice_data"}`. What is the difference?

- A) No difference — both force Claude to call a tool
- B) `"any"` forces Claude to use at least one tool of its choice; `"tool"` forces Claude to call the specific named extraction tool — the specific force is needed for a fixed pipeline step
- C) `"any"` is only valid for agents with multiple tools; `"tool"` works with a single tool
- D) `"tool"` with a name is deprecated — `"any"` is the recommended approach

---

## 21. Answer Key & Explanations

| Q | Answer | Key Reason |
|---|---|---|
| 1 | **C** | Fix descriptions first — they are the only routing signal. Routing classifiers add infrastructure on top of a broken foundation |
| 2 | **B** | Missing `isError`, `errorCategory`, `isRetryable` — coordinator can't make recovery decisions without these fields |
| 3 | **B** | `isError: false` + empty content = successful query with no results. Not an error. Agent continues |
| 4 | **B** | SSE crosses network boundaries. stdio is local-only. WebSocket is not a supported MCP transport |
| 5 | **B** | `.mcp.json` in project root, version-controlled — all developers get it on clone |
| 6 | **B** | Remove from `allowedTools` — deterministic. Prompt instructions are probabilistic and will sometimes be ignored |
| 7 | **B** | `strict: true` guarantees schema-compliant input structure. Does NOT guarantee semantic correctness |
| 8 | **B** | Grep — you're searching file CONTENTS for an import string. Glob finds files by name pattern |
| 9 | **C** | Glob — you're finding files by NAME pattern (`*.test.ts`). Grep searches contents |
| 10 | **B** | `isRetryable: false` = never retry. Fix the input format before calling again |
| 11 | **B** | `type: ["string", "null"]` + excluded from `required` — Claude can leave it null when data is absent |
| 12 | **B** | Type-parameter branching is invisible to Claude. Split into separate focused tools |
| 13 | **C** | Streamable HTTP has native OAuth support. SSE and stdio don't |
| 14 | **B** | System prompt phrase "process content" creates selection bias toward `process_content` |
| 15 | **B** | Write was in allowedTools during exploration phase. Task-scoped Read/Grep/Glob only for exploration |
| 16 | **C** | Community Jira MCP server exists — use it. Don't rebuild standard integrations |
| 17 | **B** | Permission errors = not retryable. Rotate credentials or escalate to operations |
| 18 | **C** | Rewrite descriptions with full disambiguation. Root cause is the descriptions, not the architecture |
| 19 | **B** | Resources are read-only catalogs — no side effects, reduce exploratory tool calls |
| 20 | **B** | `"any"` = at least one tool, Claude's choice; `"tool"` = this specific named tool. For a fixed pipeline step, use the specific force |

---

## Quick Cheat Sheet — Domain 2

```
TOOL DESCRIPTIONS (2.1)
  → 6 elements: what | input format | output | trigger + example queries | when NOT to use (boundary) | edge cases
  → Misrouting? Fix descriptions first — before adding routing infrastructure
  → Type-parameter tool → split into separate tools (analyze_document → extract_data_points + summarize_content + verify_claim_against_source)
  → System prompt keyword = tool name → rename tool or rephrase prompt

MCP ERROR STRUCTURE (2.2)
  → Required: isError | errorCategory | isRetryable | message
  → transient (retry) | validation (fix input) | business (escalate + customer-friendly message) | permission (rotate)
  → Business errors: include customer-friendly explanation agent can relay to user — not just internal flag
  → Rate limits = transient + isRetryable:true BUT need backoff delay — NOT immediate retry
  → "Always retry" = wrong for auth failures and validation errors
  → isError:false + empty content = SUCCESS (not an error)
  → isError:true = actual failure
  → isRetryable:false → NEVER retry, regardless of error type

TOOL SCOPING (2.3)
  → 4–5 tools max per agent
  → Remove out-of-role tools from allowedTools — don't use prompts to restrict
  → Task-scoped profiles: exploration (Read/Grep/Glob) | dev (+ Write/Bash) | deploy (+ deploy tools)
  → tool_choice: auto | any | tool (specific) | none
  → strict:true = syntax guarantee only, not semantic
  → Enum fields: always add "other" + "unclear" escape hatches
  → additionalProperties:false = prevents Claude adding unexpected fields
  → Built-in tool preferred over MCP? → enhance MCP tool description to explain its advantage
  → Tool-testing agent: test → identify failure modes → rewrite description → 40% faster

MCP CONFIGURATION (2.4)
  → .mcp.json = project-level, version-controlled, shared
  → ~/.claude.json = personal, not shared
  → Secrets: always ${ENV_VAR} substitution
  → Community MCP server > custom for standard integrations
  → stdio = local | SSE = remote (legacy) | Streamable HTTP = remote (new, OAuth-native)
  → WebSocket = NOT supported — always wrong
  → Tools from ALL configured servers available simultaneously at connection time — no per-server activation

BUILT-IN TOOLS (2.5)
  → Grep = search CONTENTS | Glob = find by NAME/PATH
  → Edit = targeted (unique text) | Write = full overwrite
  → Edit fails on non-unique → fall back to Read + Write
  → Explore incrementally: Grep entry points → Read selectively → Glob file types

MCP RESOURCES
  → Read-only data catalogs (no side effects)
  → Agents read schemas/catalogs without tool calls
  → Reduces exploratory scanning → fewer tokens
```