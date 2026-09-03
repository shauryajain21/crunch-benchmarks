# Crunch runtime harness: design and evidence

This document covers only the runtime harness around the model and the evaluations that shaped it. It is not a general Crunch guide and does not describe unrelated product architecture.

## What “harness” means

Here, **harness** means the production runtime shell that turns one question into a typed response. It parses the request, exposes tools, runs the model loop, preserves evidence, applies deterministic checks, and shapes the requested output.

It does **not** mean the benchmark harness. The benchmark harness samples questions, runs arms, records failures, and judges outputs; it measures the runtime harness but is not part of the serving path.

## Current flow

`question → one model with search/fetch → adaptive loop → write → coverage → citation/URL cleanup → output shaping`

1. Parse the question, output mode or schema, and immutable request filters.
2. Run one reasoning model with typed `search`, `fetch`, and finish tools; require a first search and allow parallel rewrites.
3. Let that model gather, inspect evidence, and decide whether to search, fetch, or stop.
4. For prose, have the same model write under the answer contract.
5. Run a bounded coverage check for omitted or unsupported requested items.
6. Select emitted sources, renumber citations, repair placement, and remove URLs not grounded in retrieved or fetched evidence.
7. Shape the result as `sourcedAnswer`, ranked `searchResults`, or validated structured output. Structured output now projects ranked evidence directly rather than requiring intermediate prose.

## Why the harness has this form

### Own the loop instead of nesting an agent API or framework

- **Function:** The shell directly controls tool exposure, evidence IDs, context caps, stopping, latency, failure isolation, and output contracts.
- **Decision:** Keep an owned model/tool loop. Do not add a nested agent API or framework layer whose internal retrieval and stopping behavior cannot be enforced by this shell.
- **Evidence:** `vendor-replay-400` (diagnostic, n=400) motivated control of the loop; it predates Crunch and is not a Crunch quality claim. `search-literal-40` (rejected, n=40) later showed that replacing the loop with one literal query reached only 34.4% decided versus frozen Deep.

### One model gathers and writes

- **Function:** The loop model retains the question, query choices, fetched evidence, and uncertainty through synthesis.
- **Decision:** Use the same model for gathering and prose; improve its answer contract instead of handing evidence to a separate writer.
- **Evidence:** `synthesis-writers-24` (rejected, n=24) scored loop `.746`, Gemini Pro `.750`, and Sonnet `.654`. The **4.8×** figure is Pro’s model-cost multiple for this writer arm, not a 4.8× end-to-end quality or production-cost claim; Sonnet was 16×.

### Expose only search and fetch as evidence tools

- **Function:** `search` discovers and rewrites; `fetch` reads a model-selected page. Stable evidence IDs connect both to citations and typed results.
- **Decision:** Keep the evidence surface narrow, cap search results and page text, fetch conservatively, and avoid automatic top-page scraping.
- **Evidence:** `scrape-top-100` (rejected, n=100) scored `.9295` with forced scraping versus `.9403` without and raised median latency from 9.5s to 30.5s. `toolbox-caps-25` (adopted, n=25) improved answered rows from 16/25 to 24/24 with 10-result and 12k-character caps.

### Make post-processing explicit

- **Function:** Deterministic code performs coverage, citation renumbering, URL allowlisting, filter enforcement, schema validation, and mode-specific finishing.
- **Decision:** Treat these as serving invariants, not prompt suggestions or judge-side repairs.
- **Evidence:** `citation-renumber-rescore` (adopted, n=100) moved identical answers from `.571` to `.748`, with 30 better and 0 worse, by fixing emitted citation IDs. `runner-status-bug` (invalid, 845 affected successes) showed why typed output—not prose presence—must determine success.

### Keep retrieval and model adapters swappable

- **Function:** Retrieval and model calls sit behind stable search/fetch and typed-finish contracts, so an arm can change without rewriting the loop or output validation.
- **Decision:** Keep adapters replaceable, but select them by traffic and measured behavior; general traffic retains external long-tail retrieval, while Vespa-only is viable for the measured enterprise cohort.
- **Evidence:** `enterprise-toolbox-v-vespa-100` (diagnostic, n=100) found the retrieval arms indistinguishable under 53% swap flips. `vespa-general-25` (rejected, n=25) reached only 20.8% decided, while `enterprise-vespa-1000` (canonical, n=1,000) reached 85.1% decided on its bounded cohort.

## Evaluation inventory

