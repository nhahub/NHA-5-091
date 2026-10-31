# Requirements

Testable requirements derived from [project.md](project.md). If the two
disagree, `project.md` is authoritative.

## Functional requirements

### FR-1 Goal contract

The system accepts a typed goal contract containing:

- dataset reference and target column
- task type and primary metric, including top-k goals
- success threshold
- constraints: wall time, compute, model complexity, inference latency
- columns unavailable at prediction time
- optional group column and time column
- optional feature acceptability overrides

Invalid or contradictory contracts are rejected before any run starts.

### FR-2 Split manager and lockbox

- The split manager creates all folds and groups exact duplicate rows.
- Declared group and time columns are honored; time data uses forward-chaining.
- The lockbox is held by the system, is never mounted in the sandbox, and is
  read once at the end with no feedback into the loop.
- The oracle set is separate from the lockbox and is accessible only to
  benchmark tooling.

### FR-3 Data auditor

- Profiles the selection pool only.
- Reports missingness, class imbalance, duplicate rows, ID-like columns,
  per-feature strength, and candidate shortcut columns.
- Never returns the lockbox or oracle to an agent.

### FR-4 Constrained pipeline interface

- The Experimenter submits an experiment spec plus pipeline source that
  implements a fixed interface.
- Preprocessing, encoding and feature selection must be fitted inside
  training folds; the interface makes fitting outside folds impossible.
- Agents cannot import evaluation modules, read lockbox paths, or write metrics.

### FR-5 Sandbox executor

- One fresh process per fold, no network, no lockbox mount, read-only data.
- CPU, memory and wall-time caps are enforced.
- Returns predictions only; untrusted code never sees held-out labels.
- A scripted attack suite verifies isolation.

### FR-6 Evaluation harness

- Computes all metrics and cost outside the sandbox.
- Enforces budgets and stops runs at iteration, time or token caps.
- Performs the one-shot lockbox check.

### FR-7 Gates and gatekeeper

| Gate | Check | Result |
| --- | --- | --- |
| G0 | Evaluator, split or lockbox access or tampering | Fail |
| G1 | Preprocessing and selection fitted inside training folds | Pass by construction or Fail |
| G2 | Exact cross-fold duplicates, near-duplicates, undeclared entity columns | Fail for exact twins, otherwise warning |
| G3 | Permuted-label control within calibrated chance band | Fail above band |
| G4 | Shortcut screen and declared-unavailable columns | Suspect; Fail only for declared-unavailable columns |
| G5 | Temporal ordering when a time column is declared | Fail if declared, otherwise warning |
| G6 | Lockbox gap against fixed thresholds | Suspect or Fail |
| G7 | Reproducibility with same seed, code hash and environment | Suspect |
| G8 | Time, memory, latency, library and complexity constraints | Fail |

- Statuses are kept separate: Pass, Fail, Suspect, Not applicable.
- Not applicable is not Pass.
- No weighted score is used.
- A Gatekeeper Fail cannot be overridden by any agent.

### FR-8 Experimenter agent

- Reads the contract, audit report and ledger summary.
- Writes an experiment spec and pipeline source, runs it, interprets results,
  and chooses refine, rerun, submit or stop.
- Stays within the iteration, time and token caps.

### FR-9 Critic agent

- Read-only over audit report, gate table, metrics, pipeline source and ledger
  summaries.
- Cannot see the Experimenter's reasoning, edit code, or turn a Fail into a Pass.
- Can veto, which counts as a Suspect with a named concern.
- Can request experiments from a fixed verification menu.

### FR-10 Verification runner

Runs only predefined checks: drop-feature ablation, group-split CV, time-split
replay, feature permutation.

### FR-11 Ledger

- Append-only JSONL record per run.
- Contains contract, experiment specs, code hash, commit hash, metrics, gate
  results, Critic reviews, token cost, timestamps and model version.
- Complete for 100% of runs.

### FR-12 Benchmark and oracle

- Injectors F1 semantic proxy, F2 duplicate or entity overlap, F3 temporal trap,
  optional F4 row-order or ID leak.
- Each retained fault must inflate selection CV and drop the oracle score.
- Two labels, reported separately: protocol-invalid and benchmark-exploiting.
- Hard negatives must be accepted: a legitimately strong feature, natural
  categorical duplicates, legitimate time features with a forward split.
- Instance metadata is hidden from agents.

### FR-13 Baselines

C0 fixed pipeline, C1 AutoGluon, C0g fixed pipeline plus a single-feature
filter. No LLM in any of them.

### FR-14 Runner, reporting and dashboard

- Matrix runner executes the configured matrix and logs cost and runtime.
- Scorecard reports per-run verdict, gate table and claim error.
- Report is templated; optional LLM narrative cannot change the verdict.
- Dashboard is read-only over the ledger and supports replay.

## Non-functional requirements

| ID | Requirement | Target |
| --- | --- | --- |
| NFR-1 | Sandbox and lockbox isolation | Zero successful escapes or reads in the attack suite |
| NFR-2 | Budget enforcement | 100% of runs stop at iteration, time or token cap |
| NFR-3 | Reproducibility | At least 95% of accepted runs re-run within tolerance |
| NFR-4 | Ledger completeness | 100% of runs |
| NFR-5 | Trace coverage | 100% of LLM calls traced |
| NFR-6 | Test coverage | At least 80% on harness and gates |
| NFR-7 | CI health | Main green; every PR reviewed |
| NFR-8 | Pinned environment | Dependencies and LLM backbone versions pinned |

## Verdict rules

| Any Fail | Unresolved Suspect | Goal met | Label |
| --- | --- | --- | --- |
| Yes | – | – | Invalid |
| No | Yes | – | Suspect / unresolved |
| No | No | Yes | Valid and performant |
| No | No | No | Valid but underperforming |

1. A candidate with any Fail is rejected; the loop continues within budget.
2. A Suspect feature is excluded by default and the performance cost reported;
   the user may accept it in the contract.
3. A Critic veto is a Suspect with a named concern; never a Fail.
4. The lockbox check is terminal and gives no feedback.

## Key performance indicators

Reported with uncertainty intervals, not pass marks:

- invalid acceptance rate, split into protocol-invalid and benchmark-exploiting
- false rejection rate
- valid-and-performant rate
- fault detection by type
- claim error (claimed score minus oracle or lockbox score)
- cost and time per valid result
- reproducibility rate

Benchmark quality: 100% of retained faults pass the sanity test, C0g catches
under 90%, manual audit of about 30 runs by two reviewers with agreement
reported (provisional Cohen's kappa of at least 0.6).
