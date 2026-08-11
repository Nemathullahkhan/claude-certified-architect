# PART I: THEORY FOUNDATIONS

This part covers all the theory you need to pass the exam successfully. The material is organized by
technologies and concepts rather than by exam domains—this helps you build a deeper understanding of
each topic.

## Chapter 1: Claude API — Fundamentals of Model Interaction

*Documentation: Messages API | Prompt Engineering*

### 1.1 API Request Structure

The Claude API follows a request–response model. Each request to the Claude Messages API includes:

```json
{
  "model": "claude-sonnet-4-6",
  "max_tokens": 1024,
  "system": "You are a helpful assistant.",
  "messages": [
    {"role": "user", "content": "Hi!"},
    {"role": "assistant", "content": "Hello!"},
    {"role": "user", "content": "How are you?"}
  ],
  "tools": [],
  "tool_choice": {"type": "auto"}
}
```

Key fields:

- `model` — model selection (`claude-opus-4-6`, `claude-sonnet-4-6`, `claude-haiku-4-5`)
- `max_tokens` — maximum number of tokens in the response
- `system` — the system prompt (defines model behavior)
- `messages` — conversation history (you must send the full history to maintain coherence)
- `tools` — definitions of available tools
- `tool_choice` — tool selection strategy

### 1.2 Message Roles

The `messages` array uses three roles:

- `user` — user messages
- `assistant` — model responses (included when sending history)
- `tool` — tool call results (the role is not explicitly set; this appears as a `tool_result` content block)

Critically important: on every API request you must send the full conversation history. The model does
not persist state between requests—each call is independent.

### 1.3 The stop_reason Field in the Response

The Claude API response includes `stop_reason`, which indicates why the model stopped generating:

| Value | Description | Action |
|---|---|---|
| `"end_turn"` | The model finished its response | Show the result to the user |
| `"tool_use"` | The model wants to call a tool | Execute the tool and return the result |
| `"max_tokens"` | Token limit reached | The response is truncated; you may need to increase the limit |
| `"stop_sequence"` | A stop sequence was encountered | Handle based on your application logic |

For agentic systems, `"tool_use"` and `"end_turn"` are the most important—they control the agent loop.

### 1.4 System Prompt

The system prompt is a special instruction that defines context and behavioral rules. It:

- Is not part of the `messages` array; it is passed separately in the `system` field
- Has priority over user messages
- Is loaded once and applies throughout the conversation
- Is used to define role, constraints, and output format

Important for the exam: system prompt wording can create unintended tool associations. For example,
an instruction like "always verify the customer" can cause the model to overuse `get_customer`, even
when it is unnecessary.

### 1.5 Context Window

The context window is the total amount of text (in tokens) the model can process at once. It includes:

- The system prompt
- The full message history
- Tool definitions
- Tool results

Key context-window problems:

1. **Lost-in-the-middle effect**: models reliably process information at the start and end of a long input but can miss details in the middle. Mitigation: place key information near the beginning or end.
2. **Accumulation of tool results**: every tool call adds output to the context. If a tool returns 40+ fields but only 5 matter, then most of the context is wasted.
3. **Progressive summarization**: when compressing history, numeric values, percentages, and dates often get lost and become vague ("about", "roughly", "a few").

## Chapter 2: Tools and tool_use

*Documentation: Tool Use*

### 2.1 What is tool_use

`tool_use` is a mechanism that allows Claude to call external functions. The model does not run code
directly—it generates a structured tool call request; your code executes it and returns the result.

### 2.2 Tool Definition

Each tool is defined using a JSON schema:

```json
{
  "name": "get_customer",
  "description": "Finds a customer by email or ID. Returns the customer profile, including name, email, order history, and account status. Use this tool BEFORE lookup_order to verify the customer's identity. Accepts an email (format: user@domain.com) or a numeric customer_id.",
  "input_schema": {
    "type": "object",
    "properties": {
      "email": {"type": "string", "description": "Customer email"},
      "customer_id": {"type": "integer", "description": "Numeric customer ID"}
    },
    "required": []
  }
}
```

Critically important aspects of a tool description:

1. **The description is the primary selection mechanism.** An LLM chooses tools based on their descriptions. Minimal descriptions ("Retrieves customer information") lead to mistakes when tools overlap.
2. **Include in the description:**
   - What the tool does and returns
   - Input formats and example values
   - Edge cases and constraints
   - When to use this tool vs similar alternatives
3. **Avoid identical or overlapping descriptions across tools.** If `analyze_content` and `analyze_document` have nearly identical descriptions, the model will confuse them.
4. **Built-in tools vs MCP tools**: agents may prefer built-in tools (Read, Grep) over MCP tools with similar functionality. To prevent this, strengthen MCP tool descriptions—highlight concrete advantages, unique data, or context that built-in tools cannot provide.

### 2.3 The tool_choice Parameter

`tool_choice` controls how the model selects tools:

| Value | Behavior | When to use |
|---|---|---|
| `{"type": "auto"}` | The model decides whether to call a tool or answer in text | Default for most cases |
| `{"type": "any"}` | The model must call some tool | When you need guaranteed structured output |
| `{"type": "tool", "name": "extract_metadata"}` | The model must call a specific tool | When you need a forced first step / execution order |

Important scenarios:

- `tool_choice: "any"` + multiple extraction tools → the model picks the best one, but you still get structured output
- Forced selection → when you must guarantee a specific first action (e.g., `extract_metadata` before enrichment)

