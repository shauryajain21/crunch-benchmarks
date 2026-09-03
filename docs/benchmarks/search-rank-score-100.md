# Cross-encoder score ordering

**Name:** Cross-encoder score ordering  
**Date:** 2026-08-24  
**ID:** `search-rank-score-100`  
**n:** 100  
**Status:** superseded  
**Source:** crunch-public:OUTPUT-MODES.md

Paired gold-answer recall.

## Arms

- Arrival
- RRF
- Raw score

## Headline

**`top10`:** `{"arrival":0.33,"rrf":0.517,"score":0.566}`

## Decision

Evidence that arrival order was wrong; raw score was later found incomparable across rewritten queries.

## Limitations

- Recall against gold answer is not user quality.

Supersedes: `search-rank-rff-50`.
