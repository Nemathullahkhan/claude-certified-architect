# Mock Test: Code Generation with Claude Code — Set 2

> Anchored to `scenario-2-code-generation.md`, domain 5.4 (context management in large codebase exploration) and 1.7 (session management), plus 2.5 (built-in tool selection) and 2.x (MCP tool description quality). Focus: exploration strategy under context constraints, subagent delegation, session resume/fork discipline, and tool-selection/description tuning.

---

## Question 1 — Code Generation with Claude Code

Your agent has spent 25 minutes exploring a game engine's rendering subsystem—reading shader code, buffer management, and frame synchronization logic. An engineer now asks it to understand how the physics engine integrates with rendering for collision debug overlays. You notice recent responses reference "typical rendering patterns" rather than the specific `VulkanPipeline` and `FrameGraph` classes it discovered earlier. What's the most effective approach?

- **A.** Spawn a sub-agent to explore physics independently, then manually synthesize its findings with the rendering knowledge accumulated in the main conversation.
- **B.** Continue in the current context with more targeted prompts referencing the specific classes by name.
- **C.** Summarize key rendering findings, then spawn a sub-agent for physics exploration with that summary in its initial context.
- **D.** Use `/clear` to reset context completely, then start fresh with physics exploration using file paths from the project's CLAUDE.md.

---

## Question 2 — Code Generation with Claude Code

An engineer asks the agent to understand how the caching layer works before adding a new cache invalidation trigger. After initial Grep searches, the agent has identified that caching logic spans 15 files including decorators, middleware, and service classes (~8,000 lines total). What's the most effective next step for building understanding while managing context constraints?

- **A.** Use the Read tool to sequentially load all 15 files, building complete understanding across the full caching implementation.
- **B.** Analyze imports and class hierarchies to identify the base cache class, Read that file to understand the interface, then trace specific invalidation implementations.
- **C.** Use Grep to search for "invalidate" and "expire" patterns across all files, then Read only those specific line ranges with minimal surrounding context.
- **D.** Use Glob to find files matching common caching patterns (`cache.py`, `caching/`), prioritize the largest files by reading them first, then check smaller files for gaps.

---

## Question 3 — Code Generation with Claude Code

An engineer asks the agent to find all callers of a function before removing it. The function is defined in a core library but is also exposed through wrapper modules that rename the function for domain-specific use (e.g., `calculateTax` in the library becomes `computeOrderTax` in the orders module). What exploration strategy will most reliably identify all callers?

- **A.** Read the library and wrapper modules to identify all exposed names for the function, then Grep for each name across the codebase.
- **B.** Use Grep to find all files that import from the library or wrapper modules, then read each file to check whether it uses the function.
- **C.** Use Grep to search for the function's original name across the codebase.
- **D.** Search for the function name in project documentation to understand intended usage patterns and navigate to documented integration points.

---

## Question 4 — Code Generation with Claude Code

An engineer used the agent yesterday to analyze a legacy authentication module, identifying two distinct refactoring approaches: extracting a microservice versus refactoring in-place. Today, they want to explore both approaches in depth—having the agent propose specific code changes for each—before deciding which to implement. What's the most effective way to structure this exploration?

- **A.** Resume yesterday's session to explore the first approach, then start a new session for the second, manually recreating the original context.
- **B.** Start two fresh sessions, manually providing a summary of yesterday's analysis findings to establish context.
- **C.** Resume yesterday's session and explore both approaches sequentially within the same conversation thread.
- **D.** Use `fork_session` to create two branches from yesterday's analysis, exploring one approach in each fork.

---

## Question 5 — Code Generation with Claude Code

After integrating a local MCP server providing code analysis tools (`analyze_dependencies`, `find_dead_code`, `calculate_complexity`), you verify the server is healthy and tools appear in the `tools/list` response. However, you observe that the agent consistently uses Grep to search for import statements instead of calling `analyze_dependencies`—even when users explicitly ask about "code dependencies." Examining tool definitions reveals:

- MCP: `analyze_dependencies` — "Analyzes dependency graph"
- Built-in: Grep — "Search file contents for a pattern using regular expressions. Returns matching lines with line numbers and surrounding context."

What's the most effective approach to improve the agent's selection of MCP tools?

- **A.** Remove Grep from available tools when the MCP server is connected to eliminate functional overlap.
- **B.** Add routing instructions to the system prompt specifying that dependency-related questions should use MCP tools rather than Grep.
- **C.** Split `analyze_dependencies` into granular tools (`list_imports`, `resolve_transitive_deps`, `detect_circular_deps`) so each has a focused purpose less likely to overlap with Grep.
- **D.** Expand MCP tool descriptions to detail capabilities and outputs—e.g., "Builds dependency graph showing direct imports, transitive dependencies, and cycles."

---

## Question 6 — Code Generation with Claude Code

