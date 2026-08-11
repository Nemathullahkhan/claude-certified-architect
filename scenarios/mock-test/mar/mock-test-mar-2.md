# Mock Test: Multi-Agent Research System (MAR) — Set 2

> Anchored to `scenario-3-multi-agent-research.md`. Covers subagent context isolation, dynamic vs. fixed routing, parallel spawning for latency, tool-splitting for reliability, structured claim-source mapping and citation fidelity, temporal provenance, confidence/uncertainty handling in synthesis, and crash-recovery state management. Distractors are intentionally tempting — read every stem twice before answering.

---

## Question 1 — Multi-Agent Research System

After the web search agent and document analysis agent complete their tasks, the coordinator invokes the synthesis agent. However, the synthesis agent responds that it cannot complete the task because no research findings were provided.

**What is the most likely cause of this issue?**

- **A.** The synthesis agent's context window is not large enough to hold the combined outputs from both previous agents.
- **B.** The coordinator did not include the outputs from the previous agents in the synthesis agent's prompt.
- **C.** The subagents need to share a single API connection to enable automatic context sharing between invocations.
- **D.** The synthesis agent needs tools that can fetch results directly from the other agents' conversation histories.

---

## Question 2 — Multi-Agent Research System

A user is expanding the research system beyond its single web search agent by adding specialized data sources. They add a financial API agent that returns structured JSON with revenue, margins, and growth rates; a news monitoring agent that returns prose summaries of recent developments; and a patent analysis agent that returns structured lists of technology areas. The synthesis agent combines these into executive briefings. Currently, it converts everything to bullet points, causing financial comparisons to lose tabular clarity and news summaries to lose narrative flow.

**What change would most improve briefing quality?**

- **A.** Standardize all subagent outputs to prose summaries with inline citations.
- **B.** Add a format conversion layer between subagents and synthesis that transforms all outputs to a common intermediate representation.
- **C.** Update the synthesis agent to render each content type appropriately — financial data as tables, news as prose.
- **D.** Standardize all subagent outputs to JSON with fields for claim, evidence, source, and confidence.

---

## Question 3 — Multi-Agent Research System

The coordinator provides detailed step-by-step instructions to the web search subagent, specifying exact search queries, source priorities, and date filters. Production monitoring reveals three issues: (1) the subagent reports "insufficient results" rather than trying alternative approaches when pre-specified searches fail, (2) research quality drops for emerging topics that don't match expected patterns, and (3) the subagent rarely surfaces valuable tangential sources.

**What's the most effective way to improve subagent adaptability?**

- **A.** Remove procedural details entirely, delegating with simple goals like "research X thoroughly" and relying on the subagent's general capabilities.
- **B.** Add explicit fallback directives to the detailed instructions: "If specified searches yield fewer than N results, attempt alternative query formulations before reporting failure."
- **C.** Implement a topic classification step where the coordinator categorizes requests as "well-defined" or "exploratory" and uses different instruction styles for each category.
- **D.** Specify research goals and quality criteria (coverage breadth, source diversity, recency) rather than procedural steps, letting the subagent determine its search strategy.

---

## Question 4 — Multi-Agent Research System

Production monitoring shows that follow-up queries like "summarize what we learned about market trends" consistently take 40+ seconds. Investigation reveals the coordinator spawns the synthesis subagent for each summarization request, passing 80K+ tokens of accumulated findings. The coordinator already has these findings in its context from orchestrating the research.

**What's the most effective way to improve response time for these follow-up summaries?**

- **A.** Pre-generate and cache summaries at multiple granularities whenever new findings accumulate.
- **B.** Have the coordinator handle straightforward summarization requests directly using its existing context, reserving subagent spawning for complex analysis.
- **C.** Enable prompt caching on the synthesis subagent to reduce the overhead of repeatedly transferring the same research findings.
- **D.** Spawn the synthesis subagent with reduced context and have it request specific findings from the coordinator on-demand.

---

## Question 5 — Multi-Agent Research System

When analyzing complex legal cases that cite multiple precedents, the document analysis subagent processes each sequentially. A landmark case citing 12 precedents takes over 3 minutes to analyze completely.

