# BFCL v4 Evaluation Artifacts

Per-category outputs from the official [`bfcl-eval`](https://github.com/ShishirPatil/gorilla/tree/main/berkeley-function-call-leaderboard) AST scorer, run locally on 2x Tesla T4.

**Aggregate scores:**

| Split | AST Accuracy |
|---|---:|
| Non-Live | 72.56% |
| Live | 58.62% |

## Files

One JSONL file per category. Each line is a single test case result.

| File | Cases | Category |
|---|---:|---|
| `BFCL_v4_simple_python_result.json` | 400 | Single tool, Python schema |
| `BFCL_v4_simple_java_result.json` | 100 | Single tool, Java schema |
| `BFCL_v4_simple_javascript_result.json` | 50 | Single tool, JavaScript schema |
| `BFCL_v4_multiple_result.json` | 200 | One call from 2-4 candidates |
| `BFCL_v4_parallel_result.json` | 200 | N calls to same tool |
| `BFCL_v4_parallel_multiple_result.json` | 200 | Coordinated multi-tool |
| `BFCL_v4_irrelevance_result.json` | 240 | Detect no-match |
| `BFCL_v4_live_simple_result.json` | 258 | One call, messy prompts |
| `BFCL_v4_live_multiple_result.json` | 1,053 | Diverse live candidate selection |
| `BFCL_v4_live_parallel_result.json` | 16 | Repeat live API for N entities |
| `BFCL_v4_live_parallel_multiple_result.json` | 24 | Multi-action single turn |
| `BFCL_v4_live_irrelevance_result.json` | 884 | Refuse live out-of-scope |
| `BFCL_v4_live_relevance_result.json` | 16 | Fire a call when warranted |

## Line format

```json
{"id": "<test-id>", "result": "[<parsed-call>]"}
```

- `result` is the model's cleaned output after `<tool_call>` extraction and refusal normalization.
- A refusal is emitted as `"no_call"`.
- Valid calls are emitted as `[func_name(arg=value, ...)]`.

## How the scores were produced

- Scorer: official `bfcl-eval` package, AST evaluator, no harness-level modifications.
- Model registered via a custom `OSSHandler` entry in `MODEL_CONFIG_MAPPING`.
- Output parser is category-aware for boolean literals: lowercase `true`/`false`/`null` for Java and JavaScript categories (AST compatibility), Python-style `True`/`False`/`None` otherwise.

## Verification

Install the official scorer and re-run against these files:

```bash
pip install bfcl-eval
BFCL_PROJECT_ROOT=. bfcl-eval --model qwen3-1.7b-tool-calling-v4
```

The aggregate numbers above should reproduce without access to the training code.

## Notes

- `simple_java` (50.00%) and `simple_javascript` (60.00%) are bounded by the training data - these schema formats were not included in training.
- `live_irrelevance` (59.84%) and `irrelevance` (74.17%) reflect the model's natural-language refusal format. The BFCL AST scorer treats plain-text refusals as equivalent to `no_call`.
- Full disclosure: no results were submitted to any public leaderboard.