### 2.4 JSON Schemas for Structured Output

Using `tool_use` with JSON schemas is the most reliable way to obtain structured output from Claude. It:

- Guarantees syntactically valid JSON (no missing braces, no trailing commas)
- Enforces the required structure (required fields are present)
- Does not guarantee semantic correctness (values can still be wrong)

Schema design — key principles:

```json
{
  "type": "object",
  "properties": {
    "category": {
      "type": "string",
      "enum": ["bug", "feature", "docs", "unclear", "other"]
    },
    "category_detail": {
      "type": ["string", "null"],
      "description": "Details if category = 'other' or 'unclear'"
    },
    "severity": {
      "type": "string",
      "enum": ["critical", "high", "medium", "low"]
    },
    "confidence": {
      "type": "number",
      "minimum": 0,
      "maximum": 1
    },
    "optional_field": {
      "type": ["string", "null"],
      "description": "Null if the information was not found in the source"
    }
  },
  "required": ["category", "severity"]
}
```

Schema design rules:

1. **Required vs optional**: mark fields as required only if the information is always available. Required fields push the model to fabricate values when data is missing.
2. **Nullable fields**: use `"type": ["string", "null"]` for information that may be absent. The model can return `null` instead of hallucinating.
3. **Enums with "other"**: add `"other"` + a detail string to avoid losing data outside your predefined categories.
4. **Enum "unclear"**: for cases where the model cannot confidently pick a category—honest `"unclear"` is better than a wrong category.

### 2.5 Syntax vs Semantic Errors

| Error type | Example | Mitigation |
|---|---|---|
| Syntax | Invalid JSON, wrong field type | `tool_use` with a JSON schema (eliminates) |
| Semantic | Totals don't add up, value in wrong field, hallucination | Validation checks, retry with feedback, self-correction |

## Chapter 3: Claude Agent SDK — Building Agentic Systems

*Documentation: Agent SDK | Hooks | Subagents | Sessions*

### 3.1 What is an Agentic Loop

The agentic loop is the core pattern for autonomous task execution. The model doesn't just answer—it
performs a sequence of actions:

1. Send a request to Claude with tools
2. Receive a response
3. Check `stop_reason`:
   - `"tool_use"` → execute the tool, append the result to history, go back to step 1
   - `"end_turn"` → the task is complete, show the result to the user
4. Repeat until completion

This is a model-driven approach: Claude decides which tool to call next based on context and prior tool
results. This differs from hard-coded decision trees where the action sequence is fixed.

Anti-patterns (avoid):

- Parsing assistant text to detect completion ("Task completed")
- Using an arbitrary iteration limit (e.g., `max_iterations=5`) as the primary stop condition
- Checking whether the assistant produced textual content as a completion signal

Correct approach: the only reliable completion signal is `stop_reason == "end_turn"`.

### 3.2 AgentDefinition Configuration

`AgentDefinition` is the agent configuration object in the Claude Agent SDK:

```python
agent = AgentDefinition(
    name="customer_support",
    description="Handles customer requests for returns and order issues",
    system_prompt="You are a customer support agent...",
    allowed_tools=["get_customer", "lookup_order", "process_refund", "escalate_to_human"],
    # For a coordinator:
    # allowed_tools=["Task", "get_customer", ...]
)
```

Key parameters:

- `name` / `description` — identification and description of the agent
- `system_prompt` — system prompt with instructions
- `allowed_tools` — list of allowed tools (principle of least privilege)

### 3.3 Hub-and-Spoke: Coordinator and Subagents

A multi-agent architecture is typically built as a hub-and-spoke topology:

```
        Coordinator
        /   |    \
Subagent1 Subagent2 Subagent3
(search)  (analysis) (synthesis)
```

The coordinator is responsible for:

- Decomposing the task into subtasks
- Deciding which subagents are needed (dynamic selection)
- Delegating work to subagents
- Aggregating and validating results
- Handling errors and retries
- Communicating results to the user

Critical principle: subagents have isolated context.

- Subagents do not automatically inherit the coordinator's conversation history
- All required context must be explicitly passed in the subagent prompt
- Subagents do not share memory across calls
- All communication flows through the coordinator (for observability and error control)

### 3.4 The Task Tool for Spawning Subagents

Subagents are spawned via the Task tool:

```python
# The coordinator's allowedTools must include "Task"
coordinator_agent = AgentDefinition(
    allowed_tools=["Task", "get_customer"]
)
```

Explicit context passing is mandatory:

```
# Bad: the subagent has no context
Task: "Analyze the document"

# Good: full context in the prompt
Task: "Analyze the following document.
Document: [full document text]
Prior search results: [web search results]
Output format requirements: [schema]"
```

Parallel spawning: a coordinator can call multiple `Task`s in one response—subagents run in parallel:

```
# One coordinator response contains:
Task 1: "Search for articles about X"
Task 2: "Analyze document Y"
Task 3: "Search for articles about Z"
# All three run concurrently
```

### 3.5 Hooks in the Agent SDK

Hooks allow interception and transformation at specific points in the agent lifecycle.

`PostToolUse` intercepts a tool result before it is provided to the model:

```python
# Example: normalize date formats from different MCP tools
@hook("PostToolUse")
def normalize_dates(tool_result):
    # Convert Unix timestamp -> ISO 8601
    # Convert "Mar 5, 2025" -> "2025-03-05"
    return normalized_result
```

