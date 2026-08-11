# Wrong Answer Tracker — Code Generation with Claude Code

> Log every missed question here with the **why**, not just the correct answer. The goal is to catch *patterns* in your mistakes, not just memorize individual answers.

**Problem-category legend:**

- 🧩 **Always-Loaded vs. Workflow-Triggered** → CLAUDE.md (universal, always active) vs. Skills (on-demand, invoked for a specific workflow)
- 👤 **Personal Override Precedence** → user-level (`~/.claude/`) same-name files override project-level for that individual without touching the shared version
- 🎯 **Automatic-Activation vs. On-Demand-Invocation** → path-specific rules activate automatically by file pattern; skills/commands are invoked deliberately for a specific task, not a file type
- 🏗️ **Inventing Non-Existent Mechanisms** → reaching for a plausible-sounding but fictional Claude Code feature instead of the real, documented one
- ⚙️ **Overengineering vs. Built-In Mechanism** → building custom infrastructure (wrappers, proxies) when Claude Code already has a native, simpler mechanism for the same problem
- 🔍 **Incomplete Coverage in Exploration Strategy** → picking an exploration approach that works for the *known*/*obvious* cases but silently misses cases that require deeper or indirect tracing (renames, aliases, indirection)
- 📛 **Name-Based vs. Identity-Based Session Resume** → confusing "I know an identifying label" (session name) with "I need to look up an opaque identifier first" (UUID) when a more direct resume mechanism already accepts the label you have

---

## Mock Test 1 (`mock-test-code-generation-1.md`) — 10/15 (67%)

### Q33 — CLAUDE.md mixing always-relevant standards with workflow-specific procedures (PR review, deploy, migrations)

- **Your answer:** B — Keep everything in CLAUDE.md but use `@import` syntax to organize into separately maintained files by category
- **Correct answer:** D — Keep universal standards in CLAUDE.md and create Skills for workflow-specific guidance (PR review, deploy, migrations) with trigger keywords
- **Category:** 🧩 Always-Loaded vs. Workflow-Triggered
- **Why you missed it:** `@import` is a *modularization* tool — it splits files for maintainability but everything imported into CLAUDE.md still loads on every single invocation, regardless of whether the task is a PR review or a plain code-writing session. The stem's actual requirement was **conditional loading** ("apply PR review, deploy, and migration guidance only when doing those tasks"), which `@import` cannot provide. Skills are invoked on-demand for a specific workflow, so they're the correct mechanism for the parts of the file that shouldn't always be active.
- **Rule to remember:** `@import` solves "this file is too big to maintain in one place" (organization). It does **not** solve "this content should only load sometimes" (conditional/selective loading). If the stem asks for guidance to apply *only* during a specific workflow, reach for Skills (task-triggered), not `@import` (always-loaded, just reorganized).

---

### Q36 — Personal customization of a shared `/commit` skill without affecting teammates

- **Your answer:** A — Create a personal version under `~/.claude/skills/` with a different name, e.g., `/my-commit`
- **Correct answer:** C — Create a personal version at `~/.claude/skills/commit/SKILL.md` with the **same name**
- **Category:** 👤 Personal Override Precedence
- **Why you missed it:** Assumed that reusing the same name would somehow conflict with or overwrite the shared project skill. In practice, a user-level skill with the same name as a project-level skill takes precedence *for that individual developer only* — it's a personal override, not a shared-file edit. The teammates who don't have a personal override still get the project version untouched. Renaming to `/my-commit` works too but isn't what the correct option actually tests: it's checking whether you know that same-name, different-scope files provide precedence-based personalization without renaming friction (remembering to type a different command).
- **Rule to remember:** In Claude Code's scoping hierarchy, a **more personal scope with the same name overrides a shared scope with that name, only for that person** — it does not mutate or affect the shared file. Don't assume "same name = conflict"; same name at a *narrower* scope is the standard override pattern.

---

### Q38 — Endpoint examples useful only when creating new endpoints, not for debugging/review in the same directory

- **Your answer:** C — Configure path-specific rules in `.claude/rules/api/` that include endpoint examples and activate when working in the API directory
- **Correct answer:** D — Create a skill that references the endpoint examples and contains pattern-following instructions, invoked on demand via a slash command
- **Category:** 🎯 Automatic-Activation vs. On-Demand-Invocation
- **Why you missed it:** Pattern-matched this to the "test conventions apply to all `*.test.tsx` files regardless of directory" scenario (path-specific rules), but the trigger condition here is different: it's not "this file *type*/*path*" — it's "this specific *task* (creating a new endpoint)" within a directory where *other* tasks (debugging, review) also happen. Path-specific rules activate automatically based on which file you're editing, so they would load the endpoint examples even during debugging or code review in that same API directory — which the stem explicitly says is undesired. A skill invoked on-demand only fires when the developer deliberately wants to generate a new endpoint.
- **Rule to remember:** Ask *what triggers the need*: if it's "any time I touch this file/path" → path-specific rules. If it's "only for this specific task, even though it happens in a directory where other unrelated tasks also occur" → a skill invoked on demand. File-path scoping and task-type scoping are not the same axis, even when both happen to live in the same directory.

