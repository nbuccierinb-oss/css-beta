# CSS NEXT — Permissions & Access Control Model 1.0

**Status:** Proposed baseline for design review; not an adopted agency policy  
**Date:** November 2, 2026  
**Branch:** `css-next`

## 1. Recommendation

CSS NEXT should use **role-based access control (RBAC) plus relationship- and attribute-based rules (ReBAC/ABAC)**.

A role alone must never grant access to every client record. Authorization is evaluated using all of the following:

1. **Identity:** who is authenticated?
2. **Agency membership:** which agency or agencies is the person authorized to act for?
3. **Role and scope:** what job function and permissions have been assigned, and at what scope?
4. **Relationship:** is the person assigned to this client, case, team, referral, or supervisory responsibility?
5. **Record attributes:** what type of record is this, how sensitive is it, and what sharing/consent restrictions apply?
6. **Requested action:** view, create, edit, submit, approve, share, export, archive, or delete?

Default rule: **deny unless an explicit policy permits the action.** A user interface hiding a button is not authorization. The server must enforce every access decision.

## 2. Authorization model

```text
AUTHENTICATED USER
       |
       v
AGENCY MEMBERSHIP (tenant boundary)
       |
       v
ROLE + PERMISSIONS (what actions may be performed?)
       |
       v
RELATIONSHIP (which client/case/team is this person connected to?)
       |
       v
RECORD ATTRIBUTES (sensitivity, consent, status, restrictions)
       |
       v
ACTION (read / create / edit / approve / share / export ...)
       |
       v
ALLOW or DENY + AUDIT EVENT
```

A user must pass every applicable layer. An agency membership in Agency A never grants access to Agency B. A role such as Officer does not grant access to an unassigned case. A valid assignment does not override a record-specific restriction.

## 3. Core entities and relationships

Use immutable IDs (UUIDs or equivalent); do not use names, email addresses, or job titles as authorization keys.

| Entity | Purpose | Key relationships |
|---|---|---|
| Agency | Tenant and policy boundary | Has users, teams, clients, cases, configuration |
| User | Authenticated human account | May have memberships in one or more agencies |
| AgencyMembership | Connects user to agency and membership status | User + agency; active dates/status |
| Role | Named job function, configurable per agency | Defines a permission bundle |
| Permission | Atomic action on a resource type | Example: `case.read`, `goal.update`, `report.submit` |
| RolePermission | Connects a role to allowed permissions | Role + permission; optional constraints |
| Team | Organizational unit | Belongs to one agency; has members |
| TeamMembership | Connects user to team | User + team + dates |
| Client | Person receiving services | Belongs to agency; may be linked to multiple cases |
| Case | A specific legal/supervision/service matter | Belongs to agency and client |
| CaseAssignment | Connects staff to a case | User + case + assignment type + effective dates |
| Record | A protected resource (note, goal, letter, report, event) | Agency + client/case + owner + sensitivity |
| ConsentGrant | Records a client's specific sharing authorization | Subject + recipient + record/category + purpose + expiry/revocation |
| AccessPolicy | Evaluates role, relationship, attributes, and action | Agency-configured, versioned |
| AuditEvent | Records sensitive access and changes | Actor + agency + action + resource + timestamp + outcome |

### Important relationship rules

- A client is not the same thing as a case. One client may have multiple cases.
- A staff member may have more than one assignment, but each assignment has a scope and effective dates.
- Assignment changes must not silently transfer access to unrelated records.
- Supervisory access is based on an explicitly configured team/caseload scope, not merely a Supervisor title.
- Agency administrators manage configuration and accounts; administration does not automatically grant access to confidential case content.
- Cross-agency sharing is denied by default and requires a documented, approved sharing mechanism.

## 4. Proposed initial role catalogue

Roles are starting templates, not hard-coded global rules. Each agency can rename or configure them, while CSS enforces the underlying permission boundaries.

