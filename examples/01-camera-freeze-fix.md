# Sprint Contract: Live Stream Reliability Fix (P0)

**Date:** 2026-03-26
**Builder:** Pipeline specialist
**Evaluator:** QA + Performance reviewer
**Planner:** Project lead

> **Real-world example.** A streaming-video pipeline was freezing every 5–15 minutes and blocking a stakeholder demo. The contract structure forced the team to commit to "30-minute zero-freeze" as the bar — not "seems fixed."

---

## Objective

Resolve the streaming freeze that occurs after 5–15 minutes on the primary feed, blocking the upcoming demo.

---

## Success Criteria

| # | Criterion | How to Test | Pass Condition | Status |
|---|-----------|-------------|----------------|--------|
| 1 | Stream runs without freeze for 30+ minutes | Start inference server, monitor feed for 30 min | Zero freezes in 30-minute continuous run | PENDING |
| 2 | Sub-stream fallback works | Connect to lower-resolution backup at 640x480 | Stream establishes and delivers frames at ≥15fps | PENDING |
| 3 | Automatic reconnection on stream drop | Kill connection mid-stream via network simulation | Server detects drop and reconnects within 5 seconds | PENDING |
| 4 | No regression in detection accuracy | Run 10 ground-truth-labeled test events on reconnected stream | All 10 events detected correctly | PENDING |

---

## Out of Scope

- Secondary-feed configuration (separate sprint)
- Multi-channel support
- Voice-command fixes
- UI changes to surface stream status

---

## Dependencies

- Source camera powered on and reachable
- Production-baseline detection model available
- Ground-truth labels for criterion 4 staged at `tests/fixtures/`

---

## Rollback Plan

Revert to the prior-version stream with manual restart protocol (current state):
```bash
git revert <fix-commit>
sudo systemctl restart inference-pipeline
```

Restart cadence will return to every 5–15 min until next fix attempt.

---

## Signatures

- **Builder:** Pipeline specialist — AGREED 2026-03-26
- **Evaluator:** QA + Performance reviewer — AGREED 2026-03-26
- **Planner:** Project lead — GATE 0 CLEARED 2026-03-26

---

## Post-sprint

### Builder self-evaluation (Gate 1)
| # | Criterion | Result | Evidence |
|---|-----------|--------|----------|
| 1 | 30+ min no freeze | PASS | 47-minute uninterrupted run logged |
| 2 | Sub-stream fallback | PASS | Verified at backup resolution, 18fps measured |
| 3 | Auto-reconnect <5s | PASS | Network-kill test → reconnected in 2.3s |
| 4 | No accuracy regression | PASS | 10/10 events detected, all classifications correct |

### Evaluator independent evaluation (Gate 2)
| # | Criterion | Result | Evidence |
|---|-----------|--------|----------|
| 1 | 30+ min no freeze | PASS | Independent 35-min run, stream stable |
| 2 | Sub-stream fallback | PASS | Tested at 5pm, 9pm, 11pm — all established |
| 3 | Auto-reconnect <5s | PASS | 3 separate kill tests: 2.1s, 2.7s, 3.0s reconnect |
| 4 | No accuracy regression | PASS | Same 10 events, plus 5 additional adversarial cases — all detected |

### Final status
- ☑ All criteria PASS — sprint complete
- Merged to `main`

---

## Lesson captured

The original "fix" before this contract was a `try/except` wrapper that caught the stream timeout but didn't actually reconnect — the stream silently died, just without an error message. **Criterion 3 (auto-reconnect within 5s)** caught this. Without that explicit testable criterion, "the freezes don't crash anymore" would have looked like success.

This is what generation/evaluation separation buys you: the Builder's narrative ("the freezes are caught") is forced to map to a testable claim ("auto-reconnect within 5s"). The narrative was technically true. The criterion was false. The contract surfaced the gap.
