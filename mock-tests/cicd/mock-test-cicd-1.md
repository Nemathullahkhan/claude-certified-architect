# Mock Test: Claude Code for CI/CD — Set 1

> First adversarial mock for Scenario 5. Covers CI headless invocation, structured CLI output, Batch vs Synchronous API selection (including multi-turn tool-calling limits), explicit review criteria, few-shot actionable feedback, self-review bias, multi-pass review, prior-findings / existing-test context injection, and trust restoration via category disable. Distractors are intentionally tempting — read every stem twice before answering.
>
> Source numbering (Q16–Q30) matches the original practice-exam question IDs — use them as-is when logging misses in your own wrong-answer tracker (see `mock-tests/wrong-answers/TEMPLATE-wrong-answer-tracker.md`).

---

## Question 16 — Claude Code for Continuous Integration

Your CI pipeline runs the Claude Code CLI (in `--print` mode) using `CLAUDE.md` to provide project context for code review, and developers generally find the reviews substantive. However, they report that integrating findings into the workflow is difficult—Claude outputs narrative paragraphs that must be manually copied into PR comments. The team wants to automatically post each finding as a separate inline PR comment at the relevant place in code, which requires structured data with file path, line number, severity level, and suggested fix.

**Which approach is most effective?**

- **A.** Add an "Output Format for Review" section to `CLAUDE.md` with examples of structured findings so Claude learns the expected format from project context.
- **B.** Use the CLI flags `--output-format json` and `--json-schema` to enforce structured findings, then parse the output to post inline comments via the GitHub API.
- **C.** Include explicit formatting instructions in the review prompt requiring each finding to follow a parseable template like `[FILE:path] [LINE:n] [SEVERITY:level] ....`
- **D.** Keep narrative review format but add a summarization step that uses Claude to generate a structured JSON summary of findings.

---

## Question 17 — Claude Code for Continuous Integration

Your team uses Claude Code for generating code suggestions, but you notice a pattern: non-obvious issues—performance optimizations that break edge cases, cleanups that unexpectedly change behavior—are only caught when another team member reviews the PR. Claude's reasoning during generation shows it considered these cases but concluded its approach was correct.

**Which approach directly addresses the root cause of this self-check limitation?**

- **A.** Run a second independent instance of Claude Code to review the changes without access to the generator's reasoning.
- **B.** Enable extended thinking mode for the generation stage to allow more thorough deliberation before producing suggestions.
- **C.** Add explicit self-review instructions to the generation prompt asking Claude to critique its own suggestions before finalizing output.
- **D.** Include full test files and documentation in prompt context so Claude better understands expected behavior during generation.

---

## Question 18 — Claude Code for Continuous Integration

Your code review component is iterative: Claude analyzes the changed file, then may request related files (imports, base classes, tests) via tool calls to understand context before providing final feedback. Your application defines a tool that lets Claude request file contents; Claude calls the tool, gets results, and continues analysis. You're evaluating batch processing to reduce API cost.

**What is the primary technical limitation when considering batch processing for this workflow?**

- **A.** Batch processing does not include correlation IDs to map outputs back to input requests.
- **B.** The asynchronous model cannot execute tools mid-request and return results for Claude to continue analysis.
- **C.** The Batch API does not support tool definitions in request parameters.
- **D.** The batch processing latency of up to 24 hours is too slow for pull request feedback, although the workflow would otherwise function.

---

## Question 19 — Claude Code for Continuous Integration

Your CI/CD system runs three Claude-based analyses: (1) fast style checks on every PR that block merging until completion, (2) comprehensive weekly security audits of the entire codebase, and (3) nightly test-case generation for recently changed modules. The Message Batches API offers 50% savings but processing can take up to 24 hours. You want to optimize API cost while maintaining an acceptable developer experience.

**Which combination correctly matches each task to an API approach?**

- **A.** Use the Message Batches API for all three tasks to maximize 50% savings, configuring the pipeline to poll for batch completion.
- **B.** Use synchronous calls for PR style checks; use the Message Batches API for weekly security audits and nightly test generation.
- **C.** Use synchronous calls for all three tasks for consistent response times, relying on prompt caching to reduce costs across workloads.
- **D.** Use synchronous calls for PR style checks and nightly test generation; use the Message Batches API only for weekly security audits.