**What's the most effective way to reduce this latency while preserving the coordinator's ability to monitor and debug the system?**

- **A.** Implement a message queue where precedent analysis tasks are processed asynchronously by a pool of worker agents.
- **B.** Create a recursive agent hierarchy where analysis agents subdivide work among child agents until reaching single-precedent granularity.
- **C.** Have the coordinator spawn parallel document analysis subagents, each handling a subset of precedents, then aggregate results before synthesis.
- **D.** Enable the document analysis subagent to spawn its own specialized subagents dynamically when it encounters cases with many citations.

---

## Question 6 — Multi-Agent Research System

The coordinator agent has `AgentDefinitions` configured for all four specialized subagents, each with appropriate descriptions, prompts, and tool restrictions. During testing, you notice the coordinator correctly reasons about when to delegate — it generates messages like "I'll ask the web search agent to find sources on this topic" — but no subagent execution ever occurs. The coordinator then proceeds as if the delegation happened and continues with incomplete information. Logs show no errors.

**What is the most likely cause?**

- **A.** The coordinator's `max_tokens` setting is too low, causing the `Task` tool invocation to be truncated before the subagent type parameter can be specified.
- **B.** The `AgentDefinitions` are configured correctly, but the coordinator's system prompt doesn't explicitly list the available subagent types, preventing the model from knowing they can be invoked.
- **C.** The coordinator's `allowedTools` configuration doesn't include `"Task"`, so while it can reason about delegation, it cannot invoke the tool required to spawn subagents.
- **D.** Subagent context isolation means task descriptions from the coordinator don't automatically reach subagents; you need to configure explicit context forwarding in `ClaudeAgentOptions`.

---

## Question 7 — Multi-Agent Research System

Your multi-agent research pipeline crashed after processing 12 of 28 documents. The web search agent had identified relevant sources, the document analysis agent had partially completed extraction, and the synthesizer had begun pattern identification. You need to resume processing without repeating work or losing fidelity of prior findings.

**What state management approach best balances information fidelity with context efficiency when restoring agent state?**

- **A.** Have each agent maintain its own persistent state file and reload it independently at the start of each session.
- **B.** Persist the coordinator's conversation log containing all task delegations and responses, providing this to agents when resuming.
- **C.** Have each agent persist a structured report to a known location. On resume, the coordinator loads the reports and injects relevant state into agent prompts.
- **D.** Index all agent outputs in a shared vector store. When resuming, each agent queries the store using semantic search to retrieve relevant prior findings.

---

## Question 8 — Multi-Agent Research System

After the web search agent finds 25 sources (120K tokens of raw content), the document analysis agent extracts key insights (15K tokens), and the synthesis agent produces a coherent narrative draft (3K tokens), the coordinator must pass context to the report generation agent for the final output with proper source citations.

**What context-passing strategy provides the best balance of completeness and efficiency?**

- **A.** Pass only the synthesis draft and have a separate post-processing pipeline match claims to sources and insert citations after the report is generated.
- **B.** Pass the synthesis draft along with a structured source index that maps key claims to their source URLs and relevant excerpts.
- **C.** Pass a condensed summary of all prior stages that preserves the main findings and attributes them to sources by name only.
- **D.** Pass the full accumulated context from all prior agents.

---

## Question 9 — Multi-Agent Research System

The document analysis agent has a single `analyze_document` tool that takes a document and a free-text instruction parameter. During evaluation, requests like "extract the key financial metrics" often return narrative summaries, while "summarize the methodology" sometimes returns raw data tables. The synthesis agent reports that 35% of analysis results require re-requests with clarified instructions.

**What's the most effective way to improve reliability?**

- **A.** Split the generic tool into purpose-specific tools — `extract_data_points`, `summarize_content`, `verify_claim_against_source` — each with defined input/output contracts.
- **B.** Keep the single tool but add an `analysis_type` enum parameter requiring explicit selection between extraction, summarization, and verification modes.
- **C.** Have the coordinator pre-classify each analysis request before passing instructions to the document analysis agent.
- **D.** Enhance the tool description with detailed examples showing how different instruction phrasings should map to different output formats.

