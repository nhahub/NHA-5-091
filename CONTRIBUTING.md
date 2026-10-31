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

Rebase on `main` daily. Merge with **rebase and merge**, then delete the branch.

## Commits

Conventional Commits with the area as scope:

```
feat(validity): add G2 duplicate check
fix(sandbox): block network access
docs(plan): finalize analysis plan
test(benchmark): add injector sanity test
```

Commit messages become history, so keep every commit meaningful.

## Pull requests

- One issue per PR, under ~400 changed lines.
- Open a draft PR early.
- Fill in the checklist: tests, no secrets, no data files, docs updated.

## Merging

- Default: **rebase and merge**; PR titles are plain English and each commit
  stays in history.
- If you squash, the PR title must follow the commit format:
  `feat(validity): add G2 duplicate gate`.
- Merge commits are disabled. Remove `wip` and fixup commits before merging,
  then delete the branch.
- Repository settings: rebase as default, squash allowed, merge commits off.

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

## Data and evaluation runs

Never commit datasets, lockbox files, oracle files, API keys, or raw run
outputs. Commit manifests and hashes only.

Record the commit hash in the run config and every ledger entry. Do not change
prompts, gates, thresholds, splits or sandbox during the main runs.