---

## Question 20 — Claude Code for Continuous Integration

Your automated reviews find real issues, but developers report the feedback is not actionable. Findings include phrases like "complex ticket routing logic" or "potential null pointer" without specifying what exactly to change. When you add detailed instructions like "always include concrete fix suggestions," the model still produces inconsistent output—sometimes detailed, sometimes vague.

**Which prompting technique most reliably produces consistently actionable feedback?**

- **A.** Further refine instructions with more explicit requirements for each part of the feedback format (location, issue, severity, proposed fix).
- **B.** Expand the context window to include more surrounding codebase so the model has enough information to propose concrete fixes.
- **C.** Implement a two-pass approach where one prompt identifies issues and a second generates fixes, allowing specialization.
- **D.** Add 3–4 few-shot examples showing the exact required format: identified issue, location in code, concrete fix suggestion.

---

## Question 21 — Claude Code for Continuous Integration

Your CI pipeline includes two Claude-based code review modes: a pre-merge-commit hook that blocks PR merge until completion, and a "deep analysis" that runs overnight, polls for batch completion, and posts detailed suggestions to the PR. You want to reduce API cost using the Message Batches API, which offers 50% savings but requires polling and can take up to 24 hours.

**Which mode should use batch processing?**

- **A.** Only the pre-merge-commit hook.
- **B.** Only the deep analysis.
- **C.** Both modes.
- **D.** Neither mode.

---

## Question 22 — Claude Code for Continuous Integration

Your automated review analyzes comments and docstrings. The current prompt instructs Claude to "check that comments are accurate and up to date." Findings often flag acceptable patterns (TODO markers, simple descriptions) while missing comments describing behavior the code no longer implements.

**What change addresses the root cause of this inconsistent analysis?**

- **A.** Include git blame data so Claude can identify comments that predate recent code changes.
- **B.** Add few-shot examples of misleading comments to help the model recognize similar patterns in the codebase.
- **C.** Filter TODO, FIXME, and descriptive comment patterns before analysis to reduce noise.
- **D.** Specify explicit criteria: flag comments only when the behavior they claim contradicts the code's actual behavior.

---

## Question 23 — Claude Code for Continuous Integration

Your automated code review system shows inconsistent severity ratings—similar issues like null pointer risks are rated "critical" in some PRs but only "medium" in others. Developer surveys show growing distrust—many start dismissing findings without reading because "half are wrong." High-false-positive categories erode trust in accurate categories.

**Which approach best restores developer trust while improving the system?**

- **A.** Temporarily disable high-false-positive categories (style, naming, documentation) and keep only high-precision categories while improving prompts.
- **B.** Keep all categories enabled but display confidence scores with each finding so developers can decide what to investigate.
- **C.** Keep all categories enabled and add few-shot examples to improve accuracy for each category over the next few weeks.
- **D.** Apply a uniform strictness reduction across all categories to bring the overall false-positive rate down.

---

## Question 24 — Claude Code for Continuous Integration

Your automated review generates test-case suggestions for each PR. Reviewing a PR that adds course completion tracking, Claude suggests 10 test cases, but developer feedback shows that 6 duplicate scenarios already covered by the existing test suite.

**What change most effectively reduces duplicate suggestions?**

- **A.** Include the existing test file in context so Claude can determine what scenarios are already covered.
- **B.** Reduce the requested number of suggestions from 10 to 5, assuming Claude prioritizes the most valuable cases first.
- **C.** Add instructions directing Claude to focus exclusively on edge cases and error conditions rather than success paths.
- **D.** Implement post-processing that filters suggestions whose descriptions match existing test names via keyword overlap.

---

## Question 25 — Claude Code for Continuous Integration

After an initial automated review identifies 12 findings, a developer pushes new commits to address issues. Re-running review produces 8 findings, but developers report that 5 duplicate previous comments on code that was already fixed in the new commits.

**What is the most effective way to eliminate this redundant feedback while maintaining thoroughness?**

- **A.** Run review only when the PR is created and in the final pre-merge state, skipping intermediate commits.
- **B.** Add a post-processing filter that removes findings that match previous ones by file paths and issue descriptions before posting comments.
- **C.** Restrict review scope to files changed in the most recent push, excluding files from earlier commits.
- **D.** Include previous review findings in context and instruct Claude to report only new or still-unresolved issues.

