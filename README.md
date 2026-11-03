# zensical-action

A reusable GitHub Actions workflow that spell-checks and builds documentation
with [Zensical](https://zensical.org/), then deploys it to GitHub Pages when the
default branch is pushed.

The workflow contains two jobs:

1. **Build** — checks out the repository, runs the Markdown spell checker,
   installs Zensical, and builds the site in strict mode.
2. **Deploy** — publishes the generated site to GitHub Pages. This job only runs
   for pushes to the repository's default branch.

It is a reusable workflow rather than a composite action because deployment
needs its own job-level environment and permissions.

Before spell-checking, the build job verifies that every word in the wordlist
appears in at least one tracked Markdown file. Unused entries are reported with
their wordlist line number and the build fails, keeping the wordlist minimal.
Matching is whole-word and case-insensitive.

## Usage

Create a workflow in your repository, for example
`.github/workflows/docs.yaml`:

```yaml
name: Documentation

on:
  push:
  pull_request:

jobs:
  docs:
    uses: Drazape/zensical-action/.github/workflows/docs.yaml@main
    permissions:
      contents: read
      pages: write
      id-token: write
```

Pin `@main` to a tag or commit SHA if you want a stable version.

The calling job must grant all three permissions. A reusable workflow cannot
request more permissions than its caller provides, and GitHub validates this
for every job in the called workflow, even when the deploy job is skipped.

## Inputs

| Input | Default | Description |
| --- | --- | --- |
| `config-file` | `docs/zensical.toml` | Path to the Zensical configuration file. |
| `site-dir` | `docs/site` | Path to the generated site directory. |
| `python-version` | `3.x` | Python version used to install Zensical. |
| `wordlist-path` | `docs/spellcheck/wordlist.txt` | Path to the spell-check wordlist. |

Example with the defaults overridden:

```yaml
jobs:
  docs:
    uses: Drazape/zensical-action/.github/workflows/docs.yaml@main
    permissions:
      contents: read
      pages: write
      id-token: write
    with:
      config-file: zensical.toml
      site-dir: site
      wordlist-path: spellcheck/wordlist.txt
```

## Repository setup

The workflow expects the calling repository to provide:

- A Zensical configuration file, by default `docs/zensical.toml`.
- A spell-check configuration file at `docs/spellcheck/config.yaml` for
  [`rojopolis/spellcheck-github-actions`](https://github.com/rojopolis/spellcheck-github-actions).
- A wordlist at `docs/spellcheck/wordlist.txt`, or the path passed as
  `wordlist-path`. This must match the wordlist referenced by the spell-check
  configuration. Blank lines and lines starting with `#` are ignored.
- GitHub Pages enabled with **GitHub Actions** as the source.

The spell-check output file `spellcheck-output.txt` is uploaded as a workflow
artifact, even when the spell check fails, so failures can be inspected from the
workflow run. Unused wordlist entries are also emitted as GitHub annotations on
the wordlist file.
