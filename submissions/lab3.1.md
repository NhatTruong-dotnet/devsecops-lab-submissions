# Lab 3.1 — Submission

## Task 1: SSH Commit Signing

### Local Configuration

- **`git config --global gpg.format`** → `ssh`
- **`git config --global user.signingkey`** → `/home/daniel/.ssh/id_ed25519.pub`
- **`git config --global commit.gpgsign`** → `true`

### Local Verification

Output of `git log --show-signature -1`:

```text
Good "git" signature for builehoangnhattruong@gmail.com with ED25519 key SHA256:S14vKSdwJotbgw2rpwk/oy6D2qfXkRMMsR826EAuSXU
Author: Truong Bui <builehoangnhattruong@gmail.com>
Date: Fri Jul 31 23:18:10 2026 +1000

    test: first signed commit
```

### GitHub Verification

- **Direct link to the most recent signed commit:** *(add your GitHub commit URL here)*
- **Verified badge screenshot:** *(insert screenshot or link to image)*

### Reflection

SSH commit signing helps prevent **Repudiation** attacks by proving that a commit was created by the legitimate owner of the signing key. Without signed commits, an attacker could forge commits using another developer's identity, making it difficult to determine who actually introduced a change. The GitHub **Verified** badge makes signed commits visible and allows team members to quickly identify authentic contributions.

---

## Task 2: Pre-commit + gitleaks

### `.pre-commit-config.yaml`

```yaml
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.30.1
    hooks:
      - id: gitleaks

  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v5.0.0
    hooks:
      - id: detect-private-key
      - id: check-added-large-files
```

### `pre-commit install` Output

```text
pre-commit installed at .git/hooks/pre-commit
```

### Blocked Commit Output

```text
Detect hardcoded secrets.................................................Failed
- hook id: gitleaks
- exit code: 1

○
│╲
│ ○
○ ░
░    gitleaks

Finding:     GH_PAT=REDACTED
Secret:      REDACTED
RuleID:      github-pat
Entropy:     4.143943
File:        submissions/leak-attempt.txt
Line:        2
Fingerprint: submissions/leak-attempt.txt:github-pat:2

11:44PM INF 0 commits scanned.
11:44PM INF scanned ~101 bytes (101 bytes) in 170ms
11:44PM WRN leaks found: 1
```

---

## PR Checklist

- [x] Task 1 — SSH signing configured + Verified badge on commit
- [x] Task 2 — `.pre-commit-config.yaml` created and gitleaks successfully blocked a commit containing a secret