Statuses use the catalog meanings: **canonical** is current headline evidence for its scope; **adopted** or **rejected** records a design choice; **superseded** was replaced; **invalid** has an instrument or implementation defect; **diagnostic** is bounded mechanism evidence; **incomplete** did not produce a scoreable sample; **blocked** did not run.

Every ID below resolves in [`benchmarks.json`](benchmarks.json); each entry also gives the cataloged source path.

### Overall end-to-end quality

- **`vendor-replay-400` — diagnostic; n=400.** Arms: Linkup Deep/Standard, Exa variants, Parallel Core. Flash checklist pass ranged from Deep `.352` to Exa Reasoning `.820`; motivation only, not Crunch evidence. Source: `Deep-eval:docs/experiment.md`.
- **`parallel-core2x-400` — incomplete; target n=400, completed 61, 338 payment errors.** Arm: Parallel Core2x. Non-random survivors are not a quality result. Source: `Deep-eval:docs/experiment.md`.
- **`vendor-fast-variants` — blocked; target n=400.** Arms: Parallel Core Fast, Core2x Fast. No score; provider credits blocked the replay. Source: `Deep-eval:docs/experiment.md`.
- **`hard-bench-20` — canonical; n=20, completed n=17.** Arms: Crunch, Exa Deep Reasoning, Linkup Deep. Crunch–Exa delta `-.028 ± .151`; treat as tied, with matrix questions the clearest gap. Sources: `crunch-public:BENCHMARKS.md`; `Deep-eval:docs/hard-bench.md`.
- **`prod100-initial` — invalid; n=100.** Arms: Crunch, Exa, Linkup Deep. Reported pass `.577/.781/.572`; unresolved citation IDs and asymmetric 1,400-character clipping invalidate magnitudes. Source: `Deep-eval:STATUS.md`.
- **`prod100-corrected` — canonical; n=100.** Arms: Crunch v7, Exa Deep Reasoning, Linkup Deep. Pass `.9403/.894/.610`, Crunch p50/p90 9.5s/14.3s; canonical internal configuration, tied with Exa on ordinary lookups. Sources: `crunch-public:BENCHMARKS.md`; `Deep-eval:CRUNCH.md`.
- **`deepsearchqa-896` — canonical; n=896.** Arms: Crunch, Production Deep. All-correct `.362` vs `.084`, 277 better/28 worse/591 tied; evidence for exhaustive multi-step questions only. Source: `crunch-public:BENCHMARKS.md`.
- **`production-1000-legacy` — superseded; n=1,000, comparable n=615.** Arms: Crunch, Production Deep. 512–96, 84.2% decided; replaced by the stratified 2,000-row campaign. Source: `crunch-public:BENCHMARKS.md`.
- **`production-2000-sourced` — canonical; n=687 from parent n=2,000.** Arms: Crunch, Frozen Linkup Deep. **578–108–1, 84.3% decided** (exact `.8426`, 95% CI `.8051–.8800`); canonical `sourcedAnswer` result. Source: `crunch-public:evals/bench-2000-20260901/RESULTS.md`.
- **`production-2000-search` — canonical; n=857 from parent n=2,000.** Arms: Crunch, Frozen Linkup Deep. 622–233–2, 72.8% decided; canonical unfiltered `searchResults` result. Source: `crunch-public:evals/bench-2000-20260901/RESULTS.md`.
- **`enterprise-toolbox-100` — diagnostic; n=100.** Arms: Toolbox Crunch, Stored Deep. 81–9–10, 90.0% decided; strong single-cohort evidence, not a global claim. Source: `crunch-public:evals/sapiom-deep-100-20260902/RESULTS.md`.
- **`enterprise-vespa-100` — diagnostic; n=100.** Arms: Vespa-only Crunch, Stored Deep. 77–13–10, 85.6% decided; proceed to larger confirmation. Source: `crunch-public:evals/sapiom-deep-100-20260902/RESULTS.md`.
- **`enterprise-vespa-1000` — canonical; n=1,000.** Arms: Vespa-only Crunch, Stored Deep. 821–144–35, 85.1% decided, p50 12.3s; proves viability only on this enterprise cohort. Source: `crunch-public:evals/sapiom-deep-1000-20260902/RESULTS.md`.

### Separate writer / synthesis

- **`synthesis-writers-24` — rejected; n=24.** Arms: loop writer, Gemini Pro writer, Claude Sonnet writer. Pass `.746/.750/.654`; Pro’s negligible `+.004` came at **4.8× model cost** and Sonnet at 16×, so keep one gather-and-write model. Source: `Deep-eval:STATUS.md`.