---

### Q42 — CLAUDE.md over 500 lines, developers can't find/update the right sections

- **Your answer:** A — Define a `.claude/config.yaml` mapping file patterns to specific sections inside CLAUDE.md
- **Correct answer:** B — Create separate Markdown files in `.claude/rules/`, each covering one topic (e.g., `testing.md`, `api-conventions.md`)
- **Category:** 🏗️ Inventing Non-Existent Mechanisms
- **Why you missed it:** `.claude/config.yaml` mapping file patterns to CLAUDE.md sections is not a real Claude Code feature — it sounds plausible (config file + glob mapping) but doesn't exist. `.claude/rules/` with separate topical Markdown files is the actual, documented mechanism for splitting a monolithic CLAUDE.md into focused modules.
- **Rule to remember:** When an option describes a *very specific-sounding config file/schema* you don't recognize from the material (e.g., `.claude/config.yaml`, `.claude/config.json`), be suspicious — cross-check it against the actual known mechanisms (CLAUDE.md, `@import`, `.claude/rules/`, `.claude/skills/`, `.claude/commands/`, `.mcp.json`) before picking it. If it's not on that list, it's very likely a distractor.

---

### Q44 — Consistent GitHub MCP tooling across 6 developers, each with personal tokens, without committing credentials

- **Your answer:** B — Create an MCP server wrapper that reads tokens from a `.env` file and proxies GitHub API calls, then add the wrapper to the project `.mcp.json`
- **Correct answer:** C — Add the server to the project `.mcp.json` using environment variable substitution (`${GITHUB_TOKEN}`) for auth and document the required environment variable in the project README
- **Category:** ⚙️ Overengineering vs. Built-In Mechanism
- **Why you missed it:** Built a custom proxy/wrapper solution for a problem that `.mcp.json`'s native environment variable substitution already solves directly — the project config stays shared and version-controlled while each developer's token resolves locally from their own environment, with zero credentials ever touching git. The wrapper approach adds unnecessary infrastructure (a whole intermediary server) to reinvent something already built in.
- **Rule to remember:** Before proposing custom infrastructure (wrappers, proxies, sidecar services) to solve a "shared config + per-developer secret" problem, check whether the config format already supports environment variable substitution or an equivalent native mechanism. Claude Code's `.mcp.json` does — favor the built-in mechanism over custom tooling whenever it fully satisfies the constraint.

---

## Mock Test 2 (`mock-test-code-generation-2.md`) — 13/15 (87%)

### Q3 — Finding all callers of a function exposed under renamed names via wrapper modules

- **Your answer:** A — Read the library and wrapper modules to identify all exposed names, then Grep for each name across the codebase
- **Correct answer:** B — Use Grep to find all files that import from the library or wrapper modules, then read each file to check whether it uses the function
- **Category:** 🔍 Incomplete Coverage in Exploration Strategy
- **Why you missed it:** A only catches renamed variants you've actually discovered by reading the wrapper modules you happened to check. If there's a deeper rename chain — a third module re-exporting `computeOrderTax` under yet another domain-specific name — A silently misses those callers because it depends on first correctly enumerating *every* alias. Tracing all files that import from the library/wrapper modules and checking actual usage is exhaustive by construction: it doesn't require you to have already guessed every possible name in advance.
- **Rule to remember:** When a stem emphasizes "most **reliably**" identify all callers/usages in a codebase with intentional renaming/aliasing, be suspicious of any strategy that depends on first enumerating a finite list of known names. Prefer strategies that trace structural relationships (imports, references) over strategies that trace known name strings — name-based search only covers what you already know to look for.

### Q12 — Resuming a specifically-named session after working on other codebases since

