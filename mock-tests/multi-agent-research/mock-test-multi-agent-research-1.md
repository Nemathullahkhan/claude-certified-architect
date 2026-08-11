# Mock Test: Multi-Agent Research System (MAR) — Set 1

> Anchored to `scenario-3-multi-agent-research.md`. Covers coordinator/subagent hub-and-spoke discipline, task decomposition and scope partitioning, error propagation and local recovery, tool interface design and least-privilege scoping, tool distribution across agents, and context management in synthesis (lost-in-the-middle, structured findings, token budget). Distractors are intentionally tempting — read every stem twice before answering.

---

## Question 1 — Multi-Agent Research System

A document analysis agent discovers that two credible sources contain directly contradictory statistics for a key metric: a government report states 40% growth, while an industry analysis states 12%. Both sources look credible, and the discrepancy could materially affect the research conclusions.

**Which approach is most effective?**

- **A.** Apply credibility heuristics to pick the most likely correct number, finish analysis with that value, and add a footnote mentioning the discrepancy.
- **B.** Include both numbers in the analysis output without marking them as conflicting, letting the synthesis agent decide which to use based on broader context.
- **C.** Stop analysis and immediately escalate to the coordinator, asking it to decide which source is more authoritative before continuing.
- **D.** Complete analysis with both numbers, explicitly annotate the conflict with source attribution, and let the coordinator decide how to reconcile the data before passing to synthesis.

---

## Question 2 — Multi-Agent Research System

The web-search and document-analysis agents have completed their tasks and returned results to the coordinator.

**Which next step is most appropriate for creating an integrated research report?**

- **A.** Each agent sends its results directly to the report-writing agent, bypassing the coordinator.
- **B.** The document analysis agent requests web-search results and merges them internally.
- **C.** The coordinator passes both sets of results to the synthesis agent for a unified integration.
- **D.** The coordinator concatenates the raw outputs from both agents and returns them as the final result.

---

## Question 3 — Multi-Agent Research System

A document analysis subagent frequently fails when processing PDF files: some have corrupted sections that trigger parsing exceptions, others are password-protected, and sometimes the parsing library hangs on large files. Currently, any exception immediately terminates the subagent and returns an error to the coordinator, which must decide whether to retry, skip, or fail the whole task. This causes excessive coordinator involvement in routine error handling.

**Which improvement is most effective?**

- **A.** Create a dedicated error-handling agent that monitors all failures via a shared queue and decides recovery actions, sending restart commands directly to subagents.
- **B.** Configure the subagent to always return partial results with a success status, embedding error details in metadata; the coordinator treats all responses as successful.
- **C.** Make the coordinator validate all documents before sending them to the subagent, rejecting documents that might cause failures.
- **D.** Implement local recovery in the subagent for transient failures and escalate to the coordinator only errors it cannot resolve, including attempted steps and partial results.

---

## Question 4 — Multi-Agent Research System

After running the system on "AI impact on creative industries," you observe that every subagent completes successfully: the web-search agent finds relevant articles, the document analysis agent summarizes them correctly, and the synthesis agent produces coherent text. However, final reports cover only visual art and completely miss music, literature, and film. In the coordinator logs, you see it decomposed the topic into three subtasks: "AI in digital art," "AI in graphic design," and "AI in photography."

**What is the most likely root cause?**

- **A.** The synthesis agent lacks instructions to detect coverage gaps.
- **B.** The document analysis agent filters out non-visual sources due to overly strict relevance criteria.
- **C.** The coordinator's task decomposition is too narrow, assigning subagents work that does not cover all relevant areas.
- **D.** The web-search agent's queries are insufficient and should be broadened to cover more sectors.

---

## Question 5 — Multi-Agent Research System

The web-search subagent returns results for only 3 of 5 requested source categories (competitor sites and industry reports succeed, but news archives and social feeds time out). The document analysis subagent successfully processes all provided documents. The synthesis subagent must produce a summary from mixed-quality upstream inputs.

**Which error-propagation strategy is most effective?**

- **A.** Continue synthesis using only successful sources and produce an output without mentioning which data was unavailable.
- **B.** The synthesis subagent returns an error to the coordinator, triggering a full retry or task failure due to incomplete data.
- **C.** The synthesis subagent asks the coordinator to retry timed-out sources with a longer timeout before starting synthesis.
- **D.** Structure the synthesis output with coverage annotations that indicate which conclusions are well-supported and where gaps exist due to unavailable sources.

