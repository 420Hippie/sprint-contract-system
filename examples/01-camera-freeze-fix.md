# Sprint Contract: Camera Freeze Fix (P0)

**Date:** 2026-03-26
**Builder:** Forge (CV pipeline director)
**Evaluator:** Beacon (QA) + Probe (performance specialist)
**Planner:** Helm

> **Real-world example.** This is a verbatim contract from a production CV pipeline. RTSP stream was freezing every 5-15 minutes, blocking a Phase 0 demo. The contract structure forced the team to commit to "30-minute zero-freeze" as the bar — not "seems fixed."

---

## Objective

Resolve the board camera RTSP freeze that occurs after 5-15 minutes on the main stream, blocking the Phase 0 demo.

---

## Success Criteria

| # | Criterion | How to Test | Pass Condition | Status |
|---|-----------|-------------|----------------|--------|
| 1 | Camera stream runs without freeze for 30+ minutes | Start inference server, monitor RTSP feed for 30 min | Zero freezes in 30-minute continuous run | PENDING |
| 2 | Sub-stream fallback works | Connect to `h264Preview_01_sub` at 640x480 | Stream establishes and delivers frames at ≥15fps | PENDING |
| 3 | Automatic reconnection on stream drop | Kill RTSP connection mid-stream via `tcpkill` | Server detects drop and reconnects within 5 seconds | PENDING |
| 4 | No regression in detection accuracy | Run 10 test throws on reconnected stream | All 10 throws detected correctly (ground truth labeled) | PENDING |

---

## Out of Scope

- Front camera ROI configuration (separate sprint)
- Multi-lane support
- Voice command fixes
- UI changes to surface stream status

---

## Dependencies

- Board camera powered on and accessible at 192.168.2.3
- Reolink firmware at version 4.x or higher
- v10 detection model available at `qcaddy-edge/models/v10/`
- Ground-truth throw labels for criterion 4 (located at `tests/fixtures/throws-2026-03-25.json`)

---

## Rollback Plan

Revert to main stream with manual restart protocol (current state):
```bash
git revert <fix-commit>
sudo systemctl restart qcaddy-inference
```

Restart cadence will return to every 5-15 min until next fix attempt.

---

## Signatures

- **Builder:** Forge — AGREED 2026-03-26
- **Evaluator:** Beacon + Probe — AGREED 2026-03-26
- **Planner:** Helm — GATE 0 CLEARED 2026-03-26

---

## Post-sprint

### Builder self-evaluation (Gate 1)
| # | Criterion | Result | Evidence |
|---|-----------|--------|----------|
| 1 | 30+ min no freeze | PASS | 47-minute uninterrupted run logged at `runs/2026-03-26-stress-test.log` |
| 2 | Sub-stream fallback | PASS | Verified at 640x480, 18fps measured |
| 3 | Auto-reconnect <5s | PASS | tcpkill test → reconnected in 2.3s |
| 4 | No accuracy regression | PASS | 10/10 throws detected, all bag classifications correct |

### Evaluator independent evaluation (Gate 2)
| # | Criterion | Result | Evidence |
|---|-----------|--------|----------|
| 1 | 30+ min no freeze | PASS | Independent 35-min run, stream stable |
| 2 | Sub-stream fallback | PASS | Tested at 5pm, 9pm, 11pm — all established |
| 3 | Auto-reconnect <5s | PASS | 3 separate kill tests: 2.1s, 2.7s, 3.0s reconnect |
| 4 | No accuracy regression | PASS | Same 10 throws, plus 5 additional adversarial throws — all detected |

### Final status
- ☑ All criteria PASS — sprint complete
- Merged to `main` at commit `b82df41`

---

## Lesson captured

The original "fix" before this contract was a `try/except` wrapper that caught the RTSP timeout but didn't actually reconnect — stream silently died, just without an error message. **Criterion 3 (auto-reconnect within 5s)** caught this. Without that explicit testable criterion, "the freezes don't crash anymore" would have looked like success.

This is what generation/evaluation separation buys you: the Builder's narrative ("the freezes are caught") is forced to map to a testable claim ("auto-reconnect within 5s"). The narrative was technically true. The criterion was false. The contract surfaced the gap.