- **Your answer:** B — Use `--session-id` with the UUID from yesterday's session transcript file
- **Correct answer:** D — Use `--resume auth-deep-dive` to load that specific session by name
- **Category:** 📛 Name-Based vs. Identity-Based Session Resume
- **Why you missed it:** The stem hands you the information you actually need — the session's **name** ("auth-deep-dive") — and a resume-by-name mechanism exists that accepts it directly. Going to fetch the UUID from a transcript file first is a valid but unnecessarily indirect extra step when the more direct, name-based option is already available and sufficient.
- **Rule to remember:** When the stem explicitly gives you an identifying label (a session *name*), check whether the resume mechanism accepts that label directly before reaching for a lower-level identifier (UUID) that requires an extra lookup step to obtain. Prefer the option that uses the information already in hand.

## Mock Test 3 — Final / Exam-Readiness Check (`mock-test-code-generation-3-final.md`) — 23/25 (92%)

### Q8 — Release Checklist (workflow-specific, 12 steps) mixed into always-loaded CLAUDE.md alongside day-to-day conventions

- **Your answer:** C — Move the checklist to `.claude/rules/release.md` with `paths: ["CHANGELOG.md"]` so it activates whenever the changelog file is touched
- **Correct answer:** A — Move the Release Checklist into a `/cut-release` skill invoked only when a release is being prepared; leave day-to-day conventions in CLAUDE.md
- **Category:** 🧩 Always-Loaded vs. Workflow-Triggered
- **Why you missed it:** This is the same underlying pattern as Mock 1's Q33 miss, wearing different clothes. The Release Checklist isn't triggered by *a file type or path* — releases touch many different files (version manifests, changelogs, tags, announcement docs) and "touching the changelog" is both an unreliable proxy (people edit changelogs for reasons unrelated to cutting a release) and an incomplete one (a release involves far more than that one file). The real trigger is a *task* ("I am cutting a release now"), which is exactly what an on-demand skill is for — not a path-specific rule, which only knows about file patterns, not developer intent.
- **Rule to remember:** Before reaching for a path-specific rule, ask: "is there one reliable file pattern that always co-occurs with this need, and only this need?" If the trigger is really a *task the developer decides to start* rather than a *file they happen to touch*, it's a skill, not a rule — even if you can name a file that's loosely associated with the task. This pattern from Mock 1 (Q33) is confirmed **still active** — it resurfaced on a fresh scenario in the final test, meaning the earlier "clean" run in Mock 2 didn't fully close it.

---

### Q17 — Skill frontmatter combination for `/generate-migration` (isolate verbose internal output + restrict to file-creation only + prompt for missing argument)

