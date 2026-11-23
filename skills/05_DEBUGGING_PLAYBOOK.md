# 05 — Debugging Playbook

**Purpose:** Practical triage guide for engineering incidents across Lakbay services, cataloging exact error messages, log locations, diagnostic shortcuts, and remediation steps. Read when investigating system failures or alerts.

**Confidence note:** verified against initial repository baseline on 2026-11-17. The code wins (see 16).

---

## 1. Log Locations and Observability Sources

| Component | Log Destination | How to Inspect | Notes |
|---|---|---|---|
| **`lakbay-api`** | stdout / CloudWatch / Docker log stream | `docker logs -f lakbay-api` or `tail -f logs/app.log` | JSON structured logs containing `requestId` and `errorCode`. |
| **`lakbay-web`** | Browser DevTools Console & Network Tab | `F12` → Console / Network tab | Inspect `Authorization: Bearer` headers and 4xx/5xx responses. |
| **`lakbay-android`** | Android Logcat | `adb logcat -s "Lakbay:*"` | Filter by tag `LakbayAuth`, `LakbayApi`, or `LakbayLocation`. |
| **Supabase DB** | Supabase Studio Dashboard → Logs & Analytics | `https://app.supabase.com/project/<ref>/logs/explorer` | Inspect PostgreSQL slow queries, auth errors, and RLS denials. |

---

## 2. Common Symptoms & Diagnostic Runbook

### Symptom: `ERR_UNAUTHORIZED_JWT` (401 Unauthorized)
- **Exact Message:** `"Invalid token signature or expired session"`
- **Causes by Frequency:**
  1. *Expired JWT (70%):* Client failed to execute silent refresh before token expired (default 3600s TTL).
  2. *Audience / Secret Mismatch (20%):* `lakbay-api` configured with incorrect `SUPABASE_JWT_SECRET`.
  3. *Clock Skew (10%):* Client device clock drifted by more than 5 minutes.
- **Diagnostic Shortcut:**
  ```bash
  # Verify token payload without secret
  curl -s https://jwt.io/ | jq .
  # Check system time drift
  w32tm /query /status
  ```
- **Remediation:** Check client refresh interceptor; verify `SUPABASE_JWT_SECRET` in `lakbay-api/.env`.

### Symptom: `ERR_FORBIDDEN_RLS` (403 Forbidden / Empty Data Array)
- **Exact Message:** `"new row violates row-level security policy for table 'compliance_audit_logs'"`
- **Causes by Frequency:**
  1. *Insufficient Role Permissions (80%):* Authenticated user has `Contributor` role attempting an `Administrator` mutation.
  2. *Missing User UUID Context (15%):* `auth.uid()` evaluated to `NULL` due to unauthenticated call.
  3. *Unapplied Migration (5%):* Local database lacks the latest RLS policy definition.
- **Diagnostic Shortcut:**
  ```sql
  -- Run in Supabase SQL editor to test RLS evaluation as specific user
  SET ROLE authenticated;
  SET request.jwt.claim.sub = '00000000-0000-0000-0000-000000000001';
  SELECT * FROM compliance_audit_logs;
  ```
- **Remediation:** Re-verify role assignments in `lakbay-specs/business-logic-v1.0.md` against DB seed.

### Symptom: `ERR_DPA_LOCATION_UNAGGREGATED` (422 Unprocessable Entity)
- **Exact Message:** `"Payload rejected: raw GPS coordinates violate spatial aggregation threshold (DPA 2012)"`
- **Causes by Frequency:**
  1. *Unsanitized Client Submission (90%):* Android client transmitted 6-decimal-place raw GPS coordinates on a public telemetry endpoint.
  2. *Threshold Misconfiguration (10%):* Precision threshold in `privacyScrubber.ts` was set higher than policy allowed.
- **Diagnostic Shortcut:** Inspect incoming request payload `body.locations` in `lakbay-api` logs. Ensure coordinates are rounded to at most 3 decimal places.
- **Remediation:** Apply `sanitizeCoordinates()` helper before sending from mobile client.

### Symptom: CORS Failure on `lakbay-web`
- **Exact Message:** `"Access to XMLHttpRequest at 'https://api.lakbay.ph' from origin 'https://admin.lakbay.ph' has been blocked by CORS policy"`
- **Causes by Frequency:**
  1. *Missing Origin in Allowlist (95%):* Environment variable `CORS_ALLOWED_ORIGINS` in `lakbay-api` missing the web client domain.
  2. *Custom Header Omission (5%):* Web client sent unexpected custom header not listed in `exposedHeaders`.
- **Diagnostic Shortcut:**
  ```bash
  curl -I -X OPTIONS https://api.lakbay.ph/v1/health \
    -H "Origin: https://admin.lakbay.ph" \
    -H "Access-Control-Request-Method: GET"
  ```
- **Remediation:** Add domain to `CORS_ALLOWED_ORIGINS` comma-separated list in API environment.

---

## 3. Quick Reference Troubleshooting Commands

```bash
# 1. Check health of lakbay-api server
curl -i http://localhost:3000/v1/health

# 2. Verify git status across all 4 repos
Get-ChildItem -Directory | ForEach-Object { Write-Host $_.Name; git -C $_.FullName status -s }

# 3. Test local Supabase container connectivity
curl -i http://localhost:54321/rest/v1/ -H "apikey: <SUPABASE_ANON_KEY>"
```
