# Host-level fetch guard

**Name:** Host-level fetch guard  
**Date:** 2026-08-20  
**ID:** `host-fetch-guard`  
**n:** ~103 domain-scoped questions; 2 runs; 15 novel-path fetches  
**Status:** adopted  
**Source:** Deep-eval:STATUS.md

Question: Should fetch allow a new path on a host already seen in results?

Operational replay and hand inspection.

## Arms

- Exact URL guard
- Seen-host guard

## Headline

| Metric | Value |
|---|---|
| `text_returned` | 14 |
| `cited` | 14 |
| `quality_delta` | -0.01 |
| `ci95` | 0.049 |

## Decision

Permit inferred paths only on previously seen hosts.

## Limitations

- Too rare for a pass-rate claim; can permit occasional crawling.
