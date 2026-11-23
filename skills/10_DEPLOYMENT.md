# 10 — Deployment

**Purpose:** Deployment sequence, maturity-based branching promotion, environment staging, release artifact distribution, and weekly tagging procedures across Lakbay repositories. Read before cutting releases or triggering deployments.

**Confidence note:** verified against initial repository baseline on 2026-11-23. The code wins (see 16).

---

## 1. Cross-Repository Release Order

Deployments must follow this sequential ordering to avoid runtime contract incompatibilities:

1. **`lakbay-specs`:** Spec updates, database migration scripts, and contract definitions are reviewed and merged first.
2. **Supabase Database:** Forward migrations applied to staging/production PostgreSQL.
   - *Miss → Deploying API before DB migration leads to missing table/column errors.*
3. **`lakbay-api`:** Server runtime deployed and health checked at `/v1/health`.
   - *Miss → Client features pointing to non-existent API routes fail immediately.*
4. **`lakbay-web`:** Static web assets deployed to CDN/hosting provider (Vercel/Cloudflare).
5. **`lakbay-android`:** Release APK/AAB compiled and uploaded to Google Play internal/closed testing track.

---

## 2. Maturity-Based Branching Model

Per `lakbay-specs/git-guidelines.md`:
```
main (Development) ──> beta (QA / System Testing) ──> release (Integration / RC) ──> Production
```

- **`main`:** Daily active development and developer unit tests.
- **`beta`:** QA and system testing against functional claims.
- **`release`:** Controlled release testing and release candidate iterations (`a`, `b`, `rc`).
- **Production:** Approved exact release candidate deployed to live environments.

---

## 3. Release Tags & Weekly Cadence

- **Tag Format:** `<PREFIX>.0X.Y.00Z`
  - `lakbay-web`: `WCN.01.0.001` (or `WLB.01.0.001`)
  - `lakbay-api`: `BCN.01.0.001` (or `BLB.01.0.001`)
  - `lakbay-android`: `ACN.01.0.001` (or `ALB.01.0.001`)
  - `lakbay-specs`: `SCN.01.0.001` (or `SLB.01.0.001`)
- **Rules:**
  - Cut on Friday at 5:00 PM or later on local clock.
  - Never tag an ongoing week early.
  - Increments weekly based on ISO week of repo's first commit.
  - Candidate suffixes: `a`, `a2`, `b`, `rc`, `rc2`.

---

## 4. Pre-Deployment Verification Checklist

- [ ] All database migrations are backward compatible.
- [ ] API integration tests pass against target database.
- [ ] Mobile app bundle is signed with release keystore.
- [ ] Zero unredacted debug logging in production builds.
- [ ] Commit headers follow `scope: Imperative summary` standard.
