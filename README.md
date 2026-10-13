# SpireGrid

**Secure Enterprise SaaS Platform on Microsoft Azure**  
*Secure by design. Built to scale.*

SpireGrid is a learning project for designing, building, securing, and operating a fictional multi-tenant SaaS application on Microsoft Azure. It will evolve through cumulative versions, with emphasis on cloud engineering, security, repeatable infrastructure, evidence-based testing, and cost control.

> **Status:** Planning and requirements baseline. No SpireGrid Azure resources or application features have been deployed. Roadmap items are goals, not claims of implementation.

## Product concept

Organizations use SpireGrid workspaces to manage projects, tasks, team membership, and basic reports. The initial design uses a shared application and database, so tenant isolation must be enforced server-side and tested directly. Microsoft Entra ID is the selected identity provider; the exact identity model and integration will be designed before implementation. All sample data will be synthetic.

## Design principles

- **Security first:** least privilege, server-side authorization, safe error handling, and tenant isolation.
- **Repeatable:** use Terraform where appropriate and review plans before applying changes.
- **Demonstrable:** preserve source, diagrams, test output, and short walkthroughs so the project remains explainable after temporary Azure resources are removed.
- **Cost-aware:** deploy only for planned milestones, verify current pricing and eligibility, and clean up billable resources after testing.
- **Honest evidence:** distinguish implemented and tested features from planned or simulated features.

## Version roadmap

| Version | Focus | Planned outcomes |
|---|---|---|
| V1 | Azure foundations | Minimal web application, foundational resources, deployment workflow, cost controls, and baseline tests. |
| V2 | Identity and governance | Entra ID integration, authorization roles, tenant membership, managed identities where supported, and secret handling. |
| V3 | Cloud administration | Networking, compute, storage, monitoring, and operational procedures. |
| V4 | Cloud security and optional AI | Security posture, detection, data protection, and optional AI after a separate security and cost review. |
| V5 | Enterprise architecture | Evaluate event-driven components, resilience, backup/recovery, scaling, and disaster recovery against documented requirements. |

Versions are cumulative. Expensive services do not need to run continuously. Local development, automated tests, architecture evidence, and short-lived Azure deployments will be used where appropriate. Simulations will never be described as proof of a live Azure integration.

## Repository guide

- [Business requirements](docs/business-requirements.md): scope, requirements, tenant-isolation rules, roadmap, and acceptance criteria.
- docs/threat-model.md: to be written before application implementation.
- docs/cost-management.md: planned cost assumptions and deployment/cleanup controls.
- docs/architecture/: diagrams and data flows as they are created.
- infrastructure/: Terraform configuration when selected and introduced.
- app/: application source after the stack is selected.
- tests/: functional, authorization, and security tests.

Folders will be added as needed rather than created all at once.

## Before deployment

Review the business requirements, document the threat model and architecture/data flow, select a minimal technology stack, verify Azure pricing and eligibility, define tests, and document cleanup procedures before deploying resources.

## Security and cost notice

Do not commit passwords, access tokens, private keys, connection strings, Terraform state, or real personal/customer data. Use synthetic data only. Azure budget alerts are notifications, not hard spending caps; inspect usage and verify cleanup after every deployment.

## License

No license has been selected yet. Until one is added, do not assume the repository grants permission to reuse its contents.
