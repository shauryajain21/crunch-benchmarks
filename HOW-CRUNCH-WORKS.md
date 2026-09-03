# Crunch: system and evidence
Crunch is a search agent: a reasoning model controls retrieval, decides when it
has enough evidence, and returns one of three typed outputs. This repository is
the privacy-preserving design record. It contains aggregate results, not raw
customer queries, customer identifiers, answers, secrets, or large artifacts.
This README is the shortest self-contained account of the current system. Each
section states the mechanism, tested alternatives, evidence, and decision.
Evidence IDs resolve to [`benchmarks.json`](benchmarks.json); deeper records are
linked at the end.
## 1. Goal and contract
**What it does**
- Answer multi-step web questions from retrieved evidence, not model memory.
- Preserve the caller's requested output type:
  - `sourcedAnswer`: prose plus cited sources.
  - `searchResults`: a ranked list of retrieved documents.
  - structured output: JSON matching the caller's exact schema.
- Treat user domain filters as immutable constraints.
- Prefer complete, supported answers while keeping latency and model cost low.
**What was tested**
- Replaying existing search vendors showed attribution and answer-contract
  problems, but those vendor probes predated Crunch and are not Crunch scores
  (`vendor-replay-400`).
- One literal search without an agent loop reached only 34.4% decided against
  frozen Deep on 40 rows (`search-literal-40`).
**Decision**
Own the complete loop. The model may plan, search, fetch, stop, and shape the
typed result, while deterministic code enforces schemas, source IDs, URLs,
filters, and failure handling.
## 2. End-to-end flow
1. Parse the question, output mode, schema, and request filters.
2. Force a first retrieval hop; the model can issue parallel query rewrites.
3. Search the configured index and return bounded snippets with stable evidence
   IDs.
4. Let the model inspect evidence, optionally search again, or fetch a page.
5. Let the model stop when the evidence is sufficient.
6. Build the requested output:
   - write and coverage-check prose for `sourcedAnswer`;
   - model-rank documents for `searchResults`;
   - project to the exact schema for structured output.
7. Validate citations, URLs, filters, and the typed contract before returning.
**Alternatives and evidence**
- A separate synthesis model added almost no quality: Gemini Pro scored 0.750
  versus 0.746 for the loop writer at 4.8× model cost; Sonnet scored 0.654 at
  16× cost (`synthesis-writers-24`).
- Fixed one- or two-hop gathering did not beat adaptive stopping
  (`forced-gather-hops-40`).
**Decision**
Use one model for gathering and writing, with deterministic checks around it.
The structured path is moving from write-then-project to direct projection from
ranked evidence; that newer path is described below.
## 3. First hop and search loop
**What it does**
- The first action must include search, preventing a memory-only answer.
- The model may issue several queries in parallel to cover entities, attributes,
  dates, and disambiguations.
- Later hops are adaptive: search again, fetch, or finish.
- For chains, the model can guess a likely intermediate and verify it rather
  than always spending a turn discovering it first. At least one query should
  avoid presupposing the guess.
**Alternatives and evidence**
- Early planner tests found malformed site dorks across models, showing prompt
  defaults also needed repair (`planner-diverse-100`).
- An explicit mandatory chain sequence tied the stock prompt 10/10 and changed
  no answer (`chain-prompt-10`).
- One literal query reached 34.4% decided on its pilot (`search-literal-40`).
- Some cheaper models could not reliably honor required first-hop tool calls
  (`model-hunt-25`).
**Decision**
Keep forced first search, parallel rewrites, and adaptive guess-and-verify.
Do not require a rigid chain plan or collapse the loop to one query.
## 4. Retrieval and fetch
**What it does**
- General Crunch uses Toolbox/Columbus retrieval, including external long-tail
  coverage.
- Each search returns at most 10 results.
- A fetched page contributes at most 12,000 characters.
- Fetching is conservative by default; analytical or matrix-shaped work may
  justify more reading.
- A fetch may use an exact retrieved URL or infer a new path on a host already
  seen in retrieval. It may not introduce an arbitrary new host.
**Alternatives and evidence**
- Adding Brave produced more candidates but a -0.042 difference-in-differences
  under the same read budget (`brave-second-index-100`).
- Forced scraping of the top three pages scored 0.9295 versus 0.9403 without
  it, while median latency rose from 9.5s to 30.5s (`scrape-top-100`).
- Deterministic intent ranking changed which pages were read but had a -0.011
  quality delta (`autoread-ranking-103`).
- Capping results/pages changed answered rows from 16/25 to 24/24, with no
  proven quality loss (`toolbox-caps-25`).
- Vespa-only scored 20.8% decided on a 25-row general-web pilot, but performed
  strongly on a distinct enterprise cohort (`vespa-general-25`,
  `enterprise-vespa-1000`).
