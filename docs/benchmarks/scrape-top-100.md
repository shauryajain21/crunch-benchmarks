# Forced top-page scraping

**Name:** Forced top-page scraping  
**Date:** 2026-08-21  
**ID:** `scrape-top-100`  
**n:** 100  
**Status:** rejected  
**Source:** Deep-eval:STATUS.md

Question: Does automatic top-page scraping beat snippets-first?

Same questions and checklist pass.

## Arms

- scrape_top=3
- scrape_top=0

## Headline

| Metric | Value |
|---|---|
| `pass_with` | 0.9295 |
| `pass_without` | 0.9403 |
| `p50_with_s` | 30.5 |
| `p50_without_s` | 9.5 |

## Decision

Disable automatic top-page scraping.

## Limitations

- No-scrape config still allows model-selected fetches.

Supersedes: `scrape-top-20`.
