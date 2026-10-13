# SpireGrid Threat Model

**Status:** Initial design draft  
**Project phase:** Requirements and design; implementation has not started  
**Scope:** Proposed V1 architecture and risks that later versions must continue to address  
**Data:** Synthetic demonstration data only  
**Last reviewed:** 2026-10-13

> This document records design-time threats and planned controls. It is not evidence that controls have been implemented or tested, and it is not a claim that SpireGrid is production-secure. Update this model when the architecture, identity flow, data store, or deployment changes.

## 1. Purpose

SpireGrid is a fictional multi-tenant SaaS project. Its proposed model uses a shared application and shared database, with organization-specific workspaces for projects and tasks. The defining security requirement is that a user in one organization must not be able to read or modify another organization's records.

This threat model supports design decisions before application code or Azure resources are created. It is a working model, not a certification or compliance assessment.

## 2. Scope and assumptions

### In scope
- Sign-in and identity validation, planned through Microsoft Entra ID.
- Identity-to-organization membership mapping.
- Authorization roles and privileged operations.
- Tenant boundaries across projects, tasks, and related records.
- API input handling, error responses, and audit logging.
- Secrets and configuration.
- Infrastructure changes, deployment workflow, monitoring, and cleanup.
- Local development and temporary Azure test deployments.

### Out of scope for the initial version
- Real customer data or production customer onboarding.
- Formal regulatory compliance certification.
- A guarantee against every possible vulnerability.
- Advanced AI, multi-region, or event-driven components until separately designed.

### Assumptions requiring confirmation
1. The application and database will be shared by multiple fictional organizations.
2. Every test record will be synthetic.
3. Microsoft Entra ID is the intended identity provider, but the exact customer identity model is not selected.
4. The application stack, API shape, database, hosting service, and deployment region have not been selected.
5. Azure resources will be temporary and deployed only after cost and eligibility checks.
6. No SpireGrid application or cloud security control is currently claimed to be implemented.

## 3. Assets to protect

| Asset | Why it matters |
|---|---|
| Tenant-owned project and task records | Cross-tenant disclosure or modification breaks the core security promise. |
| User identity and tenant memberships | Incorrect mapping can grant a user access to the wrong organization. |
| Role assignments and administrative operations | Privilege escalation can affect many users or tenants. |
| Authentication credentials, tokens, and application secrets | Theft can enable impersonation or unauthorized access. |
| Infrastructure configuration and deployment credentials | Unauthorized changes can expose services or weaken controls. |
| Audit and diagnostic logs | Logs support investigation but may expose sensitive information or secrets. |
| Terraform state and deployment artifacts | State can contain sensitive values and reveal infrastructure details. |
| Azure credits, resources, and subscription configuration | Misconfiguration or forgotten resources can cause unexpected charges. |
| Project evidence and documentation | Misleading or stale evidence undermines project credibility. |

## 4. Trust boundaries and proposed data flow

The intended logical flow is:

1. A user accesses the web application.
2. The user signs in through the selected identity provider once Entra integration is implemented.
3. The server validates the authentication result and obtains a trusted identity context.
4. The application resolves the user's SpireGrid organization membership and role server-side.
5. The authorization layer checks permission and tenant scope for the requested action.
6. The data-access layer applies tenant constraints to the query or mutation.
7. The application returns only authorized results and records appropriate security events.

### Trust boundaries

- **Browser to application:** all browser input is untrusted, including object IDs, tenant IDs, and hidden form values.
- **Identity provider to application:** tokens and identity responses must be validated for issuer, audience, signature, expiry, and applicable flow requirements using supported libraries.
- **Application to database:** queries must enforce tenant scope; UI filtering alone is insufficient.
- **Application to Azure services:** service identities and permissions must be limited to required operations.
- **Developer workstation to repository/CI:** secrets, state files, and privileged credentials must not be committed or exposed in logs.
- **Temporary deployment to retained evidence:** screenshots and logs must be reviewed to remove credentials, tokens, personal data, and sensitive configuration.

The final architecture diagram must replace this conceptual flow once the technology stack is selected.

## 5. Threat rating method

Initial qualitative ratings are design priorities, not measured probabilities.

- **Impact:** High, Medium, or Low if the threat succeeds.
- **Likelihood:** High, Medium, or Low based on the proposed design and plausible mistakes.
- **Priority:** High means address before the affected feature is considered complete. Medium means address before the related feature is released or demonstrated as secure. Ratings must be revised as the architecture and tests become concrete.

## 6. Initial threat register

