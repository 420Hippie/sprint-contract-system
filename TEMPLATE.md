# Sprint Contract: {Sprint Name}

**Date:** YYYY-MM-DD
**Builder (Generator):** {Name / agent}
**Evaluator:** {Name / agent — must NOT be the Builder}
**Planner:** {Name / agent}

---

## Objective

One or two sentences. What does this sprint deliver? Plain language, no jargon.

---

## Success Criteria

Each criterion MUST be independently testable. No subjective language ("works well", "looks good", "is fast"). Every criterion is PASS or FAIL.

| # | Criterion | How to Test | Pass Condition | Status |
|---|-----------|-------------|----------------|--------|
| 1 | {Specific, testable claim} | {Exact steps to verify — commands to run, files to check, behaviors to observe} | {What PASS looks like — measurable} | PENDING |
| 2 | | | | PENDING |
| 3 | | | | PENDING |

**Rules for writing criteria:**
- "Renders without errors" → testable: open page, check console
- "Looks clean" → ❌ NOT testable
- "Returns 200 status code in <100ms p95 over 1000 requests" → testable
- "Is fast" → ❌ NOT testable
- "All 10 unit tests pass" → testable: `npm test`
- "Works correctly" → ❌ NOT testable

If you cannot describe the test method, you do not understand the requirement well enough to build it. Stop and clarify before signing.

---

## Out of Scope

What this sprint deliberately does NOT deliver. List anything that might tempt scope creep — features adjacent to this work, refactors discovered along the way, "while we're in there" additions.

- Item 1
- Item 2

If new requirements surface mid-sprint, they go into a **contract amendment** (separate section below), not a silent addition.

---

## Dependencies

What must exist or be done before this sprint can start.

- Dependency 1
- Dependency 2

If any dependency is unmet, the sprint cannot begin. Don't sign a contract whose dependencies haven't been verified.

---

## Rollback Plan

If this sprint fails or has to be reverted, what's the procedure?

- Revert command(s)
- Manual cleanup steps
- State to return to

A sprint without a rollback plan is a sprint that can break production with no exit.

---

## Signatures

- **Builder:** {Name} — AGREED YYYY-MM-DD
- **Evaluator:** {Name} — AGREED YYYY-MM-DD
- **Planner:** {Name} — GATE 0 CLEARED YYYY-MM-DD

The contract is binding once all three sign. No mid-sprint silent changes.

---

## Contract Amendments

Track any in-flight changes to the contract here. Each amendment requires re-signature.

### Amendment 1 — YYYY-MM-DD
**Trigger:** {Why this came up mid-sprint}
**Change:** {Exact change to criteria / scope}
**Re-signed:** Builder ✓ | Evaluator ✓ | Planner ✓

---

## Post-sprint

After all criteria are evaluated, fill in:

### Builder self-evaluation (Gate 1)
| # | Criterion | Result | Evidence |
|---|-----------|--------|----------|
| 1 | | PASS / FAIL | {commit, screenshot, command output} |

### Evaluator independent evaluation (Gate 2)
| # | Criterion | Result | Evidence |
|---|-----------|--------|----------|
| 1 | | PASS / FAIL | {what was tested, what was observed} |

### Final status
- ☐ All criteria PASS — sprint complete
- ☐ Some criteria FAIL — sprint not complete, return to Builder
