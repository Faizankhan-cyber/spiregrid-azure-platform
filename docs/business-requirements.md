# SpireGrid Business Requirements

**Product:** SpireGrid: Secure Enterprise SaaS Platform on Microsoft Azure  
**Tagline:** Secure by design. Built to scale.  
**Version:** 1.0  
**Status:** Requirements baseline for design. No Azure resources deployed.

## 1. Executive summary

SpireGrid is a fictional multi-tenant SaaS web application for small and medium-sized organizations. It provides organization-specific workspaces for managing projects, tasks, team membership, and basic operational reports. The learning design uses a shared application and shared database to keep costs manageable. Each organization must remain logically isolated from every other organization. Microsoft Entra ID is the selected identity provider, with implementation details staged by version. Only synthetic, fictional data will be used.

Security, observability, documentation, cost awareness, and cleanup are requirements from the beginning. No Azure resources should be deployed until the initial architecture, threat model, cost assumptions, and V1 acceptance tests are documented.

## 2. Business problem

A shared SaaS platform can serve multiple organizations efficiently, but an authorization or data-filtering mistake can expose one organization's records to another. SpireGrid will demonstrate how to balance usability, tenant-aware authorization, repeatable infrastructure, operational visibility, and explainable student-budget costs.

## 3. Goals and non-goals

### Goals
1. Provide organization workspaces with project and task management.
2. Design Microsoft Entra ID authentication and separately enforce application authorization.
3. Enforce tenant-aware access in server-side application logic and data access.
4. Use synthetic data only.
5. Use Terraform where appropriate, with plan review and validation before deployment.
6. Validate security controls with repeatable tests and preserve evidence.
7. Check pricing, subscription eligibility, quotas, and regional availability before deployment; clean up billable resources afterward.
8. Keep documentation consistent with what is actually implemented and tested.

### Non-goals for V1
- A production-ready commercial SaaS offering or a guarantee of regulatory compliance.
- Real customer, financial, health, or other sensitive data.
- A full microservices architecture.
- AI, event streaming, multi-region deployment, or advanced disaster recovery.
- Claims of high availability or production resilience without implementation and testing.

## 4. Users and roles

| Role | Intended scope |
|---|---|
| Platform administrator | Platform configuration and tenant onboarding; restricted and audited access. |
| Organization administrator | Manage users and settings within their own organization. |
| Project manager | Manage projects and tasks within their organization. |
| Member | View assigned work and perform permitted task actions within their organization. |

These roles are an initial business model. An authorization matrix must define exact permissions before implementation. Platform administration must not accidentally provide unrestricted tenant-data access.

## 5. Functional requirements

- **FR-01 Authentication:** users sign in through Microsoft Entra ID when integration is implemented. The server validates authentication results and does not trust user-supplied identity or tenant identifiers.
- **FR-02 Organization context:** each authenticated request is associated with an authorized SpireGrid organization context.
- **FR-03 Tenant isolation:** users can read or change only records belonging to organizations for which they are authorized.
- **FR-04 Projects:** authorized users can create, view, update, and list projects within their organization.
- **FR-05 Tasks:** authorized users can create, view, update, and list tasks associated with projects in their organization.
- **FR-06 Role-based permissions:** organization administrators, project managers, and members have distinct permissions enforced server-side.
- **FR-07 Audit events:** record relevant security-sensitive actions, such as role changes and denied cross-tenant access, without logging credentials, tokens, or unnecessary sensitive data.
- **FR-08 Safe errors:** errors must not expose secrets, stack traces, or another tenant's records.
- **FR-09 Documentation:** setup, configuration, deployment, validation, cost assumptions, and teardown are documented.
- **FR-10 Synthetic data:** test with fictional organizations, users, projects, and tasks.

## 6. Non-functional requirements

- **NFR-01 Security:** least privilege, secure secret handling, server-side authorization, input validation, and secure transport wherever supported.
- **NFR-02 Tenant integrity:** derive tenant identity from a trusted, validated authorization context and check it for every relevant data operation.
- **NFR-03 Repeatability:** represent infrastructure changes as code and review Terraform formatting, validation, and plans before deployment where Terraform is used.
- **NFR-04 Observability:** provide enough logs and health checks to diagnose basic failures without leaking secrets.
- **NFR-05 Cost control:** verify current pricing, student eligibility, quotas, and regional availability; deploy only for planned milestones and remove billable resources after testing.
- **NFR-06 Maintainability:** version-control code, infrastructure, tests, and documentation.
- **NFR-07 Honest evidence:** label features as implemented and tested, temporarily demonstrated, simulated, or planned. Evidence must correspond to actual runs.

## 7. Tenant and data model

The initial model uses a shared application and shared database. Tenant-owned records must carry a tenant association such as tenant_id, or a relationship that enforces the same boundary.

Expected entities:
- Tenant / organization
- User-to-tenant membership
- Role or permission assignment
- Project
- Task
- Audit event

### Tenant-isolation rules

1. A request's tenant identifier is not proof of access.
2. Resolve authorized tenant membership using trusted server-side identity and membership records.
3. Apply tenant constraints to reads, updates, deletes, exports, and related-object lookups.
4. Check that a task's project and tenant association are consistent.
5. Deny by default when tenant context or authorization is missing.
6. Test cross-tenant access attempts using guessed IDs and modified request parameters.
7. Centralize tenant-isolation enforcement where practical; UI hiding and client-side filtering are insufficient.
8. Consider database-level isolation controls only if they fit the chosen database and access model and can be tested.

