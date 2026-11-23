# 01 — Architecture

**Purpose:** Architectural blueprint of the Lakbay platform, defining component boundaries, end-to-end data flows, regulatory constraints, and recorded architectural decisions. Read before proposing system-level changes or adding integrations.

**Confidence note:** verified against initial repository baseline on 2026-11-17. The code wins (see 16).

---

## 1. System Overview

Lakbay is a multi-tier regulatory and transit platform designed for the Philippine transportation ecosystem. It comprises a mobile Android application for commuters and operators (`lakbay-android`), an administrative web portal for regulatory compliance and audit management (`lakbay-web`), an intermediate API orchestration layer (`lakbay-api`), and an ecosystem specification repository (`lakbay-specs`). Data persistence, authentication, and database-level security policies are managed through a managed Supabase (PostgreSQL + PostgREST + Auth) infrastructure.

```
+--------------------+        +--------------------+
|   lakbay-android   |        |     lakbay-web     |
| (Mobile commuters) |        | (Admin/Regulatory) |
+---------+----------+        +---------+----------+
          |                             |
          +--------------+--------------+
                         | HTTPS / REST / JWT
                         v
              +--------------------+
              |     lakbay-api     |
              | (Fastify/Node.js)  |
              |  - Auth mediation  |
              |  - DPA sanitization|
              |  - Audit logging   |
              +---------+----------+
                         | Service Key / RLS Context
                         v
              +--------------------+
              |      Supabase      |
              |  - PostgreSQL      |
              |  - Row Level Sec   |
              |  - Supabase Auth   |
              +--------------------+
```

---

## 2. Component Ownership

| Component | Repository | Primary Ownership & Responsibilities | Boundary Invariants |
|---|---|---|---|
| **Specifications** | `lakbay-specs` | Business rules, user roles, security baselines, environmental scan, API contracts, skills library. | Single source of truth. Contains no runnable application code. |
| **API Gateway** | `lakbay-api` | Mediates traffic between clients and Supabase; enforces business logic, audit trails, and privacy transformations. | *Miss → raw client requests access DB without regulatory validation or PII redaction.* |
| **Web Front-end** | `lakbay-web` | Regulatory compliance dashboard, administrative reporting, audit visualizations. | Restricted to Administrator and Moderator access. Never persists credentials locally. |
| **Android Client** | `lakbay-android` | Commuter routing, transit reporting, mobile Google OAuth sign-in. | Treats mobile devices as hostile. Never stores Supabase `service_role` keys. |
| **Database & Auth** | Supabase Engine | Persistent storage, ACID transactions, Row-Level Security (RLS) enforcement, user identity. | Direct client DB access is restricted to authenticated RLS queries. |

---

## 3. End-to-End Data Flows

### A. Authentication Flow
1. **Initiation:** User clicks "Sign in with Google" on `lakbay-android` or `lakbay-web`.
2. **Identity Handshake:** Client completes Google OAuth2 PKCE flow and receives an ID token.
3. **Session Exchange:** Client exchanges token with Supabase Auth (`/auth/v1/token?grant_type=id_token`).
4. **Token Issuance:** Supabase returns an encrypted JWT with `sub`, `email`, and assigned `user_metadata`.
5. **API Verification:** Client attaches JWT in `Authorization: Bearer <jwt>` to requests made to `lakbay-api`.
6. **Role Resolution:** `lakbay-api` verifies signature against `SUPABASE_JWT_SECRET` and attaches user role context (`System Administrator`, `Administrator`, `Moderator`, `Contributor`, `Guest`).

