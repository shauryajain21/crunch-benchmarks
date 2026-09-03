# Initial loop-model bake-off

**Name:** Initial loop-model bake-off  
**Date:** 2026-08-19  
**ID:** `model-bakeoff-20`  
**n:** 20  
**Status:** adopted  
**Source:** Deep-eval:STATUS.md

Sequential same-question model bake-off; two checklist judges.

## Arms

- Gemini 3.7 Flash
- GPT-4.1 mini
- GPT-5.6 Luna
- four slower models

## Headline

| Metric | Value |
|---|---|
| `gemini_flash_p50_s` | 6.341 |
| `gemini_flash_p90_s` | 9.03 |
| `gemini_flash_cost` | 0.0187 |

## Decision

Use gemini-3.7-flash with no deadline; Luna remains latency-relaxed runner-up.

## Limitations

- n=20 could not reliably rank close quality arms.
