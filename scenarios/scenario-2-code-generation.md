# Scenario 2: Code Generation with Claude Code

> **Primary domains:** 3 (Claude Code Configuration & Workflows), 5 (Context Management & Reliability)
> **Task statements in play:** 3.1, 3.2, 3.3, 3.4, 3.5, 2.5, 5.4, 1.7
> **Exam weight:** This is the only scenario centered entirely on the Claude Code product. Every question here is about how a development team configures, customizes, and operates Claude Code — not about building agents with the SDK.

---

## Table of Contents
1. [The Scenario](#1-the-scenario)
2. [System Architecture](#2-system-architecture)
3. [Role of Each Domain in This Scenario](#3-role-of-each-domain-in-this-scenario)
4. [What This Scenario Tests From You](#4-what-this-scenario-tests-from-you)
5. [Domain Task-Statement Walkthrough](#5-domain-task-statement-walkthrough)
6. [Scenario-Specific Traps](#6-scenario-specific-traps)
7. [Practice Question Bank](#7-practice-question-bank)
8. [Answer Key](#8-answer-key)
9. [Quick-Recall Cheat Sheet](#9-quick-recall-cheat-sheet)

---

## 1. The Scenario

You are a senior engineer on a team of 12 developers using **Claude Code** to accelerate software development across a TypeScript monorepo with five packages: `api`, `web`, `mobile`, `shared`, and `infra`. Your team uses Claude Code for:

- **Code generation** — writing new features from specifications
- **Refactoring** — modernizing a legacy `auth` module (8,000 lines, no tests)
- **Debugging** — diagnosing issues from stack traces and test failures
- **Documentation** — generating inline comments and README files

**The team has built custom tooling:**
- A `/review` slash command that runs code review against the team's standards
- A `/test-gen` skill that generates test cases for a given file
- A `/analyze-codebase` skill that maps module dependencies

**Current pain points being addressed:**
- New hires don't receive team conventions when they start — their Claude Code behaves differently
- The `/analyze-codebase` skill floods the main conversation with verbose output
- Test conventions need to apply to all `*.test.tsx` files regardless of which directory they're in
- The `auth` module refactor is a 45-file change — risky to execute without a plan
- A single developer has a personal experimental MCP server that keeps showing up for teammates

**The real design challenge:** Claude Code is highly configurable — CLAUDE.md files, skills, rules, plan mode, sessions — but each configuration mechanism has a specific scope and purpose. The exam tests whether you know which tool to use for which problem, and specifically the hierarchy/scoping rules that determine who sees what.

---

## 2. System Architecture

```mermaid
flowchart TD
    subgraph user_level ["User Level (~/.claude/)"]
        UserMD["~/.claude/CLAUDE.md\n(personal only — not shared)"]
        UserCmds["~/.claude/commands/\n(personal slash commands)"]
        UserSkills["~/.claude/skills/\n(personal skills)"]
        UserJson["~/.claude.json\n(personal MCP servers)"]
    end

    subgraph project_level ["Project Level (.claude/)"]
        ProjMD["CLAUDE.md (root)\n(team-wide — version controlled)"]
        ProjCmds[".claude/commands/\n(team slash commands)"]
        ProjSkills[".claude/skills/\n(team skills)"]
        ProjRules[".claude/rules/\n(path-scoped rules)"]
        ProjMcp[".mcp.json\n(team MCP servers)"]
    end

    subgraph pkg_level ["Package Level (per package)"]
        PkgMD["api/CLAUDE.md\nweb/CLAUDE.md\nmobile/CLAUDE.md\n(package-specific standards)"]
    end

    ProjMD -->|"@import"| PkgMD
    ProjRules -->|"paths: [\"**/*.test.tsx\"]"| TestRule["testing.md\n(applies to all test files)"]
    ProjRules -->|"paths: [\"terraform/**/*\"]"| InfraRule["infra.md\n(applies to terraform files)"]

    UserMD -.->|"personal only\nteam doesn't see this"| PersonalDev["Individual Developer"]
    ProjMD --> TeamDev["All Developers\n(via git)"]
```

**Key architectural facts to memorize:**
- `~/.claude/CLAUDE.md` = user-level = not committed to git = teammates never see it
- `.claude/CLAUDE.md` or root `CLAUDE.md` = project-level = committed = team-wide
- `.mcp.json` = project-scoped MCP (team); `~/.claude.json` = user-scoped MCP (personal)
- Skills with `context: fork` run in an isolated sub-agent — output does not pollute main conversation

---

## 3. Role of Each Domain in This Scenario

| Domain | Role | Tested? |
|---|---|---|
| **Domain 1 — Agentic Architecture** | **Minor.** Session management (`--resume`, `fork_session`) crosses over, but no multi-agent orchestration | Yes — 1.7 only |
| **Domain 2 — Tool Design & MCP** | **Minor crossover.** Built-in tool selection (Grep vs. Glob vs. Read/Write/Edit) is a 2.5 task statement tested in this context | Yes — 2.5 only |
| **Domain 3 — Claude Code Config** | **Primary.** Owns everything: CLAUDE.md hierarchy, skills, slash commands, path-specific rules, plan mode vs. direct execution, iterative refinement | Yes — 3.1, 3.2, 3.3, 3.4, 3.5 |
| **Domain 4 — Prompt Engineering** | **Not tested.** No JSON schema / structured output / few-shot narrative here | No |
| **Domain 5 — Context & Reliability** | **Secondary.** Owns long-session context degradation during the auth module refactor, and session continuity across work sessions | Yes — 5.4, 1.7 |

**The short version:** Domain 3 is the primary lens. Nearly every question will be about how Claude Code is configured. Domain 5 comes in for the "how does context hold up during a long, multi-session refactor" questions. Domain 2 (2.5) appears for built-in tool selection questions that arise naturally when using Claude Code.

---

## 4. What This Scenario Tests From You

This scenario tests your **mastery of Claude Code's configuration hierarchy** and your ability to match the right configuration mechanism to the right problem. The core judgment required: given a symptom ("new hires don't get conventions", "skill floods the conversation", "test rules apply everywhere"), identify which specific Claude Code feature fixes it — and why all the similar-looking alternatives are wrong.

### Knowledge you must have cold

| Must know | Detail |
|---|---|
| CLAUDE.md scoping | `~/.claude/CLAUDE.md` = personal, not version-controlled; project-level = in git, team-wide |
| `@import` syntax | Pulls in external files to keep CLAUDE.md modular; used to load package-specific standards |
| Skill frontmatter | `context: fork` = isolated sub-agent; `allowed-tools` = restrict tools; `argument-hint` = prompt for missing args |
| Path-specific rules | `.claude/rules/<name>.md` with `paths: ["glob/**"]` YAML frontmatter — loads only for matching files |
| Plan mode triggers | 45+ files, architecture decisions, multiple approaches = plan mode; single-file clear fix = direct execution |
| Explore subagent purpose | Isolates verbose discovery output from main session context |
| `--resume` vs `fork_session` | `--resume` = continue same session; `fork_session` = independent branches from shared baseline |
| Grep vs Glob | Grep = searches file *content*; Glob = finds files by *name pattern* |
| Edit fallback | Edit fails on non-unique match → Read + Write |
| `/memory` command | Diagnoses which CLAUDE.md files are currently loaded |

### Judgment calls the exam will ask you to make

| Exam question type | The judgment you must apply |
|---|---|
| "New hire's Claude Code doesn't follow team standards — fix it" | Standards are in `~/.claude/CLAUDE.md` (user-level, not shared) → move to project-level |
| "Skill floods main conversation with 400 lines of output — fix it" | Add `context: fork` to skill frontmatter |
| "Test conventions must apply to all `*.test.tsx` files regardless of directory — fix it" | `.claude/rules/testing.md` with `paths: ["**/*.test.tsx"]`, not per-directory CLAUDE.md |
| "45-file auth refactor vs. single-function null pointer fix — which mode?" | Plan mode for the refactor; direct execution for the bug fix |
| "Find all callers of a function — which tool?" | Grep (searches content), not Glob (searches file names) |
| "Developer resumes session; files changed since — what to do?" | Explicitly tell the agent which files changed; don't silently resume with stale results |

### Wrong-answer patterns to immediately recognize and reject

- Any answer that puts **team conventions in `~/.claude/CLAUDE.md`** — that file is personal
- Any answer that creates **5 separate directory-level CLAUDE.md files** for a convention that applies by file type across directories — use path-specific rules instead
- Any answer that uses **plan mode for a single-file, clear-scope change** — unnecessary overhead
- Any answer that uses **Glob to find function callers** — Glob cannot search file content
- Any answer that **resumes a stale session without noting file changes** — the model will use outdated tool results

---

## 5. Domain Task-Statement Walkthrough

### 3.1 — CLAUDE.md Configuration Hierarchy

**How it shows up here:**
New hires join the team and their Claude Code doesn't follow team standards. Investigation reveals the standards are in one developer's `~/.claude/CLAUDE.md` — a user-level file that only applies to them, is never committed to git, and is invisible to every other developer.

**The CLAUDE.md hierarchy:**

```
~/.claude/CLAUDE.md          ← user-level: personal only, not version controlled
  ↕ does NOT cascade to others

{project}/CLAUDE.md          ← project-level: all team members (via git)
{project}/.claude/CLAUDE.md  ← also project-level

{project}/api/CLAUDE.md      ← directory-level: applies when editing api/ files
{project}/web/CLAUDE.md      ← directory-level: applies when editing web/ files
```

**Right vs. wrong:**

| ✅ Correct | ❌ Wrong |
|---|---|
| Move team conventions to project-level `CLAUDE.md` committed to git | Keep team conventions in `~/.claude/CLAUDE.md` and tell new hires to copy it manually |
| Use `@import ./api/standards.md` in the root `CLAUDE.md` to pull in package-specific standards | Put all package standards in a single monolithic root `CLAUDE.md` that grows unbounded |
| Use `/memory` command to debug which CLAUDE.md files are loaded and in what order | Guess why a developer's Claude Code is behaving differently |
| Split a large CLAUDE.md into `.claude/rules/` topic files (e.g., `testing.md`, `api-conventions.md`) | Maintain one 5,000-line CLAUDE.md that loads entirely on every invocation |

---

### 3.2 — Custom Slash Commands and Skills

**How it shows up here:**
Three skill design problems arise on this team:
1. `/analyze-codebase` floods the main conversation with hundreds of lines of output
2. `/test-gen` should not be allowed to delete files — it should only write
3. New developers invoke `/review` without knowing which arguments to provide

**Skills vs. commands — what each is for:**

| Feature | Location | Purpose | When to choose it |
|---|---|---|---|
| CLAUDE.md | Project root | Always-loaded universal standards | Team-wide rules active in every session |
| Slash command | `.claude/commands/` | On-demand team procedures | Repeatable multi-step workflows |
| Skill | `.claude/skills/` | On-demand with advanced configuration | Need `context: fork`, `allowed-tools`, `argument-hint` |

**Skill frontmatter options:**

```yaml
---
context: fork       # run in isolated sub-agent — output doesn't enter main conversation
allowed-tools:      # restrict which tools the skill can use
  - Write
argument-hint: "Provide the file path to generate tests for"
---
```

**Right vs. wrong:**

| ✅ Correct | ❌ Wrong |
|---|---|
| Add `context: fork` to `/analyze-codebase` — it runs isolated, returns a summary, doesn't pollute the main session | Leave `/analyze-codebase` without fork — every analysis floods the active context |
| Add `allowed-tools: [Write]` to `/test-gen` — it can write test files but cannot delete or run arbitrary bash | Leave all tools available to `/test-gen` — it could accidentally overwrite source files |
| Add `argument-hint: "Provide the file or module to review"` to `/review` — CLI prompts developer when invoked without args | Leave `/review` with no argument-hint — developers get cryptic errors or blank output |
| Create personal variant skills in `~/.claude/skills/` with different names to avoid affecting teammates | Modify the shared `.claude/skills/` skill directly for personal preference |

---

### 3.3 — Path-Specific Rules

**How it shows up here:**
The team's test conventions (naming patterns, fixture usage, assertion styles) need to apply to all `*.test.tsx` files across every package — `api/src/auth.test.tsx`, `web/src/__tests__/login.test.tsx`, `mobile/tests/auth.test.tsx`. These files are spread across 5 packages. A per-directory CLAUDE.md would require 5 separate copies of the same rules.

**How path-specific rules work:**

```
.claude/rules/testing.md    ← rule file with YAML frontmatter
```

```yaml
---
paths:
  - "**/*.test.tsx"
  - "**/*.spec.ts"
---
# Testing conventions
- Use describe/it blocks
- Mock external dependencies with jest.fn()
- Use fixtures from tests/fixtures/ — do not hardcode test data
```

This file is loaded only when editing a file that matches the glob pattern — it's inactive when editing production code.

**Right vs. wrong:**

| ✅ Correct | ❌ Wrong |
|---|---|
| `.claude/rules/testing.md` with `paths: ["**/*.test.tsx"]` — one rule file applies to all test files regardless of directory | Create separate `api/CLAUDE.md`, `web/CLAUDE.md`, `mobile/CLAUDE.md` each containing the same testing rules |
| Use glob patterns that target files by type across directories | Use directory-level CLAUDE.md files when the convention applies to a file type scattered across many directories |
| Path-specific rules for infrastructure files: `paths: ["terraform/**/*"]` — loads only when editing Terraform | Put Terraform conventions in root CLAUDE.md — loads for every file even when writing TypeScript |

---

### 3.4 — Plan Mode vs. Direct Execution

**How it shows up here:**
Two contrasting tasks on this team:
- **Task A:** Refactor the `auth` module — 45 files, potential approach choices, architectural implications
- **Task B:** Fix a null pointer bug with a clear stack trace pointing to line 47 of `auth/session.ts`

**Decision framework:**

| Task characteristic | Correct mode |
|---|---|
| Large-scale changes affecting many files | Plan mode |
| Multiple valid approaches with meaningful trade-offs | Plan mode |
| Architectural decisions (e.g., choosing between two library integrations) | Plan mode |
| Verbose discovery phase that would exhaust the context window | Plan mode + Explore subagent |
| Single-file, well-understood change with clear scope | Direct execution |
| Bug fix with a clear stack trace pointing to a specific line | Direct execution |
| Adding one validation check to one function | Direct execution |

**Right vs. wrong:**

| ✅ Correct | ❌ Wrong |
|---|---|
| Use plan mode for the 45-file auth refactor — explore the codebase, design the approach, review the plan, then execute | Jump directly into executing the auth refactor without a plan — risk of costly rework across 45 files |
| Use the Explore subagent to isolate verbose discovery output (mapping all auth module dependencies) from the main session | Run the full dependency discovery in the main session — fills context window with exploration output before implementation even starts |
| Use direct execution for the line-47 null pointer bug — scope is clear, no architecture decisions required | Use plan mode for a single-function bug fix — unnecessary overhead |

---

### 3.5 — Iterative Refinement Techniques

**How it shows up here:**
Three iterative refinement problems arise on this team:
1. A prose spec for a data transformation produces inconsistent output across runs
2. Designing a cache invalidation strategy in an unfamiliar domain
3. A set of bugs some of which interact with each other

**The techniques:**

| Problem | Correct technique | Why |
|---|---|---|
| Output format is inconsistent across runs | 2-3 concrete input/output examples | Examples communicate the exact transformation more precisely than prose |
| Unfamiliar domain (cache invalidation) — may miss important considerations | Interview pattern — Claude asks clarifying questions first | Surfaces cache invalidation strategies, failure modes, TTL requirements before implementing |
| Several bugs that interact | Single detailed message addressing all of them together | Fixing bug A may affect bug B — Claude needs to see both to avoid reintroducing one while fixing the other |
| Several independent bugs | Sequential iteration, one at a time | Addressing independent bugs together can conflate issues |
| Implementing a feature from test failures | Write tests first; share failures with Claude iteratively | Test failures are precise feedback — more reliable than prose descriptions |

---

### 2.5 — Built-in Tool Selection

**How it shows up here:**
Claude Code's built-in tools (Read, Write, Edit, Bash, Grep, Glob) each have a specific purpose. Misselecting tools during codebase exploration is a common failure.

**Tool selection decision table:**

| Task | Correct tool | Wrong tool (and why) |
|---|---|---|
| Find all callers of `authenticate()` across the codebase | **Grep** — searches file content for a pattern | Glob — finds files by name pattern, not by content |
| Find all test files (`*.test.tsx`) | **Glob** — finds files matching a name pattern | Grep — would need to read every file first |
| Modify a specific block of code with unique surrounding context | **Edit** — targeted modification using unique anchor text | Write — rewrites the entire file |
| Modify code where the target text is repeated in multiple places | **Read + Write** — read full file, make changes, write it back | Edit — fails when it can't find a unique match |
| Read a configuration file before making decisions | **Read** — loads full file contents | Bash (cat) — equivalent but not the idiomatic tool |

**Right vs. wrong:**

| ✅ Correct | ❌ Wrong |
|---|---|
| Use Grep to search for function names, error messages, or import patterns across the codebase | Use Glob to search for content — Glob only matches file paths, not file contents |
| When Edit fails because the target text appears multiple times in the file, fall back to Read + Write | Repeatedly retry Edit with the same anchor text when it fails due to non-uniqueness |
| Build codebase understanding incrementally: Grep to find entry points → Read to follow imports | Read every file in the repository upfront before starting any task |

---

### 5.4 — Context Management in Large Codebase Exploration

**How it shows up here:**
The auth module refactor involves exploring 8,000 lines of legacy code across multiple sessions. Over time, the model's answers start referencing "typical patterns" instead of the specific classes and functions it actually found in the codebase.

**Right vs. wrong:**

| ✅ Correct | ❌ Wrong |
|---|---|
| Spawn subagents to investigate specific sub-questions ("find all test files", "trace auth token dependencies") while the main agent preserves high-level coordination | Run the entire exploration in the main session — verbose discovery output fills the context window and degrades answer quality |
| Maintain a scratchpad file (`auth-findings.md`) that records key findings (module boundaries, critical dependencies, untested paths) — reference it for subsequent questions | Rely on the model's in-context memory across a long session — context degrades over time |
| Use `/compact` to reduce context usage when the window fills with verbose exploration output | Continue the session until the context window is exhausted |
| Summarize key findings from one exploration phase before spawning subagents for the next phase | Spawn subagents for the next phase without summarizing what was learned — subagents start without context |

---

### 1.7 — Session Management

**How it shows up here:**
The auth module refactor happens across multiple work sessions over several days. Mid-refactor, two different architectural approaches emerge that need to be compared.

**Session management options:**

| Situation | Correct approach |
|---|---|
| Continuing an investigation from the previous work session | `--resume <session-name>` — continues the named session with existing context |
| Comparing two refactoring approaches from the same analysis baseline | `fork_session` — creates two independent branches from the same starting point |
| Prior session has stale tool results (files have changed since then) | Start a new session; inject a structured summary of what was learned; tell the agent which files changed |
| Resuming after file changes | Inform the agent of the specific changed files for targeted re-analysis, not a full re-exploration |

**Right vs. wrong:**

| ✅ Correct | ❌ Wrong |
|---|---|
| `--resume auth-refactor-session` to continue the named investigation next day | Start a new session every morning and re-explore from scratch |
| `fork_session` from the shared analysis baseline to compare "extract auth to microservice" vs. "in-place modernization" | Start two separate fresh sessions that each redo all the exploration work |
| After files change: "Since our last session, these 3 files have been updated: [list]. Please re-analyze only those." | Resume session and hope the model's cached tool results are still accurate |

---

## 6. Scenario-Specific Traps

| Trap | Why it's wrong | Correct approach |
|---|---|---|
| Putting team conventions in `~/.claude/CLAUDE.md` | User-level config is personal — not committed to git, invisible to teammates | Move to project-level `CLAUDE.md` or `.claude/CLAUDE.md` committed to the repo |
| Using a directory-level CLAUDE.md for test conventions that span all packages | You'd need to duplicate the rules in every package directory | Use `.claude/rules/testing.md` with `paths: ["**/*.test.tsx"]` — one file, covers all |
| Running `/analyze-codebase` without `context: fork` | Hundreds of lines of analysis output flood the main conversation, consuming context for subsequent work | Add `context: fork` to the skill frontmatter — output stays isolated |
| Resuming a stale session without noting which files changed | The model may use outdated tool results for files that have been modified since the session | Either start fresh with a summary, or explicitly tell the agent which files changed |
| Using plan mode for a single-function bug fix with a clear stack trace | Plan mode adds overhead and exploration for a task with a clear, well-scoped solution | Direct execution — scope is defined, no architectural decisions needed |
| Using Glob to find all callers of a function | Glob matches file paths, not file content — it cannot find function call sites | Grep — searches file content for patterns like function names |
| Giving a skill access to all tools "for flexibility" | The skill may run destructive operations (delete files, run arbitrary bash) | Use `allowed-tools` in skill frontmatter to restrict to what the skill actually needs |

---

## 7. Practice Question Bank

> **Instructions:** All questions are anchored to Scenario 2. Read each in the context of the TypeScript monorepo and the team using Claude Code described above.

---

### 3.1 — CLAUDE.md Hierarchy (3 questions)

**Q1.** A new engineer joins the team and reports that Claude Code isn't following the team's coding standards — it's not using the team's naming conventions or error-handling patterns. Investigation reveals the standards are documented in the tech lead's `~/.claude/CLAUDE.md`. What is the root cause?

- A) The new engineer's Claude Code version is outdated and needs to be updated
- B) The standards are in a user-level CLAUDE.md file that is personal to the tech lead and never shared via version control
- C) New engineers need to run a setup command to load team CLAUDE.md files
- D) The standards file uses incorrect syntax that Claude Code doesn't recognize on new machines

---

**Q2.** The team's monorepo has five packages with different technology stacks. You want each package to load only its relevant standards (e.g., the `web` package loads React conventions; the `api` package loads REST API conventions) without duplicating content across files. The correct approach is:

- A) Create a single large root CLAUDE.md that includes all standards for all packages
- B) Create separate CLAUDE.md files in each package directory without any linking
- C) Use `@import` directives in each package's CLAUDE.md to pull in only the standards relevant to that package's domain
- D) Create separate git branches for each package with different CLAUDE.md files

---

**Q3.** A developer reports that Claude Code is behaving inconsistently — sometimes following team standards, sometimes not. You suspect a CLAUDE.md loading issue. What is the most direct way to diagnose which memory files are currently loaded?

- A) Check the git history of the CLAUDE.md files to see if any were recently modified
- B) Run the `/memory` command to see which CLAUDE.md files are loaded and in what order
- C) Add a print statement to the CLAUDE.md file that outputs its own content
- D) Compare the developer's behavior logs against the team-level standards