This is distinct from `production-2000-sourced`: the latter is an end-to-end n=687 Crunch-versus-Deep production comparison (578–108–1, 84.3%), not evidence that a separate writer improved the harness.

### Forced search and stopping

- **`search-literal-40` — rejected; n=40.** Arms: one query, Frozen Deep. One-query Crunch reached 34.4% decided; retain the adaptive loop. Source: `crunch-public:evals/RESULTS.md`.
- **`forced-gather-hops-40` — rejected; n=40.** Arms: forced one hop, forced two hops, each compared with frozen Deep. Decided rates 62.5% and 65.0%; extra forced search was not better than adaptive stopping. Source: `crunch-public:evals/RESULTS.md`.

### Prompt and answer contract

- **`planner-diverse-100` — diagnostic; n=100, comparable n=98.** Arms: GPT-4.1 mini, GPT-5.6 Luna, DeepSeek V4 Flash. Average tool calls `1.78/1.92/1.70`; malformed dorks `22/17/0`; repair prompt defaults rather than infer a pure model effect. Source: `Deep-eval:docs/experiment.md`.
- **`prompt-v6-v7` — adopted; n=100, targeted subset n=24.** Arms: v3, v6, v7. v3→v7 was `+.012 ± .041`, while missing items fell `.258→.167`; adopt v7 for mechanism and omissions, not a proven pass-rate win. Source: `Deep-eval:STATUS.md`.
- **`chain-prompt-10` — rejected; n=10.** Arms: stock prompt, sequential-chain bullet. Both passed 10/10 and all 10 tied; preserve guess-and-verify, with unknown intermediates still untested. Source: `crunch-public:behaviours.md`.

### Coverage pass

- **`coverage-pass-100` — adopted; n=100.** Arms: v7, v7 plus coverage. Overall `+.019 ± .029`, lookup delta `0`, missing `.045→.033`; keep for honest omission reduction, not the agentic headline affected by citation laundering. Source: `Deep-eval:STATUS.md`.

### Fetch and scraping

- **`scrape-top-20` — superseded; n=20.** Arms: Toolbox snippets, five scrapes per hop. Human read: 8 better/1 worse/11 same, p50 6.35s→31.515s; promising pilot replaced by n=100. Source: `Deep-eval:STATUS.md`.
- **`scrape-top-100` — rejected; n=100.** Arms: `scrape_top=3`, `scrape_top=0`. Pass `.9295` vs `.9403`, p50 30.5s vs 9.5s; disable automatic scraping while retaining model-selected fetch. Source: `Deep-eval:STATUS.md`.
- **`fetch-rule-100` — diagnostic; production n=100, hard n=20.** Arms: “tease”, “read”. Production delta `-.008 ± .024`; hard delta `+.043 ± .084`; conservative production default, eager reading only for analytical work. Source: `Deep-eval:STATUS.md`.
- **`autoread-ranking-103` — rejected; n=103, two replicates per arm.** Arms: index order, intent-ranked order. Delta `-.011 ± .044`; changed selected pages but not quality, so leave index order. Source: `Deep-eval:STATUS.md`.
- **`host-fetch-guard` — adopted; 2 runs, 15 novel-path fetches.** Arms: exact-URL guard, seen-host guard. Text returned and cited on 14/15, quality delta `-.010 ± .049`; permit inferred paths only on seen hosts. Source: `Deep-eval:STATUS.md`.

### Retrieval backend and width

- **`brave-second-index-100` — rejected; n=100, treated n=55.** Arms: Toolbox only, Toolbox+Brave. Difference-in-differences `-.042 ± .058`; extra candidates did not overcome the read budget. Source: `Deep-eval:STATUS.md`.
- **`vespa-general-25` — rejected; n=25.** Arms: Vespa-only Crunch, Frozen Deep. 20.8% decided; retain external long-tail retrieval for general traffic. Source: `crunch-public:evals/RESULTS.md`.
- **`enterprise-toolbox-v-vespa-100` — diagnostic; n=100.** Arms: Toolbox Crunch, Vespa-only Crunch. 42–35–23 with 53% swap flips; treat as tied on this cohort. Source: `crunch-public:evals/sapiom-deep-100-20260902/RESULTS.md`.

### Citations and URL safety

- **`citation-renumber-rescore` — adopted; n=100.** Arms: broken emitted IDs, renumbered emitted IDs. `.571→.748`, 30 better/0 worse; citation IDs must index emitted sources. Source: `Deep-eval:STATUS.md`.
- **`host-fetch-guard` — adopted; 2 runs, 15 novel paths.** Arms and result as above; the seen-host rule is the only measured allowance for inferred URLs. Source: `Deep-eval:STATUS.md`.