---

## Question 6 — Multi-Agent Research System

The document analysis subagent encounters a corrupted PDF file that it cannot parse. When designing the system's error handling, what is the most effective way to handle this failure?

**Which approach is most effective?**

- **A.** Return an error with context to the coordinator agent, allowing it to decide how to proceed.
- **B.** Silently skip the corrupted document and continue processing the remaining files to avoid interrupting the workflow.
- **C.** Automatically retry parsing the document three times with exponential backoff before reporting a failure.
- **D.** Throw an exception that terminates the entire research workflow.

---

## Question 7 — Multi-Agent Research System

Production logs show a persistent pattern: requests like "analyze the uploaded quarterly report" are routed to the web-search agent 45% of the time instead of the document analysis agent. Reviewing tool definitions, you find that the web-search agent has a tool `analyze_content` described as "analyzes content and extracts key information," while the document analysis agent has a tool `analyze_document` described as "analyzes documents and extracts key information."

**How should you fix the misrouting problem?**

- **A.** Add a pre-routing classifier that detects whether the user refers to uploaded files or web content before the coordinator decides on delegation.
- **B.** Rename the web-search tool to `extract_web_results` and update its description to "processes and returns information retrieved from web search and URLs."
- **C.** Add few-shot examples to the coordinator prompt showing correct routing: "User uploads a quarterly report → document analysis agent" and "User asks about a web page → web-search agent."
- **D.** Expand the document analysis tool description with usage examples like "Use for uploaded PDFs, Word docs, and spreadsheets," leaving the web-search tool unchanged.

---

## Question 8 — Multi-Agent Research System

A colleague proposes that the document analysis agent should send its results directly to the synthesis agent, bypassing the coordinator.

**What is the main advantage of keeping the coordinator as the central hub for all communication between subagents?**

- **A.** The coordinator can observe all interactions, handle errors uniformly, and decide what information each subagent should receive.
- **B.** The coordinator batches multiple requests to subagents, reducing total API calls and overall latency.
- **C.** Routing through the coordinator enables automatic retry logic that direct inter-agent calls cannot support.
- **D.** Subagents use isolated memory, and direct communication would require complex serialization that only the coordinator can perform.

---

## Question 9 — Multi-Agent Research System

The web-search subagent times out while researching a complex topic. You need to design how information about this failure is returned to the coordinator.

**Which error-propagation approach best enables intelligent recovery?**

- **A.** Return structured error context to the coordinator including the failure type, the query executed, any partial results, and potential alternative approaches.
- **B.** Catch the timeout within the subagent and return an empty result set marked as successful.
- **C.** Implement automatic exponential-backoff retries inside the subagent, only returning a generic "search unavailable" status after exhausting retries.
- **D.** Propagate the timeout exception directly to the top-level handler, terminating the entire research workflow.

---

## Question 10 — Multi-Agent Research System

In your system design, you gave the document analysis agent access to a general-purpose tool `fetch_url` so it could download documents by URL. Production logs show this agent now frequently downloads search engine results pages to perform ad hoc web search — behavior that should be routed through the web-search agent — causing inconsistent results.

**Which fix is most effective?**

- **A.** Replace `fetch_url` with a `load_document` tool that validates that URLs point to document formats.
- **B.** Remove `fetch_url` from the document analysis agent and route all URL fetching through the coordinator to the web-search agent.
- **C.** Implement filtering that blocks `fetch_url` calls to known search engine domains while allowing other URLs.
- **D.** Add instructions to the document analysis agent prompt that `fetch_url` should only be used to download document URLs, not to search.

---

## Question 11 — Multi-Agent Research System

While researching a broad topic, you observe that the web-search agent and the document analysis agent investigate the same subtopics, leading to substantial duplication in their outputs. Token usage nearly doubles without a proportional increase in research breadth or depth.

**What is the most effective way to address this?**

