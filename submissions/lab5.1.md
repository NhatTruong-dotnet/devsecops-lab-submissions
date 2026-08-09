# Lab 5.1 — Submission

## Task 1: SAST with Semgrep

### Semgrep severity breakdown
| Severity | Count |
|----------|------:|
| ERROR | 13 |
| WARNING | 14 |
| INFO | 0 |
| **Total** | 27 |

### Top 10 rules by frequency
| Rule ID | Count | OWASP category |
|---|---:|---|
| `javascript.sequelize.security.audit.sequelize-injection-express.express-sequelize-injection` | 6 | A03 — Injection |
| `yaml.github-actions.security.run-shell-injection.run-shell-injection` | 5 | A03 — Injection |
| `javascript.express.security.audit.express-check-directory-listing.express-check-directory-listing` | 4 | A01 — Broken Access Control |
| `javascript.express.security.audit.express-res-sendfile.express-res-sendfile` | 4 | A01 — Broken Access Control |
| `yaml.github-actions.security.github-actions-mutable-action-tag.github-actions-mutable-action-tag` | 4 | A08 — Software and Data Integrity Failures |
| `javascript.express.security.audit.express-open-redirect.express-open-redirect` | 1 | A01 — Broken Access Control |
| `javascript.jsonwebtoken.security.jwt-hardcode.hardcoded-jwt-secret` | 1 | A02 — Cryptographic Failures |
| `javascript.lang.security.audit.code-string-concat.code-string-concat` | 1 | A03 — Injection |
| `yaml.github-actions.security.gha-curl-pipe-shell.gha-curl-pipe-shell` | 1 | A08 — Software and Data Integrity Failures |


### Triage shortcut (Lecture 5 slide 8)
I would fix the **Sequelize SQL injection** findings first. Although the rule produces 6 findings, the most important ones are in the user-facing application routes such as `routes/login.ts` and `routes/search.ts`, where user-controlled input is directly incorporated into SQL queries. These vulnerabilities can be exploited through normal application requests, so replacing the string interpolation with parameterized queries would directly reduce the application's attack surface.

The findings under `data/static/codefixes/` should be treated separately because they are intentionally vulnerable Juice Shop challenge material rather than normal production application code.

### False-positive sample
**File:** `labs/lab5/semgrep/juice-shop/.github/workflows/update-challenges-ebook.yml`

**Rule:** `yaml.github-actions.security.run-shell-injection.run-shell-injection`

Semgrep flags the direct use of ${{ github.ref_name }} in the wget command as a potential shell injection risk. However, under the normal automated workflow, this value is limited to pushes to the trusted master and develop branches, and the job only runs in the upstream juice-shop/juice-shop repository
<img width="1024" height="532" alt="image" src="https://github.com/user-attachments/assets/5baa6013-0cc3-482b-834d-a136e70adce2" />