---

## Question 10 — Multi-Agent Research System

The web search agent has gathered several relevant sources for a research topic. The document analysis agent now needs to examine these sources.

**How does information typically flow between these two specialized subagents?**

- **A.** The agents communicate through an event-driven message queue, with the document analysis agent subscribing to web search completion events.
- **B.** The web search agent directly invokes the document analysis agent, passing the discovered sources as parameters.
- **C.** The coordinator agent receives the web search agent's output and includes relevant findings in the prompt when invoking the document analysis agent.
- **D.** Both agents access a shared memory store where the web search agent writes findings and the document analysis agent reads them.

---

## Question 11 — Multi-Agent Research System

In production, you observe that simple fact-checking queries (e.g., "What year was the Paris Climate Agreement signed?") traverse all four subagents sequentially, consuming 40+ seconds and significant tokens per query. Complex comparative research benefits from the full pipeline. Your query distribution is diverse and evolving as users discover new applications.

**What's the most effective approach to optimize for varying query complexity?**

- **A.** Implement pattern-based routing that categorizes queries by structure (single-fact vs. comparative vs. analytical) and maps each category to a predefined subagent combination.
- **B.** Create a fast-path for factual questions that bypasses subagents entirely, routing all other queries through the complete pipeline to ensure research thoroughness.
- **C.** Have the coordinator analyze each query and dynamically decide which subagents to invoke based on its assessment of query requirements.
- **D.** Train a query complexity classifier on labeled historical data to predict optimal subagent combinations, retraining periodically as query patterns evolve.

---

## Question 12 — Multi-Agent Research System

When researching "renewable energy adoption," the web search agent returns recent statistics (2024: 35% adoption) while the document analysis agent extracts data from internal reports (2022: 18% adoption). The synthesis agent incorrectly flags these as contradictory sources rather than recognizing the data shows growth over time.

**What change would best enable the synthesis agent to correctly interpret such temporal differences?**

- **A.** Require subagents to include publication or data collection dates in their structured outputs.
- **B.** Add a conflict resolution agent that automatically discards older data when newer data exists for the same metric.
- **C.** Configure the web search agent to only return results from the past 6 months.
- **D.** Instruct the synthesis agent to always treat the most recent data as authoritative and place older findings in a separate historical appendix.

---

## Question 13 — Multi-Agent Research System

The synthesis agent receives summarized findings from the web search and document analysis agents, then passes a consolidated summary to the report generator. During testing, you discover the generated reports make factual claims without proper citations — the report generator cannot attribute statements to their original sources because that metadata was lost during the summarization steps.

**What's the most effective approach to ensure proper source attribution in the final reports?**

- **A.** Have each agent output structured data separating content summaries from source metadata (URLs, document names, page numbers).
- **B.** Have the report generator query the web search agent to re-locate sources for claims in the final report.
- **C.** Instruct the synthesis agent to embed source references inline within its summary text using a consistent citation format.
- **D.** Skip summarization and pass full raw outputs from web search and document analysis directly to the report generator.

---

## Question 14 — Multi-Agent Research System

Production reviews reveal inconsistent handling of uncertainty in final reports. Sometimes conflicting subagent findings are synthesized into a single confident statement (losing nuance), while other times reports over-hedge with excessive qualifications (becoming unhelpful). When the web search agent returns "industry analysts estimate $50B market size (methodology varies)" and the document analysis agent returns "peer-reviewed study estimates $35B (±7B, 95% CI)," the coordinator either picks one arbitrarily or produces vague statements like "the market may be $35B−$50B depending on factors."

**What systematic approach best addresses this?**

- **A.** Configure subagents to only report findings meeting a high-confidence threshold, filtering uncertain information before it reaches the coordinator.
- **B.** Implement a confidence calibration layer that normalizes subagent uncertainty expressions to standardized probability scores (0.0-1.0), then weight-average findings by their calibrated confidence.
- **C.** Instruct the synthesis agent to structure reports with explicit sections distinguishing well-established findings from contested ones, preserving original source characterizations and methodological context.
- **D.** Add a verification subagent that cross-references findings across sources, only passing claims to synthesis that are corroborated by at least two independent sources.

