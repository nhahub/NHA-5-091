# Agentic ML Experimentation Platform

DEPI Agentic AI Track · Capstone Project Documents

Team: Seifeldin Ahmed Maazouz, Ahmed Alaa ElNaggar, Abdelrahman Agha, Menna El Gharably, Hazem Hassan

Version 1.0 · 30 October 2026

## 1. Project Proposal

### 1.1 Project idea

The Agentic ML Experimentation Platform is an autonomous machine learning engineer. The user provides a tabular dataset, a business objective (for example, "find the customers most likely to buy"), a success criterion (for example, "top-decile lift above 3x" or "F1 above 0.85") and constraints (time budget, explainability, cost). Instead of training a single model, the agent inspects the data, detects quality problems, class imbalance and data leakage, proposes experiments, writes and runs Python code in a sandbox, trains several models, evaluates the results, detects overfitting, changes preprocessing or hyperparameters, reruns, and keeps a full experiment history.

The final output is not a conversation. It is a trained model plus an evidence trail: a readable report explaining which experiments were run, which were rejected and why, and why the final model was chosen.

### 1.2 Problem statement

- Most of the time in a modeling project goes to data preparation, debugging and re-running failed experiments.
- Small and mid-sized data teams and analysts often have no dedicated MLOps or senior data science support.
- Classic AutoML searches over configurations but does not reason about why an approach failed, and it rarely explains its decisions in a way a manager can defend.
- Common mistakes are silent and costly: data leakage gives a model that looks excellent and fails in production, a random split on time-dependent customer data inflates results, and accuracy on imbalanced data hides a useless model.
- Existing research agents for ML engineering mainly optimize benchmark score. Auditability, respect for business constraints and cost control are secondary concerns.

### 1.3 Objectives

1. Build a multi-agent system that autonomously takes a dataset and an objective to a validated trained model.
2. Make the agent self-correcting: it detects leakage, overfitting and imbalance problems, explains them, and changes strategy.
3. Produce a complete, reproducible evidence trail for every run (hypothesis, change, result, decision).
4. Respect a user-defined goal contract: objective, metric, constraints and budget.
5. Keep the agent safe and controllable: sandboxed code execution, a hidden holdout set it can never see, iteration and cost limits, and optional human approval gates.
6. Reduce cost by routing routine work to cheaper models and reserving a stronger model for planning and critique.
7. Evaluate honestly against a strong baseline (AutoGluon) and on datasets with injected problems.

### 1.4 What makes it different

Existing systems such as AIDE, MLE-STAR and the AutoGluon Assistant (MLZero) are strong at benchmark performance. Our project positions itself as the ML engineer you can audit:

- A goal contract that the agent must respect and report against.
- An evidence trail as a first-class product, not a log file.
- Mandatory guardrails: hidden holdout, leakage and overfitting checks, and time-based splits when the data requires them.
- Optional human approval before risky actions.
- Cost-aware model routing, including support for local or cheaper models.

### 1.5 Example use case

A marketing analyst at a company uploads customer data and states: "Find the top 10% of customers most likely to buy, the model must be explainable." The agent profiles the data, notices that a feature was computed after the purchase (leakage), switches to a time-based split, handles the 2% conversion imbalance, tries several models, optimizes lift in the top decile, and delivers the model together with a report the analyst can defend to a manager.

### 1.6 Scope

**In scope**

- Tabular data (CSV and database exports).
- Binary and multiclass classification as the main task. Regression only if time remains.
- Automatic data profiling: missing values, duplicates, outliers, imbalance, leakage candidates, sensitive or personal columns flagged.
- Experiment planning, code generation, sandboxed execution, cross-validation, overfitting detection, and strategy adjustment.
- Models: scikit-learn, XGBoost, LightGBM, plus AutoGluon as a callable baseline tool.
- Goal contract (objective, metric, constraints, budget).
- Experiment ledger and memory of past runs, with retrieval of ML best-practice knowledge (RAG).
- Evidence-trail report generation.
- Human approval gates (optional per run).
- Simple interface: CLI plus a lightweight web dashboard.
- Evaluation on public datasets (for example churn, bank marketing, credit default) and datasets with injected issues; a small tabular slice of MLE-bench Lite if compute allows.

**Out of scope**

