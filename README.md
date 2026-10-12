# security-scan

Workflows that run sha256-verified gitleaks, opengrep, bandit, osv-scanner and trivy config on a
repository. The scan logic, versions and defaults live in the action
[ianchov/security-scan-action](https://github.com/ianchov/security-scan-action).

This repository gives three ways to add it:

| Way | Use it when |
|---|---|
| [Template repository](#template-repository) | You start a new repository. |
| [Reusable workflow](#reusable-workflow) | You add the scan to an existing repository with one job. |
| [Organization workflow template](#organization-workflow-template) | You want it under Actions → New workflow for every repository in your organization. |

To write your own job instead, use the action directly. See its README.

## Template repository

1. Click **Use this template** → **Create a new repository**.
2. Delete the `workflow-templates` directory and `.github/workflows/scan.yml`. Your copy uses the
   reusable workflow from this repository, not its own copy.

## Reusable workflow

Add `.github/workflows/security-scan.yml`:

```yaml
name: Security scan
on:
  pull_request:
  push:
    branches: [main]
  # New advisories appear without code changes.
  schedule:
    - cron: "0 7 * * MON"
  workflow_dispatch:

permissions:
  contents: read

jobs:
  scan:
    uses: ianchov/security-scan/.github/workflows/scan.yml@v1
```

Optional inputs, set under `with:`:

| Input | Default | Meaning |
|---|---|---|
| `runs-on` | `'"ubuntu-24.04"'` | Runner labels as JSON, for example `'["self-hosted","linux"]'`. Fork pull requests always run on `ubuntu-24.04`, because a self-hosted runner keeps state between jobs. |
| `rule-packs` | Python set | opengrep-rules directories, space or newline separated. |
| `exclude` | empty | Extra paths for opengrep, bandit, osv-scanner and trivy to skip, one per line. |

Example for a Django repository on a self-hosted runner:

```yaml
jobs:
  scan:
    uses: ianchov/security-scan/.github/workflows/scan.yml@v1
    with:
      runs-on: '["self-hosted","linux"]'
      exclude: |
        app/staticfiles
```

Put exceptions in the calling repository: `.gitleaksignore`, `osv-scanner.toml` and
`.trivyignore`. The action README describes each.

## Organization workflow template

Copy `workflow-templates/` into the organization's `.github` repository. The workflow then shows
under Actions → New workflow in every repository of the organization. GitHub shows workflow
templates only from an organization's `.github` repository, not from a personal account.
