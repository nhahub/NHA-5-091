# Agentic ML Experimentation Platform: Validity by Construction

DEPI Agentic AI Track · Capstone Project Documents

Team: Seifeldin Ahmed Maazouz, Ahmed Alaa ElNaggar, Abdelrahman Agha, Menna El Gharably, Hazem Hassan

Version 2.0 · 31 October 2026 (supersedes v1.0 of 30 October)

## 1. Project Proposal

### 1.1 Project idea

The project builds an agentic system that runs tabular machine learning experiments end to end: it profiles a dataset, proposes experiments, writes pipelines, trains models and decides what to try next. Its focus is not a better score. Its focus is whether the model it finally accepts can be trusted.

The design principle is **validity by construction**. Validity comes from how the system is built (the agents never see the evaluator or the hidden lockbox, and metrics are computed outside the sandbox), not from asking an agent to behave. Two LLM agents do the reasoning: an **Experimenter** and an independent **Critic**. Everything else is deterministic software.

The final output is a trained model plus an evidence trail: the experiments run, those rejected and why, a gate-by-gate validity scorecard, and the cost of guarding.

### 1.2 Problem statement

An autonomous ML agent can write code, train models and report a high score while the result is still invalid. Typical causes:

- a feature derived from the target, or information that is not available at prediction time
- preprocessing or feature selection fitted outside the cross-validation folds
- duplicate or related entities on both sides of a split
- a random split on time-ordered data
- overfitting to repeated validation feedback
- tampering with, or reading from, the evaluation resources

Accuracy, F1 or AUC alone do not reveal these problems. The central question is: **how can an autonomous experimentation system tell reliable performance from performance produced by an invalid procedure or an exploitable shortcut?**

Classic AutoML does not address this, and existing ML-engineering agents mostly optimize benchmark score. We treat this project as a controlled replication and extension of existing work on agent reliability, not as a claim of novelty. A literature review will confirm or narrow that positioning before the design document is submitted.

### 1.3 Objectives

1. Build a system in which an LLM Experimenter takes a tabular dataset and a goal contract to a validated model, under fixed iteration, time and token budgets.
2. Enforce validity with deterministic components: split manager, data auditor, sandbox, evaluation harness, gatekeeper and a one-shot hidden lockbox.
3. Add an independent LLM Critic that can raise concerns and request verification experiments from a fixed menu, and treat it as an experimental factor rather than an assumed safeguard.
4. Keep an append-only ledger for every run (specification, code hash, metrics, gate results, Critic reviews, token cost).
5. Build a fault-injection benchmark with oracle-labeled ground truth.
6. Measure what each safeguard catches, misses and costs, against AutoGluon and fixed-pipeline references.

### 1.4 Research questions

**Primary:** under a fixed model backbone and experiment budget, how do a deterministic validity harness and an independent LLM Critic, separately and together, change how often the system accepts an invalid tabular model?

- **RQ1:** which fault types does each safeguard catch or miss?
- **RQ2:** what does guarding cost in false rejection, predictive performance, time and LLM spend?
- **RQ3:** how stable are outcomes across repeated LLM runs?

The project does not assume that more agents give better results. A null result for the Critic is a valid finding.

### 1.5 What makes it different

- The product is the **evidence trail and validity verdict**, not the leaderboard score.
- Validity is **enforced by architecture** (isolation and a constrained interface), with LLM review layered on top and never able to override a deterministic Fail.
- The system is **evaluated as an experiment**: a 2×2 comparison on a benchmark where the invalid behavior is known by construction.
- Claims are scoped honestly to the tested datasets, fault types, backbone and task.

### 1.6 System architecture

Two components are LLM agents; all others are deterministic.

