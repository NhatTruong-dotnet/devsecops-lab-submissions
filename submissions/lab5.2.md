# Lab 5.2 — Submission

## Task 1: DAST with OWASP ZAP

### Baseline (unauthenticated) scan

- Duration: ~1–2 minutes
- Total alerts: **10**
- Unique URLs with findings: **18**

| Severity | Count |
|----------|------:|
| High | 0 |
| Medium | 2 |
| Low | 5 |
| Informational | 3 |


### Authenticated full scan

- Duration: **~8 minutes 23 seconds**
- Total alerts: **13**
- Unique URLs with findings: **23**

| Severity | Count |
|----------|------:|
| High | 2 |
| Medium | 4 |
| Low | 3 |
| Informational | 4 |


### The "10–20× more" claim : authenticated DAST finds 10–20× more issues than unauth

- Ratio (auth alerts / baseline alerts): **1.3×**
- The 10–20× increase was not observed in this scan.

1. **SQL Injection — High**
   - Required authenticated access and was unreachable to the unauthenticated scan.

2. **Vulnerable JS Library — High**
   - Was identified only after the authenticated scan reached additional application functionality.
