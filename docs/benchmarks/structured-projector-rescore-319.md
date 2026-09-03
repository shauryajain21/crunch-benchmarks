# Structured projector repair rescore

**Name:** Structured projector repair rescore  
**Date:** 2026-09-02  
**ID:** `structured-projector-rescore-319`  
**n:** 319  
**Status:** superseded  
**Source:** crunch-public:evals/RESULTS.md

Stored writes reprojected with null/coercion/source fixes, then rejudged.

## Arms

- Reprojected Crunch writes
- Frozen Linkup Deep

## Headline

| Metric | Value |
|---|---|
| `wins` | 215 |
| `losses` | 93 |
| `ties` | 11 |
| `decided_win_rate` | 0.6981 |

**`ci95`:** `[0.6468,0.7493]`

## Decision

Adopt projector repairs; use direct-projection confirmation for the newest architecture decision.

## Limitations

- Selected sample; not a fresh 456-row retrieval run.

Supersedes: `production-2000-structured-shipping`.
