# Why your AI agents praise their own bad code (and the fix)

> The framing essay for Sprint Contract System. Standalone read for operators who want to understand the *why* before they install the *what*.

---

There's a structural problem with letting an AI agent both write code and decide whether it's done.

Anthropic's research on long-running agentic work (Harness 2.0) named it directly: AI agents are **poor critics of their own output.** Asked to evaluate work they produced, they reliably over-praise it. Even when the quality is mediocre to a human observer, the agent narrates success.

This is not a model limitation that better training will fix. It's a structural problem with self-assessment under generative objectives. The same pattern shows up in humans — graduate-school exit interviews include external committees for the same reason. We don't trust the candidate to grade the dissertation.

Most operators using Claude Code, Cursor, or Codex run into this within their first month. The agent says "done." You look. It's 70% done with three subtle bugs and a missing edge case. You ask it to fix. It says "done." You look. It's now 85% done with two of the original bugs gone and one new one introduced. You spend the rest of the day in this loop.

The fix isn't a smarter agent. The fix is a structural separation that pre-commits both sides to what "done" means before any code is written.

## What a Sprint Contract is

A Sprint Contract is a markdown file. It has six sections:

1. **Objective** — one or two sentences
2. **Success Criteria** — a table of testable, binary PASS/FAIL conditions, each with an exact test method
3. **Out of Scope** — what this sprint deliberately does NOT do
4. **Dependencies** — what must exist before work starts
5. **Rollback Plan** — what to do if it fails
6. **Signatures** — Builder, Evaluator, Planner

Three things matter about how the contract is used:

**The contract is signed before any code is written.** Not after. Before. Both Builder and Evaluator must agree on every criterion. If the Evaluator pushes back on a vague criterion, the contract isn't signed until the criterion is rewritten in testable terms.

**The Builder and the Evaluator are not the same agent.** This is the structural rule. The Builder builds against the contract. The Evaluator independently verifies every criterion. There is no single agent that decides "done."

**Every criterion is binary PASS/FAIL.** No "close enough." No "good enough for a v1." If criterion 3 fails, the sprint is not complete, regardless of how good criteria 1, 2, 4, and 5 look.

That's it. The system is small. The discipline is what makes it work.

## Why this is different from "writing tests first"

Test-driven development is closely related but solves a different problem. TDD is about *implementation correctness* — the test asserts that the code does what the test says.

Sprint Contracts are about *requirement correctness* — the criterion asserts what we agreed the code should do. The criterion can have a test, sure. But the criterion can also describe behaviors that aren't unit-testable — UI rendering at specific viewports, latency under load, no-regressions on a manual eval set.

A test asks "does this function return the right value?" A criterion asks "did we ship the thing we agreed to ship?"

