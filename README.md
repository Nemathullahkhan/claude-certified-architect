# Claude Certified Architect — Foundations Study Guide

Unofficial study guide for the **Claude Certified Architect — Foundations (CCAR-F)** certification exam by Anthropic.

This repo covers all 5 exam domains with detailed explanations, anti-patterns, decision frameworks, and practice questions — plus full walkthroughs of the 6 official exam scenarios and a mock-test bank with a wrong-answer tracker.

It is actively maintained and updated to track changes to Anthropic's models, APIs, and Claude Code. Last refresh: 2026-08 — see the dated notes under [Resources](#resources) for what changed.

> **Note:** the domain deep-dive guides in `scenarios/domain-*.md` refer to this exam by its earlier short form **CCA-F**. That's the same exam as **CCAR-F** below — the short name predates Anthropic's current four-credential naming scheme.

## The Claude Certification Family

Anthropic now runs four certifications. This repo's deep-dive guide covers **CCAR-F** only; overview guides for the other three are not included here.

| Code | Credential | Audience | Fee |
|---|---|---|---|
| CCAO-F | Associate — Foundations | Business / productivity users (non-developer) | $99 |
| **CCAR-F** | **Architect — Foundations** | **Solution architects** | **$125** — *this repo* |
| CCAR-P | Architect — Professional | Senior architects owning the full solution lifecycle | $175 |
| CCDV-F | Developer — Foundations | Engineers shipping Claude apps, agents, and workflows | $125 |

All four are 120-minute proctored exams delivered via Pearson VUE, passing score 720/1,000, credentials valid 12 months (Exam Guide v1.0, effective July 2026).

## Exam Overview

- **Format:** 60 multiple-choice, scenario-based questions in 120 minutes (proctored, closed-book)
- **Passing score:** 720/1000
- **Scenarios:** 4 of 6 official scenarios randomly selected per exam
- **Delivery:** Pearson VUE (OnVUE online or test center), registered via the Anthropic Partner Academy
- **Price / validity:** $125 per attempt; certification valid 12 months (Exam Guide v1.0, effective July 2026) — supersedes the launch-period $99 fee / "first 5,000 partner employees free" terms
- **Target audience:** Solution architects with 6+ months experience building with Claude APIs, Agent SDK, Claude Code, and MCP; access is gated to the Claude Partner Network

## Domains

| Domain | Weight | Guide |
|---|---|---|
| Agentic Architecture & Orchestration | 27% | [Domain 1](scenarios/domain-1.md) |
| Tool Design & MCP Integration | 18% | [Domain 2](scenarios/domain-2.md) |
| Claude Code Configuration & Workflows | 20% | [Domain 3](scenarios/domain-3.md) |
| Prompt Engineering & Structured Output | 20% | [Domain 4](scenarios/domain-4.md) |
| Context Management & Reliability | 15% | [Domain 5](scenarios/domain-5.md) |

## How to Use This Repo

The exam doesn't test trivia — it tests whether you can look at a described production system and pick the *structurally correct* fix (hook vs. prompt, few-shot vs. architecture change, escalate vs. resolve). This repo is organized around that, in four layers:

1. **Theory** ([scenarios/theory.md](scenarios/theory.md)) — the concepts once, in one place: Messages API basics, `tool_use`, the Agent SDK (agentic loops, `AgentDefinition`, hooks, subagents), MCP, Claude Code configuration, prompt engineering, the Batches API, escalation design, multi-agent error handling, context management, and provenance.
2. **Domain deep-dives** (table above) — one long-form guide per official exam domain, each with its exam weight, a "master mental model," anti-pattern lists, and 20+ practice MCQs with answer keys. This is where most of the study time should go.
3. **Scenarios** ([scenarios/listedScenarios.md](scenarios/listedScenarios.md) + `scenario-1..6-*.md`) — the exam is scenario-anchored, so each of the 6 official scenarios gets its own walkthrough: system architecture, which domains/task-statements it exercises, scenario-specific traps, a practice question bank, and a cheat sheet.
4. **Mock tests** ([scenarios/mock-test/](scenarios/mock-test/)) — timed practice sets grouped by scenario, plus a `wrong-answers/` tracker that records *why* each miss happened (not just the right answer) and a `report/` folder tracking score trends across attempts.

### Repository structure

