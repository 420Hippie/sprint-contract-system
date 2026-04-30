# Sprint Contract: Operator Recorder UI v1

**Date:** 2026-04-08
**Builder:** Frontend specialist
**Evaluator:** QA + Visual reviewer
**Planner:** Project lead

> **Real-world example.** Frontend feature contract. The "How to Test" column does the heavy lifting here — without specific viewport sizes and behavior assertions, "looks clean" is the kind of language that ships bugs.

---

## Objective

Ship a recorder UI page at `/admin/recorder` that lets a session operator tag every event with category + slot in real time during a session, with visual feedback and undo support.

---

## Success Criteria

| # | Criterion | How to Test | Pass Condition | Status |
|---|-----------|-------------|----------------|--------|
| 1 | Page renders without console errors at 1920×1080 + 768×1024 + 390×844 viewports | Open `/admin/recorder` in Chrome at each viewport, open DevTools console | Zero errors, zero warnings, no layout overflow at any viewport | PENDING |
| 2 | Tap-to-tag an event records it in <100ms perceived latency | Click any category+slot button, observe DOM update | Visual feedback (ripple + count increment) within 100ms; event POST'd to backend within 500ms | PENDING |
| 3 | Undo last event works | Tag an event, click undo within 5 seconds | Event removed from local state and DELETE'd from server; count decrements | PENDING |
| 4 | Live event count syncs across two browser tabs | Open recorder in two tabs, tag an event in tab A | Tab B shows new count within 1 second (realtime channel) | PENDING |
| 5 | Keyboard shortcuts work (1-4 = category, A/B = slot, U = undo) | Use only keyboard, complete 10 events | All 10 events tagged via keyboard, undo works via U | PENDING |
| 6 | No crash on offline mode | Disconnect network, tag 3 events, reconnect | All 3 events sync once network returns; no UI freeze | PENDING |

---

## Out of Scope

- Multi-session management
- User authentication (assumes single-user mode for now)
- Stats/analytics views (separate sprint)
- Mobile gestures (swipe-to-undo, etc.)

---

## Dependencies

- Backend configured with `events` table per latest schema migration
- Operator dashboard framework in place
- Authenticated dev environment with feature-flag `RECORDER_V1` enabled

---

## Rollback Plan

```bash
git revert <recorder-commits>
# Disable feature flag
echo 'RECORDER_V1=false' >> .env.local
npm run dev
```

The `/admin/recorder` route returns 404 in this state, which is the prior behavior.

---

## Signatures

- **Builder:** Frontend specialist — AGREED 2026-04-08
- **Evaluator:** QA + Visual reviewer — AGREED 2026-04-08
- **Planner:** Project lead — GATE 0 CLEARED 2026-04-08

---

## Post-sprint

### Builder self-evaluation (Gate 1)
| # | Criterion | Result | Evidence |
|---|-----------|--------|----------|
| 1 | Console errors at 3 viewports | PASS | Screenshots at all 3 viewports, console clean |
| 2 | Tap-to-tag <100ms | PASS | Lighthouse perf trace, tap-to-paint = 38ms |
| 3 | Undo works | PASS | Manual test, plus unit test in `tests/recorder.spec.ts` |
| 4 | Cross-tab sync | PASS | Two-tab test, sync time ~600ms via realtime channel |
| 5 | Keyboard shortcuts | PASS | All 10 events via keyboard, evidence captured |
| 6 | Offline mode | PASS | Plane mode test, events queued in IndexedDB, synced on reconnect |

### Evaluator independent evaluation (Gate 2)
| # | Criterion | Result | Evidence |
|---|-----------|--------|----------|
| 1 | Console errors | PASS | Independent test on dev rig, all 3 viewports clean |
| 2 | Tap-to-tag <100ms | PASS | Subjectively snappy; perf trace verified |
| 3 | Undo works | PASS | Tested 5 sequential undo operations, no inconsistency |
| 4 | Cross-tab sync | PASS | Tested across 3 tabs (not 2!) — all synced |
| 5 | Keyboard shortcuts | INITIAL FAIL → FIXED | "U" key was being eaten by browser undo on focused input. Visual reviewer pushed back; Builder added `e.preventDefault()`. Re-test passed. |
| 6 | Offline mode | PASS | Independent plane-mode test |

### Final status
- ☑ All criteria PASS after one fix loop — sprint complete

---

## Lesson captured

Criterion 5 (keyboard shortcuts) caught a real bug that would have shipped: the U-for-undo binding conflicted with the browser's built-in Cmd+Z behavior on focused inputs. The Evaluator's independent test caught it because they tested with focus *inside* a text input first — the Builder had only tested with focus on the page body. **The "test method" text in the contract didn't specify focus state**, which is why the Builder's self-eval missed it.

Lesson for future contracts: when testing keyboard interactions, the test method should specify focus state explicitly. Updated TEMPLATE.md to include this guidance.