- **A.** Allow both agents to finish in parallel, then have the coordinator deduplicate overlapping results before passing them to the synthesis agent.
- **B.** The coordinator explicitly partitions the research space before delegating, assigning each agent distinct subtopics or source types.
- **C.** Implement a shared-state mechanism where agents log their current focus area so other agents can dynamically avoid duplication during execution.
- **D.** Switch to sequential execution where document analysis runs only after web search completes, using web-search results as context to avoid duplication.

---

## Question 12 — Multi-Agent Research System

During research, the web-search subagent queries three source categories with different outcomes: academic databases return 15 relevant papers, industry reports return "0 results," and patent databases return "Connection timeout."

**Which approach enables the best recovery decisions?**

- **A.** Aggregate the results into a single success-percentage metric (e.g., "67% source coverage") with detailed logs available on demand.
- **B.** Report both "timeout" and "0 results" as failures requiring coordinator intervention.
- **C.** Retry transient failures internally and report only persistent errors.
- **D.** Distinguish access failures (timeout) that require a retry decision from valid empty results ("0 results") that represent successful queries.

---

## Question 13 — Multi-Agent Research System

Production monitoring shows inconsistent synthesis quality. When aggregated results are ~75K tokens, the synthesis agent reliably cites information from the first 15K tokens (web-search headlines/snippets) and the last 10K tokens (document analysis conclusions), but often misses critical findings in the middle 50K tokens — even when they directly answer the research question.

**How should you restructure the aggregated input?**

- **A.** Summarize all subagent outputs to under 20K tokens before aggregation to keep content within the model's reliable processing range.
- **B.** Stream subagent results to the synthesis agent incrementally, processing web-search results first to completion, then adding document analysis results.
- **C.** Place a key-findings summary at the start of the aggregated input and organize detailed results with explicit section headings for easier navigation.
- **D.** Implement rotation that alternates which subagent's results appear first across research tasks to ensure both sources get equal top positioning over time.

---

## Question 14 — Multi-Agent Research System

In testing, the combined output of the web-search agent (85K tokens including page content) and the document analysis agent (70K tokens including chains of thought) totals 155K tokens, but the synthesis agent performs best with inputs under 50K tokens.

**Which solution is most effective?**

- **A.** Modify upstream agents to return structured data (key facts, quotes, relevance scores) instead of verbose content and reasoning.
- **B.** Add an intermediate summarization agent that condenses findings before passing them to synthesis.
- **C.** Have the synthesis agent process findings in sequential batches, maintaining state between calls.
- **D.** Store findings in a vector database and give the synthesis agent search tools to query during its work.

---

## Question 15 — Multi-Agent Research System

In testing, you observe that the synthesis agent often needs to verify specific claims while merging results. Currently, when verification is needed, the synthesis agent returns control to the coordinator, which calls the web-search agent and then re-invokes synthesis with the results. This adds 2–3 extra loops per task and increases latency by 40%. Your assessment shows 85% of these verifications are simple fact checks (dates, names, stats) and 15% require deeper research.

**Which approach most effectively reduces overhead while preserving system reliability?**

- **A.** Give the synthesis agent access to all web-search tools so it can handle any verification need directly without coordinator loops.
- **B.** Have the synthesis agent accumulate all verification needs and return them as a batch to the coordinator at the end, which then sends them all to the web-search agent at once.
- **C.** Have the web-search agent proactively cache extra context around each source during initial research in anticipation of synthesis needing verification.
- **D.** Give the synthesis agent a limited-scope `verify_fact` tool for simple checks, while routing complex verifications through the coordinator to the web-search agent.

---

## Answer Key

**Q1: D** — This preserves separation of responsibilities: the analysis agent completes its core work without blocking, preserves both conflicting values with clear attribution, and correctly passes reconciliation to the coordinator, which has broader context. A silently resolves the conflict behind a footnote; B fails to flag the conflict at all; C blocks progress unnecessarily on a decision the analysis agent doesn't need to make in order to finish its own work.

**Q2: C** — In a coordinator–subagent architecture, the coordinator forwards both result sets to the synthesis agent for centralized integration, preserving control and ensuring high-quality merging. A and B both route around the coordinator; D skips synthesis entirely and just concatenates raw output.

**Q3: D** — Handle errors at the lowest level capable of resolving them. Local recovery reduces coordinator workload while still escalating truly unrecoverable issues with full context and partial progress. A adds unnecessary new infrastructure; B hides real failures behind a fake success status; C tries to prevent every possible failure mode upfront, which is brittle and can't anticipate issues like a parsing library hang.