An engineer asks your agent to identify untested code paths in a legacy payment processing module spanning 45 files. After reading the first 8 source files, the agent's responses are becoming noticeably less accurate—it's forgetting previously discussed code patterns and hasn't yet located all test files or traced critical payment flows. What's the most effective approach to complete this investigation?

- **A.** Document all current findings in a summary report, clear context completely, then use that report as the sole reference for continuing the investigation.
- **B.** Spawn subagents to investigate specific questions (e.g., "find all test files for payment processing", "trace refund flow dependencies") while the main agent coordinates findings and preserves high-level understanding.
- **C.** Clear context with `/clear`, then selectively re-read only the most critical files discovered so far, writing key findings to a scratchpad file that persists between context resets.
- **D.** Switch to using Grep to search for specific function names instead of reading full files, reducing the content loaded into context for remaining exploration.

---

## Question 7 — Code Generation with Claude Code

Your codebase exploration tool stores session IDs to allow engineers to continue investigations across work sessions. An engineer spent an hour yesterday analyzing a legacy authentication module, building context about its architecture and dependencies. They want to continue today. The session ID is valid, but version control shows 3 of the 12 files the agent previously read were modified overnight by a teammate's merge. What approach best balances efficiency and accuracy?

- **A.** Resume the session without informing the agent about the changed files.
- **B.** Start a fresh session to ensure the agent works with current codebase state without stale assumptions.
- **C.** Resume the session and inform the agent which specific files changed for targeted re-analysis.
- **D.** Resume the session and immediately have the agent re-read all 12 previously analyzed files.

---

## Question 8 — Code Generation with Claude Code

An engineer's exploration subagent spent 30 minutes analyzing a legacy payment system, reading 47 files and documenting data flows. The session was interrupted when the engineer's connection dropped. While away, a teammate merged a PR that renamed two utility functions. The engineer wants to continue the same exploration. What's the most effective approach?

- **A.** Resume the subagent from its previous transcript without mentioning the changes—the architecture understanding remains valid.
- **B.** Launch a fresh subagent and include the prior transcript in the initial prompt for context.
- **C.** Launch a fresh subagent with a summary of prior findings.
- **D.** Resume the subagent from its previous transcript and inform it about the renamed functions.

---

## Question 9 — Code Generation with Claude Code

Your agent needs to insert a new helper function into the middle of a 150-line utility module, between two existing functions. The Edit tool fails because its `old_string` parameter cannot find unique text to match — the file has repetitive docstrings, variable names, and structural patterns. What's the most reliable way to complete this insertion?

- **A.** Use Edit with an extremely long `old_string` capturing 30+ lines of context to guarantee uniqueness.
- **B.** Use Edit's `replace_all` parameter to target a common pattern and embed the new function in the replacement text.
- **C.** Use Bash to append the function definition to the end of the file using heredoc syntax.
- **D.** Use Read to load the file, add the function at the appropriate location, then Write the updated file.

---

## Question 10 — Code Generation with Claude Code

A developer asks the agent to investigate why a specific API endpoint intermittently returns 500 errors. The codebase has 200+ files and the developer doesn't know which components are involved. The agent must trace the error through routing, middleware, business logic, and database layers. What task decomposition approach would be most effective?

- **A.** Have the agent first create a comprehensive plan mapping all code paths through the endpoint before beginning any file exploration or code reading.
- **B.** Have the agent dynamically generate investigation subtasks based on what it discovers at each step, adapting its exploration plan as new information about the error path emerges.
- **C.** Define a fixed sequence of investigation steps upfront—grep for error patterns, then read error handlers, then check database queries, then examine middleware—executing each step regardless of intermediate findings.
- **D.** Run parallel worker agents that simultaneously investigate all four layers, then synthesize their findings to identify where the error originates.

---

## Question 11 — Code Generation with Claude Code

Your agent has analyzed a complex service module—reading 23 source files, tracing request flows, and identifying error handling patterns. A developer wants to compare two testing strategies before committing to one: end-to-end tests with mocked external services vs. snapshot tests capturing expected outputs. They need to independently develop both approaches to evaluate trade-offs. How should you manage the sessions?

- **A.** Export the analysis session's key findings to a file, then create two new sessions that reference this file.
- **B.** Resume the analysis session with `fork_session` enabled, creating a separate branch for each testing strategy.
- **C.** Start two fresh sessions, having each re-read the relevant source files before beginning.
- **D.** Continue in the original session, developing end-to-end tests first, then snapshot tests sequentially.

---

## Question 12 — Code Generation with Claude Code

An engineer used Claude Code yesterday to investigate authentication flows in a legacy monolith, building up significant context over a 2-hour session. Today she wants to continue that specific investigation. She's worked on three other codebases since then and knows the session was named "auth-deep-dive". How should she resume?

