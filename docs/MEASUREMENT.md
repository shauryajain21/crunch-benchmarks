# Measurement and supersession guide

## Score contracts
- **Decided win rate:** wins / (wins + losses). Always show ties separately.
- **Row win rate:** wins / all sampled rows, including ties.
- **Checklist pass:** fraction of predeclared atomic requirements passed; supported, unsourced, and missing are separate.
- **All-correct:** every expected answer appears with no excessive answer.
- **Typed pairwise:** the judge sees the output-mode contract. Structured judges must see the exact schema; filtered judges must see immutable request constraints.

## Required controls
1. Pin identical questions and randomize anonymous answer order.
2. Keep failures in the denominator and report completion separately.
3. Repeat an order-swapped sample. A margin smaller than observed instability is a tie.
4. Avoid asymmetric clipping. Prefer no answer truncation; otherwise verify arm-level truncation rates.
5. Score within one judge pass. Do not compare absolute scores from different extraction passes as if paired.
6. Validate citation URLs in code; do not let prose or source-list length substitute for support.
7. Separate fresh runs, re-projections, and re-judges. They answer different causal questions.

## Known invalid instruments
- Anthropic `max_tokens=180` clipped the Sapiom-100 judge JSON and invented ties; those rows were regraded at 1024.
- The original production-100 checklist result had unresolved citation IDs and a 1,400-character clip.
- The 850 sourcedAnswer judge clipped at 1,800 characters, cutting 75% of Crunch and 20% of Deep.
- Split-finish rows were falsely failed when status was derived from a non-existent prose answer.
- A finish-reason-only response crashed the first 2,000-row campaign at row 1,520.
- Raw cross-encoder maxima were compared across independently rewritten sub-queries even though their scales were not comparable.
- Early incomplete 503/timeout runs retained fast, shallow survivors and are not quality samples.

## Canonical supersession map
- `prod100-initial` → citation and clip repairs → `prod100-corrected`.
- `production-1000-legacy` → `production-2000-sourced` and `production-2000-search`.
- `production-850-clip1800` → `production-2000-sourced`.
- `search-rank-rff-50` → `search-rank-score-100` → invalidated by `search-max-score-364` → `search-model-rerank-364` → `production-2000-search`.
- `production-2000-structured-shipping` → selected `structured-projector-rescore-319` → architecture evidence in `schema-factorial-40` and `schema-guided-direct-300`.
- `schema-guidance-40` rejects broad guidance alone; it is not contradicted by `schema-guided-direct-300`, whose established mechanism is direct evidence preservation.
- `enterprise-vespa-100` → `enterprise-vespa-1000` for that enterprise cohort. It does not supersede the rejected general-web Vespa pilot because the populations differ.
- `filtered-production-200-v0` → `filtered-production-200`; the latter is diagnostic, not launch evidence.

## Comparability warnings
- The structured 69.8% score is a selected 319-row re-projection, not a full fresh retrieval run.
- The newer 300-row structured judge saw the exact schema and is not numerically interchangeable with the old Luna scoreboard.
- The enterprise benchmark is a single-customer hash sample and not volume weighted.
- DeepSearchQA uses a replacement autorater and supports paired comparison only.
- Customer and production traffic are private samples; this archive preserves aggregates, not prompts.
