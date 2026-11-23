# 11 — Testing

**Purpose:** Test strategy, coverage expectations, contract testing, manual testing drills, and failure dimensions across the Lakbay ecosystem. Read before implementing tests or verifying PRs.

**Confidence note:** verified against initial repository baseline on 2026-11-23. The code wins (see 16).

---

## 1. Test Pyramids & Coverage Expectations

| Component | Test Framework | Target Coverage | Critical Focus Areas |
|---|---|---|---|
| **`lakbay-api`** | Vitest / Jest + Supertest | > 85% on services & middleware | RBAC enforcement, DPA spatial truncation, error envelopes |
| **`lakbay-web`** | Vitest + React Testing Library | > 70% on components | Audit log filtering, auth guards, network error states |
| **`lakbay-android`** | JUnit + MockK + Espresso | > 75% on domain & data layers | Token refresh interceptor, offline caching, coordinate anonymization |
| **`lakbay-specs`** | markdownlint-cli | 100% link & format validation | Spec consistency, no broken relative links |

---

## 2. Mandatory Verification Drills

1. **Authentication Token Expiry Drill:**
   - Simulate token expiration while user is active. Verify client intercepts 401, refreshes session silently via Supabase, and retries request without user disruption.
   - *Miss → User abruptly kicked to login screen while filling transit report.*
2. **Offline Resilience Drill (Mobile):**
   - Toggle airplane mode on Android device. Submit a report; verify report enters local Room cache and syncs once connection is restored.
3. **Data Privacy Bounds Drill:**
   - Submit GPS coordinate with 7 decimal places (`10.7441234, 124.7915678`). Verify stored record in database is truncated to 3 decimals (`10.744, 124.791`) and personal ID is detached.

---

## 3. Mocking & Test Isolation Guidelines

- **Use Local Supabase Instance:** For API integration tests, run local Supabase via `supabase start` rather than mocking SQL queries.
  - *Miss → Mocks succeed but real PostgreSQL RLS policies block live queries.*
- **Never Test Against Production Database:** All test suites must use dedicated test environment variables.
