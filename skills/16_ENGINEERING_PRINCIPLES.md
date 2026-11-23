# 16 — Engineering Principles

**Purpose:** Foundational engineering principles and immutable tenets governing all development across the Lakbay ecosystem. Read to align decisions when specifications or rules are silent.

**Confidence note:** verified against initial repository baseline on 2026-11-23. The code wins.

---

## Principle 1: Privacy by Design is Non-Negotiable
- **Reasoning:** User trust and statutory compliance (Philippine Data Privacy Act of 2012) are existential requirements. A transit platform that tracks individual movements becomes a surveillance liability.
- **Evidence:** `lakbay-specs/business-logic-v1.0.md` Required Solution Layers.
- **Rule:** Never store, log, or transmit raw, high-precision personal GPS breadcrumbs. Aggregate early, anonymize at ingestion, and enforce retention limits.

---

## Principle 2: The Specifications Repository is the Single Source of Truth
- **Reasoning:** In a distributed multi-repo system, drift between backend, web, and mobile clients creates silent runtime failures.
- **Evidence:** [`04_CHANGE_RULES.md`](file:///c:/Users/nexus/Documents/lakbay-ecosystem/lakbay-specs/skills/04_CHANGE_RULES.md) Lockstep Sets.
- **Rule:** No feature, route, or role exists until it is defined in `lakbay-specs`. Code must mirror the specs.

---

## Principle 3: Never Bypass the API Gateway for Regulatory Operations
- **Reasoning:** Direct client-to-database communication bypasses synchronous compliance auditing and input sanitization.
- **Evidence:** [`06_FAILURE_STORIES.md`](file:///c:/Users/nexus/Documents/lakbay-ecosystem/lakbay-specs/skills/06_FAILURE_STORIES.md) Incident 1.
- **Rule:** All regulatory mutations and state transitions must pass through `lakbay-api` validation middleware.

---

## Principle 4: Lockstep Contracts Break Silently at Runtime
- **Reasoning:** Compilers in independent repositories cannot detect missing fields or changed enums in peer repos.
- **Evidence:** [`04_CHANGE_RULES.md`](file:///c:/Users/nexus/Documents/lakbay-ecosystem/lakbay-specs/skills/04_CHANGE_RULES.md).
- **Rule:** Whenever a shared contract, role, or envelope changes, all peer repos must be updated and verified before promotion.

---

## Principle 5: Atomic Promotion Across Repositories
- **Reasoning:** Promoting a client before the backend supports its endpoints breaks active users in production.
- **Evidence:** [`10_DEPLOYMENT.md`](file:///c:/Users/nexus/Documents/lakbay-ecosystem/lakbay-specs/skills/10_DEPLOYMENT.md).
- **Rule:** Promote changes in sequence: Specs → Database → API → Clients.

---

## Principle 6: Clean Git Attribution — No AI Authorship
- **Reasoning:** Legal, academic, and professional accountability requires accurate human commit provenance.
- **Evidence:** `lakbay-specs/git-guidelines.md` line 257.
- **Rule:** AI agents must never be authored or co-authored on Git commits, tags, or pull requests.

---

## The Golden Rule

> **Trust the code over any document, including this one.**
> When this document disagrees with the production code or verified test suite, the code wins. Update the document immediately to eliminate the lie.
