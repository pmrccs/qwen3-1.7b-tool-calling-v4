# Errata

## 2026-09-29 — Mislabeled training loss field

`training/sweep_result_full.json` previously contained:

    "final_train_loss": 1.0547522400816283

This value is the **mean training loss across all 300 steps** (HuggingFace
Trainer's `stats.training_loss` aggregate), not the loss at the final step.

The loss at step 300 — which is what "final training loss" conventionally
means, and which matches `all_steps_combined.csv` and the model card — is
**0.9785223603248596**.

The field has been renamed to `mean_train_loss`, and a new
`final_step_train_loss` field has been added.

**Scope:** Only `sweep_result_full.json` was affected. The model card,
`all_steps_combined.csv`, the step log, and `training/README.md` all
already reported the correct final-step loss (0.97852). No other
artifact needs correction.

**Impact:** None. No benchmark scores, weights, datasets, or evaluation
artifacts are affected. This was a labeling issue on one derived scalar.
