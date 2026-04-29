# Sprint Contract System

> A pre-work contract system for AI-assisted software development. Co-signed by builder and evaluator before any code is written. Eliminates "close enough" shipping. Adapted from [Anthropic's Harness 2.0](https://www.anthropic.com/research) and battle-tested across 48+ sprints in real production work.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

## What this is

When you ask a coding agent (Claude Code, Codex, Cursor's Composer, etc.) to ship a feature, the agent will reliably:

1. Build something that *resembles* the requirement
2. Self-evaluate it as "complete"
3. Hand it back saying "done"

You then look at it and find it's 70% there with three subtle bugs and a missing edge case.

This isn't an agent failure — it's a **structural failure of letting the same agent that wrote the code grade it.** Anthropic's research is explicit: AI agents are poor critics of their own output. They reliably over-praise their own work.

The fix isn't a better agent. The fix is **separating generation from evaluation** with a binding contract written *before* work starts.

## How it works

Every sprint produces a **Sprint Contract** — a markdown file with:

- **Objective** — one sentence
- **Success Criteria** — a table of testable, binary PASS/FAIL conditions, each with an exact test method
- **Out of Scope** — what this sprint deliberately does NOT do
- **Dependencies** — what must exist before work can start
- **Rollback Plan** — what to do if it fails
- **Signatures** — Builder, Evaluator, Planner

The contract is **co-signed before any code is written**. The Builder builds against the contract. The Evaluator independently tests every criterion. There is no "close enough."

## Why this works

| Without contracts | With contracts |
|---|---|
| "App works correctly" → meaningless QA | "Navigation bar renders with logo, 3 menu items, and responsive collapse at 768px" → testable |
| Builder declares done → ships bugs | Builder *and* Evaluator must agree → bugs caught pre-merge |
| Scope creep silently expands | New work requires contract amendment → visible |
| Disagreements about what's done | Binary PASS/FAIL on each criterion → no debate |
| Agent self-praise rewarded | Agent has nowhere to hide vague success → quality forced up |

## What's in this repo

```
sprint-contract-system/
├── README.md                 ← you are here
├── LICENSE                   ← MIT
├── TEMPLATE.md               ← copy this for every new sprint
├── essay.md                  ← the long-form essay: why contracts work
└── examples/
    ├── 01-camera-freeze-fix.md         ← real-world bug fix sprint
    ├── 02-cv-model-validation.md       ← ML model evaluation sprint
    ├── 03-frontend-component.md        ← shipping a UI component
    ├── 04-data-pipeline.md             ← backend pipeline sprint
    ├── 05-research-notebook.md         ← research sprint (notebook contract variant)
    └── 06-multi-session-sprint.md      ← spanning 3+ sessions w/ progress file
```

## Quick start

1. Copy `TEMPLATE.md` to `docs/sprint-contracts/YYYY-MM-DD-<sprint-name>.md`
2. Fill in Objective, Success Criteria (with test methods), Out of Scope
3. Tag your AI evaluator (or human QA) to push back on vague criteria
4. Iterate until both sides agree on every criterion
5. Sign the contract
6. Build against it
7. Self-evaluate (Builder runs every test)
8. Independent evaluation (Evaluator runs every test)
9. Both pass → sprint is done. Any fail → sprint is not done.

## How this fits with Claude Code / Codex / etc.

The contract lives as a markdown file in your repo. You point your AI builder at it: *"Build against `docs/sprint-contracts/2026-04-27-feature-x.md`. Do not exceed scope. Self-evaluate before declaring done."* The contract becomes part of the agent's context.

For multi-session sprints, the contract is paired with a `sprint-progress.md` file that gets read at the start of each session and updated at the end. Same pattern Anthropic uses internally.

## Lineage

This system is a working operator's adaptation of:

- **Anthropic's Harness 2.0 research** (2025) — generation/evaluation separation, planner/generator/evaluator triad
- **Pre-merge code review discipline** (open source culture, decades old)
- **Acceptance test-driven development** (Lasse Koskela, ~2007)
- **Pre-registered hypotheses** (replication-crisis-era science reform)

The *AI-specific* part is structural: AI agents will praise their own work unless you put a different agent (or human) in the evaluator chair, and unless the criteria are testable enough that flattery doesn't move the needle.

## Status

- **Adopted in production:** 2026-03-26
- **Contracts shipped:** 48+
- **Failure modes prevented (documented):** scope creep, vague success criteria, self-evaluator over-praise, mid-sprint silent additions
- **License:** MIT

## What's next

Issues and PRs welcome. If you adopt this and have feedback — what worked, what didn't, what you'd change for your domain — open an issue. The README will evolve.

## Author

Built by Michael Brenneman (TruPath Ventures) while running three ventures on Claude Code. Part of [TruPath Labs](https://trupathventures.net/labs).
