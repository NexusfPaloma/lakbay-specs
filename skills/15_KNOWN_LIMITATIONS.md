# 15 — Known Limitations

**Purpose:** Catalog of known architectural gaps, deferred features, technical debt, and redesign candidates across the Lakbay platform. Read before undertaking refactoring or planning sprint goals.

**Confidence note:** verified against initial repository baseline on 2026-11-23. The code wins (see 16).

---

## 1. Architectural Constraints & Gaps

| Area | Limitation | Impact | Planned Redesign |
|---|---|---|---|
| **Cross-Repo Atomicity** | Git changes across the 4 independent repos cannot be committed in a single atomic Git transaction. | Risk of temporal inconsistency between API and clients during deployments. | Enforce strict forward-compatible schema evolution and lockstep PR checks. |
| **Spatial Aggregation Granularity** | 3-decimal truncation (~110m) loses sub-meter vehicle precision. | Necessary for DPA 2012 compliance, but precludes exact micro-location vehicle bay positioning. | Introduce localized transit stop beacon IDs rather than continuous GPS. |
| **Offline Sync Conflict Resolution** | Last-write-wins strategy on mobile sync. | Concurrent offline edits to route schedules can overwrite earlier updates. | Migrate to vector clocks or CRDT-based state reconciliation in v2.0. |

---

## 2. Deferred MVP Features

- **Automated Fine/Sanction Dispatch:** Automatic submission of violations to regulatory bodies is deferred; MVP relies on manual review by `Administrator` in `lakbay-web`.
- **Public Transit Live Heatmaps:** Real-time spatial tile rendering deferred in favor of tabular density statistics.
- **Biometric Authentication:** Deferred in favor of standard Google OAuth2 PKCE.

---

## 3. Ranked Redesign Candidates

1. **Automated Schema Code Generation:** Replace manual TypeScript and Kotlin model maintenance with OpenAPI / Supabase schema codegen to eliminate manual lockstep maintenance.
2. **WebSocket Push Notifications:** Replace client polling with Supabase Realtime WebSocket subscriptions for instant regulatory alert broadcasts.