## 8. Identity and authorization

Microsoft Entra ID is the selected identity provider. Authentication establishes who the user is; SpireGrid must separately authorize what they can do and which tenant's data they may access. Before implementation, document whether customer users are represented in a common Entra tenant, through external identities, or through a controlled demo arrangement. A shared application/database does not imply all customers share one identity tenant.

Application roles and organization membership must be checked server-side. Never commit secrets. Prefer managed identities over embedded credentials where supported by the selected Azure services.

## 9. Security and privacy baseline

- Use synthetic data only.
- Keep credentials, private keys, access tokens, and connection strings out of Git.
- Use least-privilege identities and narrowly scoped permissions.
- Restrict administrative functions and audit sensitive changes.
- Use encrypted transport for deployed endpoints.
- Validate inputs and return safe errors.
- Test authorization on direct API/server-side requests, not just the user interface.
- Do not claim a security control is effective until it has been tested.

## 10. Cost and deployment constraints

This is a student-budget project, not an always-on commercial service. Prefer local development and low-cost eligible services, but verify current prices and terms before deployment. Avoid premium networking, continuously running databases, security analytics ingestion, AI, or multi-region resources without a documented requirement and cost review.

Budget alerts are notifications, not hard spending caps. Stopping compute may leave storage or other charges. After each deployment, inspect resource inventory, remove temporary resources, and verify that remaining billable dependencies are understood. Preserve code, safe logs, screenshots, test results, and recorded walkthroughs so the project remains demonstrable after cleanup.

## 11. Cumulative version roadmap

- **V1, Azure foundations:** minimal web application and data design, foundational Azure resources, Terraform workflow where appropriate, cost controls, documentation, and baseline tests.
- **V2, identity and governance:** Microsoft Entra ID integration, roles/access governance, managed identities where supported, and secret management.
- **V3, cloud administration:** network, compute, storage, monitoring, and administrative operations.
- **V4, cloud security and optional AI:** justified security posture, detection, data protection, and potentially a constrained AI capability after separate security and cost review.
- **V5, enterprise architecture:** evaluate event-driven design, resilience, backup/recovery, disaster recovery, and scaling. Multi-region components must be justified by requirements and cost.

These mappings are learning guides, not proof that the project covers an entire certification syllabus. Verify current certification names and objectives before referencing them in project materials.

## 12. V1 acceptance criteria

V1 must not be marked complete until results are reviewed. Items dependent on deployment cannot be marked passed before the relevant deployment and validation have occurred.

| ID | Acceptance criterion | Evidence |
|---|---|---|
| AC-01 | Architecture and data-flow diagrams match the actual V1 design. | Diagrams and explanation. |
| AC-02 | Requirements, threat model, and initial cost plan are version-controlled. | Repository documents. |
| AC-03 | Terraform formatting and validation pass if Terraform is used. | Command output or CI result. |
| AC-04 | Terraform plans are reviewed and secrets are absent from configuration/state committed to Git. | Review notes and secret scan. |
| AC-05 | The application provides the project/task operations in V1 scope for synthetic records. | Repeatable tests or screenshots. |
| AC-06 | The test dataset represents at least two synthetic tenants. | Fixtures or seed data. |
| AC-07 | Tenant A cannot read or modify Tenant B records through direct API/server-side requests. | Negative test output. |
| AC-08 | Authentication and authorization status is accurately documented; unimplemented Entra integration is labelled planned. | README and design notes. |
| AC-09 | No real customer data or secrets are present in the repository. | Data review and secret scan. |
| AC-10 | Deployment and cleanup steps are documented and tested; post-cleanup resource inventory is checked after deployment. | Runbook and inventory evidence. |
| AC-11 | Cost assumptions, potentially chargeable resources, alert limitations, and cleanup responsibilities are documented before deployment. | Cost-management document. |
| AC-12 | Every screenshot and test result is labelled and reflects an actual run; planned work is not marked complete. | Evidence index and status review. |

## 13. Risks and trade-offs

| Risk | Planned response |
|---|---|
| Shared database increases impact of a tenant-isolation bug. | Enforce tenant context server-side, test negative cases, review data-access paths, and consider suitable database-level controls. |
| Entra ID integration adds configuration and testing complexity. | Document the identity model and stage implementation without falsely claiming authentication is active. |
| Student subscription and free-tier limits can change. | Verify current eligibility, region, quota, and pricing before deployment. |
| Logs and security analytics can create unexpected costs or collect excess data. | Limit ingestion, avoid secrets in logs, and enable paid analytics only when justified. |
| AI features can add cost and data-exposure risks. | Defer until a later milestone and require separate data-flow, security, and cost review. |
| Overly complex architecture distracts from learning objectives. | Add services only to satisfy a documented requirement and test their value. |
| Evidence becomes stale after changes. | Capture evidence at relevant workflow stages and label version, date, and test result. |

## 14. Design-phase exit checklist

Before any Azure deployment:

- [ ] Review business requirements and assumptions.
- [ ] Draw the initial architecture and data-flow diagram.
- [ ] Write the threat model, including cross-tenant access threats.
- [ ] Select a minimal V1 technology stack and explain why.
- [ ] Identify candidate Azure resources and check current pricing and subscription eligibility.
- [ ] Define test cases and evidence to collect.
- [ ] Establish cleanup steps and a deployment checklist.

**Current state:** requirements baseline prepared. Architecture design, threat modelling, stack selection, implementation, and deployment remain future work.