| ID | Threat and failure mode | Impact | Initial likelihood | Priority |
|---|---|---|---|---|
| T-01 | Cross-tenant read: a user changes an object ID or request parameter to retrieve another tenant's project/task. | High | High | High |
| T-02 | Cross-tenant modification: a user updates/deletes another tenant's record or attaches a task to a foreign tenant's project. | High | High | High |
| T-03 | Tenant spoofing: the server trusts a client-supplied tenant ID without verifying membership. | High | High | High |
| T-04 | Broken authentication: an invalid, expired, wrong-audience, or improperly validated token/session is accepted. | High | Medium | High |
| T-05 | Broken role authorization: a member performs administrator actions or changes their own role. | High | Medium | High |
| T-06 | Privilege escalation: excessive permissions, unsafe role assignment, or a compromised administrator grants broader access than intended. | High | Medium | High |
| T-07 | Secret exposure: passwords, client secrets, tokens, connection strings, or private keys enter Git, logs, screenshots, or CI output. | High | Medium | High |
| T-08 | Injection or unsafe input handling changes a query/command or causes unsafe processing. | High | Medium | High |
| T-09 | Sensitive error leakage: stack traces, internal identifiers, tokens, or foreign-tenant data appear in responses or logs. | Medium/High | Medium | High |
| T-10 | Inadequate audit trail: role changes, administrative actions, or repeated denied cross-tenant attempts cannot be investigated. | Medium/High | Medium | Medium |
| T-11 | Supply-chain or CI compromise: an untrusted dependency, workflow change, or leaked CI credential alters code or infrastructure. | High | Medium | High |
| T-12 | Infrastructure misconfiguration exposes a service, broadens network access, or grants excessive cloud permissions. | High | Medium | High |
| T-13 | Cost exhaustion/resource sprawl: forgotten resources, high-volume logs, or an expensive service consumes student credits. | Medium | Medium | Medium |
| T-14 | Stale or misleading evidence makes a planned, simulated, or untested control appear implemented. | Medium | Medium | Medium |
| T-15 | Abuse or denial of service makes the application unavailable or creates excessive resource use. | Medium | Medium | Medium |

## 7. Planned controls and how to test them

These are proposed controls and test cases. Do not mark them complete until implemented and evidenced.

### T-01 to T-03: tenant isolation

**Planned controls**
- Establish tenant context from a validated identity and server-side membership record.
- Do not use a request's tenant ID as authorization proof.
- Centralize tenant-aware authorization and data-access rules where practical.
- Scope list, read, update, delete, export, and relationship lookups to the authorized tenant.
- Verify that related objects, such as tasks and projects, belong to the same tenant.
- Deny by default when identity, membership, or tenant context is missing.

**Planned tests**
- Create two synthetic tenants, Tenant A and Tenant B, with separate users and records.
- Authenticate as Tenant A and attempt to read a known Tenant B record by changing its ID.
- Attempt to update and delete Tenant B records.
- Attempt to list all records without a tenant filter.
- Attempt to create a Tenant A task associated with a Tenant B project.
- Tamper with any tenant ID accepted by an endpoint.
- Repeat checks through direct API requests, not just the UI.
- Verify denied requests do not reveal foreign record contents.

**Pass condition:** every unauthorized cross-tenant operation is denied, no foreign data is returned or changed, and automated tests fail if the isolation control is removed.

### T-04: authentication validation

**Planned controls**
- Use a maintained identity library and the selected provider's supported flow.
- Validate tokens/sessions using documented requirements, including issuer, audience, expiry, and signature as applicable.
- Never treat a decoded token or browser-provided identity field as proof of authentication.
- Keep authentication separate from application role and tenant authorization.

**Planned tests**
- Unauthenticated request to a protected endpoint.
- Expired, malformed, wrong-audience, or invalid-signature token where the test setup supports these cases.
- User authenticated successfully but without a SpireGrid tenant membership.
- Document the exact Entra configuration and test environment before claiming live integration.

### T-05 and T-06: role and privilege control

**Planned controls**
- Define an authorization matrix for platform administrator, organization administrator, project manager, and member.
- Enforce permissions on the server for every sensitive operation.
- Restrict role assignment and require an appropriately authorized administrator.
- Avoid broad cloud permissions; use least-privilege service identities.
- Audit role and privilege changes.

**Planned tests**
- Member attempts an administrative operation.
- Organization administrator attempts to administer a different organization.
- User attempts to modify their own role through request parameters.
- Unauthorized user attempts tenant onboarding or role assignment.

### T-07: secret handling

**Planned controls**
- Keep secrets in local environment configuration or an appropriate secret store, never source control.
- Add suitable ignore rules and secret scanning.
- Use managed identities where supported instead of embedded Azure credentials.
- Avoid printing tokens, passwords, connection strings, and sensitive environment variables in logs.
- Use narrowly scoped, short-lived CI credentials where possible.

**Planned tests**
- Run a secret scan on the repository and review findings.
- Confirm environment files, private keys, Terraform state, and local secret files are ignored.
- Inspect CI logs and screenshots before retaining them.
- If a real secret is accidentally committed, revoke/rotate it; deleting the file alone does not invalidate the exposed credential.