---

### 3.2 — Slash Commands and Skills (3 questions)

**Q4.** The `/analyze-codebase` skill maps module dependencies across the entire monorepo and produces 400+ lines of analysis output. After running it, developers find the main conversation is filled with this output and subsequent responses are slower and less focused. What is the correct fix?

- A) Remove the `/analyze-codebase` skill and replace it with a Bash script
- B) Add `context: fork` to the skill's frontmatter so it runs in an isolated sub-agent context and returns only a summary to the main session
- C) Reduce the skill's output by instructing it to "be brief"
- D) Split the skill into three smaller skills that analyze fewer modules each

---

**Q5.** The `/test-gen` skill generates test cases for a given source file. During a review, you discover that when it encounters an existing test file, it sometimes deletes it and rewrites it from scratch, accidentally removing tests that were manually written. You need to prevent destructive file operations. The correct fix is:

- A) Add a warning to the skill's description: "Do not delete existing test files"
- B) Change the skill to ask the user for confirmation before writing any file
- C) Add `allowed-tools: [Write]` to the skill's frontmatter so it can write files but cannot delete them or run arbitrary bash
- D) Run the skill in a temporary git branch and review the diff before merging

---

**Q6.** A junior developer invokes `/review` without any arguments and gets a blank output. The skill expects a file path or module name. How do you fix this?

