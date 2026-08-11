# Mock Test: Code Generation with Claude Code — Set 1

> Anchored to `scenario-2-code-generation.md`. Covers CLAUDE.md hierarchy and modularization, skills vs. slash commands, `context: fork`, `allowed-tools`, `argument-hint`, path-specific rules, plan mode vs. direct execution, iterative refinement, MCP server scoping, and context management during large-scale refactors.

---

## Question 31 — Code Generation with Claude Code

You asked Claude Code to implement a function that transforms API responses into an internal normalized format. After two iterations, the output structure still doesn't match expectations—some fields are nested differently and timestamps are formatted incorrectly. You described requirements in prose, but Claude interprets them differently each time.

**Which approach is most effective for the next iteration?**

- **A.** Write a JSON schema describing the expected output structure and validate Claude's output against it after each iteration.
- **B.** Provide 2–3 concrete input-output examples showing the expected transformation for representative API responses.
- **C.** Rewrite requirements with more technical precision, specifying exact field mappings, nesting rules, and timestamp format strings.
- **D.** Ask Claude to explain its current understanding of the requirements to identify where interpretations diverge.

---

## Question 32 — Code Generation with Claude Code

You need to add Slack as a new notification channel. The existing codebase has clear, established patterns for email, SMS, and push channels. However, Slack's API offers fundamentally different integration approaches—incoming webhooks (simple, one-way), bot tokens (support delivery confirmation and programmatic control), or Slack Apps (two-way events, requires workspace approval). Your task says "add Slack support" without specifying integration method or requiring advanced features like delivery tracking.

**How should you approach this task?**

- **A.** Start in direct execution mode using incoming webhooks to match the existing one-way notification pattern.
- **B.** Switch to planning mode to explore integration options and architectural implications, then present a recommendation before implementation.
- **C.** Start in direct execution mode by scaffolding a Slack channel class using existing patterns, deferring the integration method decision.
- **D.** Start in direct execution mode using a bot-token approach to ensure delivery confirmation is possible.

---

## Question 33 — Code Generation with Claude Code

Your CLAUDE.md file has grown to 400+ lines containing coding standards, testing conventions, a detailed PR review checklist, deployment instructions, and database migration procedures. You want Claude to always follow coding standards and testing conventions, but apply PR review, deploy, and migration guidance only when doing those tasks.

**Which restructuring approach is most effective?**

- **A.** Move all guidance into separate Skills files organized by workflow type, leaving only a brief project description in CLAUDE.md.
- **B.** Keep everything in CLAUDE.md but use `@import` syntax to organize into separately maintained files by category.
- **C.** Split CLAUDE.md into files under `.claude/rules/` with path-bound glob patterns so each rule loads only for the relevant file types.
- **D.** Keep universal standards in CLAUDE.md and create Skills for workflow-specific guidance (PR review, deploy, migrations) with trigger keywords.

---

## Question 34 — Code Generation with Claude Code

You're tasked with restructuring your team's monolithic application into microservices. This impacts changes across dozens of files and requires decisions about service boundaries and module dependencies.

**Which approach should you choose?**

- **A.** Switch to planning mode to explore the codebase, understand dependencies, and design the implementation approach before making changes.
- **B.** Start in direct execution mode and switch to planning only after encountering unexpected complexity during implementation.
- **C.** Start in direct execution mode and make incremental changes, letting implementation reveal natural service boundaries.
- **D.** Use direct execution with detailed upfront instructions that specify each service structure.

---

## Question 35 — Code Generation with Claude Code

Your team created a `/analyze-codebase` skill that performs deep code analysis—dependency scanning, test coverage counts, and code quality metrics. After running the command, team members report Claude becomes less responsive in the session and loses the context of the original task.

**How do you most effectively fix this while keeping full analysis capabilities?**

- **A.** Add `context: fork` in the skill frontmatter to run the analysis in an isolated subagent context.
- **B.** Add `model: haiku` in frontmatter to use a faster, cheaper model for analysis.
- **C.** Split the skill into three smaller skills, each producing less output.
- **D.** Add instructions to the skill to compress all results into a short summary before displaying them.

---

## Question 36 — Code Generation with Claude Code

Your team uses a `/commit` skill in `.claude/skills/commit/SKILL.md`. A developer wants to customize it for their personal workflow (different commit message format, extra checks) without affecting teammates.

**What do you recommend?**

