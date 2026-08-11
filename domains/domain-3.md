# CCA-F Exam Prep — Domain 3: Claude Code Configuration & Workflows

> **Exam Weight: 20% — ~12 Questions. The Second-Heaviest Domain.**
> This domain tests whether you can configure Claude Code for **real team workflows** — CLAUDE.md hierarchies, custom slash commands, skills, subagents, hooks, plan mode, MCP integration, and CI/CD pipeline automation. Most candidates underestimate it because it "looks like docs reading" — until the exam shows them four configuration files and asks which one a new teammate is missing.

### Exam Quick Facts (Same as Other Domains)

| Item              | Detail                                                       |
| ----------------- | ------------------------------------------------------------ |
| Questions         | 60 multiple-choice (1 correct + 3 distractors)               |
| Duration          | **120 minutes** — no breaks, cannot pause                    |
| Passing Score     | **720 / 1000** (scaled) — no per-domain minimums             |
| Fee               | $125 USD (12-month validity; supersedes launch-period $99 / first-5,000-free terms — see [README Resources](../README.md#resources)) |
| Proctoring        | ProctorFree — webcam required, **no external resources**     |
| Scenarios         | 4 of 6 randomly selected — all questions anchored to those 4 |
| Results           | 2 business days — includes domain breakdown + digital badge  |

---

## Credits & Resources

**Repository owner:** Muhammed Nemathullah Khan

**Author of this domain guide:** [Arun Varadharajalu](https://www.linkedin.com/in/arunv11u/)

**Domain Resources:**
- **Core topics & fundamentals overview:** [CCA-F Study Guide — guide_en.md](https://github.com/paullarionov/claude-certified-architect/blob/main/guide_en.md)
- **Deep dive in each domain:** [Domain 1](./domain-1.md) · [Domain 2](./domain-2.md) · [Domain 3](./domain-3.md) · [Domain 4](./domain-4.md) · [Domain 5](./domain-5.md)
- **Scenarios:** [All 6 exam scenarios](../scenarios/listedScenarios.md)
- **Mock test and sources:** [Mock test bank](./mock-test/) · [Claude Certification Guide — Mock Exam](https://claudecertificationguide.com/mock-exam) · [CyberSkill Practice — CCAF](https://practice.cyberskill.world/practice/ccaf/exam) · [CertSafari — CCAF](https://www.certsafari.com/anthropic/claude-certified-architect-foundations)

---

## Table of Contents

1. [What This Domain Tests](#1-what-this-domain-tests)
2. [The Master Mental Model — Configuration as Layered Onboarding](#2-the-master-mental-model--configuration-as-layered-onboarding)
3. [CLAUDE.md Hierarchy — The 4 Scopes & Load Order](#3-claudemd-hierarchy--the-4-scopes--load-order)
4. [CLAUDE.md Content Patterns — What Goes Where](#4-claudemd-content-patterns--what-goes-where)
5. [Modular Rules — `.claude/rules/` with YAML Frontmatter](#5-modular-rules--claude-rules-with-yaml-frontmatter)
6. [`@imports` and Auto Memory](#6-imports-and-auto-memory)
7. [Custom Slash Commands & Skills](#7-custom-slash-commands--skills)
8. [Skills with `context: fork` — Subagent Isolation in Claude Code](#8-skills-with-context-fork--subagent-isolation-in-claude-code)
9. [Subagents in Claude Code — `.claude/agents/`](#9-subagents-in-claude-code--claude-agents)
10. [Hooks — Lifecycle Events & Configuration](#10-hooks--lifecycle-events--configuration)
11. [Plan Mode — Read-Only Architectural Thinking](#11-plan-mode--read-only-architectural-thinking)
12. [Permission Modes & `settings.json` Scopes](#12-permission-modes--settingsjson-scopes)
13. [Headless Mode (`-p`) — CI/CD Integration](#13-headless-mode--p--cicd-integration)
14. [MCP in Claude Code — `.mcp.json` Integration](#14-mcp-in-claude-code--mcpjson-integration)
15. [Session Management & Compaction](#15-session-management--compaction)
16. [Independent Review Pattern — Generation vs Review Sessions](#16-independent-review-pattern--generation-vs-review-sessions)
17. [Anti-Patterns Master List](#17-anti-patterns-master-list)
18. [Key Rules to Memorize](#18-key-rules-to-memorize)
19. [Practice Questions (25 MCQs)](#19-practice-questions-25-mcqs)
20. [Answer Key & Explanations](#20-answer-key--explanations)
21. [Quick Cheat Sheet — Domain 3](#21-quick-cheat-sheet--domain-3)

---

## 1. What This Domain Tests

| Task Statement | Description                                                                                       |
| -------------- | ------------------------------------------------------------------------------------------------- |
| 3.1            | Configure CLAUDE.md with hierarchy, scoping, and organization for personal vs team use            |
| 3.2            | Design custom slash commands and skills for repeatable team workflows                             |
| 3.3            | Configure subagents in `.claude/agents/` with scoped tools and isolated context                   |
| 3.4            | Apply Plan Mode for architectural and exploration tasks where read-only thinking is required      |
| 3.5            | Configure hooks for lifecycle interception (PreToolUse, PostToolUse, SessionStart, Stop, etc.)    |
| 3.6            | Integrate Claude Code into CI/CD using headless mode (`-p` flag) with structured output           |
| 3.7            | Integrate MCP servers into Claude Code via `.mcp.json` for team-shared tools                      |
| 3.8            | Manage session state, context isolation, compaction, and the generation vs review boundary       |

### Domain 3 in the Exam Scenarios

| Scenario                          | Domain 3 Focus                                                            |
| --------------------------------- | ------------------------------------------------------------------------- |
| **Code Generation with Claude Code** | CLAUDE.md hierarchy, plan mode, session management — PRIMARY scenario  |
| **Claude Code for CI/CD**         | Headless mode (`-p`), structured output, exit codes, secrets — PRIMARY    |
| **Developer Productivity Tools**  | Task-scoped tool profiles, skills, subagents, custom slash commands       |
| **Multi-Agent Research System**   | Skills with `context: fork` to isolate verbose exploration                |

---

## 2. The Master Mental Model — Configuration as Layered Onboarding
> ⭐ **The single most important framing for Domain 3.**

Every Claude Code configuration file is **onboarding documentation for an engineer with perfect recall but total amnesia**. Every session, Claude starts fresh — it reads your config files like a new hire reading the README. That mental model resolves nearly every exam question:

```
"Where should this instruction live?"
→ Ask: who needs to see it? Just you? → user-level. Whole team? → project-level.

"Why doesn't a teammate's Claude follow our conventions?"
→ Ask: where was the instruction written? If it's only in someone's
   ~/.claude/CLAUDE.md, it never reaches anyone else.

"Why does context bloat when running a long exploration?"
→ Ask: did the verbose output go into the main session, or into an
   isolated subagent? If main session, you need context: fork or a subagent.

"Why does the CI/CD pipeline hang?"
→ Ask: is Claude waiting for interactive input? You forgot the -p flag.
```

### The Decision Tree

```
Personal preference, only on my machine
  → ~/.claude/CLAUDE.md or ~/.claude/settings.json

Team standard, shared with everyone who clones the repo
  → ./CLAUDE.md (or .claude/CLAUDE.md) + .claude/settings.json + .mcp.json
  → MUST be committed to version control

Convention that only applies to specific file types
  → .claude/rules/<topic>.md with paths: [...] in YAML frontmatter

Reusable workflow you'll invoke repeatedly with one command
  → .claude/skills/<name>/SKILL.md (preferred) or .claude/commands/<name>.md

Specialized task with verbose intermediate output
  → .claude/agents/<name>.md (subagent with isolated context)

Deterministic enforcement (block dangerous commands, normalize data)
  → Hook in .claude/settings.json (PreToolUse / PostToolUse)

Long-running architectural exploration where you don't want writes
  → Plan Mode (--permission-mode plan or /plan)

CI/CD or scripted invocation
  → claude -p "prompt" --output-format json
```

### The Meta-Pattern (Carries Over from Domains 1 & 2)

> **Every Domain 3 wrong answer substitutes a prompt instruction for a structural fix.**

If the requirement is "always," "never," "must," "every time," or "guarantee" → the answer is a **structural** mechanism (hook, permission mode, skill scoping, separate session), not a CLAUDE.md instruction.

---

## 3. CLAUDE.md Hierarchy — The 4 Scopes & Load Order
> ⭐ **The most heavily tested Domain 3 topic.** Memorize every scope, its path, and how files combine.

Claude Code reads CLAUDE.md files from multiple locations at the start of every session and injects them into the conversation **as a user message after the system prompt**. There are **four scopes**, loaded from broadest to most specific. Critically, the files are **concatenated, not overridden** — all discovered files end up in context, ordered so that more-specific instructions are read *last*.

### The 4 Scopes (Official)

| Scope | Path | Shared With | Committed? | Use For |
| ----- | ---- | ----------- | ---------- | ------- |
| **1. Managed policy** | macOS: `/Library/Application Support/ClaudeCode/CLAUDE.md`<br>Linux/WSL: `/etc/claude-code/CLAUDE.md`<br>Windows: `C:\Program Files\ClaudeCode\CLAUDE.md` | All users in org (deployed by IT) | N/A (controlled by IT) | Compliance, security rules, "never use raw SQL" |
| **2. User instructions** | `~/.claude/CLAUDE.md` | Just you, all your projects | ❌ No | Personal preferences, your tooling shortcuts |
| **3. Project instructions** | `./CLAUDE.md` or `./.claude/CLAUDE.md` | Team via source control | ✅ Yes | Project architecture, coding standards, build commands |
| **4. Local instructions** | `./CLAUDE.local.md` (project root) | Just you, this project | ❌ No (gitignored) | Your sandbox URLs, personal test data *(deprecated — see note below)* |

> **Deprecation note:** Per the Anthropic docs, `CLAUDE.local.md` is **deprecated in favor of `@imports`** because imports work better across multiple git worktrees (a `CLAUDE.local.md` only exists in the worktree where you created it). The preferred modern pattern is to put personal notes in `~/.claude/my-project-instructions.md` and import them from project CLAUDE.md: `@~/.claude/my-project-instructions.md`. The exam may still test `CLAUDE.local.md` since it's documented and functional, but recognize the import pattern as the current recommendation.

### Load Order — Concatenation, Not Override

The official docs are explicit: "**All discovered files are concatenated into context rather than overriding each other.**" Files are ordered from filesystem root down to your working directory, so instructions closer to where you launched Claude are read **last**.

```
Order in context (top → bottom):
  1. Managed policy CLAUDE.md (if present)
  2. User CLAUDE.md (~/.claude/CLAUDE.md)
  3. Project CLAUDE.md from root, then each subdirectory walking down
  4. Within each directory: CLAUDE.local.md is appended AFTER CLAUDE.md
```

**Practical implication:** When two instructions contradict, Claude may pick one arbitrarily. The fix is not "rely on precedence" — it's to **avoid contradictions across layers**. The only guaranteed within-directory precedence is that `CLAUDE.local.md` loads AFTER `CLAUDE.md` (so personal notes are the last thing read at that level).

### Subdirectory Loading — On-Demand

CLAUDE.md files in **subdirectories below your working directory** are NOT loaded at launch. They load **on demand** when Claude reads a file in that subdirectory. This is the official monorepo-friendly behavior: a frontend/ CLAUDE.md only enters context when Claude touches a file in frontend/.

### The Single Most-Tested Exam Scenario

> A new developer joins the team. They clone the repo, run Claude Code, but it doesn't follow the project's established coding conventions. **What is the most likely root cause?**

The correct answer is almost always: **the conventions were written in someone's user-level `~/.claude/CLAUDE.md` instead of the project-level `./CLAUDE.md`**. User-level files are not committed to git, so they never reach a new teammate.

```
❌ WRONG diagnoses (common exam distractors):
- "The new developer needs to upgrade Claude Code"
- "Run /memory to manually load the conventions"
- "Anthropic's servers haven't synced the team's settings"
- "The developer should ask each team member to share their config"

✅ CORRECT diagnosis:
- "Team conventions belong in project-level ./CLAUDE.md committed
   to git — moving them there is the structural fix."
```

### The `/memory` Verification Command

```
/memory   → lists all CLAUDE.md, CLAUDE.local.md, and rules files
            currently loaded in the session; toggles auto memory;
            opens any listed file in your editor.
```

When debugging "why isn't my rule applying?", `/memory` is the authoritative source of what Claude actually loaded.

### `/init` — Generate a Starting CLAUDE.md

Run `/init` and Claude analyzes the codebase and creates a starter CLAUDE.md with build commands, test instructions, and conventions it discovers. If a CLAUDE.md already exists, `/init` **suggests improvements** rather than overwriting. Set `CLAUDE_CODE_NEW_INIT=1` for the interactive multi-phase flow that also proposes skills and hooks.

### No Restart Needed

After editing CLAUDE.md, Claude picks up changes **on the next session start** for files loaded at startup. For interactive edits during an active session, run `/memory` and re-open the session, or note that the change applies to your next conversation. (Path-scoped rules and subdirectory CLAUDE.md re-load when matching files are next read.)

### What Survives Compaction

A useful exam fact: **project-root CLAUDE.md survives `/compact`** — Claude re-reads it from disk and re-injects it after the conversation is summarized. **Nested subdirectory CLAUDE.md files do NOT auto-reinject after compaction**; they reload only when Claude next reads a file in that subdir. If an instruction "disappears" after compaction, it was either conversation-only or lives in a nested file that hasn't reloaded yet.

### `claudeMdExcludes` — Monorepo Escape Hatch

In large monorepos, ancestor CLAUDE.md files from unrelated teams can clutter context. The `claudeMdExcludes` setting in `.claude/settings.local.json` lets you skip them by glob:

```json
{
  "claudeMdExcludes": [
    "**/monorepo/CLAUDE.md",
    "/home/user/monorepo/other-team/.claude/rules/**"
  ]
}
```

**Note:** Managed-policy CLAUDE.md files cannot be excluded — organization-wide instructions always apply.

---

## 4. CLAUDE.md Content Patterns — What Goes Where

Each scope should contain only what its purpose justifies. Mixing scopes is the #1 cause of "why isn't Claude following our convention?" tickets.

### Project-Level CLAUDE.md — Team Conventions Only

```markdown
# Project: PaymentService

## Build & Test
- Use `pnpm` (not npm/yarn — pnpm is our standard)
- Run `pnpm test:unit` before committing
- All TypeScript files must pass strict type checking

## Architecture
- Adapter pattern for JWT — see src/auth/jwt-adapter.ts
- Public APIs return Result<T, Error> — never throw across boundaries

## Database
- Always use Prisma for queries — never raw SQL
- Migrations in /prisma/migrations — never modify existing migration files

## Testing
- Co-locate tests next to source files (*.test.ts)
- Vitest, not Jest
```

### User-Level CLAUDE.md — Personal Preferences

```markdown
# Personal preferences across all projects

- Prefer concise comments — assume the reader is experienced
- When suggesting refactors, propose 2-3 options with tradeoffs
- I use vim keybindings — don't suggest VS Code shortcuts
```

### Subdirectory CLAUDE.md — Component Rules (loads on-demand)

```markdown
# src/api/CLAUDE.md — API conventions (only loads when Claude touches src/api/)

- All routes use Zod validation on input
- Responses follow JSON:API spec
- Error responses must include error.code, error.message, error.requestId
```

### What Does NOT Belong in CLAUDE.md

| Content                                        | Why Not                                                                  | Where Instead                                |
| ---------------------------------------------- | ------------------------------------------------------------------------ | -------------------------------------------- |
| API keys, tokens, secrets                      | Committed to git → security breach                                       | Environment variables, `.env`, secret stores |
| Long step-by-step procedures used occasionally | Bloats context on every session                                          | A Skill (loads on demand)                    |
| Verbose tool documentation                     | Already in tool descriptions                                             | Tool descriptions themselves                 |
| "Always do X" safety rules                     | Prompts are probabilistic — won't be enforced 100%                       | A PreToolUse hook (deterministic)            |
| Personal shortcuts only meaningful to you      | Confuses teammates if put in project CLAUDE.md                           | `~/.claude/CLAUDE.md` or `CLAUDE.local.md`   |
| Component-specific rules for one subdir        | Loads on every session even when irrelevant                              | Subdir CLAUDE.md or `.claude/rules/` with paths: |

### The 200-Line CLAUDE.md Rule
> ⭐ **Directly from official docs:** "Target under 200 lines per CLAUDE.md file. Longer files consume more context and reduce adherence."

When a project CLAUDE.md grows past 200 lines, it bloats every session's context with rules that don't apply to most tasks. The fix is **not** trimming the file by deletion — it's **restructuring**:

```
Keep in CLAUDE.md (always loaded):
  - Coding standards (apply to every change)
  - Testing conventions (apply to every change)
  - Project architecture overview (always relevant)

Move to .claude/rules/ with path globs (conditional load):
  - PR review checklist → .claude/rules/pr-review.md
  - Deployment guidance → .claude/rules/deployment.md (paths: ["deploy/**"])
  - Database migration rules → .claude/rules/migrations.md (paths: ["prisma/**"])

Move to skills (load only when invoked):
  - Multi-step procedures → .claude/skills/<name>/SKILL.md
```

The exam asks this directly: "Your CLAUDE.md is over 400 lines. You want Claude to always follow coding standards but only apply PR review, deployment, and migration guidance when performing those tasks. What's the most effective restructuring?" → **Split into a lean CLAUDE.md plus topic-specific files in `.claude/rules/` with `paths:` globs.**

### AGENTS.md — Cross-Tool Compatibility

If your repo already uses `AGENTS.md` for other coding agents, **Claude Code reads `CLAUDE.md`, not `AGENTS.md`**. Bridge them with an import or symlink:

```markdown
@AGENTS.md

## Claude Code

Use plan mode for changes under `src/billing/`.
```

Or symlink: `ln -s AGENTS.md CLAUDE.md` (requires Admin/Developer Mode on Windows).

---

## 5. Modular Rules — `.claude/rules/` with YAML Frontmatter
> ⭐ **Exam-tested: conditional rule loading by file path glob.**

Files in `.claude/rules/<topic>.md` can include a YAML frontmatter that tells Claude **when** to load that rule. **Without** `paths:` frontmatter, the rule loads at launch with the same priority as `.claude/CLAUDE.md`. **With** `paths:`, the rule loads conditionally when Claude reads files matching the glob.

```yaml
---
paths:
  - "**/*.test.tsx"
  - "**/*.test.ts"
---

# Test Conventions

- Use `describe` blocks for grouping; `it` for individual tests
- Mock external dependencies with vi.mock — never with manual stubs
- Test names follow "should [behavior] when [condition]"
- Each test file must include at least one boundary-condition test
```

```yaml
---
paths:
  - "src/api/**/*"
---

# API Conventions

- All routes validated with Zod
- Responses follow JSON:API spec
- Logging via the request-scoped logger, not console.*
```

### Glob Pattern Reference (Official)

| Pattern                | Matches                                  |
| ---------------------- | ---------------------------------------- |
| `**/*.ts`              | All TypeScript files anywhere            |
| `src/**/*`             | All files under `src/`                   |
| `*.md`                 | Markdown files in project root           |
| `src/components/*.tsx` | React components in a specific directory |
| `src/**/*.{ts,tsx}`    | Multiple extensions via brace expansion  |

### Why This Matters

- Context stays lean — irrelevant conventions don't load on every operation
- Component-specific rules can be authored and reviewed independently
- New developers see exactly which rules apply to which areas

### User-Level Rules

Personal rules in `~/.claude/rules/` apply to every project on your machine. Per the official docs, user-level rules are **loaded before project rules**, giving project rules higher priority on conflict.

### Symlinks for Shared Rules

`.claude/rules/` supports symlinks. To share a rules pack across projects:

```bash
ln -s ~/shared-claude-rules .claude/rules/shared
ln -s ~/company-standards/security.md .claude/rules/security.md
```

### Exam Pattern

A question describes a team whose CLAUDE.md is loading test conventions, deployment rules, and PR review checklists on every prompt — and context fills up before useful work begins. **The fix is `.claude/rules/` with `paths:` frontmatter so each rule loads conditionally.**

---

## 6. `@imports` and Auto Memory

### `@import` Syntax

CLAUDE.md files can compose content from other files using `@path/to/file` syntax:

```markdown
See @README.md for project overview and @package.json for npm commands.

# Additional Instructions
- Git workflow: @docs/git-instructions.md
- Personal: @~/.claude/my-instructions.md
```

**Key rules:**

- Both relative and absolute paths allowed
- Relative paths resolve relative to the **importing file**, not the working directory
- Imports recurse up to **5 levels deep**
- First-time imports from external locations trigger a one-time approval dialog
- Imports inside markdown code blocks are NOT evaluated (safe to document them as examples)
- Imported files still load at launch — `@import` is for organization, not for reducing context

### Auto Memory (Claude's Own Notebook)

> ⭐ Auto Memory was added in Claude Code v2.1.59 — directly tested as part of "persistent project context."

Distinct from CLAUDE.md (which **you** write), auto memory is what **Claude** writes about itself as it works:

| | CLAUDE.md | Auto memory |
| -- | --------- | ----------- |
| **Who writes it** | You | Claude |
| **Contains** | Instructions and rules | Learnings and patterns Claude discovers |
| **Loaded into** | Every session (full) | Every session (first 200 lines or 25KB of `MEMORY.md`) |
| **Storage** | Project repo / home dir | `~/.claude/projects/<project>/memory/` |
| **Use for** | Coding standards, workflows, architecture | Build commands Claude figured out, debugging insights, your corrections |

Enable/disable via `/memory` or the `autoMemoryEnabled` setting; force-disable with `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1`. Auto memory is **machine-local** — not shared across machines.

When you tell Claude "always use pnpm, not npm" — Claude writes that to auto memory. When you say "add this to CLAUDE.md," it goes to CLAUDE.md instead. Both are plain markdown you can audit and edit.

---

## 7. Custom Slash Commands & Skills

### Custom Slash Commands

A custom slash command is a reusable prompt template invoked by typing `/command-name`. Files live in:

- **Project:** `.claude/commands/<name>.md` — shared with team via git
- **User:** `~/.claude/commands/<name>.md` — personal only

```markdown
# .claude/commands/review.md
Review the most recent git diff for:
- Security issues (SQL injection, auth bypass, secrets in code)
- Performance regressions
- Missing or broken tests
- Style inconsistencies with our coding standards

Output findings in three sections: Critical, Warnings, Suggestions.
```

Invoke with `/review`. Claude reads the command file and treats it as the prompt.

### Skills — The Successor to Custom Commands
> ⭐ **Exam-tested terminology shift: Skills now subsume slash commands.**

Per the official docs: **"Custom commands have been merged into skills."** A file at `.claude/commands/deploy.md` and a skill at `.claude/skills/deploy/SKILL.md` both create `/deploy` and work the same way. Existing `.claude/commands/` files keep working — but Skills add capabilities:

| Capability                                | Custom Command  | Skill |
| ----------------------------------------- | --------------- | ----- |
| Invoke with `/name`                       | ✅              | ✅    |
| **Auto-invocation by Claude**             | Per-file frontmatter | ✅ (default — when description matches user request) |
| YAML frontmatter (tools, hooks)           | Limited         | ✅ Full |
| Supporting files in a directory           | ❌              | ✅    |
| Subagent execution (`context: fork`)      | ❌              | ✅    |
| Path-conditional loading (`paths:`)       | ❌              | ✅    |
| Pre-approved tools (`allowed-tools:`)     | Limited         | ✅    |
| `disable-model-invocation` toggle         | ❌              | ✅    |

If both a command and a skill exist for the same name, **the skill takes precedence** (official).

### Skill Anatomy (Official)

```yaml
---
name: deploy-check
description: Validate deployment readiness with explicit gates. Use when the user asks to ship, deploy, or release.
disable-model-invocation: true   # Only runs when human explicitly invokes /deploy-check
allowed-tools: Bash(pnpm test:unit) Bash(pnpm typecheck) Read
---

# Deploy Readiness Check

Run the following gates in order. Stop and report on first failure:

1. Run `pnpm test:unit` — all tests must pass
2. Run `pnpm typecheck` — no type errors
3. Check `CHANGELOG.md` has an entry for the current version
4. Verify no `.env` files are staged for commit
5. Confirm the current branch is not main

Report PASS / FAIL with a one-line reason for each gate.
```

### Where Skills Live (Official)

| Location   | Path                                       | Applies to              |
| :--------- | :----------------------------------------- | :---------------------- |
| Enterprise | Managed settings (deployed by IT)          | All users in org        |
| Personal   | `~/.claude/skills/<skill-name>/SKILL.md`   | All your projects       |
| Project    | `.claude/skills/<skill-name>/SKILL.md`     | This project only       |
| Plugin     | `<plugin>/skills/<skill-name>/SKILL.md`    | Where plugin is enabled |

**Precedence when names collide:** enterprise > personal > project. Plugin skills use a `plugin-name:skill-name` namespace so they can't conflict.

### `disable-model-invocation: true` — When to Use It
> ⭐ **Exam-tested for high-risk operations.**

By default, Claude can **auto-invoke** a skill when it judges the description matches the user's intent. For high-risk operations (`deploy`, `delete`, `migrate`, `truncate`, `commit`, anything with side effects on production) — set `disable-model-invocation: true` so the skill only fires when a human explicitly types `/deploy`. This prevents Claude from auto-deploying when a user says "let's get this ready to ship."

The official docs are explicit on this: *"You don't want Claude deciding to deploy because your code looks ready."*

### `user-invocable: false` — The Opposite Case

Set `user-invocable: false` to make a skill invocable **only by Claude** (not by typing `/name`). Use this for background knowledge that isn't a meaningful user-facing command — e.g., a `legacy-system-context` skill that explains how an old system works.

### Live Change Detection

Per official docs, Claude Code **watches skill directories**. Adding, editing, or removing a skill under `~/.claude/skills/`, project `.claude/skills/`, or an `--add-dir` directory takes effect **within the current session, no restart**. Creating a brand-new top-level `skills/` directory that didn't exist at startup requires restarting.

### Dynamic Context Injection — The `!`command`` Pattern

Skills support shell command interpolation that runs **before** Claude sees the skill:

````yaml
---
name: pr-summary
description: Summarize changes in a pull request
context: fork
agent: Explore
allowed-tools: Bash(gh *)
---

## Pull request context
- PR diff: !`gh pr diff`
- PR comments: !`gh pr view --comments`
- Changed files: !`gh pr diff --name-only`

## Your task
Summarize this pull request...
````

The `` !`gh pr diff` `` runs first, output is inserted, then Claude sees the rendered prompt with actual data. This is **preprocessing**, not something Claude executes.

For multi-line, use the fenced form opened with ` ```! `:

````markdown
## Environment
```!
node --version
npm --version
git status --short
```
````

To disable shell execution globally, set `disableSkillShellExecution: true` in settings.

---

## 8. Skills with `context: fork` — Subagent Isolation in Claude Code
> ⭐ **Heavily exam-tested.** This is how Claude Code does the "subagent context isolation" pattern from Domain 1.

When a skill includes `context: fork` in its frontmatter, it executes in a **separate, isolated subagent context** — not in the main session. The subagent runs its own loop, completes the task, and returns **only a summary** to the parent session.

```yaml
---
name: dependency-audit
description: Scan the codebase for outdated dependencies and security advisories
context: fork                # Run in a forked subagent context
agent: Explore               # Use the built-in read-only Explore agent
allowed-tools: Bash(pnpm audit) Read Grep Glob
---

# Dependency Audit Task

Scan package.json, pnpm-lock.yaml, and report:
- Packages with known CVEs (run `pnpm audit`)
- Packages more than 2 major versions behind latest
- Unused dependencies (check imports in src/)

Return a structured summary: { critical: [], outdated: [], unused: [] }
```

### Why `context: fork` Matters

| Without `context: fork`                                                  | With `context: fork`                                                  |
| ------------------------------------------------------------------------ | --------------------------------------------------------------------- |
| Audit output (1,000+ lines) lands in main session                        | Audit runs in isolated subagent; only the summary returns             |
| Main session's context window bloats — accelerates context rot           | Main session stays lean — verbose output is contained                 |
| Every subsequent prompt pays the token cost of intermediate detail       | Subsequent prompts only pay for the condensed summary                 |

### Built-in Agent Types for `agent:` (Official)

- **`Explore`** — Haiku model; read-only search; **skips CLAUDE.md and git status** to keep its context tiny (best for fast codebase exploration)
- **`Plan`** — inherits parent model; read-only; **also skips CLAUDE.md and git status**; used for plan-mode research
- **`general-purpose`** — inherits parent model; all tools available; **loads full CLAUDE.md hierarchy** (default if `agent:` omitted)
- Any **custom subagent** from `.claude/agents/` — loads full CLAUDE.md hierarchy unless the agent definition opts out

> ⭐ **Subagent context loading rule:** ONLY built-in `Explore` and `Plan` skip CLAUDE.md and git status. ALL other subagents — including `general-purpose` and any custom subagent — load the full memory hierarchy at startup. There is no frontmatter field to change this behavior; it's hardcoded for Explore/Plan.

**Exam trap:** Misconfigured pairings silently fail. If a skill needs to write files but uses `agent: Explore` (read-only), it returns nothing useful. **Match the agent type to the skill's actual job.**

### Skills Without an Actionable Task — A Trap (Official Warning)

> Per official docs: *"`context: fork` only makes sense for skills with explicit instructions. If your skill contains guidelines like 'use these API conventions' without a task, the subagent receives the guidelines but no actionable prompt, and returns without meaningful output."*

```yaml
---
name: api-style
description: Our internal API conventions and standards
context: fork              # ⚠️ WRONG — this skill is reference material, not a task
---

# Use these API conventions:
- Always validate with Zod
- Return JSON:API responses
```

Reference-only skills should run in the main context (no `context: fork`).

### Skill Content Lifecycle

Per official docs, once a skill loads, **its content stays in context across turns**. Claude Code does NOT re-read the skill file on later turns. So:

- Write skill content as **standing instructions**, not one-time steps
- Every line in the skill body is a recurring token cost
- If a skill seems to "stop influencing behavior," the content is usually still there — Claude is just choosing other tools. Strengthen the description, or use **hooks** to enforce deterministically.

After `/compact`, Claude Code re-attaches the most recent invocation of each skill, keeping the first 5,000 tokens. Re-attached skills share a combined budget of 25,000 tokens. Older invocations may be dropped.

---

## 9. Subagents in Claude Code — `.claude/agents/`

Custom subagents in Claude Code are defined as Markdown files with YAML frontmatter, stored in:

- **Managed (org-wide):** deployed via managed settings
- **CLI-defined:** passed as JSON via `--agents` flag
- **Project:** `.claude/agents/<name>.md` — committed, shared with team
- **User:** `~/.claude/agents/<name>.md` — personal only
- **Plugin:** `<plugin>/agents/` — installed with plugins

### Built-in Subagents (Official)

Claude Code ships with several built-in subagents that Claude auto-invokes when appropriate. Knowing their defaults helps the exam — questions test which built-in is used for what:

| Built-in | Model | Tools | When Claude Uses It |
| -------- | ----- | ----- | ------------------- |
| **`Explore`** | Haiku (fast, low-latency) | Read-only (denied Write/Edit) | File discovery, code search, codebase exploration without modifications |
| **`Plan`** | Inherits from main conversation | Read-only (denied Write/Edit) | Codebase research during plan mode (prevents infinite nesting) |
| **`general-purpose`** | Inherits from main conversation | All tools | Complex multi-step tasks requiring exploration AND modification |
| `statusline-setup` | Sonnet | Configured via `/statusline` | Setting up the status line |
| `claude-code-guide` | Haiku | n/a | Answering questions about Claude Code itself |

**Important nuance:** When Claude invokes `Explore`, it can specify a thoroughness level: **quick** (targeted lookups), **medium** (balanced), or **very thorough** (comprehensive analysis).

### Subagent Priority (Official)

```
Managed (1, highest) > --agents CLI flag (2) > Project (3) > User (4) > Plugin (5, lowest)
```

When names conflict, the higher-priority definition wins.

### Subagent File Format

```yaml
---
name: code-reviewer
description: Expert code reviewer specializing in security and best practices. Use PROACTIVELY after writing or modifying code.
tools: Read, Grep, Glob, Bash
model: sonnet
---

You are a senior code reviewer. When invoked:

1. Run `git diff` to see recent changes
2. Focus on modified files
3. Review for readability, error handling, security, and test coverage

Provide feedback organized by priority:
- Critical (must fix before merge)
- Warnings (should fix soon)
- Suggestions (nice to have)
```

### Complete Frontmatter Reference (Official)

Beyond `name` and `description` (the only required fields), subagents support extensive frontmatter:

| Field | Purpose |
| ----- | ------- |
| `name` (required) | Unique identifier; lowercase + hyphens. Hooks receive this as `agent_type`. |
| `description` (required) | When Claude should delegate. Use "use PROACTIVELY" / "MUST BE USED" for auto-invoke. |
| `tools` | Allowlist of tools (comma-separated string). Inherits all tools if omitted. |
| `disallowedTools` | Denylist applied first, then `tools` resolves against the remainder. |
| `model` | `sonnet` / `opus` / `haiku` / full model ID / `inherit` (default). |
| `permissionMode` | `default` / `acceptEdits` / `auto` / `dontAsk` / `bypassPermissions` / `plan` |
| `maxTurns` | Max agentic turns before subagent stops |
| `skills` | List of skills to preload at startup — full skill content injected |
| `mcpServers` | MCP servers available to this subagent (string ref or inline definition) |
| `hooks` | Lifecycle hooks scoped to this subagent's execution |
| `memory` | `user` / `project` / `local` — persistent agent memory directory |
| `background` | `true` to always run as a background task |
| `effort` | `low` / `medium` / `high` / `xhigh` / `max` — model effort level |
| `isolation` | `worktree` to run in a temporary git worktree (isolated repo copy) |
| `color` | UI display color (`red`/`blue`/`green`/`yellow`/`purple`/`orange`/`pink`/`cyan`) |
| `initialPrompt` | Auto-submitted first user turn (only when run as main session via `--agent`) |

### `Task` Tool Renamed to `Agent` (v2.1.63)

> ⭐ Exam-relevant terminology fact.

In Claude Code v2.1.63, the **`Task` tool was renamed to `Agent`**. Existing `Task(...)` references in settings and agent definitions still work as aliases. The exam may test either name — recognize both:

- New: `tools: Agent` or `tools: Agent(worker, researcher)` (restrict spawnable subagent types)
- Old (still works): `tools: Task`

### Forked Subagents (Experimental — `CLAUDE_CODE_FORK_SUBAGENT=1`)
> ⭐ **Two different things both called "fork":** skill-level `context: fork` (a forked subagent receives only the skill content, no parent conversation) is DIFFERENT from **forked subagents** enabled via `CLAUDE_CODE_FORK_SUBAGENT=1` (an entire conversation fork that inherits parent context).

Per official docs, **forked subagents** (experimental, requires Claude Code v2.1.117+) inherit the **entire conversation history** instead of starting fresh. Enable them by setting:

```bash
export CLAUDE_CODE_FORK_SUBAGENT=1
```

When enabled, three things change:

1. Claude spawns a **fork** whenever it would otherwise use the `general-purpose` subagent. Named subagents (Explore, Plan, custom) still spawn as before.
2. Every subagent spawn runs in the **background** by default.
3. The `/fork` command spawns a fork instead of being an alias for `/branch`.

### How Forks Differ From Named Subagents

| | Fork (env var enabled) | Named subagent | Skill with `context: fork` |
| -- | --------------------- | -------------- | -------------------------- |
| **Conversation history** | Full inherited | Fresh (none) | Fresh (none) |
| **System prompt** | Same as main session | From subagent definition | From skill body + selected agent type |
| **Tools** | Same as main session | From subagent definition | From skill `allowed-tools` + agent |
| **Model** | Same as main session | From `model` field | From agent type or skill `model` |
| **Prompt cache** | Shared with main session (cheaper) | Separate cache | Separate cache |
| **Can spawn further** | NO (cannot fork a fork) | NO subagents | NO subagents |

**Manual fork invocation:** `/fork <directive>` spawns a fork from the current conversation with the directive as its task. It runs in a background panel below the prompt input; results return as a message when complete.

### Subagent Discovery & Invocation

Claude **auto-invokes** subagents when the description matches the user's intent. To encourage auto-invocation, include phrases like **"Use PROACTIVELY"** or **"MUST BE USED"** in the description (official tip).

Users can also invoke explicitly:

- `@code-reviewer` — directly invoke by name
- "Use the code-reviewer subagent to check my recent changes" — natural language
- `claude --agent code-reviewer` from CLI

### Subagent Context — Fresh Conversation, NOT Empty
> ⭐ **Critical correction:** A common myth is that subagents "start with nothing." That's wrong.

Per official docs, a non-fork subagent's initial context contains:

| Loaded by default | Notes |
| ----------------- | ----- |
| **The subagent's own system prompt** | From the markdown body (or `prompt` field for `--agents` JSON). NOT the full Claude Code system prompt. |
| **The task message** | The delegation prompt Claude writes when handing off. |
| **CLAUDE.md and memory** | ALL levels of the memory hierarchy: `~/.claude/CLAUDE.md`, project rules, `CLAUDE.local.md`, managed policy. **EXCEPTION: built-in Explore and Plan agents skip this** to keep their context tiny. |
| **Git status snapshot** | Taken at parent session start. Skipped for Explore/Plan or when `includeGitInstructions: false`. |
| **Preloaded skills** | Full content of skills listed in the agent's `skills` frontmatter field. Built-in agents don't preload skills. |

**What is NOT inherited:**

- Your **conversation history** with the parent — subagents do not see prior messages
- The **files Claude has already read** in the parent session
- The **skills you've already invoked** in the parent
- **MCP servers** unless scoped via `mcpServers` field or string reference

**Critical constraint (per official docs):** *"Subagents cannot spawn other subagents."* If you need nested delegation, use Skills or chain subagents from the main conversation. This prevents infinite recursion.

> The exception is **forked subagents** (separate from skill-level `context: fork`) — see "Forked subagents" below. A fork inherits the FULL conversation history of the parent, including all messages and tool calls.

### Skills vs Subagents — Know the Difference
> ⭐ **Directly exam-tested.**

| Aspect           | Skill                                                       | Subagent                                                            |
| ---------------- | ----------------------------------------------------------- | ------------------------------------------------------------------- |
| **Context**      | Runs in the **main conversation** (unless `context: fork`)  | Runs in its **own isolated context window**                         |
| **Returns**      | Full inline output to the conversation                      | Only the final message back to the parent (verbatim or summarized)  |
| **Best for**     | Inline reusable workflows, prompt templates                 | Verbose/specialized tasks where intermediate work should stay out   |
| **Invocation**   | `/skill-name` or auto-invoke when description matches        | `@agent-name`, auto-invoke, or via `Task` tool from another agent   |
| **Tool scoping** | `allowed-tools:` in frontmatter                             | `tools:` in frontmatter                                             |
| **Definition**   | `.claude/skills/<name>/SKILL.md`                            | `.claude/agents/<name>.md`                                          |
| **Memory**       | Stateless                                                    | Can have persistent memory (user-scope)                            |

### When to Choose Which

```
Inline reusable workflow, lightweight output → Skill
Verbose intermediate work you want hidden from main context → Subagent
                                                              OR Skill with context: fork
Long-running specialized work with own tools → Subagent
Task with explicit instructions you want isolated → Skill with context: fork
Recurring specialist role (security reviewer, test writer) → Subagent
```

---

## 10. Hooks — Lifecycle Events & Configuration

Claude Code hooks are **shell commands or HTTP endpoints** invoked at specific lifecycle events. They're configured in `settings.json` (at any scope level) and let you observe, control, and extend Claude Code's agent loop deterministically.

### The Hook Events (Official Complete List)

The official docs list a much larger set of events than most third-party guides cover. The exam can test any of the bolded ones below; the rest are good to recognize.

| Event | When It Fires | Can Block? | Common Use |
| ----- | ------------- | ---------- | ---------- |
| **`SessionStart`** | Session begins or resumes (matchers: `startup`, `resume`, `clear`, `compact`) | No (exit 2 shows stderr; continues) | Initialize logging, **re-inject context after compaction** (matcher: `compact`) |
| `Setup` | `--init-only`, or `--init`/`--maintenance` in `-p` mode | No | One-time CI prep |
| **`UserPromptSubmit`** | User submits prompt, before Claude processes | ✅ Yes | Validate prompts, inject context, redact secrets |
| `UserPromptExpansion` | Skill/command expands into a prompt | ✅ Yes | Block expansion based on rules |
| **`PreToolUse`** | Before any tool call executes (matcher: tool name) | ✅ Yes (deny blocks the tool) | Block dangerous commands, validate inputs |
| **`PermissionRequest`** | When a permission dialog appears | ✅ Yes (interactive only) | Auto-approve known-safe operations |
| `PermissionDenied` | Tool call denied by auto-mode classifier | Return `{retry: true}` for retry | Adaptive permission feedback |
| **`PostToolUse`** | After a tool call **succeeds** | Can send feedback (cannot undo) | Auto-format, run tests, trim verbose output, log |
| **`PostToolUseFailure`** | After a tool call **fails** | Cannot undo | Log errors, trigger alerts |
| `PostToolBatch` | After a batch of parallel tool calls resolves | ✅ Yes | Validate batch result before next model call |
| **`Notification`** | Claude sends a notification | No | Desktop alerts, Slack notifications |
| `SubagentStart` | A subagent is spawned | No | Subagent telemetry |
| **`SubagentStop`** | Subagent finishes | Exit 2 forces continuation | Per-subagent cleanup or validation |
| `TaskCreated` / `TaskCompleted` | Task is being created or completed | No | Task-tracking telemetry |
| **`Stop`** | Claude finishes responding | Exit 2 forces Claude to keep working | Final checks, summaries, validation |
| `StopFailure` | Turn ended due to API error (matcher: `rate_limit`, `authentication_failed`, etc.) | Output ignored | Error reporting |
| `TeammateIdle` | Agent team teammate going idle | No | Team coordination |
| **`InstructionsLoaded`** | A CLAUDE.md or `.claude/rules/*.md` is loaded | No | **Debug "why isn't my rule applying?"** — official recommended use |
| `ConfigChange` | Configuration file changes during a session (matcher: `user_settings`, `project_settings`, `local_settings`, `policy_settings`, `skills`) | ✅ Yes | Audit log, block unauthorized edits |
| `CwdChanged` | Working directory changes (e.g., `cd`) | No | Reload env vars (direnv pattern) |
| `FileChanged` | A watched file changes on disk (matcher: filename list) | No | Reload env when `.envrc` changes |
| `WorktreeCreate` / `WorktreeRemove` | Worktree create/remove | ✅ Yes (replaces default git behavior) | Custom worktree setup |
| **`PreCompact`** | Before context compaction (matcher: `manual`, `auto`) | No | Save key facts externally before compression |
| **`PostCompact`** | After context compaction completes | No | Re-inject critical context after compact |
| `Elicitation` / `ElicitationResult` | MCP server requests/receives user input | No | MCP elicitation hooks |
| **`SessionEnd`** | Session terminates (matcher: `clear`, `resume`, `logout`, etc.) | No | True session-end cleanup |

### Hook Types (Beyond Just Shell Commands)

Hooks aren't limited to shell commands. The official `type` field supports four other handler types — important for exam-level questions:

| Type | What It Does | When to Use |
| ---- | ------------ | ----------- |
| **`command`** (default) | Runs a shell command | Most cases — deterministic rules, formatters, validators |
| `http` | POSTs event JSON to a URL; response controls outcome | Centralized audit service across a team; cloud function handlers |
| `mcp_tool` | Calls a tool on a connected MCP server | Hook delegates to a custom MCP capability |
| `prompt` | Single-turn LLM evaluation (Haiku by default) | Decisions needing **judgment** — "is the task really done?" |
| `agent` | Multi-turn subagent with tool access (experimental) | Verification that requires inspecting code (e.g., "did tests pass?") |

The `prompt` and `agent` types are how you do **non-deterministic** hooks — judgment calls instead of fixed rules. This is the natural counterpart to "use hooks for safety; use prompts for style" — when judgment IS the safety check, a prompt-based hook is the right tool.

### Configuration Example

```json
// .claude/settings.json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          { "type": "command", "command": "./scripts/audit-bash.sh" }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          { "type": "command", "command": "prettier --write \"$CLAUDE_FILE\"" }
        ]
      }
    ]
  }
}
```

### Critical Hook Behaviors (Exam-Tested)

- **`PreToolUse` fires BEFORE any permission-mode check.** A hook returning `permissionDecision: "deny"` blocks the tool **even in `bypassPermissions` mode or with `--dangerously-skip-permissions`.** This is the documented safety net for high-risk environments.
- **Hooks can tighten restrictions but NOT loosen them.** Per official docs: *"a hook returning 'allow' does not bypass deny rules from settings."* If a managed-settings deny rule matches, the call is blocked even when a hook returns `"allow"`.
- **Exit 0 is "no objection," not "approve."** Normal permission flow still applies.
- **Exit 2 blocks the action.** Stderr is shown to Claude as feedback so it can adjust. Don't mix exit 2 with JSON output — Claude Code ignores JSON when you exit 2.
- **JSON output is the alternative to exit codes for fine-grained control.** Write `permissionDecision: "allow" | "deny" | "ask"` (plus `"defer"` in `-p` mode) inside a `hookSpecificOutput` block.
- **`PostToolUse` cannot undo a tool execution** — the tool has already run. Use it for cleanup, formatting, or feedback to Claude on what to do next.
- **`PermissionRequest` hooks do NOT fire in non-interactive mode (`-p`).** Use `PreToolUse` for automated permission decisions in CI/CD.
- **`Stop` fires whenever Claude finishes responding**, not only at task completion. Don't put expensive cleanup in `Stop` if you'll re-enter the loop — use `SessionEnd`.
- **Stop hook block cap = 8.** Claude Code overrides a `Stop` hook after it blocks **8 times in a row** without progress. Your script must check the `stop_hook_active` JSON field and exit early when it's true, or you'll hit the cap. Raise it via `CLAUDE_CODE_STOP_HOOK_BLOCK_CAP` if you legitimately need more.
- **Multiple PreToolUse hooks rewriting the same tool input run in parallel — last to finish wins (non-deterministic).** Avoid having more than one hook modify the same tool's input.
- **PreToolUse permission outcomes combine "most restrictive wins":** `deny` overrides `ask` overrides `allow` across multiple hooks on the same event.
- **`InstructionsLoaded` hook is the documented way to debug "which rules loaded and why."** Use it to log which CLAUDE.md / `.claude/rules/*.md` files entered context, including path-glob-matched ones.
- **Re-injecting context after compaction is done via `SessionStart` with matcher `compact`** (not `PreCompact`). Anything your script writes to stdout is added to Claude's context.

### Hook Timeouts (Official)

| Hook type | Default timeout |
| --------- | --------------- |
| `command`, `http`, `mcp_tool` (most events) | 10 minutes |
| `command`, `http`, `mcp_tool` on `UserPromptSubmit` | 30 seconds |
| `prompt` | 30 seconds |
| `agent` | 60 seconds |

Override per-hook with a `timeout` field in seconds.

### Skill-Scoped Hooks

Skills can declare hooks in their own frontmatter that fire **only while the skill is active**. Same hook events as the global system:

```yaml
hooks:
  PreToolUse:
    - matcher: Bash
      hooks:
        - type: command
          command: ./scripts/audit-bash.sh
  PostToolUse:
    - matcher: Edit
      hooks:
        - type: command
          command: prettier --write "$CLAUDE_FILE"
```

When a `Stop` hook is defined in a **subagent's** frontmatter, it's automatically scoped to that subagent (converted to `SubagentStop`).

### Hooks vs Prompt Instructions — Same Rule as Domain 1

| Use Case | Hook (Deterministic) | Prompt Instruction (Probabilistic) |
| -------- | -------------------- | ----------------------------------- |
| Block `rm -rf` commands | ✅ PreToolUse with deny | ❌ "Never run dangerous commands" — will fail eventually |
| Auto-format code after edits | ✅ PostToolUse running Prettier | ❌ "Always format your code" — Claude may forget |
| Style suggestions | ❌ Overkill for soft guidance | ✅ Prompt instruction is fine |
| Tone preference | ❌ Hooks can't enforce tone | ✅ Prompt instruction is the right tool |

### Hooks Are the Answer to "How Do We Guarantee X?"

If an exam question says "always," "never," "must," "every time," or "guarantee" — the answer is a hook, not a prompt instruction. This rule carries directly from Domain 1.

---

## 11. Plan Mode — Read-Only Architectural Thinking
> ⭐ **Exam-tested specifically for the Code Generation scenario.**

Plan Mode is a **permission mode** where Claude analyzes and plans without making any modifications. No `Write`, no `Edit`, no `Bash` with side effects — just `Read`, `Grep`, `Glob`, and reasoning.

### What Plan Mode Allows vs Blocks

Per Claude Code's source-prompt design:

| ✅ Available in Plan Mode | ❌ Blocked in Plan Mode |
| ------------------------ | ----------------------- |
| `Read`, `Grep`, `Glob`, `LS` (read-only exploration) | `Edit`, `Write` (any file modification) |
| `Bash` — **read-only/safe commands only** (e.g., `cat`, `head`, `tail`, `ls`, `git status`, `git log`, `git diff`, `grep`, `find`, `jq`) | `Bash` with destructive/mutating commands (e.g., `rm`, `mv`, `git commit`, `npm install`) |
| `WebFetch`, `WebSearch` | Any tool with side effects |
| `Skill` (read-only skills) | Skills that would write files |
| `ExitPlanMode` (the tool that surfaces the plan for approval) | |

When Claude finalizes a plan, it calls **`ExitPlanMode`** — a dedicated tool that presents the plan and requests user approval. On approval, the previous permission mode is restored and full tools are available again.

### How to Activate

```bash
# Via CLI flag at startup
claude --permission-mode plan

# Via slash command inside the REPL
/plan

# Or cycle through permission modes with Shift+Tab
# (Shift+Tab twice = Plan Mode in the default cycle order)
```

### When to Use Plan Mode

| Use Plan Mode for                                                              | Don't use Plan Mode for                                  |
| ------------------------------------------------------------------------------ | -------------------------------------------------------- |
| Architectural design before committing to an approach                          | Quick edits — overhead is wasted                         |
| Comparing two implementation strategies side-by-side                            | Bug fixes you already understand                         |
| Onboarding to an unfamiliar codebase                                            | Tasks where you'll definitely write code immediately     |
| Risky changes where you want a full read-only analysis first                    | Exploratory coding where iteration is expected            |
| Generating a structured plan to hand off to a different session for execution   | Greenfield code with no existing context to analyze       |

### Why It Beats "Just Tell Claude Not to Edit"

```
❌ Prompt instruction: "Don't make any edits, just analyze."
   → Probabilistic — Claude may edit anyway, especially on long tasks

✅ Plan Mode: tools that would edit are NOT available
   → Deterministic — Claude literally cannot edit even if asked
```

### The Plan → Execute Handoff Pattern

```
Session 1: claude --permission-mode plan
  → Claude analyzes auth subsystem, produces a structured plan
  → Plan is saved to a file or copied for handoff

Session 2: claude (normal permission mode, possibly new session)
  → Inject the plan as the first message:
    "Implement the following plan: <paste plan>"
  → Now Claude can write/edit/run — but it's executing a reviewed plan,
    not improvising
```

This is the **independent review pattern** applied to your own work — separating "thinking" from "doing" prevents Claude from rationalizing decisions made under execution pressure.

---

## 12. Permission Modes & `settings.json` Scopes

### The Permission Modes (Official — 6 Modes)

> ⭐ The most recent addition is **`auto` mode** (released March 24, 2026) — a classifier-driven middle ground between `default` and `bypassPermissions`. Likely to appear on the exam.

| Mode | Behavior | When to Use |
| ---- | -------- | ----------- |
| **`default`** | Standard permission checking — asks before each new tool/file | Daily interactive use |
| **`plan`** | Read-only; cannot Edit/Write/run side-effect commands | Architectural analysis, code review |
| **`acceptEdits`** | Auto-approves Edit and Write operations for paths in working dir / `additionalDirectories` | Trusted automation, focused refactor sessions |
| **`auto`** | A background **classifier reviews each tool call** in real time. Safe actions proceed; risky ones (mass deletions, exfil attempts) are blocked or escalated | Long autonomous sessions where you want guardrails without bypass |
| **`dontAsk`** | Auto-DENIES all permission prompts — only pre-allowed tools execute | CI/CD pipelines and locked-down environments with explicit allowlists |
| **`bypassPermissions`** | Skip all permission prompts (still respects `PreToolUse` deny hooks AND safety-sensitive path checks like `.git/`, `.claude/`, `.vscode/`) | Isolated CI containers with hardened hooks |

**Critical nuance from source-code analysis:** Even `bypassPermissions` cannot skip safety-sensitive path checks. Writes to `.git/`, `.claude/`, `.vscode/`, `.idea/`, `.husky/`, and root/home-directory removals (`rm -rf /`) **still prompt** as a circuit breaker. This is "bypass-immune."

**Shift+Tab cycle order:** `default → acceptEdits → plan → auto` (in default config). The official docs note `dontAsk` and `bypassPermissions` are not in the default Shift+Tab cycle — they're set via CLI flag or settings.

Set a default via `--permission-mode` or `permissions.defaultMode` in settings.

### `--dangerously-skip-permissions` (the `-y` cousin)

For fully unattended CI/CD runs in isolated containers:

```bash
claude -p "review and commit lint fixes" --dangerously-skip-permissions
```

> **Critical safety note:** `PreToolUse` hooks STILL fire and CAN STILL deny actions even when `--dangerously-skip-permissions` is set. This is the documented safety mechanism. Use this combination — skip-permissions + hardened `PreToolUse` hooks — for hands-off CI/CD. **Never use `--dangerously-skip-permissions` on a personal machine without protective hooks.**

### `settings.json` — Scope Precedence

| File | Scope | Committed? |
| ---- | ----- | ---------- |
| Managed settings (server/MDM) | Org-wide, cannot be overridden | N/A |
| `.claude/settings.json` | Project — shared with team | ✅ Yes |
| `.claude/settings.local.json` | Project — your machine only | ❌ No (gitignored by default) |
| `~/.claude/settings.json` | User — all your projects on this machine | ❌ No |

**Precedence (highest to lowest):** Managed > Project > Project Local > User. **Permission rules MERGE across scopes** (additive); other settings OVERRIDE.

### What Goes Where (Common Confusion)

```
.claude/settings.json (committed)
  → Team-wide permission allow/deny rules
  → Team-wide hooks (e.g., auto-format on save)
  → Team's MCP server references (or use .mcp.json)
  → Default permission mode for the project

.claude/settings.local.json (NOT committed, gitignored)
  → Personal allowlist additions
  → Local debugging hooks
  → Machine-specific overrides
  → claudeMdExcludes for monorepo noise reduction

~/.claude/settings.json (personal, all projects)
  → Personal preferences (status line, spinner tips)
  → Personal hooks that apply everywhere
```

---

## 13. Headless Mode (`-p`) — CI/CD Integration
> ⭐ **The single most distinctive Domain 3 exam pattern.** Every CCA-F scenario about CI/CD anchors here.

The `-p` (or `--print`) flag runs Claude Code **non-interactively**: process the prompt, print to stdout, exit. No interactive UI, no waiting for user input. **This is the foundation for every CI/CD integration.**

### The Hanging-Pipeline Exam Question
> The most famous Domain 3 exam question

A GitHub Actions job hangs forever when calling Claude Code. The four answer options:

```
A) Set CLAUDE_HEADLESS=true       ← Not a real environment variable. WRONG.
B) Redirect /dev/null to stdin    ← A hack; not the intended fix. WRONG.
C) Use the --batch flag           ← --batch is not a valid Claude Code flag. WRONG.
D) Add the -p flag to the command ← CORRECT.
```

**Why:** Without `-p`, Claude Code launches its interactive REPL and waits for input that never comes in a CI runner. `-p` tells Claude Code "this is a one-shot prompt; print the answer and exit."

> **Bonus exam fact:** Per the Claude Code CLI help text — *"The workspace trust dialog is skipped when Claude is run with the `-p` mode. Only use this flag in directories you trust."*

### The Core Pattern

```bash
claude -p "Review this PR for security issues" \
  --output-format json \
  --max-turns 5

echo "Exit code: $?"   # 0 = success, non-zero = error
```

### Required Components for CI/CD

```bash
# 1. Set ANTHROPIC_API_KEY as a CI secret
export ANTHROPIC_API_KEY="${{ secrets.ANTHROPIC_API_KEY }}"

# 2. Use -p (or --print) — non-interactive
claude -p "your prompt"

# 3. Choose an output format that scripts can parse
--output-format text          # Plain text (default; not parser-friendly)
--output-format json          # Single structured result with metadata
--output-format stream-json   # Streaming JSON events (each message as separate JSON)

# 4. (Optional) Specify input format
--input-format text           # Default
--input-format stream-json    # Bidirectional streaming via stdin (Agent SDK pattern)

# 5. Cap iterations to control cost
--max-turns 5

# 6. (Optional) Append to the system prompt — only valid with --print
--append-system-prompt "Focus on security issues; ignore style nits"

# 7. (Optional) Bypass interactive permission prompts in isolated containers
--dangerously-skip-permissions   # Only with hardened PreToolUse hooks

# 8. (Optional) Provide an MCP tool to handle permission prompts non-interactively
--permission-prompt-tool mcp__auth__prompt
```

### Parsing the JSON Output (Official Schema)

```bash
RESULT=$(claude -p "review the diff" --output-format json)

# Official JSON response shape:
# {
#   "type": "result",
#   "subtype": "success",
#   "result": "response text here...",
#   "session_id": "abc123",
#   "total_cost_usd": 0.003,
#   "is_error": false,
#   "duration_ms": 1234,
#   "duration_api_ms": 800,
#   "num_turns": 6
# }

# Check is_error first
if [[ $(echo "$RESULT" | jq -r '.is_error') == "true" ]]; then
  echo "Claude error: $(echo "$RESULT" | jq -r '.result')" >&2
  exit 1
fi

# Extract the response
echo "$RESULT" | jq -r '.result'

# Monitor cost
COST=$(echo "$RESULT" | jq -r '.total_cost_usd')
echo "Cost: \$$COST"

# Capture session_id to chain follow-up turns
SESSION=$(echo "$RESULT" | jq -r '.session_id')
claude --resume "$SESSION" -p "Now fix the issues you identified"
```

> **Field-name exam note:** The JSON output uses `result` (not `content`), `total_cost_usd` (not `cost_usd`), `is_error` (boolean, not an `.error` string field), and includes `session_id` for chaining follow-up turns. Memorize these — they're testable specifics.

### Piping Input via stdin

```bash
# Pipe a git diff for automatic review
git diff HEAD~1 | claude -p "Review this diff for bugs and security issues"

# Pipe a file
cat src/utils.ts | claude -p "Add TypeScript types to this file"
```

### Common CI/CD Workflows

| Workflow | Pattern |
| -------- | ------- |
| **PR auto-review** | `git diff origin/main...HEAD \| claude -p "Review for bugs" --output-format json` |
| **Auto-generate release notes** | `git log --oneline last-tag..HEAD \| claude -p "Generate release notes"` |
| **Issue triage** | `claude -p "Classify this issue: <body>" --output-format json` → parse labels |
| **Nightly tech-debt scan** | `claude -p "Scan for TODO/FIXME with priority"` → run as scheduled GitHub Action |
| **Pre-merge lint** | `claude -p "Check this PR meets our standards" --max-turns 3` |

### Resume and Continue in Headless Mode

```bash
# Continue most recent conversation
claude -c -p "Now refactor for performance"

# Resume specific session by ID
claude --resume <session-id> -p "Fix the failing tests"
```

### What to Keep OUT of Headless Mode

| Don't use headless for                                                | Why                                                            |
| --------------------------------------------------------------------- | -------------------------------------------------------------- |
| Interactive debugging — open the REPL instead                          | `-p` exits after one prompt; no follow-up turns                |
| Multi-turn deliberation that requires user input                       | No interactive permission prompts; auto-deny unless hooked     |
| Operations the Anthropic Batch API would handle better                 | Batch API is 50% cheaper for non-time-sensitive bulk work      |

### Headless `-p` vs Batch API — Exam Trap
> ⭐ **Directly tested in the Domain 3 + Domain 4 crossover.**

The exam describes a manager who wants to move both a **blocking pre-merge check** AND a **nightly tech-debt report** to the Anthropic Batch API. What's wrong?

```
✅ Nightly tech-debt report → Batch API (50% cheaper, no SLA needed)
❌ Blocking pre-merge check → Keep on synchronous headless -p
   (Batch API has up to a 24-hour latency window; developers
    waiting on a PR cannot tolerate that)
```

**The rule:** Headless `-p` for **synchronous** CI/CD that humans wait on. Batch API for **asynchronous** scheduled work where latency doesn't matter.

---

## 14. MCP in Claude Code — `.mcp.json` Integration

### The Three Configuration Files (Quick Recap from Domain 2)

| File | Scope | Committed? |
| ---- | ----- | ---------- |
| `.mcp.json` (project root) | Team — shared MCP servers | ✅ Yes |
| `~/.claude.json` | Personal MCP servers | ❌ No |
| `.claude/settings.local.json` | Local overrides | ❌ No |

### The Domain 3-Specific Exam Pattern

```
New developer clones the repo. Claude Code doesn't have access to the team's Jira MCP server.
Possible causes (in order of likelihood):

1. The MCP server is in someone's ~/.claude.json instead of project .mcp.json
   → Fix: Move to .mcp.json, commit to git

2. The .mcp.json references an environment variable not set on the new machine
   → Fix: Document required env vars in CLAUDE.md or README

3. The new developer's Claude Code version is too old to support MCP
   → Fix: Document minimum version in CLAUDE.md, run claude --version
```

### Secrets Pattern (Domain 2 Crossover)

```json
// .mcp.json — committed to git, NO secrets
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
```

The actual `JIRA_API_TOKEN` value is set in each developer's shell environment (or a secret manager) — never in the file.

### Headless MCP Configuration

In CI/CD, MCP servers can be loaded via `--mcp-config`:

```bash
claude -p "create a Jira ticket for this bug" \
  --mcp-config /etc/ci/mcp-servers.json \
  --allowedTools "mcp__jira" \
  --output-format json
```

### MCP-Plus-Skill Pattern (Cross-Domain)

A common production pattern: a **skill** that calls an **MCP server**:

```yaml
---
name: triage-bug
description: Triage a bug report — create Jira ticket, post to Slack
allowed-tools: mcp__jira__create_issue mcp__slack__post_message
---

# Triage Workflow

1. Read the bug report
2. Use `mcp__jira__create_issue` to create a JIRA ticket with severity label
3. Use `mcp__slack__post_message` to notify #engineering with ticket link
4. Return: { ticket_id, slack_message_ts }
```

Tools from MCP servers appear as `mcp__<server>__<tool-name>` and can be whitelisted in the skill's `allowed-tools`.

---

## 15. Session Management & Compaction

Domain 3 reuses Domain 1's session model — resume, continue, and new-session-with-summary — but in the Claude Code context.

### CLI Commands

```bash
# Start a new interactive session
claude

# Continue the most recent conversation
claude -c
claude --continue

# Resume a specific session by ID
claude -r <session-id> "your next prompt"
claude --resume <session-id>

# In headless mode: continue or resume with -p
claude -c -p "now refactor for performance"
claude --resume <session-id> -p "fix the failing tests"
```

### The "Resume After File Changes" Trap
> Same as Domain 1, but tested in Claude Code context

If files have been modified since the last session (e.g., by a different developer's commit), and you just `--resume`, Claude will reason from its **stale analysis** of those files — silently producing wrong results.

```bash
# ❌ WRONG — just resume, hope for the best
claude --resume <id>

# ✅ CORRECT — resume + explicit notice
claude --resume <id>
> Files src/auth.ts and src/jwt-service.ts have been refactored since
> our last session. Please re-read them before proceeding.
```

### Compaction — `/compact` and What Survives

When context fills up, run `/compact` to summarize the conversation and free space. Per official docs:

- **Project-root CLAUDE.md survives compaction** — Claude re-reads it from disk and re-injects it
- **Nested subdirectory CLAUDE.md files are NOT re-injected automatically** — they reload only when Claude next reads a file there
- **Skills**: the most recent invocation of each skill is re-attached, first 5,000 tokens each, combined 25,000-token budget; older invocations may drop
- **Conversation-only instructions** ("for this session, always use X") are LOST during compaction — promote them to CLAUDE.md if they should persist

If an instruction "disappears" after `/compact`, it was either conversation-only or lives in a nested file that hasn't reloaded yet.

### When to Start Fresh vs Resume

```
Resume when:
  - Same task, prior context still valid
  - Files unchanged or you noted the changes
  - Continuing where you left off naturally

Start fresh when:
  - Fundamental assumption changed
  - More than ~50% of analyzed files were modified
  - Task domain entirely shifted (e.g., research → implementation)
  - You're transitioning from Plan Mode to execution
```

When starting fresh after a phase transition, inject a **structured summary** of decisions made (Domain 1 pattern):

```markdown
## Decisions made in the research phase
- Using JWT RS256 (not HS256) for backward compatibility
- Adapter pattern wraps existing JWT service during migration
- Token lifetime: 15min access / 7-day refresh

## Starting point for implementation
- Begin: src/auth/jwt-adapter.ts
- Do NOT modify: src/auth/legacy-jwt.ts (other services depend on it)
```

### `/clear` vs `/compact` vs New Session

| Action | What it does |
| ------ | ------------ |
| `/clear` | Wipes conversation history; **keeps CLAUDE.md and system prompt** |
| `/compact` | Summarizes conversation; re-injects project CLAUDE.md and most recent skills |
| New session (`claude` again) | Fresh start; reloads everything from disk |

---

## 16. Independent Review Pattern — Generation vs Review Sessions
> ⭐ **Directly exam-tested.**

The same Claude session that generated code will **rationalize its own decisions** during review. This is not Claude being adversarial — it's a structural problem: the generation context contains all the reasoning chains that justify the original choice. Review under that anchor is biased.

### The Pattern

```
Session A: Generate code
  → Claude writes the JWT adapter
  → Context contains: design choices, tradeoffs considered, why X over Y

Session B: Review the code — NEW session, no generation context
  → Inject only the diff/files
  → Claude reviews from fresh eyes, with no commitment to prior decisions
  → Catches issues Session A would rationalize away
```

### Exam Pattern

A scenario describes a team using a single Claude session to both write code and review it. Review quality is poor — Claude misses bugs in its own work. **The fix is an independent review session with no generation context.** Possible distractor answers:

```
❌ "Use a stronger model for review"
❌ "Add a system prompt saying 'be critical of your own code'"
❌ "Run /clear between generation and review" (clears history but keeps CLAUDE.md and system prompt; doesn't fully reset)
✅ "Start a new session for review with only the diff loaded"
```

### Why This Beats Self-Review Prompts

Self-review prompts are probabilistic. A fresh session is **structural** — Claude literally doesn't have the generation context to rationalize from. This is the same deterministic > probabilistic meta-pattern from Domains 1 and 2.

---

## 17. Anti-Patterns Master List

| Anti-Pattern | Why It's Wrong | Correct Approach |
| ------------ | -------------- | ---------------- |
| Team conventions in `~/.claude/CLAUDE.md` | Not version-controlled — new teammates don't get them | Move to `./CLAUDE.md` or `.claude/CLAUDE.md`, commit to git |
| 200+ line monolithic CLAUDE.md | Bloats every session's context with rules that don't apply | Split into lean CLAUDE.md + `.claude/rules/<topic>.md` with `paths:` |
| Treating CLAUDE.md scopes as strict overrides | Files are concatenated; arbitrary pick on conflict | Avoid contradictions across layers; project beats user in practice but no hard guarantee |
| Secrets committed in `.mcp.json` | Security breach — credentials in git | Use `${ENV_VAR}` substitution; values from shell env |
| Forgetting `-p` in CI/CD | Pipeline hangs waiting for interactive input | Always use `claude -p "prompt"` in scripts |
| Using `--batch` flag | Not a valid Claude Code flag — exam distractor | Use `-p` for headless; Batch API is a separate API tier |
| Treating headless output as plain text | Errors not surfaced; can't parse cost or partial results | Use `--output-format json` and parse `.is_error`, `.result`, `.total_cost_usd` |
| Looking for `.content` or `.error` in JSON output | Wrong field names; will be empty | Correct fields: `result`, `is_error`, `total_cost_usd`, `num_turns`, `session_id` |
| Prompt instruction for "never run dangerous commands" | Probabilistic — will fail eventually | `PreToolUse` hook that exits 2 (deny) — works even with `--dangerously-skip-permissions` |
| Same session for code generation AND review | Generation context rationalizes decisions under review | Independent review session with only the diff loaded |
| `--resume` after file changes without notification | Claude reasons from stale analysis — silent wrong answers | Resume + explicitly note which files changed |
| `disable-model-invocation: false` on `/deploy` skill | Claude can auto-deploy when user says "let's ship" | Set `disable-model-invocation: true` for high-risk operations |
| `context: fork` on a reference-only skill | Subagent has guidelines but no task — returns nothing | Only use `context: fork` for skills with an actionable task |
| `agent: Explore` on a skill that needs to write | Silent failure — Explore is read-only | Match agent type to skill's actual capability needs |
| Putting deployment guidance in main CLAUDE.md | Loads on every session even for trivial edits | Move to `.claude/rules/deployment.md` with `paths: ["deploy/**"]` |
| Mixing personal preferences in project CLAUDE.md | Confuses teammates; pollutes shared context | Personal → `~/.claude/CLAUDE.md` or `CLAUDE.local.md` |
| Using `Stop` hook for one-time cleanup | Stop fires every time Claude finishes responding | Use `SessionEnd` for actual session-end cleanup |
| Relying on `PermissionRequest` hook in `-p` mode | These hooks don't fire in headless mode | Use `PreToolUse` for automated decisions in CI/CD |
| Multiple PreToolUse hooks modifying the same tool input | Last to finish wins (non-deterministic order) | One hook per tool input; chain logic inside that hook |
| Verbose codebase scans in main session | Floods main context — accelerates context rot | `context: fork` with `agent: Explore` to keep verbose output isolated |
| Setting `--dangerously-skip-permissions` on a personal machine | No safety net against destructive operations | Reserve for CI containers; combine with strict `PreToolUse` hooks |
| Hardcoding `JIRA_TOKEN: "abc123"` in `.mcp.json` | Token leaks on first git push | Always `${JIRA_TOKEN}` substitution |
| Custom slash command for high-risk operations | Auto-invocable by Claude when description matches user intent | Use a skill with `disable-model-invocation: true` |
| Plan Mode as a prompt instruction ("just plan, don't edit") | Probabilistic — Claude may edit anyway | `claude --permission-mode plan` — Edit/Write literally unavailable |
| Restarting Claude Code after CLAUDE.md edit | Unnecessary — Claude reads CLAUDE.md on next session/turn cycle | Just continue; or open `/memory` to verify |
| Moving a blocking pre-merge check to the Batch API | Batch has up to 24-hour latency — developers wait | Synchronous `-p` for blocking checks; Batch for overnight reports |
| Putting MCP servers in `~/.claude.json` for team use | Personal scope — not shared with teammates | Project MCP servers → `.mcp.json` committed to repo |
| Running `/memory` to "load" CLAUDE.md | `/memory` shows what's already loaded — doesn't force a load | Edit CLAUDE.md and submit a prompt; or `/memory` to verify |
| Using one massive subagent for everything | Defeats the purpose of context isolation | Define narrow, role-specific subagents in `.claude/agents/` |
| Forgetting that nested CLAUDE.md doesn't re-inject after `/compact` | Subdirectory rules silently vanish from context | Re-trigger by reading a file in that subdir; or promote to project CLAUDE.md |
| Using `PreCompact` hook to inject context | Fires BEFORE compaction, so injection is lost in the summary | Use `SessionStart` hook with matcher `compact` to inject AFTER compaction |
| Hook returning `"allow"` to bypass managed-settings deny rule | Doesn't work — hooks can tighten, not loosen | Use `PreToolUse` deny to add restrictions; never expect "allow" to override deny |
| Stop hook re-blocking without checking `stop_hook_active` | Hits the 8-block cap; Claude is forced to end the turn | Parse `stop_hook_active` from JSON input; exit 0 when it's true |
| Mixing exit 2 with JSON output | Claude Code ignores JSON when exit 2 — silent failure | Choose ONE: exit 2 for simple block, or exit 0 with JSON for structured control |
| Multiple PreToolUse hooks rewriting the same tool input | Last to finish wins (non-deterministic order) | One hook per tool input; chain logic inside that hook |
| Putting personal preferences in `CLAUDE.local.md` for multi-worktree work | Only exists in the worktree where created — doesn't propagate | Use `@~/.claude/my-project-instructions.md` import (the new recommended pattern) |
| Using `prompt`/`agent` type hooks for deterministic policy | LLM-based — probabilistic | Use `command` type hooks for deterministic rules; reserve `prompt`/`agent` for judgment |
| Setting `--dangerously-skip-permissions` and expecting it to unblock everything | `PreToolUse` deny hooks STILL block; settings deny rules STILL apply | Combine with hardened hooks; understand it bypasses INTERACTIVE prompts only |
| Looking for `Edit`/`Write` in Plan Mode | Structurally unavailable | Use read-only tools (Read/Grep/Glob/safe Bash); call `ExitPlanMode` to surface plan for approval |

---

## 18. Key Rules to Memorize

```
1.  CLAUDE.md scopes: Managed (enterprise) | User (~/.claude) | Project (./CLAUDE.md) | Local (./CLAUDE.local.md)
2.  Files are CONCATENATED, not overridden — more-specific files are read LAST
3.  Project-level CLAUDE.md = ./CLAUDE.md or .claude/CLAUDE.md, committed to git
4.  User-level CLAUDE.md = ~/.claude/CLAUDE.md, NOT committed
5.  Local-level CLAUDE.md = ./CLAUDE.local.md, gitignored, personal project preferences
6.  Subdirectory CLAUDE.md loads ON-DEMAND when Claude touches files there (not at launch)
7.  Team convention not propagating to new dev = it's in someone's user-level file
8.  Target under 200 lines per CLAUDE.md — longer files reduce adherence
9.  /memory shows currently loaded files; /init generates a starter CLAUDE.md
10. .claude/rules/<topic>.md with YAML frontmatter paths: ["..."] loads conditionally
11. Rules WITHOUT paths: load at launch with same priority as .claude/CLAUDE.md
12. claudeMdExcludes in settings.local.json skips noisy ancestor CLAUDE.md files
13. Managed-policy CLAUDE.md cannot be excluded — org rules always apply
14. @import syntax composes files into CLAUDE.md; max depth 5; relative to importing file
15. Auto memory = Claude's notebook in ~/.claude/projects/<proj>/memory/MEMORY.md (first 200 lines / 25KB loaded)
16. AGENTS.md is read by other tools; bridge with @AGENTS.md import in CLAUDE.md or symlink
17. Custom slash commands live in .claude/commands/<name>.md → /name
18. Skills supersede commands — same /name invocation, with more capabilities
19. Skills auto-invoke when description matches user intent — unless disable-model-invocation: true
20. disable-model-invocation: true for high-risk ops (deploy/delete/migrate/commit)
21. user-invocable: false hides skill from /menu but Claude can still auto-invoke it
22. Skill content STAYS in context across turns once loaded — every line is a recurring token cost
23. Skill priority on name collision: enterprise > personal > project; plugins use namespaced names
24. context: fork in skill = runs in isolated subagent, summary back to main
25. agent: Explore (read-only) | Plan (no writes) | general-purpose | or any custom subagent
26. Explore and Plan agents SKIP CLAUDE.md and git status to keep context tiny
27. Misconfigured agent type silently fails — match to actual capability
28. context: fork only makes sense for skills with EXPLICIT TASKS — not reference material
29. Skills run in main context (unless fork); subagents run in their own context
30. Subagents in .claude/agents/<name>.md — auto-invoke or @name or "use X subagent"
31. Subagent priority: Managed > CLI(--agents) > Project > User > Plugin
32. Use phrases like "use PROACTIVELY" or "MUST BE USED" in subagent descriptions
33. Subagents start with FRESH CONVERSATION but DO load CLAUDE.md by default (except built-in Explore/Plan which skip it)
33a. Subagents CANNOT spawn other subagents — use Skills or chain from main conversation (prevents infinite recursion)
33b. Built-in subagents: Explore (Haiku, read-only) | Plan (inherit model, read-only) | general-purpose (all tools, inherit model) | statusline-setup (Sonnet) | claude-code-guide (Haiku)
33c. "Task" tool was renamed to "Agent" in v2.1.63 — Task(...) still works as alias
33d. Forked subagents (CLAUDE_CODE_FORK_SUBAGENT=1) inherit FULL parent conversation — DIFFERENT from skill-level context: fork
34. Hooks configured in settings.json — events: PreToolUse, PostToolUse, UserPromptSubmit, Stop, SessionStart, SessionEnd, PreCompact, Notification, SubagentStop, PostToolUseFailure
35. PreToolUse fires BEFORE permission check — deny overrides --dangerously-skip-permissions
36. PostToolUse cannot undo a tool — use for format/log/feedback
37. Stop fires every response end — not just task complete (use SessionEnd for real cleanup)
38. PermissionRequest hooks DON'T fire in -p mode — use PreToolUse for CI/CD
39. Skill-scoped hooks fire only while skill is active
40. Multiple PreToolUse hooks that rewrite tool input run in parallel — last to finish wins (avoid)
41. Plan Mode = --permission-mode plan OR /plan — Edit/Write/destructive Bash UNAVAILABLE
42. Plan → Execute handoff: plan in session A → execute in fresh session B with plan as input
43. Permission modes (6 total): default | plan | acceptEdits | auto (classifier) | dontAsk (auto-deny) | bypassPermissions
43a. Auto mode (released Mar 24 2026) = classifier reviews each tool call — middle ground between default and bypass
43b. Shift+Tab cycle order: default → acceptEdits → plan → auto (dontAsk and bypassPermissions need CLI flag/settings)
43c. Even bypassPermissions can't skip safety-sensitive paths: .git/, .claude/, .vscode/, .idea/, .husky/, rm -rf / — these still prompt
44. --dangerously-skip-permissions still respects PreToolUse deny hooks — combine for safe CI/CD
45. Headless mode = claude -p "prompt" — non-interactive, exits after response
46. Missing -p = pipeline hangs (#1 Domain 3 exam answer)
47. CLAUDE_HEADLESS=true is NOT real — distractor
48. --batch is NOT a real flag — distractor
49. --print mode SKIPS the workspace trust dialog — only use in trusted directories
50. Required for CI: ANTHROPIC_API_KEY as secret + -p + --output-format json
51. JSON output fields: result (response text), is_error (bool), total_cost_usd, num_turns, session_id
52. --max-turns N caps iteration cost in headless mode
53. Pipe via stdin: git diff | claude -p "review"
54. --continue (-c) resumes most recent | --resume <id> resumes specific session
55. --mcp-config <file> loads MCP servers in headless mode
56. Synchronous -p for blocking checks | Batch API for async overnight work
57. Moving a blocking pre-merge check to Batch API is WRONG — Batch has up to 24hr latency
58. .mcp.json (project, committed) for team MCP servers; ~/.claude.json (personal)
59. MCP secrets: ${ENV_VAR} substitution always — never hardcoded
60. Tools from MCP servers appear as mcp__<server>__<tool> in allowed-tools lists
61. settings.json scopes: Managed > .claude/settings.json > .claude/settings.local.json > ~/.claude/settings.json
62. Permission rules MERGE across scopes; other settings OVERRIDE
63. settings.local.json is gitignored by default — personal overrides
64. --resume after file changes: ALWAYS explicitly tell Claude what changed
65. Start fresh + structured summary when fundamental assumption changed
66. Independent review session > self-review prompt — structural, not probabilistic
67. Generation context biases review — separate sessions for generate and review
68. /clear = clears conversation but KEEPS CLAUDE.md and system prompt
69. /compact = summarizes conversation; re-injects project CLAUDE.md but NOT nested ones
70. Conversation-only instructions are LOST after /compact — promote to CLAUDE.md to persist
71. Live change detection: edits to existing skills/rules dirs apply in current session — new top-level dirs need restart
72. !`command` syntax in skills/commands runs shell BEFORE Claude sees content — preprocessing, not Claude execution
73. The meta-pattern: every Domain 3 wrong answer substitutes a prompt instruction for a structural fix
74. Configuration is layered onboarding for an engineer with perfect recall but total amnesia
75. CLAUDE.local.md is DEPRECATED in favor of @imports — use @~/.claude/my-project-instructions.md pattern instead
76. Hook types: command (default) | http | mcp_tool | prompt (LLM judgment) | agent (LLM + tools, experimental)
77. Hook timeouts: command/http/mcp_tool = 10 min | UserPromptSubmit = 30s | prompt = 30s | agent = 60s
78. Hooks CAN tighten restrictions, CANNOT loosen them — "allow" does not bypass settings deny rules
79. Re-injecting context after compaction uses SessionStart hook with matcher: "compact" — not PreCompact
80. InstructionsLoaded hook is the documented way to debug "which CLAUDE.md/rules files loaded and why"
81. Stop hook block cap = 8 consecutive blocks; check stop_hook_active field to avoid hitting the cap
82. ConfigChange hook (matcher: user_settings/project_settings/local_settings/policy_settings/skills) audits config changes
83. CwdChanged + FileChanged hooks pair with CLAUDE_ENV_FILE for direnv-style auto-env-reload
84. permissionDecision values: "allow" | "deny" | "ask" | "defer" (defer is -p mode only)
85. Don't mix exit 2 with JSON output — Claude Code ignores JSON when you exit 2
86. PreToolUse outcome combine: most restrictive wins (deny > ask > allow across multiple hooks)
87. Plan Mode allows Bash for SAFE/read-only commands (cat, head, ls, git status/log/diff) — blocks destructive (rm, mv, commit)
88. ExitPlanMode is a dedicated tool that surfaces the plan for user approval — exits plan mode on approval
89. Shift+Tab cycles permission modes; Shift+Tab twice = Plan Mode in default cycle
90. --append-system-prompt "..." (only with --print) appends to the system prompt in headless mode
91. --input-format stream-json enables bidirectional NDJSON streaming via stdin (Agent SDK pattern)
92. --permission-prompt-tool mcp__auth__prompt routes permission decisions to an MCP tool in headless
```

---

## 19. Practice Questions (25 MCQs)

---

**Q1.** A new developer joins the team, clones the repo, and runs Claude Code. Despite the team having well-established coding conventions, Claude does not follow them in the new developer's sessions. What is the most likely root cause?

- A) The new developer needs to upgrade to the latest Claude Code version
- B) The conventions were written in someone's `~/.claude/CLAUDE.md` instead of the project's `./CLAUDE.md`, so they were never committed to git
- C) Anthropic's servers have not synced the team settings to the new machine
- D) The new developer needs to run `/memory --reload` to fetch the project conventions

---

**Q2.** A team's project CLAUDE.md has grown to over 400 lines. It contains coding standards, testing conventions, PR review checklists, deployment guidance, and database migration rules. Context fills up quickly even on simple edits. What is the most effective restructuring?

- A) Trim CLAUDE.md to under 100 lines by removing the least important sections
- B) Move everything to user-level CLAUDE.md so it doesn't load on every project session
- C) Keep coding standards and testing conventions in CLAUDE.md (always relevant); move PR review, deployment, and migration guidance to `.claude/rules/<topic>.md` files with `paths:` globs for conditional loading
- D) Split into one large file per developer's machine

---

**Q3.** A CI/CD pipeline runs `claude "review the latest PR for security issues"` in a GitHub Actions job. The job hangs indefinitely. What is the correct fix?

- A) Set the environment variable `CLAUDE_HEADLESS=true` before the command
- B) Redirect `/dev/null` to stdin so Claude doesn't wait for input
- C) Add the `--batch` flag to enable non-interactive mode
- D) Add the `-p` (or `--print`) flag — `claude -p "review the latest PR..."` — to run in headless mode

---

**Q4.** A team has a `/deploy` skill that runs production deployment scripts. Currently it has no `disable-model-invocation` field. What risk does this create?

- A) The skill cannot be invoked by users at all
- B) Claude may auto-invoke the skill if a user's prompt matches the skill description (e.g., "let's get this ready to ship"), triggering a production deploy without explicit user action
- C) The skill will fail when invoked because the field is required
- D) The skill will only run if the user has admin permissions

---

**Q5.** A skill named `dependency-audit` runs a 1,000-line dependency scan. Without isolation, the verbose output fills the main session's context window and slows subsequent prompts. What is the correct fix?

- A) Reduce the dependency scan's verbosity by truncating output to 100 lines
- B) Add `context: fork` to the skill's frontmatter with `agent: Explore` — the scan runs in an isolated subagent context and only the summary returns to the main session
- C) Move the skill to user scope so it doesn't affect the team's main session
- D) Run the scan as a separate `claude -p` invocation outside the session

---

**Q6.** A developer's `~/.claude/CLAUDE.md` says "use 4-space indentation." The project's `./CLAUDE.md` says "use 2-space indentation." What is the most accurate statement about what Claude will do?

- A) Always 4 spaces — user preferences take precedence
- B) Always 2 spaces — project files are concatenated AFTER user files and read last, so in practice project rules typically win on conflict
- C) Whichever was edited most recently
- D) Both — Claude will ask the user which to use

---

**Q7.** A team wants to ensure Claude never runs `rm -rf` commands, even when developers use `--dangerously-skip-permissions` in CI containers. What is the correct enforcement mechanism?

- A) Add "never run rm -rf commands" to CLAUDE.md in capital letters
- B) Configure a `PreToolUse` hook that matches `Bash` and denies any command containing `rm -rf` — these hooks fire before any permission-mode check and override `--dangerously-skip-permissions`
- C) Set `permissions.defaultMode` to `plan` in `settings.json`
- D) Remove the Bash tool from all developer machines

---

**Q8.** A skill has `context: fork` set but its content is just reference material about API conventions, with no actionable task. What happens when the skill is invoked?

- A) The skill works correctly — Claude uses the conventions inline
- B) The forked subagent receives the conventions but no actionable prompt; it returns nothing useful
- C) The skill fails with a syntax error
- D) The conventions are applied to all future tool calls in the main session

---

**Q9.** A team uses one Claude Code session to both generate code and review it. Review quality is poor — Claude misses bugs in its own work. What is the correct fix?

- A) Use a more powerful model for review
- B) Add a system prompt instruction: "Be highly critical of your own code"
- C) Start an independent review session with only the diff loaded as input — no generation context to rationalize from
- D) Run `/clear` between generation and review

---

**Q10.** A developer wants to analyze a complex authentication subsystem before making any changes. They want Claude to read code, reason about architecture, and produce a plan — but NOT make any edits. What is the correct approach?

- A) Add "Don't make any edits, just analyze" to CLAUDE.md
- B) Use `--permission-mode plan` (or `/plan`) — Edit/Write/destructive Bash tools become unavailable, making it structurally impossible to edit
- C) Manually approve every edit one at a time
- D) Disable the Edit tool in the team's `settings.json`

---

**Q11.** A `.claude/rules/testing.md` file has the frontmatter `paths: ["**/*.test.tsx"]`. When does Claude load this rule?

- A) Always — frontmatter `paths` is decorative only
- B) Only when Claude reads a file matching the glob `**/*.test.tsx`
- C) Only when the user explicitly invokes it with `/testing`
- D) Only when running in plan mode

---

**Q12.** Where should a team's shared Jira MCP server configuration live so every developer gets it on `git clone`?

- A) `~/.claude.json` on each developer's machine
- B) `.mcp.json` in the project root, committed to version control
- C) `.claude/settings.local.json` with a note in the README to copy it manually
- D) The system prompt in CLAUDE.md, with the Jira credentials inline

---

**Q13.** A CI job uses `claude -p "review the diff" --output-format json` and processes the response. The script extracts `.content` and `.error` fields, but always reads them as empty. What is wrong?

- A) The JSON output field names are wrong — Claude's CLI JSON uses `result` for the response text and `is_error` (boolean) for error state; the correct fields are `result`, `is_error`, `total_cost_usd`, `num_turns`, and `session_id`
- B) The output format should be `text`, not `json`
- C) The script needs to wait for the streaming output to complete
- D) The API key is missing

---

**Q14.** A skill is configured with `agent: Explore` (read-only). The skill's task is "scan the codebase for outdated dependencies and update package.json with the new versions." What happens?

- A) The skill works correctly — Explore can write files when needed
- B) The skill silently fails — Explore is read-only, so the package.json update never happens
- C) The skill is auto-converted to `agent: general-purpose` at runtime
- D) Claude Code raises a syntax error on the YAML frontmatter

---

**Q15.** A team wants to move both a blocking pre-merge CI check and a nightly tech-debt scan to the Anthropic Batch API. What is the correct decision?

- A) Move both — the 50% cost savings justifies it
- B) Move neither — Batch API is not suitable for either workflow
- C) Move only the nightly tech-debt scan to Batch API (no latency SLA needed); keep the blocking pre-merge check on synchronous `claude -p` (developers can't wait up to 24 hours for a PR check)
- D) Move only the pre-merge check to Batch API for cost savings

---

**Q16.** A developer runs `claude --resume <session-id>`. Since the last session, two core authentication files were refactored by a teammate. The developer doesn't mention the changes. What is the most likely outcome?

- A) Claude detects the file changes automatically and re-reads them
- B) Claude reasons from its stale previous analysis of the refactored files, producing silently incorrect results
- C) Claude refuses to resume because file checksums no longer match
- D) Claude asks the developer which files have changed before proceeding

---

**Q17.** What does the `/memory` slash command do in Claude Code?

- A) Forces a reload of all CLAUDE.md files from disk
- B) Lists all CLAUDE.md, CLAUDE.local.md, and rules files currently loaded in the session, lets you toggle auto memory, and opens listed files in your editor — useful for debugging why a rule isn't applying
- C) Clears the conversation history
- D) Compacts the context window to free up tokens