- A) Add a validation check in the skill's body that exits with an error message if no argument is provided
- B) Add `argument-hint: "Provide the file or module path to review"` to the skill's frontmatter — Claude Code prompts the developer when invoked without arguments
- C) Change the skill to always review the currently open file if no argument is given
- D) Document the required argument format in the team wiki

---

### 3.3 — Path-Specific Rules (2 questions)

**Q7.** Test conventions need to apply to all `*.test.tsx` files across all 5 packages in the monorepo. Which approach correctly implements this?

- A) Add testing conventions to the root `CLAUDE.md` — they'll apply everywhere
- B) Create `api/CLAUDE.md`, `web/CLAUDE.md`, `mobile/CLAUDE.md`, `shared/CLAUDE.md`, and `infra/CLAUDE.md`, each containing the same testing rules
- C) Create `.claude/rules/testing.md` with YAML frontmatter `paths: ["**/*.test.tsx"]` — rules activate only when editing matching test files
- D) Add testing conventions to each package's `CLAUDE.md` using `@import` directives

---

**Q8.** Infrastructure-as-code conventions (Terraform naming, resource organization) should only be active when editing files in the `terraform/` directory. Which configuration correctly implements this while minimizing irrelevant context during TypeScript development?

- A) Add Terraform conventions to the root `CLAUDE.md` — they load for all file types but only "apply" when relevant
- B) Create `.claude/rules/infra.md` with `paths: ["terraform/**/*"]` in the YAML frontmatter
- C) Create `terraform/CLAUDE.md` with the Terraform conventions
- D) Use `@import` to conditionally include Terraform conventions based on the file type

