# Qwen3-1.7B Tool-Calling v4 — Evaluation Artifacts

Raw evaluation outputs for [pmrccs/qwen3-1.7b-tool-calling-v4](https://huggingface.co/pmrccs/qwen3-1.7b-tool-calling-v4).

Both benchmarks were run locally. Not submitted to any public leaderboard.

## Contents

- `results/bfcl_v4/` — per-category JSON outputs from the official `bfcl-eval` AST scorer
- `results/nexusraven/` — per-query results and aggregate metrics from a custom harness
- `training/` — step-level training logs (`loss`, `eval_loss`, `learning_rate`, `grad_norm`) and the final trainer state

## Aggregate scores

| Benchmark | Metric | Value |
|---|---|---|
| BFCL v4 | Non-Live AST | 72.56% |
| BFCL v4 | Live AST | 58.62% |
| NexusRaven | Overall F1 (strict) | 0.8485 |
| NexusRaven | Overall F1 (lenient) | 0.8764 |

## Method

**Model.** LoRA r=16 adapter over `unsloth/Qwen3-1.7B`, merged to fp16. Trained for 300 steps with effective batch 32 on 2× Kaggle T4. LR 5e-5 cosine, warmup 5%. Training data: 85% `NousResearch/hermes-function-calling-v1`, 15% `MadeAgents/xlam-irrelevance-7.5k`.

**BFCL v4.** Official `bfcl-eval` package, AST scorer, run on 2× T4. Model registered via a custom `OSSHandler` entry in `MODEL_CONFIG_MAPPING`. Output parser is category-aware for boolean literals (lowercase for Java/JS AST compatibility). No harness-level scoring modifications.

**NexusRaven.** Custom harness over `Nexusflow/NexusRaven_API_evaluation` (318 queries, 65 APIs). Reports both strict and lenient scoring. Strict rejects type mismatches (`"2"` ≠ `2`); lenient coerces via `float()` and lowercase comparison. Natural-language refusals are scored as equivalent to `no_call`.

## Notes

- `live_irrelevance` and `irrelevance` scores reflect the model's natural-language refusal format, which the BFCL AST scorer treats as equivalent to `no_call`.
- `simple_java` and `simple_javascript` scores reflect the fact that these schema formats were not part of the training distribution.
- Per-query `raw_output` fields are preserved in the NexusRaven result files for inspection.

## Verifying the scores

BFCL v4 scores can be independently verified by running the official `bfcl-eval` scorer against the JSON files in `results/bfcl_v4/`:

```bash
pip install bfcl-eval
BFCL_PROJECT_ROOT=. bfcl-eval --model qwen3-1.7b-tool-calling-v4 --test-category <category>
```

## License

MIT (evaluation artifacts only). Model weights are licensed cc-by-nc-4.0.
