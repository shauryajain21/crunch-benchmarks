# Resume — 2026-09-03 laptop close

Open this first. Catalog and first-hop are already on GitHub. The bakeoff lives on disk under `~/tmp`.

## Already pushed

- Eval writeup pages + design-doc links: `crunch-benchmarks` `main` `e8dcb84`
- First-hop A/B cataloged as diagnostic: `8a1fb02` / `1c4a60d` — page `docs/benchmarks/firsthop-ab-100.md`
- Decisions / harness design doc on `main` (`b3ec1ce` and follow-ups)
- First-hop ran against crunch-public `478a4ae`

## This save

- crunch-public `production-evals` `769d158` — server-side request filters + gather-to-schema for structured
- crunch-public `b4e68eb` — pairwise judge clip/token controls (mid-rationale cuts were being scored as ties)
- crunch-public `187fe01` — leftover filtered / Vespa / gather-to-schema eval runners (no logs, no jsonl, no keys)
- This file on `crunch-benchmarks` `main`

## Do not commit / do not delete

- `~/tmp/crunch-*` live run dirs
- Deep-eval is dirty (`crunch-parity-fixes`): local docs + `data/*.json` + `results/`. Left uncommitted (customer prompts / artifacts)
- Keys: OpenAI / Anthropic / Gemini / Baseten are already in your env and shell history. Do not paste them here.

## In-flight / incomplete

- **`model-eval-200`** — generation only, **no quality judge**. Catalog page `docs/benchmarks/model-eval-200.md` is stale vs disk.
- **`vespa-prod1000`** — never started (historically blocked on Infisical/Columbus). No artifacts.
- First-hop is **done** (100/100 both arms + judge). Raw dir kept; no shipped-config change.

## Bakeoff progress on disk

Dir: `/Users/shaurya/tmp/crunch-model-eval-200-20260903`  
Script: `run_bakeoff.py`  
Questions: `crunch-public/evals/filtered-200-20260903/questions.jsonl`  
Counts at save (unique `query_id` / 200). “Clean” = `status==200` and no `error`.

| Arm | Unique | Clean | Notes |
|---|---:|---:|---|
| `gemini-direct` | 200 | 200 | Done |
| `glm-baseten` | 200 | 191 | Manifest 199/1; 10 extra resume lines |
| `luna-safe` | 200 | 187 | Manifest 192 ok / 8 fail |
| `deepseek-baseten` | 150 | 47 | 207 lines on disk; still writing; many `HTTP 429` / hard-cap. **Two `--resume` processes were overlapping** (left running). Sleep will kill them. |
| `luna` | 117 | 116 | Unfinished; no `finished_at` |
| `gemini` (OpenRouter) | 200 | 44 | Rest OpenRouter 402 |
| `glm` (OpenRouter) | 200 | 7 | Rest OpenRouter 402 |
| `qwen` | 5 | 3 | Smoke only |
| `deepseek` (OpenRouter) | 5 | 5 | Smoke only |

## First-hop raw dir

`/Users/shaurya/tmp/crunch-firsthop-ab-100-20260903`  
`shipped/` 100/100, `free/` 100/100, `judge_pairwise.jsonl` 140 lines (100 + 40 swaps). Headline: shipped 41, free 24, tie 35.

## How to resume a bakeoff arm

One arm at a time. **8 workers max. Do not exceed 8.** Script caps in-flight `WebSearchTool` at **24**. Do not overlap Toolbox arms.

```bash
cd /Users/shaurya/tmp/crunch-model-eval-200-20260903
python3 -u run_bakeoff.py --arm <arm> --workers 8 --resume
```

If `deepseek-baseten` still has two processes after you reopen, stop the extras before starting anything else on Toolbox. Then resume that arm alone, or judge the completed arms.

Other leftover dirs (not this session’s critical path): `~/tmp/crunch-ab`, `~/tmp/crunch-schema-guidance-pilot`.

## Do first when you reopen

1. Read this file. Confirm `~/tmp/crunch-model-eval-200-20260903` is still there.
2. Check whether `deepseek-baseten` processes survived sleep. Do not start another Toolbox arm on top of them.
3. **Useful next:** pairwise-judge the completed generation arms (`gemini-direct`, `glm-baseten`, `luna-safe`) — first-hop is already cataloged. Finish `deepseek-baseten` only if 429s have cooled.
4. Do not start `vespa-prod1000` until Columbus/Infisical is healthy, and not while a Toolbox bakeoff arm is running.