**Decision**
Keep bounded Toolbox retrieval and model-selected fetches for general traffic.
Do not add a second index or automatic scraping. Vespa-only is viable for the
measured enterprise cohort, not established as a general replacement.
## 5. Stopping and budgets
**What it does**
- The model decides when enough evidence has been gathered.
- The shipped model runs without a model deadline; quality is protected from a
  premature time cutoff.
- Search and fetch payload caps bound context growth.
- Output-specific finishing avoids work that will be discarded.
**Alternatives and evidence**
- Forced one-hop and two-hop arms scored 62.5% and 65.0% decided on 40 rows;
  neither justified overriding adaptive stopping (`forced-gather-hops-40`).
- In `searchResults`, replacing write-then-discard with a `done` call reduced
  latency from 44.1s to 8.4s without a measured quality loss
  (`search-split-finish-200`).
- Automatic scraping made median latency 3.2× worse without quality gain
  (`scrape-top-100`).
**Decision**
Use model-decided stopping, no model deadline, bounded retrieval payloads, and
the cheapest valid finish for each output type.
## 6. Answer writing and coverage
**What it does**
- The v7 answer contract asks for dated verdicts, explicit negative findings,
  source adjudication, shown arithmetic, and complete entity-by-attribute
  coverage.
- A final coverage pass checks for requested items that were omitted or left
  unsupported.
**Alternatives and evidence**
- v7 improved overall checklist pass by only 0.012 versus v3, inside a 0.041
  confidence interval, but reduced missing items on the targeted subset from
  25.8% to 16.7% (`prompt-v6-v7`).
- The coverage pass added 0.019 overall, with missing items falling from 4.5%
  to 3.3%; ordinary lookup gain was zero, and some task-prompt gain was citation
  laundering (`coverage-pass-100`).
- Larger separate writers were far more expensive and did not improve quality
  (`synthesis-writers-24`).
**Decision**
Keep v7 and the coverage pass for their observed omission mechanism, not as a
claim of a large pass-rate gain. Keep writing in the loop model.
## 7. Citations and URL safety
**What it does**
- Every citation ID indexes the final emitted source list.
- Sources are selected first, then renumbered so IDs cannot point to removed or
  reordered entries.
- Generated raw URLs are never trusted. Output URLs must come from retrieved or
  fetched evidence and pass the source allowlist.
- Citation placement is repaired before return.
- Filtered outputs receive a final defensive URL-domain check.
**Alternatives and evidence**
- Broken source numbering caused a false 0.179 checklist loss. Renumbering the
  same answers raised pass from 0.571 to 0.748, with 30 better and zero worse
  rows (`citation-renumber-rescore`).
- The seen-host guard returned and cited text on 14/15 inferred-path fetches,
  with no proven quality change (`host-fetch-guard`).
- Final filtered Crunch had zero observed domain violations in 200 rows;
  comparator documents had 33 (`filtered-production-200`).
**Decision**
Citation/source consistency and retrieved-URL allowlisting are invariants, not
model preferences. A plausible generated URL is not enough.
## 8. `sourcedAnswer`
**What it does**
- Runs the full adaptive loop.
- Writes a direct prose answer under the v7 contract.
- Applies the coverage pass.
- Returns the answer with a validated, renumbered source list.
**Alternatives and evidence**
- Separate synthesis was rejected on quality and cost
  (`synthesis-writers-24`).
- Asymmetric judge clips created false losses: a 1,800-character clip cut 75%
  of Crunch versus 20% of Deep (`production-850-clip1800`).
- The corrected production benchmark scored 578 wins, 108 losses, and 1 tie:
  84.3% of decided rows (`production-2000-sourced`).
**Decision**
This remains the canonical prose path. Evaluate complete answers with a
typed-output judge and no asymmetric clipping.
## 9. `searchResults`
**What it does**
- Runs search rewrites and gathering without writing a prose answer.
- The model chooses a single ranking across all sub-query result lists.
- A typed `done` call ends the loop and returns the ranked documents.
**Alternatives and evidence**
- Arrival-order concatenation was not a real ranking.
- Reciprocal-rank fusion improved top-10 gold recall from 0.355 to 0.521 on 50
  rows but was superseded (`search-rank-rff-50`).
- Raw cross-encoder scores looked stronger in an early test but were not
  comparable across independently rewritten queries; the resulting production
  arm scored only 45.2% (`search-max-score-364`).
- Model ranking scored 76.1% decided on the 364-row pilot
  (`search-model-rerank-364`).
- The canonical production result is 622 wins, 233 losses, and 2 ties: 72.8%
  decided (`production-2000-search`).