Outgoing-call interception hook blocks actions that violate policy:

```python
# Example: block refunds above $500
@hook("PreToolUse")
def enforce_refund_limit(tool_call):
    if tool_call.name == "process_refund" and tool_call.args.amount > 500:
        return redirect_to_escalation(tool_call)
```

Key difference: hooks vs prompt instructions

| Attribute | Hooks | Prompt instructions |
|---|---|---|
| Guarantee | Deterministic (100%) | Probabilistic (>90%, not 100%) |
| When to use | Critical business rules, financial operations, compliance | General preferences, recommendations, formatting |
| Example | Block refunds > $500 | "Try to solve before escalating" |

Rule: when failure has financial, legal, or safety consequences—use hooks, not prompts.

## Chapter 4: Model Context Protocol (MCP)

*Documentation: MCP | Tools | Resources | Servers*

### 4.1 What is MCP

The Model Context Protocol (MCP) is an open protocol for connecting external systems to Claude. MCP
defines three primary resource types:

1. **Tools** — functions the agent can call to perform actions (CRUD operations, API calls, command execution)
2. **Resources** — data the agent can read for context (documentation, database schemas, content catalogs)
3. **Prompts** — predefined prompt templates for common tasks

### 4.2 MCP Servers

An MCP server is a process that implements the MCP protocol and provides tools/resources. When you
connect to an MCP server:

- All tools are discovered automatically
- Tools from all connected servers are available at once
- Tool descriptions determine how the model will use them

### 4.3 Configuring MCP Servers

Project configuration (`.mcp.json`) — for team usage:

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "jira": {
      "command": "npx",
      "args": ["-y", "mcp-server-jira"],
      "env": {
        "JIRA_TOKEN": "${JIRA_TOKEN}"
      }
    }
  }
}
```

Key points:

- `.mcp.json` is stored at the project root and managed in version control
- Environment variables (`${GITHUB_TOKEN}`) are used for secrets—tokens themselves are not committed
- Available to all project contributors

User configuration (`~/.claude.json`) — for personal/experimental servers:

- Stored in the user's home directory
- Not shared via version control
- Suitable for personal experiments and testing

Choosing servers:

- For standard integrations (Jira, GitHub, Slack), prefer existing community MCP servers
- Build your own servers only for unique, team-specific workflows

### 4.4 The isError Flag in MCP

When an MCP tool encounters an error, it uses `isError: true` in the response. This signals to the agent
that the call failed.

Structured error (good):

```json
{
  "isError": true,
  "content": {
    "errorCategory": "transient",
    "isRetryable": true,
    "message": "The service is temporarily unavailable. Timeout while calling the orders API.",
    "attempted_query": "order_id=12345",
    "partial_results": null
  }
}
```

Generic error (anti-pattern):

```json
{
  "isError": true,
  "content": "Operation failed"
}
```

A generic error gives the agent no information for decision-making—should it retry, change the query, or
escalate?

### 4.5 MCP Resources

Resources are data that an agent can request to get context without taking actions:

- Content catalogs (e.g., a list of all project tasks, hierarchical navigation)
- Database schemas (understanding data structure)
- Documentation (API references, internal guides)
- Issue/task summaries

Resource advantage: the agent does not need exploratory tool calls to understand what data exists. A
resource provides an immediate "map."

## Chapter 5: Claude Code — Configuration and Workflows

*Documentation: Claude Code | Memory / CLAUDE.md | Skills | MCP | Hooks | Sub-agents | GitHub Actions | Headless*

### 5.1 The CLAUDE.md Hierarchy

CLAUDE.md is the instruction file(s) for Claude Code. There is a three-level hierarchy:

1. **User-level**: `~/.claude/CLAUDE.md`
   - Applies only to that user
   - NOT shared via VCS
   - Personal preferences and working style
2. **Project-level**: `.claude/CLAUDE.md` or a root `CLAUDE.md`
   - Applies to all project contributors
   - Managed via VCS
   - Coding standards, testing standards, architectural decisions
3. **Directory-level**: `CLAUDE.md` in subdirectories
   - Applies when working with files in that directory
   - Conventions specific to that part of the codebase

Common mistake: a new team member does not receive project instructions because they were placed
in `~/.claude/CLAUDE.md` (user-level) instead of `.claude/CLAUDE.md` (project-level).

### 5.2 @path Syntax (File Imports)

CLAUDE.md can reference external files using `@path`, making configuration modular:

```markdown
# Project CLAUDE.md
Coding standards are described in @./standards/coding-style.md
Test requirements are in @./standards/testing-requirements.md
Project overview is in @README.md and dependencies are in @package.json
```

Rules for `@path`:

- Use `@` immediately before the file path (no space)
- Relative and absolute paths are supported
- Relative paths are resolved relative to the file that contains the import
- Maximum import nesting depth is 5

This avoids duplication and lets each package include only relevant standards.

### 5.3 The .claude/rules/ Directory

`.claude/rules/` is an alternative to a monolithic CLAUDE.md, used to organize rules by topic:

```
.claude/rules/
  testing.md            -- testing conventions
  api-conventions.md    -- API conventions
  deployment.md         -- deployment rules
  react-patterns.md     -- React patterns
