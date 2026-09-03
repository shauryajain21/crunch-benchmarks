# Vespa-only versus Toolbox on production-100

**Name:** Vespa-only versus Toolbox on production-100  
**Date:** 2026-08-31  
**ID:** `vespa-prod100`  
**n:** 100  
**Status:** diagnostic  
**Source:** Deep-eval:SESSION.md

Question: Does Vespa-only hybrid match Toolbox Crunch on production-100?

Same Crunch agent; DAG antoine_xp_2_internal_hybrid_50_50_10 (lexical+semantic Vespa+RRF, no Brave). One checklist extract pass versus stored Toolbox answers.

## Arms

- Vespa-only hybrid Crunch
- Toolbox Crunch full_v7cov_ns

## Headline

| Metric | Value |
|---|---|
| `vespa_pass` | 0.892 |
| `toolbox_pass` | 0.938 |
| `paired_better` | 3 |
| `paired_worse` | 15 |
| `paired_tie` | 82 |
| `vespa_supported` | 0.814 |
| `toolbox_supported` | 0.863 |
| `vespa_p50_s` | 16.5 |
| `toolbox_p50_s` | 9.5 |

## Decision

Vespa-only drops about 5 points on ordinary production lookups versus Toolbox. Not a collapse; keep external long-tail retrieval for general traffic.

## Limitations

- Same-sitting checklist versus Toolbox; not versus Deep.
