# 00 — Index

**Purpose:** Master directory, reading order by engineering task, document status, and contribution guidelines for the Lakbay skills library. Read this first when joining the project or starting a new cross-cutting workflow.

**Confidence note:** verified against initial repository baseline on 2026-11-17. The code wins (see 16).

---

## Reading Order by Task

| Task | Read Order | Why |
|---|---|---|
| **Onboarding to Lakbay** | `01_ARCHITECTURE` → `02_PROJECT_STRUCTURE` → `14_DEVELOPER_WORKFLOWS` → `16_ENGINEERING_PRINCIPLES` | Understand the 4-repo topology, boundaries, environment setup, and foundational principles. |
| **Adding or modifying an API route** | `08_API` → `04_CHANGE_RULES` → `07_DATABASE` → `09_SECURITY` | Prevent breaking client contracts, violating lockstep sets, or exposing unauthenticated endpoints. |
| **Database schema & migration changes** | `07_DATABASE` → `04_CHANGE_RULES` → `08_API` | Ensure RLS policies, identity invariants, and lockstep column definitions are maintained. |
| **Investigating an outage or bug** | `05_DEBUGGING_PLAYBOOK` → `06_FAILURE_STORIES` → `04_CHANGE_RULES` | Rapidly match symptoms to known causes, consult post-mortems, and verify lockstep integrity. |
| **Writing and reviewing PRs** | `03_CODE_STYLE` → `04_CHANGE_RULES` → `11_TESTING` → `*-specs/git-guidelines.md` | Follow commit formatting, style rules, mandatory tests, and AI author exclusion. |
| **Preparing a release / deployment** | `10_DEPLOYMENT` → `*-specs/git-guidelines.md` → `15_KNOWN_LIMITATIONS` | Ensure correct release order across repos, verify weekly ISO version tags, and review open risks. |
| **Performance tuning or optimization** | `12_PERFORMANCE` → `07_DATABASE` → `15_KNOWN_LIMITATIONS` | Avoid bypassing intentional regulatory throttles or hot-path safeguards. |
| **Evaluating third-party packages** | `13_DEPENDENCIES` → `09_SECURITY` | Adhere to minimal dependency policy and prevent client-side credential leakage. |

---

## Skills Document Inventory

