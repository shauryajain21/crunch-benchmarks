# Eval writeups

One page per catalog id. Shared campaigns keep a single file. Raw prompts, answers, and logs stay out of git.

| ID | n | Status | Result |
|---|---:|---|---|
| [`vendor-replay-400`](vendor-replay-400.md) | 400 | diagnostic | Motivated owning the search loop; vendor results are supporting evidence, not Crunch claims. |
| [`parallel-core2x-400`](parallel-core2x-400.md) | target 400; completed 61 | incomplete | Do not use as a quality result. |
| [`vendor-fast-variants`](vendor-fast-variants.md) | target 400 | blocked | Not run; archive as blocked. |
| [`planner-diverse-100`](planner-diverse-100.md) | 100 (comparable_n=98) | diagnostic | Prompt defaults, not only model choice, needed repair; superseded the narrow n=20 plann… |
| [`model-bakeoff-20`](model-bakeoff-20.md) | 20 | adopted | Use gemini-3.7-flash with no deadline; Luna remains latency-relaxed runner-up. |
| [`model-hunt-25`](model-hunt-25.md) | 25 | adopted | Keep gemini-3.7-flash: no measured arm was cheaper, faster, and better. |
| [`native-cheap-models`](native-cheap-models.md) | target 25 | blocked | Leave as an optional offline-tier question. |
| [`hard-bench-20`](hard-bench-20.md) | 20 (completed_n=17) | canonical | Treat Crunch and Exa as tied; matrix questions remain Crunch's clearest gap. |
| [`prod100-initial`](prod100-initial.md) | 100 | invalid | Retain only as evidence that led to harness repairs. |
| [`citation-renumber-rescore`](citation-renumber-rescore.md) | 100 | adopted | Renumber citations after selecting emitted sources. |
| [`judge-clip-1400`](judge-clip-1400.md) | 100 | invalid | Invalidate pre-21-Aug cross-system magnitudes and remove asymmetric clipping. |
| [`prompt-v6-v7`](prompt-v6-v7.md) | 100 | adopted | Adopt v7 for mechanism and reduced omissions, not as a proven pass-rate win. |
| [`synthesis-writers-24`](synthesis-writers-24.md) | 24 | rejected | Keep one model for loop and writing; improve the contract instead. |
| [`coverage-pass-100`](coverage-pass-100.md) | 100 | adopted | Keep for honest omission reduction; do not claim its task-prompt headline as quality gain. |
| [`scrape-top-100`](scrape-top-100.md) | 100 | rejected | Disable automatic top-page scraping. |
| [`scrape-top-20`](scrape-top-20.md) | 20 | superseded | Initially promising but superseded by the powered n=100 rejection. |
| [`fetch-rule-100`](fetch-rule-100.md) | production_n=100, hard_n=20 | diagnostic | Keep conservative default for production; use read behavior only for analytical work. |
| [`autoread-ranking-103`](autoread-ranking-103.md) | 103 | rejected | Leave index order as default; mechanism changed pages but not measured quality. |
| [`host-fetch-guard`](host-fetch-guard.md) | ~103 | adopted | Permit inferred paths only on previously seen hosts. |
| [`domain-scope-108`](domain-scope-108.md) | asker_dork_n=52, metadata_domain_n=56 | adopted | Keep explicit user domain constraints hard. |
| [`brave-second-index-100`](brave-second-index-100.md) | 100 | rejected | Do not add retrieval breadth when the evidence-read budget is the bottleneck. |
| [`toolbox-caps-25`](toolbox-caps-25.md) | 25 | adopted | Cap results at 10 and fetched page text at 12k. |
| [`prod100-corrected`](prod100-corrected.md) | 100 | canonical | Canonical internal-harness configuration; quality is tied with Exa on ordinary lookups. |
| [`deepsearchqa-896`](deepsearchqa-896.md) | 896 | canonical | Use as evidence on exhaustive multi-step questions. |
| [`production-1000-legacy`](production-1000-legacy.md) | 1000 (comparable_n=615) | superseded | Historical production result; use the stratified 2,000-row run for current claims. |
| [`structured-projection-50`](structured-projection-50.md) | 50 | adopted | Include the question and validate output; projection should preserve prose quality. |
| [`search-rank-rff-50`](search-rank-rff-50.md) | 50 | superseded | Replace concatenation with a real ranking; later superseded by model ranking. |
| [`search-rank-score-100`](search-rank-score-100.md) | 100 | superseded | Evidence that arrival order was wrong; raw score was later found incomparable across re… |
| [`search-head-to-head-97`](search-head-to-head-97.md) | decisions_n=97 | diagnostic | Treat as tie; ranking mechanism required a production typed-output judge. |
| [`chain-prompt-10`](chain-prompt-10.md) | 10 | rejected | Preserve guess-and-verify; do not spend a turn sequencing guessable entities. |
| [`production-850-clip1800`](production-850-clip1800.md) | 292 (mode=sourcedAnswer) | invalid | Invalidate the 61% number; raise clip to 5,000 or do not truncate. |
| [`search-max-score-364`](search-max-score-364.md) | 364 | invalid | Do not compare raw reranker scores across rewritten queries. |
| [`search-model-rerank-364`](search-model-rerank-364.md) | 364 | adopted | Use the model to choose across sub-query result lists. |
| [`search-split-finish-200`](search-split-finish-200.md) | 200 | adopted | End searchResults with a done call instead of writing discarded prose. |
| [`runner-status-bug`](runner-status-bug.md) | affected_successes=845 | invalid | Derive success from typed output, not presence of prose. |
| [`runner-row1520-crash`](runner-row1520-crash.md) | target 2000; stopped at 1520 | incomplete | Make each row failure-isolated and resume-safe. |
| [`production-2000-sourced`](production-2000.md) | 687 (parent_n=2000) | canonical | Canonical sourcedAnswer production result. |
| [`production-2000-search`](production-2000.md) | 857 (parent_n=2000) | canonical | Canonical unfiltered searchResults result. |
| [`production-2000-structured-shipping`](production-2000.md) | 456 (parent_n=2000) | superseded | Do not use as current structured quality; it exposed schema coercion defects. |
| [`structured-projector-rescore-319`](structured-projector-rescore-319.md) | 319 | superseded | Adopt projector repairs; use direct-projection confirmation for the newest architecture… |
| [`structured-abstention-194`](structured-abstention-194.md) | 194 | rejected | Do not match Deep's null rate mechanically. |
| [`search-literal-40`](search-literal-40.md) | 40 | rejected | Keep the adaptive loop. |
| [`forced-gather-hops-40`](forced-gather-hops-40.md) | 40 | rejected | Let the model decide when to stop; more forced searches were not better. |
| [`vespa-general-25`](vespa-general-25.md) | 25 | rejected | Do not remove external long-tail retrieval for general traffic. |
| [`gather-to-schema-319`](gather-to-schema-319.md) | 319 | rejected | Do not skip prose unless evidence is ranked and preserved for the projector. |
| [`enterprise-toolbox-100`](enterprise-toolbox-100.md) | 100 (mode=sourcedAnswer) | diagnostic | Strong customer-specific evidence; not representative enough for global claim. |
| [`enterprise-vespa-100`](enterprise-vespa-100.md) | 100 (mode=sourcedAnswer) | diagnostic | Proceed to larger Vespa confirmation. |
| [`enterprise-toolbox-v-vespa-100`](enterprise-toolbox-v-vespa-100.md) | 100 | diagnostic | Treat retrieval arms as tied. |
| [`enterprise-vespa-1000`](enterprise-vespa-1000.md) | 1000 (mode=sourcedAnswer) | canonical | Canonical enterprise-cohort result; proves Vespa-only viability on this traffic. |
| [`filtered-production-200-v0`](filtered-production-200-v0.md) | 200 | invalid | Invalidate v0 and constrain model-written site dorks. |
| [`filtered-production-200`](filtered-production-200.md) | 200 (population=405462) | diagnostic | Retain immutable filters, but do not ship unchanged: filtered searchResults and empty-r… |
| [`schema-guidance-40`](schema-guidance-40.md) | 40 | rejected | Do not ship broad guidance alone: completion rose but correctness tied and latency incr… |
| [`schema-factorial-40`](schema-factorial-40.md) | 40 | diagnostic | Direct ranked-evidence projection is promising; broad guidance itself remains unproven. |
| [`schema-guided-direct-300`](schema-guided-direct-300.md) | 300 | adopted | Adopt direct ranked-evidence projection as the established lever; narrow guidance befor… |
| [`firsthop-ab-100`](firsthop-ab-100.md) | 100 (mode=sourcedAnswer) | diagnostic | Keep the shipped first-hop rule. The quality lean does not beat 40% swap flips, and med… |
| [`model-eval-200`](model-eval-200.md) | 200 | diagnostic | Keep Gemini 3.7 Flash. GLM 117–53–30; DeepSeek 66–106–28 after resume to 151/200. Swap … |
| [`claim-verify-100`](claim-verify-100.md) | 100 (graded_n=90) | rejected | Leave verify off. Lexical unsupported flags did not match the judge; the pass added lat… |
| [`vespa-prod100`](vespa-prod100.md) | 100 | diagnostic | Vespa-only drops about 5 points on ordinary production lookups versus Toolbox. Not a co… |
| [`vespa-hard-20`](vespa-hard-20.md) | 20 | diagnostic | Hard-set drop is larger than prod-100 and concentrated in matrix, numeric, and temporal… |
| [`vespa-dsqa-898`](vespa-dsqa-898.md) | 898 | diagnostic | Vespa-only is about 7 points worse, faster, and more expensive (longer snippets). Use a… |
| [`vespa-dsqa-450-h16`](vespa-dsqa-450-h16.md) | 450 | diagnostic | Extra hops mostly stopped forced writes. Quality +0.9 vs vespa 8-hop is noise and does … |
| [`dsqa-16hop-50`](dsqa-16hop-50.md) | 50 | diagnostic | 16 hops raised all-correct 40%→46% on n=50 and almost removed forced writes. CI is wide… |
| [`vespa-prod1000`](vespa-prod1000.md) | target 1000 | blocked | Not run; Infisical/Columbus died before start. Do not invent a score. |
| [`judge-max-tokens-180`](judge-max-tokens-180.md) | campaign=enterprise-toolbox-100, regraded=True | invalid | Do not use any mid-run ties from the 180-token clip. The published Sapiom-100 scores ar… |
