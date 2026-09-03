# SearchResults done-call finish

**Name:** SearchResults done-call finish  
**Date:** 2026-09-01  
**ID:** `search-split-finish-200`  
**n:** 200  
**Status:** adopted  
**Source:** crunch-public:evals/RESULTS.md

Question: Should searchResults finish with a done call instead of writing discarded prose?

Sign test on paired outputs.

## Arms

- Write then discard prose
- Done call

## Headline

| Metric | Value |
|---|---|
| `wins` | 152 |
| `losses` | 48 |
| `decided_win_rate` | 0.76 |
| `quality_p` | 0.67 |
| `latency_before_s` | 44.1 |
| `latency_after_s` | 8.4 |

## Decision

End searchResults with a done call instead of writing discarded prose.

## Limitations

- A harness initially marked answerless success rows failed.
