# Vespa-only DeepSearchQA

**Name:** Vespa-only DeepSearchQA  
**Date:** 2026-08-31  
**ID:** `vespa-dsqa-898`  
**n:** 898  
**Status:** diagnostic  
**Source:** Deep-eval:SESSION.md

Question: Does Vespa-only hybrid match Toolbox Crunch on DeepSearchQA?

Official-style all-correct autorater (gemini-3.5-flash) on both arms in one sitting. Not the published 2.5-flash leaderboard. First 450 was an intermediate slice of the same 8-hop run.

## Arms

- Vespa-only hybrid Crunch
- Toolbox dsqa900_h8

## Headline

| Metric | Value |
|---|---|
| `vespa_all_correct` | 0.2929 |
| `toolbox_all_correct` | 0.3608 |
| `vespa_correct` | 263 |
| `toolbox_correct` | 324 |
| `paired_n` | 897 |
| `both` | 214 |
| `vespa_only` | 49 |
| `toolbox_only` | 110 |
| `neither` | 524 |
| `mcnemar_chi2` | 22.6 |
| `first450_vespa` | 0.313 |
| `first450_toolbox` | 0.374 |
| `vespa_p50_s` | 25.1 |
| `toolbox_p50_s` | 30.1 |
| `vespa_cost_per_1000` | 71.7 |
| `toolbox_cost_per_1000` | 46.06 |

## Decision

Vespa-only is about 7 points worse, faster, and more expensive (longer snippets). Use as the vespa-versus-toolbox DSQA result; do not quote against the public 36.2% board.

## Limitations

- Replacement Gemini 3.5 Flash autorater; dsqa_530 returned empty.
- Cost gap is mostly extra Gemini input tokens from uncapped Vespa chunks.
