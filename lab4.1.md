# Lab 4.1 — Submission

## Task 1: Syft + Grype on Juice Shop

### SBOM stats

* `juice-shop.cdx.json` component count: **3069**
* `juice-shop.cdx.json` size: **1,832,094 bytes** (~1.83 MB)
* `juice-shop.spdx.json` component count: **908 packages**
* `juice-shop.spdx.json` size: **3,159,981 bytes** (~3.16 MB)

Syft cataloged **908 packages**, **287 executables**, and **2,160 file locations** from the Juice Shop `v20.0.0` image. The CycloneDX SBOM reports 3,069 components because it includes more component-level information than the SPDX package count.

### Grype severity breakdown

| Severity   |   Count |
| ---------- | ------: |
| Critical   |      12 |
| High       |      70 |
| Medium     |      55 |
| Low        |      11 |
| Negligible |       7 |
| **Total**  | **155** |

Grype identified **155 vulnerability matches** in total. Of these, **134 have fixes available**, while **21 are currently not fixed**.

### Top 10 CVEs

> **Note:** Grype's top results include both CVE identifiers and GitHub Security Advisories (GHSA). The identifiers below are reported exactly as returned by Grype rather than being converted or inferred.

| Vulnerability ID    | Severity | Package      | Installed       | Fix             |
| ------------------- | -------- | ------------ | --------------- | --------------- |
| GHSA-35jh-r3h4-6jhm | High     | lodash       | 2.4.2           | 4.17.21         |
| GHSA-c7hr-j4mj-j2w6 | Critical | jsonwebtoken | 0.1.0           | 4.2.2           |
| GHSA-c7hr-j4mj-j2w6 | Critical | jsonwebtoken | 0.4.0           | 4.2.2           |
| GHSA-87vv-r9j6-g5qv | Medium   | moment       | 2.0.0           | 2.11.2          |
| GHSA-jf85-cpcp-j695 | Critical | lodash       | 2.4.2           | 4.17.12         |
| GHSA-8hfj-j24r-96c4 | High     | moment       | 2.0.0           | 2.29.2          |
| CVE-2026-45447      | High     | libssl3t64   | 3.5.5-1~deb13u2 | 3.5.6-1~deb13u2 |
| GHSA-p6mc-m468-83gw | High     | lodash.set   | 4.3.2           | No fix listed   |
| GHSA-446m-mv8f-q348 | High     | moment       | 2.0.0           | 2.19.3          |
| GHSA-fvqr-27wr-82fm | Medium   | lodash       | 2.4.2           | 4.17.5          |

### Fix-available rate

Out of the top 10 vulnerability matches, **9 have a fix available**, giving a fix-available rate of **90%**. This suggests that patching should prioritize the vulnerabilities with both a **fix available** and **High or Critical severity**, following Lecture 4's triage shortcut. In particular, packages such as `jsonwebtoken`, `lodash`, and `moment` should be prioritized because several high-severity findings have known fixed versions, while `lodash.set` should be tracked separately because no fix is currently listed.