---

### 3.4 — Plan Mode vs. Direct Execution (3 questions)

**Q9.** The team needs to refactor the legacy `auth` module — 8,000 lines across 45 files, with a choice between extracting it to a separate microservice or modernizing it in-place. Which mode should be used?

- A) Direct execution — experienced engineers can execute refactors directly without planning
- B) Plan mode — large-scale changes with multiple valid approaches and architectural decisions require exploration before committing to changes
- C) Plan mode for the first 10 files; direct execution for the rest
- D) Direct execution with a `--dry-run` flag to preview changes before applying them

---

**Q10.** A developer finds a null pointer exception in the application logs. The stack trace points to line 47 of `api/src/auth/session.ts`: `user.profile.settings.theme`. The fix requires adding a null check for `user.profile`. Which mode is appropriate?

- A) Plan mode — any change to the auth module requires careful planning due to its size
- B) Plan mode — null pointer fixes can have cascading effects that need to be mapped first
- C) Direct execution — the scope is clear (one line in one file), the change is well-understood, and no architectural decisions are required
- D) Plan mode — the legacy auth module has no tests, so all changes are risky

---

**Q11.** During the auth module refactor, the discovery phase involves tracing all call chains, mapping module dependencies, and cataloging all exported interfaces — producing hundreds of lines of intermediate output. You need this discovery output to inform the implementation plan without consuming the implementation session's context budget. The correct approach is:

- A) Run the discovery in the main session and start a new session for implementation
- B) Use the Explore subagent to run the verbose discovery phase — it returns a summary to the main session rather than filling the main context with raw output
- C) Use plan mode for discovery and direct execution for implementation
- D) Save the discovery output to a file and read it at the start of the implementation session

---

### 3.5 — Iterative Refinement (3 questions)

**Q12.** A developer asks Claude Code to transform CSV data into a specific JSON format. Despite several tries with different prose descriptions, the output is inconsistent — sometimes the structure is right but the field names are wrong, other times vice versa. The most effective fix is:

- A) Use more precise adjectives in the description: "exactly correct," "precisely formatted"
- B) Provide 2-3 concrete examples showing a sample CSV input and the exact expected JSON output
- C) Break the transformation into two steps: first transform the structure, then rename the fields
- D) Switch to plan mode to design the transformation algorithm before implementing it

---

**Q13.** A developer needs to implement a cache invalidation strategy for the `auth` module's session tokens, but has never worked with distributed caching before. Before Claude Code starts writing code, what technique should be used?

- A) Provide detailed implementation requirements in the prompt so Claude Code can implement without asking questions
- B) Use the interview pattern — have Claude Code ask clarifying questions to surface cache invalidation strategies, TTL requirements, failure modes, and consistency requirements before implementing
- C) Ask Claude Code to implement a "standard" cache invalidation strategy and refine from there
- D) Run Claude Code in plan mode to enumerate all possible approaches before implementing

