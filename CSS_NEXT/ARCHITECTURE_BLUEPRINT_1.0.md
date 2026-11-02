# PROJECT CSS NEXT — Architecture Blueprint 1.0

**Status:** Working blueprint / draft for review  
**Date:** November 2, 2026  
**Repository:** `nbuccierinb-oss/css-beta`  
**Development branch:** `css-next`  
**Production/default branch:** `main` — protected by process; do not develop CSS NEXT directly on this branch.

---

## 1. Product vision

Build a modular, configurable platform that helps justice organizations support people through accountability, opportunity, rehabilitation, restorative practices, and measurable personal growth.

DeKalb County Court Services is the proving ground, not the ceiling. The long-term product should be adaptable to adult probation, juvenile justice, specialty courts, drug/recovery courts, pretrial services, community corrections, reentry programs, restorative justice organizations, and other justice-serving agencies.

### Mission

> To transform the way justice organizations support individuals by connecting accountability, opportunity, rehabilitation, restorative practices, and measurable personal growth.

### Guiding principles

1. **Accountability without humiliation.**
2. **Structure without unnecessary barriers.**
3. **Progress measured by personal growth, not just compliance.**

### Primary measure of success

Lives impacted. Product activity (logins, messages, completed forms, alerts) is not the same as outcomes. Where feasible, evaluate barriers addressed, resources accessed, goals achieved, sustained engagement, and progress reported by clients and staff.

---

## 2. Architecture strategy

Use a modular, agency-independent platform with four layers:

1. **CSS Core** — shared identity, case, workflow, data, permissions, security, and audit capabilities.
2. **Feature Modules** — focused capabilities that can be enabled, configured, or withheld by agency.
3. **Agency Configuration** — jurisdiction-specific policies, workflows, language, roles, retention, and notification rules.
4. **Role-Based Experiences** — Officer Workspace, Client Experience, Supervisor Workspace, and Agency Administration.

The application should not encode DeKalb County assumptions into the core. Agency-specific behavior belongs in configuration or explicitly scoped modules.

### Conceptual architecture

```text
ROLE-BASED EXPERIENCES
  Officer Workspace | Client Experience | Supervisor | Agency Admin
                              |
FEATURE MODULES
  Goals & Phases | Calendar | Court Reports | Resources
  Learning Center | Restorative Reflection | Communications
  Client Bulletin | SAFEZONE (separate feasibility track)
                              |
CSS CORE
  Identity | Agencies | Users | Clients | Cases | Assignments
  Goals | Timeline | Notes | Events | Notifications
  Permissions | Consent | Audit | Reporting
                              |
PLATFORM SERVICES
  Secure API | Database | Authentication | Authorization
  Encryption | Logging | Backups | Monitoring | Integrations
```

This is a logical architecture, not a commitment to a specific programming framework or hosting provider. Technology selection follows requirements, security review, and a small technical proof of concept.

---

## 3. CSS Core — foundational capabilities

The core should own the concepts and rules shared across modules:

- **Agency and jurisdiction:** agency identity, jurisdiction, configuration, enabled modules, policy settings.
- **Identity and access:** users, authentication, role assignments, account lifecycle, multi-factor authentication where appropriate.
- **Client and case records:** stable client IDs, case IDs, agency ownership, case status, assigned staff, dates, and permitted relationships.
- **Goals and progress:** goals, action steps, barriers, commitments, status changes, and history.
- **Phases:** agency-configurable phase names, criteria, transitions, and staff permissions.
- **Timeline:** durable, append-oriented record of meaningful case events with source, actor, timestamp, and visibility rules.
- **Notes and contacts:** structured contact records and case notes with authorship, timestamps, edit history, and permissions.
- **Calendar and tasks:** typed events, reminders, due dates, and links to the relevant client/case.
- **Notifications:** in-app and approved external channels, delivery status where available, acknowledgments, and escalation history.
- **Reporting:** operational and outcome reporting with agency-level access controls.
- **Audit:** record access and material changes to sensitive records and configuration.