| Role template | Intended scope | Default access principle |
|---|---|---|
| Client | Own client account and explicitly client-visible records | Own information only; no access to staff-only notes or other clients |
| Assigned Officer / Caseworker | Specifically assigned cases | Work records necessary for assigned responsibilities |
| Specialist / Resource Coordinator | Explicitly referred clients or selected resource records | Minimum fields necessary for the referral/service task |
| Supervisor | Assigned team or approved supervisory caseload | Oversight within configured scope; sensitive content may require additional permission |
| Agency Administrator | Agency configuration and user administration | No automatic access to case content |
| Auditor / Compliance Reviewer | Explicitly approved audit scope | Read-only access to permitted records and audit trails |
| System Operator | Technical platform operations | No routine content access; tightly controlled, logged, time-limited support access |

No role should receive unrestricted access across every agency.

## 5. Starter permission matrix

This is a conservative product-design baseline. Agencies must review it against their laws, contracts, policies, labor agreements, and actual workflows.

| Resource / action | Client | Assigned officer | Specialist | Supervisor | Agency admin |
|---|---|---|---|---|---|
| Own profile / permitted client fields | Read/update allowed fields | Read assigned client | Read referral-minimum fields | Read within scope | Manage account metadata, not case narrative by default |
| Case details | Client-visible subset | Read/update assigned case | Referral subset only | Read within supervisory scope | No default content access |
| Goals and progress | Read permitted goals; contribute if enabled | Read/create/update assigned case goals | Only explicitly shared/referral fields | Read within scope; approve if configured | No default content access |
| Staff case notes | No | Read/create for assigned case | No, unless specifically authorized | Read within scope if policy permits | No default content access |
| Client calendar | Read permitted items | Manage assigned case events | Only relevant referral events | Read within scope | Configure event types, not automatically view contents |
| Court reports | Client-visible subset only if approved | Read/create/submit for assigned case as authorized | No by default | Review/approve if configured | No default content access |
| Resource referral | Read own referral status | Create/manage assigned referral | Manage assigned referral | Oversight within scope | Configure resource catalogue |
| Learning Center | Use own permitted content | Assign/recommend if enabled | No by default | Oversight if configured | Manage content configuration |
| Private reflection letters | Author only; client controls approved sharing | No letter-content access by default | No by default | No by default | No by default |
| Agency users/roles/settings | No | No | No | Limited delegated admin if configured | Manage within agency |
| Audit records | Own permitted activity summary only | Access logs only as policy permits | No by default | Scoped oversight | Manage audit access; sensitive audit content separately permissioned |

**Do not interpret the matrix as final law or agency policy.** In particular, case-note visibility, court-report sharing, supervisor access, and client access to legal documents need explicit policy decisions.

## 6. Record-level and field-level restrictions

Some records require stronger restrictions than the role matrix alone can provide.

Examples:
- A staff case note may be visible to the assigned officer and authorized supervisor, but not to the client.
- A client-facing calendar may show an appointment time without exposing internal staff notes.
- A referral partner may receive only the information needed to provide a service, not the full case file.
- A restorative reflection letter is private by default and should not be exposed through ordinary officer or supervisor permissions.
- A victim-related record may require additional restrictions on identity, location, contact information, and disclosure.
- SAFEZONE location events require tightly scoped access, minimal notification content, retention rules, and human review.

Use field-level projections or purpose-built views where necessary. Do not send hidden sensitive fields to the browser and rely on the interface to conceal them.

## 7. Client consent and sharing

Consent is a separate control from staff role permissions. A consent grant must be specific and understandable.

At minimum, record:
- Who is sharing the information?
- Who is the recipient?
- Which record or category is shared?
- For what stated purpose?
- What information will the recipient see?
- When does permission expire?
- Can it be revoked, and what happens to copies already received?
- Was the client given an understandable explanation?

For **Words That Move Me Forward**:
- Private by default.
- A general share action must not offer the assigned probation officer/caseworker as a recipient.
- No automatic delivery to victims or family members.
- No assumption that a letter is safe or appropriate to send.
- Officer/caseworker visibility of letter content remains a separate policy decision; the conservative default is no content access.
- The client must receive a clear privacy notice explaining any applicable legal limits. CSS must not promise absolute confidentiality.

