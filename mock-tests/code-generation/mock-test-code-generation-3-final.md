# Mock Test: Code Generation with Claude Code — Set 3

> Built around the gap patterns identified across your prior Code Generation mocks, plus broader coverage of CLAUDE.md/skills/rules configuration, built-in tool selection, MCP server/tool judgment, and session and context-management discipline during long-running Claude Code work. Distractors are intentionally tempting — read every stem twice before answering.

---

## Question 1 — Code Generation with Claude Code

Your team's CLAUDE.md contains: (1) a mandatory rule that all new API endpoints must validate input with the team's shared schema library, and (2) a 60-line incident-response runbook for what to do when a production deploy fails (rollback steps, who to page, log locations). Both are currently in the root CLAUDE.md. You want the validation rule to remain always-active, but the incident runbook should only load when an actual incident is being handled.

**What is the correct restructuring?**

- **A.** Keep both in CLAUDE.md but wrap the runbook section in HTML comments that Claude is instructed to skip unless "incident" is mentioned.
- **B.** Move the incident runbook into a Skill (e.g., `/incident-response`) invoked on demand; keep the always-applicable validation rule in CLAUDE.md.
- **C.** Use `@import` to pull the runbook into a separate file, keeping it organized but still loaded on every invocation.
- **D.** Move both the validation rule and the runbook into `.claude/rules/` with a broad glob pattern covering all files.

---

## Question 2 — Code Generation with Claude Code

Your team has a shared `.claude/skills/deploy/SKILL.md` that runs the standard deployment checklist. One senior engineer wants to add two extra personal verification steps (checking a dashboard they personally rely on) every time they run `/deploy`, without changing the behavior for the other 11 engineers on the team.

**What is the correct approach?**

- **A.** Edit the shared `.claude/skills/deploy/SKILL.md` directly and add the two steps behind a conditional check for their username.
- **B.** Create `~/.claude/skills/deploy/SKILL.md` (same name, user-level scope) containing the checklist plus their two extra steps — this takes precedence for them only, and the shared team skill remains untouched for everyone else.
- **C.** Ask the rest of the team to also add the two steps to their own workflow manually so everyone stays consistent.
- **D.** Rename their personal version to `/deploy-personal` so there's no ambiguity about which command runs.

---

## Question 3 — Code Generation with Claude Code

