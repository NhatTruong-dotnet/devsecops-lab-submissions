# Lab 5.3 — Submission

## Task 1: GitHub Actions SAST + DAST Pipeline

### Workflow file

Paste the full content of `.github/workflows/lab5-sast-dast.yml`:

```yaml
name: SAST and DAST

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read

env:
  IMAGE: bkimminich/juice-shop:v20.0.0
  JUICE_SHOP_TAG: v20.0.0
  ZAP_IMAGE: ghcr.io/zaproxy/zaproxy:stable
  RESULTS_DIR: results
  DOCKER_NETWORK: lab5-net

jobs:
  sast:
    name: SAST — Semgrep
    runs-on: ubuntu-latest
    timeout-minutes: 30

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Create results directory
        run: |
          mkdir -p "${RESULTS_DIR}"
          chmod 777 "${RESULTS_DIR}"

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: Install Semgrep
        run: pip install semgrep

      - name: Clone Juice Shop source (pinned to v20.0.0)
        run: |
          git clone --depth 1 \
            https://github.com/juice-shop/juice-shop.git \
            .juice-shop
          cd .juice-shop
          git fetch --depth 1 origin tag "${JUICE_SHOP_TAG}" || true
          git checkout "${JUICE_SHOP_TAG}"

      - name: Run Semgrep (JSON report)
        id: semgrep-json
        continue-on-error: true
        run: |
          semgrep scan \
            --config=p/owasp-top-ten \
            --config=p/javascript \
            --config=p/secrets \
            .juice-shop \
            --json -o "${RESULTS_DIR}/semgrep.json" \
            --severity ERROR --severity WARNING \
            --no-error

      - name: Run Semgrep (human-readable summary)
        id: semgrep-txt
        continue-on-error: true
        run: |
          semgrep scan \
            --config=p/owasp-top-ten \
            --config=p/javascript \
            .juice-shop \
            --severity ERROR \
            --no-error | tee "${RESULTS_DIR}/semgrep.txt"

      - name: Upload SAST reports
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: lab5-sast-reports
          path: |
            ${{ env.RESULTS_DIR }}/semgrep.json
            ${{ env.RESULTS_DIR }}/semgrep.txt
          if-no-files-found: warn
          retention-days: 30

      - name: Fail on Semgrep scan error
        run: |
          if [ ! -s "${RESULTS_DIR}/semgrep.json" ]; then
            echo "Semgrep scan failed: ${RESULTS_DIR}/semgrep.json was not produced."
            exit 1
          fi
          echo "Semgrep JSON report ready."

      - name: SAST security gate
        run: |
          python3 scripts/security_gate.py semgrep \
            "${RESULTS_DIR}/semgrep.json"

  dast:
    name: DAST — OWASP ZAP
    runs-on: ubuntu-latest
    timeout-minutes: 45

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Create results directory
        run: |
          mkdir -p "${RESULTS_DIR}"
          chmod 777 "${RESULTS_DIR}"

      - name: Create Docker network
        run: docker network create "${DOCKER_NETWORK}" 2>/dev/null || true

      - name: Start Juice Shop
        run: |
          docker run -d --name juice-shop --network "${DOCKER_NETWORK}" \
            -p 127.0.0.1:3000:3000 \
            "${IMAGE}"

      - name: Wait for Juice Shop to be ready
        run: |
          for i in $(seq 1 60); do
            if curl -sf -o /dev/null http://127.0.0.1:3000/rest/admin/application-version; then
              echo "Juice Shop ready"
              exit 0
            fi
            sleep 2
          done
          echo "Juice Shop failed to start"
          docker logs juice-shop
          exit 1

      - name: Run ZAP baseline (unauthenticated) scan
        id: zap-baseline
        continue-on-error: true
        run: |
          set +e
          docker run --rm --network "${DOCKER_NETWORK}" \
            --user root \
            -v "${{ github.workspace }}/${RESULTS_DIR}:/zap/wrk" \
            -w /zap/wrk \
            "${ZAP_IMAGE}" \
            zap-baseline.py -t http://juice-shop:3000 \
            -r baseline-report.html -J baseline-report.json

          exit_code=$?
          set -e

          echo "${exit_code}" > "${RESULTS_DIR}/zap-baseline.exit"

          # Exit 2 = issues found (expected for Juice Shop)
          # Exit 1 / other unexpected exit codes = scan error
          if [ "$exit_code" -ne 0 ] && [ "$exit_code" -ne 2 ]; then
            echo "ZAP baseline scan failed with exit code: ${exit_code}"
            exit "$exit_code"
          fi

          if [ ! -s "${RESULTS_DIR}/baseline-report.json" ]; then
            echo "ZAP baseline scan finished (exit ${exit_code}) but baseline-report.json was not created."
            exit 1
          fi

          echo "ZAP baseline scan finished (exit code: ${exit_code})"

      - name: Run ZAP authenticated scan
        id: zap-auth
        continue-on-error: true
        run: |
          docker run --rm --network "${DOCKER_NETWORK}" \
            --user root \
            -e _JAVA_OPTIONS="-Xmx512m" \
            -v "${{ github.workspace }}/scripts:/zap/wrk/scripts" \
            -v "${{ github.workspace }}/${RESULTS_DIR}:/zap/wrk/results" \
            "${ZAP_IMAGE}" \
            zap.sh -cmd \
            -autorun /zap/wrk/scripts/zap/zap-auth.yaml \
            -port 8090

          if [ ! -s "${RESULTS_DIR}/auth-report.json" ]; then
            echo "ZAP authenticated scan finished but auth-report.json was not created."
            exit 1
          fi

          echo "ZAP authenticated scan report ready."

      - name: Compare baseline vs authenticated reports
        id: zap-compare
        continue-on-error: true
        run: |
          bash scripts/zap/compare_zap.sh \
            "${RESULTS_DIR}/baseline-report.json" \
            "${RESULTS_DIR}/auth-report.json" \
            "${RESULTS_DIR}/zap-comparison.txt"

      - name: Stop Juice Shop
        if: always()
        run: |
          docker stop juice-shop 2>/dev/null || true
          docker rm juice-shop 2>/dev/null || true
          docker network rm "${DOCKER_NETWORK}" 2>/dev/null || true

      - name: Upload DAST reports
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: lab5-dast-reports
          path: |
            ${{ env.RESULTS_DIR }}/baseline-report.json
            ${{ env.RESULTS_DIR }}/baseline-report.html
            ${{ env.RESULTS_DIR }}/auth-report.json
            ${{ env.RESULTS_DIR }}/auth-report.html
            ${{ env.RESULTS_DIR }}/zap-comparison.txt
            ${{ env.RESULTS_DIR }}/zap-baseline.exit
          if-no-files-found: warn
          retention-days: 30

      - name: Fail on ZAP scan error
        run: |
          missing=0

          for report in baseline-report.json auth-report.json; do
            if [ ! -s "${RESULTS_DIR}/${report}" ]; then
              echo "Missing or empty report: ${RESULTS_DIR}/${report}"
              missing=1
            fi
          done

          if [ "${missing}" -eq 1 ]; then
            echo "ZAP scan failed (report not produced)."
            exit 1
          fi

          if [ "${{ steps.zap-compare.outcome }}" = "failure" ]; then
            echo "ZAP compare step failed."
            exit 1
          fi

          echo "ZAP reports ready."

      - name: DAST security gate
        run: |
          python3 scripts/security_gate.py zap \
            "${RESULTS_DIR}/baseline-report.json" \
            "${RESULTS_DIR}/auth-report.json"
```