---

**Q14.** A developer finds four bugs in the auth module: two that interact with each other (bug A changes session state that bug B reads) and two that are independent (C and D). What is the most effective iterative refinement strategy?

- A) Fix all four bugs in a single detailed message describing each one
- B) Fix the interacting bugs (A and B) together in a single message; fix C and D sequentially in separate follow-up messages
- C) Fix all four bugs sequentially, one at a time, in separate messages
- D) Fix the independent bugs first (C and D), then fix the interacting bugs (A and B) together

---

### 2.5 — Built-in Tool Selection (2 questions)

**Q15.** You need to find every place in the monorepo where the function `validateAuthToken()` is called. Which built-in tool is correct?

- A) Glob with pattern `**/*.ts` — finds all TypeScript files, then check each one
- B) Grep — searches file contents across the codebase for the pattern `validateAuthToken`
- C) Read — reads each TypeScript file and scans for the function name
- D) Bash with `find . -name "*.ts"` to locate files, then check each

---

**Q16.** Claude Code is trying to update a specific code block in `api/src/auth/session.ts`. The target code block appears in a standard pattern that is repeated 4 times in the file. The Edit tool fails because it cannot find a unique match. What is the correct fallback?

- A) Use Bash to run a `sed` replacement command
- B) Add more surrounding context to the Edit call until the match becomes unique
- C) Use Read to load the full file contents, make the targeted modification, then use Write to save the updated file
- D) Split the file into smaller files so each code block is in its own file

---

### 5.4 — Context in Large Codebase Exploration (2 questions)

