# Stratified production-2000

**Name:** Stratified production replay  
**Date:** 2026-09-01  
**Parent n:** 2,000 (mode-stratified)  
**Arms:** Crunch vs frozen Linkup Deep  
**Judge:** Blind typed pairwise Luna

Machine-readable IDs in [`benchmarks.json`](../../benchmarks.json): `production-2000-sourced`, `production-2000-search`, `production-2000-structured-shipping`.

## Mix

| Mode | n | Catalog id | Status |
|---|---:|---|---|
| sourcedAnswer | 687 | `production-2000-sourced` | canonical |
| searchResults | 857 | `production-2000-search` | canonical |
| structured (shipping) | 456 | `production-2000-structured-shipping` | superseded |

## sourcedAnswer — `production-2000-sourced`

- **578–108–1**; **84.3%** decided (95% CI 80.5–88.0%)
- Judge: 5,000-character clip; 60 mode swaps
- Canonical sourcedAnswer production result
- Supersedes `production-1000-legacy`, `production-850-clip1800`

## searchResults — `production-2000-search`

- **622–233–2**; **72.8%** decided (95% CI 69.4–76.1%)
- Judge: complete result lists
- Canonical unfiltered searchResults result
- Supersedes `search-model-rerank-364`, `search-split-finish-200`

## Structured shipping — `production-2000-structured-shipping`

- **221–219–16**; **50.2%** decided
- Near-tie; organization-specific format failures dominated
- Not current structured quality. Later evidence: `structured-projector-rescore-319`, `schema-guided-direct-300`

## Caveats

- Frozen Deep replay dropped date and domain filters.
- Structured shipping is superseded; do not use it as current structured quality.
- This archive keeps aggregates only. Raw customer queries and judge jsonl are excluded.