Your team wants Claude to reference a specific set of "database migration safety checks" (verify backward compatibility, check for locking operations, confirm a rollback script exists) — but only when a developer is actually authoring a new migration file, not simply when browsing or editing other files in the `migrations/` directory (e.g., fixing a typo in an old migration's comment, or reading migration history for context).

**What is the most appropriate mechanism?**

- **A.** A path-specific rule under `.claude/rules/` with `paths: ["migrations/**/*"]` so it loads automatically whenever any file in that directory is touched.
- **B.** A skill (e.g., `/new-migration`) that bundles the safety checks and is invoked deliberately when a developer is starting a new migration — not tied to merely opening files in the directory.
- **C.** Add the safety checks to the root CLAUDE.md so they're always visible regardless of context.
- **D.** Create a `migrations/CLAUDE.md` directory-level file containing the safety checks.

---

## Question 4 — Code Generation with Claude Code

A developer proposes the following fix for a recurring problem: "We should add a `.claude/priority.json` file that lets us rank which CLAUDE.md sections take precedence when there's a conflict between root and package-level standards." A teammate is skeptical.

**What should you tell them?**

- **A.** This is a reasonable idea and matches how Claude Code resolves conflicting CLAUDE.md instructions today.
- **B.** `.claude/priority.json` is not a real Claude Code mechanism — treat proposals naming unfamiliar, suspiciously specific config files as likely invented unless verified against actual documented configuration surfaces.
- **C.** This is correct, but the file should be named `.claude/config.yaml` instead for consistency with other config files.
- **D.** This is unnecessary because CLAUDE.md files never conflict with each other.

---

## Question 5 — Code Generation with Claude Code

Your team's `.mcp.json` defines a shared internal documentation search server. Each developer has a different internal API key for it, and the team wants to avoid every developer maintaining a separate personal `.mcp.json` fork just to swap in their own key.

**What is the most effective configuration?**

- **A.** Build a small local proxy service that each developer runs, which injects their key before forwarding requests to the internal docs server, then point `.mcp.json` at the proxy.
- **B.** Use `${DOCS_API_KEY}` environment variable substitution in the committed `.mcp.json`, and document in the README that each developer sets `DOCS_API_KEY` in their local environment.
- **C.** Commit one shared API key to `.mcp.json` that all developers use, and rotate it manually if it's ever compromised.
- **D.** Have each developer maintain their own untracked copy of `.mcp.json` locally, ignored via `.gitignore`.

---

## Question 6 — Code Generation with Claude Code

Before deleting a deprecated utility function `formatCurrency()`, you need to find every caller. The function is re-exported from a `utils/index.ts` barrel file, and at least one package re-exports it again under a different name (`money.ts` exports it as `toMoneyString`). You aren't fully certain how many layers of re-export exist across the monorepo.

**Which strategy most reliably finds every caller?**

- **A.** Grep for `formatCurrency` and `toMoneyString` since those are the two names you've already identified.
- **B.** Read the barrel files you know about to enumerate every exported alias, then Grep for each alias name across the codebase.
- **C.** Grep for all files that import from the module defining `formatCurrency` (and its known re-export files), then read each matching file to determine actual usage — this doesn't depend on having already discovered every alias in the chain.
- **D.** Search the codebase's documentation for references to currency formatting utilities.

---

## Question 7 — Code Generation with Claude Code

A developer named a Claude Code session `perf-investigation` last week while profiling a slow API endpoint. They've since started and ended three unrelated sessions on other tasks. They now want to pick the perf investigation back up with all of its accumulated findings.

**What is the correct, most direct command?**

- **A.** `claude --continue`, since it resumes the most recently used session.
- **B.** Search the session transcript log files for the UUID associated with `perf-investigation`, then resume using `--session-id <uuid>`.
- **C.** `claude --resume perf-investigation`, since the session's name is already known and can be passed directly.
- **D.** Start a new session and paste in a manual summary of what was found during the profiling work.

---

## Question 8 — Code Generation with Claude Code

Your CLAUDE.md includes a section titled "Release Checklist" (12 steps covering changelog updates, version bumping, tagging, and announcement posting) that should only matter when someone is actually cutting a release — it's irrelevant noise during day-to-day feature work, and developers say it "gets in the way" of quickly scanning the always-relevant conventions above it.

**What is the correct fix?**

- **A.** Move the Release Checklist into a `/cut-release` skill invoked only when a release is being prepared; leave day-to-day conventions in CLAUDE.md.
- **B.** Use `@import ./release-checklist.md` from CLAUDE.md so it's in its own file, better organized, but still loaded on every invocation.
- **C.** Move the checklist to `.claude/rules/release.md` with `paths: ["CHANGELOG.md"]` so it activates whenever the changelog file is touched.
- **D.** Leave it in CLAUDE.md but add a header saying "SKIP UNLESS CUTTING A RELEASE" so Claude knows to ignore it most of the time.

---

## Question 9 — Code Generation with Claude Code

A new hire copies a teammate's personal `~/.claude/CLAUDE.md` file (which contains that teammate's individual preferences, like preferring tabs over spaces) onto their own machine, assuming it will make their Claude Code match the team's actual conventions. Two weeks later, code reviews reveal the new hire's Claude Code is violating the team's documented spaces-based style guide (which lives in the project's `.claude/CLAUDE.md`).

**What is the most accurate explanation?**

- **A.** The new hire's `~/.claude/CLAUDE.md` (personal, user-level) contains instructions that conflict with the project-level `.claude/CLAUDE.md`, and personal preferences at that narrower, individual scope can influence that individual's sessions alongside — and inconsistently with — the shared project standard.
- **B.** Copying another person's `~/.claude/CLAUDE.md` has no effect since user-level files are ignored when a project-level file exists.
- **C.** The new hire needs to also copy the teammate's `.mcp.json` for the conventions to take effect.
- **D.** CLAUDE.md files are only loaded once per machine at install time, so copying it after Claude Code was already configured had no effect.

---

## Question 10 — Code Generation with Claude Code

Your team wants a specific set of "accessibility review" guidelines (ARIA labeling, color contrast checks, keyboard navigation) applied whenever a developer explicitly asks Claude to review a UI component for accessibility — but NOT automatically every time any `.tsx` file in the `web/` package is opened or edited, since most `.tsx` edits are unrelated to accessibility work.

**What is the correct mechanism?**