- **A.** Create a personal version under `~/.claude/skills/` with a different name, e.g., `/my-commit`.
- **B.** Add conditional logic based on username in the project skill frontmatter.
- **C.** Create a personal version at `~/.claude/skills/commit/SKILL.md` with the same name.
- **D.** Set `override: true` in the personal skill frontmatter to prioritize it over the project version.

---

## Question 37 — Code Generation with Claude Code

Your team has used Claude Code for months. Recently, three developers report Claude follows the guidance "always include comprehensive error handling," but a fourth developer who just joined says Claude does not follow it. All four work in the same repo and have up-to-date code.

**What is the most likely cause and fix?**

- **A.** The guidance lives in the original developers' user-level `~/.claude/CLAUDE.md` files, not in the project `.claude/CLAUDE.md`. Move the instruction to the project-level file so all team members receive it.
- **B.** The new developer's `~/.claude/CLAUDE.md` contains conflicting instructions overriding project settings; they should delete the conflicting section.
- **C.** Claude Code learns per-user preferences over time; the new developer must repeat the requirement until Claude "remembers" it.
- **D.** Claude Code caches CLAUDE.md after first read; original developers use cached versions. Everyone should clear the Claude Code cache.

---

## Question 38 — Code Generation with Claude Code

You find that including 2–3 full endpoint implementation examples as context significantly improves consistency when generating new API endpoints. However, this context is useful only when creating new endpoints—not when debugging, reviewing code, or other work in the API directory.

**Which configuration approach is most effective?**

- **A.** Add endpoint examples and pattern documentation to the project CLAUDE.md so they are always available.
- **B.** Manually reference endpoint examples in every generation request by copying code into the prompt.
- **C.** Configure path-specific rules in `.claude/rules/api/` that include endpoint examples and activate when working in the API directory.
- **D.** Create a skill that references the endpoint examples and contains pattern-following instructions, invoked on demand via a slash command.

---

## Question 39 — Code Generation with Claude Code

Your team created a `/migration` skill that generates database migration files. It takes the migration name via `$ARGUMENTS`. In production you observe three issues: (1) developers often run the skill without arguments, causing poorly named files, (2) the skill sometimes uses database schema details from unrelated prior conversations, and (3) a developer accidentally ran destructive test cleanup when the skill had broad tool access.

**Which configuration approach fixes all three problems?**

- **A.** Use positional parameters `$1` and `$2` instead of `$ARGUMENTS` to enforce specific inputs, include explicit schema file references via `@` syntax for context control, and add a frontmatter description warning about destructive operations.
- **B.** Add `argument-hint` in frontmatter to request required parameters, use `context: fork` to isolate execution, and restrict `allowed-tools` to file-write operations.
- **C.** Split into `/migration-create` and `/migration-apply` skills, add validation instructions to request migration name if missing, and use different `allowed-tools` scopes for each.
- **D.** Add validation instructions in the skill SKILL.md to ensure `$ARGUMENTS` is a valid name, add prompts to ignore prior conversation context, and list prohibited operations to avoid.

---

## Question 40 — Code Generation with Claude Code

Your codebase contains areas with different coding conventions: React components use functional style with hooks, API handlers use async/await with specific error handling, and database models follow the repository pattern. Test files are distributed across the codebase next to the code under test (e.g., `Button.test.tsx` next to `Button.tsx`), and you want all tests to follow the same conventions regardless of location.

**What is the most supported way to ensure Claude automatically applies the correct conventions when generating code?**

- **A.** Put all conventions in the root CLAUDE.md under headings for each area and rely on Claude to infer which section applies.
- **B.** Create skills in `.claude/skills/` for each code type, embedding conventions in each SKILL.md.
- **C.** Place a separate CLAUDE.md file in each subdirectory containing conventions for that area.
- **D.** Create rule files under `.claude/rules/` with YAML frontmatter specifying glob patterns to conditionally apply conventions based on file paths.

---

## Question 41 — Code Generation with Claude Code

You want to create a custom slash command `/review` that runs your team's standard code review checklist. It should be available to every developer when they clone or update the repository.

**Where should you create the command file?**

- **A.** In `~/.claude/commands/` in each developer's home directory.
- **B.** In the project repository under `.claude/commands/`.
- **C.** In `.claude/config.json` as an array of commands.
- **D.** In the root project CLAUDE.md.

---

## Question 42 — Code Generation with Claude Code

Your team's CLAUDE.md grew beyond 500 lines mixing TypeScript conventions, testing guidance, API patterns, and deployment procedures. Developers find it hard to locate and update the right sections.

**What approach does Claude Code support to organize project-level instructions into focused topical modules?**

