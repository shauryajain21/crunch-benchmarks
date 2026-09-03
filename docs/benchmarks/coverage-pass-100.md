# Post-loop coverage pass

**Name:** Post-loop coverage pass  
**Date:** 2026-08-21  
**ID:** `coverage-pass-100`  
**n:** 100  
**Status:** adopted  
**Source:** Deep-eval:STATUS.md

Question: Does a post-loop coverage pass raise quality?

Paired checklist judge with traffic-type split.

## Arms

- v7
- v7 plus coverage

## Headline

| Metric | Value |
|---|---|
| `v7_pass` | 0.911 |
| `v7_plus_coverage` | 0.930 |
| `pass_delta` | 0.019 |
| `ci95` | 0.029 |
| `lookup_delta` | 0 |
| `missing_before` | 0.045 |
| `missing_after` | 0.033 |

## Decision

Keep for honest omission reduction; do not claim its task-prompt headline as quality gain.

## Limitations

- Agentic-task gain partly rewarded citation laundering.