**Q17.** After 3 hours of exploring the legacy auth module, a developer notices Claude Code's answers have become generic — referencing "typical session management patterns" instead of the specific `AuthSessionManager` class it found earlier. What is the most effective immediate fix?

- A) Start a fresh session with a detailed description of what was found so far
- B) Use `/compact` to reduce context usage and have the agent continue from a scratchpad file (`auth-findings.md`) that recorded the specific classes and findings earlier in the session
- C) Ask Claude Code to re-read all the auth module files it has already read
- D) Increase the `max_tokens` parameter to give the model more space to reason

---

**Q18.** The team wants to map all dependencies of the `auth` module as a discovery step before starting the refactor. This mapping will read 50+ files and produce extensive intermediate output. The best approach to avoid polluting the implementation session's context is:

- A) Do the discovery mapping in the main session and then start a new session for implementation
- B) Spawn a subagent specifically tasked with the dependency mapping — it investigates and returns a structured summary to the main session
- C) Run the discovery phase in plan mode and the implementation in direct execution mode
- D) Use Glob to list all files first, then read only the most important ones

---

### 1.7 — Session Management (2 questions)

**Q19.** A developer spent 2 hours analyzing the auth module on Monday, naming the session "auth-refactor." It's now Tuesday. They want to continue from where they left off, incorporating Monday's findings. What is the correct command?

- A) `claude --new-session auth-refactor` — creates a fresh session with the same name
- B) `claude --resume auth-refactor` — resumes the named session with its existing context
- C) `claude --load auth-refactor` — loads the session's memory files
- D) `claude --continue` — automatically continues the most recent session

---

**Q20.** After analyzing the auth module, two viable refactoring approaches emerge from the same analysis baseline: (A) extract auth to a microservice, and (B) modernize in-place. The team wants to explore both independently before choosing one. The correct approach is:

- A) Start two new sessions from scratch — one for each approach
- B) Use `fork_session` to create two independent branches from the shared analysis baseline, then explore each approach in its own branch
- C) Explore approach A first; if it doesn't work, switch to approach B in the same session
- D) Create a new git branch for each approach and start separate sessions on each branch

---

## 8. Answer Key

**Q1: B**
User-level CLAUDE.md (`~/.claude/CLAUDE.md`) is personal — it's never committed to git and is invisible to other team members. This is the most common onboarding misconfiguration. The fix is to move team conventions to a project-level CLAUDE.md that is version-controlled.

**Q2: C**
`@import` directives allow the root CLAUDE.md to selectively include only the relevant standards file for each package, keeping each package's configuration focused without duplication. A monolithic root file (A) loads everything for everyone. Separate files without linking (B) duplicate shared content. Separate branches (D) create an unmaintainable configuration split.

**Q3: B**
The `/memory` command is specifically designed to show which memory files are currently loaded and their loading order, making it the direct diagnostic tool for CLAUDE.md hierarchy issues.

**Q4: B**
`context: fork` runs the skill in an isolated sub-agent context. The verbose analysis output stays in that isolated context; only a summary is returned to the main conversation. This preserves the main session's context for subsequent work.

**Q5: C**
`allowed-tools` in the skill frontmatter is a deterministic restriction — it configures which tools the skill can access. A warning in the description (A) is probabilistic. Confirmation prompts (B) add friction to every run. Git branching (D) is a review mechanism, not a prevention mechanism.

**Q6: B**
`argument-hint` in frontmatter causes Claude Code to prompt the developer for the required parameter when the skill is invoked without arguments — the correct, built-in mechanism for this use case. In-skill validation (A) happens after invocation. Auto-defaulting to the current file (C) may not be the intended behavior. Wiki documentation (D) doesn't prevent the blank-output problem.

**Q7: C**
A `.claude/rules/` file with `paths: ["**/*.test.tsx"]` loads the rules only when editing matching files, regardless of which directory they're in. This is a single source of truth for the convention. Root CLAUDE.md (A) loads testing rules for all file types, including production code. Separate per-directory files (B) duplicate the rules. Per-package `@import` (D) still requires adding the import to each package file.

**Q8: B**
`.claude/rules/infra.md` with `paths: ["terraform/**/*"]` activates only when editing Terraform files. This keeps Terraform conventions out of the context when developers are writing TypeScript. Root CLAUDE.md (A) loads Terraform conventions for every file type. A directory-level `terraform/CLAUDE.md` (C) achieves the same result but is less flexible for conventions spanning multiple terraform sub-directories.

**Q9: B**
Plan mode is designed for exactly this situation: large-scale changes, multiple valid approaches (microservice extraction vs. in-place modernization), and architectural decisions that require exploration before commitment. Executing directly (A, C, D) risks costly rework if the wrong approach is chosen after 45 files have been changed.

**Q10: C**
The scope is completely clear: one null check, one line, one file, no architectural decisions. Plan mode adds unnecessary exploration overhead for a well-understood, single-file change.

**Q11: B**
The Explore subagent is specifically designed for verbose discovery phases — it does the exploration work and returns a structured summary to the main session without filling the main session's context with hundreds of lines of intermediate output. Starting a new session (A) loses the context connection. Plan/direct split (C) is about the planning phase, not about isolating verbose output. Reading from a saved file (D) is a workaround, not the intended pattern.

