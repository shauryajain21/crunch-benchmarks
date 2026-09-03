# Expanded shipped-config model hunt

**Name:** Expanded shipped-config model hunt  
**Date:** 2026-09-02  
**ID:** `model-hunt-25`  
**n:** 25  
**Status:** adopted  
**Source:** Deep-eval:docs/model-hunt.md

Same 25 questions and shipped Crunch config; checklist comparisons within judge pass.

## Arms

- Gemini 3.7 Flash
- Luna
- Gemini Flash Lite variants
- Qwen
- OSS-120B
- other open models

## Headline

| Metric | Value |
|---|---|
| `baseline_pass` | 0.879 |
| `luna_pass` | 0.873 |
| `gemini_35_lite_pass` | 0.692 |
| `qwen_pass` | 0.768 |
| `baseline_p50_s` | 9.4 |
| `baseline_cost_per_1000` | 14.71 |

## Decision

Keep gemini-3.7-flash: no measured arm was cheaper, faster, and better.

## Limitations

- Some slow arms were unjudged; Qwen could not honor required first-hop tools.

Supersedes: `model-bakeoff-20`.