---

## Successful workflow run


[Run History](https://github.com/NhatTruong-dotnet/devsecops-lab-submissions/actions/runs/31986493619)

Expected result:

```text
SAST — Semgrep       ✓ Success
DAST — OWASP ZAP     ✓ Success
```

Both jobs must pass for the CI security pipeline to be considered successful.

---

# Job: `sast` — Step Explanation

## Triggers (`on:`)

```yaml
on:
  pull_request:
    branches: [main]
  push:
    branches: [main]
  workflow_dispatch:
```

The workflow runs when a pull request targets `main`, when code is pushed directly to `main`, or when the workflow is manually triggered. Running SAST and DAST on pull requests provides security feedback before changes are merged, while running them on `main` provides another verification point after changes reach the main branch. Manual execution is useful when security scans need to be rerun without creating a new code change.

---

## Step: Clone Juice Shop source (pinned to v20.0.0)

The workflow clones the Juice Shop source code and checks out the specific `v20.0.0` tag. Pinning the version makes the security scan reproducible because the same application version is tested each time rather than whatever code happens to be on the latest `main` branch.

This matches Lab 5.1 because the same Juice Shop version was used as the target for the local Semgrep analysis. Using the same pinned version makes it easier to compare local and CI security results.

---

## Step: Run Semgrep (JSON report)

Semgrep uses the `p/owasp-top-ten`, `p/javascript`, and `p/secrets` rulesets. These rules cover common web application security issues, JavaScript-specific security patterns, and potentially exposed secrets.

The `--no-error` option prevents security findings from being treated as a Semgrep execution error. This allows the scan to generate its JSON report even when vulnerabilities are found, while the separate `security_gate.py` script is responsible for making the final pass/fail security decision.

---

## Step: Upload SAST reports

The workflow uploads the following files as the `lab5-sast-reports` artifact:

```text
results/semgrep.json
results/semgrep.txt
```

The JSON report is useful for automated processing, while the text report provides a human-readable summary. `if: always()` ensures the upload step can still run even if an earlier step fails, allowing available security evidence to be preserved for troubleshooting.

---

# Job: `dast` — Step Explanation

## Step: Start Juice Shop / Wait for Juice Shop to be ready

Juice Shop must be running before ZAP can perform dynamic security testing. The application is started inside Docker and connected to the same Docker network as the ZAP container, allowing ZAP to access it through the hostname `juice-shop`.

The health-check loop repeatedly sends a request to the Juice Shop endpoint until the application responds successfully. This prevents ZAP from starting before the target application is ready and displays the container logs if the application fails to start.

---

## Step: Run ZAP baseline (unauthenticated) scan

`zap-baseline.py` is the OWASP ZAP baseline scanning script used for automated web application security testing. It scans the running Juice Shop application and generates both JSON and HTML reports.

The workflow accepts exit code `2` because it represents security alerts being found, which is expected when scanning the intentionally vulnerable Juice Shop application. Unexpected exit codes such as `1` are treated as scan failures because they may indicate that ZAP did not successfully complete the scan.

---

## Step: Run ZAP authenticated scan

The authenticated scan uses:

```text
scripts/zap/zap-auth.yaml
```

This configuration tells ZAP how to perform the authenticated security scan and access functionality that may only be available after login.

The workflow sets:

```bash
_JAVA_OPTIONS="-Xmx512m"
```

This limits the Java heap available to ZAP to 512 MB. The setting helps control memory consumption inside the GitHub Actions runner and reduces the risk of the ZAP Java process consuming excessive memory during the scan.

---

## Step: Compare baseline vs authenticated reports

The `compare_zap.sh` script compares:

```text
baseline-report.json
```

with:

```text
auth-report.json
```

and generates:

```text
zap-comparison.txt
```

The comparison identifies differences between the vulnerabilities discovered during unauthenticated and authenticated testing. This automates the analysis performed in Lab 5.2 and makes that comparison repeatable as part of the CI pipeline.

---

## Step: Stop Juice Shop

The Juice Shop container and Docker network are removed after the DAST scan. The cleanup uses `if: always()` so that cleanup still happens even if ZAP, the comparison script, or another previous step fails.

This prevents unused containers and Docker networks from being left behind on the GitHub Actions runner and makes repeated workflow executions more reliable.

---

# One-paragraph reflection

This CI pipeline complements the local Semgrep and ZAP scans from Labs 5.1 and 5.2 by automatically performing the same security checks whenever code changes are submitted to the repository. Local scans are useful during development because they provide fast feedback while investigating or fixing vulnerabilities, while the CI pipeline provides a consistent security check and automated security gate before changes are merged or accepted into the main branch. Local scans are still useful when debugging scanner configuration, testing new rules, or developing changes before pushing them to GitHub.



The main DevSecOps principle demonstrated by this implementation is that security testing should be integrated directly into the software development lifecycle rather than being performed only after the application has been completed.

SAST provides a static source-code perspective, while DAST provides a runtime application perspective. The security gate then converts the scanner results into an automated CI/CD decision.
