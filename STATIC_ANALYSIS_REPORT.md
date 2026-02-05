# Static Analysis Report (repo-wide) — oracle-db-examples-315632

Generated: 2026-02-05

Scope: This repository is a multi-language examples collection (Java, JavaScript/Node, Python, SQL/PLSQL, etc.). Findings below prioritize security and dependency risks first, then bug patterns and maintainability.

---

## Executive summary (prioritized)

### P0 — High: Dependency vulnerabilities in Node example projects (npm audit)
**Impact:** High severity DoS / header-manipulation issues present in transitive dependencies of several Node/Express sample apps.

**Evidence (projects with high findings):**
- `javascript/files-up-and-down/hr_app_buffering/`
- `javascript/files-up-and-down/hr_app_streaming/`
- `javascript/idcs-authentication/hr_app/`
- `javascript/rest-api/part-1-web-server-basics/hr_app/`
- `javascript/rest-api/part-2-database-basics/hr_app/`
- `javascript/rest-api/part-3-handling-get-requests/hr_app/`
- `javascript/rest-api/part-4-handling-post-put-and-delete-requests/hr_app/`
- `javascript/rest-api/part-5-manual-pagination-sorting-and-filtering/hr_app/`

**Representative vulnerable packages reported by `npm audit --audit-level=high`:**
- `qs < 6.14.1` (HIGH): arrayLimit bypass can cause memory exhaustion (DoS)
- `on-headers < 1.1.0`: response header manipulation issue
- Vulnerable dependency chains commonly include: `body-parser`, `express`, `morgan`, `express-session`

**Remediation notes:**
- In each affected directory run:
  - `npm audit fix`
- If `npm audit fix` requires major upgrades:
  - Update Express middleware stack to supported versions
  - Re-test sample endpoints and update documentation accordingly

---

### P1 — High: Insecure-by-default patterns that could be copied into real apps

#### 1) Hard-coded HTTP redirect URLs for IDCS auth sample
**File:**
- `javascript/idcs-authentication/hr_app/config/authentication.js`

**Issue:**
- Redirect URLs use `http://localhost:3000/...`:
  - `loginRedirectUrl: 'http://localhost:3000/callback'`
  - `logoutRedirectUrl: 'http://localhost:3000'`

**Risk:**
- Encourages insecure transport if copied to non-local environments.

**Remediation notes:**
- Parameterize redirect URLs via environment variables, document “use https in production”.
- Add a short security note in the sample README.

#### 2) Shell execution in a Python notebook
**File:**
- `python/python-oracledb/notebooks/4-CSV.ipynb`

**Issue:**
- Uses `os.system("wc -l testwrite.csv")`

**Risk:**
- Not exploitable as written, but promotes a risky pattern if later modified with user input.

**Remediation notes:**
- Prefer:
  - `subprocess.run(["wc","-l","testwrite.csv"], check=True)`
  - or pure Python counting

#### 3) `eval` usage in Nashorn examples/docs
**Files (examples/docs):**
- `javascript/nashorn/ReadMe-JavaScript-OJVM.md`
- `javascript/nashorn/selectJS-javax-wrapper.sql`

**Issue:**
- `engine.eval(...)` appears in examples.

**Risk:**
- `eval` is dangerous if fed untrusted input; acceptable for controlled examples with explicit warning.

**Remediation notes:**
- Ensure the evaluated script is a static/local resource.
- Add warnings in docs: “Do not eval untrusted input”.

---

## P2 — Medium: JDBC raw Statement usage / SQL concatenation footguns
**Representative files flagged by heuristic search (not necessarily vulnerable as written):**
- `java/jdbc/Tomcat_Servlet/src/UCPServlet.java`
- `java/jdbc/WebLogicServer_Servlet/src/UCPServlet.java`
- Many samples under `java/jdbc/**` and `java/HRWebApp/**` use `createStatement()`.

**Notes:**
- The inspected servlet examples use static SQL (DDL/DML with literals), so no injection vector is present as-is.
- These patterns become vulnerable when user input is concatenated into SQL strings.

**Remediation notes:**
- Add comments in samples: “use PreparedStatement and bind variables for any external input”.
- Where feasible, convert examples that accept parameters to `PreparedStatement`.

---

## P3 — Medium: TODO/FIXME placeholders and deliberate failing tests
Repo-wide search (`TODO|FIXME|XXX`) indicates:
- Many “TODO: replace DB_URL/DB_USERNAME/DB_PASSWORD” style placeholders.
- Some test files contain `fail("TODO: Fix this test");`.
- Some code paths throw placeholder exceptions like `IllegalStateException("TODO")`.

**Risk:**
- Not a security vulnerability by itself, but can confuse users or CI runs if tests are executed.

**Remediation notes:**
- Mark incomplete sections clearly in module READMEs.
- Consider isolating incomplete tests behind profiles or exclude patterns.

---

## P4 — Low: Formatting / misc
- Python syntax check: `python -m compileall -q python` succeeded (no syntax errors under `python/`).
- Some files show CRLF line endings (seen in grep output), which can cause noisy diffs on Unix.

---

## How this report was derived (methods)
- Python: `python -m compileall -q python`
- Heuristic security greps:
  - JS: `eval(`, `new Function`, `child_process.exec/spawn`
  - Python: `os.system`, `subprocess.*`, `pickle.loads`, `yaml.load`, `shell=True`
  - Java: `createStatement`, `Statement.execute`
  - Hardcoded `http://` search across languages
- Dependency audit:
  - `npm audit --audit-level=high` in each directory containing `package.json` under `javascript/**/hr_app/**`

---

## Suggested next steps (optional hardening)
1. Update Node sample `package-lock.json` / dependencies to clear HIGH findings and re-run `npm audit`.
2. Add brief “Security Notes” sections to READMEs for:
   - IDCS auth sample
   - Nashorn `eval` sample
   - Any sample executing OS commands
3. (If treating examples as templates) Refactor parameterized JDBC examples to use bind variables everywhere.

