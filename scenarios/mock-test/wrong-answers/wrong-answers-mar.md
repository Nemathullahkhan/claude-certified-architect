# Wrong Answer Tracker — Multi-Agent Research (MAR)

> Log every missed question here with the **why**, not just the correct answer. The goal is to catch *patterns* in your mistakes, not just memorize individual answers.

**Problem-category legend:**

- 🔄 **Preemptive Gatekeeping vs. Local-Recovery-then-Escalate** → trying to prevent every possible failure upfront (validation, filtering, blocklists) instead of letting the subagent attempt recovery locally and escalate only what it genuinely can't resolve, with context
- 🏷️ **Asymmetric Description Fix vs. Eliminating the Overlap** → fixing only one side of a pair of overlapping/confusable tool names or descriptions, leaving the actual source of ambiguity (the other tool) untouched
- 🔧 **Remove-and-Reroute vs. Constrain-at-the-Interface** → responding to tool misuse by stripping the capability entirely and pushing the request through another agent, instead of replacing the overly broad tool with a narrower, validating one that permits the legitimate use case but blocks the illegitimate one
- 📭 **Empty-Result vs. Access-Failure Conflation** → treating a valid "no results found" response as equivalent to (or requiring the same intervention as) an actual access/connectivity failure

---

## Practice Mock (`scenario-3-multi-agent-research.md §7`) — 22/22 (100%)

No misses. Full marks on the first attempt at the scenario's own practice question bank.

---

## Mock Test 1 (`mock-test-mar-1.md`) — 11/15 (73.3%)

### Q3 — Document analysis subagent fails on corrupted/password-protected/hanging PDFs; every exception terminates the subagent and forces the coordinator into routine error triage

