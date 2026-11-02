# CSS NEXT Demo Login 0.1

A browser-only role-based demonstration for Project CSS NEXT. This is a prototype, not a production justice case-management system.

## Open the demo
Open `index.html` in a modern browser. No build step or external libraries are required.

## Demo accounts

| Role | Username | Password |
|---|---|---|
| Officer | `officer.demo` | `OfficerDemo01!` |
| Supervisor | `supervisor.demo` | `SupervisorDemo01!` |
| Resource Coordinator | `resource.demo` | `ResourceDemo01!` |
| Agency Administrator | `admin.demo` | `AdminDemo01!` |

Select an account card on the login page to autofill its credentials.

## What to explore
- Role-specific workspace and navigation
- Officer view limited to assigned sample cases
- Supervisor department-level sample visibility
- Resource Coordinator limited referral-focused view
- Agency Administrator configuration view without confidential case-note access by default
- Caseload Continuity sample with explicit temporary coverage concepts
- Four standard CSS divisions: Adult Probation, Juvenile Probation, Problem-Solving Courts, Pretrial Services

## Important security limitation
The usernames and passwords are intentionally embedded in public client-side code. Anyone who can inspect the page can discover them. The login is a demonstration gate only; it is not authentication or access control. All data is fictional. Never enter real client information, identifying information, credentials, or confidential records.

Production CSS NEXT requires real identity/authentication, server-side authorization on every request, secure data storage, audit logging, agency isolation, and a reviewed deployment/security architecture. A hidden page or client-side password check must never be used to protect real case data.