### Output modes, ranking, and projection

- **`structured-projection-50` — adopted; n=50.** Arms: old projector, question-aware validated projector, prose ceiling. Strict `.600/.620/.620`, recall `.717`; include the question and validate once. Source: `Deep-eval:results/output_modes/README.md`.
- **`search-rank-rff-50` — superseded; n=50.** Arms: arrival order, reciprocal-rank fusion. Top-10 recall `.355→.521`; proved concatenation inadequate, later replaced by model ranking. Source: `Deep-eval:results/output_modes/README.md`.
- **`search-rank-score-100` — superseded; n=100.** Arms: arrival, RRF, raw score. Top-10 recall `.330/.517/.566`; raw cross-query scores were later invalidated as incomparable. Source: `crunch-public:OUTPUT-MODES.md`.
- **`search-head-to-head-97` — diagnostic; 97 decisions.** Arms: Crunch ranked top 10, Deep top 10. 53–40–4 with 23% swap flips; margin was below instability, so treat as tied. Source: `crunch-public:OUTPUT-MODES.md`.
- **`search-max-score-364` — invalid; n=364.** Arm: maximum raw cross-encoder score. Reported win rate 45.2%; narrow rewrites dominated due to scale mismatch, so do not compare these scores across queries. Source: `crunch-public:evals/RESULTS.md`.
- **`search-model-rerank-364` — adopted; n=364.** Arms: model reranking, Frozen Deep. 277–87–0, 76.1% decided; let the model rank across rewritten-query lists. Source: `crunch-public:evals/RESULTS.md`.
- **`search-split-finish-200` — adopted; n=200.** Arms: write then discard prose, typed `done`. 152–48 with quality p=`.67`; latency 44.1s→8.4s, so finish search results without prose. Source: `crunch-public:evals/RESULTS.md`.
- **`production-2000-structured-shipping` — superseded; n=456 from parent n=2,000.** Arms: shipping Crunch projector, Frozen Deep. 221–219–16, 50.2% decided; exposed schema coercion defects and is not current structured quality. Source: `crunch-public:evals/bench-2000-20260901/RESULTS.md`.
- **`structured-projector-rescore-319` — superseded; n=319 selected rows.** Arms: reprojected Crunch writes, Frozen Deep. 215–93–11, 69.8% decided; adopted repairs, but this was selected reprojection rather than a fresh run. Source: `crunch-public:evals/RESULTS.md`.
- **`structured-abstention-194` — rejected; n=194.** Arms: higher null rate, Frozen Deep. 42.9% decided; do not mechanically match Deep’s null rate. Source: `crunch-public:evals/RESULTS.md`.
- **`gather-to-schema-319` — rejected; n=319.** Arms: gather-to-schema, write-to-schema. Direct path reached 56.0% decided versus Deep; do not skip prose when evidence is unranked and not preserved. Source: `crunch-public:evals/RESULTS.md`.
- **`schema-guidance-40` — rejected; n=40, 882 leaf opportunities.** Arms: current write, broad schema-guided write. Nonempty `.268→.329`, stable wins 8–9, latency 22.6s→29.9s; broad guidance alone did not improve correctness. Source: `crunch-public@c1e405d:evals/schema-guidance-pilot-20260903/RESULTS.md`.
- **`schema-factorial-40` — diagnostic; n=40, 160 generation calls.** Arms: control/guided write and control/guided direct. Pooled direct-vs-write 31–20–29; direct ranked-evidence projection was promising, broad guidance remained unproven. Source: `crunch-public@8c58c5e:evals/schema-factorial-20260903/RESULTS.md`.
- **`schema-guided-direct-300` — adopted; n=300, 600 generation calls.** Arms: gather→write→project, guided gather→ranked evidence→project. 139–79–82, 63.8% decided, cost +17.3%; adopt direct evidence projection, narrow guidance before rollout. Source: `crunch-public@3f00dd1:evals/schema-ab-300-20260903/RESULTS.md`.

### Filters

