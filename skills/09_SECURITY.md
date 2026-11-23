# 09 — Security

**Purpose:** Threat modeling, authentication and authorization architecture, Data Privacy Act compliance, secrets handling, and security checklists for Lakbay. Read before touching authentication, cryptography, or user data.

**Confidence note:** verified against initial repository baseline on 2026-11-23. The code wins (see 16).

---

## 1. Threat Model & Asset Protection

| Asset | Threat | Mitigating Control |
|---|---|---|
| **Commuter Location Data** | Re-identification of individuals via GPS trails; stalker or surveillance risks. | Aggressive spatial truncation (3 decimals ~110m), temporal bucketing, no persistent per-user breadcrumb trails. |
| **Supabase Service Key** | Malicious client extracts key and bypasses RLS to dump or delete all tables. | Key is restricted to server-side `lakbay-api` runtime; never included in Android APK or Web bundle. |
| **User Role Claims** | Client-side manipulation of user claims to escalate privileges to `System Administrator`. | Roles are resolved server-side from `user_profiles` table, never trusted from client-provided headers. |
| **Regulatory Audit Trail** | Compromised admin account deletes evidence of unauthorized configuration change. | `compliance_audit_logs` has append-only RLS policy (no `UPDATE` or `DELETE` grants). |

---

## 2. Authentication & Authorization Layers

1. **Authentication (Supabase Auth + Google OAuth):**
   - Mobile and Web use Google Sign-In with PKCE flow.
   - Tokens are issued as standard RS256-signed JWTs with 1-hour expiration.
2. **Authorization (Layer 1 — API RBAC):**
   - `lakbay-api` middleware validates role hierarchy:
     `System Administrator` > `Administrator` > `Moderator` > `Contributor` > `Guest`.
3. **Authorization (Layer 2 — PostgreSQL RLS):**
   - Even if API middleware is misconfigured, Supabase Row-Level Security restricts query results by evaluating `auth.uid()`.

---

## 3. Philippine Data Privacy Act (DPA 2012) Mandates

- **No Per-User Trails:** Location coordinates must never be stored in direct association with an immutable user identity.
- **Aggregation Thresholds:** Telemetry density is only queryable if at least 5 distinct anonymous pings exist in a given spatial-temporal cell.
- **Retention Limits:** Raw ephemeral pings expire after 48 hours; aggregate metrics are purged after 90 days.
- **Data Resale Prohibition:** Commercial resale of raw or derived user transit data is strictly prohibited by architectural contract.

---

## 4. Secrets Handling & Change Checklist

- **Environment Variables Only:** Secrets must be injected via runtime environment:
  - `SUPABASE_URL`: Public endpoint.
  - `SUPABASE_ANON_KEY`: Safe for public client consumption with RLS.
  - `SUPABASE_SERVICE_ROLE_KEY`: **Confidential**; server-only.
  - `SUPABASE_JWT_SECRET`: **Confidential**; used for local signature verification.
- **Security Pre-Commit Checklist:**
  - [ ] No `.env` files or hardcoded keys committed to git.
  - [ ] No PII (names, emails, phone numbers, exact coordinates) in log statements.
  - [ ] All new database tables have `ENABLE ROW LEVEL SECURITY` explicitly enabled.
