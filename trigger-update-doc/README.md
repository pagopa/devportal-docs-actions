# Trigger Update Doc

Saves the product name and PR title, body and diff to
`working/<repository>-<pr-number>.diff` on branch `<repository>-<pr-number>`
in `pagopa/devportal-docs-updater`.

```yaml
on:
  pull_request:
    types: [closed]

jobs:
  update-doc:
    if: github.event.pull_request.merged == true
    runs-on: ubuntu-latest
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

Replace `<ref>` with a trusted tag or commit. The secret must have contents
read/write access to `pagopa/devportal-docs-updater`; secrets are not normally
available to workflows triggered by pull requests from forks.