```

Key feature: YAML frontmatter with `paths` for conditional loading:

```markdown
---
paths: ["src/api/**/*"]
---
For API files, use async/await with explicit error handling.
Each endpoint must return a standard response wrapper.
```

```markdown
---
paths: ["**/*.test.tsx", "**/*.test.ts"]
---
Tests must use describe/it blocks.
Use data factories instead of hardcoding.
Do not mock the database—use a test database.
```

How it works:

- A rule is loaded only when Claude Code edits a file matching the `paths` pattern
- This saves context and tokens—irrelevant rules are not loaded
- Glob patterns let you apply conventions by file type regardless of location (ideal for tests scattered across the codebase)

When to use `.claude/rules/` with `paths` vs directory-level CLAUDE.md:

- `.claude/rules/` with `paths` — when conventions apply to files spread across many directories (tests, migrations)
- Directory-level CLAUDE.md — when conventions are tied to a specific directory and are not needed elsewhere

### 5.4 Custom Slash Commands and Skills

> Note: in the current Claude Code version, custom commands (`.claude/commands/`) are unified with skills (`.claude/skills/`). Both formats create `/name` commands. The exam guide references `.claude/commands/`—that format is still supported.

Slash commands are reusable prompt templates invoked via `/name`:

`.claude/commands/` format (legacy, supported):

```
.claude/commands/
  review.md      -- /review    -- standard code review
  test-gen.md    -- /test-gen  -- test generation
```

`.claude/skills/` format (current):

```
.claude/skills/
  review/SKILL.md    -- /review    -- with frontmatter configuration
  test-gen/SKILL.md
```

Project commands (`.claude/commands/` or `.claude/skills/`):

- Stored in VCS and available to everyone when cloning the repo
- Ensure consistent workflows across the team

User commands (`~/.claude/commands/` or `~/.claude/skills/`):

- Personal commands not shared via VCS
- For individual workflows

### 5.5 Skills — .claude/skills/

Skills are advanced commands configured via SKILL.md frontmatter:

```markdown
---
context: fork
allowed-tools: ["Read", "Grep", "Glob"]
argument-hint: "Path to the directory to analyze"
---
Analyze the code structure in the specified directory.
Output a report on dependencies and architectural patterns.
```

Frontmatter parameters:

| Parameter | Description |
|---|---|
| `context: fork` | Runs the skill in an isolated subagent. Verbose output does not pollute the main session |
| `allowed-tools` | Restricts which tools are available (security—e.g., the skill cannot delete files if not allowed) |
| `argument-hint` | Hint that asks for an argument when invoked without parameters |

When to use a skill vs CLAUDE.md:

- Skill — on-demand invocation for a specific task (review, analysis, generation)
- CLAUDE.md — always-loaded general standards and conventions

Personal skills (`~/.claude/skills/`):

- Create personal variants under different names so you don't affect teammates

### 5.6 Planning Mode vs Direct Execution

Planning mode:

- The model only investigates and plans; it does not make changes
- Uses Read, Grep, Glob to explore the codebase
- Produces an implementation plan that the user approves
- Safe exploration with no side effects

When to use planning mode:

- Large changes (dozens of files)
- Multiple plausible approaches (microservices: how to define boundaries?)
- Architectural decisions (which framework? what structure?)
- Unfamiliar codebase (you must understand before changing)
- Library migrations affecting 45+ files

When to use direct execution:

- Single-file fixes with a clear stack trace
- Adding one validation check
- Well-understood, unambiguous changes

Combined approach:

1. Planning mode for investigation and design
2. User approves the plan
3. Direct execution to implement the approved plan

Explore subagent — a specialized subagent for exploring the codebase:

- Isolates verbose output from the main context
- Returns only a summary
- Prevents context-window exhaustion in multi-phase tasks

### 5.7 The /compact Command

`/compact` is a built-in command for compressing context:

- Summarizes prior history to free up the context window
- Used in long investigation sessions when the context fills up with verbose tool output
- Risk: exact numeric values, dates, and specific details can be lost during summarization

### 5.8 The /memory Command

`/memory` is a built-in command for managing memory between sessions:

- Opens the CLAUDE.md file for editing, allowing you to save notes, preferences, and context
- Information persists across sessions and is automatically loaded on startup
- Useful for storing project conventions, user preferences, frequently used commands, and current work context
- Alternative to re-explaining the same instructions in every session

### 5.9 Claude Code CLI for CI/CD

The `-p` (or `--print`) flag:

```bash
claude -p "Analyze this pull request for security issues"
```

- Non-interactive mode: processes the prompt, prints to stdout, exits
- Does not wait for user input
- The only correct way to run Claude in CI/CD pipelines

Structured output for CI:

```bash
claude -p "Review this PR" --output-format json --json-schema '{"type":"object",...}'
```

- `--output-format json` — output in JSON
- `--json-schema` — validate output against a schema
- The result can be parsed to automatically post inline PR comments

Session context isolation: The same Claude session that generated code is often less effective at
reviewing it (the model retains its reasoning context and is less likely to challenge its own decisions). Use
an independent instance for review.

Preventing duplicate comments: When re-reviewing after new commits, include prior review results in
context and instruct Claude to report only new or unresolved issues.

### 5.10 fork_session and Session Management

`--resume <session-name>` resumes a named session:

```bash
claude --resume investigation-auth-bug
```

- Continues a prior conversation with saved context
- Useful for long investigations across multiple sessions
- Risk: if files changed since the prior session, tool results may be stale

`fork_session` creates an independent branch from shared context:

```
Codebase investigation
       |
   fork_session
    /       \
