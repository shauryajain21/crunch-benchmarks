# Citation renumbering rescore

**Name:** Citation renumbering rescore  
**Date:** 2026-08-21  
**ID:** `citation-renumber-rescore`  
**n:** 100  
**Status:** adopted  
**Source:** Deep-eval:STATUS.md

Question: How much of the early quality gap was broken citation IDs?

Regraded identical stored answers.

## Arms

- Broken emitted IDs
- Renumbered emitted IDs

## Headline

| Metric | Value |
|---|---|
| `before` | 0.571 |
| `after` | 0.748 |
| `delta` | 0.179 |
| `better` | 30 |
| `worse` | 0 |
| `unsourced_before` | 0.240 |
| `unsourced_after` | 0.063 |

## Decision

Renumber citations after selecting emitted sources.

## Limitations

- Measures a plumbing repair, not a model improvement.
