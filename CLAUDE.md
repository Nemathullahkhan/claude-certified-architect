# CLAUDE.md

Orientation for Claude Code sessions working in this repo.

## What this repo is

A personal study repo for the **Claude Certified Architect – Foundations (CCA-F)** exam. It is almost entirely markdown study material — there is no application code to build or run yet. See `README.md` for the human-facing study-path explanation.

## Directory map

```
.
├── README.md                  # Study-path guide (start here)
├── exam-guide.pdf              # Official Anthropic exam guide
├── Grep-vs-Glob.MD             # Quick reference: Grep vs Glob semantics
├── package.json / tsconfig.json  # Hands-on practice scaffold (see caveat below)
├── theory/
│   └── theory.md              # 13-chapter theory foundation (API, tool_use, Agent SDK, MCP, Claude Code, prompt engineering, batches, escalation, error handling, context management, provenance, built-in tools)
├── domains/
│   ├── domains.md              # Raw Anthropic task-statement text (knowledge/skills) for Domains 1–5 — source material the domain-N.md guides are built from
│   └── domain-1.md .. domain-5.md  # Deep-dive exam-prep guide per official domain (weight, mental models, anti-patterns, 20+ MCQs + answer keys each)
├── scenarios/
│   ├── listedScenarios.md      # The 6 official exam scenarios (4 of 6 appear on any given exam)
│   └── scenario-1..6-*.md       # One expanded walkthrough per scenario, with practice questions + cheat sheet
└── mock-tests/
    ├── customer-support-agent/ multi-agent-research/ cicd/ code-generation/   # Practice exams grouped by scenario
    ├── wrong-answers/            # Miss log: wrong answer + why + rule to remember (not just correct answers)
    └── report/                   # Score-trend summaries per scenario across attempts
```

## Caveat: the scaffold is not implemented

`package.json` defines two scripts that do not currently work:
- `task1` → `tsx domain-1/tasks/1/agenticLoop.ts`
- `test:model` → `tsx model/test-model.ts`

Neither `domain-1/` nor `model/` exists at the repo root (`domains/domain-1.md` is a markdown guide, not that directory). `tsconfig.json`'s `include` (`src`, `model`) also points at directories that don't exist yet. Treat these as placeholders for future hands-on practice code, not working scripts — don't assume they run without first creating the referenced files.

## Working in this repo

- The `domain-N.md` and `scenario-N-*.md` files are large (700–3300 lines). Use Grep to locate a section/topic before reading the whole file.
- `mock-tests/wrong-answers/*.md` files reference a `decision-framework.md` that does not exist yet — don't assume it's present.