Approach A:  Approach B:
Redux        Context API
```

- Both forks inherit context up to the branch point
- Afterwards, they diverge independently
- Useful for comparing approaches or testing strategies

When to start a new session instead of resuming:

- Tool results are stale (files changed)
- A lot of time has passed and context has degraded
- It is better to restart with "Here is a short summary of what we found: ..." than to resume with old tool data

## Chapter 6: Prompt Engineering — Advanced Techniques

*Documentation: Prompt Engineering | Anthropic Cookbook*

### 6.1 Few-shot Prompting

Few-shot prompting is the inclusion of 2–4 input/output examples in a prompt to demonstrate the expected
behavior.

Why few-shot is more effective than textual descriptions:

- A vague instruction like "be more precise" can be interpreted in many ways
- An example unambiguously shows the expected format and decision logic
- The model generalizes the pattern to new cases (it does not just repeat the examples)

Types of few-shot examples and when to use them:

**1. Examples for ambiguous scenarios:**

```
Request: "My order is broken"
Action: Call get_customer -> lookup_order -> check status.
Rationale: "broken" may mean a damaged item; you need order details.

Request: "Get me a manager"
Action: Immediately call escalate_to_human.
Rationale: The customer explicitly requests a human. Do not attempt to solve autonomously.
```

**2. Examples for output formatting:**

Finding example:

```json
{
  "location": "src/auth/login.ts:42",
  "issue": "SQL injection in the username parameter",
  "severity": "critical",
  "suggested_fix": "Use a parameterized query"
}
```

**3. Examples to separate acceptable vs problematic code:**

```javascript
// Acceptable (do not flag):
const items = data.filter(x => x.active);

// Problem (flag):
const items = data.filter(x => x.active == true); // Use strict equality ===
```

**4. Examples for extraction from different document formats:**

Document with inline citations:

```
"As shown in the study (Smith, 2023), the rate is 42%."
-> {"value": "42%", "source": "Smith, 2023", "type": "inline_citation"}
```

Document with bibliography references:

```
"The rate is 42%. [1]"
-> {"value": "42%", "source": "reference_1", "type": "bibliography"}
```

**5. Examples for informal measurements:**

```
Text: "about two handfuls of rice"
-> {"amount": "~100g", "original_text": "two handfuls", "precision": "approximate"}

Text: "a pinch of salt"
-> {"amount": "~1g", "original_text": "a pinch", "precision": "approximate"}
```

Few-shot is especially effective for extracting informal and non-standard measurement units that are too
diverse for purely rule-based instructions.

Format normalization rules in prompts: When using strict JSON schemas for structured output, add
normalization rules in the prompt:

```
Normalization:
- Dates: always ISO 8601 (YYYY-MM-DD); "yesterday" -> compute an absolute date
- Currency: numeric amount + currency code; "five bucks" -> {"amount": 5, "currency": "USD"}
- Percentages: decimal fraction; "half" -> 0.5
```

This prevents semantic errors where the JSON is syntactically valid but values are inconsistent.

### 6.2 Explicit Criteria vs Vague Instructions

Bad (vague):

```
Check code comments for accuracy.
Be conservative—report only high-confidence findings.
```

Good (explicit criteria):

```
Flag a comment as problematic ONLY if:
1. The comment describes behavior that CONTRADICTS the actual code behavior
2. The comment references a non-existent function or variable
3. A TODO/FIXME comment refers to a bug that has already been fixed in code

Do NOT flag:
- Comments that are merely stylistically outdated
- Comments with minor wording inaccuracies
- Missing comments (that is a separate category)
```

Define severity criteria with examples:

```
CRITICAL: Runtime failure for users
  Example: NullPointerException while processing a payment
HIGH: Security vulnerability
  Example: SQL injection, XSS, missing authorization checks
MEDIUM: Logic bug without immediate impact
  Example: Wrong sorting, off-by-one error
LOW: Code quality
  Example: Duplication, suboptimal algorithm for small data
```

### 6.3 Prompt Chaining

Prompt chaining breaks a complex task into a sequence of focused steps:

```
Step 1: Analyze auth.ts (local issues only)
-> Output: list of issues in auth.ts

Step 2: Analyze database.ts (local issues only)
-> Output: list of issues in database.ts

Step 3: Integration pass (cross-file dependencies)
-> Output: issues at module boundaries
```

Why this matters:

- Avoids attention dilution—when the model receives too many files at once, it may miss bugs in some files while providing shallow commentary on others
- Ensures consistent analysis quality per file
- Allows separate analysis of cross-file interactions

When to use prompt chaining vs dynamic decomposition:

- Prompt chaining — predictable, repeatable tasks (code review, file migrations)
- Dynamic decomposition — open-ended investigations where subtasks become clear only during execution

### 6.4 The "Interview" Pattern

Before implementing a solution, Claude asks clarifying questions:

```
Claude: "Before implementing caching for the API, a few questions:
1. Which cache invalidation strategy do you prefer—TTL or event-based?
2. Is stale data acceptable when the cache is unavailable?
3. Should caching be per-user or global?
4. What is the expected data volume to cache?"
```

When this is useful:

- Unfamiliar domain (fintech, healthcare, legal systems)
- Tasks with non-obvious implications (cache strategies, failure modes)
- Multiple viable approaches where the best choice depends on context

### 6.5 Validation and Retry-with-Feedback

When extracted data fails validation:

```
Step 1: Extract data from the document
Step 2: Validate (Pydantic, JSON Schema, business rules)
Step 3: If there's an error—retry with context:
  - The original document
  - The previous (incorrect) extraction
  - The specific error: "Field 'total' = 150, but sum(line_items) = 145. Re-check values."
