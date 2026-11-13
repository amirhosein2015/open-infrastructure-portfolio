# Architecture Decision Records (ADRs)

> **Illustrative decisions.** These ADRs document the reasoning behind a *proposed* design for a fictional 15-person company. They are not decisions made by a real organization.
> Status values: **Proposed** (in this portfolio) · **Deferred** (intentionally not decided yet).
> Assumption IDs (A-nn) refer to [`architecture/reference-scenario.md`](../architecture/reference-scenario.md).

## Index

| ID | Title | Status |
|----|-------|--------|
| ADR-001 | Documentation-first portfolio, no live deployment | Proposed |
| ADR-002 | Single Azure subscription with one resource group per environment | Proposed |
| ADR-003 | Microsoft Entra ID as the single identity source; licence-dependent controls kept optional | Proposed |
| ADR-004 | Defer application hosting and database product selection | Deferred |
| ADR-005 | No VPN, Kubernetes, or multi-subscription design by default | Proposed |
| ADR-006 | Treat Microsoft 365 data protection separately from Azure workload backup | Proposed |
| ADR-007 | Use Infrastructure as Code (Azure Bicep) as a documented concept, not a deliverable | Proposed |

---

## ADR-001 — Documentation-first portfolio, no live deployment

1. **Context:** The portfolio author has no paid cloud, server, or software subscriptions and does not want to install a local lab. The scenario organization is assumed to already own its tenants (A-01).
2. **Problem:** How can the work demonstrate infrastructure judgment without a running environment, and without misleading readers?
3. **Options considered:**
   - A) Deploy a lab on free trials or local virtualization.
   - B) Documentation-first: architecture, diagrams, decisions, procedures.
   - C) Publish untested infrastructure code.
4. **Decision:** Option B.
5. **Reasoning:** Architecture quality shows in requirements analysis, trade-offs, and operational thinking, which documentation can demonstrate. Trials expire and local labs conflict with stated constraints. Untested code would imply validation that did not happen.
6. **Advantages:** No cost; reproducible by anyone; focuses on reasoning; honest.
7. **Disadvantages:** No hands-on proof; some details (real portal behaviour, quotas, edge cases) cannot be verified.
8. **Risks:** Readers may mistake design for experience. *Mitigation:* status labels on every document and a "Limitations" section in the README.
9. **Revisit when:** The author gains access to a sandbox environment, at which point selected designs could be implemented and clearly marked *Implemented*.

---

## ADR-002 — Single Azure subscription with one resource group per environment

1. **Context:** Three environments (Development, Staging, Production) and 15 employees (A-10: limited budget).
2. **Problem:** How should Azure resources be organized so environments are manageable and access can be controlled?
3. **Options considered:**
   - A) One subscription, one resource group per environment (possibly more resource groups per environment for shared vs. app resources).
   - B) One subscription per environment.
   - C) One resource group for everything, separated by naming only.
4. **Decision:** Option A as the starting point.
5. **Reasoning:** Azure RBAC, tagging, and policy can be scoped to resource groups, which gives workable access boundaries with minimal administrative overhead. Option C offers no real access boundary. Option B gives stronger isolation but adds overhead that is hard to justify at this size without a specific driver.
6. **Advantages:** Simple billing view; low overhead; clear RBAC scopes; easy to understand and document.
7. **Disadvantages:** Subscription-level limits, quotas, and some policies are shared; a subscription-level mistake or compromise affects all environments.
8. **Risks:** Over-broad roles assigned at subscription scope would nullify the separation. *Mitigation:* assign roles at resource-group scope, restrict Production, keep subscription-level roles to a very small set of separate admin accounts.
9. **Revisit when:** Compliance or customer requirements demand stronger isolation, Production must be owned by a different team or budget, subscription quotas become limiting, or the organization grows significantly.

> **Note:** Resource-group separation is *logical separation*. It is not the same as complete security isolation; see `architecture/cloud-architecture.md` (Phase 5).

---

## ADR-003 — Microsoft Entra ID as the single identity source; licence-dependent controls kept optional

1. **Context:** Microsoft 365 and Azure both rely on Entra ID (A-02). Licence tiers are unknown (A-03).
2. **Problem:** How to define identity controls without assuming features the organization may not have licensed?
3. **Options considered:**
   - A) Design around premium capabilities (e.g. Conditional Access, Intune) as mandatory.
   - B) Define a baseline that works on any tier, and list premium controls as optional enhancements.
   - C) Use separate identity stores per system.
4. **Decision:** Option B, with Entra ID as the single identity source.
5. **Reasoning:** One identity source simplifies onboarding/offboarding and auditing. A tier-independent baseline avoids unsupported claims; premium features are documented as upgrades with the dependency stated.
6. **Advantages:** Honest about licensing; usable on a limited budget; clear upgrade path.
7. **Disadvantages:** Baseline may offer less granular control than premium options; endpoint management in particular may be limited without the relevant licence.
8. **Risks:** Licensing details change. *Mitigation:* every licensing statement must be verified against current official Microsoft documentation and carry a date and source.
9. **Revisit when:** Licence tiers are confirmed, security requirements increase, or the device fleet grows beyond what manual processes can manage.