---

**Q18.** A team's `.mcp.json` includes `"JIRA_API_TOKEN": "sk-jira-actual-token-here-abc123"`. A developer commits and pushes this file. What is the consequence and the correct fix?

- A) Nothing — `.mcp.json` is automatically encrypted by Claude Code before commit
- B) The token is now exposed in git history; the correct fix is to rotate the token immediately AND change `.mcp.json` to use `"JIRA_API_TOKEN": "${JIRA_API_TOKEN}"` so future commits substitute from environment variables
- C) Only a warning — the token is encrypted in transit so git history is safe
- D) Claude Code will refuse to read the file, so no exposure occurred

---

**Q19.** A skill is invoked with `/audit`. Its YAML frontmatter has `hooks:` defining a `PostToolUse` hook that runs Prettier after every Edit. When does this hook fire?

- A) For every Edit in the entire session, even after the audit skill completes
- B) Only while the audit skill is the active context — the hook is scoped to the skill's lifecycle
- C) Never — skill-level hooks are not supported
- D) Only if the user also adds the hook to `settings.json`

---

**Q20.** A developer edits CLAUDE.md while in an active Claude Code session. After running `/compact`, they notice some of their newly added instructions are gone. What is happening?

- A) `/compact` corrupts the CLAUDE.md file
- B) Project-root CLAUDE.md survives `/compact` (re-read from disk), but instructions added only as conversation messages are LOST in compaction; the developer should add persistent rules to CLAUDE.md, not just chat
- C) The session needs to be restarted to pick up CLAUDE.md edits
- D) `/compact` permanently deletes the session

