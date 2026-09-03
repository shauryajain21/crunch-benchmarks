# Decision archive

Exhaustive historical record. The engineer-facing decision document is [`DECISIONS.md`](DECISIONS.md). Evidence IDs resolve to [`benchmarks.json`](../benchmarks.json). “Adopted” means supported for the stated scope, not universally optimal.

## Agent loop and stopping

- **Own the loop — adopted.** A reasoning model over the internal index made latency, retrieval, and output shaping controllable. Vendor replay motivated the work; `model-bakeoff-20`, `prod100-corrected`.
- **Gemini 3.7 Flash, low effort, no deadline — adopted.** It was the only tested model that cleared the latency and quality envelope. Luna is the runner-up when latency is relaxed. `model-bakeoff-20`, `model-hunt-25`.
- **Forced first search with parallel queries — adopted.** It prevents memory-only answers and exploits parallel tool calls. Qwen arms that rejected required tools were not configuration-equivalent. `model-hunt-25`.
- **Model-decided stopping — adopted.** One or two forced gather hops both underperformed the adaptive path. `forced-gather-hops-40`.
- **Guess-and-verify rather than mandatory sequencing — adopted with caveat.** It saves a hop when an intermediate is in model prior; at least one non-presupposing query is still desirable. `chain-prompt-10`.
- **Separate synthesis model — rejected.** Pro added negligible quality at 4.8× model cost; Sonnet was worse at 16×. `synthesis-writers-24`.
- **Coverage pass — adopted with bounded claim.** It lowers omission and unsourced rates, but its headline gain on agentic prompts included citation laundering. `coverage-pass-100`.

## Retrieval, filtering, and fetches

- **Toolbox/Columbus retrieval — adopted for general Crunch.** Cap each search at 10 results and fetched text at 12k characters. `toolbox-caps-25`.
- **External long-tail retrieval — retained for general traffic.** Vespa-only scored 20.8% on a tiny general-web slice, although it works extremely well on the enterprise cohort. `vespa-general-25`, `enterprise-vespa-1000`.
- **Brave as a second index — rejected.** It added candidates but not evidence under a fixed read budget. `brave-second-index-100`.
- **Hard user domain scope — adopted.** Explicit domain filters beat broad hints; model-written dorks may not broaden the caller's allowlist. `domain-scope-108`, `filtered-production-200`.
- **Defensive output URL filtering — adopted.** Final filtered Crunch had zero observed domain violations. Date correctness remains unobservable in returned documents. `filtered-production-200`.
- **Automatic scrape-top — rejected.** It consumed most latency and did not improve the powered n=100 result. `scrape-top-100`.
- **Conservative fetch default — adopted.** Prompt and tool description must agree. Analytical/matrix tasks may use the more eager rule. `fetch-rule-100`.
- **Host-level fetch guard — adopted.** The model may infer a new path only on a seen host; arbitrary domains remain blocked. `host-fetch-guard`.
- **Deterministic auto-read ranking — rejected.** It selected more plausible pages but did not improve quality. `autoread-ranking-103`.

## Prompts, citations, and answers

- **v7 answer contract — adopted.** Dated verdicts, explicit negatives, source adjudication, arithmetic, and complete entity×attribute coverage reduce omissions. Its pass-rate delta alone is not proven. `prompt-v6-v7`.
- **Citation IDs must index emitted sources — adopted invariant.** Renumbering fixed a 0.179 false quality loss with zero regressions. `citation-renumber-rescore`.
- **Never trust raw URLs from generation — adopted invariant.** Strip unretrieved URLs, allowlist sources, and repair citation placement before return. `host-fetch-guard`, `prod100-corrected`.

## Output modes and schemas

- **`searchResults` uses model ranking — adopted.** Concatenation was not ranking; raw reranker scores were incomparable across rewrites. `search-rank-rff-50`, `search-max-score-364`, `search-model-rerank-364`.
- **`searchResults` ends with `done` — adopted.** Writing prose then discarding it added latency without quality. `search-split-finish-200`.
- **Structured output is schema-validated — adopted.** Include the question, allow schema-valid nulls, coerce only where the schema permits, and retry validation. `structured-projection-50`, `structured-projector-rescore-319`.
- **Mechanical abstention — rejected.** Matching the comparator's null rate lowered quality. `structured-abstention-194`.
- **Broad schema guidance during gathering — rejected alone.** It filled more fields but tied on correctness and added latency. `schema-guidance-40`.
- **Ranked evidence projected directly to schema — adopted as the next architecture.** The 300-row confirmation beat write→project, mainly by preserving requested URLs. Broad guidance versus unguided direct is still unproven, so production work should narrow the checklist. `schema-factorial-40`, `schema-guided-direct-300`.

## Reliability and judging

- **Rows are failure-isolated and resumable — adopted.** A finish-reason-only response killed an early 2,000-row run; one bad row must not terminate a campaign. `runner-row1520-crash`.
- **Success follows the typed contract, not prose presence — adopted.** Answer-derived status falsely marked 845 split-finish rows failed. `runner-status-bug`.
- **No asymmetric answer clipping — adopted invariant.** The 1,400- and 1,800-character clips generated false deltas. `judge-clip-1400`, `production-850-clip1800`.
- **Blind, paired, typed-output judges with swaps and ties — adopted.** A margin smaller than order instability is a tie. Exact schema and filter constraints must be judge-visible. See [`MEASUREMENT.md`](MEASUREMENT.md).