- **A.** Start fresh and re-read the same files.
- **B.** Use `--session-id` with the UUID from yesterday's session transcript file.
- **C.** Use `--continue` to pick up where the most recent conversation left off.
- **D.** Use `--resume auth-deep-dive` to load that specific session by name.

---

## Question 13 — Code Generation with Claude Code

During testing, you observe that in extended exploration sessions (30+ minutes), the agent starts giving inconsistent answers about code structure it discussed earlier. Engineers report having to repeat context about modules they've already explored. What's the most effective approach to address this?

- **A.** Have the agent maintain a scratchpad file that records key findings, referencing it for subsequent questions.
- **B.** Switch to a higher-capacity model tier to provide more context window space for accumulated exploration data.
- **C.** Implement automatic context clearing every 15 minutes to ensure the agent starts with fresh, uncontaminated context.
- **D.** Create summaries of all source files before exploration begins, loading only these compressed representations into context.

---

## Question 14 — Code Generation with Claude Code

After adding an MCP server with specialized code refactoring tools (`extract_function`, `rename_variable`, `inline_function`), you notice the agent still uses basic text manipulation via Write and Bash `sed` commands for refactoring tasks. The MCP server is connected and healthy. Examining the configuration, you find each MCP tool has a minimal description like "`extract_function`: extracts a function from code." What's the most effective way to improve adoption of the MCP refactoring tools?

- **A.** Implement a request classifier that detects refactoring intent and automatically routes those requests to the MCP server before the agent processes them.
- **B.** Remove the Write tool from the agent's configuration for refactoring sessions so it must use the MCP tools for code modifications.
- **C.** Accept this as expected behavior since simpler tools like `sed` are more predictable than specialized refactoring tools.
- **D.** Enhance the MCP tool descriptions to explain when each tool is preferable to text manipulation and clarify expected inputs and outputs.

---

## Question 15 — Code Generation with Claude Code

An engineer who just joined the team asks the agent to help them understand the authentication and authorization architecture before making security improvements. The codebase has 800+ files across multiple services. What exploration strategy will most effectively build understanding, given Claude's built-in tools and context limits?

- **A.** Read any CLAUDE.md and README files first, then ask the engineer to specify which 10-15 files are most important for understanding the auth system.
- **B.** Launch parallel subagents to explore different services simultaneously, then synthesize their findings into an architectural overview.
- **C.** Use Grep to find authentication entry points, read those files, then follow imports and function calls to map the auth flow incrementally.
- **D.** Read all files containing "auth", "login", "permission", or "token" in their content or filename.

---

## Answer Key

**Q1: C** — The context has already degraded ("typical rendering patterns" instead of the specific classes found earlier), so continuing in the same context (B) inherits that degradation into a *new* topic (physics). A summary handed to a fresh sub-agent carries forward only the distilled, accurate findings without the accumulated noise. A skips the summarization step, risking the sub-agent starting with nothing useful to connect physics to rendering. D throws away everything, including hard-won findings that don't need to be rediscovered.

**Q2: B** — Building understanding of a large, structurally organized subsystem (decorators, middleware, service classes) should start from the architectural entry point (the base class) and expand outward through actual usage, not brute-force reading (A), line-isolated keyword hits stripped of surrounding structure (C), or a size-based reading heuristic that ignores the actual class hierarchy (D).

**Q3: B** — When a function is deliberately renamed by wrapper layers, you cannot assume you know every alias in advance (there could be further renaming down the chain, e.g., a third module wrapping `computeOrderTax` under yet another name). Tracing every file that *imports* from the library or its wrappers and checking actual usage is exhaustive by construction — it doesn't depend on correctly guessing or discovering every renamed variant upfront. A only catches names you've already found by reading the wrappers you happened to check; if there's a deeper rename chain, A silently misses callers. C misses every renamed caller entirely. D relies on documentation being complete and current, which is not guaranteed.

**Q4: D** — Two approaches that need independent, in-depth exploration from the *same* analytical starting point is exactly what `fork_session` is for — both branches inherit yesterday's authentication-module analysis without redoing it, and each can diverge without polluting the other. A and C interleave or contaminate the two explorations in a single thread. B discards the actual session context and substitutes a manually-written summary, losing fidelity and duplicating work across two fresh sessions.

**Q5: D** — The tools/list confirms the server is healthy and connected — this isn't a wiring/connectivity problem, it's a **selection** problem caused by a vague MCP description ("Analyzes dependency graph") next to a much richer built-in description for Grep. Expanding the MCP description to explain what it actually returns (direct/transitive imports, cycles) gives the model the information it needs to prefer it correctly. A removes a generally useful tool just to force routing. B patches the symptom with brittle prompt-level routing instructions rather than fixing the actual root cause (a bad description). C adds unnecessary tool proliferation without addressing why the model didn't choose the original tool in the first place.

