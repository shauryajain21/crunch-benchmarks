# Experiment register

Every recoverable material Crunch run is represented below, including invalid, rejected, superseded, diagnostic, incomplete, and blocked work. Related arms are grouped when they answered one decision. Raw customer prompts, answers, secrets, and large logs are deliberately absent.

The machine-readable record is [`benchmarks.json`](../benchmarks.json). Scores below are compact summaries; limitations and provenance live in that record.

| Date | ID | Type | Status | Result / disposition |
|---|---|---|---|---|
| 2026-08-17 | `vendor-replay-400` | vendor_probe | **diagnostic** | Motivated owning the search loop; vendor results are supporting evidence, not Crunch claims. |
| 2026-08-17 | `parallel-core2x-400` | vendor_probe | **incomplete** | Do not use as a quality result. |
| 2026-08-17 | `vendor-fast-variants` | vendor_probe | **blocked** | Not run; archive as blocked. |
| 2026-08-18 | `planner-diverse-100` | planner | **diagnostic** | Prompt defaults, not only model choice, needed repair; superseded the narrow n=20 planner impression. |
| 2026-08-19 | `model-bakeoff-20` | model_selection | **adopted** | Use gemini-3.7-flash with no deadline; Luna remains latency-relaxed runner-up. |
| 2026-09-02 | `model-hunt-25` | model_selection | **adopted** | Keep gemini-3.7-flash: no measured arm was cheaper, faster, and better. |
| 2026-09-02 | `native-cheap-models` | model_selection | **blocked** | Leave as an optional offline-tier question. |
| 2026-08-19 | `hard-bench-20` | benchmark | **canonical** | Treat Crunch and Exa as tied; matrix questions remain Crunch's clearest gap. |
| 2026-08-19 | `prod100-initial` | benchmark | **invalid** | Retain only as evidence that led to harness repairs. |
| 2026-08-21 | `citation-renumber-rescore` | reliability | **adopted** | Renumber citations after selecting emitted sources. |
| 2026-08-21 | `judge-clip-1400` | measurement | **invalid** | Invalidate pre-21-Aug cross-system magnitudes and remove asymmetric clipping. |
| 2026-08-21 | `prompt-v6-v7` | prompt | **adopted** | Adopt v7 for mechanism and reduced omissions, not as a proven pass-rate win. |
| 2026-08-21 | `synthesis-writers-24` | architecture | **rejected** | Keep one model for loop and writing; improve the contract instead. |
| 2026-08-21 | `coverage-pass-100` | architecture | **adopted** | Keep for honest omission reduction; do not claim its task-prompt headline as quality gain. |
| 2026-08-21 | `scrape-top-100` | fetching | **rejected** | Disable automatic top-page scraping. |
| 2026-08-20 | `scrape-top-20` | fetching | **superseded** | Initially promising but superseded by the powered n=100 rejection. |
| 2026-08-21 | `fetch-rule-100` | fetching | **diagnostic** | Keep conservative default for production; use read behavior only for analytical work. |
| 2026-08-20 | `autoread-ranking-103` | fetching | **rejected** | Leave index order as default; mechanism changed pages but not measured quality. |
| 2026-08-20 | `host-fetch-guard` | reliability | **adopted** | Permit inferred paths only on previously seen hosts. |
| 2026-08-20 | `domain-scope-108` | filters | **adopted** | Keep explicit user domain constraints hard. |
| 2026-08-20 | `brave-second-index-100` | retrieval | **rejected** | Do not add retrieval breadth when the evidence-read budget is the bottleneck. |
| 2026-08-21 | `toolbox-caps-25` | reliability | **adopted** | Cap results at 10 and fetched page text at 12k. |
| 2026-08-21 | `prod100-corrected` | benchmark | **canonical** | Canonical internal-harness configuration; quality is tied with Exa on ordinary lookups. |
| 2026-08-24 | `deepsearchqa-896` | benchmark | **canonical** | Use as evidence on exhaustive multi-step questions. |
| 2026-08-24 | `production-1000-legacy` | benchmark | **superseded** | Historical production result; use the stratified 2,000-row run for current claims. |
| 2026-08-24 | `structured-projection-50` | structured | **adopted** | Include the question and validate output; projection should preserve prose quality. |
| 2026-08-24 | `search-rank-rff-50` | search_results | **superseded** | Replace concatenation with a real ranking; later superseded by model ranking. |
| 2026-08-24 | `search-rank-score-100` | search_results | **superseded** | Evidence that arrival order was wrong; raw score was later found incomparable across rewritten queries. |
| 2026-08-24 | `search-head-to-head-97` | search_results | **diagnostic** | Treat as tie; ranking mechanism required a production typed-output judge. |
| 2026-08-25 | `chain-prompt-10` | prompt | **rejected** | Preserve guess-and-verify; do not spend a turn sequencing guessable entities. |
| 2026-09-01 | `production-850-clip1800` | measurement | **invalid** | Invalidate the 61% number; raise clip to 5,000 or do not truncate. |
| 2026-09-01 | `search-max-score-364` | search_results | **invalid** | Do not compare raw reranker scores across rewritten queries. |
| 2026-09-01 | `search-model-rerank-364` | search_results | **adopted** | Use the model to choose across sub-query result lists. |
| 2026-09-01 | `search-split-finish-200` | architecture | **adopted** | End searchResults with a done call instead of writing discarded prose. |
| 2026-09-01 | `runner-status-bug` | measurement | **invalid** | Derive success from typed output, not presence of prose. |
| 2026-09-01 | `runner-row1520-crash` | reliability | **incomplete** | Make each row failure-isolated and resume-safe. |
| 2026-09-01 | `production-2000-sourced` | benchmark | **canonical** | Canonical sourcedAnswer production result. |
| 2026-09-01 | `production-2000-search` | benchmark | **canonical** | Canonical unfiltered searchResults result. |
| 2026-09-01 | `production-2000-structured-shipping` | structured | **superseded** | Do not use as current structured quality; it exposed schema coercion defects. |
| 2026-09-02 | `structured-projector-rescore-319` | structured | **superseded** | Adopt projector repairs; use direct-projection confirmation for the newest architecture decision. |
| 2026-09-01 | `structured-abstention-194` | structured | **rejected** | Do not match Deep's null rate mechanically. |
| 2026-09-01 | `search-literal-40` | architecture | **rejected** | Keep the adaptive loop. |
| 2026-09-01 | `forced-gather-hops-40` | stopping | **rejected** | Let the model decide when to stop; more forced searches were not better. |
| 2026-09-01 | `vespa-general-25` | retrieval | **rejected** | Do not remove external long-tail retrieval for general traffic. |
| 2026-09-01 | `gather-to-schema-319` | structured | **rejected** | Do not skip prose unless evidence is ranked and preserved for the projector. |
| 2026-09-02 | `enterprise-toolbox-100` | benchmark | **diagnostic** | Strong customer-specific evidence; not representative enough for global claim. |
| 2026-09-02 | `enterprise-vespa-100` | benchmark | **diagnostic** | Proceed to larger Vespa confirmation. |
| 2026-09-02 | `enterprise-toolbox-v-vespa-100` | retrieval | **diagnostic** | Treat retrieval arms as tied. |
| 2026-09-02 | `enterprise-vespa-1000` | benchmark | **canonical** | Canonical enterprise-cohort result; proves Vespa-only viability on this traffic. |
| 2026-09-03 | `filtered-production-200-v0` | filters | **invalid** | Invalidate v0 and constrain model-written site dorks. |
| 2026-09-03 | `filtered-production-200` | benchmark | **diagnostic** | Retain immutable filters, but do not ship unchanged: filtered searchResults and empty-result handling regress. |
| 2026-09-03 | `schema-guidance-40` | structured | **rejected** | Do not ship broad guidance alone: completion rose but correctness tied and latency increased. |
| 2026-09-03 | `schema-factorial-40` | structured | **diagnostic** | Direct ranked-evidence projection is promising; broad guidance itself remains unproven. |
| 2026-09-03 | `schema-guided-direct-300` | structured | **adopted** | Adopt direct ranked-evidence projection as the established lever; narrow guidance before production rollout. |
| 2026-09-03 | `firsthop-ab-100` | stopping | **diagnostic** | Forced 3-search first hop 41–24–35 vs free first turn; 40% swap flips, no shipped change. |

## Scope rules

- Vendor-only records are included only where they motivated Crunch, selected a comparator, or exposed a harness contract.
- A grouped record can represent repeated arms or order-swapped grades of the same sample; it does not pretend those repeats are independent.
- “Not run” ideas are omitted unless work was explicitly blocked or an incomplete artifact could otherwise be mistaken for a result.
- Local-only artifacts are cited by path or commit, but their raw rows are not copied here.
