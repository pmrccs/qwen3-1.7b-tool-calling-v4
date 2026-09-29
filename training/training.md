# Training Artifacts

Step-level training logs, trainer state, and run summary for `pmrccs/qwen3-1.7b-tool-calling-v4`.

Model: LoRA r=16 adapter over `unsloth/Qwen3-1.7B`, merged to fp16.
Run: 300 steps, effective batch 32, 2x Kaggle T4 via PyTorch DDP.

## Files

### `step_log_full.jsonl`

Per-step training metrics, one JSON object per line.

```json
{"step": 1, "wall_time": 0.0, "loss": 2.54814, "learning_rate": 0.0, "grad_norm": 0.834}
{"step": 25, "wall_time": 206.9, "loss": 1.59098, "eval_loss": 1.25769, "learning_rate": 4.99e-05}
```

| Field | Present on | Meaning |
|---|---|---|
| `step` | all lines | Global step number |
| `wall_time` | all lines | Seconds since the training loop started |
| `loss` | training step | Training loss at that step |
| `eval_loss` | eval step | Evaluation loss on the held-out 5% split |
| `learning_rate` | all lines | LR at that step |
| `grad_norm` | some lines | Gradient L2 norm before the optimizer step |

### `trainer_state_full.json`

Full HF `TrainerState.log_history`. Contains every logged entry in the order it was produced. Useful for reconstructing exact sequences or re-plotting.

### `sweep_result_full.json`

Final run summary:

```json
{
  "tag": "full",
  "learning_rate": 5e-05,
  "max_steps": 300,
  "final_train_loss": 0.97852,
  "final_eval_loss": 0.61857,
  "best_eval_loss": 0.61841,
  "steps_completed": 300
}
```

### `all_steps_combined.csv`

Flat CSV version of `step_log_full.jsonl`. Columns:

```
tag,step,wall_time,loss,eval_loss,learning_rate,grad_norm
```

Intended for loading into pandas or a spreadsheet. Blank cells where a field wasn't logged for that step.

## Run summary

| Metric | Value | Step |
|---|---:|---:|
| Initial training loss | 2.54814 | 1 |
| Final training loss | 0.97852 | 300 |
| Initial evaluation loss | 1.25769 | 25 |
| Best evaluation loss | 0.61841 | 300 |
| Final evaluation loss | 0.61857 | 300 |
| Total training FLOPs | 7.3865e+16 | - |

## Evaluation loss progression

| Step | Train loss | Eval loss | LR |
|---:|---:|---:|---:|
| 1 | 2.54814 | - | 0.00e+00 |
| 25 | 1.59098 | 1.25769 | 4.99e-05 |
| 50 | 1.45040 | 0.85552 | 4.85e-05 |
| 75 | 1.12508 | 0.71383 | 4.52e-05 |
| 100 | 0.67489 | 0.66835 | 4.05e-05 |
| 125 | 0.98662 | 0.65285 | 3.45e-05 |
| 150 | 1.12018 | 0.64302 | 2.79e-05 |
| 175 | 1.23996 | 0.63248 | 2.10e-05 |
| 200 | 1.05181 | 0.62608 | 1.45e-05 |
| 225 | 1.13671 | 0.62162 | 8.69e-06 |
| 250 | 0.92527 | 0.61901 | 4.15e-06 |
| 275 | 1.32625 | 0.61857 | 1.18e-06 |
| 300 | 0.97852 | 0.61841 | 1.37e-08 |

Eval loss saturates after step ~200. The final 100 steps contributed ~0.008 of improvement - the model extracted essentially all signal this data distribution provides. Further gains require a different data mix, not more steps.

## Notes

- Training loss is per-step and noisy; the evaluation loss is the cleaner convergence signal.
- The 5% held-out split was created by `train_test_split(test_size=0.05, seed=42)` from the combined dataset. It is not a held-out benchmark - it is an in-distribution validation slice.
- `grad_norm` is present only on steps where the HF Trainer logged it (not every step).
