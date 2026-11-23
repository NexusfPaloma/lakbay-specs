# 14 — Developer Workflows

**Purpose:** Practical developer recipes, local environment initialization, cross-repo synchronization, commit authoring, and feature delivery procedures for Lakbay. Read when setting up your machine or starting a new task.

**Confidence note:** verified against initial repository baseline on 2026-11-23. The code wins (see 16).

---

## 1. Initial Local Setup

1. **Clone All Repositories:** Clone the four sibling repositories under `lakbay-ecosystem/`:
   ```bash
   git clone https://github.com/NexusfPaloma/lakbay-specs.git
   git clone https://github.com/NexusfPaloma/lakbay-api.git
   git clone https://github.com/NexusfPaloma/lakbay-web.git
   git clone https://github.com/NexusfPaloma/lakbay-android.git
   ```
2. **Configure Node.js Tooling:** Ensure Node.js (v20+ LTS) and `pnpm` are installed.
3. **Setup Android SDK:** Install Android Studio with SDK API 34+ and JDK 17.

---

## 2. Recipe: Adding a New Feature Across Repositories

1. **Step 1 — Define Specification (`lakbay-specs`):**
   - Document new functional behavior in `lakbay-specs/business-logic-v1.x.md`.
   - Update lockstep sets in `lakbay-specs/skills/04_CHANGE_RULES.md`.
2. **Step 2 — Implement & Test API Endpoint (`lakbay-api`):**
   - Add route, validation schema, and unit tests.
   - Run `pnpm test` and `pnpm typecheck`.
3. **Step 3 — Integrate Clients (`lakbay-web` and `lakbay-android`):**
   - Consume new endpoint in web and mobile services.
   - Test UI states (loading, success, error).
4. **Step 4 — Atomic Git Commits:**
   - Commit changes in each repo following `scope: Imperative summary` (see `git-guidelines.md`).
   - *Miss → Uncoordinated PR merges break integration branch.*

---

## 3. Recipe: Cutting Weekly Releases

1. Verify local time is **Friday 5:00 PM or later**.
2. Determine current ISO week and ensure previous week has no untagged commits.
3. Apply annotated tags across repositories:
   - `lakbay-specs`: `git tag -a SCN.01.0.001 -m "Weekly release 001"`
   - `lakbay-api`: `git tag -a BCN.01.0.001 -m "Weekly release 001"`
   - `lakbay-web`: `git tag -a WCN.01.0.001 -m "Weekly release 001"`
   - `lakbay-android`: `git tag -a ACN.01.0.001 -m "Weekly release 001"`

*Found something confusing or surprising during setup? Add an entry to [`INBOX.md`](file:///c:/Users/nexus/Documents/lakbay-ecosystem/lakbay-specs/skills/INBOX.md).*