- **A.** A `.claude/rules/accessibility.md` file with `paths: ["web/**/*.tsx"]` so it's active for all component work.
- **B.** A `/a11y-review` skill containing the guidelines, invoked deliberately when a developer wants an accessibility review — not tied to merely editing a `.tsx` file.
- **C.** Add the guidelines to `web/CLAUDE.md` so they load for every file in the package.
- **D.** Add the guidelines to the root CLAUDE.md so they're always available regardless of package.

---

## Question 11 — Code Generation with Claude Code

Your team wants every developer's Claude Code session to automatically pick up a personal, experimental MCP server (a half-finished internal tool one developer is prototyping) without that server appearing in anyone else's session or getting accidentally committed to the shared project configuration.

**What is the correct configuration?**

- **A.** Add the experimental server to `.mcp.json` but comment it out with a note asking others not to enable it.
- **B.** Configure the experimental server in that developer's personal `~/.claude.json` (user scope) — it stays local to their machine and never touches the version-controlled project config.
- **C.** Build a feature-flag system inside the MCP server itself so it only activates for a specific username.
- **D.** Add the server to `.mcp.json` in a separate git branch that only that developer checks out.

---

## Question 12 — Code Generation with Claude Code

You need to confirm that a legacy `LegacyPaymentGateway` class has zero remaining callers before deleting it. The class was originally imported directly in several places, but over the past two years, three different wrapper/adapter modules were introduced that import and re-export it under domain-specific names, and you don't have a complete list of those wrappers.

**Which approach is most reliable?**

- **A.** Grep for `LegacyPaymentGateway` directly, and if no results, conclude it's safe to delete.
- **B.** Grep for all files that import from the module where `LegacyPaymentGateway` is defined (including transitively, by checking what imports each file that itself imports it), then inspect actual usage in each — this surfaces indirect/aliased usage without requiring you to already know every wrapper name.
- **C.** Ask each team lead whether their team still uses `LegacyPaymentGateway`, and rely on their answers.
- **D.** Search recent git commit messages for mentions of `LegacyPaymentGateway` to see if it was recently touched.

---

## Question 13 — Code Generation with Claude Code

You're 90 minutes into a Claude Code session refactoring the `auth` module. The context usage indicator shows the window is at roughly 78% full, and you still have two more files to modify plus a final cross-file consistency check to run.

**What is the correct action?**

- **A.** Keep working until the context window is completely full, then decide what to do.
- **B.** Use `/compact` now to summarize completed work into a dense state block while preserving key findings, then continue with the remaining steps.
- **C.** Start an entirely new session and re-explore the auth module from scratch to get a clean context.
- **D.** Ask Claude to be more concise in its responses for the remainder of the session.

---

## Question 14 — Code Generation with Claude Code

You need to find every file in the monorepo that contains a TODO comment referencing "auth migration" (e.g., `// TODO: remove after auth migration`), regardless of file extension or directory.

**Which tool is correct?**

- **A.** Glob with pattern `**/*auth-migration`* to find files with that phrase in the filename.
- **B.** Grep for the string `auth migration` across all files, since you're searching file contents, not filenames.
- **C.** Bash `find . -name "*TODO*"` to locate files with TODO in the name.
- **D.** Read every file in the repository and manually scan for the phrase.

---

## Question 15 — Code Generation with Claude Code

Your team added an MCP server tool called `search_symbols` that performs indexed, cross-repository symbol search (much faster than text search for finding class/function definitions across 50+ repos). Despite this, Claude Code continues to default to Grep for these lookups. The `search_symbols` tool's description simply says: "Searches for symbols."

**What is the most effective fix?**

- **A.** Remove Grep from the available toolset so Claude has no choice but to use `search_symbols`.
- **B.** Rewrite the `search_symbols` description to explain what it returns, when to prefer it over Grep (e.g., "10x faster for cross-repo symbol lookups; use instead of Grep when searching for a definition across multiple repositories"), and its input/output format.
- **C.** Add a system-prompt instruction telling Claude to always prefer `search_symbols` over Grep.
- **D.** Accept the behavior since Grep is a reliable, well-known tool and works fine for this use case.

---

## Question 16 — Code Generation with Claude Code

Two engineers want to independently prototype two different approaches to fixing a flaky integration test, starting from the same shared root-cause analysis session (which already traced the flakiness to a race condition in a shared test fixture).

**What is the correct session management approach?**