- **A.** Define a `.claude/config.yaml` mapping file patterns to specific sections inside CLAUDE.md.
- **B.** Create separate Markdown files in `.claude/rules/`, each covering one topic (e.g., `testing.md`, `api-conventions.md`).
- **C.** Split instructions into `README.md` files in relevant subdirectories that Claude automatically loads as instructions.
- **D.** Create multiple files named CLAUDE.md at different levels of the directory tree, each overriding parent instructions.

---

## Question 43 — Code Generation with Claude Code

You create a custom skill `/explore-alternatives` that your team uses to brainstorm and evaluate implementation approaches before choosing one. Developers report that after running the skill, subsequent Claude responses are influenced by the alternatives discussion—sometimes referencing rejected approaches or retaining exploration context that interferes with actual implementation.

**How should you most effectively configure this skill?**

- **A.** Use the `!` prefix in the skill to run exploration logic as a bash subprocess.
- **B.** Add `context: fork` in the skill frontmatter.
- **C.** Split into two skills—`/explore-start` and `/explore-end`—to mark boundaries when exploration context should be discarded.
- **D.** Create the skill in `~/.claude/skills/` instead of `.claude/skills/`.

---

## Question 44 — Code Generation with Claude Code

Your team wants to add a GitHub MCP server for searching PRs and checking CI status via Claude Code. Each of six developers has their own personal GitHub access token. You want consistent tooling across the team without committing credentials to version control.

**Which configuration approach is most effective?**

- **A.** Have each developer add the server in user scope via `claude mcp add --scope user`.
- **B.** Create an MCP server wrapper that reads tokens from a `.env` file and proxies GitHub API calls, then add the wrapper to the project `.mcp.json`.
- **C.** Add the server to the project `.mcp.json` using environment variable substitution (`${GITHUB_TOKEN}`) for auth and document the required environment variable in the project README.
- **D.** Configure the server in project scope with a placeholder token, then tell developers to override it in their local config.

---

## Question 45 — Code Generation with Claude Code

You're adding error-handling wrappers around external API calls across a 120-file codebase. The work has three phases: (1) discover all call sites and patterns, (2) collaboratively design the error-handling approach, and (3) implement wrappers consistently. In Phase 1, Claude generates large output listing hundreds of call sites with context, quickly filling the context window before discovery finishes.

**Which approach is most effective to complete the task while maintaining implementation consistency?**

- **A.** Use an Explore subagent for Phase 1 to isolate verbose discovery output and return a summary, then continue Phases 2–3 in the main conversation.
- **B.** Do all phases in the main conversation, periodically using `/compact` to reduce context usage while moving through files.
- **C.** Switch to headless mode with `--continue`, passing explicit context summaries between batch calls to maintain continuity.
- **D.** Define the error-handling pattern in CLAUDE.md, then process files in batches across multiple sessions relying on the shared memory file for consistency.

---

## Answer Key

