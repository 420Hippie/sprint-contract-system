# Sprint Contract: Event Pipeline — End-to-End

**Date:** 2026-04-12
**Builder:** Backend specialist
**Evaluator:** QA + Integration reviewer
**Planner:** Project lead

> **Real-world example.** Backend pipeline from event capture → validation → enrichment → persistence → emission. Multi-component contracts surface integration gaps that single-component contracts miss.

---

## Objective

Implement the event pipeline: detector emits raw event → validator drops malformed → enricher adds session/team context → persister writes to database → emitter publishes to dashboard websocket. End-to-end latency budget 250ms p95.

---

## Success Criteria

| # | Criterion | How to Test | Pass Condition | Status |
|---|-----------|-------------|----------------|--------|
| 1 | Detector emits raw events to validator | `tail -f logs/detector.log` while triggering test events | At least one `event_emitted` log line per detected event | PENDING |
| 2 | Validator drops events missing required fields | Send 10 events to validator with deliberate omissions (test fixture: `tests/malformed-events.json`) | Validator logs reject reason for each, none reach enricher | PENDING |
| 3 | Enricher adds session_id, team_id, event_index | Inspect persister input via test mode | All persisted events have all three fields populated | PENDING |
| 4 | Persister writes within authorization scope | Attempt to write event with mismatched session_id | Database rejects with 403, error logged, no orphan rows | PENDING |
| 5 | Emitter publishes to dashboard within 250ms p95 of detector emit | Run 100 sequential events, time detector_emit_ts → dashboard_receive_ts | p95 latency ≤ 250ms over 100 events | PENDING |
| 6 | At-least-once delivery: kill emitter mid-stream, recover, no events lost | Send 50 events, kill emitter at event 25, restart | All 50 events present in dashboard subscription stream after recovery | PENDING |
| 7 | No duplicate persistence on retries | Send same event twice (network retry simulation) | Persister deduplicates by `event_id`, only one row in DB | PENDING |

---

## Out of Scope

- Schema migrations (assumes table is in place)
- Dashboard UI rendering (separate sprint)
- Multi-session concurrent handling
- Historical replay/backfill

---

## Dependencies

- Database migrations applied
- Detector emitting to local stream `event-stream:raw`
- Dashboard subscribed to channel `events:session_<id>:events`
- Test fixtures available at `tests/malformed-events.json` and `tests/timing-fixtures.json`

---

## Rollback Plan

Pipeline components are independently deployed:

```bash
# Roll back validator
git revert <validator-commit> && pm2 restart validator

# Roll back enricher
git revert <enricher-commit> && pm2 restart enricher

# Etc. for each component
```

If full pipeline rollback needed, restore from `tag/pipeline-pre-sprint-2026-04-12`.

---

## Signatures

- **Builder:** Backend specialist — AGREED 2026-04-12
- **Evaluator:** QA + Integration reviewer — AGREED 2026-04-12
- **Planner:** Project lead — GATE 0 CLEARED 2026-04-12

---

## Post-sprint

### Builder self-evaluation (Gate 1)
| # | Criterion | Result | Evidence |
|---|-----------|--------|----------|
| 1 | Detector → validator | PASS | 30 events, all logged |
| 2 | Validator drops malformed | PASS | 10/10 fixtures rejected with correct reason |
| 3 | Enricher adds context | PASS | All persisted rows show populated fields |
| 4 | Authorization enforced | PASS | Manual test, mismatched session got 403 |
| 5 | p95 latency ≤ 250ms | PASS | 100 events, p95 = 187ms |
| 6 | At-least-once recovery | PASS | Kill test passed, 50/50 events recovered |
| 7 | No duplicates | PASS | Retry test, single row in DB |

### Evaluator independent evaluation (Gate 2)
| # | Criterion | Result | Evidence |
|---|-----------|--------|----------|
| 1 | Detector → validator | PASS | Confirmed independently |
| 2 | Validator drops malformed | INITIAL FAIL → FIXED | Found edge case: empty string vs null treated differently. Fixture #11 added (`event_id: ""`). Validator updated to reject. |
| 3 | Enricher adds context | PASS | Verified with two parallel sessions, no cross-contamination |
| 4 | Authorization enforced | PASS | Independent attempt, 403 confirmed |
| 5 | p95 latency ≤ 250ms | PASS | Independent run, p95 = 195ms |
| 6 | At-least-once recovery | PASS | Independent kill test, 50/50 recovered |
| 7 | No duplicates | PASS | Confirmed |

### Final status
- ☑ All criteria PASS after one fix loop — sprint complete

---

## Lesson captured

Criterion 2 caught an edge case the Builder's self-eval missed: empty string `""` vs null. The Builder's fixture set treated all malformed events as containing `null` for missing fields, but the actual upstream emitter sometimes sent `""`. The Evaluator added fixture #11 during independent testing.

Updated the test fixture standard going forward: malformed-event fixtures must cover empty string, null, undefined (in JS contexts), and missing-key cases. **This is now an Evaluator checklist item for any data-pipeline contract.**