---

## Question 26 — Claude Code for Continuous Integration

Your pipeline script runs `claude "Analyze this pull request for security issues"`, but the job hangs indefinitely. Logs show Claude Code is waiting for interactive input.

**What is the correct approach to run Claude Code in an automated pipeline?**

- **A.** Add a `--batch` flag: `claude --batch "Analyze this pull request for security issues"`.
- **B.** Add the `-p` flag: `claude -p "Analyze this pull request for security issues"`.
- **C.** Redirect stdin from `/dev/null`: `claude "Analyze this pull request for security issues" < /dev/null`.
- **D.** Set the environment variable `CLAUDE_HEADLESS=true` before running the command.

---

## Question 27 — Claude Code for Continuous Integration

A pull request changes 14 files in an inventory tracking module. A single-pass review that analyzes all files together produces inconsistent results: detailed feedback on some files but shallow comments on others, missed obvious bugs, and contradictory feedback (a pattern is flagged in one file but identical code is approved in another file in the same PR).

**How should you restructure the review?**

- **A.** Run three independent full-PR review passes and flag only issues that appear in at least two of the three runs.
- **B.** Split into focused passes: review each file individually for local issues, then run a separate integration-oriented pass to examine cross-file data flows.
- **C.** Require developers to split large PRs into smaller submissions of 3–4 files before running automated review.
- **D.** Switch to a larger model with a bigger context window so it can pay sufficient attention to all 14 files in one pass.

---

## Question 28 — Claude Code for Continuous Integration

Your automated code review averages 15 findings per pull request, and developers report a 40% false-positive rate. The bottleneck is investigation time: developers must click into each finding to read Claude's rationale before deciding whether to fix or dismiss it. Your `CLAUDE.md` already contains comprehensive rules for acceptable patterns, and stakeholders rejected any approach that filters findings before developers see them.

**What change best addresses investigation time?**

- **A.** Require Claude to include its rationale and confidence estimate directly in each finding.
- **B.** Add a post-processor that analyzes finding patterns and automatically suppresses those that match historical false-positive signatures.
- **C.** Categorize findings as "blocking issues" vs "suggestions," with different review requirements by level.
- **D.** Configure Claude to show only high-confidence findings, filtering uncertain flags before developers see them.

---

## Question 29 — Claude Code for Continuous Integration

Analysis of your automated code review shows large differences in false-positive rates by finding category: security/correctness findings have 8% false positives, performance findings 18%, style/naming findings 52%, and documentation findings 48%. Developer surveys show growing distrust—many start dismissing findings without reading because "half are wrong." High-false-positive categories erode trust in accurate categories.

**Which approach best restores developer trust while improving the system?**

- **A.** Temporarily disable high-false-positive categories (style, naming, documentation) and keep only high-precision categories while improving prompts.
- **B.** Keep all categories enabled but display confidence scores with each finding so developers can decide what to investigate.
- **C.** Keep all categories enabled and add few-shot examples to improve accuracy for each category over the next few weeks.
- **D.** Apply a uniform strictness reduction across all categories to bring the overall false-positive rate down.

---

## Question 30 — Claude Code for Continuous Integration

Your team wants to reduce API costs for automated analysis. Currently, synchronous Claude calls support two workflows: (1) a blocking pre-merge check that must complete before developers can merge, and (2) a technical debt report generated overnight for review the next morning. Your manager proposes moving both to the Message Batches API to save 50%.

**How should you evaluate this proposal?**

- **A.** Move both to batch processing with fallback to synchronous calls if batches take too long.
- **B.** Move both workflows to batch processing with status polling to verify completion.
- **C.** Use batch processing only for technical debt reports; keep synchronous calls for pre-merge checks.
- **D.** Keep synchronous calls for both workflows to avoid issues with batch result ordering.

---

## Answer Key

**Q16: B** — `--output-format json` with `--json-schema` enforces structured output at the CLI level, guaranteeing well-formed JSON with the required fields (file path, line number, severity, suggested fix) that can be reliably parsed and posted as inline PR comments via the GitHub API. `CLAUDE.md` examples (A) and prompt templates (C) are probabilistic; a second summarization pass (D) adds cost and another failure point without guaranteeing schema compliance.