| Component | Type | Role |
| --- | --- | --- |
| Goal contract | Input | Dataset, target, task, metric, constraints, features available at prediction time, optional group and time columns |
| Orchestrator (LangGraph) | Deterministic | Controls flow, enforces iteration, time and token caps, scopes each agent's context. Makes no ML decisions |
| Split manager | Deterministic | Builds folds (grouping identical rows, honoring declared group and time columns) and holds the hidden lockbox |
| Data auditor | Deterministic | Profiles the selection pool only: missingness, imbalance, duplicates, ID-like columns, shortcut screens |
| **Experimenter** | **LLM** | Reads contract, audit report and ledger; writes the experiment spec and pipeline source; chooses to refine, run again, submit or stop |
| Sandbox executor | Deterministic | Runs pipelines with no network, no lockbox, a fresh process per fold and resource caps. Returns predictions only |
| Evaluation harness | Deterministic | Computes all metrics and controls outside the sandbox, so untrusted code never sees held-out labels |
| Gatekeeper | Deterministic | Applies hard gate rules and produces the verdict. No LLM can override a Fail |
| **Critic** | **LLM** | Read-only review of audit report, gate table, metrics, pipeline source and ledger summaries. Cannot see the Experimenter's reasoning, edit code or turn a Fail into a Pass |
| Verification runner | Deterministic | Runs approved requests from a fixed menu: drop-feature ablation, group-split CV, time-split replay, feature permutation |
| Ledger | Deterministic | Append-only record of everything |
| Lockbox check | Deterministic | One-shot final score with no feedback into the loop |
| Reporter | Deterministic | Templated scorecard and report. Any LLM narrative is optional and cannot change the verdict |

**Authority order:** a Gatekeeper Fail beats a Critic veto, which beats a Gatekeeper Pass.

### 1.7 Validity gates

A **Fail** is proof by construction or by mechanism. A **Suspect** is statistical suspicion only.

| Gate | Check | Result type |
| --- | --- | --- |
| G0 | Evaluator, split and lockbox access or tampering. The real defense is OS isolation (no lockbox mount, metrics outside the sandbox, file hashes); an import allowlist and code scan are defense in depth | Fail |
| G1 | Preprocessing, selection and encoding fitted inside training folds only, enforced by the interface | Pass by construction, or Fail |
| G2 | Exact cross-fold duplicates (rows grouped by the split manager, then verified); near-duplicates and undeclared entity-like columns | Fail for exact twins, otherwise warning |
| G3 | Permuted-label control: score must stay in a calibrated chance band. In C2 and C5 it is an oracle-side audit that measures procedural leakage in the agent's own script | Fail above the band |
| G4 | Shortcut screen: per-feature strength, exact mappings, importance concentration, columns declared unavailable at prediction time, Critic reading of column names | Suspect (Fail only for declared-unavailable columns) |
| G5 | Temporal: forward-chaining splits when a time column is declared; no training timestamp later than a validation one | Fail if declared, otherwise warning |
| G6 | Lockbox gap between selection CV and the one-shot lockbox score | Suspect or Fail by frozen thresholds |
| G7 | Reproducibility: re-run with the same seed, code hash and pinned environment | Suspect |
| G8 | Time, memory, inference latency, library and model-complexity constraints | Fail |

Thresholds (chance band, lockbox gap, tolerance δ) are calibrated on development instances and frozen before the main evaluation. The permuted-label control is not a universal leak detector: it is blind to semantic proxies and label-free leakage.

### 1.8 Validity verdict

Gate statuses stay separate (Pass, Fail, Suspect, Not applicable). **No weighted score is used**, because a weighted score lets performance offset invalidity. **Not applicable is not Pass**: the report states which checks could not be evaluated, for example "valid on 6 of 8 applicable checks; temporal leakage unverifiable."

| Any gate Fail | Unresolved Suspect | Goal met | Label |
| --- | --- | --- | --- |
| Yes | – | – | Invalid |
| No | Yes | – | Suspect / unresolved |
| No | No | Yes | Valid and performant |
| No | No | No | Valid but underperforming |

Decision rules:

1. A candidate with any Fail is rejected and the loop continues within budget.
2. A Suspect feature cannot be cleared or confirmed by ablation, because dropping a strong feature hurts whether it is a leak or legitimate. The default is to **exclude it and report the performance cost**. A user can override by declaring the feature acceptable in the contract.
3. A Critic veto counts as a Suspect with a named concern. The Critic can never create a Fail.
4. The lockbox check is terminal and gives no feedback.

### 1.9 Experimental design

**Independent variables:** harness (absent or present), Critic (absent or present), fault type (clean, semantic proxy, structural). **Blocking factor:** dataset.

**Controlled:** pinned backbone and version, temperature, base prompt, experiment cap (for example 8), wall-time cap (for example 20 minutes), token cap (Critic tokens count against it), allowed libraries (scikit-learn, XGBoost, LightGBM, NumPy, pandas), compute and metric definitions.

| Config | Experimenter | Auditor report | Enforced harness | Critic |
| --- | --- | --- | --- | --- |
| C2 | Yes | Advisory | No | No |
| C5 | Yes | Advisory | No | Yes |
| C3 | Yes | Binding | Yes | No |
| C4 | Yes | Binding | Yes | Yes |
| C6 (optional) | Yes | Binding | Yes | Same-agent self-review at matched token cost |