```
.
├── README.md                   # This file
├── CLAUDE.md                   # Orientation notes for Claude Code sessions
├── exam-guide.pdf               # Official Anthropic exam guide
├── Grep-vs-Glob.MD              # Quick reference: Grep vs Glob semantics
├── package.json / tsconfig.json   # Hands-on practice scaffold (work in progress, not yet populated)
└── scenarios/
    ├── theory.md                 # Theory foundations (13 chapters)
    ├── domains.md                 # Raw Anthropic task-statement text for Domains 1–5
    ├── domain-1.md .. domain-5.md   # Domain deep dives (one per row in the table above)
    ├── listedScenarios.md         # Overview of the 6 official exam scenarios
    ├── scenario-1-customer-support.md
    ├── scenario-2-code-generation.md
    ├── scenario-3-multi-agent-research.md
    ├── scenario-4-developer-productivity.md
    ├── scenario-5-cicd.md
    ├── scenario-6-structured-extraction.md
    └── mock-test/
        ├── csa/ mar/ cicd/ code-generation/   # Mock tests grouped by scenario
        ├── wrong-answers/      # Per-scenario miss logs with root-cause + rule to remember
        └── report/             # Score-trend reports per scenario
```

### Recommended study path

1. Read `scenarios/theory.md` once, start to finish, for the shared vocabulary.
2. Work through `scenarios/domain-1.md` → `domain-5.md` in order (they're ordered by exam weight) — read the mental model and anti-pattern lists, then answer each domain's practice MCQs before checking the answer key.
3. Read `scenarios/listedScenarios.md`, then each `scenario-N-*.md` — these show how the domains combine inside one realistic system.
4. Take a mock test from `scenarios/mock-test/{csa,mar,cicd,code-generation}/`. Score it cold, no notes.
5. For every miss, log it in the matching `scenarios/mock-test/wrong-answers/*.md` file with the *reason* you missed it, not just the correct letter.
6. Check `scenarios/mock-test/report/` for the score trend per scenario; retake targeted drills on whatever category is still weak until misses stop recurring.

### Hands-on practice scaffold (work in progress)

`package.json` and `tsconfig.json` set up a TypeScript scaffold (`@anthropic-ai/claude-agent-sdk`, `@anthropic-ai/sdk`) for building small runnable examples alongside the theory (e.g. an agentic loop for Domain 1). The referenced files (`domain-1/tasks/1/agenticLoop.ts`, `model/test-model.ts`) don't exist yet — this is a placeholder for future hands-on exercises, not a working example today.

## Resources

- [guide_en.md — Core topics & fundamentals overview](https://github.com/paullarionov/claude-certified-architect/blob/main/guide_en.md)
- [Claude Certification Guide — Mock Exam](https://claudecertificationguide.com/mock-exam)
- [CyberSkill Practice — CCAF](https://practice.cyberskill.world/practice/ccaf/exam)
- [CertSafari — CCAF](https://www.certsafari.com/anthropic/claude-certified-architect-foundations)
- Pearson VUE — Anthropic certification program *(register via the Anthropic Partner Academy)*
- Anthropic Skilljar — Building with Claude API
- Building Effective Agents (Anthropic Research)
- Claude Tool Use docs · Claude Code docs · MCP Introduction · Claude Agent SDK overview

*(The four items above are referenced by name only — pending a confirmed current URL for each, since Anthropic's docs recently reorganized; see the note below rather than a possibly-stale link.)*

**Note (2026-07):** Anthropic split its documentation — API docs now live under `platform.claude.com/docs/en/*` and Claude Code docs under `code.claude.com/docs/en/*`; older `docs.anthropic.com/en/docs/*` URLs redirect. The SDK has also been renamed from "Claude Code SDK" to the **Claude Agent SDK**. Current model lineup: **claude-fable-5** (Anthropic's most capable widely released model), **claude-opus-5** (most capable Opus-tier), **claude-sonnet-5**, and **claude-haiku-4-5**. Earlier-generation models (e.g. `claude-opus-4-7`, `claude-sonnet-4-6`) remain active but superseded.

**Program update (2026-07):** the certification family now spans four credentials (CCAO-F, CCAR-F, CCAR-P, CCDV-F), delivered proctored via Pearson VUE and registered through the Anthropic Partner Academy. Exam Guide v1.0 (effective July 2026) sets a $125 exam fee and 12-month certification validity for CCAR-F, superseding the launch-period "$99 / first 5,000 partner employees free" terms.

## Credits

**Repository owner:** Muhammed Nemathullah Khan — maintains this repo's domain deep-dives, scenario walkthroughs, and mock-test/wrong-answer tracking.

**Author of the domain deep-dive guides (`scenarios/domain-1.md` … `domain-5.md`):** [Arun Varadharajalu](https://www.linkedin.com/in/arunv11u/)

**Additional sources referenced in the domain guides:**
- [guide_en.md — core topics & fundamentals overview](https://github.com/paullarionov/claude-certified-architect/blob/main/guide_en.md) *(that guide itself credits its exam-domain breakdown to a compilation by **@hooeem** on X)*
- [Claude Certification Guide — Mock Exam](https://claudecertificationguide.com/mock-exam)
- [CyberSkill Practice — CCAF](https://practice.cyberskill.world/practice/ccaf/exam)
- [CertSafari — CCAF](https://www.certsafari.com/anthropic/claude-certified-architect-foundations)

## License

This is an unofficial community study guide. Claude Certified Architect is a certification program by Anthropic.
