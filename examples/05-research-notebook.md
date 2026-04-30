# Notebook Contract: Architecture Ceiling on Holdout Split

**Date:** 2026-04-19
**Builder (Author):** ML specialist
**Evaluator:** QA + research-discipline gate
**Planner:** Project lead

> **Real-world example — research variant.** Notebook contracts apply the same generation/evaluation separation to research work. Pre-registered hypotheses + binary exit criteria prevent post-hoc rationalization. Adapted for ML/data-science workflows where the "build" is a notebook and "ship" is a public publication.

---

## Research question

What accuracy ceiling can a given model architecture family reach on the chosen dataset under proper holdout splitting? Is the plateau the model's, or the data's?

---

## Pre-registered hypotheses

These hypotheses are **registered before any model is trained.** Outcomes that match get reported as confirmed; outcomes that don't get reported as falsified. No hypothesis is added or modified after results are seen.

- **H1:** Architecture matters at the margin (>5pp F1 spread across the architecture ladder)
- **H2:** The temporal-vs-static gap is small (<5pp F1) on the task class
- **H3:** Holdout-split accuracy plateaus below a target threshold for the architecture family
- **H4:** Frame-level architectures and sequence-level architectures converge within noise

---

## Success Criteria (Exit Conditions)

Each criterion is binary PASS/FAIL. Failure on any criterion means the notebook does not ship as a public publication.

| # | Criterion | How to Test | Pass Condition | Status |
|---|-----------|-------------|----------------|--------|
| 1 | All architectures train to convergence with identical splits | Run `scripts/run_architecture_sweep.py --all-archs --seeds 3` | Every arch completes 3 seeds each, no early-stop or NaN crashes | PENDING |
| 2 | Each architecture has error bars across ≥3 seeds | Inspect `runs/<sweep>/*/metrics.json` | Every arch has ≥3 seed entries with mean and std reported | PENDING |
| 3 | Holdout splitting verified (no leakage) | Run `tests/test_split_integrity.py` | Test passes: zero overlap of `participant_id` across train/val/test | PENDING |
| 4 | Permuted-labels sanity check on smallest arch | Run training with shuffled labels, 5 epochs | Accuracy stays at chance — confirms no data leak | PENDING |
| 5 | Per-arch result table renders in notebook | Render notebook end-to-end | Table includes mean ± std for each arch, sorted by F1 | PENDING |
| 6 | Pre-registered hypotheses each marked confirmed/falsified with evidence | Notebook discussion section | All hypotheses adjudicated with explicit data citation | PENDING |
| 7 | Notebook runs end-to-end on a fresh kernel in <30 min on free-tier GPU | Submit to public platform as draft, run from scratch | Completes without manual intervention, total runtime <30 min | PENDING |
| 8 | No bare claim of "best architecture" without statistical caveat | Manual read of conclusion section | Every comparative claim cites error bars and seed count | PENDING |

---

## Out of Scope

- Hyperparameter tuning (this notebook uses literature defaults — that's the limitation we're studying)
- Out-of-class extension
- Cross-dataset validation
- Inference latency benchmarks (separate notebook)

---

## Dependencies

- Frozen test split available at `data/splits/holdout-2026-04-15.npz`
- All architecture scaffolds in `models/`
- Cloud training budget approved
- Dataset cached locally

---

## Rollback Plan

A failed notebook does not ship. Results go into `_drafts/notebook-rejected-2026-04-XX.md` documenting which criteria failed and why. Lessons feed the next notebook contract.

---

## Signatures

- **Builder:** ML specialist — AGREED 2026-04-19
- **Evaluator:** QA + research-discipline rule — AGREED 2026-04-19
- **Planner:** Project lead — GATE 0 CLEARED 2026-04-19

---

## Post-sprint

### Builder self-evaluation (Gate 1)
| # | Criterion | Result | Evidence |
|---|-----------|--------|----------|
| 1 | All archs trained | PASS | All seeds in `runs/<sweep>/...` |
| 2 | Error bars on each | PASS | Mean ± std reported for all architectures |
| 3 | No leakage | PASS | `tests/test_split_integrity.py` passes |
| 4 | Permuted-labels sanity | PASS | Accuracy at chance, within noise |
| 5 | Result table renders | PASS | Notebook cell renders the table |
| 6 | Hypotheses adjudicated | PASS | Discussion explicitly addresses each H1–H4 |
| 7 | <30 min on free-tier GPU | PASS | 24:18 total |
| 8 | No bare "best" claims | PASS | Discussion uses "led on this benchmark with overlapping error bars vs. ..." |

### Evaluator independent evaluation (Gate 2)
| # | Criterion | Result | Evidence |
|---|-----------|--------|----------|
| 1 | All archs trained | PASS | Independently verified each `metrics.json` |
| 2 | Error bars | PASS | Recomputed from raw seed runs, matches |
| 3 | No leakage | PASS | Independent check of `participant_id` overlap |
| 4 | Permuted-labels | PASS | Re-ran sanity check on smallest arch |
| 5 | Table renders | PASS | Confirmed |
| 6 | Hypotheses | PASS | Each hypothesis cites specific seed-mean ± std with evidence |
| 7 | Runtime | PASS | Independent submission, 26:14 |
| 8 | No bare claims | PASS | Read full discussion, no overclaims |

### Final status
- ☑ All criteria PASS — notebook ready for publication

---

## Lesson captured

One pre-registered hypothesis was falsified by the data. This was a *good* outcome of the contract: pre-registering forced the Builder to commit to a position before seeing data. When the result came in different from the prediction, the discussion section says so explicitly. Without pre-registration, the temptation would have been to retroactively redefine the hypothesis or quietly drop it.

This is the value of notebook contracts: **they make the analysis honest by binding it before the data is seen.**
