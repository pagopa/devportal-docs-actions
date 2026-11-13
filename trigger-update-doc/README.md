# Trigger Update Doc

Saves the product name and PR title, body and diff to
`working/<repository>-<pr-number>.diff` on branch `<repository>-<pr-number>`
in `pagopa/devportal-docs-updater`.

The action uses two separate credentials:

- The calling workflow's `GITHUB_TOKEN` reads the source PR. Grant the workflow
  read access to contents, pull requests and issues.
- The `updater-gh-token` input authenticates Git when cloning and pushing to
  `pagopa/devportal-docs-updater`. It must be the installation token of the
  `devportal-docs-updater-internal` GitHub App, generated in the calling
  workflow by the `app-token` step shown below. The action passes this token to
  `actions/checkout`, which configures Git credentials for the checkout and
  removes them after the job.

To generate the token, configure the source repository with:

- the Actions variable `DEVPORTAL_DOCS_UPDATER_INTERNAL_CLIENT_ID`, containing
  the client ID of the `devportal-docs-updater-internal` GitHub App;
- the Actions secret `DEVPORTAL_DOCS_UPDATER_INTERNAL_APP_PRIVATE_KEY`,
  containing the private key of the same GitHub App.

For example, in the source repository:

```yaml
name: Trigger documentation update
on:
  pull_request:
    types: [closed]

jobs:
  update-doc:
    if: github.event.pull_request.merged == true
    runs-on: ubuntu-latest
    concurrency:
      group: doc-update-${{ github.repository }}-${{ github.event.pull_request.number }}
      cancel-in-progress: false
    permissions:
      contents: read
      pull-requests: read
      issues: read
    steps:
      - name: Generate a token for `devportal-docs-updater-internal` GitHub App
        id: app-token
        uses: actions/create-github-app-token@<ref 1>
        with:
          client-id: ${{ vars.DEVPORTAL_DOCS_UPDATER_INTERNAL_CLIENT_ID }}
          private-key: ${{ secrets.DEVPORTAL_DOCS_UPDATER_INTERNAL_APP_PRIVATE_KEY }}
          owner: ${{ github.repository_owner }}
          repositories: devportal-docs-updater
          permission-contents: write
          
      - name: Trigger documentation update
        uses: pagopa/devportal-docs-actions/trigger-update-doc@<ref 2>
        with:
          product-name: idpay
          pr-number: ${{ github.event.pull_request.number }}
          updater-gh-token: ${{ steps.app-token.outputs.token }}
```

Replace `<ref 1>` with a trusted tag or commit of
`actions/create-github-app-token` and `<ref 2>` with a trusted tag or commit
containing this action. Secrets are not normally available to workflows
triggered by pull requests from forks; use a trusted workflow in which
`DEVPORTAL_DOCS_UPDATER_INTERNAL_APP_PRIVATE_KEY` is available.

Reruns reuse the existing branch for the source PR and add a commit only if the
file has changed. If the branch was deleted, the action recreates it. The job's
`concurrency` configuration prevents simultaneous updates for the same PR.
An unchanged rerun does not push and therefore does not retrigger a downstream
workflow listening for pushes; rerun that workflow separately when needed.