**Q6: B** — Context degradation ("forgetting previously discussed code patterns") mid-investigation, with more ground still to cover (45 files, only 8 read), is the classic case for subagent delegation: spin off targeted, bounded sub-investigations while the main agent retains high-level coordination and doesn't accumulate all the raw exploration output itself. A throws away findings entirely and forces a full context wipe. C is a heavier, less structured version of the same clearing approach. D changes tool choice but doesn't solve the accumulation-of-context problem for the investigation already underway.

**Q7: C** — This is the standard "stale session with known specific changes" pattern: resume (preserve the hour of accumulated architectural understanding) and explicitly tell the agent which files changed so it can do a *targeted* re-analysis rather than either working on stale assumptions (A) or discarding an hour of valid analysis unnecessarily (B), or wastefully re-reading all 12 files including the 9 that are still accurate (D).

**Q8: D** — Only two specific, known facts changed (two renamed utility functions) — resuming the transcript preserves the 30 minutes of valid analysis, and explicitly informing the agent of the renames lets it correct just the affected parts without rebuilding everything. A silently carries a factual error forward. B and C discard the actual transcript in favor of a fresh subagent, which is less efficient than a targeted correction to an otherwise-valid transcript.

**Q9: D** — When Edit cannot find a unique anchor in a file full of repetitive structure, Read + Write is the reliable fallback: load the full file, insert the function at the correct location in memory, and write it back — deterministic regardless of how repetitive the surrounding text is. A may still fail or become fragile if the "unique" 30-line block coincidentally repeats elsewhere. B (`replace_all`) is designed for replacing *every* match, not a single targeted insertion — using it here risks corrupting multiple locations. C bypasses the intended insertion point entirely, appending to the end instead of placing the function where requested.

**Q10: B** — With 200+ files and no upfront knowledge of which components are involved, an intermittent error's root cause is genuinely unpredictable in advance — the most effective decomposition adapts as each step reveals where to look next. A commits to a comprehensive plan before any real evidence exists, which is premature given the unknown scope. C locks in a fixed investigation order regardless of what's found, which can waste effort chasing layers irrelevant to this specific error. D parallelizes across all four layers simultaneously without letting early findings (e.g., "the error only happens under a specific load condition in middleware") focus the remaining investigation, which is likely to be less efficient and harder to synthesize than a single adaptive thread.

**Q11: B** — Comparing two independent strategies from a shared analytical baseline (23 files, request flows, error patterns already understood) is exactly what `fork_session` supports — each strategy explores independently without redoing the 23-file analysis and without the two efforts contaminating each other. A substitutes a written summary for the actual session context, losing fidelity. C throws away the completed analysis and duplicates the file-reading work twice. D interleaves both strategies in one thread, risking cross-contamination between the two approaches under evaluation.

**Q12: D** — She knows the specific session **name** ("auth-deep-dive"), not the UUID, so `--resume auth-deep-dive` is the direct, correct mechanism. `--continue` (C) resumes the *most recent* session — but she's worked on three other codebases since, so the most recent session is not the one she wants. Looking up the UUID from a transcript file (B) works but is an unnecessary, indirect extra step when the name-based resume is available and she already has the name. Starting fresh (A) discards two hours of valid, still-relevant investigation.

**Q13: A** — "Inconsistent answers about code structure discussed earlier" during long sessions is the direct symptom of context degradation; a scratchpad file that persistently records key findings gives the agent (and the engineer) a reliable, referenceable record that survives regardless of how the model's in-context recall degrades. B treats the symptom (running out of room) rather than the actual failure mode (degraded recall/accuracy over a long session, which isn't purely a capacity problem). C is a blunt overcorrection that discards legitimately useful context on a fixed timer regardless of task state. D compresses *before* exploration even begins, which doesn't address findings that accumulate *during* the actual investigation.

**Q14: D** — The MCP server is healthy and connected — the adoption problem is a **description quality** problem, not a routing, access, or capability problem. Minimal descriptions ("extracts a function from code") give the model no signal about when the specialized tool is preferable to a familiar general-purpose one like `sed`. A adds an external classifier layer to route around a fixable root cause. B removes a broadly useful tool entirely, which is a blunt and disruptive workaround. C accepts a symptom as "expected" without addressing the fixable underlying cause.

**Q15: C** — Starting from Grep-located authentication entry points and incrementally following imports/calls builds an accurate, evidence-based mental model without loading the entire 800+ file codebase into context. A defers the real work back onto the (new) engineer, who by definition doesn't yet know the codebase well enough to name the right 10-15 files. B parallelizes exploration before establishing even a basic map of how the services relate, risking redundant or disconnected findings. D is an unbounded, keyword-based reading strategy that would pull in many irrelevant files (e.g., unrelated "token" matches) while still not guaranteeing full coverage of the actual auth flow.