```

When retry will be effective:

- Format errors (date in the wrong format)
- Structural errors (a field placed in the wrong location)
- Arithmetic inconsistencies (the model can re-check)

When retry will NOT help:

- The information is absent from the source document
- The required context is external (the data is in another document not provided)

Pydantic as a validation tool: Pydantic is a Python library for schema-based data validation. For the
exam, the key points are:

- Structural validation: types, requiredness, enum constraints checked in code after receiving JSON from Claude
- Semantic validation: custom validators enforce business logic (sum of items equals total; start_date < end_date)
- Validate–retry loops: on Pydantic validation failure, construct an error message and re-prompt Claude with the error context
- JSON Schema generation: Pydantic models can generate JSON Schema for `tool_use`, providing a single source of truth

### 6.6 Self-correction

A pattern for detecting internal contradictions:

```json
{
  "stated_total": "$150.00",
  "calculated_total": "$145.00",
  "conflict_detected": true,
  "line_items": [
    {"name": "Widget A", "price": 75.00},
    {"name": "Widget B", "price": 70.00}
  ]
}
```

The model extracts both the stated value and a computed value—if they differ, `conflict_detected`
allows you to handle the discrepancy.

## Chapter 7: Message Batches API

*Documentation: Message Batches*

### 7.1 Overview

The Message Batches API lets you submit batches of requests for asynchronous processing:

| Attribute | Value |
|---|---|
| Savings | 50% compared to synchronous calls |
| Processing window | Up to 24 hours (no latency SLA guarantee) |
| Multi-turn tool calling | Not supported (one request = one response) |
| Correlation | `custom_id` field to link request and response |

### 7.2 When to Use Batch API vs Synchronous API

| Task | API | Why |
|---|---|---|
| Pre-merge PR check | Synchronous | The developer is waiting; 24 hours is unacceptable |
| Overnight tech-debt report | Batch | Result is needed by morning; 50% savings |
| Weekly security audit | Batch | Not urgent; 50% savings |
| Interactive code review | Synchronous | Immediate response required |
| Processing 10,000 documents | Batch | Bulk processing; savings are significant |

### 7.3 Using custom_id

```json
{
  "custom_id": "doc-invoice-2024-001",
  "params": {
    "model": "claude-sonnet-4-6",
    "max_tokens": 1024,
    "messages": [{"role": "user", "content": "Extract data from: ..."}]
  }
}
```

`custom_id` allows you to:

- Link the result to the original document
- On failure, re-submit only the failed documents
- Avoid re-processing successful documents

### 7.4 Handling Failures in Batches

1. Submit a batch of 100 documents
2. 95 succeed; 5 fail (context limit exceeded)
3. Identify failures by `custom_id`
4. Modify strategy (e.g., split long documents into chunks)
5. Re-submit only the 5 failed documents

### 7.5 SLA Planning

If you need a result in 30 hours and the Batch API can take up to 24 hours:

- Submission window: 30 - 24 = 6 hours
- Batches must be submitted no later than 24 hours before the deadline
- For frequent submissions, split into 4-hour windows

## Chapter 8: Task Decomposition Strategies

### 8.1 Fixed Pipelines (Prompt Chaining)

Each step is defined in advance:

```
Document -> Metadata extraction -> Data extraction -> Validation -> Enrichment -> Final output
```

When to use:

- The task structure is predictable (reviews always follow the same template)
- All steps are known up front
- You need stability and reproducibility

### 8.2 Dynamic Adaptive Decomposition

Subtasks are generated based on intermediate results:

```
1. "Add tests for a legacy codebase"
2. -> First: map the structure (Glob, Grep)
3. -> Found: 3 modules with no tests, 2 with partial coverage
4. -> Prioritize: start with the payments module (high risk)
5. -> During work: discovered a dependency on an external API
6. -> Adapt: add a mock for the external API before writing tests
```

When to use:

- Open-ended investigative tasks
- When the full scope is unknown up front
- When each step depends on the results of the previous step

### 8.3 Multi-pass Code Review

For pull requests with 10+ files:

```
Pass 1 (per-file): Analyze auth.ts -> list local issues
Pass 1 (per-file): Analyze database.ts -> list local issues
Pass 1 (per-file): Analyze routes.ts -> list local issues
...
Pass 2 (integration): Analyze relationships between files
-> Cross-file issues: inconsistent types, circular dependencies
```

Why a single pass over 14 files is bad:

- Attention dilution: deep analysis for some files, shallow for others
- Inconsistent comments: a pattern is flagged in one file but approved in another
- Missed bugs: obvious errors get skipped due to cognitive overload

## Chapter 9: Escalation and Human-in-the-Loop

### 9.1 When to Escalate to a Human

Escalation triggers (clear rules):

| Situation | Action |
|---|---|
| The customer explicitly asks "get me a manager" | Escalate immediately; do not attempt to solve |
| Policy does not cover the request | Escalate (e.g., competitor price matching when policy is silent) |
| The agent cannot make progress | Escalate after a reasonable number of attempts |
| Financial operation above a threshold | Escalate (preferably enforced via a hook, not a prompt) |
| Multiple matches when searching for a customer | Ask for additional identifiers; do not guess |

What is NOT a reliable trigger:

| Unreliable method | Why it fails |
|---|---|
| Sentiment analysis | Customer mood does not correlate with case complexity |
| Model self-rated confidence (1–10) | The model can be confidently wrong; calibration is poor |
| An automatic classifier | Overengineering; may require training data you don't have |

### 9.2 Escalation Patterns

Immediate escalation:

```
Customer: "I want to speak to a manager"
Agent: [immediately calls escalate_to_human]
NOT: "I can help with your issue, let me..."
```

Escalation after an attempt to resolve:

```
Customer: "My refrigerator broke two days after purchase"
Agent: [checks the order, offers a warranty replacement]
If the customer is not satisfied -> escalate
```

Nuanced escalation (acknowledge → resolve → escalate on reiteration):

```
Customer: "This is outrageous, I'm very unhappy with the quality!"
Agent: [acknowledges frustration] "I understand your frustration."
       [offers resolution] "I can offer a replacement or a refund."
