# Lab 7.1 — Submission

## Task 1: Trivy Image + Config Scan

### Image scan severity breakdown

| Severity | Total | With fix available |
|----------|------:|------------------:|
| Critical | 10 | 8 |
| High | 61 | 60 |
| **Total** | **71** | **68** |

Trivy found **71 HIGH or CRITICAL vulnerabilities** in the image. Of these, **68 have a fix available**, while 3 do not currently have a fixed version listed by Trivy.

### Top 10 CVEs with fixes

| CVE | Severity | Package | Installed | Fix |
|-----|----------|---------|-----------|-----|
| CVE-2023-46233 | CRITICAL | crypto-js | 3.3.0 | 4.2.0 |
| CVE-2026-71851 | CRITICAL | crypto-js | 3.3.0 | 4.0.0 |
| CVE-2015-9235 | CRITICAL | jsonwebtoken | 0.1.0 | 4.2.2 |
| CVE-2015-9235 | CRITICAL | jsonwebtoken | 0.4.0 | 4.2.2 |
| CVE-2019-10744 | CRITICAL | lodash | 2.4.2 | 4.17.12 |
| CVE-2026-59873 | CRITICAL | tar | 4.4.19 | 7.5.19 |
| CVE-2026-59873 | CRITICAL | tar | 6.2.1 | 7.5.19 |
| CVE-2026-59873 | CRITICAL | tar | 7.5.15 | 7.5.19 |
| CVE-2026-14456 | HIGH | libssl3t64 | 3.5.5-1~deb13u2 | 3.5.7-1~deb13u2 |
| CVE-2026-45447 | HIGH | libssl3t64 | 3.5.5-1~deb13u2 | 3.5.6-1~deb13u2 |

### Compared to Lab 4's Grype scan

**CVE found by both tools: CVE-2026-14456**

Both Trivy and Grype found `CVE-2026-14456` in the `libssl3t64` package, version `3.5.5-1~deb13u2`. They also reported the same fixed version, `3.5.7-1~deb13u2`. This shows that both tools were able to correctly identify the same vulnerability in the Debian package.

**CVE found by Grype but not Trivy: CVE-2026-5450**

Grype found `CVE-2026-5450` in `libc6` version `2.41-12+deb13u2`, but Trivy did not report it in its results. One possible reason is that the two tools use different vulnerability databases and package-matching methods, so an advisory available to Grype may not be available or matched by Trivy. This shows why using more than one vulnerability scanner can give better coverage.
