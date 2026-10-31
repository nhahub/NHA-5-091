# Simplified Requirements

One page. Full detail in [project.md](project.md) and [requirements.md](requirements.md).

## Goal

An LLM Experimenter takes a tabular dataset and a goal contract to a validated
model under fixed iteration, time and token budgets. A deterministic harness
enforces validity; an independent LLM Critic reviews but cannot override a hard
failure.

**Primary research question:** under a fixed backbone and budget, how do the
deterministic harness and the LLM Critic, separately and together, change how
often the system accepts an invalid tabular model?

## Must have

| # | Requirement | Done when |
| --- | --- | --- |
| 1 | Goal contract | Dataset, target, metric, constraints, prediction-time unavailable columns, optional group and time columns |
| 2 | Split manager and hidden lockbox | Folds owned by the system; lockbox never mounted for agents |
| 3 | Data auditor | Profiles the selection pool only and reports missingness, imbalance, duplicates, ID-like columns, shortcut screens |
| 4 | Constrained pipeline interface | Agents write pipelines through a fixed interface; no access to evaluator or lockbox |
| 5 | Sandbox executor | Fresh process per fold, no network, no lockbox, resource caps, predictions only |
| 6 | Evaluation harness | All metrics computed outside the sandbox |
| 7 | Gates G0–G8 and gatekeeper | Hard Pass / Fail / Suspect / Not applicable; no LLM can override a Fail |
| 8 | Experimenter agent | Bounded loop: propose, run, interpret, refine, submit or stop |
| 9 | Critic agent | Read-only review; can veto and request verification experiments |
| 10 | Verification runner | Fixed menu: drop-feature ablation, group-split CV, time-split replay, feature permutation |
| 11 | Ledger | Append-only record of specs, code hashes, metrics, gate results, reviews and cost |
| 12 | Benchmark and oracle | Fault injectors F1–F4, oracle sets, hidden manifests, hard negatives |
| 13 | Baselines | C0 fixed pipeline, C1 AutoGluon, C0g fixed pipeline plus single-feature filter |
| 14 | Runner and scorecard | Matrix runner, cost and runtime logging, validity scorecard and report |

## Guard configurations

| Config | Experimenter | Auditor report | Harness | Critic |
| --- | --- | --- | --- | --- |
| C0 | No (fixed pipeline) | n/a | n/a | n/a |
| C1 | No (AutoGluon) | n/a | n/a | n/a |
| C0g | No (fixed + filter) | n/a | n/a | n/a |
| C2 | Yes | Advisory | No | No |
| C5 | Yes | Advisory | No | Yes |
| C3 | Yes | Binding | Yes | No |
| C4 | Yes | Binding | Yes | Yes |
| C6 (optional) | Yes | Binding | Yes | Same-agent self-review |

## Out of scope

Images, text, audio, time-series forecasting, deep learning and GPUs;
additional agents or multi-agent debate; reinforcement learning; weighted
validity scores; production deployment and drift monitoring.

Dashboard is in scope but read-only over the ledger. Stretch items (RAG, memory,
model routing, human approval gates, MLE-bench slice, CrewAI comparison,
regression) are built only after the core works and never switched on in the
evaluated configurations.

## Cut order if behind

1. C6
2. Seeds from 3 to 2 (144 to 96 runs)
3. Temporal variant (F3)
4. One evaluation dataset
5. Critic menu reduced to two experiments
6. Dashboard reduced to CLI scorecard plus recorded replay
7. C5, last

**Never cut:** harness isolation, the oracle, the C2/C3/C4 comparison, the
fixed evaluation version, and the written analysis plan.

## Deadlines

| Milestone | Date |
| --- | --- |
| System analysis and design | 6 Nov 2026 |
| Guarded C3 vertical slice | 13 Nov 2026 |
| Evaluation version fixed (commit hash recorded) | 20 Nov 2026 |
| Implementation submitted | 30 Nov 2026 |
| Final presentation, testing, reports | 4 Dec 2026 |
