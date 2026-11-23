# 03 — Code Style

**Purpose:** Comprehensive coding conventions, comment philosophy, error handling strategies, logging rules, and anti-patterns across TypeScript, Kotlin, and Markdown in Lakbay. Read before submitting pull requests.

**Confidence note:** verified against initial repository baseline on 2026-11-17. The code wins (see 16).

---

## 1. Comment Philosophy: The "Why" at the Line

- **Never comment what code does:** Readable code explains what. Comments must explain *why* non-obvious constraints, business rules, or legal limits exist.
- **Cite regulatory or incident context:** When enforcing a throttle, spatial aggregation, or retention limit, reference the specific mandate or incident.
- **Example:**
  ```typescript
  // CORRECT: Explains the regulatory invariant and consequence of removal
  // DPA 2012 Rule: GPS coordinates must be truncated to 3 decimal places (~110m)
  // before storage to prevent individual commuter tracking without warrant.
  const anonymizedLat = Math.round(rawLat * 1000) / 1000;

  // INCORRECT: Redundant description of syntax
  // Round latitude to three decimals
  const lat = Math.round(rawLat * 1000) / 1000;
  ```

---

## 2. Tooling and Linting Standards

| Repository | Primary Linter / Formatter | Command | Strict Rules Enforced |
|---|---|---|---|
| `lakbay-api` | ESLint + Prettier + `tsc --noEmit` | `pnpm lint` / `pnpm typecheck` | `no-explicit-any`, `no-floating-promises`, strict null checks |
| `lakbay-web` | ESLint + React Hooks Plugin | `pnpm lint` | Exhaustive deps, strict prop types, no raw inline SQL |
| `lakbay-android` | Android Lint + ktlint + Detekt | `./gradlew lint ktlintCheck` | No hardcoded strings, no uncaught coroutine exceptions |
| `lakbay-specs` | markdownlint | `npx markdownlint-cli "**/*.md"` | Header line length, fence style consistency, link integrity |

---

## 3. Naming Conventions

- **TypeScript / JavaScript:**
  - Interfaces and Types: PascalCase (`UserRolePayload`, `ApiResponseEnvelope`).
  - Variables and Functions: camelCase (`verifyAuthToken()`, `sanitizeLocationData()`).
  - Constants: UPPER_SNAKE_CASE (`MAX_LOCATION_RETENTION_DAYS`, `DEFAULT_PAGE_SIZE`).
- **Kotlin / Android:**
  - Classes and Objects: PascalCase (`RegulatoryComplianceManager`).
  - Functions and Properties: camelCase (`fetchAuditLogs()`).
  - Compose Functions: PascalCase (`ComplianceReportCard()`).

---

## 4. Error Handling & Standard Responses

All API errors must return a standardized JSON error response envelope. Never return raw database stack traces or unhandled 500 HTML pages.

```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "ERR_UNAUTHORIZED_ROLE",
    "message": "User does not possess sufficient privileges to perform this audit update.",
    "details": { "requiredRole": "Administrator", "providedRole": "Contributor" }
  },
  "meta": {
    "timestamp": "2026-11-17T17:51:54Z",
    "requestId": "req-9a8b7c6d-5e4f"
  }
}
```

- **HTTP Status Codes:**
  - `400 Bad Request`: Schema validation failure via Zod.
  - `401 Unauthorized`: Missing, expired, or malformed JWT token.
  - `403 Forbidden`: Authenticated user lacks RBAC role permissions.
  - `404 Not Found`: Resource does not exist or user is blocked by RLS.
  - `422 Unprocessable Entity`: Business logic invariant violation.
  - `500 Internal Server Error`: Unexpected unhandled runtime error (must log error ID).

---

## 5. Logging Rules

1. **Zero PII Logging:** Never write Personally Identifiable Information (email, full name, phone number, raw GPS location) to logs.
   - *Miss → Exposure in cloud logs violates Philippine Data Privacy Act of 2012.*
2. **Structured JSON Logs:** All server logs in `lakbay-api` must emit structured JSON containing `requestId`, `timestamp`, `level`, and `userId` (hashed or UUID only).
3. **No Console Log in Production:** Android `Log.d()` and Web `console.log()` must be stripped during release builds.

---

## 6. Anti-Patterns and Banned Practices

| Banned Anti-Pattern | Cost That Banned It | Correct Replacement |
|---|---|---|
| **Direct Client SQL via PostgREST** | Client app exposed database schema directly, bypassing regulatory audit triggers. | Proxy all mutations through `lakbay-api` endpoints. |
| **Storing JWT in LocalStorage on Web** | XSS vulnerability allowed token theft. | Use HTTP-only, secure, SameSite cookies or memory-stored tokens with silent refresh. |
| **Silent Catch Blocks (`catch (e) {}`)** | Obscured database network drops; left UI in frozen loading state. | Log error with `requestId` and display user-friendly fallback state. |
| **Hardcoding Environment Fallbacks** | Caused accidental production connection to dev databases during CI deployments. | Fail fast at application boot if required env vars are missing. |

---

## 7. Commit Conventions (per `git-guidelines.md`)

- **Format:** `scope: Imperative summary` (50 chars or fewer, no trailing period, sentence case).
- **Allowed Application Scopes:**
  - `lakbay-api/<feature>:` or `lakbay-api:`
  - `lakbay-web/<feature>:` or `lakbay-web:`
  - `lakbay-android/<feature>:` or `lakbay-android:`
  - `lakbay-specs/<feature>:` or `lakbay-specs:`
- **Exclusion of AI Agent:** Never add AI agent identity, co-authorship, or signatures to commit headers, commit bodies, tags, or pull requests.
