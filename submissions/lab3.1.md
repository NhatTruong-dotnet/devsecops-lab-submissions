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