Customer: "No, I want to talk to someone!"
Agent: [customer insists again -> immediate escalation]
```

Key principle: acknowledge emotion first, then propose a concrete solution, and only escalate if the
customer reiterates the desire for a human. Do not escalate on the first expression of dissatisfaction (that
is not the same as requesting a manager).

Escalation for a policy gap:

```
Customer: "Competitor X has this item 30% cheaper—give me a discount"
Policy: covers price adjustments only on your own site
Agent: [escalates — policy does not cover competitor price matching]
```

### 9.3 Structured Handoff Protocols

On escalation, the agent should pass a structured summary to a human:

```json
{
  "customer_id": "CUST-12345",
  "customer_name": "Ivan Petrov",
  "issue_summary": "Refund request for a damaged item",
  "order_id": "ORD-67890",
  "root_cause": "Item arrived damaged; photos attached",
  "actions_taken": [
    "Verified customer via get_customer",
    "Confirmed order via lookup_order",
    "Offered a standard replacement — customer insists on a refund"
  ],
  "refund_amount": "$89.99",
  "recommended_action": "Approve a full refund",
  "escalation_reason": "Customer requested to speak with a manager"
}
```

The human operator does not have access to the full conversation transcript—they only see this summary.
Therefore it must be complete and self-contained.

### 9.4 Confidence Calibration and Human Oversight

For data extraction systems:

1. **Field-level confidence scores**: the model outputs a confidence score per extracted field
2. **Calibration**: use labeled validation sets to tune thresholds
3. **Routing**:
   - High confidence + stable accuracy → automated processing
   - Low confidence or ambiguous sources → human review

Stratified random sampling:

- Even for high-confidence extractions, regularly audit a sample
- An aggregate 97% accuracy can hide 40% errors for a particular document type
- Analyze accuracy by document type and by field, not only overall

## Chapter 10: Error Handling in Multi-agent Systems

### 10.1 Error Categories

| Category | Examples | Retryable | Agent action |
|---|---|---|---|
| Transient | Timeout, 503, network failure | Yes | Retry with exponential backoff |
| Validation | Invalid input format, missing required field | No (fix input) | Modify request and retry |
| Business | Policy violation, threshold exceeded | No | Explain to the user; propose an alternative |
| Permission | Access denied | No | Escalate |

### 10.2 Error-handling Anti-patterns

| Anti-pattern | Problem | Correct approach |
|---|---|---|
| Generic status "search unavailable" | The coordinator can't decide how to recover | Return error type, query, partial results, alternatives |
| Silent suppression (empty result = success) | Coordinator thinks there were no matches, but it was a failure | Distinguish "no results" from "search failure" |
| Aborting the whole workflow on one failure | You lose all partial results | Continue with partial results; annotate gaps |
| Infinite retries inside a subagent | Latency and wasted resources | Local recovery (1–2 retries), then propagate to coordinator |

### 10.3 A Structured Subagent Error

```json
{
  "status": "partial_failure",
  "failure_type": "timeout",
  "attempted_query": "AI impact on music industry 2024",
  "partial_results": [
    {"title": "AI Music Generation Report", "url": "...", "relevance": 0.8}
  ],
  "alternative_approaches": [
    "Try a narrower query: 'AI music composition tools'",
    "Use an alternative data source"
  ],
  "coverage_impact": "Not covered: AI impact on music production"
}
```

This provides the coordinator with the information needed to decide:

- Retry with a modified query?
- Use partial results?
- Delegate to a different subagent?
- Continue without this section and annotate the gap?

### 10.4 Coverage Annotations in the Final Synthesis

```markdown
## Report: AI Impact on Creative Industries

### Visual Art (FULL COVERAGE)
[research results]

### Music (PARTIAL COVERAGE — search agent timeout)
[partial results]
⚠️ Note: coverage for this section is limited due to a timeout in the search agent.

### Literature (FULL COVERAGE)
[research results]
```

## Chapter 11: Context Management in Production Systems

### 11.1 Extract Facts into a Separate Block

Instead of relying on conversation history (which degrades during summarization), extract key facts into a
structured block:

```
=== CASE FACTS (updated whenever a new fact appears) ===
Customer ID: CUST-12345
Order ID: ORD-67890
Order Date: 2025-01-15
Order Amount: $89.99
Issue: Damaged item on delivery
Customer Request: Full refund
Status: Pending manager approval
===
```

Include this block in every prompt, regardless of how history is summarized.

### 11.2 Trimming Tool Results

If `lookup_order` returns 40+ fields but you only need 5 for the current task:

```python
# PostToolUse hook: keep only relevant fields
@hook("PostToolUse", tool="lookup_order")
def trim_order_fields(result):
    return {
        "order_id": result["order_id"],
        "status": result["status"],
        "total": result["total"],
        "items": result["items"],
        "return_eligible": result["return_eligible"]
    }