---

**Q21.** A `PreToolUse` hook returns `permissionDecision: "deny"` for the Bash tool when the command contains `sudo`. A developer runs Claude Code with `--dangerously-skip-permissions` in a CI container. Claude tries to run `sudo rm -rf /tmp/cache`. What happens?

- A) The command runs because `--dangerously-skip-permissions` bypasses all checks
- B) The hook still blocks the command — `PreToolUse` hooks fire before any permission-mode check and CAN override `--dangerously-skip-permissions`
- C) The hook is silently ignored in headless mode
- D) Claude Code crashes

---

**Q22.** A team's project has files across `src/api/`, `src/web/`, and `src/workers/`. The team wants different coding rules for each area, without bloating context on every session. What is the most maintainable structure?

- A) One massive `CLAUDE.md` at the root with all rules
- B) A lean root `CLAUDE.md` for shared rules PLUS `src/api/CLAUDE.md`, `src/web/CLAUDE.md`, `src/workers/CLAUDE.md` (which load on-demand when Claude touches files in those subdirectories), or `.claude/rules/<topic>.md` with appropriate `paths:` globs
- C) One `CLAUDE.md` per developer in `~/.claude/CLAUDE.md` containing all rules
- D) No `CLAUDE.md` at all — rely on prompt instructions per session

