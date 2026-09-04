# Five-model generation bake-off on filtered-200

**Name:** Five-model generation bake-off on filtered-200  
**Date:** 2026-09-03  
**ID:** `model-eval-200`  
**n:** 200  
**Status:** diagnostic  
**Source:** local:/Users/shaurya/tmp/crunch-model-eval-200-20260903

Question: Does another loop model beat Gemini 3.7 Flash on the pinned filtered-200 sample?

Live Crunch through local Toolbox. Blind typed Sonnet 4.5 pairwise vs Gemini, 40 swaps, no truncation, failures included. OpenRouter arms were dropped (HTTP 402). Qwen and Grok were not hosted.

## Arms

- Gemini 3.7 Flash (direct) — baseline, 200/200
- GLM-5 Flash (Baseten) — 199/200
- GPT-5.6 Luna (OpenAI) — 192/200
- DeepSeek V4 Flash (Baseten) — 151/200

## Headline

| Metric | Value |
|---|---|
| `glm_vs_gemini` | 117–53–30 |
| `glm_decided` | 0.688 |
| `luna_vs_gemini` | 56–123–21 |
| `luna_decided` | 0.313 |
| `deepseek_vs_gemini` | 66–106–28 |
| `deepseek_decided` | 0.384 |
| `deepseek_quality_only` | 66–57–28 |
| `glm_swap_flips` | 0.475 |
| `gemini_p50_s` | 9.8 |
| `glm_p50_s` | 26.1 |
| `deepseek_ok` | 151 |

## Decision

Keep Gemini 3.7 Flash as the shipped loop model. GLM is the only challenger that won, but it is slower and the 47.5% swap flips block a switch. Luna loses. DeepSeek still loses once failures are included; on the 151 rows it finished it is a slight lean (53.7%) and 12× slower.

## Limitations

- Filtered-200 sample, not ordinary production lookups; does not replace `model-hunt-25`.
- One GLM structured row is a grader parse-error counted as a tie.
- DeepSeek 49 failures are mechanical Gemini wins. Structured completed 1/15.
- DeepSeek swaps were graded on the first 97-row pass, not resampled after resume.
