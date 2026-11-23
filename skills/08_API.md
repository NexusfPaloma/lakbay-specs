# 08 — API

**Purpose:** REST API standards, standard response and error envelopes, route inventory, authentication, request validation, and endpoint extension procedures for `lakbay-api`. Read before adding or modifying server endpoints.

**Confidence note:** verified against initial repository baseline on 2026-11-23. The code wins (see 16).

---

## 1. Standard Response Envelopes

Every endpoint in `lakbay-api` must return the standard JSON envelope. Clients (`lakbay-web` and `lakbay-android`) rely on the root `success` boolean to discriminate responses.

### Success Envelope
```json
{
  "success": true,
  "data": {
    "id": "c1f8a84b-9721-4d1a-8c33-6df51b2a9254",
    "status": "active"
  },
  "error": null,
  "meta": {
    "timestamp": "2026-11-23T17:58:00Z",
    "requestId": "req-4a2b-8c1d"
  }
}
```

### Error Envelope
```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "ERR_VALIDATION_FAILED",
    "message": "Input validation error",
    "details": {
      "field": "coordinates",
      "issue": "Precision exceeds permitted aggregation limit"
    }
  },
  "meta": {
    "timestamp": "2026-11-23T17:58:00Z",
    "requestId": "req-4a2b-8c1d"
  }
}
```
- *Miss → Clients encounter parsing errors or unhandled null reference exceptions.*

---

## 2. Route Inventory & Authorization Matrix

| Method | Route | Auth Required | Minimum Role | Purpose |
|---|---|---|---|---|
| `GET` | `/v1/health` | None | Public / Guest | Health check & readiness probe |
| `POST` | `/v1/auth/session` | Bearer JWT | Guest / Any | Validate OAuth exchange with Supabase |
| `GET` | `/v1/users/me` | Bearer JWT | Contributor | Fetch authenticated profile details |
| `POST` | `/v1/telemetry/density` | Bearer JWT | Contributor | Ingest aggregated commuter telemetry |
| `GET` | `/v1/compliance/audits` | Bearer JWT | Administrator | Fetch regulatory audit logs |
| `POST` | `/v1/compliance/override` | Bearer JWT | System Administrator | Execute administrative system override |

---

## 3. How to Add a New Endpoint

1. **Update Specs First:** Add the proposed contract to `lakbay-specs` (see [`04_CHANGE_RULES.md`](file:///c:/Users/nexus/Documents/lakbay-ecosystem/lakbay-specs/skills/04_CHANGE_RULES.md) Lockstep Set 2).
2. **Define Validation Schema:** Create Zod schema in `lakbay-api/src/schemas/<feature>.ts`.
   - *Miss → Unsanitized inputs reach database queries.*
3. **Implement Controller:** Add handler in `lakbay-api/src/routes/v1/<feature>.ts`.
4. **Register Route:** Attach route with appropriate RBAC middleware in `lakbay-api/src/routes/v1/index.ts`.
5. **Update Client SDKs:** Add request method to `lakbay-web/src/services/` and `lakbay-android/.../LakbayApiService.kt`.

---

## 4. Common Mistakes

- **Returning Naked Arrays:** Always wrap arrays inside `{ "items": [...] }` within `data` to allow future metadata expansion without breaking clients.
- **Leaking Stack Traces:** Database driver exceptions must be caught and mapped to standard error codes (`ERR_DATABASE_ERROR`); never return raw PostgreSQL error objects to the client.
