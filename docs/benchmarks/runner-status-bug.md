# Answer-derived status in split-finish runs

**Name:** Answer-derived status in split-finish runs  
**Date:** 2026-09-01  
**ID:** `runner-status-bug`  
**n:** affected_successes=845  
**Status:** invalid  
**Source:** crunch-public:evals/RESULTS.md

Recomputed run status.

## Arms

- status from answer text
- status from output contract

## Headline

| Metric | Value |
|---|---|
| `reported_success_rate` | 0.01 |
| `corrected_quality_win_rate` | 0.728 |

## Decision

Derive success from typed output, not presence of prose.

## Limitations

- Invalidated only operational status, not stored result documents.
