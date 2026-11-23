# 02 — Project Structure

**Purpose:** Complete annotated directory tree for the four repositories in the Lakbay ecosystem, establishing strict code placement rules and structural naming conventions. Read before creating new files, packages, or directories.

**Confidence note:** verified against initial repository baseline on 2026-11-17. The code wins (see 16).

---

## 1. Multi-Repository Ecosystem Tree

The workspace contains four independent Git repositories coordinated under the `lakbay-ecosystem` directory:

```
lakbay-ecosystem/
├── AGENTS.md                          # Global workspace entry point & rule definitions
├── handoff.md                         # Senior engineer skills handoff instructions
│
├── lakbay-specs/                      # Repository: Ecosystem specifications & contracts
│   ├── .git/
│   ├── AGENTS.md                      # Spec repo summary instructions
│   ├── business-logic-v1.0.md         # Baseline functional requirements & role definitions
│   ├── git-guidelines.md              # Unified commit convention & versioning standard
│   ├── sense-making.md                # Environmental scanning methodology & criteria
│   ├── planning/
│   │   └── sense-making.csv           # PESTLE environmental scan data findings
│   └── skills/                        # Institutional engineering knowledge library (00–16)
│       ├── 00_INDEX.md
│       ├── INBOX.md
│       └── [01–16]_*.md
│
├── lakbay-api/                        # Repository: Backend API gateway & mediator
│   ├── .git/
│   ├── AGENTS.md                      # Backend service scope & guidelines
│   ├── package.json                   # Node.js runtime configuration
│   ├── tsconfig.json                  # TypeScript compiler settings
│   ├── src/
│   │   ├── index.ts                   # Fastify/Express server entry point
│   │   ├── config/                    # Environment variable validation & secrets load
│   │   ├── middleware/                # JWT auth, role RBAC, DPA privacy scrubbers
│   │   ├── routes/                    # API v1 routes (/auth, /regulatory, /users)
│   │   ├── services/                  # Supabase client orchestration & business rules
│   │   └── types/                     # Shared TypeScript request/response contracts
│   └── tests/                         # Integration and unit test suites
│
├── lakbay-web/                        # Repository: Regulatory & compliance web app
│   ├── .git/
│   ├── AGENTS.md                      # Web application scope & guidelines
│   ├── package.json                   # Web dependencies (React/Next.js/Vite)
│   ├── src/
│   │   ├── components/                # UI components (compliance tables, audit views)
│   │   ├── pages/                     # Routed pages (Dashboard, AuditLogs, Settings)
│   │   ├── hooks/                     # Custom React hooks (useAuth, useCompliance)
│   │   ├── services/                  # HTTP client communicating with lakbay-api
│   │   └── types/                     # Local presentation state definitions
│   └── tests/                         # Component and e2e test specifications
│
└── lakbay-android/                    # Repository: Android mobile client
    ├── .git/
    ├── AGENTS.md                      # Mobile client scope & guidelines
    ├── build.gradle.kts               # Root Gradle build script
    └── app/
        ├── build.gradle.kts           # Module-level Android dependencies
        └── src/main/
            ├── AndroidManifest.xml    # App manifest & permissions
            ├── java/org/lakbay/
            │   ├── data/              # Repositories & LakbayApi Retrofit service
            │   ├── domain/            # Use cases & business models
            │   └── ui/                # Activities, Jetpack Compose screens, ViewModels
            └── res/                   # Drawables, layouts, localized strings
```

---

## 2. Placement Rules: Where Code Goes and Must Not Go

| Category | Permitted Locations | Prohibited Locations | Why & What Breaks |
|---|---|---|---|
| **Ecosystem Contracts** | `lakbay-specs/` | `lakbay-api/src/types/` (must mirror specs, not invent them) | *Miss → Deserialization drift across Android and Web clients.* |
| **Database Queries & Mutations** | `lakbay-api/src/services/` | `lakbay-web/src/` or `lakbay-android/app/` | *Miss → Clients bypass regulatory audit hooks, violating DPA 2012 compliance.* |
| **Secrets & Keys** | Server runtime environment (`.env`) | Client bundles, Git trees, or `lakbay-specs/` | *Miss → Exposure of `SUPABASE_SERVICE_ROLE_KEY` leads to full DB compromise.* |
| **Business Logic** | `lakbay-api/src/services/` | UI Components in `lakbay-web` or `lakbay-android` | *Miss → Inconsistent business rules and double-charge or unverified status mutations.* |
| **UI State & Layouts** | `lakbay-web/src/components/`, `lakbay-android/.../ui/` | `lakbay-api/` | *Miss → Unwanted coupling between presentation layers and headless services.* |

---

## 3. Structural Naming Conventions

### File & Directory Conventions
- **`lakbay-specs/skills/`:** Every document must use the two-digit padded prefix followed by uppercase topic: `NN_TOPIC.md` (e.g., `01_ARCHITECTURE.md`, `14_DEVELOPER_WORKFLOWS.md`).
- **TypeScript / JavaScript:**
  - Files: camelCase for utilities (`tokenValidator.ts`), PascalCase for React components (`ComplianceDashboard.tsx`).
  - Directories: lowercase kebab-case (`auth-guard/`, `audit-logger/`).
- **Kotlin / Android:**
  - Classes & Interfaces: PascalCase (`LakbayAuthRepository.kt`, `TransitReportViewModel.kt`).
  - XML Layouts & Drawables: lowercase snake_case (`activity_main.xml`, `ic_transit_bus.xml`).

### Semantic Identifiers
- **Database Tables:** lowercase snake_case plural (`users`, `compliance_audit_logs`, `transit_routes`).
- **Environment Variables:** UPPER_SNAKE_CASE prefixed by service namespace (`SUPABASE_URL`, `SUPABASE_ANON_KEY`, `LAKBAY_API_PORT`).
- **Release Tags:** `<PREFIX>.0X.Y.00Z` (e.g., `BCN.01.0.001`, `WCN.01.0.001`, `ACN.01.0.001`, `SCN.01.0.001`).
