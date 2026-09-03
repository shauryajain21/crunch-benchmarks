# Toolbox DeepSearchQA at 16 hops

**Name:** Toolbox DeepSearchQA at 16 hops  
**Date:** 2026-08-31  
**ID:** `dsqa-16hop-50`  
**n:** 50  
**Status:** diagnostic  
**Source:** Deep-eval:SESSION.md

Question: Do 16 hops help Toolbox DeepSearchQA?

Only 16-hop Toolbox Crunch run. Same 50 questions at 8 hops for the paired comparison.

## Arms

- Toolbox 16 hops
- Toolbox 8 hops

## Headline

| Metric | Value |
|---|---|
| `h16_all_correct` | 0.46 |
| `h16_correct` | 23 |
| `h8_all_correct` | 0.4 |
| `h8_correct` | 20 |
| `f1_h16` | 0.633 |
| `f1_h8` | 0.607 |
| `forced_writes_h16` | 1 |
| `forced_writes_h8` | 10 |
| `mean_hops_h16` | 6.2 |
| `used_all_16` | 1 |

## Decision

16 hops raised all-correct 40%→46% on n=50 and almost removed forced writes. CI is wide; no full-900 16-hop Toolbox run.

## Limitations

- n=50; CI about ±14 points. Not vespa-only.