---

## Question 15 — Multi-Agent Research System

In production, final reports frequently contain claims without proper source attribution. Investigation shows that while the web search and document analysis agents correctly attach citations to their outputs, the synthesis agent loses track of which sources support which conclusions when combining findings.

**What's the most effective architectural change?**

- **A.** Maintain complete transcripts of all subagent interactions and add a citation-resolution agent to analyze logs and determine attributions before report generation.
- **B.** Require all subagents to output structured claim-source mappings that the synthesis agent must preserve and merge when combining findings from multiple sources.
- **C.** Add a verification step where the report generator uses semantic similarity matching against original sources to reconstruct which claims came from which documents.
- **D.** Have the coordinator inject source identifier prefixes into text before each handoff, then parse these prefixes at report generation to reconstruct citations.

---

## Answer Key

**Q1: B** — Subagents have zero inherited context from the coordinator's conversation history; if the coordinator's `Task` prompt to the synthesis agent didn't explicitly include the prior agents' outputs, the synthesis agent has genuinely nothing to work with. A, C, and D all invent mechanisms (automatic context sharing, shared API connections, cross-agent history access) that don't exist in this architecture.

**Q2: C** — Different subagent output types carry different natural presentation formats (financial comparisons want tables, news wants narrative flow); forcing everything into one format (bullet points, prose-only, or a single JSON shape) discards the structure that made each type legible in the first place. The fix belongs at the synthesis agent's rendering layer — render by content type. A and D both flatten meaningfully different data into a single format upstream, and B adds an unnecessary intermediate conversion layer instead of just rendering appropriately at the point where it matters.

**Q3: D** — All three symptoms (rigid failure on pre-specified searches, poor adaptability on emerging topics, missed tangential sources) point to the same root cause: overly prescriptive procedural instructions remove the subagent's ability to adapt. Specifying goals and quality criteria instead of exact steps lets the subagent determine its own strategy while still being accountable to clear success criteria. A removes too much guidance (no quality bar at all); B and C both add more procedural scaffolding on top of the same fundamentally over-specified approach.

**Q4: B** — The coordinator already has the findings in its own context from orchestrating the research — spawning a whole subagent and re-transmitting 80K+ tokens for a summarization the coordinator could do directly is pure overhead. Reserve subagent spawning for work that actually needs a dedicated agent (complex analysis), not simple restatement of information already in hand. A adds speculative pre-computation for an unpredictable set of queries; C reduces transfer cost but still pays the latency of spawning and round-tripping a subagent unnecessarily; D still spawns the subagent and adds extra request/response cycles.

**Q5: C** — Parallelizing at the coordinator level (spawning multiple document analysis subagents, each with a distinct subset of precedents, then aggregating) preserves the hub-and-spoke visibility the coordinator needs to monitor and debug, while still getting the latency benefit of parallel execution. A (queue + worker pool) and D (subagent spawning its own subagents) both push work outside the coordinator's direct line of sight; B (recursive hierarchy) adds unnecessary structural depth for what's fundamentally a flat, partitionable batch of precedents.

**Q6: C** — Correct reasoning about delegation with zero actual execution and no errors is the signature of a capability gap, not a knowledge gap: the coordinator "wants" to call `Task` but the tool isn't in its `allowedTools`, so the call never happens and the model just proceeds with whatever it already has. A `max_tokens` truncation (A) would typically produce visible errors or malformed calls; B misdiagnoses a tool-access problem as a prompt-content problem when `AgentDefinitions` were already stated to be correctly configured; D describes real context-isolation behavior but misapplies it to the wrong symptom — this isn't about content not reaching a spawned subagent, it's about the subagent never being spawned at all.