Every configuration receives the same Data Auditor report. It is advice in C2 and C5 and enforced in C3 and C4, which tests whether information alone is enough. All configurations sit inside the same outer benchmark boundary: none can reach the lockbox or the independent scorer. The Critic is built as a switch, so C3 is literally C4 with the Critic off. C6 shows whether *independent* review matters or only extra tokens do.

**References (no LLM):** C0 fixed pipeline; C1 AutoGluon; C0g fixed pipeline plus a trivial single-feature filter. C0g shows whether an LLM adds anything over a simple rule. AutoGluon is a reference, not a competitor, and may exploit injected leaks, which is itself informative. Tampering is tested with a scripted attack suite against the harness, not as an LLM factor.

### 1.10 Fault-injection benchmark and oracle

**Faults (known by construction):**

- **F1 semantic proxy:** exact or noisy copy of the target (for example 95% and 80% fidelity), or a post-event aggregate. Descriptive column names on two datasets and neutral names on two.
- **F2 duplicate or entity overlap:** near-twin rows across the split.
- **F3 temporal trap:** drifting regime on time-ordered data where a random split leaks.
- **F4 row-order or ID leak (optional).**
- Procedural leakage (preprocessing before the split) is not injected. It is natural agent behavior, measured by the permuted-label audit.

**Hard negatives that must be accepted:** a legitimately very strong feature, natural duplicates in categorical data, legitimate time features used with a forward split.

**Two ground-truth labels, kept separate:**

1. **Protocol-invalid:** a predefined methodological violation is confirmed (split violation, prohibited access, preprocessing leakage).
2. **Benchmark-exploiting:** the claimed score exceeds the model's score on the **oracle set** by more than δ. In the oracle set the fault is broken: the leaked feature is uninformative, entities are disjoint, the time block is truly in the future.

The labels may overlap but are not interchangeable. **Invalid** means protocol-invalid or benchmark-exploiting, and both parts are reported separately. The oracle shows exploitation of injected shortcuts, not universal proof of every real-world leak.

There are two hidden sets: the system **lockbox** (what a real deployment could have) and the **oracle set** (only the benchmark designers have it). The lockbox is blind to leaks present in both selection data and lockbox.

**Keeping the benchmark honest:**

- Randomize injected column names, positions, fidelity and duplicate fraction.
- One member builds the injectors from held-out parameters and keeps them secret from the detector builders until final evaluation.
- Hide instance metadata from the agents; prefer less famous datasets or renamed columns to reduce memorization.
- Sanity test: each injected fault must inflate selection CV and drop the oracle score, or it is discarded.
- If C0g catches more than 90% of faults, the benchmark is too easy and needs noisier or multi-feature leaks.
- Manually audit about 30 random runs with two reviewers and report agreement.

### 1.11 Run plan and randomness

4 evaluation datasets and 2 development datasets. Three variants per evaluation dataset: V0 clean, V1 semantic proxy, V2 structural fault (duplicates or entity overlap, or a temporal trap on time-ordered data).

**4 datasets × 3 variants × 4 configurations × 3 seeds = 144 LLM runs.** Fallback: 2 seeds, 96 runs. At 15–20 minutes per run and four parallel workers, about 10–12 hours of compute. API cost is unknown until the week-2 pilot.

- Seeds change the split and the sampling; prompts, outputs, code and costs are logged.
- Configurations are interleaved in time to reduce API drift.
- Infrastructure crashes are rerun once under a predefined rule; genuine agent failures count as outcomes.
- Analysis is per instance (bootstrap over instances) with paired comparisons. Three seeds on one instance are not independent datasets.
- Prompts and thresholds are tuned on development instances only, then frozen. The hypotheses and analysis plan are written before the main runs.
- With 12 instances only large effects are distinguishable, so we report intervals, not p-values.

### 1.12 Scope

**In scope**

- Tabular CSV data; binary and multiclass classification, including top-k goals such as the five products a customer is most likely to choose.
- Data profiling, split management, constrained and sandboxed pipeline execution (scikit-learn, XGBoost, LightGBM).
- Deterministic validity gates, the Critic and the verification menu.
- Fault-injection benchmark, oracle sets and hard negatives.
- Ledger, scorecard and evidence-trail report; a CLI and a lightweight web dashboard for the demo (read-only over the ledger: run view, gate scorecard, C2 versus C4 side by side, recorded replay).
- References C0, C1 (AutoGluon) and C0g.