### Stable identity rule

Use immutable IDs (UUIDs or equivalent) for agencies, users, clients, cases, goals, reports, calendar events, timeline entries, and other records. Names are display values, not database keys. This prevents collisions when two clients share a name and supports name changes without breaking relationships.

### Multi-agency rule

Every tenant-owned record must be scoped to an agency, and every request must be authorized against that agency boundary. Cross-agency access is denied by default and only permitted through an explicit, documented sharing arrangement.

---

## 4. Feature modules

Modules use core services and must not create separate, inconsistent copies of client identity or case history.

| Module | Purpose | Initial direction |
|---|---|---|
| Goals & Phases | Build and track practical client goals and progress | Core-adjacent; migrate proven behavior |
| Client Calendar | Appointments, urine testing, court dates, home contacts, reminders | Rebuild on stable IDs and configurable event types |
| Court Reports | Prepare and track court-report workflow | Recover the proven six-step checklist |
| Resource Connections | Connect clients to housing, jobs, food, transportation, and support | Build around verified resources and referral follow-through |
| Learning Center | Guided skill-building and practical learning | Preserve the 12-pillar prototype as a content starting point |
| Probation Dictionary | Plain-language explanations of justice-system terms | Preserve content; make agency-configurable |
| Communications | Secure, permissioned staff/client communications | Messenger prototype is not a production messaging system |
| Client Bulletin | Publish monthly resource bulletins by category, with publication and expiration/archive controls | Candidate pilot: DeKalb County Client Resource Bulletin |
| Words That Move Me Forward | Voluntary guided reflection and writing | Privacy-first; private by default |
| SAFEZONE AWARE / MONITOR | Location-based awareness or legally authorized monitoring | Separate legal, technical, privacy, and provider feasibility track |

### Words That Move Me Forward

Working subtitle: **Reflection. Accountability. Healing. Growth.**

Potential writing paths include self, future self, someone harmed, family, addiction, and free writing. Include the reflection prompt: “What did writing this letter help you understand about yourself?”

Requirements:
- Voluntary, never a mandatory assignment by default.
- Private and unshared by default.
- Client-controlled sharing with clear, understandable permissions.
- Do not expose letter contents to a probation officer/caseworker through a general share control. The agency must separately decide whether staff see completion metadata only or have any narrowly authorized access.
- Clearly disclose legal limits to privacy; do not promise absolute confidentiality.
- Victim-directed letters remain private and unsent by default. CSS must not automatically deliver them or encourage contact contrary to court orders, safety plans, or victim preferences.
- Consider a client-controlled revisit/reflection at a later date (for example, six months) without treating writing as a clinical assessment.

**Open policy decision:** define exactly what the assigned officer/caseworker can see (no access, completion metadata only, or a narrowly governed exception). This must be resolved before implementation.

### SAFEZONE

Two distinct modes:

- **SAFEZONE AWARE:** supplemental client awareness warnings and permitted system-health/event information. Not a substitute for court-ordered electronic monitoring.
- **SAFEZONE MONITOR:** only for a legally authorized monitoring arrangement, with approved monitoring-provider integration preferred where formal monitoring is required.

Non-negotiable safeguards:
- Separate court-ordered exclusion distance from any earlier warning buffer.
- Do not reveal protected addresses or victim identities unnecessarily.
- Phone GPS is imperfect; device permissions, battery, signal, background limits, and a phone left behind can create gaps or false alerts.
- A proximity event is not proof of entry, intent, a violation, or a finding of fact.
- No automatic violation finding, sanction, or adverse case decision based solely on a phone alert.
- Preserve event context (accuracy, timestamp, duration, movement estimate, reporting gaps, warning delivery) for human review.
- Establish legal authority, client notice, retention, access controls, audit, cybersecurity, and operational response before any pilot.

#### SAFEZONE notification concept

