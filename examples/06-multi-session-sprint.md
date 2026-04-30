# Sprint Contract: Operator Dashboard v1 — Multi-Session

**Date:** 2026-03-30 (start) → 2026-04-08 (close)
**Builder:** Frontend specialist
**Evaluator:** QA + Visual reviewer
**Planner:** Project lead
**Sessions:** 4

> **Real-world example — multi-session variant.** Sprints longer than one work session need a paired progress file that gets read at the start of each session and updated at the end. Same pattern Anthropic's Harness 2.0 uses internally. Prevents context-window collapse from losing what was already done.

---

## Objective

Ship Operator Dashboard v1: a Next.js dashboard that shows live session state, event history, current scores, and an operator panel for session control. Replaces the current ad-hoc HTML page.

---

## Success Criteria

| # | Criterion | How to Test | Pass Condition | Status |
|---|-----------|-------------|----------------|--------|
| 1 | Dashboard renders at `/dashboard` | `npm run dev`, open URL | Page loads, no console errors, all sections visible | PENDING |
| 2 | Live session state updates within 500ms of event | Tag an event via recorder, observe dashboard | Score, event count, last-event card update <500ms | PENDING |
| 3 | Event history shows last 50 events with scrollback | Tag 60 events over time | Last 50 visible, oldest scrolls off, no perf degradation | PENDING |
| 4 | Operator panel: pause/resume/reset session | Click each control, observe state | All three controls produce expected backend state change | PENDING |
| 5 | Responsive at 1080p TV viewport (1920×1080) and laptop (1440×900) | View at both sizes | No layout breakage at either viewport | PENDING |
| 6 | Stays usable for 60-minute continuous session | Run for 60 min with 5+ events/min | No memory leak (heap stable), no UI freeze | PENDING |

---

## Out of Scope

- Mobile responsive design
- User authentication
- Multi-session views
- Historical analytics (separate sprint)
- Voice command integration

---

## Dependencies

- Recorder UI v1 shipped (criterion 2 depends on it emitting events)
- Realtime channel `dashboard:session_<id>` configured
- `pm2` available for the 60-min stress test

---

## Rollback Plan

Operator Dashboard v1 is a new route. Rollback = revert all commits in the sprint and remove the route file. Old ad-hoc HTML page remains accessible at `legacy/dashboard.html` for reference.

---

## Signatures

- **Builder:** Frontend specialist — AGREED 2026-03-30
- **Evaluator:** QA + Visual reviewer — AGREED 2026-03-30
- **Planner:** Project lead — GATE 0 CLEARED 2026-03-30

---

## Sprint Progress File

> Lives at `memory/sprint-progress-dashboard-v1.md`. Read at the START of every session. Updated at the END of every session before context closes.

### Session 1 (2026-03-30)
**Completed this session:**
- Scaffolded Next.js route at `app/dashboard/`
- Implemented `<SessionStateCard />` component with mock data
- Wired realtime client (`lib/realtime.ts`)

**Contract criteria status:**
| # | Status | Notes |
|---|--------|-------|
| 1 | IN PROGRESS | Page renders with mock data, real data wiring next session |
| 2 | NOT STARTED | |
| 3 | NOT STARTED | |
| 4 | NOT STARTED | |
| 5 | NOT STARTED | |
| 6 | NOT STARTED | |

**Next session should:**
1. Wire realtime subscription to session state
2. Replace mock data with live channel data
3. Verify criterion 1 fully (real data, no errors)

**Known issues:**
- TypeScript inference is loose on the realtime payload — needs explicit typing

**Git state:**
- Branch: `feat/dashboard-v1`
- Last commit: `a4f2b91` — "scaffold dashboard v1 route + session-state-card with mock data"

---

### Session 2 (2026-04-02)
**Completed this session:**
- Wired realtime subscription
- Live session state working
- Implemented `<EventHistory />` with last-50 scrollback
- Operator panel buttons placed (not yet wired)

**Contract criteria status:**
| # | Status | Notes |
|---|--------|-------|
| 1 | DONE | Renders, no errors |
| 2 | DONE | <500ms verified, ~280ms typical |
| 3 | DONE | 60-event test passed, scrollback works |
| 4 | IN PROGRESS | UI placed, backend hookup next session |
| 5 | NOT STARTED | |
| 6 | NOT STARTED | |

**Next session should:**
1. Wire operator panel to backend (`POST /api/session/pause`, etc.)
2. Test responsive at 1080p TV viewport
3. Set up the 60-min stress test (criterion 6)

**Known issues:**
- Realtime channel occasionally drops on idle >5min — known issue, need keep-alive

**Git state:**
- Branch: `feat/dashboard-v1`
- Last commit: `c8e2d04` — "wire realtime, ship event history with scrollback"

---

### Session 3 (2026-04-05)
**Completed this session:**
- Operator panel wired end-to-end (pause/resume/reset)
- Realtime keep-alive implemented
- Tested responsive at 1080p TV — fixed two CSS issues (overflow on history, type-size on scoreboard)

**Contract criteria status:**
| # | Status | Notes |
|---|--------|-------|
| 1 | DONE | |
| 2 | DONE | |
| 3 | DONE | |
| 4 | DONE | All three controls verified |
| 5 | DONE | Both viewports tested, fixes shipped |
| 6 | IN PROGRESS | 60-min test starting overnight |

**Next session should:**
1. Review overnight stress-test results
2. Submit to Evaluator if criterion 6 passes
3. Begin Gate 2 evaluation cycle

**Known issues:**
- None new

**Git state:**
- Branch: `feat/dashboard-v1`
- Last commit: `1a7b6c2` — "wire operator panel, fix TV viewport CSS, add keepalive"

---

### Session 4 (2026-04-08) — Gate 2 + close
**Completed this session:**
- Reviewed overnight stress-test: 73-min run, heap stable, no UI freeze, criterion 6 PASS
- Submitted to Evaluator for Gate 2
- Evaluator ran independent eval, all 6 criteria PASS
- Merged to `main` at commit `4e9f3a7`
- Sprint closed

**Contract criteria status:**
| # | Status | Notes |
|---|--------|-------|
| 1-6 | DONE | All PASS by Evaluator Gate 2 |

**Sprint closed.** Postmortem at `memory/postmortems/2026-04-08-dashboard-v1.md`.

---

## Post-sprint

### Final evaluator independent evaluation (Gate 2)
All 6 criteria PASS. See sprint-progress-dashboard-v1.md session 4 for evidence per criterion.

### Final status
- ☑ All criteria PASS — sprint complete after 4 sessions
- Branch merged: `feat/dashboard-v1` → `main` at `4e9f3a7`

---

## Lesson captured

The progress file was the difference between this sprint shipping in 4 sessions and shipping in 8. Without it, sessions 2 and 3 would have spent 30+ minutes re-orienting ("where was I, what's done, what's broken"). The 5-minute end-of-session write is repaid 3-6× at the start of the next session.

**Rule:** any sprint that doesn't complete in one session must have a progress file. No exceptions. Read at start, write at end.
