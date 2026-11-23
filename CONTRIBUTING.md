# Contributing

## Layout

| Path | Contents |
|------|----------|
| `.github/workflows/<name>.yaml` | Reusable workflows (`on: workflow_call`) |
| `.github/workflows/test-<name>.yaml` | Self-test, when this repository does not already use it |
| `.github/workflows/ci.yaml`, `release.yaml` | This repository's own CI and release |
| `actions/<name>/action.yaml` | Composite actions |
| `actions/<name>/scripts/` | Optional helper scripts (`.sh` or `.py`) |
| `docs/workflows/<name>.md`, `docs/actions/<name>.md` | Inputs, outputs, setup and a usage example |

Reusable workflows must live directly in `.github/workflows/`; GitHub does not support
subdirectories. Do not start their names with `ci`, `release` or `test-`.

## Conventions

- Use the `.yaml` extension.
- Use kebab-case names for workflows, actions, inputs and outputs.
- Every new reusable workflow or action is exercised by a workflow in this repository, and has a
  catalog entry in [README.md](README.md) and a doc in [docs/](docs). When this repository uses
  it on its own events, that caller is the test. Otherwise, add a `test-<name>.yaml`.

## Dependencies and pinning

- Do not use third-party actions. GitHub-owned actions (`actions/*`) are allowed.
- Pin every `uses:` by full commit SHA with a version comment: `@<sha> # vX.Y.Z`, except the
  `./` paths described below.
- Install tools in `run:` steps at a pinned version. Verify downloaded binaries against a
  checksum.
- In reusable workflows and actions, reference this repository's own actions by SHA too, never by
  tag or `./` path. A consumer that pins a reusable workflow by SHA would otherwise get whatever
  the tag points to, and `./` paths resolve against the consumer's repository. Use a SHA already
  on `main` and bump it in a follow-up PR.
- This repository's own workflows, which no consumer calls, reference its reusable workflows and
  actions by `./` path, so they run the code from the same commit.

## Permissions

Every workflow and job declares `permissions:` explicitly. Start from `contents: read` and only
add what is needed.

## Linting

CI runs yamllint, actionlint (which includes shellcheck on `run:` blocks), shellcheck on `*.sh`
files, and ruff. Tool versions are pinned in [ci.yaml](.github/workflows/ci.yaml) and bumped
manually, except shellcheck, which comes from the runner image. To run locally:

```shell
yamllint --strict .
actionlint
git ls-files -z '*.sh' | xargs -0 --no-run-if-empty shellcheck
ruff check .
ruff format --check .
```

## Releasing

1. Merge changes to `main`.
2. In the Actions tab, run the [Release workflow](.github/workflows/release.yaml) on `main` and
   choose `patch`, `minor` or `major`.

The workflow tags the head of `main` with the next `vX.Y.Z` and moves `vX` to the same commit.
Do not create release tags by hand.

Bump the major version for any breaking change to any workflow or action. Choose `major` for the
first release to start at `v1.0.0`.
