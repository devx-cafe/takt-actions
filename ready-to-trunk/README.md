# ready-to-trunk

A GitHub composite action that fast-forward merges a `ready/*` branch to trunk, then performs context-aware PR/issue closure and branch cleanup.

## Overview

This is a single-action repository, adapted from the [`ready` action](https://github.com/lakruzz/workflows/tree/main/actions/ready) in [lakruzz/workflows](https://github.com/lakruzz/workflows).

The workflow is:

1. Push commits to a `ready/<branch-name>` branch.
2. The `ready-to-trunk` workflow triggers and fast-forward merges the commit to `main` (or a configured target branch).
3. Optionally closes the associated PR, linked issues, and deletes the development and ready branches.

## Usage

Reference this action in your own workflow:

```yaml
- uses: devx-cafe/ready-to-trunk@main
  with:
    user_name: "Ready Pusher" # required
    user_email: "ready-pusher@example.com" # required
    token: ${{ secrets.READY_PUSHER }} # required – PAT with contents/issues/pull-requests write
    # target_branch: main      # default: main
    # delete_dev_branch: true  # default: true
    # delete_ready_branch: true # default: true
    # close_pr: true           # default: true
    # close_issue: true        # default: true
```

> **Note:** `${{ github.token }}` will **not** work for `token` because pushes made with that token do not trigger subsequent workflows. Use a personal access token (PAT) stored as a repository secret (e.g. `READY_PUSHER`).

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

This repository ships a ready-to-use workflow (`.github/workflows/ready-to-trunk.yml`) that triggers on any `ready/*` push. To use it in your own repository, copy the workflow file and set the `READY_PUSHER` secret.