**Decision**
Use model ranking across rewrites and finish with `done`. Do not concatenate
arrival order or compare raw reranker maxima across queries.
## 10. Structured output and direct-evidence experiments
**What it does now**
- The projector receives the question and target schema.
- It permits null only when valid for the schema, coerces only where allowed,
  validates the result, and retries validation once.
- The shipping architecture historically gathered evidence, wrote prose, then
  projected that prose to JSON.
**What was tested**
- Question-aware validation matched the prose ceiling at 0.62 strict accuracy
  on 50 rows (`structured-projection-50`).
- Mechanical abstention scored 42.9% decided and was rejected
  (`structured-abstention-194`).
- Projecting unranked snippets directly scored 56% versus Deep; directness alone
  was not enough (`gather-to-schema-319`).
- Broad schema guidance filled more leaves (32.9% versus 26.8%) but correctness
  tied 9–8 and latency rose from 22.6s to 29.9s (`schema-guidance-40`).
- A four-arm 40-row holdout isolated ranked-evidence projection as promising,
  while guidance versus unguided direct remained tied
  (`schema-factorial-40`).
- The 300-row confirmation compared current gather→write→project with guided
  gather→ranked evidence→project. Direct evidence won 139–79 with 82 ties:
  63.8% decided, 95% CI 57.2–69.9%, p=0.000058
  (`schema-guided-direct-300`).
- Its non-empty leaf rate rose from 39.6% to 42.5%, while estimated model cost
  rose 17.3%, from $33.27 to $39.02 per 1,000.
**Decision/status**
Adopt ranked-evidence projection as the next structured architecture. Narrow
the schema checklist before production rollout: the gain is concentrated in
one schema family, and broad guidance itself is still unproven. The 63.8% score
is an architecture A/B against current Crunch, not a production-vs-Deep score.
## 11. Filters
**What it does**
- Caller-supplied include domains are hard constraints.
- Model-written site dorks may narrow but never broaden that allowlist.
- Returned URLs are checked again after generation.
**Alternatives and evidence**
- Hard include-domain scope beat a broad host hint by 0.132 on 52 explicit-dork
  rows; metadata-domain gain was 0.033 (`domain-scope-108`).
- Filtered v0 allowed model dorks to escape scope and was invalidated
  (`filtered-production-200-v0`).
- Corrected v1 had 0 observed domain violations and scored 67–32 with 101 ties,
  or 67.7% decided, against live Deep (`filtered-production-200`).
- However, Crunch completed 192/200, and its filtered `searchResults` slice
  lost 6–20 with 49 ties. Date compliance could not be audited from returned
  documents.
**Decision/status**
Keep immutable filters and defensive URL checks. Do not ship the measured
filtered configuration unchanged until empty-result handling and filtered
`searchResults` recover. Date-filter correctness remains unverified.
## 12. Models and reliability
**What it does**
- Gemini 3.7 Flash at low effort is the loop and writing model.
- Runs have no model deadline.
- Each benchmark row is failure-isolated and resumable.
- Success is derived from the requested typed output, not whether prose exists.
**Alternatives and evidence**
- The initial n=20 bake-off selected Flash at 6.34s median, 9.03s p90, and
  $0.0187 per query; the sample was too small to rank close quality arms
  (`model-bakeoff-20`).
- The n=25 shipped-config hunt kept Flash: pass 0.879 versus Luna 0.873, with no
  tested arm cheaper, faster, and better (`model-hunt-25`).
- A finish-reason-only response crashed an early 2,000-row run at row 1,520
  (`runner-row1520-crash`).
- Prose-derived status falsely marked 845 successful `searchResults` rows as
  failures (`runner-status-bug`).
**Decision**
Keep Flash; Luna is the latency-relaxed runner-up. Isolate row failures, resume
campaigns safely, and judge success against the typed contract.
## 13. Judging and benchmark method
**Method**
- Pin identical questions and compare anonymous outputs in random order.
- Use a judge that sees the output-mode contract. Structured judges see the
  exact schema; filtered judges see immutable constraints.
- Keep failures in the denominator and report completion separately.
- Repeat an order-swapped sample. If the margin is smaller than order
  instability, call the result a tie.
- Report wins, losses, and ties. “Decided” means wins / (wins + losses); row win
  rate includes ties in the denominator.
- Use checklist pass for predeclared atomic requirements and all-correct only
  when every expected answer must appear with no excess answer.
- Do not compare scores produced by different judge passes as if paired.
- Distinguish a fresh run, a stored-output re-projection, and a re-judge.
**Rejected or repaired instruments**
- Citation IDs that did not resolve (`prod100-initial`).
- 1,400- and 1,800-character asymmetric answer clips (`judge-clip-1400`,
  `production-850-clip1800`).
