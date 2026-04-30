# Sprint Contract: Detection Model Validation

**Date:** 2026-04-15
**Builder:** ML specialist
**Evaluator:** QA + CV reviewer
**Planner:** Project lead

> **Real-world example.** Validating a new computer-vision detection model before promoting it to production-default. The contract forced concrete F1 / latency / FP-rate targets instead of "the new model looks better."

---

## Objective

Validate the new candidate detection model on out-of-distribution holdout data. Decide go/no-go on promoting to production-default.

---

## Success Criteria

| # | Criterion | How to Test | Pass Condition | Status |
|---|-----------|-------------|----------------|--------|
| 1 | F1 ≥ target threshold on 50-image OOD holdout | Run `scripts/eval_candidate.py --holdout ood-2026-04` | F1 ≥ target with confidence threshold sweep documented | PENDING |
| 2 | False-positive rate per primary class ≤ 2/50 images | Same eval, bucket by class | At most 2 of 50 images show 2+ "primary-object" detections (truth = 1) | PENDING |
| 3 | Inference latency p95 ≤ target | Time 1000 single-frame inferences on production hardware | p95 latency ≤ target over 1000 runs | PENDING |
| 4 | NMS sweep identifies optimal threshold | Sweep conf [0.50, 0.60, 0.70, 0.80, 0.90] | Pareto curve plotted, optimal threshold annotated with rationale | PENDING |
| 5 | No regression vs prior model on existing 100-image regression suite | Run regression script | F1(candidate) ≥ F1(prior) − 0.005 (allow 0.5% noise) | PENDING |

---

## Out of Scope

- Training a next-iteration candidate
- Edge deployment to production hardware (separate sprint)
- Model size optimization
- Calibration recalibration with new model

---

## Dependencies

- Candidate model exported to `models/candidate/`
- OOD holdout dataset frozen at `data/holdouts/ood-2026-04/` (50 images, ground-truth labels)
- Production hardware inference rig available
- Prior production model snapshot at `models/prior-prod/`

---

## Rollback Plan

If candidate fails any criterion, prior model remains the production default. Tag candidate evaluation results in `evals/candidate-rejected-YYYY-MM-DD.md` for future reference. ML team retrains with documented gaps.

---

## Signatures

- **Builder:** ML specialist — AGREED 2026-04-15
- **Evaluator:** QA + CV reviewer — AGREED 2026-04-15
- **Planner:** Project lead — GATE 0 CLEARED 2026-04-15

---

## Post-sprint

### Builder self-evaluation (Gate 1)
| # | Criterion | Result | Evidence |
|---|-----------|--------|----------|
| 1 | F1 ≥ target | PASS | F1 met target at conf 0.70 |
| 2 | FP rate ≤ 2/50 | INITIAL FAIL → FIXED | At default conf: 13 FPs. Re-evaluated at conf 0.70: 0 FPs. |
| 3 | p95 latency | PASS | Within target over 1000 runs |
| 4 | NMS sweep | PASS | See `evals/candidate-nms-sweep.png` — conf 0.70 selected |
| 5 | No regression | PASS | Candidate F1 above prior baseline |

### Evaluator independent evaluation (Gate 2)
| # | Criterion | Result | Evidence |
|---|-----------|--------|----------|
| 1 | F1 ≥ target | PASS | Re-ran eval, F1 confirmed at conf 0.70 |
| 2 | FP rate ≤ 2/50 | PASS | Counted independently: 0 FPs at conf 0.70 |
| 3 | p95 latency | PASS | Re-ran on same hardware, within target |
| 4 | NMS sweep | PASS | Pareto curve verified, threshold rationale sound |
| 5 | No regression | PASS | Independent re-run matches |

### Final status
- ☑ All criteria PASS — candidate promoted to production-default

---

## Lesson captured

Criterion 2 (FP rate ≤ 2/50) initially failed at the default threshold (13 FPs). Without this criterion, the F1 score alone would have looked like victory. The over-detection problem would have shipped to production silently and shown up as scoring bugs in user-facing data.

The contract forced an FP-rate test alongside F1, because "high F1" can hide either FN or FP failures depending on class balance. Splitting the metric was the Evaluator's pushback during contract negotiation. The Builder initially proposed only F1 as the criterion.