- **Your answer:** C — Make the coordinator validate all documents before sending them to the subagent, rejecting documents that might cause failures.
- **Correct answer:** D — Implement local recovery in the subagent for transient failures and escalate to the coordinator only errors it cannot resolve, including attempted steps and partial results.
- **Category:** 🔄 Preemptive Gatekeeping vs. Local-Recovery-then-Escalate
- **Why you missed it:** Upfront validation feels proactive, but it's fundamentally reactive to failure modes you already know about — it can't anticipate a parsing library hanging on a large-but-otherwise-valid file, and password-protection or corruption may not be detectable without actually attempting to open the file (which is itself the operation that's failing). The actual fix is architectural: push transient-failure recovery down to the subagent (where the context and ability to retry/adjust actually live), and reserve coordinator involvement for genuinely unresolvable cases — with enough detail (attempted steps, partial results) for the coordinator to act intelligently. This is the two-tier error-recovery pattern (subagent handles what it can, escalates only what it can't) applied to a new failure domain (PDF parsing) rather than API timeouts.
- **Rule to remember:** When a stem describes "excessive coordinator involvement in routine error handling," the fix is almost never "stop the errors from reaching the subagent in the first place" (that's usually impossible or brittle) — it's "let the subagent handle more of the recovery itself, and escalate only the genuinely unresolvable remainder with full context." Preemptive gatekeeping is a tempting distractor whenever the failure cause sounds like something you could theoretically screen for in advance — but real-world failure modes (hangs, corruption, protection) often can't be detected without attempting the operation.

---

### Q7 — `analyze_content` (web-search agent) and `analyze_document` (document analysis agent) have near-identical descriptions, causing 45% misrouting

- **Your answer:** D — Expand the document analysis tool description with usage examples like "Use for uploaded PDFs, Word docs, and spreadsheets," leaving the web-search tool unchanged.
- **Correct answer:** B — Rename the web-search tool to `extract_web_results` and update its description to "processes and returns information retrieved from web search and URLs."
- **Category:** 🏷️ Asymmetric Description Fix vs. Eliminating the Overlap
- **Why you missed it:** Improving one tool's description is directionally correct, but the actual root cause is that **both** tools have generic, overlapping names/descriptions ("analyzes content"/"analyzes documents," both "extracts key information"). Leaving the web-search tool's confusable name and description untouched means the ambiguity that caused the misrouting in the first place is still there from the model's perspective when it's deciding between the two — you've only made one side of the comparison better, not resolved the actual overlap between them.
- **Rule to remember:** When a misrouting problem is explicitly caused by **two** tools having overlapping/near-identical descriptions, the fix must address **both** sides of the pair — typically by differentiating names and descriptions for each, not just enriching one. An answer that "fixes tool A, leaves tool B exactly as described in the problem statement" is only ever a half-fix when the *problem itself* was framed as a comparison between two tools.

---

### Q10 — Document analysis agent's general-purpose `fetch_url` tool is being misused to perform ad hoc web search via search-engine-results pages

- **Your answer:** B — Remove `fetch_url` from the document analysis agent and route all URL fetching through the coordinator to the web-search agent.
- **Correct answer:** A — Replace `fetch_url` with a `load_document` tool that validates that URLs point to document formats.
- **Category:** 🔧 Remove-and-Reroute vs. Constrain-at-the-Interface
- **Why you missed it:** Removing `fetch_url` entirely also removes the document analysis agent's *legitimate* ability to fetch documents by URL — a real capability it needs — and forces every one of those legitimate cases through an unnecessary coordinator → web-search-agent round trip. The actual problem isn't "this agent has URL-fetching capability at all," it's "this agent's URL-fetching tool is too permissive, letting it fetch search-results pages it should never touch." The precise fix is a **narrower, validating tool** (`load_document`, which checks that a URL actually resolves to a document format) — it keeps the legitimate capability while making the illegitimate use case structurally impossible, rather than removing capability wholesale and adding latency for the common case.
- **Rule to remember:** When a tool is being misused for something *adjacent to* its intended purpose (not something totally unrelated), check whether a **constrained replacement tool that validates the input** solves it before reaching for "remove the tool and route through another agent instead." Removing a tool the agent genuinely needs for its core job — just to prevent misuse at the edges — is usually overcorrection; a validating, narrower interface is the least-privilege fix that preserves the legitimate use case.

---

### Q12 — Web-search subagent gets 15 results (academic), "0 results" (industry reports), and "Connection timeout" (patents) — three different outcomes from one query batch

- **Your answer:** B — Report both "timeout" and "0 results" as failures requiring coordinator intervention.
- **Correct answer:** D — Distinguish access failures (timeout) that require a retry decision from valid empty results ("0 results") that represent successful queries.
- **Category:** 📭 Empty-Result vs. Access-Failure Conflation
- **Why you missed it:** "0 results" from industry reports is a **successful query that legitimately found nothing** — it's informative (there may genuinely be no industry reports on this topic) and requires no coordinator intervention at all. Treating it the same as a connection timeout (a genuine access failure that *does* need a retry decision) collapses two semantically different outcomes into one, which either causes the coordinator to waste effort "fixing" something that isn't broken, or — worse — trains the system to distrust legitimate empty results going forward. This is the same "empty result ≠ error" principle that appears throughout the exam material (e.g., `lookup_order` returning no orders vs. a database timeout) applied to search source outcomes instead of a lookup tool.
- **Rule to remember:** Whenever a stem presents multiple outcomes from parallel operations and one of them is an explicit "0 results" / empty-but-valid response, do not lump it in with genuine failures (timeouts, connection errors, exceptions) just because both "didn't produce data." Ask: did the operation *complete successfully and find nothing*, or did the operation *fail to complete at all*? Only the second category needs a coordinator retry/recovery decision.

---

## Mock Test 2 (`mock-test-mar-2.md`) — 14/15 (93.3%)

### Q7 — Pipeline crashed mid-run (12 of 28 documents processed); need to resume without repeating work or losing fidelity, while staying context-efficient

- **Your answer:** Not captured — the source platform only marked this question as **Incorrect** without recording which option was selected. Logging the question and correct rationale below so the pattern is tracked even without the exact wrong choice.
- **Correct answer:** C — Have each agent persist a structured report to a known location. On resume, the coordinator loads the reports and injects relevant state into agent prompts.
- **Category:** 🧾 State-Manifest Precision vs. Full-Fidelity or Infrastructure Overreach
- **Why this is the correct answer (and what the tempting alternatives get wrong):** This is the crash-recovery "state manifest" pattern — structured, per-agent checkpoints that the coordinator loads and selectively injects on resume. It's the only option that satisfies *both* stated constraints (fidelity **and** context efficiency) at once:
  - **B** (persist the coordinator's full conversation log of all delegations/responses) preserves maximum fidelity but explicitly fails the efficiency constraint — replaying an entire delegation history is exactly the kind of bulk the question is asking you to avoid.
  - **D** (shared vector store + semantic search retrieval on resume) adds indexing/embedding infrastructure and non-deterministic retrieval to a problem that a simple structured checkpoint solves more directly and deterministically — classic overengineering.
  - **A** (each agent maintains and reloads its own state file independently) breaks centralized coordination — it lets agents resume without the coordinator selecting or filtering what's actually relevant, undermining the hub-and-spoke model this whole scenario is built on.
- **Rule to remember:** Crash-recovery questions in a coordinator/subagent architecture almost always resolve to the same pattern: **structured per-agent/per-task state, persisted at checkpoints, loaded and selectively injected by the coordinator on resume** — not a full raw log (fidelity without efficiency), not a shared retrieval store (unnecessary infrastructure), and not independent agent-level reloading (breaks centralized control). When a question stem explicitly names two constraints ("fidelity" *and* "efficiency"), immediately eliminate any option that only satisfies one of them.

---

## Recurring Patterns Across Mistakes

1. **Reaching for prevention/removal at the wrong layer instead of a more targeted, precise fix** (Mock 1: Q3, Q10) — both misses proposed a structurally heavier or more disruptive fix (upfront document validation; full tool removal + reroute) when a more precise, narrower fix existed (local recovery + selective escalation; a validating replacement tool). When a stem describes friction or misuse, check for a fix that surgically addresses the *specific* overreach or failure mode before reaching for a broader gatekeeping/removal solution.
2. **Treating a "fix one side of a two-sided problem" option as sufficient** (Mock 1: Q7) — when two tools are explicitly named as the source of ambiguity, watch for distractors that only improve one of them; the correct answer almost always needs to address the actual overlap between both.
3. **Conflating "no data returned" outcomes that have different underlying causes** (Mock 1: Q12) — a recurring exam theme (also seen in Domain 5's `isError: false` + empty content vs. `isError: true` distinction) now confirmed as a live gap in the multi-agent research context specifically: distinguish valid-empty-result from actual-access-failure before deciding on any recovery/intervention action.
4. **Picking an option that satisfies only one of two explicitly stated constraints** (Mock 2: Q7) — new pattern, watch for recurrence. When a stem states two requirements at once (e.g., "fidelity AND efficiency," "completeness AND speed"), immediately screen out any option that clearly sacrifices one of them, even if it fully satisfies the other.

## Score Trend

| Attempt | Score | Missed |
|---|---|---|
| Practice Mock | 22/22 (100%) | — |
| Mock Test 1 | 11/15 (73.3%) | Q3, Q7, Q10, Q12 |
| Mock Test 2 | 14/15 (93.3%) | Q7 |

**Trend: 100% → 73.3% → 93.3%.** Strong recovery — none of Mock 1's three confirmed patterns (preemptive-fix overreach, asymmetric tool-description fixes, empty-result/failure conflation) recurred in Mock 2, suggesting those gaps are closing. The single Mock 2 miss is a new, distinct pattern (dual-constraint screening in crash-recovery/state-management questions) rather than a repeat of an earlier mistake.

## Action Items Before Next Mock Test

- [ ] Before picking a fix that "prevents the failure from happening at all" (validation, filtering, removal), check whether a narrower fix exists that lets the subagent recover locally or constrains the tool at the interface instead — prevention-at-all-costs is a recurring distractor pattern. *(0 misses in Mock 2 — likely closing.)*
- [ ] When a misrouting problem names two overlapping tools, verify the correct answer addresses both sides of the overlap — don't accept an option that only improves one tool's description. *(0 misses in Mock 2 — likely closing.)*
- [ ] Always separate "the operation completed and found nothing" from "the operation failed to complete" before deciding whether coordinator intervention is warranted. *(0 misses in Mock 2 — likely closing.)*
- [ ] **New:** For state-management/crash-recovery questions, explicitly check every option against *all* stated constraints (e.g., fidelity AND efficiency) before picking — eliminate any option that clearly sacrifices one, even a tempting full-fidelity or infrastructure-heavy one.
- [ ] Take a third mock test or a short targeted drill on dual-constraint screening (crash recovery, context passing, latency-vs-completeness trade-offs) to confirm this new Mock 2 pattern doesn't recur before calling Scenario 3 exam-ready.
