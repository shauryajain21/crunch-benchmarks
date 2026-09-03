# Finish-reason-only model response

**Name:** Finish-reason-only model response  
**Date:** 2026-09-01  
**ID:** `runner-row1520-crash`  
**n:** target 2000; stopped at 1520  
**Status:** incomplete  
**Source:** crunch-public:evals/RESULTS.md

Production run interrupted by indexing an absent message.

## Arms

- Original runner

## Headline

| Metric | Value |
|---|---|
| `completed_before_crash` | 1520 |

## Decision

Make each row failure-isolated and resume-safe.

## Limitations

- No complete result; not scoreable.
