# devportal-docs-actions

Reusable GitHub Actions for the [`devportal-docs`](https://github.com/pagopa/devportal-docs)
documentation update pipeline.

## Actions

- **[`trigger-update-doc`](./trigger-update-doc/README.md)**: sends the product name
  and PR title, body and diff to
  [`devportal-docs-updater`](https://github.com/pagopa/devportal-docs-updater)
  on a branch dedicated to that PR.