**Q7: C** — Structured, per-agent reports written to a known location, loaded and selectively injected by the coordinator on resume, is the state-manifest pattern: it preserves exactly the fidelity needed (each agent's actual completed work) without paying for the full raw history. A breaks centralized coordination by having every agent reload independently with no coordinator-mediated selection of what's relevant. B preserves everything but at the cost of context efficiency — a full delegation-and-response log is exactly the kind of bulk the question asks you to avoid. D introduces retrieval non-determinism and embedding/indexing infrastructure for a problem that a simple structured checkpoint already solves more directly and deterministically.

**Q8: B** — A structured source index mapping claims to URLs and excerpts, passed alongside the synthesis draft, is the compact structured format that preserves exactly what the report generator needs (attribution) without the cost of passing the full 120K+15K raw token trail (D) or a citation scheme that's too thin to actually support proper citations (C, name-only attribution loses the excerpt/URL detail; A defers attribution to a fragile post-hoc matching step disconnected from the actual generation).

**Q9: A** — This is the canonical tool-splitting scenario: a single tool with a free-text instruction parameter is being asked to behave like three different tools (extraction, summarization, verification), and the model can't reliably infer which behavior is wanted from phrasing alone. Splitting into purpose-specific tools with defined contracts removes the ambiguity structurally. B still relies on the model correctly picking an enum value from the same ambiguous request; C and D add compensating layers (pre-classification, better examples) around a tool that's still fundamentally overloaded.

**Q10: C** — All inter-subagent information flow goes through the coordinator — it takes the web search agent's output and explicitly includes the relevant findings in the document analysis agent's `Task` prompt. A, B, and D all describe direct or shared-infrastructure communication paths between subagents that don't exist in the hub-and-spoke architecture.

**Q11: C** — With a diverse and *evolving* query distribution, any solution that depends on a fixed, pre-defined mapping (A) or a trained classifier requiring periodic retraining (D) will lag behind new query patterns as they emerge. Letting the coordinator itself assess each query's requirements and decide dynamically adapts immediately to new patterns without needing a retraining or remapping cycle. B is a reasonable partial step (a fast-path for factual queries) but doesn't generalize to the full range of evolving complexity the way dynamic coordinator assessment does.

**Q12: A** — The web search and document analysis findings aren't actually contradictory — they're the same metric at two different points in time, and the synthesis agent has no way to know that without dates attached to each finding. Requiring `publication_date`/data-collection dates in structured outputs is the direct fix for exactly this "differences look like contradictions" failure mode. B destroys the historical data point entirely; C limits legitimate historical research; D imposes a blanket policy (recency = authority) that won't hold for all metrics or use cases.

**Q13: A** — Structured outputs that separate content from source metadata (URLs, document names, page numbers) preserve exactly the information the report generator needs for attribution, and they preserve it at the source — the point where the metadata actually exists — rather than trying to recover it after it's already been lost in prose. B adds an unnecessary round-trip after the fact; C is a step in the right direction but embeds attribution as fragile inline text formatting rather than structured data the pipeline can reliably parse; D solves the problem by brute force (skip summarization entirely) at the cost of the token-efficiency benefits summarization exists to provide.

**Q14: C** — Preserving each source's original characterization (methodology, confidence interval, "estimates" vs. "peer-reviewed study") and presenting well-established vs. contested findings in distinct sections keeps both the fidelity of the underlying disagreement and the report's usefulness intact. B invents a normalization scheme (weight-averaging calibrated probability scores) that doesn't actually resolve a genuine methodological disagreement, it just produces a new, arguably less meaningful number. A discards potentially valuable but uncertain findings before the coordinator even sees them. D imposes an arbitrary two-source corroboration requirement that may exclude legitimate single-source findings (e.g., an authoritative peer-reviewed study) that don't need duplication to be credible.

**Q15: B** — The web search and document analysis agents already attach citations correctly — the failure is specifically that the synthesis agent doesn't preserve that structure when merging. Requiring subagents to output structured claim-source mappings (which they already do, per the stem) and requiring the synthesis agent to preserve and merge that structure (rather than flattening it into prose) fixes the actual point of failure. A, C, and D all add compensating machinery downstream (log analysis, semantic-similarity reconstruction, prefix-parsing) to recover attribution that should never have been lost in the first place.
