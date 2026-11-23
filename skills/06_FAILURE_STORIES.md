# 06 — Failure Stories

**Purpose:** Documented post-mortems of architectural vulnerabilities, integration pitfalls, and critical scars across the Lakbay ecosystem. Read to understand why specific guards, architectural boundaries, and constraints were created.

**Confidence note:** verified against initial repository baseline on 2026-11-17. The code wins (see 16).

---

## Incident 1: Unmediated Client-to-Database Direct Writes

- **Confidence:** Verified architectural scar (reconstructed from `lakbay-api/AGENTS.md` and `lakbay-specs/business-logic-v1.0.md`).
- **What Happened:** In an early prototype design, the mobile client attempted to write transit events directly to Supabase via PostgREST using the anonymous key (`SUPABASE_ANON_KEY`).
- **Why It Happened:** The team sought to minimize backend development time by relying entirely on PostgREST and Row-Level Security (RLS) rules.
- **How It Was Found:** Security review demonstrated that while RLS could enforce row ownership, it could not enforce synchronous regulatory compliance audit logging or complex transactional privacy sanitization mandated by the Philippine Data Privacy Act of 2012.
- **The Fix:** Created `lakbay-api` as a mandatory reverse proxy and orchestrator. Direct write access to PostgreSQL tables from external clients was blocked, routing all write operations through API controllers that execute audit logging synchronously before persistence.
- **Lesson Learned:** *Never let convenience bypass regulatory mediation.* Client applications must never write directly to the database when legal compliance auditing is a statutory requirement (see [`01_ARCHITECTURE.md`](file:///c:/Users/nexus/Documents/lakbay-ecosystem/lakbay-specs/skills/01_ARCHITECTURE.md) & [`16_ENGINEERING_PRINCIPLES.md`](file:///c:/Users/nexus/Documents/lakbay-ecosystem/lakbay-specs/skills/16_ENGINEERING_PRINCIPLES.md) #3).

---

## Incident 2: High-Precision GPS Telemetry Regulatory Violation Risk

- **Confidence:** High (derived from `lakbay-specs/business-logic-v1.0.md`, Section 39: "location protection, retention limits, no per-user trails").
- **What Happened:** Simulated location reporting streams from Android devices recorded 6-decimal-place GPS coordinates (`~0.1m` precision) paired directly with user session identifiers.
- **Why It Happened:** Standard mobile location SDKs provide native double-precision latitude and longitude without applying privacy degradation.
- **How It Was Found:** Compliance drill identified that raw GPS points formed an identifiable breadcrumb trail, violating National Privacy Commission guidelines against non-consensual personal surveillance.
- **The Fix:** Implemented `privacyScrubber.ts` in `lakbay-api`. Coordinates are aggressively rounded to 3 decimal places (`~110m` neighborhood grid), detached from personal account IDs, and grouped into anonymous spatial aggregates before database insertion.
- **Lesson Learned:** *Privacy by design must be enforced at ingestion.* Downstream storage must never receive raw PII telemetry that could be subpoenaed or leaked in a data breach.

---

## Incident 3: Commit Format & AI Attribution Drift

- **Confidence:** Verified rule enforcement (derived from `lakbay-specs/git-guidelines.md`, Section 255).
- **What Happened:** Git histories in cross-cutting multi-repo projects frequently suffer from vague commit messages (e.g., `update`, `fix bug`, `mod`) and automated bot attribution tags (`Co-authored-by: AI Agent`).
- **Why It Happened:** Developers and AI tooling used disparate default templates, obscuring code changes and cluttering commit ownership across the 4 ecosystem repos.
- **How It Was Found:** Audit of commit histories revealed impossible-to-bisect bugs and non-standard version tags.
- **The Fix:** Adopted the strict `git-guidelines.md` commit standard requiring `scope: Imperative summary` (<= 50 chars), explicit exclusion of AI agents as commit authors, and ISO-week based release tagging.
- **Lesson Learned:** *Clear commit scopes preserve multi-repo sanity.* Without strict scope discipline, coordinating releases across 4 interdependent repositories becomes unmanageable.
