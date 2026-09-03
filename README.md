# Crunch benchmarks

**[Design decisions](docs/DECISIONS.md)** — also [CRUNCH-HARNESS-DESIGN.md](CRUNCH-HARNESS-DESIGN.md).

Private, privacy-preserving record of Crunch design decisions and evaluations. Raw customer prompts, answers, secrets, and large logs are intentionally excluded.

## Current evidence

| Scope | n | Crunch result | Comparator |
|---|---:|---:|---|
| DeepSearchQA | 896 | **36.2% all-correct** | Deep 8.4% |
| [Production sourcedAnswer](docs/benchmarks/production-2000.md) | 687 | **84.3% decided** (578–108–1) | Frozen Linkup Deep |
| Production searchResults | 857 | **72.8% decided** (622–233–2) | Frozen Linkup Deep |
| Structured direct projection | 300 | **63.8% decided** (139–79–82) | Current Crunch write→project |
| Filtered production | 200 | **67.7% decided**; 0 observed domain violations | Live Linkup Deep |
| Enterprise Vespa-only | 1,000 | **82.1% of rows; 85.1% decided** | Stored Linkup Deep |

The structured result is an architecture A/B, not a direct replacement for the production-vs-Deep score. Filtered Crunch is not launch-ready because searchResults quality and completion regressed.

## Archive
- [Eval writeups](docs/benchmarks/)
- [Decision archive](docs/DECISIONS-ARCHIVE.md)
- [Complete experiment register](docs/EXPERIMENTS.md)
- [Timeline](docs/TIMELINE.md)
- [Measurement and supersession rules](docs/MEASUREMENT.md)
- [Machine-readable catalog](benchmarks.json)

All headline numbers link back to aggregate local evidence in the catalog. Historical, rejected, invalid, incomplete, and blocked runs remain visible there so they cannot be rediscovered as new facts.
