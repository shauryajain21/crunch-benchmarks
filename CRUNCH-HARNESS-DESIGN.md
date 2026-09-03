This is the Crunch design-decision record (also at [`docs/DECISIONS.md`](docs/DECISIONS.md)).

# Crunch runtime design

Crunch turns a question into a grounded, typed API response by running one fast reasoning model inside a small, controlled search loop.

`question → one fast reasoning model → search/fetch loop → answer → coverage/citation cleanup → API output`

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

### 1. Stop adaptively

- The model can stop after enough evidence or continue with another search or fetch.
- Forced one-hop scored 62.5% and forced two-hop scored 65.0%, each on n=40 against Deep.
- The roughly 76% figure comes from the separate n=200 split-finish `searchResults` arm against Deep, not a directly paired three-arm test. We therefore avoid fixed hop counts.

### 2. Use snippets first

- Search returns snippets first; the model fetches full pages only when needed.
- On n=100, automatic scraping scored 0.930 at 30.5s p50. No automatic scraping scored 0.940 at 9.5s.
- Scraping added large latency without quality gain, so automatic top-page scraping was disabled.

### 3. Return ten results per search

- Each search returns at most ten results, reducing model context and retrieval cost.
- On n=25, ten versus twenty results changed quality by +0.015 ± 0.086.
- The quality difference was a null, so the lower-cost, smaller-context limit stayed at ten.

### 4. Use the v7 answer contract

- The v7 answer contract tells the model to directly answer every requested part, resolve conflicting evidence, show calculations, clearly state missing information, and cite each factual claim.
- On 24 difficult questions, missing requirements fell from 25.8% to 16.7%. The full n=100 gain was only +0.012 ± 0.041.
- We adopted v7 for fewer omissions and its clearer mechanism, not as a large aggregate quality win.

### 5. Run a coverage pass

- After writing, the harness checks whether requested parts are missing and fills supported gaps.
- On n=100, score rose from 0.911 to 0.930; the benefit was concentrated in agentic tasks.
- We kept the pass as an omission safeguard, not as a broad lookup-quality claim.

### 6. Repair and renumber citations

- Deterministic cleanup maps citations to the final emitted source list and renumbers them.
- Regrading the same stored answers raised score from 0.571 to 0.748; unsourced claims fell from 23.8% to 6.1%.
- Because the answers did not change, this showed a plumbing failure. Citation repair became part of finalization.

### 7. Model-rank `searchResults`

- The model chooses across result lists from rewritten subqueries instead of comparing raw cross-query scores.
- On the same n=364 benchmark slice, score-max reached 45.2% against Deep and model reranking reached 76.1% against Deep.
- These are separate comparisons to Deep, not necessarily a direct reranker-versus-score judge. The score-scale failure still justified model ranking.

### 8. End `searchResults` with `done`

- Once ranking is complete, the model calls `done` instead of writing prose that the API discards.
- On n=200, quality was unchanged by the paired sign test (p=0.67), while latency fell from 44.1s to 8.4s.
- The large latency gain with no measured quality loss made `done` the finish contract.

### 9. Keep filters outside the model

- The harness applies caller domain and date filters as immutable constraints outside model-written queries.
- On n=200 filtered requests, Crunch returned 0 domain-violating URLs versus 33 for Deep.
- Crunch `searchResults` recall and reliability were worse, so the filter design stayed but the release was not approved.

### 10. Own the agent loop

- The harness controls model turns, tool calls, budgets, failures, and stopping.
- On production `sourcedAnswer` traffic, n=687 produced 578 Crunch wins, 108 losses, and 1 tie against frozen Deep ([`production-2000-sourced`](docs/benchmarks/production-2000.md)).
- This supports the whole Crunch system, but does not isolate loop ownership.

## Remaining principles

These are not restated in the evaluation-backed choices above.

- **One fast reasoning model.** The same model gathers evidence and writes the answer, keeping the question, queries, evidence, and uncertainty in one context. A separate Pro writer scored 0.750 versus 0.746 for the loop writer at 4.8× model cost.
- **Row-level failures.** One bad row must not stop a campaign; rows are isolated and resumable.
- **Blind paired judging.** Compare typed outputs with swaps and ties; a margin smaller than order instability is a tie.
- **Force grounding first.** First turn: `tool_choice=required`, at least 3 distinct parallel searches, no final answer on that turn.
  - Eval: n=100 production `sourcedAnswer`, same IDs, Gemini 3.7 Flash, Claude Sonnet 4.5 blind pairwise, 40 swaps. Shipped (`min_first_searches=3`, first-turn required, no answer turn 1) vs free (`min_first=0`, auto, first-turn answer allowed): **41–24–35** (63.1% decided); **16/40 swap flips (40%)** — margin does not beat order instability. Median latency ~10.5s vs ~10.6s. Free never answered on turn 1 (never-answer-from-memory stayed on); first-turn searches 2.03, 61 rows <3. Shipped 3.31, never <3 ([`firsthop-ab-100`](docs/benchmarks/firsthop-ab-100.md)).
  - Decision: keep the shipped first-hop rule. Diagnostic, not a shipped-config change. 3 stays the default because the lean is the right direction and latency is not worse; the swap rate blocks locking “3 is optimal.”
  - Later-hop stopping, not first-hop count: one literal query n=40 scored 34.4%; forced 1-hop/2-hop gather n=40 scored 62.5%/65.0%.
- **Guess-and-verify on chains.** On multi-hop questions the model often guesses the intermediate entity and fires an open check in the same turn.
  - Eval: 10 chain questions, stock prompt vs extra “do this in order” instruction, both 10/10, 0–0–10, same ~2.2 hops.
  - Decision: keep stock guess-and-verify; extra chain wording did not help. Small n; entities were likely known from pretraining.
- **Host allowlist for fetch.** A host must first appear in search results; another path on that host is allowed. Reject any-host and exact-URL-only.
  - Eval: ~103 domain-scoped questions, exact-URL vs host allowlist (rank_guard). Exact-URL refused 14 legitimate fetches. Host allowlist allowed 15 inferred-path fetches; 14 returned text and were cited. Quality −0.010 ± 0.049 (null).
  - Shipped because it unblocked real official-page fetches with no measured quality cost.
