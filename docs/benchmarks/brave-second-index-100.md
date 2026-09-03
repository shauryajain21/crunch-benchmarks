# Brave as second retrieval index

**Name:** Brave as second retrieval index  
**Date:** 2026-08-20  
**ID:** `brave-second-index-100`  
**n:** 100  
**Status:** rejected  
**Source:** Deep-eval:STATUS.md

Difference-in-differences using untreated rows as control.

## Arms

- Toolbox only
- Toolbox plus Brave

## Headline

| Metric | Value |
|---|---|
| `treated_delta` | -0.078 |
| `control_delta` | -0.036 |
| `difference_in_differences` | -0.042 |
| `ci95` | 0.058 |

## Decision

Do not add retrieval breadth when the evidence-read budget is the bottleneck.

## Limitations

- Run-to-run noise was as large as candidate effects.
