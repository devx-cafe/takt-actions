# takt-actions

GitHub composite actions that power the [Takt](https://www.lakruzz.com/stories/takt/) lean-delivery workflow.

## What is Takt?

**Takt** is a lean software-development workflow for GitHub that enables a true one-piece flow: one issue → one commit → fast-forward merge to `main`. It eliminates the wait-states and task-switching caused by traditional Pull Request peer-review gates by replacing them with automated quality checks and clear ownership.

The full workflow is implemented across two GitHub repositories:

| Repository                                              | Role                                                                                                                                     |
| ------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| [`devx-cafe/gh-tt`](https://github.com/devx-cafe/gh-tt) | A `gh` CLI extension that gives developers three commands — `workon`, `wrapup`, `deliver` — to drive the day-to-day flow from a terminal |
| **`devx-cafe/takt-actions`** (this repo)                | GitHub composite actions consumed by the CI/CD workflows that run when code is delivered                                                 |

## Actions in this repository

### [`pr-to-ready`](pr-to-ready/)

Bridges the gap between GitHub Pull Requests and the Takt flow.
On PR approval (or manual dispatch), it collapses the PR's commit history into a single synthetic commit parented on `origin/main` and pushes it to a `ready/<branch>` branch — identical to what `gh tt deliver` does for issue branches.

### [`ready-to-trunk`](ready-to-trunk/)

Triggered by a push to any `ready/*` branch.
Fast-forward merges the commit to the target branch (`main` by default), then performs context-aware cleanup: closes the PR, closes linked issues, and deletes the `ready/*` and development branches.

## How the pieces fit together

```text
Developer workspace          GitHub Actions CI
───────────────────          ─────────────────────────────────────────────
gh tt workon -i <N>   →  creates / checks-out issue branch
gh tt wrapup          →  push  →  wrapup.yml  (runs checks on dev branch)
gh tt deliver         →  push  →  ready.yml
                                   ├─ trunk-worthy job  (quality gate)
                                   └─ merge-to-trunk job
                                        └─ uses: takt-actions/ready-to-trunk

                           OR, for teams that use Pull Requests:

Open PR / approval    →  pr-to-ready.yml
                           └─ uses: takt-actions/pr-to-ready  →  ready/*
                                                                    └─ (same ready.yml flow above)
```

## Quick start

Copy the workflow files from `.github/workflows/` into your own repository and add a `READY_PUSHER` secret (a PAT with `contents`, `pull-requests`, and `issues` write permissions):

- **`ready.yml`** — triggers on `ready/**` pushes; runs quality checks then calls `ready-to-trunk`.
- **`pr-to-ready.yml`** — triggers on PR approval; calls `pr-to-ready` to enter the Takt flow.

See each action's own README for full input documentation:

- [pr-to-ready/README.md](pr-to-ready/README.md)
- [ready-to-trunk/README.md](ready-to-trunk/README.md)

## Related resources

- [Takt: The Lean Workflow — blog series](https://www.lakruzz.com/stories/takt/)
- [`devx-cafe/gh-tt` — the CLI companion](https://github.com/devx-cafe/gh-tt)
