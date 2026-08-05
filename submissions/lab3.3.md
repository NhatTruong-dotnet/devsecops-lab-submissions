# Lab 3.3 — Submission

## Task: Gitleaks CI Scan

### Workflow file

```yaml
name: Gitleaks Scan

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read

jobs:
  gitleaks:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Run Gitleaks
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Successful workflow run

- ✅ **Green (Success) workflow run:**
  - https://github.com/NhatTruong-dotnet/devsecops-lab-submissions/actions/runs/30819594953

---

## Job step explanation

### Triggers (`on:`)

This workflow is triggered when code is pushed to the `main` branch, when a pull request targets `main`, and when it is manually started using `workflow_dispatch`. Running Gitleaks on `pull_request` helps prevent secrets from being merged into the main branch, while running it on `push` provides an additional verification that the branch remains free of exposed secrets. The manual trigger is useful for performing an on-demand scan without making a new commit.

### Job: `gitleaks` / `runs-on: ubuntu-latest`

The `gitleaks` job executes the secret scanning process within GitHub Actions. It runs on GitHub-hosted `ubuntu-latest`, which provides a clean, secure, and fully managed Linux environment with broad compatibility for GitHub Actions, reducing maintenance compared to self-hosted runners.

### Step: Checkout repository

The `actions/checkout@v4` action downloads the repository source code to the workflow runner so later steps can access the files. Setting `fetch-depth: 0` retrieves the complete Git history instead of only the latest commit, allowing Gitleaks to scan both the current files and historical commits for accidentally committed secrets.

### Step: Run Gitleaks

The `gitleaks/gitleaks-action@v2` action scans the repository and its Git history for secrets such as API keys, passwords, and access tokens using Gitleaks' built-in detection rules. The workflow provides the automatically generated `GITHUB_TOKEN`, which is a temporary GitHub App installation token created for each workflow run. This token allows the action to authenticate with GitHub APIs when needed, and its permissions are restricted to `contents: read` to follow the principle of least privilege.

---

## One-paragraph reflection

CI scanning remains essential even when every developer uses a Gitleaks pre-commit hook because local hooks can be skipped, disabled, or incorrectly configured. Running Gitleaks in the CI pipeline provides a centralized and consistent security control that every change must pass before being merged, ensuring the repository is scanned regardless of each developer's local environment.
