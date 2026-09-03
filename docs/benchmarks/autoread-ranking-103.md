# Deterministic page auto-read ranking

**Name:** Deterministic page auto-read ranking  
**Date:** 2026-08-20  
**ID:** `autoread-ranking-103`  
**n:** 103  
**Status:** rejected  
**Source:** Deep-eval:STATUS.md

Two replicates per arm and three judge passes.

## Arms

- Index order
- Intent-ranked order

## Headline

| Metric | Value |
|---|---|
| `delta` | -0.011 |
| `ci95` | 0.044 |
| `blog_section_share_before` | 0.33 |
| `after` | 0.15 |

## Decision

Leave index order as default; mechanism changed pages but not measured quality.

## Limitations

- Ranking cannot recover an absent URL.