**Q31: B** — Inconsistent structure and formatting across runs is the classic "prose is ambiguous" signal. Concrete input/output examples pin down field nesting and timestamp formatting simultaneously and more reliably than a schema (A, which validates *after* the fact rather than preventing the ambiguity), more technical prose (C, which is just a more detailed version of the same failed approach), or asking Claude to self-report its understanding (D, which diagnoses but doesn't fix the ambiguity).

**Q32: B** — Multiple integration approaches with real architectural trade-offs (one-way vs. two-way, workspace approval requirements, delivery confirmation) plus an underspecified task description is exactly when plan mode earns its keep — explore the options and get alignment before writing code. A and D silently pick an integration method without surfacing the trade-off to the person who owns the requirement. C defers the decision but still starts building before the decision is made, risking rework.

**Q33: D** — Coding standards and testing conventions are "always active" — they belong in CLAUDE.md. PR review, deploy, and migration guidance are workflow-specific and only relevant when doing those tasks — that's exactly what on-demand Skills are for. A removes universal standards from CLAUDE.md entirely, which is wrong since some guidance genuinely needs to always load. B keeps everything always-loaded regardless of `@import` organization — the token cost problem isn't solved. C (path-bound rules) fits file-type-scoped conventions, not workflow-triggered procedures like "when doing a PR review."

**Q34: A** — Dozens of files, service-boundary decisions, and module-dependency trade-offs are the textbook large-scale-refactor-with-architectural-decisions case for plan mode. B risks costly rework once "unexpected complexity" surfaces after changes are already underway. C lets an unplanned process define architecture reactively — risky for a decision this consequential. D substitutes exhaustive upfront prompting for actual exploration, which doesn't replace understanding the existing dependencies first.

**Q35: A** — `context: fork` runs the analysis in an isolated sub-agent; the main session only receives a summary, preserving both responsiveness and the original task's context — and it keeps the skill's full analysis capability intact. B trades capability/quality for speed, not what was asked. C reduces output per skill but multiplies invocations and still pollutes context. D relies on the skill self-summarizing probabilistically rather than architecturally isolating the verbose work.

**Q36: A** — A personal skill under `~/.claude/skills/` with a different name is additive and isolated — it doesn't touch the shared skill at all, so teammates are unaffected. C overwrites/shadows the same name, which is fragile and could confuse teammates about which version runs. B modifies the shared project file, affecting everyone. D (`override: true`) is not a real Claude Code skill frontmatter mechanism for this purpose.

**Q37: A** — Classic onboarding misconfiguration: instructions living only in individual developers' personal `~/.claude/CLAUDE.md` files never reach new team members because that file isn't version-controlled or shared. Moving the guidance to the project-level CLAUDE.md makes it visible to everyone via git. B, C, and D invent mechanisms (conflicting personal overrides, "learning" preferences, caching) that aren't how CLAUDE.md loading works.

**Q38: C** — The context (endpoint examples) is useful only for a specific task type in a specific directory — exactly the "activates only for matching files" behavior of path-specific rules under `.claude/rules/`. A loads the examples for every file type, including debugging and review work where they're irrelevant. B doesn't scale and relies on the developer remembering every time. D requires manual invocation via a slash command rather than automatically activating when working in that directory.

**Q39: B** — `argument-hint` solves problem (1) by prompting for the missing name; `context: fork` solves problem (2) by isolating the skill's execution from unrelated prior conversation context; `allowed-tools` restricted to file-write operations solves problem (3) by removing the destructive capability entirely. A and D address individual symptoms with instructions/conventions that are still probabilistic (positional params don't stop cross-context bleed; ignoring prior context via prompt is not enforced). C splits the skill but doesn't address the schema-bleed or tool-access problems directly.

**Q40: D** — Conventions that apply by file type/pattern regardless of location (tests scattered next to their source files) are exactly what path-specific rules with glob-pattern frontmatter are designed for. A relies on Claude correctly inferring which section of a monolithic file applies. B requires manual, on-demand invocation rather than automatic application. C would require duplicating test conventions in every subdirectory that has a test file, which doesn't scale across a scattered file layout.

**Q41: B** — Slash commands intended for the whole team must live in the project repository under `.claude/commands/` so they're version-controlled and available to everyone on clone/pull. A is user-level and personal — it won't reach teammates. C and D are not how Claude Code slash commands are defined.

**Q42: B** — `.claude/rules/` supports splitting instructions into focused, topical Markdown files, which is the supported mechanism for modularizing a CLAUDE.md that has grown unwieldy. A and C describe mechanisms Claude Code doesn't use for this purpose. D (multiple files literally named CLAUDE.md scattered by directory level) conflates directory-level CLAUDE.md scoping with topical splitting — it doesn't organize by topic, only by directory.

**Q43: B** — The symptom — rejected alternatives and exploration context leaking into subsequent implementation work — is exactly what `context: fork` prevents by running the skill in an isolated sub-agent whose internal deliberation doesn't persist into the main conversation. A repurposes a bash-execution mechanism for something it isn't designed to solve. C adds workflow complexity (remembering to invoke a second command) without guaranteeing the context is actually discarded. D changes personal vs. team scope, not context isolation.

**Q44: C** — Environment variable substitution in the committed `.mcp.json` (e.g., `${GITHUB_TOKEN}`) keeps the server configuration itself shared and version-controlled while each developer's actual token stays local and out of git — exactly the "consistent tooling, no committed credentials" requirement. A produces six inconsistent, unmanaged personal configurations instead of one team-wide definition. B adds unnecessary custom proxy infrastructure to solve a problem environment variable substitution already solves. D commits a placeholder secret pattern to version control and relies on developers remembering to override it, which is fragile and still touches committed config with credential-shaped content.

**Q45: A** — The Explore subagent isolates Phase 1's verbose discovery output in its own context and returns a summary, protecting the budget needed for Phases 2 (collaborative design) and 3 (consistent implementation) in the main conversation. B tolerates the context-filling problem and just intermittently compacts it away, losing detail. C moves to headless mode, which doesn't address the collaborative design phase's need for back-and-forth discussion. D pushes consistency onto a shared memory file across sessions rather than solving the immediate context-exhaustion problem in Phase 1.
