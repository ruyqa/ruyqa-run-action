# RuyQA GitHub Action

Upload your test results to RuyQA and get a verdict on every pull request: did the change work, and is there test evidence for every file it touched?

## Usage

```yaml
name: tests
on: [pull_request]

permissions:
  contents: read
  pull-requests: write        # lets the action post the verdict comment

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0       # RuyQA needs history to see which files the pull request changed

      # ...run your tests and write a JUnit file...

      - uses: ruyqa/ruyqa-run-action@v1
        with:
          file: test-results/junit.xml
          github-token: ${{ secrets.GITHUB_TOKEN }}
        env:
          RUYQA_API_KEY: ${{ secrets.RUYQA_API_KEY }}
```

Each pull request gets one comment, updated on every push, saying **Verified**, **Needs attention**, **Unverified** or **Pending**, with the failing tests, the changed files that no test covers, and existing tests that look related but are not linked yet.

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `file` | Yes | | Path to test results file |
| `format` | No | `junit` | `junit`, `playwright` or `json` |
| `title` | No | auto | Test run title |
| `project` | No | from `ruyqa.yml` | Project slug (older setups; the API key already identifies the project) |
| `browser` | No | | Browser the tests ran in (`chromium`, `firefox`, `webkit`), so the Cross Browser agent can compare runs |
| `analyze` | No | `false` | Run AI failure analysis |
| `api-key` | No | env var | API key override (prefer the `RUYQA_API_KEY` secret) |
| `verify` | No | `true` | Judge the change and post a verdict. Set `false` to only upload results |
| `fail-on` | No | `needs-attention` | `needs-attention`, `unverified` (also fail when nothing proves the change) or `never` |
| `github-token` | No | | Token for the PR comment, usually `secrets.GITHUB_TOKEN` |
| `api-url` | No | `https://api.ruyqa.com/api/v1` | Point at a self-hosted RuyQA |

## Outputs

`run-id`, `url`, `passed`, `failed`, `total`, `verdict`, `verification-id`.

## How the verdict is decided

No model opinion is involved. A change is **Verified** when results were uploaded, nothing failed, every test linked to the changed code ran, and every changed code file has a test linked to it (or coverage data showing it ran). Documentation, images and lockfiles are ignored.

Tests are linked to code with `covers:<path>` tags. On first connect, RuyQA reads your file list and suggests links by matching test names to file names: open **Verify** in the app, or ask your coding agent to run `suggest_coverage_links` (see `@ruyqa/mcp`).

## Authentication

Copy your project key from the **Connect a source** page in RuyQA, and add it as the repository secret `RUYQA_API_KEY`.
