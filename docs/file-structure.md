# File Structure

The source tree is created in week 1. This file is the only place the full tree
is maintained.

```
.
├─ README.md
├─ CONTRIBUTING.md
├─ .gitignore
├─ pyproject.toml
├─ uv.lock
├─ .python-version
├─ .env.example
├─ configs/
│  ├─ llm.yaml                 # pinned backbone, temperature
│  ├─ budgets.yaml             # iterations, wall time, tokens
│  ├─ gate-thresholds.yaml     # fixed after the pilot
│  ├─ contracts/               # example goal contracts
│  └─ matrix/                  # run matrix definitions
├─ data/
│  ├─ README.md
│  └─ manifests/{dev,eval}/    # dataset id, hash, schema, fault spec
├─ docker/
│  ├─ dev.Dockerfile
│  ├─ sandbox.Dockerfile
│  └─ compose.yaml
├─ docs/
│  ├─ project.md
│  ├─ requirements.md
│  ├─ simplified-requirements.md
│  ├─ arch.md
│  ├─ file-structure.md
│  └─ archive/
│     └─ project-v1.md         # superseded v1.0, reference only
├─ src/amlab/
│  ├─ contracts/               # GoalContract, ExperimentSpec, GateResult, Verdict, LedgerEntry
│  ├─ ledger/
│  ├─ orchestration/           # LangGraph graph, state, budgets
│  ├─ agents/
│  │  ├─ experimenter.py
│  │  ├─ critic.py
│  │  └─ prompts/
│  ├─ audit/
│  ├─ splits/
│  ├─ sandbox/
│  ├─ evaluation/
│  ├─ gates/                   # g0 ... g8, gatekeeper
│  ├─ verification/
│  ├─ reporting/
│  ├─ benchmark/               # injectors, oracle
│  ├─ baselines/               # C0, C1, C0g
│  ├─ runner/
│  └─ cli/
├─ apps/dashboard/             # post-evaluation, read-only over the ledger
├─ tests/
│  ├─ unit/
│  ├─ integration/
│  ├─ security/                # scripted tampering attack suite
│  ├─ benchmark/               # injector sanity and manifest tests
│  └─ fixtures/
├─ scripts/
└─ .github/
   ├─ workflows/
   ├─ PULL_REQUEST_TEMPLATE.md
   └─ ISSUE_TEMPLATE/
```

## Rules

- One package per component. Shared schemas live only in `contracts/`.
- `apps/dashboard/` stays outside `src/amlab/` so it cannot affect the fixed
  evaluation version.
- `configs/gate-thresholds.yaml` is fixed before the main evaluation.
- Tests mirror the source layout.

## Ownership

| Workstream | Paths |
| --- | --- |
| A. Agents | `src/amlab/agents/`, `src/amlab/orchestration/`, `apps/dashboard/` |
| B. Sandbox and evaluation | `src/amlab/sandbox/`, `src/amlab/evaluation/`, `tests/security/` |
| C. Validity | `src/amlab/splits/`, `src/amlab/audit/`, `src/amlab/gates/`, `src/amlab/verification/` |
| D. Benchmark | `src/amlab/benchmark/`, `data/manifests/` |
| E. Baselines and evaluation | `src/amlab/baselines/`, `src/amlab/runner/`, `src/amlab/reporting/` |
| Shared | `src/amlab/contracts/`, `docs/`, `configs/`, `.github/` |

## Data policy

- Never commit raw datasets, lockbox files, oracle files, API keys, or raw run
  outputs.
- Commit manifests, hashes and schemas only.
- Lockbox and oracle live outside the repository. Only benchmark tooling has
  read access to the oracle.
- Run artifacts go to an ignored `runs/` directory, not to git.

## Naming

- Files and directories: lowercase with hyphens for docs, snake_case for Python.
- Docs stay ASCII in file names.
