# deploy-to-repo

A GitHub composite action that downloads a build artifact and force-pushes its contents to a target repository branch.

## Inputs

| Input             | Required | Default      | Description                                                                                                                                   |
| ----------------- | -------- | ------------ | --------------------------------------------------------------------------------------------------------------------------------------------- |
| token             | no       | github.token | Token used for the push to the target repo. Defaults to `GITHUB_TOKEN`, but a **PAT** is required if follow-on workflows should be triggered. |
| target_repository | yes      | n/a          | Target repository in owner/repo format.                                                                                                       |
| artifact_name     | yes      | n/a          | Name of the artifact to download.                                                                                                             |
| artifact_path     | no       | \_site       | Local path where the artifact is downloaded.                                                                                                  |
| target_branch     | no       | main         | Branch to push deployment contents to.                                                                                                        |
| user_name         | yes      | n/a          | Git user name for deployment commit.                                                                                                          |
| user_email        | yes      | n/a          | Git user email for deployment commit.                                                                                                         |
| commit_message    | no       | ""           | Commit message. Empty falls back to an auto-generated message.                                                                                |
| cname             | no       | ""           | Optional CNAME file content.                                                                                                                  |
| status_context    | no       | deploy:repo  | Optional status context for gh set-status.                                                                                                    |

## Usage

name: Deploy To Repo

on:
workflow_dispatch:

jobs:
deploy:
runs-on: ubuntu-latest
permissions:
contents: write
statuses: write

    steps:
      - name: Deploy artifact to target repo
        uses: devx-cafe/takt-actions/deploy-to-repo@v1
        with:
          target_repository: my-org/my-site-repo
          artifact_name: site-${{ github.sha }}
          target_branch: main
          user_name: Ready Pusher Bot
          user_email: ready-pusher@my-org.github.com
          cname: www.example.com

## Notes

- This action uses GH_TOKEN from github.token.
- For cross-repository deploys, ensure the token has write access to the target repository.