### T-08 and T-09: input and error safety

**Planned controls**
- Use parameterized database access or a safe ORM pattern.
- Validate inputs on the server and apply sensible size and type constraints.
- Return generic client-facing errors while recording safe diagnostic details server-side.
- Never return records before authorization is complete.

**Planned tests**
- Submit malformed IDs and unexpected types.
- Submit oversized or invalid fields.
- Exercise injection payloads appropriate to the selected stack.
- Verify error responses do not include stack traces, secrets, or foreign-tenant records.

### T-10: audit and monitoring

**Planned controls**
- Record security-relevant actions such as authentication failures, role changes, administrative operations, and denied access attempts.
- Include timestamp, action, outcome, and suitable actor/tenant reference without logging tokens or unnecessary personal information.
- Define retention and access limits for logs.
- Avoid enabling costly analytics ingestion before pricing and need are reviewed.

**Planned tests**
- Trigger representative allowed and denied actions.
- Verify events contain intended fields and omit secrets.
- Verify users cannot alter or delete audit records unless a documented privileged process permits it.

### T-11 and T-12: CI and infrastructure security

**Planned controls**
- Require review of infrastructure changes and inspect Terraform plans before apply.
- Pin or constrain dependencies appropriately and run relevant dependency/security checks.
- Use least-privilege repository and cloud permissions.
- Avoid exposing management ports to the public internet without a documented need.
- Keep Terraform state out of Git and treat state as potentially sensitive.
- Protect deployment workflows and use GitHub Actions permissions narrowly.

**Planned tests**
- Run formatting, validation, static checks, and dependency checks selected for the stack.
- Review the plan for public exposure, broad roles, and unexpected resources.
- Confirm state files and secrets are absent from the repository.
- Inspect cloud resource inventory after temporary deployment and after cleanup.

### T-13: cost and resource sprawl

**Planned controls**
- Check service pricing, subscription eligibility, quotas, and regional availability before each deployment.
- Deploy only the minimum resources required for a planned milestone.
- Use budget alerts as notifications, not as a guaranteed hard cap.
- Maintain a cleanup checklist and verify residual resources, disks, logs, and other billable dependencies.
- Avoid continuous deployment of expensive analytics, AI, premium networking, or multi-region components without explicit justification.

**Planned tests**
- Review estimated costs and resource list before deployment.
- Record the resources created by each test run.
- After teardown, inspect the subscription inventory and resolve unexpected leftovers.

### T-14: evidence integrity

**Planned controls**
- Label evidence as implemented/tested, temporarily demonstrated, simulated, or planned.
- Record version/commit and test command where practical.
- Re-capture evidence after material changes.
- Never claim a screenshot proves a backend security property unless an appropriate test supports that claim.

**Planned tests**
- Compare README status against source code, tests, and actual deployment state.
- Keep acceptance criteria open until evidence is reviewed.

### T-15: availability and abuse

**Planned controls**
- Apply input limits, sensible timeouts, and request/resource constraints appropriate to the selected stack.
- Avoid publicly exposing administrative endpoints unnecessarily.
- Consider rate limiting only when justified by the architecture and available hosting capabilities.
- Keep demo exposure limited and shut down temporary resources after testing.

**Planned tests**
- Exercise invalid and repeated requests in a controlled local/test environment.
- Verify errors are handled safely and a test cannot create unbounded work or cost.

## 8. Design decisions still open

These choices affect the threat model and must be decided before implementation:

1. Application language and framework.
2. Database technology and tenant-scoping strategy.
3. Hosting model for V1.
4. API shape and authentication/session approach.
5. Entra ID tenant/customer identity model for the demonstration.
6. How roles and tenant memberships are stored and administered.
7. Which automated security checks run locally and in CI.
8. Which Azure resources, if any, are required for the first live milestone.

Record important choices in the decisions documentation after comparing viable options. Avoid selecting expensive services merely to make the diagram look enterprise-grade.

## 9. Residual risks and limitations

- A shared database means a tenant-filtering defect could have a broad impact.
- A threat model cannot prove that the implementation is secure.
- Qualitative likelihood ratings are initial estimates and may change after architecture and testing.
- Azure service availability, subscription limits, pricing, and identity licensing must be verified at implementation time.
- This student project is not a substitute for a formal security review, penetration test, or compliance assessment.

## 10. Review and change triggers

Review this document when:
- the framework, database, hosting, or identity model is selected;
- an API or data model changes;
- a new Azure service or privileged identity is introduced;
- AI, analytics, external integrations, or additional tenant-management features are added;
- a test reveals a security weakness;
- deployment exposure or data classification changes.

## 11. Current status

**Draft created; controls are proposed, not implemented.** The next design task is to draw the initial architecture/data-flow diagram and compare a minimal V1 technology stack. No Azure deployment is authorized by this document alone.
