# Corrected production-100 checklist benchmark

**Name:** Corrected production-100 checklist benchmark  
**Date:** 2026-08-21  
**ID:** `prod100-corrected`  
**n:** 100  
**Status:** canonical  
**Source:** crunch-public:BENCHMARKS.md

One checklist pass after citation, clipping, prompt, coverage, and scraper fixes.

## Arms

- Crunch v7
- Exa Deep Reasoning
- Linkup Deep

## Headline

| Metric | Value |
|---|---|
| `crunch_p50_s` | 9.5 |
| `crunch_p90_s` | 14.3 |
| `model_cost_per_1000` | 14.71 |

**`pass`:** `{"crunch":0.9403,"exa":0.894,"deep":0.61}`

**`unsourced`:** `{"crunch":0.028,"exa":0.076,"deep":0.115}`

## Decision

Canonical internal-harness configuration; quality is tied with Exa on ordinary lookups.

## Limitations

- Overall lead leans on agentic prompts; internal harness, not public crunch.py.

Supersedes: `prod100-initial`.
