# Sprint Contract: v15 RT-DETR Model Validation

**Date:** 2026-04-15
**Builder:** Cortex (ML director)
**Evaluator:** Beacon (QA) + Gauge (CV specialist)
**Planner:** Helm

> **Real-world example.** Validating a new CV detection model before promoting it from staging to production. The contract forced concrete F1 / latency targets instead of "the new model looks better."

---

## Objective

Validate v15 RT-DETR-S CoreML detection model on out-of-distribution holdout. Decide go/no-go on promoting to production-default.

---

## Success Criteria

| # | Criterion | How to Test | Pass Condition | Status |
|---|-----------|-------------|----------------|--------|
| 1 | F1 ≥ 0.95 on 50-image OOD holdout | Run `scripts/eval_v15.py --holdout ood-2026-04` | F1 ≥ 0.95 with confidence threshold sweep documented | PENDING |
| 2 | False-positive rate on board count ≤ 2/50 images | Same eval, bucket by class | At most 2 of 50 images show 2+ "board" detections (truth = 1) | PENDING |
| 3 | Inference latency p95 ≤ 75ms | Time 1000 single-frame inferences on Apple M2 | p95 latency ≤ 75ms over 1000 runs | PENDING |
| 4 | NMS sweep identifies optimal threshold | Sweep conf [0.50, 0.60, 0.70, 0.80, 0.90] | Pareto curve plotted, optimal threshold annotated with rationale | PENDING |
| 5 | No regression vs v14 on existing 100-image regression suite | Run `scripts/regression_v14_v15.py` | F1(v15) ≥ F1(v14) - 0.005 (allow 0.5% noise) | PENDING |

---

## Out of Scope

- Training a v16 candidate
- Edge deployment to Jetson (separate sprint)
- Model size optimization
- Calibration recalibration with new model

---

## Dependencies

- v15 CoreML model exported to `models/v15/yolo11s_rtdetr_960.mlpackage`
- OOD holdout dataset frozen at `data/holdouts/ood-2026-04/` (50 images, ground-truth labels)
- Apple M2 inference rig available
- v14 production model snapshot at `models/v14-prod/`

---

## Rollback Plan

If v15 fails any criterion, v14 remains the production default. Tag v15 evaluation results in `evals/v15-rejected-YYYY-MM-DD.md` for future reference. Cortex retrains with documented gaps.

---

## Signatures

- **Builder:** Cortex — AGREED 2026-04-15
- **Evaluator:** Beacon + Gauge — AGREED 2026-04-15
- **Planner:** Helm — GATE 0 CLEARED 2026-04-15

---

## Post-sprint

### Builder self-evaluation (Gate 1)
| # | Criterion | Result | Evidence |
|---|-----------|--------|----------|
| 1 | F1 ≥ 0.95 | PASS | F1 = 0.994 at conf 0.70 |
| 2 | FP rate ≤ 2/50 | INITIAL FAIL → FIXED | Conf 0.50: 13 FPs. Re-evaluated at conf 0.70: 0 FPs. |
| 3 | p95 latency ≤ 75ms | PASS | p95 = 62ms over 1000 runs |
| 4 | NMS sweep | PASS | See `evals/v15-nms-sweep.png` — conf 0.70 selected |
| 5 | No regression vs v14 | PASS | F1(v15) = 0.994 vs F1(v14) = 0.967 |

### Evaluator independent evaluation (Gate 2)
| # | Criterion | Result | Evidence |
|---|-----------|--------|----------|
| 1 | F1 ≥ 0.95 | PASS | Re-ran eval, F1 = 0.994 confirmed at conf 0.70 |
| 2 | FP rate ≤ 2/50 | PASS | Counted independently: 0 FPs at conf 0.70 |
| 3 | p95 latency ≤ 75ms | PASS | Re-ran on same M2, p95 = 64ms |
| 4 | NMS sweep | PASS | Pareto curve verified, threshold rationale sound |
| 5 | No regression vs v14 | PASS | Independent re-run matches |

### Final status
- ☑ All criteria PASS — v15 promoted to production-default
- Tag: `models/v15/PRODUCTION-DEFAULT-2026-04-19`

---

## Lesson captured

Criterion 2 (FP rate ≤ 2/50) initially failed at the default threshold (13 FPs). Without this criterion, the F1 score alone (0.994) would have looked like victory. The over-detection problem would have shipped to production silently and shown up as scoring bugs in user-facing data.

The contract forced an FP-rate test alongside F1, because "high F1" can hide either FN or FP failures depending on class balance. Splitting the metric was Beacon's pushback during contract negotiation. Cortex initially proposed only F1 as the criterion.