- **Your answer:** C — `allowed-tools: [Write]`, `argument-hint: "..."` (omit `context: fork` since file generation doesn't produce verbose output worth isolating)
- **Correct answer:** A — `context: fork`, `allowed-tools: [Write]`, `argument-hint: "..."`
- **Category:** 🆕 New — Underestimating When `context: fork` Is Needed
- **Why you missed it:** The stem explicitly states the skill "produces verbose internal schema-diffing output" that shouldn't clutter the main conversation — that is the textbook trigger for `context: fork`, regardless of whether the *end result* (a new migration file) sounds like a simple, low-output operation. The mistake was judging the need for isolation by the nature of the skill's *purpose* ("just generates a file") rather than by its *actual documented behavior* (verbose internal output during execution). A skill can produce a small, simple final artifact while still generating a lot of noisy intermediate reasoning/output that deserves isolation.
- **Rule to remember:** Judge the need for `context: fork` by what the stem says the skill *actually outputs during execution*, not by what its end deliverable is. "Generates one file" and "produces verbose internal output while generating that file" are not mutually exclusive — don't let a simple-sounding deliverable talk you out of an isolation need the stem already told you exists.

---

## Recurring Patterns Across Mistakes

1. **Confusing "reorganize/modularize" with "conditionally load"** (Mock 1: Q33, echoed in Q42) — `@import` and topical file splits both restructure content, but only actual scoping mechanisms (Skills, path-specific rules) control *when* something loads. Watch for stems that ask for conditional/selective application specifically.
2. **Misjudging scope-override behavior** (Mock 1: Q36) — assumed same-name files across scopes create conflicts rather than clean, expected overrides. Review the precedence order: user-level > project-level for personal customization, without mutating the shared file.
3. **Conflating file-path scoping with task-type scoping** (Mock 1: Q38) — both "applies to this directory" and "applies to this workflow" can look similar when the workflow happens to occur in a specific directory. Isolate what actually triggers the need for the context/rule before choosing the mechanism.
4. **Reaching for invented or overbuilt solutions instead of the actual native mechanism** (Mock 1: Q42, Q44) — a recurring instinct to construct a bespoke config file or wrapper service when Claude Code already has a simpler, documented, built-in way to do it (`.claude/rules/`, `${VAR}` substitution in `.mcp.json`). Before proposing new infrastructure, scan the known toolset first.
5. **Choosing a strategy that covers only the known/enumerable cases, missing indirect ones** (Mock 2: Q3) — new pattern. When a problem involves renaming, aliasing, or indirection, name/string-based strategies only cover what's already been discovered; structural tracing (imports, references) is exhaustive by construction.
6. **Reaching for an indirect identifier-lookup path when a direct, label-based mechanism already accepts the information given** (Mock 2: Q12) — new pattern. When the stem hands you a name/label, check for a resume/lookup mechanism that accepts that label before assuming you need to go fetch a lower-level ID first.
7. **Mistaking a loosely-associated file for the real trigger** (Mock 3: Q8) — recurrence of pattern #1/🧩. When no single file/path reliably and completely captures "when this task happens," the trigger is task-based (skill), not path-based (rule) — even if one file is tempting to bind to.
8. **Letting a simple-sounding end deliverable override an explicitly stated verbose-output trigger** (Mock 3: Q17) — new pattern. Judge `context: fork` need by what the stem says the skill *actually outputs during execution*, not by how simple its final artifact sounds.

## Score Trend

| Mock Test | Score | Missed |
|---|---|---|
| Mock 1 | 10/15 (67%) | Q33, Q36, Q38, Q42, Q44 |
| Mock 2 | 13/15 (87%) | Q3, Q12 |
| Mock 3 (Final) | 23/25 (92%) | Q8, Q17 |

**Trend: 67% → 87% → 92%.** Continued improvement, but Mock 3 surfaced an important signal: **Q8 is a direct recurrence of the Mock 1 Q33 pattern** (always-loaded vs. workflow-triggered), just in a fresh scenario. Mock 2's clean run on that category was not full mastery — it just didn't happen to test a variant that used a "tempting but incomplete" file-based trigger as a distractor. This category should be treated as **still active, not closed**, until it survives a dedicated re-drill. Q17 is a new, narrower pattern (judging `context: fork` necessity by the skill's *deliverable* rather than its *stated execution behavior*) — isolated so far, but worth watching for recurrence.

## Action Items Before Next Mock Test

- [ ] Before answering, ask: "does this option *restructure* the content, or does it *change when/whether it loads*?" — these are different problems requiring different mechanisms.
- [ ] Memorize scope precedence cold: `~/.claude/` (personal) same-name file overrides `.claude/` (project) for that individual only — it never edits or breaks the shared file for teammates.
- [ ] When a rule/context should apply only for a *specific task* (not a file type or path), default to skills/commands (on-demand) over path-specific rules (automatic-by-path). **Watch specifically for a distractor that names one file loosely associated with the task (e.g., CHANGELOG.md for "cutting a release") — a loosely-associated file is not the same as a reliable, complete trigger.**
- [ ] Cross-check any option naming a specific config file/schema (e.g., `.claude/config.yaml`) against the real known mechanism list: `CLAUDE.md`, `@import`, `.claude/rules/`, `.claude/skills/`, `.claude/commands/`, `.mcp.json`. If it's not one of these, treat it as a likely distractor.
- [ ] Before proposing custom infrastructure (wrappers, proxies) for a config/secrets problem, check whether the native config format (e.g., `.mcp.json` env var substitution) already solves it.
- [ ] For "find all callers/usages" questions involving renaming or aliasing, prefer strategies that trace structural relationships (imports/references) over strategies that depend on enumerating known name strings — the latter can't cover indirection you haven't discovered yet.
- [ ] For session-resume questions, check whether the stem already gave you a usable label (session name) before assuming you need a lower-level identifier (UUID) — use the most direct mechanism that matches the information already in hand.
- [ ] **New:** For skill-frontmatter questions, judge `context: fork` necessity from the stem's stated *execution behavior* (verbose output, internal reasoning, discovery phases), not from how simple or small the skill's *final deliverable* sounds.
- [ ] **Priority:** Category 🧩 (always-loaded vs. workflow-triggered) needs a dedicated re-drill of 3-5 fresh questions before Scenario 2 can be called fully exam-ready — it has now caused a miss in 2 of 3 mocks.
