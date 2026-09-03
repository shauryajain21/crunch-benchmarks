# Question-aware schema projection

**Name:** Question-aware schema projection  
**Date:** 2026-08-24  
**ID:** `structured-projection-50`  
**n:** 50  
**Status:** adopted  
**Source:** Deep-eval:results/output_modes/README.md

Reprojected identical stored prose; one schema-validation retry.

## Arms

- Old projector
- Question-aware validated projector
- Prose ceiling

## Headline

| Metric | Value |
|---|---|
| `strict_old` | 0.6 |
| `strict_new` | 0.62 |
| `strict_prose` | 0.62 |
| `recall_new` | 0.717 |

## Decision

Include the question and validate output; projection should preserve prose quality.

## Limitations

- DSQA explanation field reduced sensitivity.