- Images, text, audio and time-series forecasting as input modalities.
- Deep learning model training and GPU-heavy workloads.
- True personalized recommender systems and uplift (causal) modeling (listed as future work).
- Production deployment at scale, drift monitoring and automated retraining.
- Handling datasets larger than what a single machine can process comfortably.
- Automatically deciding what the business goal should be: the agent chooses how to reach the target, never whether the target is the right one.

### 1.7 Technology stack

Python, LangGraph (agent orchestration), LangSmith (agent tracing and evaluation), MLflow (experiment tracking), scikit-learn, XGBoost, LightGBM, AutoGluon (baseline tool), pandas and NumPy, Docker (sandboxed execution), a vector store such as Chroma or FAISS (RAG and memory), FastAPI and a simple web frontend, GitHub and GitHub Actions. LLMs are accessed through APIs, with a strong model for planning and critique and a cheaper or local model for routine code generation. CrewAI is explored in a small side comparison for learning purposes.

### 1.8 Learning outcomes

Multi-agent design, stateful orchestration with LangGraph, tool calling, short-term and long-term memory, RAG, human-in-the-loop control, guardrails and sandboxing, tracing and evaluation of agents with LangSmith, and cost-aware LLM routing.

## 2. Project Plan

The plan follows the official DEPI schedule: system analysis and design by 6 Nov 2026, implementation (source code and execution) by 30 Nov 2026, and final presentation, testing and reports by 4 Dec 2026. The full scope is kept. Work is organized in six phases, and phases 3 to 5 run in parallel across the five workstreams in section 3.

### Phase 1 – Scoping, Analysis and Design (30 Oct – 6 Nov 2026)

- Submit the project name, description, team and repository link (LMS, 31 Oct).
- Finalize requirements, goal-contract format, architecture diagrams, use cases and ledger schema.
- Turn the competitor review into the literature review section.
- Select and prepare 8–10 evaluation datasets and the injected-issue versions; run the AutoGluon baseline.
- Set up the GitHub repository, CI, Docker sandbox and MLflow; start LangSmith tracing.
- **Deliverables (due 6 Nov):** system analysis and design document, repository with CI, baseline results.

### Phase 2 – Core Tools and Executor (7 – 13 Nov 2026)

- Build and unit-test the tool library: data profiler, leakage scanner, split manager (random and time-based), cross-validation runner, overfitting detector, metric calculator.
- Build the sandbox Executor with timeouts and no network access.
- Build the Coder component that turns an experiment spec into runnable code.
- **Deliverables:** tested tool library, working end-to-end run on one dataset, sandbox security tests.

### Phase 3 – Agent Loop (14 – 20 Nov 2026)

- Build the LangGraph loop: Orchestrator, Critic, accept/adjust/stop branching.
- Implement the hidden-holdout merge gate (a change is accepted only if it improves hidden validation).
- Connect experiment logging to MLflow and tracing to LangSmith.
- **Deliverables:** self-correcting loop working on 3 datasets, ledger with full history, first live demo.

### Phase 4 – Intelligence Features (14 – 27 Nov 2026, parallel with Phase 3)

- Goal-contract parsing and enforcement.
- Long-term memory of past experiments and RAG over ML best-practice notes.
- Model routing between a strong and a cheaper model, with cost tracking.
- Human approval gates.
- **Deliverables:** goal-contract support, memory and RAG module, routing with cost report, approval flow.

### Phase 5 – Reporting, Interface and Evaluation (14 – 30 Nov 2026, parallel with Phases 3 and 4)

- Evidence-trail report generator, CLI and web dashboard.
- Full evaluation: all datasets, AutoGluon baseline comparison, injected-issue tests, local-versus-strong-model comparison, MLE-bench Lite slice and the CrewAI side comparison if time allows.
- **Deliverables (due 30 Nov):** implementation with source code and execution, report generator, interface, evaluation results with charts.

### Phase 6 – Hardening and Final Delivery (1 – 4 Dec 2026)

- Bug fixing, performance and cost tuning, security review.
- Final documentation, user guide, demo dataset with hidden leakage.
- Final presentation and demo rehearsal.
- **Deliverables (due 4 Dec):** testing results, final report, final presentation and live demo.

### Priority tiers

The schedule is tight, so every feature stays in scope but is ranked. If time runs short, the lowest tier slips first.

- **Tier 1 (must be done by 30 Nov):** sandbox, tool library, agent loop, hidden-holdout gate, ledger and MLflow, LangSmith tracing, report generator, AutoGluon baseline comparison, leakage demo.
- **Tier 2 (should be done by 30 Nov):** goal contract, memory and RAG, model routing with cost tracking, CLI and web dashboard, human approval gates.
- **Tier 3 (stretch, may continue after the final presentation):** MLE-bench Lite slice, CrewAI side comparison, multiclass and regression support.