```

This conserves context and reduces noise.

### 11.3 Position-aware Input

Place critical information with the lost-in-the-middle effect in mind:

```
[KEY FINDINGS — at the top]
Found 3 critical vulnerabilities...

[DETAILED RESULTS — middle]
=== File auth.ts ===
...
=== File database.ts ===
...

[ACTION ITEMS — at the end]
Priority: fix auth.ts vulnerabilities before merge.
```

### 11.4 Scratchpad Files

In long investigations, the agent can write key findings to a scratchpad file:

```markdown
# investigation-scratchpad.md
## Key findings
- PaymentProcessor in src/payments/processor.ts inherits from BaseProcessor
- refund() is called from 3 places: OrderController, AdminPanel, CronJob
- External PaymentGateway API has a rate limit of 100 req/min
- Migration #47 added refund_reason (NOT NULL) — 2024-12-01
```

When context degrades (or in a new session), the agent can consult the scratchpad instead of re-running
discovery.

### 11.5 Delegating to Subagents to Protect Context

```
Main agent: "Investigate dependencies of the payments module"
-> Subagent (Explore): reads 15 files, traces imports
-> Returns: "Payments depends on AuthService, OrderModel, and the external PaymentGateway API"
Main agent: keeps one line in context instead of 15 files
```

Separate context layer: In multi-agent systems, each subagent operates within a limited context budget
—it receives only the information required for its task. The coordinator acts as a separate context layer: it
aggregates subagent outputs, stores global state, and allocates context. This prevents "context leakage,"
where one agent consumes the window with information irrelevant to others.

Constrained context budgets for subagents:

- Send minimal context: a specific task + necessary data
- Instruct the subagent to return structured results, not raw data dumps
- Use `allowedTools` to limit the subagent's toolset—fewer tools means fewer distractions and lower context cost

### 11.6 Structured State Persistence (for crash recovery)

Each agent exports its state to a known location:

```json
// agent-state/web-search-agent.json
{
  "status": "completed",
  "queries_executed": ["AI music 2024", "AI music composition"],
  "results_count": 12,
  "key_findings": [],
  "coverage": ["music composition", "music production"],
  "gaps": ["music distribution", "music licensing"]
}
```

The coordinator loads a manifest on resume:

```json
// agent-state/manifest.json
{
  "web-search": "completed",
  "doc-analysis": "in_progress",
  "synthesis": "not_started"
}
```

## Chapter 12: Preserving Provenance

### 12.1 The Attribution Loss Problem

When summarizing results from multiple sources, the "claim → source" link can be lost:

Bad: "The AI music market is estimated at $3.2B." (No source, no year.)

Good:

```json
{
  "claim": "The AI music market is estimated at $3.2B.",
  "source_url": "https://example.com/report",
  "source_name": "Global AI Music Report 2024",
  "publication_date": "2024-06-15",
  "confidence": 0.9
}
```

### 12.2 Handling Conflicting Data

When two sources provide different values:

```json
{
  "claim": "Share of AI-generated music on streaming platforms",
  "values": [
    {
      "value": "12%",
      "source": "Spotify Annual Report 2024",
      "date": "2024-03",
      "methodology": "Automated classification"
    },
    {
      "value": "8%",
      "source": "Music Industry Association Survey",
      "date": "2024-07",
      "methodology": "Survey of 500 labels"
    }
  ],
  "conflict_detected": true,
  "possible_explanation": "Difference in methodology and time period"
}
```

Do not arbitrarily choose one value. Preserve both with attribution and let the coordinator decide.

### 12.3 Include Dates for Correct Interpretation

Without dates, temporal differences can be misinterpreted as contradictions:

Bad: "Source A says 10%, source B says 15%. Contradiction."

Good: "Source A (2023) says 10%, source B (2024) says 15%. Likely +5% growth over a year."

### 12.4 Render by Content Type

Don't force everything into one format:

- Financial data → tables
- News and analysis → prose
- Technical findings → structured lists
- Time series → chronological ordering

## Chapter 13: Claude Code Built-in Tools

### 13.1 Tool Selection Reference

| Task | Tool | Example |
|---|---|---|
| Find files by name/pattern | Glob | `**/*.test.tsx`, `src/components/**/*.ts` |
| Search within files | Grep | Function name, error message, import |
| Read a file in full | Read | Load a file for analysis |
| Write a new file | Write | Create a file from scratch |
| Edit an existing file precisely | Edit | Replace a specific snippet via unique text match |
| Run a shell command | Bash | git, npm, run tests, build |

### 13.2 Incremental Investigation Strategy

Do not read all files at once. Build understanding incrementally:

1. Grep: find entry points (function definition, export)
2. Read: read the found files
3. Grep: find usages (import, calls)
4. Read: read consumer files
5. Repeat until you have a complete picture

### 13.3 Fallback: Read + Write Instead of Edit

When Edit fails due to a non-unique text match:

1. Read — load the full file content
2. Modify the content programmatically
3. Write — write the updated version
