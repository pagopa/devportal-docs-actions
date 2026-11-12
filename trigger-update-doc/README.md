# Trigger Update Doc

Saves the product name and PR title, body and diff to
`working/<repository>-<pr-number>.diff` on branch `<repository>-<pr-number>`
in `pagopa/devportal-docs-updater`.

The action uses two separate credentials:

- The calling workflow's `GITHUB_TOKEN` reads the source PR. Grant the workflow
  read access to contents, pull requests and issues.
- The `updater-gh-token` input authenticates Git when cloning and pushing to
  `pagopa/devportal-docs-updater`. Provide a GitHub App installation token or a
  fine-grained personal access token with contents read and write access to that
  repository. Store a personal access token as an Actions secret. The action
  passes this token to `actions/checkout`, which configures Git credentials for
  the checkout and removes them after the job.

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
      - uses: pagopa/devportal-docs-actions/trigger-update-doc@<ref>
        with:
          product-name: it-wallet
          pr-number: ${{ github.event.pull_request.number }}
          updater-gh-token: ${{ secrets.DEVPORTAL_DOCS_UPDATER_TOKEN }}
```

Replace `<ref>` with a trusted tag or commit containing the action. Secrets are
not normally available to workflows triggered by pull requests from forks; use a
trusted workflow in which the destination token is available.

Reruns reuse the existing branch for the source PR and add a commit only if the
file has changed. If the branch was deleted, the action recreates it. The job's
`concurrency` configuration prevents simultaneous updates for the same PR.
An unchanged rerun does not push and therefore does not retrigger a downstream
workflow listening for pushes; rerun that workflow separately when needed.