- **A.** Both engineers work in the same session sequentially, one after the other, to avoid losing the shared analysis.
- **B.** Use `fork_session` to create two independent branches from the shared analysis session — each engineer explores their approach independently without redoing the root-cause tracing.
- **C.** Each engineer starts a completely new session and re-runs the root-cause investigation independently to avoid interference.
- **D.** One engineer resumes the session and works on approach A; once done, `--resume`s again for approach B in the same thread.

---

## Question 17 — Code Generation with Claude Code

Your `/generate-migration` skill needs to: (1) not clutter the main conversation with the verbose schema-diffing output it produces internally, and (2) only be allowed to create new files, never modify or delete existing ones, and (3) prompt the developer if they forget to specify a migration name.

**Which frontmatter combination is correct?**

- **A.** `context: fork`, `allowed-tools: [Write]`, `argument-hint: "Provide the migration name"`
- **B.** `context: fork`, `allowed-tools: [Read, Write, Edit, Bash]`, `argument-hint: "Provide the migration name"`
- **C.** `allowed-tools: [Write]`, `argument-hint: "Provide the migration name"` (omit `context: fork` since file generation doesn't produce verbose output worth isolating)
- **D.** `context: fork`, `allowed-tools: [Write]` (omit `argument-hint` since the skill can just ask conversationally if the name is missing)

---

## Question 18 — Code Generation with Claude Code

A team's `.mcp.json` currently hardcodes a shared internal Slack bot token as a plain string value so the `/notify-team` MCP tool works out of the box for everyone who clones the repo.

**What is wrong with this, and what should change?**

- **A.** Nothing is wrong — since `.mcp.json` is meant to be shared, hardcoding the working token is the simplest way to guarantee it works for every developer.
- **B.** The token should never be a literal value in a version-controlled file; use `${SLACK_BOT_TOKEN}` environment variable substitution in `.mcp.json` and document that each developer (or the CI environment) must set that variable.
- **C.** Move the entire MCP server configuration to each developer's personal `~/.claude.json` so the token isn't shared via git.
- **D.** Store the token in the root CLAUDE.md instead, since CLAUDE.md supports secrets natively.

---

## Question 19 — Code Generation with Claude Code

You're planning a refactor that requires understanding how a `NotificationService` is used across `api`, `web`, and `mobile` — this requires reading roughly 30 files and will produce substantial intermediate output before you even start designing the refactor.

**What is the most effective approach to preserve context for the actual refactor work?**

- **A.** Read all 30 files directly in the main session so you have full context before starting the plan.
- **B.** Use an Explore subagent to perform the 30-file investigation and return a structured summary to the main session, preserving the main session's context budget for the design and implementation phases.
- **C.** Skip exploration and rely on the model's general knowledge of typical notification service patterns.
- **D.** Read the files in the main session, then run `/compact` immediately afterward to reclaim the space.

---

## Question 20 — Code Generation with Claude Code

A task requires: (1) adding a single new required field to an existing internal API response (clear, single-file change, no ambiguity) and (2) as a related but separable follow-up, deciding whether to version the API or make a backward-compatible additive change — a decision affecting client contracts across 3 downstream teams.

**What is the most effective approach?**

- **A.** Use plan mode for the entire task, since API changes should always be planned regardless of scope.
- **B.** Handle the field addition with direct execution once the versioning decision is made; use plan mode specifically for the versioning decision, since it involves a real architectural trade-off with multi-team impact — don't conflate the two.
- **C.** Use direct execution for both, since the field addition is simple and the versioning question can be resolved informally in conversation.
- **D.** Use plan mode only for the field addition, since that's the part that actually touches code.

---

## Question 21 — Code Generation with Claude Code

Your team's custom MCP tool `run_lint_check` occasionally fails because the linter binary isn't installed in the current environment. It currently returns `{"error": "lint failed"}` for this case, and Claude Code repeatedly retries the same failing call.

**What is the correct fix?**

- **A.** Have the tool return a structured error with `errorCategory: "permission"` and `isRetryable: false`, since environment/installation issues won't resolve on their own — the agent should stop retrying and report the missing dependency instead.
- **B.** Have the tool return `errorCategory: "transient"` and `isRetryable: true` so Claude retries with backoff until the linter becomes available.
- **C.** Leave the error message as-is; the agent should eventually stop retrying on its own after enough attempts.
- **D.** Remove the `run_lint_check` tool entirely since it's unreliable.

---

## Question 22 — Code Generation with Claude Code

A developer is implementing a rate-limiting strategy for an internal API but has never designed one before and isn't sure what dimensions matter (per-user vs. per-IP, sliding window vs. token bucket, what happens on burst traffic, whether to rate-limit by endpoint or globally).

**What is the most effective technique before Claude starts implementing?**

- **A.** Provide a very detailed, prescriptive prompt specifying exact algorithm and thresholds so there's no ambiguity for Claude to resolve.
- **B.** Use the interview pattern — have Claude ask clarifying questions to surface the missing design dimensions (traffic patterns, burst tolerance, scope of limiting) before any code is written.
- **C.** Ask Claude to implement "a standard, sensible rate limiter" and iterate from there once something exists to react to.
- **D.** Provide 2-3 examples of rate limiter implementations from other codebases as few-shot examples.

---

## Question 23 — Code Generation with Claude Code

Your monorepo has: (1) a testing convention that must apply to every `*.test.ts` file regardless of package, (2) an `/analyze-deps` skill that produces 300+ lines of dependency-graph output, and (3) team members who've noticed a new hire's Claude Code isn't applying an "always write descriptive commit messages" rule that "works for everyone else." Investigation shows the rule lives in a senior engineer's `~/.claude/CLAUDE.md`.

**Which combination of fixes correctly addresses all three issues?**

- **A.** Move the commit-message rule to project-level CLAUDE.md; create `.claude/rules/testing.md` with `paths: ["**/*.test.ts"]`; add `context: fork` to `/analyze-deps`.
- **B.** Move the commit-message rule to project-level CLAUDE.md; create a `web/CLAUDE.md`, `api/CLAUDE.md`, etc. for testing conventions; add `context: fork` to `/analyze-deps`.
- **C.** Tell the new hire to manually copy the senior engineer's `~/.claude/CLAUDE.md`; create `.claude/rules/testing.md`; split `/analyze-deps` into three smaller skills.
- **D.** Move the commit-message rule to project-level CLAUDE.md; create `.claude/rules/testing.md` with `paths: ["**/*.test.ts"]`; instruct `/analyze-deps` to "summarize your findings briefly."

---

## Question 24 — Code Generation with Claude Code

A developer spent yesterday afternoon (session named `cache-redesign`) mapping how the current caching layer works across 12 files, producing detailed findings about cache key structure and invalidation triggers. Overnight, a teammate merged a refactor that renamed the main `CacheManager` class to `DistributedCacheManager` and changed its constructor signature — 2 of the 12 previously-read files were affected. The developer wants to continue today, now comparing two invalidation strategies (event-driven vs. TTL-based) independently before choosing one.

**What is the most effective sequence of actions?**

- **A.** Start two fresh sessions from scratch, one per strategy, and re-map the caching layer in each.
- **B.** Resume `cache-redesign` by name, inform the agent specifically about the renamed class and changed constructor in the 2 affected files, then use `fork_session` to branch into two independent explorations — one per invalidation strategy.
- **C.** Resume `cache-redesign` by name without mentioning the overnight changes, then explore both strategies sequentially in the same thread.
- **D.** Look up the UUID for `cache-redesign` from the transcript logs, resume by UUID, and explore both strategies together in one message to save time.

---

## Question 25 — Code Generation with Claude Code

Three hours into a large legacy-module exploration (topic: order-processing pipeline), you notice: the agent has started saying things like "typically, order pipelines use a saga pattern" instead of naming the specific `OrderStateMachine` and `PaymentReconciler` classes it found an hour ago. You also still have 15 more files to review, and the context usage indicator shows 82%.

**What is the most effective combined response?**

- **A.** Clear context entirely and start over, since the agent's degraded answers mean everything it has said so far is unreliable.
- **B.** Compact now (summarizing completed findings into a dense state block, preserving the specific class names as structured facts rather than prose) and/or delegate the remaining 15-file review to a subagent that returns a condensed summary — either way, preserve the already-discovered specific findings rather than letting them dissolve into generic prose.
- **C.** Keep going in the same context without any changes, since 82% still leaves some room and the agent will likely self-correct once it re-reads its own earlier messages.
- **D.** Increase `max_tokens` so the agent has more room to include the specific class names in its next response.

---

## Answer Key

**Q1: B** — The validation rule is universal and belongs in CLAUDE.md; the incident runbook is workflow-specific and should only load when actually handling an incident, which is what an on-demand skill is for. Wrapping content in HTML comments (A) and `@import` (C) both still load the runbook on every invocation regardless of task; a broad rules glob covering everything (D) doesn't distinguish the two rule types at all.

**Q2: B** — A personal `~/.claude/skills/deploy/SKILL.md` with the same name takes precedence for that engineer only, leaving the shared team skill untouched for everyone else — same name at a narrower scope is the standard override pattern, not a conflict. Editing the shared file (A) affects all 11 other engineers; asking everyone to adopt the habit manually (C) doesn't scale or persist; renaming (D) works but isn't the mechanism actually being tested.

**Q3: B** — The trigger is "starting a new migration" (a task), not "any file under `migrations/`" (a path) — reading old migrations or fixing a typo elsewhere in the directory shouldn't pull in the safety checklist. Task-type scoping calls for on-demand invocation via a skill, not automatic path-based activation, even though both happen in the same directory.

**Q4: B** — `.claude/priority.json` is not a real, documented Claude Code mechanism. Cross-check any suspiciously specific, unfamiliar config file name against the actual documented configuration surfaces (`CLAUDE.md`, `@import`, `.claude/rules/`, `.claude/skills/`, `.claude/commands/`, `.mcp.json`) before accepting it as legitimate.

**Q5: B** — `${DOCS_API_KEY}` environment variable substitution in the committed `.mcp.json` is the native, built-in mechanism for a shared config with a per-developer secret. A custom proxy (A) reinvents infrastructure Claude Code already provides; a shared hardcoded key (C) is a security anti-pattern; untracked personal forks (D) create configuration drift across the team.

**Q6: C** — Tracing all files that import from the defining module (and its known re-export points) and checking actual usage is exhaustive by construction — it doesn't depend on already knowing every alias. Name-based Grep for only the aliases you've already found (A, B) silently misses deeper, undiscovered rename chains; documentation (D) may be stale or incomplete.

**Q7: C** — The developer already has the session's name (`perf-investigation`); `--resume perf-investigation` uses that directly. An indirect UUID lookup (B) is an unnecessary extra step when the label already works; `--continue` (A) resumes the wrong (most recent) session since three unrelated sessions happened since; manually summarizing (D) discards the actual accumulated context.

**Q8: A** — The Release Checklist is workflow-specific — only relevant when cutting a release — exactly what an on-demand skill is for. `@import` (B) still loads it on every invocation regardless of organization. Binding it to `CHANGELOG.md` (C) is a plausible-sounding but mismatched trigger, since release prep involves far more than touching the changelog, and the changelog might be touched for unrelated reasons too. A "skip unless" header (D) is a probabilistic instruction, not a structural fix.

**Q9: A** — Personal `~/.claude/CLAUDE.md` files apply to that individual's own sessions and can directly conflict with or diverge from project-level standards — copying someone else's personal preferences doesn't give you the team's actual conventions, which live in the project-level file. B, C, and D all invent behavior that isn't how CLAUDE.md loading actually works.

**Q10: B** — "When a developer explicitly asks for an accessibility review" is a task trigger, not a file-path trigger — most `.tsx` edits are unrelated to accessibility, so automatic path-based activation (A, C, D) would surface irrelevant guidance on nearly every component edit.

**Q11: B** — Personal, experimental, single-developer tools belong in `~/.claude.json` (user scope) — never in the shared, version-controlled `.mcp.json`, even commented out (A) or hidden behind a branch (D), both of which risk surfacing for the team. A username-based feature flag inside the server itself (C) is unnecessary complexity for a problem the existing scoping mechanism already solves.

**Q12: B** — Tracing imports transitively (including what imports the wrapper modules, not just the original module) surfaces indirect/aliased usage without requiring a complete upfront list of every wrapper name. Direct-name Grep only (A) misses all aliased callers; asking team leads (C) or checking commit messages (D) are unreliable, non-exhaustive proxies for actual usage.

**Q13: B** — 70–85% is the zone to compact: summarize completed work into a dense state block, preserve key findings, and continue. Waiting until the window is completely full (A) risks truncation from the worst position; starting an entirely new session (C) discards valid accumulated work; asking for more concise responses (D) is a linguistic ask that doesn't restructure the already-accumulated context.

**Q14: B** — You're searching file contents for a phrase that could appear in any file regardless of name — Grep is correct. Glob (A) and a name-based `find` (C) both search filenames, not contents; reading every file (D) is the anti-pattern built-in search tools exist to avoid.

**Q15: B** — Assuming the server is healthy and connected, a minimal description ("Searches for symbols") gives Claude no reason to prefer it over the familiar built-in Grep — this is a description problem, not a connectivity one. Removing Grep (A) is disruptive; system-prompt routing instructions (C) patch the symptom rather than the root cause; accepting the status quo (D) ignores a fixable pattern — enhancing the MCP tool's description to explain its advantage over the built-in.

**Q16: B** — Two engineers need independent, in-depth exploration of two different approaches from the same shared analytical starting point — exactly what `fork_session` is for. Sequential same-session work (A, D) risks cross-contamination between the two approaches; fully independent fresh sessions (C) needlessly duplicates the root-cause tracing work.

**Q17: A** — All three needs map directly to three distinct frontmatter fields: `context: fork` isolates the verbose schema-diffing output, `allowed-tools: [Write]` permits only file creation (not modification or deletion), and `argument-hint` prompts for the missing migration name. B grants unnecessarily broad tool access that undermines the "never modify/delete" requirement; C drops the context isolation needed for verbose output; D drops the deterministic argument-prompting mechanism in favor of unreliable conversational inference.

**Q18: B** — Never commit literal secret values to a version-controlled file; `${SLACK_BOT_TOKEN}` substitution is the built-in mechanism that keeps `.mcp.json` shared and safe simultaneously. Moving the whole server to personal config (C) breaks the "consistent tooling for everyone" goal; CLAUDE.md (D) has no special secret-handling mechanism and is the wrong file type entirely.

**Q19: B** — A 30-file investigation producing substantial intermediate output is exactly what the Explore subagent isolates — it returns a structured summary, preserving the main session's context for the actual refactor design and implementation. Reading everything directly (A) consumes the budget needed later; skipping exploration (C) risks an ungrounded plan; reading then compacting (D) still burns context before compaction happens.

**Q20: B** — These are two genuinely different-scope problems bundled into one task description: the field addition is single-file and unambiguous (direct execution), while the versioning decision has real architectural trade-offs affecting three downstream teams (plan mode). Treating the whole task uniformly in either direction (A, C, D) fails to recognize that scope varies within a single ticket.

**Q21: A** — A missing linter binary is an environment/setup issue that won't resolve itself on retry — `isRetryable: false` is correct, and the agent should stop retrying and surface the actual problem instead of looping. Treating it as transient (B) causes exactly the repeated-retry behavior described in the stem; doing nothing (C) leaves the vague error in place; removing the tool (D) is disproportionate to a fixable categorization problem.

**Q22: B** — Rate limiting is an unfamiliar-domain, unknown-design-dimensions problem — the interview pattern surfaces missing considerations (burst tolerance, scope of limiting, traffic patterns) before any code commits to an approach. Prescriptive upfront detail (A) assumes the developer already knows what they don't know; "standard implementation" (C) may not fit actual constraints; examples (D) help with format consistency, not with eliciting unknown requirements.

**Q23: A** — Each of the three problems maps to a distinct, correct mechanism: project-level CLAUDE.md for the universal commit-message rule (not personal `~/.claude/CLAUDE.md`, and not asking the new hire to manually copy files); `.claude/rules/testing.md` with a glob pattern for a convention spanning file types across packages (not per-package CLAUDE.md duplication); `context: fork` for the verbose skill (not just asking it to "be brief," which is probabilistic).

**Q24: B** — Resume by name (the label is already known, no UUID lookup needed) and explicitly inform the agent of the two specifically-known changes (renamed class, changed constructor) for targeted re-analysis — don't discard the 12-file analysis, but don't silently proceed on stale assumptions either. Then `fork_session` lets the two invalidation strategies be explored independently from that corrected baseline, since sequential exploration in one thread (C) or two fresh full re-investigations (A) both fail either the "don't lose valid work" or "don't contaminate independent approaches" requirement.

**Q25: B** — This combines two moves: compacting to preserve the specific findings (`OrderStateMachine`, `PaymentReconciler`) as structured facts rather than letting them fade into generic prose, and/or delegating the remaining 15 files to a subagent so the main session's context isn't further strained. Clearing everything (A) discards genuinely valid findings; continuing unchanged (C) ignores both the degradation symptom and the 82% usage; `max_tokens` (D) controls output length, not the input context problem causing the degradation.