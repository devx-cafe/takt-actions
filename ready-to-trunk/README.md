# ready-to-trunk

A GitHub composite action that fast-forward merges a `ready/*` branch to trunk, then performs context-aware PR/issue closure and branch cleanup.

## Overview

This action is part of the [takt-actions](https://github.com/devx-cafe/takt-actions) repository, adapted from the [`ready` action](https://github.com/lakruzz/workflows/tree/main/actions/ready) in [lakruzz/workflows](https://github.com/lakruzz/workflows).

The workflow is:

1. Push commits to a `ready/<branch-name>` branch.
2. The `ready-to-trunk` workflow triggers and fast-forward merges the commit to `main` (or a configured target branch).
3. Optionally closes the associated PR, linked issues, and deletes the development and ready branches.

## Usage

Reference this action in your own workflow:

```yaml
- uses: actions/checkout@v4
  with:
    fetch-depth: 0
    token: ${{ secrets.READY_PUSHER }} # PAT with contents/issues/pull-requests write

- uses: devx-cafe/takt-actions/ready-to-trunk@v1
  # with: # Uncomment to override defaults
  # target_branch: main
  # user_name: "Ready Pusher Bot"
  # user_email: "ready-pusher@<your-org>.github.com"
  # delete_dev_branch: true
  # delete_ready_branch: true
  # close_pr: true
  # close_issue: true
```

> **Note:** `${{ github.token }}` will **not** work for the `checkout` token because pushes made with `GITHUB_TOKEN` do not trigger subsequent workflows. Use a Personal Access Token (PAT) stored as a repository secret (e.g. `READY_PUSHER`).

## Inputs

| Input                 | Required | Default | Description                                                    |
| --------------------- | -------- | ------- | -------------------------------------------------------------- |
| `user_name`           |          | -       | `Ready Pusher Bot`.                                            |
| `user_email`          |          | -       | `ready-pusher@${{ github.repository_owner }}.github.com`       |
| `target_branch`       |          | `main`  | Branch to fast-forward merge into                              |
| `delete_dev_branch`   |          | `true`  | Delete the development branch after delivery                   |
| `delete_ready_branch` |          | `true`  | Delete the `ready/*` branch after delivery                     |
| `close_pr`            |          | `true`  | Close an open PR for the development branch                    |
| `close_issue`         |          | `true`  | Close issue(s) linked to the PR or prefixed in the branch name |

## Built-in workflow

This repository ships a ready-to-use workflow (`.github/workflows/ready.yml`) that triggers on any `ready/*` push. To use it in your own repository, copy the workflow file and set the `READY_PUSHER` secret.