You can have green tests and a failed sprint. (You shipped something that passes its tests but isn't what was wanted.) You can have a passed sprint with no automated tests. (Manual eval against the criterion was the test.)

In practice, well-written criteria become well-written tests. But the criteria come first.

## The mechanical part: what makes criteria testable

Most teams adopting this fail at the criteria-writing step. Their first contract has a row that says:

> *Criterion 1: Authentication works correctly.*

That's a wish, not a criterion. There's nothing to test. Both Builder and Evaluator could honestly believe this passes while disagreeing about what "correctly" means.

Compare:

> *Criterion 1: User can log in with email + password, log out, and the session token expires after 7 days of inactivity. **How to test:** Run `tests/auth.spec.ts` (5 tests covering login, logout, token refresh, expiry, invalid credentials). All 5 must pass.*

The second criterion is a contract. The first is a vibe.

The Evaluator's job during contract negotiation is to push back on vibes until they become contracts. This is uncomfortable. It feels like the Evaluator is being pedantic. The Evaluator is supposed to be uncomfortable, and the discomfort is the point. Every vague criterion that ships unmodified is a future bug.

A useful test: if the Builder and Evaluator disagree about whether the work passes the criterion, the criterion is bad. Rewrite it.

## Why contracts work better than principles

Most teams try to fix the "agent over-praises its own work" problem with principles. *"Agents must be honest." "Agents must verify before claiming done." "Agents must run tests."*

Principles fail because they're aspirational and unenforced. The agent says "I have verified this is complete" because that's what a finished response looks like. The principle has no teeth.

A contract has teeth because the criteria are *external to the agent that built the work*. The Builder agent cannot make criterion 3 pass by saying "criterion 3 passes." Some other agent (or person) must independently verify it. The principle of honesty is replaced by the structural requirement that two different evaluators must agree.

This is the same reason peer review exists in science, code review exists in software, and clinical trials require independent review boards. In every domain where self-assessment is unreliable, the solution is structural separation, not better promises from the assessor.

## Contracts in multi-session work

A sprint that completes in one work session needs only the contract. A sprint that spans multiple sessions needs a paired progress file:

```
sprints/
├── 2026-04-12-feature-x.md           ← the contract
└── progress/
    └── 2026-04-12-feature-x.md       ← the progress file
```

The progress file is updated at the end of every session. It has four sections per session entry:

1. What got completed this session
2. Status of every contract criterion
3. What the next session should do first
4. Known issues and git state

The progress file is read at the start of every session, before any other work. The first thing the Builder does in session 2 is open the progress file from session 1, recover context, and work from there.

Without progress files, multi-session sprints lose 30-50% of their time to re-orientation. With them, the cost is a 5-minute write at the end of each session and a 5-minute read at the start of the next.

## What you give up

This system has costs.

**Contracts are slow to write.** A real contract — with testable criteria, every "How to Test" filled in, two-sided agreement on every row — takes 30-60 minutes for a non-trivial sprint. For a small sprint it's overhead.

**Negotiation has friction.** The first time the Evaluator pushes back on the Builder's draft criterion, both sides will be annoyed. Drafting a clean criterion that the Evaluator co-signs takes effort. This is the right amount of friction for shippable work, and the wrong amount for throwaway scripts.

**Some sprints don't need this.** Bug fixes that take 10 minutes don't need a contract. Exploratory spikes don't need a contract. Don't bring this hammer to every nail.

The right rule: **if the sprint is going to ship to users, customers, or production data, write a contract. If the sprint is going to inform your next decision, just write a question and an answer.**

## What you get

In the operator's seat — running multiple ventures or projects on Claude Code — Sprint Contracts are the difference between "what did we actually ship this week" being clear vs. fuzzy.

A few specific things they buy you:

- **No more "close enough" arguments.** PASS/FAIL is binary. Either criterion 3 passes or it doesn't. The sprint isn't done until every criterion passes.
- **Mid-sprint scope is visible.** When you discover something important mid-sprint, it goes into a contract amendment. The amendment requires re-signature. You can't silently expand scope.
- **Clear handoffs across sessions and agents.** The contract is the source of truth. Anyone walking into the sprint mid-flight reads the contract and the progress file and knows where to pick up.
- **Postmortem extraction is mechanical.** When a sprint fails, you can point at exactly which criterion failed and why. Lessons get captured in the right place.

## Where this came from

Sprint Contracts are an operator-shop adaptation of Anthropic's Harness 2.0 research. Anthropic uses planner/generator/evaluator separation internally and across their long-running agentic workloads. The "agents praise their own work" finding is reproducible and is one of the strongest signals in agentic AI research from the last two years.

We're not the first to apply it outside Anthropic, and we won't be the last. Cursor's Composer mode has elements of it (the build-then-review loop). Codex shops have started writing similar pre-work specs. The naming and the specific structure here come from running this system across three real ventures (a sports tech startup, a hospitality venue, a research lab) for several months and finding what survived contact with reality.

## The bundle

The repo at `github.com/trupath-labs/sprint-contract-system` (publication pending) contains:

- The contract template
- Six worked examples drawn from real production sprints
- This essay
- An MIT license

Drop the template into your repo at `docs/sprint-contracts/` and start. The first three contracts you write will be slower and uglier than your eventual workflow. The system gets faster as your team — agent or human or both — develops the negotiation muscle.

After about a month of consistent use, contracts stop feeling like overhead and start feeling like the only sane way to ship.

— Michael, from the lab
