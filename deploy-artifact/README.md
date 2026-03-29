# deploy-artifact

A GitHub composite action that downloads a pre-built artifact and deploys it to GitHub Pages.

## Inputs

| Input          | Required | Default                | Description                                  |
| -------------- | -------- | ---------------------- | -------------------------------------------- |
| artifact_name  | no       | site-${{ github.sha }} | Name of the artifact to download.            |
| artifact_path  | no       | \_site                 | Local path where the artifact is downloaded. |
| status_context | no       | deploy:pages           | Optional status context for gh set-status.   |

## Outputs

| Output   | Description                            |
| -------- | -------------------------------------- |
| page_url | URL of the deployed GitHub Pages site. |

## Usage

name: Deploy Pages

on:
workflow_dispatch:

jobs:
deploy:
runs-on: ubuntu-latest
permissions:
pages: write
id-token: write
statuses: write

    steps:
      - name: Deploy artifact via Pages
        id: pages
        uses: devx-cafe/takt-actions/deploy-artifact@v1
        with:
          artifact_name: site-${{ github.sha }}

      - name: Show deployed URL
        run: echo "Deployed to ${{ steps.pages.outputs.page_url }}"

## Notes

- This action uses GH_TOKEN from github.token.
- It explicitly sources shared helper scripts in each step and calls helper functions directly.
- Caller workflow must grant pages: write and id-token: write permissions.
