# 12 — Performance

**Purpose:** Performance constraints, hot paths, rate limiting, deliberate throttling for compliance, and optimization guidelines across Lakbay services. Read before optimizing code or configuring network boundaries.

**Confidence note:** verified against initial repository baseline on 2026-11-23. The code wins (see 16).

---

## 1. Hard Limits & Capacity Guidelines

| Metric | Target Limit | Enforced At | Consequence of Exceeding |
|---|---|---|---|
| **API Response Time (P95)** | < 150ms | `lakbay-api` | Degraded mobile commuter experience on high-latency cellular networks |
| **Telemetry Ingestion Rate** | Max 1 req / 15s per client | `lakbay-api` rate limiter | Returns `429 Too Many Requests`; prevents denial-of-service from runaway clients |
| **Spatial Grid Resolution** | ~110m (3 decimal places) | Ingestion filter | Enforces Data Privacy Act compliance |
| **Database Connection Pool** | Max 20 connections | Supabase PgBouncer | Prevents database connection exhaustion during peak transit hours |

---

## 2. Hot Paths vs Deliberate Inefficiencies

### Hot Paths (Optimize Aggressively)
- **Token Verification:** Cache validated JWT claims in memory for 60 seconds in `lakbay-api` to avoid redundant cryptographic public-key lookups.
- **Aggregate Tile Queries:** Pre-calculate transit density buckets on a 5-minute cron worker so client queries read pre-computed rows rather than running live spatial aggregations.

### Deliberate Inefficiencies (Do Not Bypass)
- **Synchronous Compliance Logging:** Writing to `compliance_audit_logs` is deliberately executed synchronously on administrative actions.
  - *Reason:* An administrative action must never commit if its audit entry fails to persist. Bypassing this for performance violates regulatory auditability.
- **Mobile Location Throttling:** Android client GPS polling is throttled to 15-second intervals to preserve commuter battery life.

---

## 3. Profiling & Diagnostics

- **Database Slow Queries:** Check `pg_stat_statements` on Supabase to identify unindexed lookups.
- **API Bottlenecks:** Use Fastify built-in metrics and Node.js clinic.js profiler to inspect event loop lag.
