## Bonus: History Rewrite

### Before
Output of `git log -p | grep -c 'ghp_'`: **2**

### After
Output of `git log -p | grep -c 'ghp_'`: **0**  
Output of `git log -p | grep -c 'REDACTED'`: **2**

### The two-step pattern in real life
1. `git filter-repo --replace-text replacements.txt` — Rewrite the repository history to remove the exposed secret from every commit.
2. Rotate or revoke the compromised secret immediately. Rewriting Git history removes the secret from the repository, but it does not invalidate credentials that may have already been exposed.

### Two real-world gotchas you discovered
1. `git filter-repo` rewrites the entire commit history, so every commit receives a new commit hash (SHA). If the repository has already been pushed to a remote, a force push is required, and anyone working on the repository must re-sync with the rewritten history.
2. I was surprised that a single `git filter-repo --replace-text` command replaced every occurrence of the secret throughout the repository's history. It is much faster and less error-prone than manually editing commits with interactive rebase.