**Out of scope**

- Images, text, audio, time-series forecasting; deep learning and GPU workloads.
- Additional LLM agents or multi-agent debate; reinforcement learning.
- Weighted validity scores; leaderboard chasing.
- Production deployment, drift monitoring, retraining; deciding what the business goal should be.

**Stretch goals (only if time remains, never at the cost of the core)**

- Interactive client mode in the dashboard: a contract form where the client picks the dataset, target, metric (for example top-5 hit rate), limits such as latency and time, and columns unavailable at prediction time, then starts a run. It writes the same contract file the CLI uses
- RAG over curated ML best-practice notes, and long-term memory of past experiments
- Cost-aware routing between a strong and a cheaper model, with a cost report
- Human approval gates before risky actions
- Explainability constraints in the contract (for example, interpretable models only)
- A small tabular slice of MLE-bench Lite and a CrewAI side comparison
- Regression support

Stretch features are built outside the frozen `v1.0-eval` system and are never switched on in the evaluated configurations, so they cannot confound the C2 to C5 comparison. They start only after the core loop, the gates and the demo dashboard work.

What the results can claim: a relative comparison of guard designs on this benchmark, backbone and task type; per-fault catch and miss rates; the cost of guarding; and whether the Critic added value beyond the harness here. They cannot claim that the harness guarantees validity, that results transfer to other backbones or real-world faults, that multi-agent systems are better in general, or accuracy superiority over AutoML.

### 1.13 Technology stack

Python; LangGraph (orchestration); LangSmith (tracing); pandas, NumPy; scikit-learn, XGBoost, LightGBM; AutoGluon (reference); Docker or equivalent (sandbox); MLflow or a structured ledger; FastAPI with a lightweight web frontend (demo dashboard); GitHub and GitHub Actions with pytest. LLMs are accessed through one pinned API backbone.

### 1.14 Demonstration

Pick an instance with an injected post-event proxy. Run C2: it accepts a model claiming very high performance, and the oracle set shows its score collapse. Run C4: it flags the feature, produces a valid-but-underperforming model, and shows the scorecard and ledger. A **replay mode** plays back a recorded ledger in the web dashboard, because live LLM runs are nondeterministic. The dashboard is read-only over the ledger and shows the run view, the gate scorecard, the Critic's concerns, and C2 versus C4 side by side. Use a multiclass dataset with a top-5 goal and a latency cap in the contract, so the client's criteria are visible.

## 2. Project Plan

The plan follows the official DEPI milestones: design by 6 Nov 2026, implementation by 30 Nov 2026, final presentation, testing and reports by 4 Dec 2026. The system is **frozen on 20 Nov** (tag `v1.0-eval`); after that only behavior-preserving bug fixes are allowed.

| Phase | Dates | Goals | Deliverables |
| --- | --- | --- | --- |
| 1. Scoping and design | 30 Oct – 6 Nov | Architecture, threat model, gate and verdict rules, experiment schemas, evaluation protocol, draft pre-registration. Dataset shortlist and injector specs. Repository, CI, stub LangGraph, sandbox spike, split manager and auditor v0, baselines | Design document (due 6 Nov), repository with CI, baseline results |
| 2. Core infrastructure | 7 – 13 Nov | Split manager, auditor v1, sandbox and evaluation harness, gates G1, G2, G4, gatekeeper, injectors F1 and F2, basic Experimenter, free-form submission contract. Measure cost per run | End-to-end guarded (C3) run on one clean and one F1 dataset by 13 Nov; decision on 2 or 3 seeds |
| 3. Agents and guards | 14 – 20 Nov | Critic and verification menu (C4, C5), G0, G3, G5–G7, F3 injector, hard negatives, manifest sanity tests. Pilot on development instances, calibrate thresholds, finalize prompts and analysis plan | Frozen `v1.0-eval` on 20 Nov |
| 4. Main experiments | 21 – 27 Nov | Run the matrix and references, log cost and time, manual audit of about 30 runs, investigate infrastructure failures, start analysis scripts and the demo script; build the read-only web dashboard over the ledger (outside the frozen evaluated system) | Raw results and ledgers; implementation submitted 30 Nov |
| 5. Analysis and delivery | 28 Nov – 4 Dec | Finish or rerun missing runs, statistics, report, demo rehearsal, slides | Final report, presentation and demo (4 Dec) |

