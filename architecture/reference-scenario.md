# Reference Scenario and Assumptions

> **Illustrative scenario.** This describes a fictional organization used to ground the architecture. It is not a real company's verified infrastructure.

## 1. Organization profile

| Attribute | Value |
|-----------|-------|
| Type | Small software development company |
| Headcount | 15 employees |
| Location | One office; some employees may work remotely occasionally |
| Devices | Company-managed Windows laptops |
| Productivity | Microsoft 365 |
| Identity | Microsoft Entra ID |
| Cloud | Microsoft Azure (primary reference platform) |
| Environments | Development, Staging, Production |
| IT staffing | No dedicated large IT team; administration is a part-time or small-team responsibility |

## 2. Illustrative roles

| Role | Approx. count (illustrative) |
|------|------------------------------|
| Employee (non-technical) | 3 |
| Developer | 6 |
| Senior Developer / Team Lead | 2 |
| IT Administrator | 1 |
| Cloud Administrator | 1 (may be combined with IT Administrator) |
| Management | 2 |

Counts are examples to make the design concrete. Role combinations in a real small company are common and are handled in the access model by assigning *separate accounts* per privilege level.

## 3. Assumptions

Assumptions are labelled **A-nn** so other documents can reference them.

| ID | Assumption | Why it matters |
|----|------------|----------------|
| A-01 | The organization already holds paid Microsoft 365 and Azure tenants; the portfolio author does not. | Design is realistic; deployment by the author is not required. |
| A-02 | One Microsoft Entra tenant serves both Microsoft 365 and Azure. | Single identity source. |
| A-03 | Specific Microsoft 365 / Entra licence tiers are **unknown**. | Intune, Conditional Access, and similar features are treated as licence-dependent. |
| A-04 | Application technology stack, traffic, and data volumes are **unknown**. | Hosting and database products are deferred (see ADR-004). |
| A-05 | No specific regulatory or compliance obligations are assumed. | Controls are a good-practice baseline, not a compliance mapping. |
| A-06 | Business RPO/RTO requirements are **not defined**. | Values are not invented; the process for defining them is documented. |
| A-07 | Internet connectivity is a single business ISP line. | Redundant WAN is listed as a growth option, not a baseline. |
| A-08 | Developers work on company-managed laptops; personal devices are not used for company data in the baseline. | Simplifies endpoint policy. |
| A-09 | The source code is hosted in a managed Git service; the specific provider is a separate decision. | Keeps DevOps documentation vendor-aware but not locked in. |
| A-10 | Budget is limited; costs must be visible and controllable. | Favors managed services and deferring optional capabilities. |

## 4. Explicit non-assumptions

The following are **not** assumed:

- A particular programming language, framework, or database
- Azure Virtual Machines, AKS/Kubernetes, or a dedicated Virtual Network for every workload
- Multiple Azure subscriptions
- A universal VPN requirement
- Conditional Access, Intune, or other premium licence features
- Any compliance framework (ISO 27001, SOC 2, GDPR obligations, etc.)

## 5. Constraints on the portfolio itself

- Documentation-first: no purchase of cloud, server, or software; no local installation.
- Browser-based, free tooling only (GitHub, diagrams.net, Markdown, PDF).
- No claims of deployment, testing, or production experience.

## 6. Growth path (preview)

The design should remain valid as the organization grows beyond 15 people. The expected extension points (detailed in later phases) are: additional resource groups or subscriptions when ownership or governance requires it, stronger identity controls when licensing allows, additional network segments, and more automated delivery pipelines.
