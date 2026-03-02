# pr-to-ready

A GitHub composite action that collapses a PR's development branch into a single synthetic commit and pushes it to a `ready/<branch-name>` branch, handing off to the [`ready-to-trunk`](../ready-to-trunk/) action for the final merge.

## Overview

This action is designed for teams that want to bring Pull Requests into the [Takt](https://www.lakruzz.com/stories/takt/) workflow. It bridges the gap between GitHub's PR-centric model and the Takt single-piece-flow: on PR approval the entire commit history on the development branch is squashed into one clean, fast-forward-able commit and placed on a `ready/*` branch, exactly as `gh tt deliver` would do for an issue branch.

The workflow is:

1. A developer opens a PR from a feature branch targeting `main`.
2. A reviewer approves the PR (or a maintainer triggers a `workflow_dispatch`).
3. `pr-to-ready` kicks in, squashes the PR's commits into a single synthetic commit parented on `origin/main`, and pushes it to `ready/<head-branch>`.
4. The `ready-to-trunk` workflow picks up the `ready/*` push and fast-forward merges it to `main`, then performs cleanup.

## Usage

Add the workflow below to your repository (`.github/workflows/pr-to-ready.yml`) and make sure the `READY_PUSHER` secret is set to a Personal Access Token (PAT) with `contents: write` permission:

```yaml
name: PR to Ready

on:
  workflow_dispatch:
    inputs:
      pr_number:
        description: Pull request number to deliver
        required: true
        type: number
  pull_request_review:
    types:
      - submitted

concurrency:
  group: pr-to-ready-${{ github.event.pull_request.number || inputs.pr_number }}
  cancel-in-progress: true

jobs:
  deliver-pr:
    name: Deliver PR to ready/*
    runs-on: ubuntu-latest
    env:
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    permissions:
      contents: write
      pull-requests: write

    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
          token: ${{ secrets.READY_PUSHER }}

      - uses: devx-cafe/takt-actions/pr-to-ready@v1
        # with: # Uncomment to override defaults
        # user_name: "Ready Pusher Bot"
        # user_email: "ready-pusher@<your-org>.github.com"
```

> **Note:** `${{ github.token }}` will **not** work for the `checkout` token because pushes made with `GITHUB_TOKEN` do not trigger subsequent workflows. Use a PAT stored as a repository secret (e.g. `READY_PUSHER`).

## Inputs

| Input        | Required | Default                                                  | Description                     |
| ------------ | -------- | -------------------------------------------------------- | ------------------------------- |
| `user_name`  |          | `Ready Pusher Bot`                                       | Git author name for the commit  |
| `user_email` |          | `ready-pusher@${{ github.repository_owner }}.github.com` | Git author email for the commit |

## How it works

1. **Gate check** – The action verifies that the event is either a `workflow_dispatch` (requires `pr_number` input) or a `pull_request_review` event where the review state is `approved`, the PR targets `main`, and the PR is from the same repository (not a fork).
2. **PR context** – Resolves the PR number, title, head ref, and head SHA from the event payload or via `gh pr view`.
3. **Synthetic commit** – Creates a new commit whose tree matches the PR's head tree but whose parent is `origin/main`, so it is always fast-forward-able. The commit message is the PR title (with any `[WIP]` prefix stripped), followed by `- resolves #<PR_NUMBER>`, and a log of the original commits for traceability.
4. **Push** – Force-pushes the synthetic commit to `ready/<head-branch>`.

## Guard conditions

The action is a no-op (exits with an annotated failure message) when:

- The triggering event is not `workflow_dispatch` or `pull_request_review`.
- The review state is not `approved`.
- The PR does not target `main`.
- The PR is not open.
- The PR originates from a fork.
