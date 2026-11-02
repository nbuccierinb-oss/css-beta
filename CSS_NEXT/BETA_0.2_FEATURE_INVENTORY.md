# CSS Beta 0.2 — Feature Inventory & Migration Register

**Audit date:** November 2, 2026  
**Source branch:** `main`  
**Source file:** `index.html`  
**CSS NEXT working branch:** `css-next`  
**Purpose:** Record what exists, what is only a prototype, what needs recovery, and what CSS NEXT should do with each capability.

> This is a static code inventory. It is not a live browser regression test. “Present” means represented in the reviewed source; it does not mean production-ready or fully tested.

## Status key

- **Preserve / rebuild:** valuable behavior to carry into the new architecture.
- **Recover:** known historical functionality not found in current `main`.
- **Prototype only:** concept/UI exists, but backend, security, or operational integration is missing.
- **New module:** planned for CSS NEXT; not found in the reviewed current source.
- **Defer:** requires policy, legal, privacy, technical, or vendor review before implementation.

## 1. Current Beta 0.2 feature inventory

| Capability | Evidence in current `main` | Current maturity / limitation | CSS NEXT decision |
|---|---|---|---|
| Officer dashboard | Dashboard markup and count update functions | Demo dashboard; counters derive from local browser data | Preserve layout concepts; rebuild against authorized server data |
| Caseload / client cards | Three sample cards, search, recent activity ordering | Demo client list; identity is based on display names in several places | Rebuild around stable client and case IDs |
| Add client | Modal, form, local save/restore | Explicitly labeled demo-only; data saved in browser; user fields rendered without consistent escaping | Preserve workflow idea; rebuild with validation and safe rendering |
| Client phases | Phase 1–3 labels and display | Labels exist; no complete configurable phase/transition engine identified | Rebuild as agency-configurable workflow |
| Goals | Create, render, view details, update progress | Stored in localStorage under `cssGoals`; keyed by client name; not a multi-user record system | Preserve user flow; use stable IDs, history, permissions, and durable storage |
| Goal-to-timeline event | New goal adds a temporary timeline item | Temporary in-memory logic; does not create a durable timeline record | Rebuild using core TimelineEvent records |
| Client timeline | Static timeline markup and dynamic demo entries | Dynamic entries are temporary; duplicate/nested rendering implementations complicate behavior | Rebuild as durable, filterable, auditable event history |
| Client calendar | Month view; office, urine, court, home, court-report event types | Browser-local storage keyed by client name; no server sync, permissions, or reliable notification service | Preserve calendar behavior; rebuild with stable IDs and configurable event types |
| Multiple calendar events per day | Current code supports event arrays, add/edit/remove, legacy normalization | Present in latest October 27 commits; local-only persistence | Preserve and test as a core workflow |
| Court Report checklist | Not found in current `main` | Six-step interactive checklist and client/report-specific save/restore were present in September 17 historical source | **Recover** the B8 behavior and re-test before migration |
| Outlook calendar handoff | Generates Outlook compose/deep link | Handoff only; does not confirm event was saved or sync changes; temporary test button uses a fixed historical date | Preserve as optional integration; remove test-only UI and add honest status |
| Client App | Simulated full-screen client-facing view | Not a separately authenticated client app; uses browser state and demo activity | Rebuild as a separate role-based experience |
| Client activity tracking | Session and activity history in localStorage | Local browser activity only; not a reliable server-side usage record | Rebuild only where there is a defined service or supervision need |
| Learning Center | 12 pillars, 3 learning levels, guided questions, reflections | Progress stored locally; content is a useful prototype, not an assessment or clinical instrument | Preserve content as a starting point; review language, accessibility, and outcomes |
| Probation Dictionary | Searchable terms and categories | Static content in source; no agency content-management workflow identified | Preserve and make content configurable |
| CSS Messenger | Messenger UI and canned message actions | Demo interface; no actual secure message delivery, identity, storage, or delivery receipts | Rebuild only with secure messaging requirements and policies |
| Contact recording / case notes | Contact modal, generated case-note text, copy action | UI workflow exists; no production case-note storage/audit model identified in this file | Preserve workflow; connect to secure notes service |
| Dashboard counts | Total caseload, progress, attention, critical barriers | Critical barrier type mismatch: client record stores Boolean, dashboard compares to string `"Yes"` | Fix through typed model and tests in new implementation |

## 2. Historical behavior to recover

### Court Report checklist — B8

Historical source reviewed: commit `9ba2eaa62b6222cc15b8a094511679108e4cbc50` (September 17, 2026).

The historical checklist included:
1. Review case plan
2. Review goals & progress
3. Review barriers
4. Review compliance
5. Draft court report
6. Submit / route report

