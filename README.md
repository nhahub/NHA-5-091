# Agentic ML Experimentation Platform

An agentic system for tabular machine learning that is built for **validity by
construction**: an LLM Experimenter and an independent LLM Critic work inside a
deterministic harness that owns the splits, runs untrusted code in a sandbox,
computes metrics outside it, and applies hard validity gates. The product is a
trained model plus an evidence trail and a validity verdict, not a leaderboard
score.

## Documentation

| Document | What it covers |
| --- | --- |
| [docs/project.md](docs/project.md) | Full project document: proposal, research questions, experiment design, benchmark, plan, roles, risks, KPIs |
| [docs/simplified-requirements.md](docs/simplified-requirements.md) | One-page checklist: must-haves, configurations, cut order, deadlines |
| [docs/requirements.md](docs/requirements.md) | Testable functional and non-functional requirements |
| [docs/arch.md](docs/arch.md) | Components, authority order, data flow, isolation rules |
| [docs/file-structure.md](docs/file-structure.md) | Repository layout and what belongs where |

## Status

Phase 1 (analysis and design). The source tree is scaffolded in week 1; see
[docs/file-structure.md](docs/file-structure.md).

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).
