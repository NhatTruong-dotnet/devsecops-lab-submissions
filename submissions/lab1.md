# Lab 1 — Submission

## Triage Report: OWASP Juice Shop

### Scope & Asset

- **Asset:** OWASP Juice Shop (local lab instance)
- **Image:** `bkimminich/juice-shop:v20.0.0`
- **Image digest:** `sha256:fd58bdc9745416afce8184ee0666278a436574633ea7880365153a63bfd418b0`
- **Host OS:** Ubuntu 24.04
- **Docker version:** 29.6.1

### Deployment Details

- **Run command:**
  ```bash
  docker run -d --name juice-shop -p 127.0.0.1:3000:3000 bkimminich/juice-shop:v20.0.0
  ```
- **Access URL:** <http://127.0.0.1:3000>
- **Network exposure:** ☑ Yes (127.0.0.1 only)
- **Container restart policy:** `no`

### Health Check

- **HTTP status (`/`):** `200`

- **API check (first 200 chars of `/api/Products`):**
  ```text
  {"status":"success","data":[{"id":1,"name":"Apple Juice (1000ml)","description":"The all-time classic.","price":1.99,"deluxePrice":0.99,"image":"apple_juice.jpg","createdAt":"2026-0
  ```

- **Container status:**
  ```text
  CONTAINER ID   IMAGE                           COMMAND                  CREATED          STATUS          PORTS                      NAMES
  327f36b1b422   bkimminich/juice-shop:v20.0.0   "/nodejs/bin/node /j…"   20 minutes ago   Up 20 minutes   127.0.0.1:3000->3000/tcp   juice-shop
  ```

### Initial Surface Snapshot

- ☑ Login/Registration visible
- ☑ Product listing and search available
- ☑ Admin/Account area discoverable
- ☑ Client-side errors visible in DevTools console
- **Local storage / cookies:** `<list what you saw>`

### Security Headers

**Command:**

```bash
curl -I http://127.0.0.1:3000 2>&1 | head -20
```

**Output:**

```text
% Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
  0   9903   0      0   0      0      0      0                              0
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
Feature-Policy: payment 'self'
X-Recruiting: /#/jobs
Accept-Ranges: bytes
Cache-Control: public, max-age=0
Last-Modified: Mon, 27 Jul 2026 13:32:17 GMT
ETag: W/"26af-19fa3c6b8ca"
Content-Type: text/html; charset=UTF-8
Content-Length: 9903
Vary: Accept-Encoding
Date: Mon, 27 Jul 2026 13:55:13 GMT
Connection: keep-alive
Keep-Alive: timeout=5
```

**Missing security headers (OWASP Top 10:2025 – A06):**

- [x] `Content-Security-Policy`
- [x] `Strict-Transport-Security`
- [ ] `X-Content-Type-Options: nosniff`
- [ ] `X-Frame-Options`

## PR Template Setup

- File: `.github/pull_request_template.md`
- Sections included: Summary / Changes / Impact / Testing / Screenshots / Deployment Notes / Reviewer Notes
- Checklist items:
  
- [ ] PR title follows the convention
- [ ] Code is self-reviewed
- [ ] Tests added or updated
- [ ] Documentation updated
- [ ] No secrets committed
- [ ] No unnecessary files included
      
- Auto-fill verified: [X] Yes — PR description showed my template (https://github.com/NhatTruong-dotnet/devsecops-lab-submissions/pull/2)

## Bonus: CI Smoke Test

- Workflow file: `.github/workflows/lab1-smoke.yml`
- Trigger: `pull_request` on main
- Run URL (must be green): [run history](https://github.com/NhatTruong-dotnet/devsecops-lab-submissions/actions/runs/30330905322)
- Workflow run duration: 15s
- Curl response excerpt:
  ```
  Homepage HTTP status: 200
  ```
- [x] Task 1 done — Juice Shop deployed, triage report in submissions/lab1.md
- [x] Task 2 done — .github/PULL_REQUEST_TEMPLATE.md created
- [x] Bonus done — lab1-smoke.yml runs green on this PR
