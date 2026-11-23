# 04 — Change Rules

**Purpose:** Exhaustive catalog of cross-repository lockstep sets, hidden coupling, high-risk change zones, and blast-radius verification requirements across the Lakbay ecosystem. Read before modifying shared models, routes, or configurations.

**Confidence note:** verified against initial repository baseline on 2026-11-17. The code wins (see 16).

---

## 1. Cross-Repository Lockstep Sets

A lockstep set is a collection of files across different repositories or modules that must be updated together in the same change or coordinated deployment. Failing to update any member causes silent runtime breakage or security failures.

### Lockstep Set 1: User Roles and Permission Hierarchy
When adding, modifying, or deprecating a user level or permission tier:
1. `lakbay-specs/business-logic-v1.0.md` (Authority table)
2. `lakbay-api/src/types/auth.ts` (`enum UserLevel { SYSTEM_ADMIN, ADMIN, MODERATOR, CONTRIBUTOR, GUEST }`)
3. `lakbay-api/src/middleware/rbac.ts` (Route access gates)
4. `lakbay-web/src/types/auth.ts` (Navigation & component visibility)
5. `lakbay-android/app/src/main/java/org/lakbay/domain/model/UserRole.kt` (Mobile role dispatch)
- *Miss → User is granted access to forbidden admin actions in API or UI crashes on undefined role serialization.*

### Lockstep Set 2: API Contract and Response Envelope
When modifying API request schemas or standard JSON response envelopes:
1. `lakbay-specs/skills/08_API.md` (Specification contract)
2. `lakbay-api/src/types/envelope.ts` (API response serializer)
3. `lakbay-web/src/services/apiClient.ts` (Web Axios/Fetch interceptor)
4. `lakbay-android/app/src/main/java/org/lakbay/data/remote/ApiResponse.kt` (Retrofit Gson/Moshi converter)
- *Miss → Mobile or web client throws unhandled deserialization exception; UI displays white screen.*

### Lockstep Set 3: Data Privacy Act Retention & Aggregation Policies
When altering location retention duration or geo-spatial aggregation precision:
1. `lakbay-specs/business-logic-v1.0.md` (Required Solution Layers, Section 39)
2. `lakbay-api/src/middleware/privacyScrubber.ts` (GPS precision truncation & retention TTL worker)
3. `lakbay-web/src/components/RegulatoryComplianceView.tsx` (Compliance threshold reporting)
- *Miss → Fine or legal sanction under Philippine National Privacy Commission (NPC) for non-compliant PII retention.*

### Lockstep Set 4: Ecosystem Release Tags & Versioning
When bumping weekly release tags or maintenance branches:
1. `lakbay-specs/git-guidelines.md` (Tag derivation tables)
2. `lakbay-api/package.json` (`version`)
3. `lakbay-web/package.json` (`version`)
4. `lakbay-android/app/build.gradle.kts` (`versionName` & `versionCode`)
- *Miss → Client and API mismatched build telemetry; inability to triage release-specific production crashes.*

---

## 2. High-Risk Change Areas

| Subsystem | Risk Description | Failure Mode | Mandatory Retest Step |
|---|---|---|---|
| **Supabase RLS Policies** | Modifying PostgreSQL row-level security policies directly in database migrations. | Leaks records across tenants or completely blocks client queries. | Run full integration test suite with both `Guest` and `Admin` JWT tokens. |
| **OAuth2 Callback Handlers** | Updating redirect URIs or PKCE challenge generators in client apps. | Complete login outage for Android and Web users. | Perform manual OAuth2 Google Sign-in on a fresh emulator and web incognito window. |
| **Location Telemetry Pipeline** | Modifying coordinates ingestion endpoint in `lakbay-api`. | High-volume write lock contention or unredacted GPS logging. | Execute load drill with 500 concurrent simulated location pings; verify `compliance_audit_logs`. |

---

## 3. Retest Matrix by Repository Change

| Modified Repository | Required Regression Tests | Mandatory Verification Commands |
|---|---|---|
| `lakbay-specs` | Verify documentation links, markdownlint formatting, and lockstep consistency. | `npx markdownlint-cli "**/*.md"` |
| `lakbay-api` | Run unit tests, TypeScript typechecking, and endpoint integration suite. | `pnpm --filter lakbay-api test && pnpm --filter lakbay-api typecheck` |
| `lakbay-web` | Run linter, production bundle build, and smoke test authentication flow. | `pnpm --filter lakbay-web build && pnpm --filter lakbay-web test` |
| `lakbay-android` | Run Gradle lint, unit tests, and assemble debug APK. | `./gradlew testDebugUnitTest assembleDebug` |
