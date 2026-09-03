# Crunch architecture

Compact current-state architecture. Evidence IDs resolve to
[`benchmarks.json`](benchmarks.json). See
[`docs/DECISIONS.md`](docs/DECISIONS.md) and
[`docs/EXPERIMENTS.md`](docs/EXPERIMENTS.md) for exhaustive history.

## Model

- Choice: Gemini 3.7 Flash, low effort, no model deadline.
- Function: Plans retrieval, evaluates evidence, stops, and writes or ranks.
- Decision: Keep one model for gathering and writing; Luna is the latency-relaxed runner-up.
- Evidence: Shipped-config pass 0.879 vs Luna 0.873; no tested arm was cheaper, faster, and better (`model-hunt-25`). Separate Pro writing scored 0.750 vs loop writing 0.746 at 4.8× model cost (`synthesis-writers-24`).

## Harness

- Function: Parses the question, output mode, schema, and request filters; exposes typed search, fetch, and finish tools.
- Decision: Own the loop and enforce deterministic contracts around model choices.
- Evidence: Corrected internal benchmark: Crunch 0.940, Exa 0.894, Deep 0.610 checklist pass; Crunch and Exa are treated as tied on ordinary lookups (`prod100-corrected`).

## Part 1 — Forced first hop

- Function: Requires at least one search before any answer; permits parallel query rewrites.
- Decision: Prevent memory-only answers; use guess-and-verify for likely intermediates, with a non-presupposing query when needed.
- Evidence: One literal query reached 34.4% decided (`search-literal-40`); mandatory chain sequencing changed no answers in 10/10 ties (`chain-prompt-10`).

## Part 2 — Search loop and retrieval

- Function: Runs adaptive rewrites over Toolbox/Columbus; returns at most 10 results per search with stable evidence IDs.
- Decision: Retain external long-tail coverage for general traffic; do not add Brave under the same read budget.
- Evidence: Brave difference-in-differences was -0.042 (`brave-second-index-100`); Vespa-only general-web pilot reached 20.8% decided (`vespa-general-25`).

## Part 3 — Fetch

- Function: Reads model-selected pages, capped at 12,000 characters.
- Decision: Fetch conservatively; allow inferred paths only on a host already seen in retrieval; do not auto-scrape top results.
- Evidence: Auto-scraping scored 0.9295 vs 0.9403 and raised median latency 9.5s→30.5s (`scrape-top-100`). Seen-host fetches returned and cited text on 14/15 novel paths (`host-fetch-guard`).

## Part 4 — Stopping

- Function: Lets the model search again, fetch, or finish when evidence is sufficient.
- Decision: Use adaptive stopping; bound context with result and page caps, not a model deadline.
- Evidence: Forced one-hop and two-hop arms reached 62.5% and 65.0% decided (`forced-gather-hops-40`); caps improved answered rows from 16/25 to 24/24 (`toolbox-caps-25`).

## Part 5 — Write

- Function: Produces direct prose with dated verdicts, explicit negatives, source adjudication, shown arithmetic, and complete requested fields.
- Decision: Use the v7 contract in the loop model; do not add a separate synthesis model.
- Evidence: v7 reduced targeted missing items from 25.8% to 16.7%; overall +0.012 remained inside a 0.041 CI (`prompt-v6-v7`).

## Part 6 — Coverage

- Function: Checks the draft for omitted or unsupported requested items.
- Decision: Keep the pass for omission reduction, not as a broad quality claim.
- Evidence: Missing items fell 4.5%→3.3%; overall pass rose 0.019, while ordinary lookups gained 0 (`coverage-pass-100`).

## Part 7 — Citations and URL safety

- Function: Selects emitted sources, renumbers citations, repairs placement, and rejects URLs absent from retrieved or fetched evidence.
- Decision: Treat citation resolution and retrieved-URL allowlisting as invariants; generated plausibility is insufficient.
- Evidence: Renumbering identical answers raised pass 0.571→0.748, with 30 better and 0 worse (`citation-renumber-rescore`). Seen-host inference is the only allowed novel path (`host-fetch-guard`).

## Part 8 — Output shaping

- Function: Finishes differently for `sourcedAnswer`, `searchResults`, and structured output.
- Decision: Write prose only when requested; rank documents for search; project ranked evidence directly to schemas.
- Evidence: Search `done` reduced latency 44.1s→8.4s without measured quality loss (`search-split-finish-200`). Ranked-evidence projection won 139–79 with 82 ties vs write→project (`schema-guided-direct-300`).

## Part 9 — Filters

- Function: Applies immutable caller domain constraints to retrieval and final URLs.
- Decision: Model-written dorks may narrow, never broaden; keep defensive output checks. Do not ship the measured filtered configuration unchanged until empty results and filtered search recover.
- Evidence: Hard scope gained 0.132 on 52 explicit-dork rows (`domain-scope-108`). Corrected filtered Crunch had 0 domain violations vs Deep's 33, but completed 192/200 and lost filtered search 6–20 with 49 ties (`filtered-production-200`).

## Part 10 — Reliability

- Function: Isolates row failures, resumes campaigns, and derives success from the requested typed output.
- Decision: A missing model message or absent prose cannot crash or falsely fail a valid typed result.
- Evidence: One finish-reason-only response stopped a run at row 1,520 (`runner-row1520-crash`); prose-derived status falsely failed 845 successful search rows (`runner-status-bug`).

## Part 11 — Judge and measurement

- Function: Compares anonymous paired outputs with mode-aware judges, order swaps, explicit ties, failures, and completion.
- Decision: Judge full typed outputs; exact schemas and immutable filters are visible; avoid asymmetric clipping and cross-pass score comparisons.
- Evidence: A 1,800-character clip cut 75% of Crunch vs 20% of Deep and changed the corrected win rate from 61.3% to 84.2% (`production-850-clip1800`). Full rules: [`docs/MEASUREMENT.md`](docs/MEASUREMENT.md).

## Output

### `sourcedAnswer`

- Shape: `{ answer: string, sources: Source[] }`
- Path: Search → optional fetch → write → coverage → citation and URL validation.
- Evidence: 578–108–1; 84.3% decided vs frozen Deep, n=687 (`production-2000-sourced`).

### `searchResults`

- Shape: `{ results: SearchResult[] }`
- Path: Search → optional fetch → model rank across rewrites → typed `done`.
- Evidence: 622–233–2; 72.8% decided vs frozen Deep, n=857 (`production-2000-search`).

### Structured

- Shape: Exact caller schema.
- Path: Search → optional fetch → ranked evidence → question-aware projection → validation → one retry.
- Evidence: Direct ranked evidence beat current write→project 139–79–82; 63.8% decided, 95% CI 57.2–69.9%, n=300; cost +17.3% (`schema-guided-direct-300`).

## Canonical eval summary

- DeepSearchQA first — n=896: Crunch 36.2% all-correct vs production Deep 8.4%; +27.8 points, 277 better / 28 worse / 591 tied (`deepsearchqa-896`).
- Production `sourcedAnswer` — n=687: 578–108–1; 84.3% decided (`production-2000-sourced`).
- Production `searchResults` — n=857: 622–233–2; 72.8% decided (`production-2000-search`).
- Structured architecture A/B — n=300: 139–79–82; 63.8% decided (`schema-guided-direct-300`).
- Filtered production — n=200: 67–32–101; 67.7% decided; diagnostic, not launch-ready (`filtered-production-200`).
- Enterprise Vespa-only — n=1,000: 821–144–35; 85.1% decided on that cohort (`enterprise-vespa-1000`).

The archive contains 54 benchmark records. Claims above are limited to each named population and instrument.