The historical implementation saved checklist progress under `cssCourtReportChecklists`, keyed by client display name and report date, and restored it when the same client/report was opened.

**Important:** This is historical prototype behavior, not a verified feature in the November 2 `main` snapshot. The prior project checkpoint recorded that B8 passed live testing, including isolation between two demo clients. CSS NEXT must preserve the intended behavior and improve its identity model by using stable client, case, and report IDs.

### Other historical knowledge

The project checkpoint records that the duplicate-Sarah client selection issue was fixed and verified in September. The current source still contains client-card selection logic, but a fresh live browser regression test is required before relying on that old test result.

## 3. Known source-level issues

1. **Browser-local persistence:** goals, calendar events, demo clients, learning progress, and activity are stored in localStorage. This is not a shared, secure, multi-user system of record.
2. **Name-based relationships:** several records use client names as keys. Display names can change or collide.
3. **Temporary timeline events:** newly added goal timeline items are not durably stored.
4. **Critical barrier type mismatch:** Boolean is written, string is compared.
5. **Duplicate function declarations:** `openLearnMode` and `closeLearnMode` are declared twice at top level; later definitions override earlier ones.
6. **Timeline rendering duplication:** a temporary timeline renderer is nested inside `saveGoal`, while another renderer exists globally.
7. **Unsafe HTML interpolation:** some demo-client fields are inserted into `innerHTML` without consistent escaping.
8. **Inconsistent localStorage error handling:** not every read/write is protected against malformed or unavailable browser storage.
9. **No server integration identified:** the reviewed source contains no fetch/XHR/WebSocket/IndexedDB/API client; the file relies on local browser storage.
10. **Prototype status is explicit:** the add-client UI warns users not to enter identifying or confidential information.

## 4. New CSS NEXT modules not present in current source

| Planned capability | Status | Required discovery/design before build |
|---|---|---|
| Agency configuration | New module | Jurisdiction, roles, workflows, enabled modules, language, retention, notifications |
| Authentication and authorization | New foundation | Staff/client identity, agency tenancy, assignment rules, session security, MFA policy |
| Resource Connections | New/expand | Verified resource directory, referral workflow, consent, referral follow-up |
| Client Bulletin | New module | Staff publishing, categories, dates, expiration/archive, client view, printable/downloadable PDF, optional notifications |
| Words That Move Me Forward | New module | Private-by-default writing, client-controlled sharing, officer visibility policy, legal privacy disclosures |
| SAFEZONE AWARE | Defer pending feasibility | Legal authority, device limitations, false-alert handling, client notice, human review |
| SAFEZONE MONITOR | Defer pending feasibility | Court authorization, approved provider integration, response protocol, audit, reliability and security |

## 5. Migration priority

### Foundation first
- Agency, user, role, client, case, assignment, and stable ID model.
- Authentication and server-side authorization.
- Database migrations, audit events, backups, and API boundaries.
- Synthetic test fixtures and automated test harness.

### First workflow migration
- Client/case profile and assignments.
- Goals, progress history, and durable timeline.
- Calendar and reminders.
- Recover Court Report checklist with client/report isolation.
- Contact notes and case-note workflow.

### Second workflow migration
- Client-facing calendar and permitted client app views.
- Learning Center and Probation Dictionary.
- Resource Connections and Client Bulletin.
- Secure communications only after delivery, retention, and permission requirements are settled.

### Separate feasibility tracks
- Words That Move Me Forward privacy/consent policy.
- SAFEZONE legal/technical/vendor assessment.

## 6. Minimum acceptance tests for migrated features

- A record created for Client A never appears under Client B, even if names are similar.
- Two clients with the same display name remain distinct because their IDs differ.
- A user cannot read or modify records outside their agency or permissions by changing a URL or client identifier.
- Goal creation, progress updates, and timeline events survive logout/login and device changes.
- Calendar events persist and are visible only to authorized roles.
- Court Report checklist state restores for the correct client, case, and report, and never leaks to another record.
- Client-visible content is limited to explicitly authorized fields.
- Every sensitive change records actor, time, action, and affected record.
- Malformed or unavailable storage/API responses produce recoverable errors rather than silent data loss.
- User-supplied content is safely rendered and validated.
- Automated tests cover core workflows; staff acceptance testing is recorded before release.

## 7. Audit boundary

This inventory is based on:
- Current `main` source snapshot reviewed November 2, 2026.
- Historical September 17 source snapshot for the Court Report checklist.
- Existing project checkpoint notes for earlier B8 and client-selection testing.

No browser interaction, database inspection, security penetration test, accessibility audit, or production data migration was performed as part of this source review.