### B. Regulatory Compliance & Data Privacy Flow
1. **Event Dispatch:** Client submits transit telemetry or operator reports.
2. **Mediation:** `lakbay-api` intercepts the payload.
3. **Privacy Sanitization:** Under the Philippine Data Privacy Act of 2012, raw location trails are stripped of individual identifiers and bucketed into spatial-temporal aggregates (see [`09_SECURITY.md`](file:///c:/Users/nexus/Documents/lakbay-ecosystem/lakbay-specs/skills/09_SECURITY.md)).
4. **Audit Trail:** Regulatory audit logs are written synchronously to `compliance_audit_logs`.
5. **Persistence:** Sanitized records are written to PostgreSQL.

---

## 4. Hard Architectural Boundaries

1. **No Direct Client Database Write Access:** Neither `lakbay-android` nor `lakbay-web` may execute arbitrary `INSERT`/`UPDATE` operations directly against Supabase tables without passing through `lakbay-api` validation middleware.
   - *Miss → Regulatory audit logs are bypassed, violating compliance mandates.*
2. **Zero Service-Role Keys in Client Distributions:** `SUPABASE_SERVICE_ROLE_KEY` must never exist in `lakbay-android` APKs or `lakbay-web` client bundles.
   - *Miss → Full database compromise bypassing all RLS policies.*
3. **Spec-First Synchronization:** Any modification to data models, API endpoints, or user role hierarchies must land in `lakbay-specs` before implementation begins in downstream codebases.
   - *Miss → Uncoordinated schema drift and runtime deserialization failures.*

---

## 5. Architectural Decision Records (ADRs)

### Decision: Multi-Repository Ecosystem Topology (2026-11-17, Nexus Paloma)
- **Status:** Accepted
- **Context:** Lakbay consists of Android, Web, Backend, and Governance layers with separate release cadences.
- **Choice:** Maintain four discrete git repositories (`lakbay-specs`, `lakbay-api`, `lakbay-android`, `lakbay-web`) sharing a unified git and versioning standard.
- **Alternatives:** Monorepo with Lerna/Turborepo; monolithic backend with bundled static assets.
- **Consequences:** Requires strict lockstep discipline across repos (see [`04_CHANGE_RULES.md`](file:///c:/Users/nexus/Documents/lakbay-ecosystem/lakbay-specs/skills/04_CHANGE_RULES.md)) and atomic version tagging.

### Decision: Dedicated Mediation API (`lakbay-api`) over Direct BaaS (2026-11-17, Nexus Paloma)
- **Status:** Accepted
- **Context:** Supabase offers direct client-to-database queries via PostgREST with RLS.
- **Choice:** Insert `lakbay-api` as a mandatory reverse proxy and business logic mediator.
- **Alternatives:** Pure serverless client-to-Supabase architecture.
- **Consequences:** Slight latency overhead on write paths, but guarantees strict regulatory audit logging and Data Privacy Act sanitization prior to DB write.

### Decision: Supabase for Identity and Data Persistence (2026-11-17, Nexus Paloma)
- **Status:** Accepted
- **Context:** Need managed PostgreSQL with reliable OAuth2 integration and real-time event capability.
- **Choice:** Adopt Supabase managed instance with declarative SQL migrations and Row Level Security.
- **Alternatives:** Custom self-hosted MongoDB + Auth0; AWS RDS + Cognito.
- **Consequences:** Tight coupling to PostgreSQL RLS idioms; operational dependency on Supabase service availability.

---

## 6. Ranked Technical Debt Register

| Rank | Debt Item | Risk Impact | Remediation Plan |
|---|---|---|---|
| **1** | Greenfield Client Implementations | High: `lakbay-android` and `lakbay-web` lack full application scaffolding. | Scaffold initial app modules following contracts in `lakbay-specs`. |
| **2** | API Middleware Stubs | High: `lakbay-api` requires concrete route handlers and Fastify plugin integration. | Implement `/v1/` routes according to [`08_API.md`](file:///c:/Users/nexus/Documents/lakbay-ecosystem/lakbay-specs/skills/08_API.md). |
| **3** | Automated Cross-Repo CI Contract Tests | Medium: Contract breaks between API and clients are not currently blocked by CI. | Add GitHub Actions matrix to test API OpenAPI schemas against client models. |