**Q12: B**
Concrete input/output examples communicate the exact transformation far more reliably than any prose description, however precise. Examples eliminate ambiguity about field names, nesting structure, and data types simultaneously. Precision adjectives (A) don't reduce ambiguity. Splitting into two steps (C) adds complexity without addressing the core communication problem. Plan mode (D) is not the right tool for a formatting consistency problem.

**Q13: B**
The interview pattern — having Claude Code ask clarifying questions first — is specifically designed for unfamiliar domains where the developer may not have anticipated important design considerations (TTL policies, eviction strategies, consistency modes, failure handling). Providing detailed upfront requirements (A) assumes the developer already knows what they don't know. "Standard strategy" (C) may not fit the specific constraints. Plan mode (D) helps with approach comparison, not with eliciting unknown requirements.

**Q14: B**
Interacting bugs (A and B) must be fixed together because fixing one may affect the other — they share state. Independent bugs (C and D) can be fixed sequentially without risk of interference. Mixing all four in one message (A) conflates interacting and independent issues. Pure sequential (C) risks reintroducing bug A when fixing bug B. Fixing independent bugs first (D) is unnecessarily constrained.

**Q15: B**
Grep searches file contents — it finds the string `validateAuthToken` wherever it appears in any file. Glob (A) matches file paths/names, not content. Read (C) would require reading every file individually. Bash find (D) also finds files by path, not by content.

**Q16: C**
When Edit fails due to a non-unique match, Read + Write is the correct fallback: read the full file, make the specific modification in memory, write the updated content back. This is reliable regardless of how many times the target pattern repeats. Bash sed (A) works but is less idiomatic. Adding more context to Edit (B) may work but becomes fragile. Splitting the file (D) is a major structural change to fix a tool limitation.

**Q17: B**
`/compact` reduces the context by summarizing verbose history, and the scratchpad file (`auth-findings.md`) provides a persistent record of specific findings that survives compaction. Together they address both the token budget problem and the context degradation problem. A fresh session (A) loses all accumulated context. Re-reading all files (C) wastes the remaining context budget. Increasing `max_tokens` (D) doesn't fix context degradation — it delays it.

**Q18: B**
A subagent investigates and returns a structured summary — the exploration output stays in the subagent's context, not the main session's. The main session receives only the actionable findings. Doing it in the main session (A) and then starting a new session (A) loses the context connection. Plan/direct split (C) doesn't address the context pollution problem. Glob + selective reading (D) avoids the subagent pattern entirely but doesn't prevent context filling.

**Q19: B**
`--resume <session-name>` resumes the named session with all its existing context. `--new-session` (A) would create a fresh session, losing Monday's work. `--load` and `--continue` (C, D) are not the correct flags.

**Q20: B**
`fork_session` creates two independent branches from the shared analysis baseline — both approaches start from the same understanding of the auth module, ensuring a fair comparison without duplicating the analysis work. Starting from scratch (A) means each branch redoes all the exploration independently. Sequential exploration (C) in the same session means the second approach is influenced by the first approach's investigation. Git branching (D) separates the codebase but not the Claude Code session context.

---

## 9. Quick-Recall Cheat Sheet

**CLAUDE.md hierarchy (3.1)**
- `~/.claude/CLAUDE.md` = personal, not version-controlled, invisible to teammates
- Project-level `CLAUDE.md` = in git = team-wide
- `@import` = modular standards loading per package
- `/memory` = diagnostic command to see which files are loaded

**Skills & commands (3.2)**
- `context: fork` = skill runs isolated; main session stays clean
- `allowed-tools` = deterministic restriction on which tools a skill can use
- `argument-hint` = prompts developer for required parameters on invocation
- Skills in `~/.claude/skills/` = personal; `.claude/skills/` = team-shared

**Path-specific rules (3.3)**
- `.claude/rules/<name>.md` with `paths: ["glob/**"]` = activates only for matching files
- Use for conventions that span multiple directories (test files, terraform)
- Prefer over directory-level CLAUDE.md when the convention applies by file type, not location

**Plan mode vs. direct execution (3.4)**
- Plan mode: 45+ files, architecture decisions, multiple valid approaches
- Direct execution: single-file fix, clear stack trace, well-understood scope
- Explore subagent: isolates verbose discovery output from main session

**Iterative refinement (3.5)**
- Inconsistent format → concrete input/output examples (not better prose)
- Unfamiliar domain → interview pattern (Claude asks first)
- Interacting bugs → one detailed message; independent bugs → sequential

**Built-in tools (2.5)**
- Grep = search file content; Glob = find files by name pattern
- Edit = targeted modification with unique anchor text
- Edit failure → Read + Write fallback

**Context management (5.4)**
- Scratchpad files (`findings.md`) persist key findings across context boundaries
- `/compact` reduces context usage during extended sessions
- Subagents for verbose exploration phases = main session stays clean

**Session management (1.7)**
- `--resume <name>` = continue a named session
- `fork_session` = independent branches from shared baseline
- Files changed since last session → explicitly tell the agent what changed