- **`domain-scope-108` — adopted; n=108 (52 asker-dork, 56 metadata-domain).** Arms: hard `includeDomains`, broad host hint. Deltas `+.132` (95% CI `.040–.227`) and `+.033`; keep explicit caller domains hard. Source: `Deep-eval:STATUS.md`.
- **`filtered-production-200-v0` — invalid; n=200.** Arm: Vespa-only Crunch v0. Include-domain violations occurred before immutable-filter repair; invalidate v0. Source: `crunch-public:evals/filtered-200-20260903/RESULTS.md`.
- **`filtered-production-200` — diagnostic; n=200 across 11 strata.** Arms: Vespa-only Crunch v1, live Linkup Deep. 67–32–101, 67.7% decided; Crunch had 0 domain violations vs 33, but succeeded 192/200 and lost filtered search 6–20–49, so do not ship unchanged. Source: `crunch-public:evals/filtered-200-20260903/RESULTS.md`.

### Model selection

- **`model-bakeoff-20` — adopted; n=20.** Arms: Gemini 3.7 Flash, GPT-4.1 mini, GPT-5.6 Luna, four slower models. Gemini p50/p90 6.341s/9.03s at `.0187` per row; select Gemini, with Luna as latency-relaxed runner-up. Sources: `Deep-eval:STATUS.md`; `Deep-eval:docs/index-agent.md`.
- **`model-hunt-25` — adopted; n=25.** Arms: Gemini 3.7 Flash, Luna, Gemini Flash Lite variants, Qwen, OSS-120B, other open models. Pass `.879/.873/.692/.768` for named scored arms; no measured arm was cheaper, faster, and better than Gemini. Source: `Deep-eval:docs/model-hunt.md`.
- **`native-cheap-models` — blocked; target n=25.** Arms: Qwen 3.7 Flash native, OSS-120B native. No score; native APIs were not run, leaving only an optional offline-tier question. Source: `Deep-eval:docs/model-hunt.md`.

### Reliability

- **`toolbox-caps-25` — adopted; n=25.** Arms: 20 hits/uncapped pages, 10 hits/12k pages. Answered 16/25 before and 24/24 after; quality `+.015 ± .086` is a null, but caps remain operationally adopted. Source: `Deep-eval:STATUS.md`.
- **`runner-row1520-crash` — incomplete; target n=2,000, stopped at 1,520.** Arm: original runner. A finish-reason-only response crashed indexing of an absent message; isolate failures and make campaigns resumable. Source: `crunch-public:evals/RESULTS.md`.

### Judge and measurement corrections

- **`judge-clip-1400` — invalid; n=100.** Arms: 1,400-character clip, 16,000-character clip. Regrading shifted Crunch v7 `+.094`, Exa `+.096`, Deep `0`; invalidate pre-21-Aug cross-system magnitudes and remove asymmetric clipping. Source: `Deep-eval:STATUS.md`.
- **`production-850-clip1800` — invalid; n=292 `sourcedAnswer` rows.** Arms: Crunch, Frozen Deep. Reported 61.3% versus corrected 84.19%; the clip affected 75% of Crunch and 20% of Deep, so use 5,000 characters or no truncation. Source: `crunch-public:evals/RESULTS.md`.
- **`runner-status-bug` — invalid; 845 affected successes.** Arms: status from answer text, status from output contract. Reported success 1% versus corrected quality win rate 72.8%; derive status from typed output. Source: `crunch-public:evals/RESULTS.md`.

## Current harness decisions

- Own the adaptive loop and keep the runtime shell independent of nested agent frameworks.
- Use Gemini 3.7 Flash, low effort, with no model deadline; require a first search and let the model stop.
- Use one model for gathering and prose writing; do not add a separate synthesis model.
- Expose typed search and fetch only; cap each search at 10 results and fetched text at 12k characters.
- Keep conservative model-selected fetch, seen-host inferred paths, and no automatic top-page scraping.
- Keep v7’s answer contract and the bounded coverage pass.
- Enforce emitted citation IDs, retrieved/fetched URL allowlists, immutable caller domains, and typed success in code.
- Rank `searchResults` with the model and finish via `done`; project ranked evidence directly for structured output and validate the exact schema.
- Keep retrieval/model adapters swappable; retain external long-tail retrieval for general traffic and scope Vespa-only claims to measured cohorts.
- Use blind paired, output-aware judges with ties, swaps, completion accounting, and no asymmetric clipping.

## Unresolved questions

- Can a cheaper native model satisfy first-hop tool use, latency, and quality on a completed parity run?
- Does broad schema guidance add value over unguided direct ranked-evidence projection?
- Does direct projection generalize beyond the schema family that drove the n=300 gain, and can its 17.3% cost increase be reduced?
- Can filtered `searchResults` and empty-result handling recover without weakening immutable domain safety?
- How should date-filter compliance be audited when returned documents do not expose enough date evidence?
- When is eager fetch beneficial on analytical or matrix questions, and what reliable trigger should activate it?