A consent grant does not override law, a court order, a safety restriction, or another person's rights.

## 8. Authorization decision sequence

For each protected API request, the server should:

1. Authenticate the user and validate the session.
2. Resolve the active agency context and verify active membership.
3. Load the requested resource using its immutable ID and confirm it belongs to that agency.
4. Resolve the user's active role permissions within that agency.
5. Verify the necessary client/case/team/referral relationship and its effective dates.
6. Evaluate record sensitivity, ownership, consent, legal restrictions, and workflow state.
7. Check whether the requested action is permitted (including separation-of-duties rules).
8. Allow or deny; record an audit event for sensitive reads, changes, exports, and denied access as appropriate.
9. Return only the fields the user is authorized to see.

Do not trust agency IDs, user IDs, client IDs, role names, or permission flags supplied by the browser without server-side validation.

## 9. Security controls

- Deny by default and enforce authorization on every server-side request.
- Prevent insecure direct object references: changing a record ID must not reveal another person's record.
- Enforce tenant isolation in service and data-access layers; consider database row-level security as defense in depth.
- Use least privilege and time-bound assignments.
- Remove access promptly when memberships or assignments end.
- Require stronger authentication for privileged actions and administrative accounts.
- Separate duties for sensitive actions such as report approval, access-policy changes, bulk export, and break-glass access.
- Log sensitive reads, writes, exports, sharing changes, permission changes, and break-glass events.
- Do not put sensitive content in audit messages, URLs, analytics, or notification subjects.
- Provide a controlled, time-limited emergency access process only if agency policy authorizes it. Require a reason, notify appropriate oversight, and audit every use.
- Test both positive and negative cases. Prove that unauthorized access is denied.

## 10. Decisions needed from the pilot agency

Before finalizing the first role matrix, interview officers, supervisors, administrators, and client representatives and obtain policy/legal review.

1. Can supervisors see every case in their unit, or only assigned supervisory cases?
2. Which case notes are client-visible, and which are staff-only?
3. Who may create, edit, submit, and approve court reports?
4. Can specialists see a referral only, or a limited client profile?
5. Which client calendar events and reminders are visible to clients?
6. Can a client authorize sharing of a specific resource referral or reflection letter with an outside recipient? How is identity and revocation handled?
7. What can assigned officers see about Words That Move Me Forward: nothing, completion metadata only, or another explicitly defined view?
8. Who can export records, and under what approval process?
9. What are the rules for staff reassignment, leave coverage, case transfer, and terminated access?
10. What emergency/break-glass access is permitted, who reviews it, and how quickly?
11. What retention, legal hold, disclosure, and audit rules apply?
12. What client access, correction, and appeal rights apply in the pilot jurisdiction?

## 11. Acceptance tests before pilot

- An officer cannot read a case that is not assigned to them by changing a URL or API identifier.
- A user in Agency A cannot access Agency B records, even if they know the record ID.
- A supervisor can access only the supervisory scope configured by their agency.
- An administrator can manage agency users and settings without automatically seeing case notes or private letters.
- A specialist can see only the fields and referrals needed for their assigned task.
- A client cannot see staff-only notes, another client's information, or records not marked client-visible.
- Expired or revoked assignments stop granting access immediately or within the documented session-revocation window.
- Revoked consent stops future CSS access and delivery according to the documented policy.
- Every sensitive access, export, permission change, and emergency access event is auditable.
- A user cannot approve their own restricted action when separation of duties is required.
- Automated tests cover allowed and denied combinations, including similar names, multiple cases, multiple agencies, expired assignments, and restricted records.

## 12. Definition of done

The permissions model is ready for implementation only when:
- The entity/relationship model is agreed.
- The pilot agency has approved the initial role and permission matrix.
- Client privacy and consent rules are documented.
- Server-side authorization design is reviewed.
- Data access and audit requirements are tested.
- Legal, privacy, security, and operational reviewers have signed off on the pilot scope.

**Design principle:** A person's job title is not a blank check. Access follows legitimate responsibility, defined purpose, and the minimum information needed to do the work.