> **Verification pending:** Which MFA and baseline-protection options are available at each licence tier will be confirmed from Microsoft Learn in Phase 4. No claim is made here.

---

## ADR-004 — Defer application hosting and database product selection

1. **Context:** The technology stack, traffic, and data requirements are unknown (A-04).
2. **Problem:** The Azure design needs hosting and data tiers, but choosing a product now would be guesswork.
3. **Options considered:**
   - A) Pick a default (e.g. virtual machines or containers) now.
   - B) Document decision criteria and a short comparison of candidate options; choose when requirements are known.
4. **Decision:** Option B — **Deferred**.
5. **Reasoning:** Hosting and database choices depend on the application's runtime, scaling pattern, operational skills, and cost. Defaulting to any single option would contradict the "no unjustified technology" principle.
6. **Advantages:** Avoids overengineering and unsupported assumptions; keeps the design reusable.
7. **Disadvantages:** The Azure diagram will show hosting and database as generic tiers rather than named products.
8. **Risks:** The architecture may look incomplete. *Mitigation:* provide a decision framework and comparison in `decisions/technology-comparison.md`.
9. **Revisit when:** A concrete application profile (runtime, expected load, data model, team skills) is provided.

---

## ADR-005 — No VPN, Kubernetes, or multi-subscription design by default

1. **Context:** Small team, limited budget, and a requirement to avoid overengineering.
2. **Problem:** Which advanced components, if any, belong in the baseline?
3. **Options considered:**
   - A) Include them to show breadth.
   - B) Exclude by default; document the specific triggers that would justify adding each.
4. **Decision:** Option B.
5. **Reasoning:** Each component adds cost and operational burden. A VPN is justified only if resources require private network connectivity; Kubernetes only if container orchestration needs exist; extra subscriptions only for a concrete isolation or governance driver.
6. **Advantages:** Simpler, cheaper, easier to operate and secure.
7. **Disadvantages:** May need rework if requirements change quickly.
8. **Risks:** Under-provisioning relative to future needs. *Mitigation:* documented triggers and growth path.
9. **Revisit when:** Private connectivity to cloud resources is required, workloads need orchestration, or governance needs exceed one subscription.

---

## ADR-006 — Treat Microsoft 365 data protection separately from Azure workload backup

1. **Context:** The organization uses both Microsoft 365 (Exchange Online, SharePoint Online, OneDrive) and Azure workloads.
2. **Problem:** How should backup and recovery be designed so that no data class is silently unprotected?
3. **Options considered:**
   - A) One generic "backup" section covering everything.
   - B) Separate protection approaches for Azure workloads and Microsoft 365 data, each with its own scope, retention, and restore procedure.
4. **Decision:** Option B.
5. **Reasoning:** These are different platforms with different protection mechanisms. Azure workload backup should not be assumed to cover Exchange Online, SharePoint, or OneDrive. Built-in Microsoft 365 retention and recovery features have limits that must be evaluated against business requirements.
6. **Advantages:** Makes coverage gaps visible; supports accurate restore planning.
7. **Disadvantages:** More documentation; possible additional tooling or cost for Microsoft 365 backup.
8. **Risks:** Assuming built-in features meet requirements. *Mitigation:* document the built-in capabilities from official sources in Phase 7 and decide on third-party backup only against defined RPO/RTO.
9. **Revisit when:** RPO/RTO or retention requirements are defined, or after a restore test reveals gaps.

> RPO/RTO values are intentionally not set. They must be derived from business requirements (A-06). A successful backup job does not prove recoverability; restore testing is a required operational activity.

---

## ADR-007 — Use Infrastructure as Code (Azure Bicep) as a documented concept, not a deliverable

1. **Context:** Repeatable infrastructure is good practice, but this portfolio cannot test code (ADR-001).
2. **Problem:** How to represent IaC without publishing unvalidated templates?
3. **Options considered:**
   - A) Full Bicep templates and pipelines.
   - B) Describe the IaC approach, module structure, and review process; include short snippets only if labelled *illustrative, untested*.
   - C) Omit IaC.
4. **Decision:** Option B.
5. **Reasoning:** Azure Bicep is Azure's native declarative IaC language and fits the Azure-first scope. Explaining how IaC fits change management and environment consistency shows understanding without implying tested code.
6. **Advantages:** Honest; relevant; avoids untested artifacts.
7. **Disadvantages:** Cannot demonstrate syntax proficiency or deployment results.
8. **Risks:** Readers expecting working code. *Mitigation:* clear labelling and README limitations.
9. **Revisit when:** A sandbox is available to validate templates, at which point tested modules could be added and marked *Implemented*.