**Q17: A** — A second independent Claude Code instance without access to the generator's reasoning directly addresses the root cause by avoiding confirmation bias. Extended thinking (B) and self-review instructions (C) keep the same reasoning context that already rationalized the approach; more docs in context (D) improves generation quality but does not fix self-check bias.

**Q18: B** — The Batch API is fire-and-forget: it has no mechanism to intercept a tool call mid-request, execute the tool, and return results for Claude to continue analysis. That is fundamentally incompatible with iterative tool-calling workflows. Latency (D) is a real constraint for PR feedback but is not the *primary technical* limitation named by the stem — the workflow would not "otherwise function." Correlation IDs (A) do exist via `custom_id`; tool definitions (C) are not the blocker.

**Q19: B** — PR style checks block developers and require synchronous calls. Weekly security audits and nightly test generation are scheduled, latency-tolerant jobs that can use the Message Batches API for 50% savings. Batching everything (A) breaks blocking UX; sync for nightly work (D) leaves savings on the table.

**Q20: D** — Few-shot examples are the most effective technique when detailed instructions alone still produce inconsistent format. Further refining prose instructions (A) is what already failed; more context (B) and two-pass specialization (C) do not address format consistency the way concrete examples do.

**Q21: B** — Deep analysis already runs overnight with polling — it matches the Batch API's asynchronous model and captures 50% savings. The pre-merge hook is blocking and cannot tolerate up to 24 hours with no latency SLA.

**Q22: D** — Explicit criteria ("flag only when claimed behavior contradicts actual code behavior") replaces a vague instruction with a precise definition of the problem, reducing both false positives on acceptable patterns and misses of truly misleading comments. Few-shot examples of misleading comments (B) help recognition but do not define *what counts* as a problem the way explicit categorical criteria do — and the stem's root cause is the vague criterion itself.

**Q23: A** — Temporarily disabling high-false-positive categories immediately stops trust erosion while preserving value from high-precision categories, and creates space to improve prompts before re-enabling. Confidence scores (B) and few-shot (C) leave the noise live while trust continues to erode; uniform strictness reduction (D) harms accurate categories.

**Q24: A** — Including the existing test file fixes the root cause: Claude can only avoid already-covered scenarios if it knows what tests exist. Reducing count (B), restricting to edge cases (C), and keyword post-filters (D) are proxies that do not give Claude the missing information.

**Q25: D** — Including prior findings and instructing Claude to report only new or still-unresolved issues uses Claude's reasoning to avoid redundant feedback while preserving thoroughness. Skipping intermediate reviews (A) and narrowing to latest-push files (C) lose coverage; post-processing filters (B) are brittle string matching.

**Q26: B** — `-p` / `--print` is the documented non-interactive flag. `--batch` (A) and `CLAUDE_HEADLESS` (D) are not real Claude Code mechanisms; stdin redirect (C) is a Unix workaround that does not properly address Claude Code's command syntax.

**Q27: B** — Per-file local passes + a separate cross-file integration pass directly addresses attention dilution. Consensus across three full-PR runs (A) can suppress real intermittent catches; forcing smaller PRs (C) shifts burden without fixing the system; a larger context window (D) does not solve attention quality.

**Q28: A** — Including rationale and confidence *in each finding* reduces investigation time without filtering anything out — satisfying the stakeholder constraint that all findings remain visible. Post-processor suppression (B) and high-confidence-only filtering (D) both filter findings before developers see them (explicitly rejected). Blocking-vs-suggestion categorization (C) changes review *process* but does not put the triage information where developers need it (inside the finding itself).

**Q29: A** — Same trust-restoration pattern as Q23 with explicit FP-rate evidence by category: temporarily disable the noisy categories (style/naming/documentation at ~50% FP) while keeping high-precision ones and improving prompts. Leaving noise live (B, C) continues trust erosion; uniform strictness cuts (D) degrade accurate categories.

**Q30: C** — Match API to latency requirements: Batch for overnight technical debt reports; Synchronous for blocking pre-merge checks. Moving both to batch (A, B) breaks developer merge UX; avoiding batch entirely over ordering concerns (D) is a misconception — `custom_id` correlates results.
