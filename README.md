# Open Infrastructure Architecture Portfolio

**IT Infrastructure, Cloud Architecture, Systems Administration & DevOps Design**

> **Status: Phase 1 (Foundation) — draft.**
> This is a documentation-first architecture portfolio. Everything described here is a **proposed design** for an **illustrative** organization. Nothing in this repository has been deployed, tested, or validated in a live environment.

---

## 1. Overview

This repository documents how the IT infrastructure of a small software development company could be designed, secured, documented, and operated. It covers identity, office networking, Microsoft 365, Microsoft Azure, DevOps, and IT operations.

The project is independent. It contains no employer names, branding, or confidential information, and is intended to be reusable across different roles and organizations.

## 2. Objectives

- Demonstrate systematic infrastructure thinking: requirements, options, trade-offs, decisions.
- Produce documentation that a small IT team could actually maintain.
- Keep the design proportionate to a 15-person organization (no enterprise overengineering).
- Distinguish clearly between baseline controls, optional capabilities, and licence-dependent features.

## 3. Reference scenario (summary)

An illustrative software development company with **15 employees**, company-managed laptops, one office (wired and wireless), Microsoft 365, Microsoft Entra ID, Microsoft Azure, and Development / Staging / Production environments.

Full scenario and assumptions: [`architecture/reference-scenario.md`](architecture/reference-scenario.md).

This is **not** a real company's verified infrastructure.

## 4. Design principles

1. **Proportionality** — design for 15 people with a clear growth path, not for 1,500.
2. **Secure by design** — least privilege, MFA, separated admin accounts, auditability.
3. **Justified choices** — every major technology choice has a recorded reason and alternatives.
4. **No hidden licensing assumptions** — features that depend on a licence tier are labelled as such.
5. **Honest status** — proposed, recommended, conceptual, illustrative, and implemented are never mixed up.
6. **Maintainability** — consistent naming, tagging, ownership, and version-controlled documentation.
7. **Recoverability over backup** — a backup is only proven by a successful restore test.

## 5. Scope

**In scope:** architecture documentation, decision records, network and identity design, access model, Azure resource organization, DevOps process design, operations and governance procedures, and editable diagrams.

**Out of scope:** live deployment, a local lab, executable infrastructure code as a deliverable, application source code, compliance certification, and pricing quotes.

## 6. Limitations

- No part of the design has been implemented or load-tested.
- IP addresses, VLAN IDs, and names are **examples**.
- RPO/RTO targets are intentionally **not defined**; they must come from business requirements.
- Microsoft licensing and feature availability change; any licensing statement must be re-verified against official Microsoft documentation before use.
- Any code or configuration snippet that appears later is labelled *illustrative, untested*.

## 7. Architecture domains

| # | Domain | Focus |
|---|--------|-------|
| 1 | Identity and Endpoint Management | Entra ID, MFA, RBAC, lifecycle, laptop security |
| 2 | Office Network | Firewall, switching, VLANs, Wi-Fi, DHCP/DNS |
| 3 | Microsoft 365 and Collaboration | Exchange Online, Teams, SharePoint, OneDrive, data protection |
| 4 | Azure Cloud Infrastructure | Subscription, resource groups, environments, Key Vault, monitoring |
| 5 | DevOps and Automation | Git workflow, CI/CD, artifact promotion, IaC concepts (Bicep) |
| 6 | IT Operations and Governance | Monitoring, incidents, changes, backup/DR, cost, runbooks |

## 8. Repository structure (planned)

```
open-infrastructure-portfolio/
├── README.md
├── architecture/
│   ├── reference-scenario.md
│   ├── overview.md
│   ├── infrastructure-design.md
│   ├── cloud-architecture.md
│   └── network-architecture.md
├── security/
│   ├── identity-management.md
│   ├── access-control-matrix.md
│   └── security-policies.md
├── operations/
│   ├── backup-and-recovery.md
│   ├── monitoring-and-alerting.md
│   ├── onboarding-offboarding.md
│   ├── incident-response.md
│   └── change-management.md
├── decisions/
│   ├── architecture-decisions.md
│   └── technology-comparison.md
└── diagrams/            # diagrams.net (.drawio) source files + exported images
```

## 9. Diagrams

Six editable diagrams (diagrams.net) are planned: main architecture, office network, identity and access, Azure resources, DevOps workflow, backup and recovery.

## 10. Terminology used in this repository

| Label | Meaning |
|-------|---------|
| **Proposed** | Part of the design in this portfolio |
| **Recommended** | Suggested option; alternatives are documented |
| **Conceptual** | Described at design level, no concrete configuration |
| **Illustrative** | Example values or snippets; not tested |
| **Implemented** | Actually built and verified (currently: none) |

## 11. Project status

| Phase | Content | Status |
|-------|---------|--------|
| 1 | Foundation: README, scenario, assumptions, ADRs | Draft |
| 2 | Main architecture | Draft |
| 3 | Network | Draft |
| 4 | Identity and security | Draft |
| 5 | Azure | Draft |
| 6 | DevOps | Draft |
| 7 | IT operations | Not started |
| 8 | Final review, PDF, presentation | Not started |

## 12. Licence

To be decided before publication (a documentation licence such as CC BY 4.0 is a common choice).
