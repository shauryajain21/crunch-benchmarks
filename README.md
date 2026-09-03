# Crunch benchmarks

[Major decisions for Crunch](CRUNCH-HARNESS-DESIGN.md)

Private, privacy-preserving record of Crunch design decisions and evaluations. Raw customer prompts, answers, secrets, and large logs are intentionally excluded.

## Current evidence

| Scope | n | Crunch result | Comparator |
|---|---:|---:|---|
| [DeepSearchQA](docs/benchmarks/deepsearchqa-896.md) | 896 | **36.2% all-correct** | Deep 8.4% |
| [Production sourcedAnswer](docs/benchmarks/production-2000.md) | 687 | **84.3% decided** (578–108–1) | Frozen Linkup Deep |
| [Production searchResults](docs/benchmarks/production-2000.md) | 857 | **72.8% decided** (622–233–2) | Frozen Linkup Deep |
| [Structured direct projection](docs/benchmarks/schema-guided-direct-300.md) | 300 | **63.8% decided** (139–79–82) | Current Crunch write→project |
| [Filtered production](docs/benchmarks/filtered-production-200.md) | 200 | **67.7% decided**; 0 observed domain violations | Live Linkup Deep |
| [Enterprise Vespa-only](docs/benchmarks/enterprise-vespa-1000.md) | 1,000 | **82.1% of rows; 85.1% decided** | Stored Linkup Deep |