### Resources required

- **People:** five team members (see section 3); mentor reviews at the end of each phase.
- **Compute:** team laptops are enough for tabular data; a shared small cloud VM or a free-tier service for long evaluation runs.
- **LLM budget:** API credits for a strong model and a cheaper model, with a monthly cap agreed by the team; local models for cost experiments.
- **Software:** GitHub, Docker, MLflow, LangGraph, LangSmith (free tier), AutoGluon, a vector store.
- **Data:** public datasets only, no private or personal data.
- **Knowledge base:** curated ML best-practice documents for the RAG module.

## 3. Task Assignment and Roles

Roles below are a flexible mock-up: Member 1 to Member 5 are placeholders until the team decides who takes which workstream. Names are filled in once decided, and workstreams can be adjusted to each member's strengths during Phase 1. Each workstream also has a named backup so that no one is a single point of failure.

**Workstream A – Agent orchestration and loop (lead: Member 1, backup: Member 2)**

- LangGraph graph design, Orchestrator and Critic logic, accept/adjust/stop control flow.
- Hidden-holdout merge gate, iteration and budget limits.
- Memory integration and model routing.

**Workstream B – Data tools and ML experimentation (lead: Member 2, backup: Member 3)**

- Data profiler, leakage scanner, split manager, overfitting detector.
- Model training templates, AutoGluon baseline tool, metric calculation (including lift and PR-AUC).
- Experiment specification format.

**Workstream C – Infrastructure, sandbox and backend (lead: Member 4, backup: Member 5)**

- Docker sandbox, execution security, timeouts, resource limits.
- MLflow and LangSmith setup, CI with GitHub Actions, repository structure.
- Backend API (FastAPI) and storage of the ledger.

**Workstream D – Evaluation, benchmarks and KPIs (lead: Member 3, backup: Member 1)**

- Dataset selection, injected-issue datasets, baseline runs.
- Evaluation scripts, KPI measurement, results charts.
- Agent evaluation in LangSmith, cost and latency tracking.

**Workstream E – Reporting, interface and documentation (lead: Member 5, backup: Member 4)**

- Evidence-trail report generator, CLI and web dashboard.
- RAG knowledge base preparation and retrieval evaluation.
- Documentation, user guide, presentations and demo script.

**Shared responsibilities**

- Everyone writes unit tests for their own components and reviews at least one pull request per week.
- Weekly team meeting and a short written progress update; a monthly mentor review.
- The CrewAI side comparison is done by Workstream A and B members as a small learning task in Phase 4.

## 4. Risk Assessment and Mitigation

Likelihood and impact are rated Low, Medium or High.

**R1 – LLM API cost grows beyond budget** (likelihood: High, impact: High)

- Mitigation: monthly budget cap with automatic stop, per-run cost limits, cheaper or local models for routine steps, response caching, and small evaluation runs during development.

**R2 – Scope creep** (likelihood: High, impact: High)

- Mitigation: fixed scope of tabular classification, a written out-of-scope list, new features only after the core loop is evaluated, and a mentor check at each phase end.

**R3 – The agent loops forever or makes poor decisions** (likelihood: Medium, impact: High)

- Mitigation: hard limits on iterations, time and cost; a Critic that must justify continuing; structured ledger to detect repeated actions; LangSmith traces to debug decisions.

**R4 – Unsafe or destructive generated code** (likelihood: Medium, impact: High)

- Mitigation: Docker sandbox with no network access, read-only data mounts, resource and time limits, and a library of tested tools so the agent writes less free-form code.

**R5 – Data leakage in our own evaluation** (likelihood: Medium, impact: High)

- Mitigation: the agent never sees the final holdout, a deterministic hidden validation split, and independent review of the evaluation scripts by a second team member.

**R6 – Difficulty integrating components** (LangGraph, sandbox, MLflow, LangSmith, RAG) (likelihood: Medium, impact: Medium)

- Mitigation: define interfaces and the ledger schema in Phase 1, build a thin end-to-end version early (Phase 2), and integrate continuously with CI.

**R7 – Weaker or cheaper LLMs produce lower-quality code and decisions** (likelihood: High, impact: Medium)

- Mitigation: route planning and critique to the strong model, use validated tools instead of free-form code, and measure the quality gap as a KPI rather than hiding it.

