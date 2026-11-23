# 07 — Database

**Purpose:** Schema design, primary key identity strategy, Row-Level Security (RLS) policies, integrity invariants, soft-delete rules, and migration guidelines for the Supabase (PostgreSQL) backend. Read before writing any SQL migration or altering data relationships.

**Confidence note:** verified against initial repository baseline on 2026-11-17. The code wins (see 16).

---

## 1. Identity & Primary Key Strategy

- **Primary Keys:** Every table in Lakbay must use `id UUID PRIMARY KEY DEFAULT gen_random_uuid()`.
  - *Miss → Sequential integer IDs expose business metrics (user growth rate, order volume) to enumeration attacks.*
- **User References:** Foreign keys referencing system actors must link directly to `auth.users(id)` with `ON DELETE CASCADE` or `ON DELETE RESTRICT` depending on regulatory audit retention requirements.
- **Timestamps:** Every table must include `created_at TIMESTAMPTZ NOT NULL DEFAULT timezone('utc'::text, now())` and `updated_at TIMESTAMPTZ NOT NULL DEFAULT timezone('utc'::text, now())`.

---

## 2. Table Map & Schema Model

```
+--------------------+       1:N       +------------------------+
|    auth.users      | --------------> |      user_profiles     |
| (Supabase Managed) |                 | (id, role, full_name)  |
+--------------------+                 +------------------------+
                                                    |
                                                    | 1:N
                                                    v
+-----------------------------+       1:N       +------------------------+
|     transit_aggregates      | <-------------- | compliance_audit_logs  |
| (grid_id, density, epoch)   |                 | (id, actor_id, action) |
+-----------------------------+                 +------------------------+
```

| Table Name | Purpose | Retention Policy | Primary Constraints |
|---|---|---|---|
| `user_profiles` | Extends `auth.users` with Lakbay user levels (`System Administrator`, `Administrator`, `Moderator`, `Contributor`, `Guest`). | Active while account exists; soft-deleted upon request. | `id` PK (FK `auth.users.id`), `role` NOT NULL. |
| `compliance_audit_logs` | Immutable audit trail recording administrative and regulatory actions. | Retained for 5 years per statutory regulatory mandates. | `id` PK, `actor_id` NOT NULL, `action` NOT NULL. Append-only. |
| `transit_aggregates` | Spatially anonymized commuter density records. | Auto-purged after 90 days. | `grid_id` NOT NULL, `time_bucket` NOT NULL. UNIQUE(`grid_id`, `time_bucket`). |

---

## 3. Row-Level Security (RLS) & Integrity Invariants

1. **RLS Must Be Enabled on Every Table:**
   ```sql
   ALTER TABLE user_profiles ENABLE ROW LEVEL SECURITY;
   ALTER TABLE compliance_audit_logs ENABLE ROW LEVEL SECURITY;
   ALTER TABLE transit_aggregates ENABLE ROW LEVEL SECURITY;
   ```
   - *Miss → Any authenticated client with a valid Supabase token can read or overwrite all ecosystem data.*
2. **Profiles Policy:** Users can only view their own profile or public contributor profiles:
   ```sql
   CREATE POLICY "Users view own profile" ON user_profiles
     FOR SELECT USING (auth.uid() = id);
   ```
3. **Audit Logs Immutability:** No `UPDATE` or `DELETE` policies are permitted on `compliance_audit_logs`:
   ```sql
   -- Only INSERT and SELECT policies are created. Never grant UPDATE/DELETE permissions.
   ```
   - *Miss → Tampering with audit logs compromises regulatory compliance integrity.*

---

## 4. Deletes & Retention Rules (Data Privacy Act of 2012)

- **Soft Deletion by Default:** Entities containing user data must include `deleted_at TIMESTAMPTZ NULL`. Queries must filter `WHERE deleted_at IS NULL`.
- **Hard Deletion / Anonymization:** When a user invokes their right to erasure under the Data Privacy Act, `auth.users` record is deleted, `user_profiles.full_name` is scrubbed to `"Anonymized User"`, but associated `compliance_audit_logs` remain intact with `actor_id` retained for legal auditing.

---

## 5. Migration Strategy & Dangerous Operations

- **Tooling:** All schema changes must be authored as SQL migration files managed by Supabase CLI (`supabase/migrations/<timestamp>_<name>.sql`).
- **Forbidden Operations:**
  - `DROP COLUMN` without prior 2-week deprecation cycle across `lakbay-api` and client serializers.
  - Adding a `NOT NULL` column without a `DEFAULT` value on tables with existing rows (causes exclusive table lock and downtime).
  - Executing ad-hoc DDL queries via the production Supabase dashboard UI without version-controlled migration files.
- **Indexing Rules:**
  - Foreign keys must always have a supporting B-tree index (`CREATE INDEX idx_audit_actor ON compliance_audit_logs(actor_id)`).
  - Spatial and time-series queries on `transit_aggregates` must use composite indexes: `(grid_id, time_bucket DESC)`.