The initial concept is 24/7 notifications to the assigned officer and designated supervisor, with work email/Outlook as a primary channel where approved. Email content should be minimal and direct staff to authenticated CSS for details. Delivery is not proof of acknowledgment or response. Critical events require a separate agency emergency protocol; CSS notifications do not replace 911 or guarantee intervention.

Possible acknowledgment/escalation targets (for agency review, not adopted policy):
- Critical: immediate notification and emergency protocol.
- High: immediate notification; proposed escalation after 15 minutes without acknowledgment.
- Medium: immediate notification; proposed escalation after 60 minutes.
- System interruption: immediate notification; proposed escalation after 60 minutes.

Agencies must configure priorities, on-call rosters, escalation windows, and response responsibilities to match law, staffing, and policy. Acknowledgment means a person has accepted responsibility for review; it does not mean the event is resolved.

---

## 5. Role-based experiences

### Officer Workspace
Caseload, client/case overview, goals, phases, timeline, notes, calendar, court-report workflow, referrals, and authorized client-app navigation.

### Client Experience
A simple, mobile-first experience for permitted calendar items, reminders, goals, resource bulletins, learning content, and voluntary reflection. Show only information the client is authorized to see.

### Supervisor Workspace
Caseload oversight, workload and workflow visibility, permitted alerts, escalation acknowledgment, quality review, and agency-defined reporting.

### Agency Administration
Agency configuration, roles and permissions, module enablement, workflow settings, notification policies, retention settings, and audit access.

Role names and permissions must be configurable. Never rely on hiding a button in the interface as the security control; authorization must be enforced by the server for every protected operation.

---

## 6. Data, privacy, and security requirements

CSS may handle highly sensitive justice-related information. Security and privacy are foundational requirements, not later enhancements.

- Do not use browser `localStorage` as the system of record for real client/case data.
- Do not place confidential client content in GitHub, source code, demo fixtures, URLs, or unprotected logs.
- Use authenticated server-side APIs and a durable database for production records.
- Enforce authorization on the server, including agency, case assignment, role, and record-level permissions.
- Encrypt data in transit and at rest; manage secrets outside the repository.
- Use audit trails for sensitive record access and changes.
- Define data retention, legal holds, deletion, export, breach response, and backup/recovery policies with counsel and agency leadership.
- Minimize collection and notification content; disclose what is collected and who can see it.
- Separate demo/test data from production data and environments.
- Use synthetic data for development and testing.
- Complete threat modeling, security testing, accessibility review, and legal/privacy review before a real-client pilot.

This blueprint is not a legal compliance certification. Applicable law, procurement requirements, records rules, and agency policy must be evaluated by qualified reviewers before deployment.

---

## 7. Beta 0.2 audit — current repository snapshot

**Audit date:** November 2, 2026  
**Reviewed branch:** `main`  
**Reviewed file:** `index.html`  
**Reviewed blob:** `623987d5277b5ba75afc390303f37b4d4bafbbb6`

Static inspection found:
- One application HTML file, approximately 5,319 lines / 139 KB.
- One embedded stylesheet and one embedded JavaScript block.
- 72 function declarations, 81 element IDs, and no duplicate element IDs detected.
- 50 buttons, 22 inputs, 7 selects, and 13 textareas.
- No external JavaScript or CSS dependencies detected.
- Persistence is browser-local through `localStorage`; no server API/database integration was found in this file.
- The code explicitly labels the add-client workflow as a demo and warns not to enter identifying or confidential client information.

### Current prototypes identified

Dashboard/caseload, client profiles, goals and progress, client calendar, Outlook handoff, simulated Client App, activity tracking, 12-pillar Learning Center, Probation Dictionary, Messenger prototype, contact recording/case-note generation, and client timeline.

### Audit issues to carry into CSS NEXT

