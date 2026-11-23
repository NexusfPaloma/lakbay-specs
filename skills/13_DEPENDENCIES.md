# 13 — Dependencies

**Purpose:** Dependency policy, approved packages, upgrade traps, and deliberate library rejections across Lakbay repositories. Read before adding or updating any package or SDK.

**Confidence note:** verified against initial repository baseline on 2026-11-23. The code wins (see 16).

---

## 1. Third-Party Dependency Policy

- **Minimal Surface Area:** Every third-party library adds maintenance burden and security risk. Always evaluate native language/runtime capabilities before introducing a package.
- **Licensing:** Only permissive open-source licenses are permitted (MIT, Apache-2.0, BSD-3-Clause). AGPL and GPL-licensed libraries are strictly prohibited.
- **Vulnerability Auditing:** `pnpm audit` (Web/API) and Gradle dependency checks (Android) must run in CI. High or Critical CVEs fail the build.

---

## 2. Approved Ecosystem Core Dependencies

| Component | Approved Libraries | Purpose |
|---|---|---|
| `lakbay-api` | Fastify, `@supabase/supabase-js`, Zod, jose (JWT) | High-performance HTTP server, database client, schema validator, cryptographic verification. |
| `lakbay-web` | React, Vite / Next.js, TailwindCSS, Axios | UI presentation, build bundling, styling, HTTP transport. |
| `lakbay-android` | Kotlin Coroutines, Retrofit, OkHttp, Jetpack Compose | Mobile concurrency, networking, reactive UI. |

---

## 3. Deliberate Rejections & Upgrade Traps

| Library / Tool | Decision | Rationale |
|---|---|---|
| **Prisma ORM** | **Rejected** | Introduces heavy Rust query engine binary and complicates native Supabase RLS integration. |
| **Client Supabase SDK with Direct DB Mutations** | **Rejected in Mobile/Web** | Bypasses `lakbay-api` regulatory mediation and DPA audit hooks. |
| **Redux / Heavy Global State** | **Rejected in Web** | Overkill for dashboard requirements; React Context / TanStack Query provides cleaner lifecycle management. |

---

## 4. Upgrade Traps
- **Supabase JS SDK v2 Upgrades:** Major version bumps change auth listener callback signatures (`onAuthStateChange`). Always verify token refresh logic before merging SDK upgrades.