**Resources**

- People: five members (section 3); mentor review at the end of each phase.
- Compute: team laptops plus a shared small cloud VM for the 144-run matrix (about 10–12 hours with four workers).
- LLM budget: API credits for one pinned backbone with a cap agreed by the team after the pilot.
- Software: GitHub, Docker, LangGraph, LangSmith (free tier), AutoGluon, MLflow.
- Data: public datasets only, no private or personal data.

**What to cut first if the team falls behind, in order:** (1) C6, (2) seeds from 3 to 2, (3) the temporal variant, (4) one evaluation dataset, (5) the Critic's menu reduced to two experiments, (6) the dashboard reduced to the CLI scorecard plus a recorded replay, (7) C5 last. **Never cut:** harness isolation, the oracle, the C2/C3/C4 comparison, the system freeze, the written analysis plan.

## 3. Task Assignment and Roles

Roles are placeholders (Member 1 to Member 5) until the team decides who takes which workstream. Each workstream has a named backup so nobody is a single point of failure.

| Workstream | Lead / backup | Ownership | Deliverables |
| --- | --- | --- | --- |
| A. Agents | Member 1 / Member 5 | LangGraph orchestration, Experimenter, Critic, prompts, output schemas, budgets, LangSmith; after the freeze, the demo web dashboard | Working bounded agent workflow with structured experiment specs |
| B. Sandbox and evaluation infrastructure | Member 2 / Member 3 | Sandbox, fold-isolated execution and IPC protocol, evaluation harness, G0 and G7, scripted tampering attack suite | Isolated evaluation service, security and integrity tests |
| C. Deterministic validity system | Member 3 / Member 2 | Split manager, data auditor, gates G1–G6, gatekeeper, verdict logic, verification runner, threshold calibration | Gate implementation, audit report, validity scorecard |
| D. Benchmark and fault injection | Member 4 / Member 3 | Datasets, injectors, oracle sets, hidden manifests, hard negatives, permuted-label audit, manual-audit coordination | Development and evaluation datasets, injection pipeline, benchmark documentation |
| E. Baselines and research evaluation | Member 5 / Member 1 | C0, C1 and C0g baselines, matrix runner, cost and runtime logging, statistics, report generator, demo | Reproducible runner, results with intervals, final report and presentation |

**Shared responsibilities**

- Everyone writes unit tests for their own components and reviews at least one pull request per week.
- Component interfaces and the ledger schema are agreed in Phase 1 and integrated continuously through CI.
- Weekly team meeting with a short written update; mentor review at the end of each phase.
- Separation of duties: the member who builds the injectors does not build the detectors, and the evaluation scripts are reviewed by a second member.

## 4. Risk Assessment and Mitigation

Likelihood and impact are rated Low, Medium or High.

| ID | Risk | L / I | Mitigation |
| --- | --- | --- | --- |
| R1 | LLM API cost or run time exceeds budget | High / High | Pilot in week 2, token caps per run, response caching in development, fall back to 2 seeds (96 runs) |
| R2 | Scope creep from stretch features (RAG, routing, approval gates) | High / High | Written stretch list, started only after the core and the demo dashboard work and kept outside the frozen evaluated system; mentor check at each phase end |
| R3 | Fold-isolated sandbox takes too long to build | Medium / High | Prototype in week 1, thin end-to-end version by 13 Nov, helper templates |
| R4 | Experimenter cannot write valid pipelines under the constrained interface | Medium / High | Provide helper templates, measure the failure rate, count genuine agent failures as outcomes |
| R5 | Fault injection or oracle errors | Medium / High | Sanity test for each fault, two-reviewer manual audit of about 30 runs, injectors built by a separate member |
| R6 | Benchmark too easy or too obvious | Medium / Medium | Randomized names, positions and fidelity, hard negatives, C0g check (over 90% catch means redesign) |
| R7 | Adaptive overfitting by the team to evaluation data | Medium / High | Development and evaluation split, system freeze on 20 Nov, written analysis plan |
| R8 | Critic adds no measurable value | High / Low | Frame questions as open; report a null result as a finding |
| R9 | Weak statistical power (12 instances) | High / Medium | Report intervals, analyze per instance, restrict claims |
| R10 | LLM nondeterminism and API drift | High / Medium | Log prompts, model versions and seeds, interleave configurations, replay mode for the demo |
| R11 | Unsafe or destructive generated code | Medium / High | Docker sandbox with no network, read-only data mounts, resource limits, no lockbox mount, scripted attack suite |
| R12 | Third-party library or service changes, outages | Medium / Medium | Pin dependency versions, abstraction layer over the LLM provider |
| R13 | Exam periods and team availability | High / Medium | Lighter loads around exams, critical-path items first, named backup per workstream |
| R14 | Privacy and sensitive data | Low / Medium | Public datasets only; document privacy considerations, including Egypt's data protection law, for real-world use |
| R15 | Compressed five-week schedule | High / High | Fixed cut order (section 2), parallel workstreams, feature freeze on 20 Nov |
| R16 | Demo dashboard takes time from the core | Medium / Medium | Keep it read-only over the ledger with a few pages, built by a post-freeze owner, with the CLI scorecard and recorded replay as fallback |