---

**Q23.** A team uses `claude -p` in GitHub Actions but does not set `ANTHROPIC_API_KEY`. What happens?

- A) Claude Code uses a free tier with reduced capabilities
- B) Claude Code fails at startup with an authentication error — the API key is required for headless mode in CI
- C) Claude Code falls back to a cached response from the last interactive session
- D) Claude Code automatically generates a temporary API key

---

**Q24.** When should you use a subagent (`.claude/agents/`) instead of a skill?

- A) Subagents are always preferred — skills are deprecated
- B) Use a subagent when you need verbose intermediate work isolated from the main session's context AND a recurring specialist role (security reviewer, test runner); use a skill for inline reusable workflows that run in the main context (or a skill with `context: fork` for one-off isolated tasks)
- C) Use a subagent only for tasks under 100 lines of output
- D) Subagents only work in plan mode; skills work everywhere

---

**Q25.** A `.claude/settings.json` has hook configurations and permission rules. A developer wants personal-only adjustments without modifying the team's shared settings. Where should they put their overrides?

- A) Edit `.claude/settings.json` and add a `// personal` comment
- B) `.claude/settings.local.json` — gitignored by default, supports the same schema, and merges with the team settings (permission rules merge; other settings override at higher precedence)
- C) `~/.claude/CLAUDE.md` — settings can be embedded in CLAUDE.md
- D) Create a new file `.claude/settings.<username>.json`