| Document | Topic | Description | Last Verified | Status |
|---|---|---|---|---|
| [`00_INDEX.md`](file:///c:/Users/nexus/Documents/lakbay-ecosystem/lakbay-specs/skills/00_INDEX.md) | Index & Directory | Reading order, verification status, and change index | 2026-11-17 | Active |
| [`INBOX.md`](file:///c:/Users/nexus/Documents/lakbay-ecosystem/lakbay-specs/skills/INBOX.md) | Contribution Inbox | Staging area for new findings, scars, and field notes | 2026-11-17 | Active |
| [`01_ARCHITECTURE.md`](file:///c:/Users/nexus/Documents/lakbay-ecosystem/lakbay-specs/skills/01_ARCHITECTURE.md) | Architecture | End-to-end model, boundaries, decisions, and debt register | 2026-11-17 | Active |
| [`02_PROJECT_STRUCTURE.md`](file:///c:/Users/nexus/Documents/lakbay-ecosystem/lakbay-specs/skills/02_PROJECT_STRUCTURE.md) | Project Structure | 4-repo directory trees, placement rules, and naming | 2026-11-17 | Active |
| [`03_CODE_STYLE.md`](file:///c:/Users/nexus/Documents/lakbay-ecosystem/lakbay-specs/skills/03_CODE_STYLE.md) | Code Style | Comment philosophy, naming, logging, and anti-patterns | 2026-11-17 | Active |
| [`04_CHANGE_RULES.md`](file:///c:/Users/nexus/Documents/lakbay-ecosystem/lakbay-specs/skills/04_CHANGE_RULES.md) | Change Rules | Lockstep sets, cross-repo synchronization, and blast radiuses | 2026-11-17 | Active |
| [`05_DEBUGGING_PLAYBOOK.md`](file:///c:/Users/nexus/Documents/lakbay-ecosystem/lakbay-specs/skills/05_DEBUGGING_PLAYBOOK.md) | Debugging Playbook | Symptom catalog, log sources, and diagnostic commands | 2026-11-17 | Active |
| [`06_FAILURE_STORIES.md`](file:///c:/Users/nexus/Documents/lakbay-ecosystem/lakbay-specs/skills/06_FAILURE_STORIES.md) | Failure Stories | Architectural incident reconstructions and lessons learned | 2026-11-17 | Active |
| [`07_DATABASE.md`](file:///c:/Users/nexus/Documents/lakbay-ecosystem/lakbay-specs/skills/07_DATABASE.md) | Database | Supabase/PostgreSQL schema, RLS policies, migrations | 2026-11-17 | Active |
| [`08_API.md`](file:///c:/Users/nexus/Documents/lakbay-ecosystem/lakbay-specs/skills/08_API.md) | API | Standard response envelopes, routes, auth, and validation | 2026-11-17 | Active |
| [`09_SECURITY.md`](file:///c:/Users/nexus/Documents/lakbay-ecosystem/lakbay-specs/skills/09_SECURITY.md) | Security | Threat model, Data Privacy Act compliance, secrets, roles | 2026-11-17 | Active |
| [`10_DEPLOYMENT.md`](file:///c:/Users/nexus/Documents/lakbay-ecosystem/lakbay-specs/skills/10_DEPLOYMENT.md) | Deployment | Release staging, maturity branching, and release checklists | 2026-11-17 | Active |
| [`11_TESTING.md`](file:///c:/Users/nexus/Documents/lakbay-ecosystem/lakbay-specs/skills/11_TESTING.md) | Testing | Test pyramids, contract validation, and manual drills | 2026-11-17 | Active |
| [`12_PERFORMANCE.md`](file:///c:/Users/nexus/Documents/lakbay-ecosystem/lakbay-specs/skills/12_PERFORMANCE.md) | Performance | Hard limits, throttling, hot paths, and deliberate delays | 2026-11-17 | Active |
| [`13_DEPENDENCIES.md`](file:///c:/Users/nexus/Documents/lakbay-ecosystem/lakbay-specs/skills/13_DEPENDENCIES.md) | Dependencies | Approved packages, upgrade traps, and deliberate rejections | 2026-11-17 | Active |
| [`14_DEVELOPER_WORKFLOWS.md`](file:///c:/Users/nexus/Documents/lakbay-ecosystem/lakbay-specs/skills/14_DEVELOPER_WORKFLOWS.md) | Developer Workflows | Step-by-step developer procedures and setup recipes | 2026-11-17 | Active |
| [`15_KNOWN_LIMITATIONS.md`](file:///c:/Users/nexus/Documents/lakbay-ecosystem/lakbay-specs/skills/15_KNOWN_LIMITATIONS.md) | Known Limitations | Technical debt, deferred features, and redesign targets | 2026-11-17 | Active |
| [`16_ENGINEERING_PRINCIPLES.md`](file:///c:/Users/nexus/Documents/lakbay-ecosystem/lakbay-specs/skills/16_ENGINEERING_PRINCIPLES.md) | Principles | Numbered core tenets and golden rules across Lakbay | 2026-11-17 | Active |

---

## How to Contribute

1. When you encounter a bug, trap, surprising constraint, or architectural decision, file an entry in [`INBOX.md`](file:///c:/Users/nexus/Documents/lakbay-ecosystem/lakbay-specs/skills/INBOX.md).
2. During regular refreshes, verified entries from `INBOX.md` are integrated into their respective home documents (see the routing table in `INBOX.md`).
3. If an existing rule is wrong or outdated, update it in the same PR that updates the code. Never quietly drop teammate contributions.

---

## Changelog

- **2026-11-17:** Initial synthesis of the complete 18-document skills library for the Lakbay multi-repo ecosystem (`lakbay-specs`, `lakbay-api`, `lakbay-android`, `lakbay-web`). Verified against initial baseline.
