# Contributing

## Branches

`main` is protected and must stay green. No direct pushes.

Create a short-lived branch from `main` (lifetime of a few days):

```
feat/<area>/<slug>    feat/validity/g2-duplicate-check
fix/<area>/<slug>     fix/sandbox/lockbox-mount
docs/<slug>           docs/arch-gates
chore/<slug>          chore/ci-cache
```

Areas: `agents`, `harness`, `validity`, `benchmark`, `eval`, `infra`, `docs`.

Rebase on `main` daily. Squash merge, then delete the branch.

## Commits

Conventional Commits with the area as scope:

```
feat(validity): add G2 duplicate check
fix(sandbox): block network access
docs(plan): freeze analysis plan
test(benchmark): add injector sanity test
```

## Pull requests

- One issue per PR, target under ~400 changed lines.
- Open a draft PR early.
- PR title becomes the squash commit message.
- Fill in the PR checklist: tests, no secrets, no data files, docs updated.

## Reviews

- 1 approval normally.
- 2 approvals for `src/amlab/gates/`, `src/amlab/splits/`,
  `src/amlab/sandbox/`, `src/amlab/evaluation/`, and
  `configs/gate-thresholds.yaml`.
- Separation of duties: whoever builds the fault injectors does not review the
  detectors, and evaluation scripts are reviewed by a second member.
- Everyone reviews at least one PR per week.

## CI

Every PR must pass: lint and format check, type check, unit tests, Docker
sandbox smoke test, and benchmark manifest sanity. Heavy experiment runs are
never part of CI.

Target: at least 80% unit-test coverage on the harness and gates.

## Tags and freeze

| Tag | Date | Meaning |
| --- | --- | --- |
| `v0.1-design` | 6 Nov 2026 | Design document, CI, baselines |
| `v0.2-slice` | 13 Nov 2026 | Guarded C3 run on one clean and one F1 instance |
| `v1.0-eval` | 20 Nov 2026 | **System freeze** |
| `v1.1-submission` | 30 Nov 2026 | Implementation submitted |
| `v1.2-final` | 4 Dec 2026 | Final report, presentation, demo |

After `v1.0-eval`, only `fix/freeze/*` branches touching evaluated code are
allowed. They must be behavior-preserving, need 2 approvals, and are recorded in
`docs/project.md` or the release notes.

## Data

Never commit datasets, lockbox files, oracle files, API keys, or raw run
outputs. Commit manifests and hashes only.
