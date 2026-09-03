# Vespa-only DeepSearchQA at 16 hops

**Name:** Vespa-only DeepSearchQA at 16 hops  
**Date:** 2026-08-31  
**ID:** `vespa-dsqa-450-h16`  
**n:** 450  
**Status:** diagnostic  
**Source:** Deep-eval:SESSION.md

Question: Do 16 hops close the vespa-versus-toolbox DSQA gap?

First 450 DSQA questions, vespa-hybrid, same gemini-3.5-flash autorater.

## Arms

- Vespa 16 hops / 240s
- Vespa 8 hops
- Toolbox 8 hops

## Headline

| Metric | Value |
|---|---|
| `vespa_16h` | 0.322 |
| `vespa_8h` | 0.313 |
| `toolbox_8h` | 0.374 |
| `correct_16h` | 145 |
| `vs_vespa_8h_better` | 32 |
| `vs_vespa_8h_worse` | 28 |
| `forced_writes_16h` | 19 |
| `forced_writes_8h` | 122 |
| `mean_hops_16h` | 7 |
| `hit_16` | 19 |
| `cost_per_1000_16h` | 87.33 |
| `cost_per_1000_8h` | 71.32 |

## Decision

Extra hops mostly stopped forced writes. Quality +0.9 vs vespa 8-hop is noise and does not close the Toolbox gap.

## Limitations

- 450-row slice, not the full 896/898.