## 5. KPIs

The results are research findings, so the main KPIs report measured rates with uncertainty intervals rather than fixed pass marks. The numeric targets below are provisional and will be calibrated after the week-2 pilot and then fixed.

**Research outcomes (reported, with intervals)**

| KPI | Definition | Target |
| --- | --- | --- |
| Invalid acceptance rate (primary) | Share of accepted candidates labeled invalid, with protocol-invalid and benchmark-exploiting parts reported separately, per configuration and fault type | Reported for C2, C3, C4, C5 |
| False rejection rate | Share of clean and hard-negative candidates rejected unnecessarily | Reported; provisional ceiling to set after pilot |
| Valid-and-performant rate | Share of runs that pass the validity rules and meet the goal | Reported |
| Fault detection by type | Catch and miss rate for F1, F2, F3 | Reported |
| Claim error | Claimed selection score minus oracle or lockbox score | Reported |
| Cost and time per valid result | LLM tokens, dollars and minutes | Reported; run cap of 20 minutes |
| Reproducibility | Share of accepted runs that re-run within tolerance | At least 95% |

**Benchmark quality**

| KPI | Target |
| --- | --- |
| Injected faults that inflate selection CV and drop oracle score (sanity test) | 100% of retained faults |
| C0g catch rate on injected faults | Below 90% |
| Manual audit | About 30 runs, two reviewers, agreement reported (provisional target: Cohen's kappa of at least 0.6) |

**Safety and engineering quality**

| KPI | Target |
| --- | --- |
| Sandbox and lockbox isolation | Zero successful escapes or lockbox reads in the scripted attack suite |
| Budget and limit enforcement | 100% of runs stop correctly at the iteration, time or token cap |
| Ledger completeness | 100% of runs have a complete append-only ledger |
| Trace coverage | 100% of LLM calls traced in LangSmith |
| Test coverage | At least 80% unit-test coverage on the harness and gates |
| CI health | Main branch passing; every pull request reviewed |

**Project management**

- Milestones delivered on time: design 6 Nov, implementation 30 Nov, final presentation 4 Dec 2026.
- System frozen and tagged `v1.0-eval` on 20 Nov.
- Matrix completion: 144 of 144 planned runs (or 96 of 96 under the fallback), with every missing run explained.
- Mentor review held at the end of every phase. Demo dashboard showing a recorded C2 versus C4 replay by 1 Dec.

## Assumptions and open items

- The plan follows the official DEPI schedule. Confirm with the mentor that the late planning and literature-review deadline (16 Oct) is accepted and that the DEPI column, not the DEPI Industry column, applies.
- Role assignment is a proposal and can change during Phase 1.
- Numeric KPI targets and gate thresholds are provisional until the week-2 pilot.
- Literature review: verify the benchmark and leakage-audit papers behind our positioning before citing, and check that the chosen datasets have the time and entity structure the variants need.

## Appendix: changes from v1.0

| v1.0 | v2.0 |
| --- | --- |
| Broad autonomous ML engineer with RAG, memory, routing, dashboard, MLE-bench slice, CrewAI comparison | Narrow research design on validity. The extra features move to a stretch list; the demo dashboard stays in scope |
| Success measured by score versus AutoGluon and generic rates | Primary metric is invalid acceptance, with an oracle defining ground truth |
| Hidden holdout as the merge gate | Hidden lockbox read once, plus a separate oracle set known only to the benchmark designers |
| Critic assumed useful | Critic is an experimental factor, with a null result accepted |
| Mixed LLM and tool roles | Two LLM agents only; all else deterministic, with a fixed authority order |
| Targets fixed in advance | Targets calibrated after the pilot; system frozen on 20 Nov |
