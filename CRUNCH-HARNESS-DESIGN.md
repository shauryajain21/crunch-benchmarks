Crunch design-decision record. `[docs/DECISIONS.md](docs/DECISIONS.md)` points here; the exhaustive history is `[docs/DECISIONS-ARCHIVE.md](docs/DECISIONS-ARCHIVE.md)`.

# Why Crunch does what it does

Crunch turns a question into a grounded, typed API response by running one fast reasoning model inside a small, controlled search loop.

`question → forced first hop → one fast reasoning model → search/fetch loop → answer → coverage/citation cleanup → API output`

Naming: **Deep V1** is the current production Deep (LanggraphDeep + DistilLabs formatter). **Deep V2** is Crunch. This doc says Crunch and Deep V1. Evidence IDs (e.g. `scrape-top-100`) resolve to `[docs/benchmarks/](docs/benchmarks/)` and `[benchmarks.json](benchmarks.json)`. “Adopted” means supported for the stated scope, not universally optimal.

## Contents

1. [Overall design](#overall-design)
2. [What the harness owns / what retrieval owns](#what-the-harness-owns)
3. [Design choices shaped by evaluation](#design-choices-shaped-by-evaluation), in loop order
  - Model: [1. Gemini 3.7 Flash](#1-one-fast-reasoning-model-gemini-37-flash)
  - Gather: [2. Force the first hop](#2-force-the-first-hop) · [3. Stop adaptively](#3-stop-adaptively) · [4. Snippets first](#4-use-snippets-first) · [5. Ten results per search](#5-return-ten-results-per-search) · [6. Host allowlist for fetch](#6-host-allowlist-for-fetch) · [7. Guess-and-verify on chains](#7-guess-and-verify-on-chains)
  - Write: [8. v7 answer contract](#8-use-the-v7-answer-contract) · [9. Coverage pass](#9-run-a-coverage-pass) · [10. Repair citations](#10-repair-and-renumber-citations)
  - `searchResults`: [11. Model-rank](#11-model-rank-searchresults)
  - Contract: [12. Own the agent loop](#12-own-the-agent-loop)
4. [Tried and rejected](#tried-and-rejected)
5. [Known limits](#known-limits)

## Overall design

The request enters a runtime harness that reads the question, requested output shape, and caller filters.
The harness starts one fast reasoning model and requires it to search before answering.
The model can rewrite the query, inspect snippets, fetch selected pages, and repeat when evidence is weak.
It stops when it has enough support, then writes the answer from the evidence it gathered.
The harness checks coverage, repairs citation numbering and placement, rejects unsupported URLs, and shapes the final API response.

The same flow supports prose answers, ranked search results, and structured output.
The model and retrieval providers sit behind stable interfaces, so either can change without changing the response contract.

## What the harness owns

- Request parsing, caller filters, output mode, and schema.
- The model/tool loop, tool limits, evidence IDs, context limits, and stopping rules.
- The rule that evidence must be gathered before an answer is accepted.
- Coverage checks, citation cleanup, URL safety, schema validation, and typed success or failure.
- Final response shaping for each API output mode.

## What retrieval owns

- Searching indexes or the web for candidate documents.
- Returning snippets, document metadata, and stable source URLs.
- Fetching page text when the model selects a result to read.
- Retrieval ranking and provider-specific behavior behind the search/fetch interface.

Retrieval supplies evidence; it does not decide when the whole task is complete or define the final API response.

## Design choices shaped by evaluation

Each entry: what Crunch does, the eval, the number, the decision. Ordered by where it sits in the loop.

### 1. One fast reasoning model: Gemini 3.7 Flash

- The same model plans, searches, reads, stops, and writes. Planning, evidence, and uncertainty stay in one context.
- Model choice on n=25, same questions, shipped config. Checklist pass scores: Flash 0.879, Luna 0.873, Qwen 0.768, Gemini Flash Lite 0.692. Flash p50 was 9.4s at $14.71 per 1,000. No arm was cheaper, faster, and better. (`[model-hunt-25](docs/benchmarks/model-hunt-25.md)`, supersedes `[model-bakeoff-20](docs/benchmarks/model-bakeoff-20.md)`).
- Separate writer on n=24 hard questions. Gemini Pro as writer scored 0.750 vs 0.746 for the loop model writing its own answer, at 4.8× model cost. (`[synthesis-writers-24](docs/benchmarks/synthesis-writers-24.md)`).
- Decision: Flash, one model for loop and writing. Luna is the latency-relaxed runner-up.

### 2. Force the first hop

- Turn 1: `tool_choice=required`, at least 3 distinct parallel searches, no answer on that turn. Never answer from memory.
- Shipped (`min_first_searches=3`) vs free (`min_first=0`, answer allowed on turn 1) on n=100 production `sourcedAnswer`, blind pairwise with 40 swaps. Win–loss–tie: **41–24–35** (63.1% decided). But **16/40 swap flips (40%)**, so the margin does not beat order instability. Latency was ~10.5s vs ~10.6s. Free averaged 2.03 first-turn searches with 61 rows below 3; shipped averaged 3.31 and never went below 3. (`[firsthop-ab-100](docs/benchmarks/firsthop-ab-100.md)`).
- Decision: the rule is shipped; the eval was diagnostic. 3 stays because the lean is in the right direction at no latency cost. The swap rate blocks calling 3 “optimal.”
- A single literal query (no rewrite, one hop) scored 34.4% vs Deep V1 on n=40 (`[search-literal-40](docs/benchmarks/search-literal-40.md)`). The first hop needs multiple rewritten queries.

### 3. Stop adaptively

- After the first hop the model decides to stop or to search/fetch again. No fixed hop count.
- Forced 1-hop scored 62.5% vs Deep V1; forced 2-hop scored 65.0%; each on n=40 (`[forced-gather-hops-40](docs/benchmarks/forced-gather-hops-40.md)`). Adaptive stopping reached ~76% vs Deep V1 on a different n=200 `searchResults` arm (`[search-split-finish-200](docs/benchmarks/search-split-finish-200.md)`).
- Not a paired three-arm test; the direction is consistent. Decision: no forced hop counts.

### 4. Use snippets first

- Search returns snippets; the model fetches full pages only when it selects one.
- On n=100, automatic top-page scraping scored 0.930 (p50 30.5s). Without automatic scraping: 0.940 (p50 9.5s). (`[scrape-top-100](docs/benchmarks/scrape-top-100.md)`).
- Decision: automatic scraping off. Large latency cost, no quality gain.

### 5. Return ten results per search

- Each search returns at most ten results.
- On n=25, ten vs twenty results: quality delta +0.015 ± 0.086, a null (`[toolbox-caps-25](docs/benchmarks/toolbox-caps-25.md)`).
- Decision: ten. Same quality, smaller context, lower retrieval cost.

### 6. Host allowlist for fetch

- A host must first appear in search results; any path on that host may then be fetched. Rejected: any-host, and exact-URL-only.
- On ~103 domain-scoped questions, exact-URL vs host allowlist: exact-URL refused 14 legitimate fetches; host allowlist allowed 15 inferred-path fetches, of which 14 returned text and were cited. Quality delta −0.010 ± 0.049, a null (`[host-fetch-guard](docs/benchmarks/host-fetch-guard.md)`).
- Decision: host allowlist. Unblocks official-page fetches at no measured quality cost.

### 7. Guess-and-verify on chains

- On multi-hop questions the model guesses the intermediate entity and fires a verifying search in the same turn, instead of strictly hop-by-hop.
- On 10 chain questions, stock prompt vs added “do this in order” instruction: both scored 10/10, pairwise 0–0–10, ~2.2 hops each (`[chain-prompt-10](docs/benchmarks/chain-prompt-10.md)`).
- Decision: keep stock behavior. Small n; entities were likely known from pretraining.

### 8. Use the v7 answer contract

- The model must answer every requested part, resolve conflicting evidence, show calculations, state what is missing, and cite each factual claim.
- On 24 hard questions, missing requirements fell 25.8% → 16.7%. On the full n=100 aggregate, quality delta was +0.012 ± 0.041, a null (`[prompt-v6-v7](docs/benchmarks/prompt-v6-v7.md)`).
- Decision: adopted on mechanism (fewer omissions), not on aggregate score.

### 9. Run a coverage pass

- After writing, the harness checks for missing requested parts and fills gaps that the gathered evidence supports.
- On n=100, score rose 0.911 → 0.930, concentrated in agentic tasks (`[coverage-pass-100](docs/benchmarks/coverage-pass-100.md)`).
- Decision: keep as an omission safeguard. Not a lookup-quality claim.

### 10. Repair and renumber citations

- Deterministic cleanup maps every citation to the final emitted source list and renumbers.
- Regrading the same stored answers: score rose 0.571 → 0.748; unsourced claims fell 23.8% → 6.1% (`[citation-renumber-rescore](docs/benchmarks/citation-renumber-rescore.md)`).
- The answers did not change, so this was a plumbing failure. Decision: citation repair is part of finalization.

### 11. Model-rank `searchResults`

- The model ranks across result lists from its rewritten subqueries. Raw retrieval scores are not compared across queries.
- Same n=364 slice vs Deep V1: score-max scored 45.2% (`[search-max-score-364](docs/benchmarks/search-max-score-364.md)`); model rerank scored 76.1% (`[search-model-rerank-364](docs/benchmarks/search-model-rerank-364.md)`).
- Two separate comparisons to Deep V1, not a direct reranker-vs-score judge. Decision: model ranking; cross-query scores are not on one scale.

### 12. Own the agent loop

- The harness controls model turns, tool calls, budgets, failures, and stopping. The model does not decide the contract.
- On production `sourcedAnswer`, n=687 vs frozen Deep V1: win–loss–tie 578–108–1 (`[production-2000-sourced](docs/benchmarks/production-2000.md)`). This is the whole system, not loop ownership alone.
- The isolation is in `[docs/EXPLORING-DEEP.md](docs/EXPLORING-DEEP.md)` §2: swapping the planner and writer inside Deep V1 did not fix one-hop stopping, generic queries, or the split answer/source assembly. The harness, not the model, set those behaviors.
- Decision: the harness owns the loop.

## Evidence


| Tried | Result | Evidence |
|---|---|---|
| **Fixed hop count.** Force the loop to stop after exactly one or two search rounds, no model decision. | Forcing the loop to stop after exactly one or two search rounds scored 62.5% and 65.0% vs Deep V1 (n=40 each). Letting the model decide when to stop reached ~76% on a separate n=200 arm. Fixed hop counts under-search hard questions and over-search easy ones. | `forced-gather-hops-40`, `search-split-finish-200` |
| **Literal query, one hop.** Send the customer's text verbatim as the only search, no rewriting, no follow-up. | Sending the customer's text as the only query, one hop, scored 34.4% vs Deep V1 (n=40). Without rewrites the loop cannot ask the sub-questions the customer implied. | `search-literal-40` |
| **Auto-scrape top pages.** Fetch full text of the top results on every search before the model reads anything. | Fetching the top pages on every search dropped quality −0.010 (0.940 → 0.930) and added ~21s p50 (9.5s → 30.5s) on n=100. Snippets already carried the answer; full text only added context and latency. | `scrape-top-100` |
| **Twenty results per search.** Double the per-search result cap from ten to twenty. | Doubling results per search moved quality +0.015 ± 0.086 on n=25, a null, while adding context tokens and retrieval cost. Ten was kept. | `toolbox-caps-25` |
| **Exact-URL fetch guard.** Allow fetch only for URLs literally present in search results, not other paths on the same host. | Allowing fetch only for URLs literally returned by search refused 14 legitimate fetches (official pages reachable by an inferred path). The host allowlist let 15 such fetches through, 14 were cited, quality unchanged (−0.010 ± 0.049). | `host-fetch-guard` |
| **Separate stronger writer.** Let Flash gather evidence, then hand it to Gemini Pro to write the final answer. | Handing the gathered evidence to a stronger writer scored 0.750 vs 0.746 for the loop model writing its own answer, at 4.8× model cost (n=24 hard questions). The loop model already had the context; a bigger writer did not. | `synthesis-writers-24` |
| **Ordered-hops chain prompt.** Add an instruction telling the model to resolve multi-hop questions step by step. | Adding a step-by-step instruction for multi-hop questions gave 10/10 for both arms, 0–0–10 pairwise, same ~2.2 hops. The model already guesses the intermediate entity and verifies it; the instruction changed nothing. | `chain-prompt-10` |
| **Score-max ranking for `searchResults`.** Merge result lists from rewritten subqueries and rank by highest raw retrieval score. | Ranking by the highest raw retrieval score across rewritten subqueries scored 45.2% vs Deep V1; having the model rank the merged lists scored 76.1% on the same n=364 slice. Scores from different queries are not on one scale. | `search-max-score-364`, `search-model-rerank-364` |
| **Other loop models.** Run the shipped Crunch config on GPT-5.6 Luna, Gemini Flash Lite, Qwen, OSS-120B, and other open models. | On the same 25 questions and shipped config, Flash passed 0.879; Luna 0.873 (slower), Qwen 0.768, Flash Lite 0.692. No arm was cheaper, faster, and better at once. Qwen could not honor the required first-hop tool calls. | `model-hunt-25` |
| **Free first hop.** Drop the ≥3-searches rule and let the model choose how many searches to run on turn 1. | Letting the model pick how many first-turn searches to run: shipped rule (≥3) led 41–24–35 on n=100, but 16/40 swaps flipped (40%), so the margin is inside judge noise. Free averaged 2.03 searches; shipped 3.31. Kept ≥3 as the safer default, not as a proven optimum. | `firsthop-ab-100` |


## Known limits

- Several adoptions rest on nulls with a stated mechanism (v7 contract, ten results, host allowlist, first-hop count). They are defensible, not proven optimal.
- Small-n evals (`chain-prompt-10`, `model-bakeoff-20`, `toolbox-caps-25`) cannot rank close arms.
- Live-traffic p99, Gemini rate-limit behavior under production load, and the Vertex vs direct-client path are not measured in this repo. Do not read benchmark latencies as SLOs.
- Some Deep V1 comparisons used a frozen snapshot, others live Deep V1. Which is which is in [`docs/MEASUREMENT.md`](docs/MEASUREMENT.md).