- Cross-query raw score comparisons (`search-max-score-364`).
- Prose-derived success for typed outputs (`runner-status-bug`).
- Incomplete timeout/503 survivor sets as quality samples.
**Decision**
Canonical claims use blind paired judging, typed contracts, swaps, explicit
ties/failures, and the supersession rules in
[`docs/MEASUREMENT.md`](docs/MEASUREMENT.md).
## 14. Current canonical and decision-driving results
DeepSearchQA is listed first because it is the broad public-style test of
exhaustive multi-step answering.
- **DeepSearchQA, n=896:** Crunch 36.2% all-correct; production Deep 8.4%;
  paired delta 27.8 points (277 better, 28 worse, 591 tied). A replacement
  Gemini autorater prevents public leaderboard comparison (`deepsearchqa-896`).
- **Production `sourcedAnswer`, n=687:** 578–108–1; 84.3% decided, 95% CI
  80.5–88.0%, versus frozen Linkup Deep (`production-2000-sourced`).
- **Production `searchResults`, n=857:** 622–233–2; 72.8% decided, 95% CI
  69.4–76.1%, versus frozen Linkup Deep (`production-2000-search`).
- **Structured direct-evidence A/B, n=300:** 139–79–82; 63.8% decided versus
  current Crunch write→project (`schema-guided-direct-300`).
- **Filtered production, n=200:** 67–32–101; 67.7% decided and 0 observed Crunch
  domain violations versus live Deep. Diagnostic, not launch-ready
  (`filtered-production-200`).
- **Enterprise Vespa-only, n=1,000:** 821 wins, 144 losses, 35 ties; 82.1% row
  wins and 85.1% decided versus stored Deep (`enterprise-vespa-1000`).
- **Corrected ordinary-lookups harness, n=100:** Crunch checklist pass 0.940,
  Exa 0.894, Deep 0.610; treat Crunch and Exa as tied because the overall lead
  leans on agentic prompts (`prod100-corrected`).
- **Hard multi-hop, n=20:** judge results disagree slightly; treat Crunch and
  Exa as tied, with matrix questions Crunch's clearest gap (`hard-bench-20`).
Production comparisons used private, stratified samples. The archive stores
aggregates only. Frozen Deep replays dropped request date/domain filters, so
unfiltered quality scores do not establish filter compliance.
## 15. Rejected alternatives
- One literal search instead of an adaptive loop.
- Mandatory sequential chain discovery.
- Forced one- or two-hop stopping.
- A separate or larger synthesis writer.
- Brave as a second general index under the same read budget.
- Vespa-only for general-web traffic.
- Automatic top-page scraping.
- Deterministic auto-read ranking.
- Search-result concatenation, RRF as the final method, or cross-query raw-score
  ordering.
- Writing prose and discarding it for `searchResults`.
- Mechanical structured abstention.
- Broad schema guidance by itself.
- Direct structured projection from unranked snippets.
Rejected means “not selected for the measured scope,” not impossible forever.
The complete register preserves superseded, invalid, incomplete, and blocked
runs so old results are not rediscovered as new facts.
## 16. Known limitations
- DeepSearchQA uses a replacement autorater and supports paired comparison only.
- Production and enterprise populations are private; only aggregates are kept.
- The enterprise result is one hash-sampled cohort, over-represents a repeated
  wrapper pattern, and is not volume weighted.
- The structured 300-row gain is schema-family dependent, costs more, and does
  not isolate broad guidance from unguided direct projection.
- The older 69.8% structured rescore selected prior losses and reprojected
  stored writes; it is not a fresh full run.
- Filtered completion and `searchResults` regressed; date compliance is not
  observable in returned documents.
- Matrix-shaped questions remain the clearest known reasoning/coverage gap.
- Small pilots establish direction or mechanism, not universal superiority.
- Order swaps can be unstable; margins below that instability are treated as
  ties.
## 17. Sources and deeper records
- [`docs/DECISIONS.md`](docs/DECISIONS.md): compact durable decision log.
- [`docs/EXPERIMENTS.md`](docs/EXPERIMENTS.md): every material adopted,
  rejected, superseded, invalid, diagnostic, incomplete, and blocked run.
- [`docs/MEASUREMENT.md`](docs/MEASUREMENT.md): score definitions, controls,
  invalid instruments, and supersession map.
- [`docs/TIMELINE.md`](docs/TIMELINE.md): chronological build and repair history.
- [`benchmarks.json`](benchmarks.json): machine-readable samples, methods,
  scores, decisions, limitations, supersession, and source paths.
All claims above are bounded to the population and instrument named by their
evidence ID. “Adopted” means supported for that scope, not universally optimal.
