# NexusRaven Evaluation Artifacts

Per-query results and aggregate metrics for `pmrccs/qwen3-1.7b-tool-calling-v4` on the [`Nexusflow/NexusRaven_API_evaluation`](https://huggingface.co/datasets/Nexusflow/NexusRaven_API_evaluation) dataset (318 queries, 65 APIs).

Run locally with a custom harness. Not submitted to any public leaderboard.

## Aggregate scores

| Metric | Lenient | Strict |
|---|---:|---:|
| Function Retrieval F1 | 0.8847 | 0.8847 |
| Argument Inference F1 | 0.8725 | 0.8318 |
| Overall F1 | 0.8764 | 0.8485 |

**Scoring definitions:**

- **Lenient** - coerces types via `float()` and lowercase comparison (`"2"` matches `2`; `"True"` matches `True`).
- **Strict** - rejects type mismatches. `"2"` does not match `2`; `"True"` does not match `True`.

The ~3-point gap between lenient and strict on argument inference reflects the model emitting string-typed values where the schema expects numeric or boolean types.

## Files

### `results_full.json`

One JSON object per line. Each line is a single test-case result.

```json
{
  "idx": 0,
  "task_id": "cve_cpe#0",
  "dataset": "cve_cpe",
  "query": "...",
  "ground_truth": {"name": "searchCPE", "arguments": {"keywordSearch": "Microsoft Exchange 2010", "limit": 2}},
  "prediction": "[searchCPE(keywordSearch='Microsoft Exchange 2010', limit=2)]",
  "raw_output": "<tool_call>\n{\"name\": \"searchCPE\", \"arguments\": {...}}\n</tool_call>",
  "degenerate_hit": false
}
```

Field definitions:

| Field | Meaning |
|---|---|
| `idx` | Global index across the 318-query run |
| `task_id` | `<dataset>#<idx>` composite key |
| `dataset` | One of `cve_cpe`, `emailrep`, `virustotal`, `toolalpaca` |
| `query` | The natural-language prompt |
| `ground_truth` | Gold answer in `{name, arguments}` form |
| `prediction` | Parsed and normalized call string |
| `raw_output` | Unfiltered decoder output before parsing |
| `degenerate_hit` | `true` if the n-gram guard truncated an output |

`raw_output` is preserved so any prediction can be traced back to the exact token stream the model produced.

### `metrics_full.json`

Aggregate metrics. Two top-level keys:

```json
{
  "lenient": {
    "run_mode": "full",
    "function_retrieval": {"precision": 0.8889, "recall": 0.8805, "f1": 0.8847},
    "argument_inference": {"precision": 0.8694, "recall": 0.8757, "f1": 0.8725},
    "overall": {"precision": 0.8755, "recall": 0.8772, "f1": 0.8764},
    "counts": {},
    "per_dataset": {}
  },
  "strict": {}
}
```

`per_dataset` breaks the same metrics down by `cve_cpe`, `emailrep`, `virustotal`, and `toolalpaca`.

## Per-dataset breakdown (strict)

| Dataset | N | Function F1 | Argument F1 | Overall F1 |
|---|---:|---:|---:|---:|
| `emailrep` | 38 | 0.9737 | 0.9565 | 0.9634 |
| `virustotal` | 151 | 0.8079 | 0.9415 | 0.9006 |
| `toolalpaca` | 51 | 0.9495 | 0.7965 | 0.8431 |
| `cve_cpe` | 78 | 0.9487 | 0.5977 | 0.7063 |

`cve_cpe` is the weakest: function selection is 94.87% but argument inference is 59.77%. Dominant failure modes are missing optional parameters (`limit`, `verbose`) and enum-value mismatches.

## Notes

- Natural-language refusals are scored as equivalent to `no_call` via an extended refusal regex. This is more permissive than the reference NexusRaven harness but faithful to the model's design (see the model card's "Refusal format" section).
- Both strict and lenient metrics are reported so the effect of type-coercion on the final score is visible.
