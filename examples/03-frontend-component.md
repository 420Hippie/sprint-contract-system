# Sprint Contract: Mission Control Recorder UI v1

**Date:** 2026-04-08
**Builder:** Prism (frontend director)
**Evaluator:** Beacon (QA) + Pixel (visual specialist)
**Planner:** Helm

> **Real-world example.** Frontend feature contract. The "How to Test" column does the heavy lifting here — without specific viewport sizes and behavior assertions, "looks clean" is the kind of language that ships bugs.

---

## Objective

Ship a recorder UI page at `/mission-control/recorder` that lets a session operator tag every cornhole throw with sign + slot in real time during gameplay, with visual feedback and undo support.

---

## Success Criteria

| # | Criterion | How to Test | Pass Condition | Status |
|---|-----------|-------------|----------------|--------|
| 1 | Page renders without console errors at 1920×1080 + 768×1024 + 390×844 viewports | Open `/mission-control/recorder` in Chrome at each viewport, open DevTools console | Zero errors, zero warnings, no layout overflow at any viewport | PENDING |
| 2 | Tap-to-tag a throw records the event in <100ms perceived latency | Click any sign+slot button, observe DOM update | Visual feedback (ripple + count increment) within 100ms; event POST'd to `/api/throws` within 500ms | PENDING |
| 3 | Undo last throw works | Tag a throw, click undo within 5 seconds | Throw removed from local state and DELETE'd from server; count decrements | PENDING |
| 4 | Live throw count syncs across two browser tabs | Open recorder in two tabs, tag a throw in tab A | Tab B shows new count within 1 second (Supabase realtime) | PENDING |
| 5 | Keyboard shortcuts work (1-4 = sign, A/B = slot, U = undo) | Use only keyboard, complete 10 throws | All 10 throws tagged via keyboard, undo works via U | PENDING |
| 6 | No crash on offline mode | Disconnect network, tag 3 throws, reconnect | All 3 throws sync once network returns; no UI freeze | PENDING |

---

## Out of Scope

- Multi-game session management
- User authentication (assumes single-user mode for now)
- Stats/analytics views (separate sprint)
- Mobile gestures (swipe-to-undo, etc.)

---

## Dependencies

- Supabase project configured with `throws` table per migration `0042_throws_schema.sql`
- Mission Control framework in place (`/mission-control` route exists)
- Authenticated dev environment with feature-flag `RECORDER_V1` enabled

---

## Rollback Plan

```bash
git revert <recorder-commits>
# Disable feature flag
echo 'RECORDER_V1=false' >> .env.local
npm run dev
```

The `/mission-control/recorder` route returns 404 in this state, which is the prior behavior.

---

## Signatures

- **Builder:** Prism — AGREED 2026-04-08
- **Evaluator:** Beacon + Pixel — AGREED 2026-04-08
- **Planner:** Helm — GATE 0 CLEARED 2026-04-08

---

## Post-sprint

### Builder self-evaluation (Gate 1)
| # | Criterion | Result | Evidence |
|---|-----------|--------|----------|
| 1 | Console errors at 3 viewports | PASS | Screenshots at all 3 viewports, console clean |
| 2 | Tap-to-tag <100ms | PASS | Lighthouse perf trace, tap-to-paint = 38ms |
| 3 | Undo works | PASS | Manual test, plus unit test in `tests/recorder.spec.ts` |
| 4 | Cross-tab sync | PASS | Two-tab test, sync time ~600ms via Supabase channel |
| 5 | Keyboard shortcuts | PASS | All 10 throws via keyboard, video at `tests/evidence/recorder-kb.mp4` |
| 6 | Offline mode | PASS | Plane mode test, throws queued in IndexedDB, synced on reconnect |

### Evaluator independent evaluation (Gate 2)
| # | Criterion | Result | Evidence |
|---|-----------|--------|----------|
| 1 | Console errors | PASS | Independent test on dev rig, all 3 viewports clean |
| 2 | Tap-to-tag <100ms | PASS | Subjectively snappy; perf trace verified |
| 3 | Undo works | PASS | Tested 5 sequential undo operations, no inconsistency |
| 4 | Cross-tab sync | PASS | Tested across 3 tabs (not 2!) — all synced |
| 5 | Keyboard shortcuts | INITIAL FAIL → FIXED | "U" key was being eaten by browser undo on focused input. Pixel pushed back; Prism added `e.preventDefault()`. Re-test passed. |
| 6 | Offline mode | PASS | Independent plane-mode test |

### Final status
- ☑ All criteria PASS after one fix loop — sprint complete

---

## Lesson captured

Criterion 5 (keyboard shortcuts) caught a real bug that would have shipped: the U-for-undo binding conflicted with the browser's built-in Cmd+Z behavior on focused inputs. Pixel's independent test caught it because she tested with focus *inside* a text input first — Builder had only tested with focus on the page body. **The "test method" text in the contract didn't specify focus state**, which is why the Builder's self-eval missed it.

Lesson for future contracts: when testing keyboard interactions, the test method should specify focus state explicitly. Updated TEMPLATE.md to include this guidance.