**Q4: C** — The coordinator decomposed a broad topic only into visual-art subtasks, missing music, literature, and film entirely. Since subagents executed their assignments correctly, the narrow decomposition is the obvious root cause. A, B, and D all misdiagnose a decomposition problem as a downstream agent problem.

**Q5: D** — Coverage annotations implement graceful degradation with transparency, preserving value from completed work while propagating uncertainty to enable informed decisions about confidence. A hides the gap entirely; B discards otherwise-usable partial results; C delays synthesis for sources that may not be worth retrying at all.

**Q6: A** — Returning an error with context to the coordinator is the most effective approach because it lets the coordinator make an informed decision — skip the file, try an alternative parsing method, or notify the user — while maintaining visibility into the failure. B and D both remove that visibility (silently or catastrophically); C is a reasonable local step but doesn't address what happens after retries are exhausted.

**Q7: B** — Renaming the web-search tool to `extract_web_results` and updating its description to explicitly reference web search and URLs directly removes the root cause by eliminating semantic overlap between the two tool names and descriptions. A and C add compensating infrastructure/instructions around the actual problem. D only fixes one side of the overlapping pair — the web-search tool's confusable name and description remain untouched, so the ambiguity persists.

**Q8: A** — The coordinator pattern provides centralized visibility into all interactions, uniform error handling across the system, and fine-grained control over what information each subagent receives — these are the primary advantages of a star-shaped communication topology. B, C, and D describe secondary or fabricated benefits that aren't the core architectural reason for the hub-and-spoke design.

**Q9: A** — Returning structured error context — including failure type, executed query, partial results, and alternative approaches — gives the coordinator everything needed to make intelligent recovery decisions. It preserves maximum context for informed coordination-level decision-making. B hides the failure as success; C strips context down to a generic status after retries; D escalates disproportionately by killing the whole workflow.

**Q10: A** — Replacing a general-purpose tool with a document-specific tool that validates URLs against document formats fixes the root cause by constraining capability at the interface level. This follows the principle of least privilege, making undesired search behavior impossible rather than merely discouraged. B removes the capability entirely and adds coordinator round-trips for legitimate document URL fetching; C is a brittle domain-blocklist that new search engines or aggregators can evade; D is a probabilistic prompt instruction layered on top of a tool that still has more capability than the agent needs.

**Q11: B** — Having the coordinator explicitly partition the research space before delegating is most effective because it addresses the root cause — unclear task boundaries — before any work begins. It preserves parallelism while preventing duplicated effort and wasted tokens. A tolerates the waste and cleans up after the fact; C adds real-time coordination complexity between agents that should stay isolated; D sacrifices parallelism entirely.

**Q12: D** — A timeout (access failure) and "0 results" (valid empty result) are semantically different outcomes requiring different responses. Distinguishing them allows the coordinator to retry the patent database while accepting the industry reports "0 results" as a valid, informative finding. A buries the distinction inside a single aggregate metric; B treats a legitimate, informative result as a failure requiring unnecessary intervention; C is a reasonable partial step but doesn't itself establish the actual distinction the coordinator needs to see.

**Q13: C** — Putting a key-findings summary at the start leverages primacy effects so critical information sits in the most reliably processed position. Adding explicit section headings throughout helps the model navigate and attend to mid-input content, directly mitigating the "lost in the middle" phenomenon. A loses detail through premature summarization; B doesn't address position effects at all, just ordering; D is an arbitrary rotation that doesn't structurally fix the underlying attention pattern.

**Q14: A** — Modifying upstream agents to return structured data fixes the root cause by reducing token volume at the source while preserving essential information. It avoids passing bulky page content and reasoning traces that inflate tokens without improving the synthesis step. B, C, and D all add architecture around the bloated output instead of fixing what the upstream agents actually return.

**Q15: D** — A limited-scope fact-verification tool lets the synthesis agent handle 85% of simple checks directly, eliminating most loops, while preserving the coordinator delegation path for the 15% of complex verifications. This applies least privilege while significantly reducing latency. A over-grants full web-search capability for what's mostly simple fact-checking; B just delays the same round-trips instead of eliminating most of them; C speculatively front-loads work that may not end up being needed.