**R8 – LLM outputs are non-deterministic, making results hard to reproduce** (likelihood: High, impact: Medium)

- Mitigation: log prompts, model versions and seeds, store every generated script in the ledger, and evaluate over multiple runs with averages.

**R9 – Benchmark compute is too heavy** (likelihood: Medium, impact: Medium)

- Mitigation: use MLE-bench Lite or a small tabular slice only, check feasibility in Phase 1, and rely mainly on our own datasets if needed.

**R10 – Academic exam periods reduce availability** (likelihood: High, impact: Medium)

- Mitigation: plan lighter workloads around exam weeks, finish critical-path items before them, and keep a backup for every workstream.

**R11 – Third-party service or library changes or outages** (likelihood: Medium, impact: Medium)

- Mitigation: pin dependency versions, keep an abstraction layer over the LLM provider, and keep a local-model fallback.

**R12 – Privacy and sensitive data concerns** (likelihood: Low, impact: Medium)

- Mitigation: use only public datasets, flag personal and sensitive columns in the profiler, and document privacy considerations (including Egypt's data protection law) for real-world use.

**R13 – Team member unavailable or unequal contribution** (likelihood: Medium, impact: Medium)

- Mitigation: named backups, weekly progress updates, pull-request reviews, and a shared task board.

**R14 – Compressed schedule: about five weeks from design to implementation** (likelihood: High, impact: High)

- Mitigation: priority tiers in the plan, five workstreams running in parallel, a thin end-to-end version by 13 Nov, a feature freeze on 30 Nov, and a weekly check against the official milestones.

## 5. KPIs

The targets below are proposed. They will be calibrated against the AutoGluon baseline results at the end of Phase 1 and then fixed.

### Agent effectiveness

- **End-to-end success rate:** at least 90% of runs on our evaluation datasets produce a valid trained model with a report, without human intervention.
- **Leakage detection rate:** at least 80% of injected leakage cases are detected, with a false-positive rate of at most 10%.
- **Overfitting detection:** at least 85% of injected overfitting scenarios are detected and corrected.
- **Self-correction improvement:** the final model beats the first-iteration model on hidden validation in at least 80% of runs.

### Model quality

- **Performance versus baseline:** the final model's primary metric is within 2% (relative) of the AutoGluon baseline, or better, on at least 70% of datasets.
- **Goal-contract compliance:** at least 90% of runs satisfy the stated metric target and constraints, or clearly report why they could not.
- **Generalization gap:** difference between validation and hidden holdout score of at most 3 percentage points on average.

### Efficiency and cost

- **Cost per run:** at most a fixed cap agreed in Phase 1 (target: low single-digit US dollars per run) using routing between strong and cheap models.
- **Cost saving from routing:** at least 30% lower cost than using the strong model for every step, with a quality drop of at most 5%.
- **Time to final model:** at most 30 minutes for datasets up to 100,000 rows.
- **Local-versus-strong model gap:** measured and reported (no fixed target).

### Transparency and reproducibility

- **Evidence-trail coverage:** 100% of runs have a complete ledger (every experiment with hypothesis, change, result and decision).
- **Reproducibility:** at least 95% of logged experiments can be re-run from the ledger and reproduce their metrics within tolerance.
- **Report quality:** average of at least 4 out of 5 on a rubric (clarity, correctness, usefulness) scored by at least three reviewers.
- **Trace coverage:** 100% of LLM and tool calls traced in LangSmith.

### Safety and engineering quality

- **Sandbox safety:** zero successful escapes or network accesses in the security test suite.
- **Budget and limit enforcement:** 100% of runs stop correctly when iteration, time or cost limits are hit.
- **Test coverage:** at least 80% unit-test coverage on the tool library.
- **CI health:** main branch passing CI at all times; every pull request reviewed.

### Project management

- **Milestone delivery:** all three official milestones delivered on time (design 6 Nov, implementation 30 Nov, final presentation 4 Dec 2026).
- **Mentor review:** a review held at the end of every phase.

## Assumptions

- The plan follows the official DEPI schedule (DEPI column). Planning, literature review and requirements were due on 16 Oct 2026, so they are covered late by this document; confirm with the mentor that this is accepted and that the DEPI column, not the DEPI Industry column, applies.
- The role assignment is a proposal and can be changed by the team.
- KPI targets are initial values to be calibrated after the baseline results in Phase 1.
