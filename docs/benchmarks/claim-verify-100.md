# Lexical claim verification pass

**Name:** Lexical claim verification pass  
**Date:** 2026-08-20  
**ID:** `claim-verify-100`  
**n:** 100 (graded_n=90)  
**Status:** rejected  
**Source:** Deep-eval:STATUS.md

Question: Does a lexical claim-verification pass raise checklist quality?

Same 100 production questions; five systems in one checklist pass after structural fixes. Ten extract errors left n=90 graded.

## Arms

- crunch_v3
- crunch_v4 (+verify)

## Headline

| Metric | Value |
|---|---|
| `v3_pass` | 0.622 |
| `v4_pass` | 0.611 |
| `delta` | -0.011 |
| `ci95` | 0.029 |
| `flagged_rows` | 73 |
| `mechanical_fix_rate` | 0.68 |
| `p50_v3_s` | 10.51 |
| `p50_v4_s` | 13.48 |
| `cost_per_1000_v3` | 12.17 |
| `cost_per_1000_v4` | 18.36 |

## Decision

Leave verify off. Lexical unsupported flags did not match the judge; the pass added latency and cost.

## Limitations

- Ten extract errors; pre-21-Aug clip and citation bugs still apply to absolute pass rates.