---

## 20. Answer Key & Explanations

| Q  | Answer | Key Reason                                                                                                          |
| -- | ------ | ------------------------------------------------------------------------------------------------------------------- |
| 1  | **B**  | User-level CLAUDE.md is not version-controlled; team conventions must live in project-level files committed to git |
| 2  | **C**  | Lean CLAUDE.md + `.claude/rules/` with `paths:` for conditional loading — the canonical exam answer                |
| 3  | **D**  | `-p` is the headless flag; without it Claude Code launches its REPL and waits for input that never comes           |
| 4  | **B**  | Without `disable-model-invocation: true`, Claude can auto-invoke high-risk skills when description matches intent  |
| 5  | **B**  | `context: fork` with `agent: Explore` isolates verbose output; only the summary returns to the main session       |
| 6  | **B**  | Files are concatenated; project files are read after user files, so in practice project rules win on conflict      |
| 7  | **B**  | `PreToolUse` hooks fire before permission-mode checks; deny works even with `--dangerously-skip-permissions`       |
| 8  | **B**  | `context: fork` only makes sense for skills with explicit actionable tasks, not reference material                 |
| 9  | **C**  | Independent review session — structural fix; generation context biases self-review                                 |
| 10 | **B**  | Plan Mode makes Edit/Write structurally unavailable, not just discouraged                                          |
| 11 | **B**  | `paths:` frontmatter loads the rule conditionally when Claude reads files matching the glob                        |
| 12 | **B**  | `.mcp.json` at project root, committed to git — all developers get it on clone                                     |
| 13 | **A**  | JSON output uses `result`, `is_error`, `total_cost_usd`, `num_turns`, `session_id` — not `content`/`error`         |
| 14 | **B**  | `agent: Explore` is read-only — write tasks silently fail when paired with it                                      |
| 15 | **C**  | Batch API for async non-time-sensitive; synchronous `-p` for blocking checks developers wait on                    |
| 16 | **B**  | Claude doesn't detect file changes — it reasons from stale prior analysis until explicitly told what changed       |
| 17 | **B**  | `/memory` lists what's loaded, toggles auto memory, opens files — the debugging tool for "why isn't my rule applying?" |
| 18 | **B**  | Token is in git history (rotate immediately); future commits should use `${ENV_VAR}` substitution                  |
| 19 | **B**  | Skill-level hooks are scoped — they fire only while the skill is the active context                                |
| 20 | **B**  | Project-root CLAUDE.md survives compaction; conversation-only instructions do NOT — promote them to CLAUDE.md      |
| 21 | **B**  | `PreToolUse` hooks deny even in `--dangerously-skip-permissions` mode — this is the documented safety net          |
| 22 | **B**  | Subdirectory CLAUDE.md files load on-demand when Claude touches that subdirectory — additive context where relevant |
| 23 | **B**  | `ANTHROPIC_API_KEY` is required for headless CI/CD — must be injected as a CI secret                              |
| 24 | **B**  | Subagents = isolated context + recurring specialist role; skills = inline workflows (or use `context: fork` for one-off isolation) |
| 25 | **B**  | `.claude/settings.local.json` is the documented personal-override location, gitignored by default                 |