1. **No production data architecture:** browser-local storage is device/browser-specific and is not a secure, synchronized multi-user system of record.
2. **Court Report checklist missing from current main:** the six-step checklist and client/report-specific save/restore logic exist in the September 17, 2026 historical snapshot (commit `9ba2eaa62b6222cc15b8a094511679108e4cbc50`) but are absent from the reviewed November 2 `main` file. Recover and test this behavior during migration.
3. **Timeline persistence:** new-goal timeline entries are generated using temporary in-memory logic and are not durable.
4. **Critical-barrier count mismatch:** newly added clients store `criticalBarrier` as a Boolean, while the dashboard count compares it to the string `"Yes"`.
5. **Duplicate/overridden function definitions:** `openLearnMode` and `closeLearnMode` are declared more than once at top level; the later declarations override the earlier ones. Timeline rendering also has a nested temporary implementation and a separate global implementation.
6. **Input escaping:** some dynamically rendered demo-client fields are interpolated into `innerHTML` without consistent escaping. Production code must use safe DOM rendering and server-side validation.
7. **Inconsistent storage error handling:** some storage reads/writes are guarded; others can throw and interrupt an interaction.
8. **Name-based relationships:** goals and calendar data are keyed by client display name in places. Replace with stable client/case IDs.
9. **Feature maturity:** Messenger, client app, Outlook handoff, and several interactions are prototypes, not proof of secure delivery, real-time synchronization, or production integration.

This is a static source review, not a live browser test, security assessment, or full regression test. Each migrated feature needs acceptance criteria and repeatable tests.

---

## 8. Migration strategy

Use a **hybrid rebuild**: create a clean CSS NEXT foundation while preserving and selectively migrating proven Beta 0.2 behavior.

1. Freeze the current `main` branch as the reference baseline. Do not develop CSS NEXT directly on it.
2. Keep all CSS NEXT work on `css-next` and subsequent feature branches.
3. Maintain an inventory of each Beta 0.2 feature: behavior, data shape, known defects, acceptance criteria, and migration decision.
4. Recover the historical Court Report checklist behavior and verify client/report isolation before migration.
5. Define the core data model and authorization rules before building feature screens.
6. Build the foundation with synthetic test data and automated tests.
7. Migrate one module at a time; do not retire a proven feature until its replacement passes agreed tests.
8. Keep demo data separate. Do not assume browser-local demo data is included in the GitHub repository or automatically migrate it.
9. Use pull requests and review before merging any CSS NEXT work into `main`.
10. Pilot with a small, approved group only after security, privacy, legal, accessibility, and operational readiness reviews.

### Proposed first milestones

- **N1 — Blueprint review:** agree on core entities, role model, module boundaries, and open policy decisions.
- **N2 — Beta inventory:** complete a feature-by-feature behavior and regression checklist.
- **N3 — Technical foundation:** choose stack and hosting after requirements; establish authentication, agency scoping, API, database, migrations, tests, and audit logging.
- **N4 — Core workflow:** client/case, assignments, goals, phases, timeline, notes, calendar.
- **N5 — First migration:** Court Report checklist, then other proven modules.
- **N6 — Client experience:** mobile-first experience, resource bulletin, learning content, and voluntary reflection.
- **N7 — Advanced tracks:** SAFEZONE feasibility and provider/legal review before any implementation.

---

## 9. Decisions still required

1. Which client/case data fields are essential for the first pilot, and which should not be collected?
2. What is the initial authentication and identity model for staff and clients?
3. What exact access (if any) should officers/caseworkers have to Words That Move Me Forward content?
4. Which agency-level roles and permissions are required for the first pilot?
5. What are the agency's approved retention, audit, notification, and escalation policies?
6. Which first pilot organization and workflow will validate the core platform?
7. Which hosting, database, and application stack meet the agency's security, procurement, accessibility, and budget requirements?

---

## 10. Definition of done

A feature is not done because its screen looks complete. It is done when:
- Its user need and workflow are clear.
- Permissions and privacy rules are documented and enforced.
- Data persists correctly and is scoped to the correct agency, client, and case.
- Errors, edge cases, and accessibility have been tested.
- Staff/client acceptance criteria pass.
- Audit and support needs are addressed.
- The feature demonstrably supports the mission and does not create unnecessary burden.

**North star:** lives impacted, not features shipped.