---

## 21. Quick Cheat Sheet — Domain 3

```
CLAUDE.md HIERARCHY (3.1)
  → 4 scopes: Managed (enterprise) | User (~/.claude) | Project (./CLAUDE.md) | Local (./CLAUDE.local.md — DEPRECATED, use @imports instead)
  → CONCATENATED, not overridden — more-specific files read LAST
  → Subdirectory CLAUDE.md loads on-demand when Claude touches files there
  → Team convention not propagating? It's in user-level — move to project
  → 200+ line CLAUDE.md? Split: lean root + .claude/rules/<topic>.md with paths:
  → /memory = list what's loaded | /init = generate starter | /clear ≠ reset (keeps CLAUDE.md)
  → claudeMdExcludes in settings.local.json skips monorepo noise
  → @import composes files; max depth 5; relative to importing file
  → Modern personal-prefs pattern: @~/.claude/my-project-instructions.md (cross-worktree)
  → AGENTS.md not read by Claude — bridge with @AGENTS.md import

AUTO MEMORY (3.1)
  → Claude's notebook: ~/.claude/projects/<proj>/memory/MEMORY.md
  → First 200 lines / 25KB loaded into every session
  → /memory toggles; CLAUDE_CODE_DISABLE_AUTO_MEMORY=1 force-disables
  → Machine-local, not shared across machines

SKILLS & COMMANDS (3.2)
  → Custom commands in .claude/commands/<name>.md → /name
  → Skills in .claude/skills/<name>/SKILL.md (preferred, supersedes commands)
  → Skill frontmatter: name, description, allowed-tools, hooks, paths, context, agent, disable-model-invocation, user-invocable, model
  → disable-model-invocation: true for high-risk ops (deploy/delete/commit/migrate)
  → user-invocable: false hides from /menu (background knowledge only)
  → If both command and skill exist with same name → skill wins
  → Skill content STAYS in context across turns — every line is recurring token cost
  → !`command` runs shell BEFORE Claude sees content (preprocessing)

CONTEXT: FORK & SUBAGENTS (3.3)
  → context: fork in skill = run in isolated subagent, summary back to main
  → agent: Explore (read-only) | Plan (no writes) | general-purpose | custom
  → Explore and Plan SKIP CLAUDE.md and git status (tiny context)
  → Misconfigured agent type silently fails — match to actual capability
  → context: fork only for skills with EXPLICIT TASKS
  → Skill = main context | Subagent = own context window
  → Subagents in .claude/agents/<name>.md — auto-invoke or @name
  → Use "PROACTIVELY" / "MUST BE USED" in subagent description for auto-invoke
  → Subagent priority: Managed > --agents CLI > Project > User > Plugin
  → Subagents DO load CLAUDE.md by default (except built-in Explore and Plan, which skip it)
  → Subagents CANNOT spawn other subagents
  → Built-ins: Explore (Haiku, read-only) | Plan (read-only, plan mode) | general-purpose (all tools)
  → Task tool was renamed to Agent in v2.1.63 — Task(...) still works as alias
  → Forked subagents (CLAUDE_CODE_FORK_SUBAGENT=1) inherit FULL parent conversation
  →   ↑ different from skill-level context: fork (which has FRESH context)
  → Subagent frontmatter fields: name, description, tools, disallowedTools, model, permissionMode, maxTurns, skills, mcpServers, hooks, memory, background, effort, isolation, color, initialPrompt

PLAN MODE (3.4)
  → claude --permission-mode plan OR /plan OR Shift+Tab twice
  → Available: Read, Grep, Glob, LS, safe Bash (cat/head/ls/git status/log/diff), WebFetch, WebSearch, Skill
  → Blocked: Edit, Write, destructive Bash (rm/mv/commit/push/install)
  → ExitPlanMode tool surfaces the plan for user approval — exits on approval
  → Plan in session A → execute in fresh session B with plan as input
  → Beats "just don't edit" prompt instruction every time

PERMISSION MODES (3.4 — 6 modes total)
  → default — asks before each new tool/file
  → plan — read-only; Edit/Write/destructive Bash unavailable
  → acceptEdits — auto-approve Edit/Write in working dir
  → auto — classifier reviews each call (released Mar 24, 2026) — middle ground
  → dontAsk — auto-DENY all prompts; only pre-allowed tools execute (CI lockdown)
  → bypassPermissions — skip all prompts (still respects PreToolUse deny + safety paths)
  → Shift+Tab cycle: default → acceptEdits → plan → auto
  → Safety-immune paths even in bypass: .git/ .claude/ .vscode/ .idea/ .husky/ + rm -rf /

HOOKS (3.5)
  → Many events: PreToolUse, PostToolUse, PostToolUseFailure, UserPromptSubmit, UserPromptExpansion, Stop, StopFailure, SubagentStart, SubagentStop, SessionStart, SessionEnd, PreCompact, PostCompact, InstructionsLoaded, ConfigChange, CwdChanged, FileChanged, PermissionRequest, PermissionDenied, Notification, PostToolBatch, TaskCreated, TaskCompleted, WorktreeCreate, WorktreeRemove, Elicitation, ElicitationResult, Setup, TeammateIdle
  → Hook types: command (default) | http | mcp_tool | prompt (LLM) | agent (LLM+tools)
  → Configured in .claude/settings.json with matchers
  → PreToolUse fires BEFORE permission check — deny overrides --dangerously-skip-permissions
  → Hooks tighten, never loosen — "allow" doesn't bypass settings deny rules
  → PostToolUse cannot undo a tool — use for format/log/feedback
  → Stop fires every response end (use SessionEnd for true session cleanup)
  → Stop block cap = 8 consecutive; check stop_hook_active to avoid
  → Don't mix exit 2 with JSON output — Claude ignores JSON on exit 2
  → permissionDecision: allow | deny | ask | defer (defer is -p only)
  → PermissionRequest hooks DON'T fire in -p mode — use PreToolUse for CI/CD
  → Skill-scoped hooks fire only while skill is active
  → Multiple PreToolUse rewriting same input → last to finish wins (avoid)
  → Re-inject context after compact: SessionStart hook with matcher "compact"
  → Hook timeouts: command/http/mcp_tool=10min | UserPromptSubmit=30s | prompt=30s | agent=60s
  → InstructionsLoaded = debug "which rules loaded and why"

HEADLESS MODE / CI-CD (3.6)
  → claude -p "prompt" (non-interactive — exits after response)
  → Missing -p = pipeline hangs (#1 exam answer)
  → -p mode SKIPS workspace trust dialog — only in trusted directories
  → CLAUDE_HEADLESS=true is NOT real | --batch is NOT real (distractors)
  → Required: ANTHROPIC_API_KEY as CI secret
  → --output-format json (parseable result/is_error/total_cost_usd/num_turns/session_id/duration_ms)
  → --input-format stream-json for bidirectional NDJSON via stdin
  → --append-system-prompt "..." (only with --print) appends to system prompt
  → --permission-prompt-tool mcp__auth__prompt routes permission decisions via MCP tool
  → --max-turns N caps cost
  → --dangerously-skip-permissions in CI containers only, + PreToolUse hooks for safety
  → --continue (-c) | --resume <id> work with -p
  → --mcp-config <file> loads MCP servers in headless
  → Synchronous -p for blocking checks | Batch API for async overnight work

MCP IN CLAUDE CODE (3.7)
  → .mcp.json (project, committed) for team servers
  → ~/.claude.json (personal, not committed)
  → Secrets: ${ENV_VAR} always — never hardcoded
  → Tools appear as mcp__<server>__<tool> in allowed-tools

SESSION MANAGEMENT & COMPACTION (3.8)
  → claude (new) | claude -c (continue most recent) | claude --resume <id>
  → After file changes: --resume + EXPLICITLY note what changed
  → /compact: re-injects project CLAUDE.md but NOT nested subdir CLAUDE.md
  → Conversation-only rules LOST after /compact — promote to CLAUDE.md
  → /clear: clears conversation but KEEPS CLAUDE.md and system prompt
  → Independent review session (no generation context) for unbiased review

SETTINGS PRECEDENCE
  → Managed > .claude/settings.json > .claude/settings.local.json > ~/.claude/settings.json
  → Permission rules MERGE; other settings OVERRIDE
  → settings.local.json is gitignored — personal overrides

META-PATTERN
  → Every Domain 3 wrong answer substitutes a prompt instruction for a structural fix
  → Configuration is layered onboarding for an engineer with perfect recall but total amnesia
  → "Why doesn't this work?" → Ask: was it written somewhere everyone reads, or only you?
```

---

*CCA-F Domain 3 Study Guide | Prepared for Arun | May 2026*
*v4 — Re-verified against authoritative Claude Code sub-agents docs (code.claude.com/docs/en/sub-agents)*
*Corrections: subagents DO load CLAUDE.md (except Explore/Plan), 6 permission modes (added auto + dontAsk),*
*full frontmatter field reference, Task→Agent rename (v2.1.63), forked subagents distinct from skill-level context: fork,*
*subagents cannot spawn other subagents, built-in subagent table (Explore Haiku, Plan inherit, general-purpose all)*
*Companion to: Domain 1 (Agentic Architecture, 27%) | Domain 2 (Tool Design & MCP, 18%)*
*Next: Domain 4 — Prompt Engineering & Structured Output (20%)*