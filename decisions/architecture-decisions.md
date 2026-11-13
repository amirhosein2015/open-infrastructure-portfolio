Architecture Decision Records (ADRs)

Illustrative decisions. These ADRs document the reasoning behind a proposed design for a fictional 15-person company. They are not decisions made by a real organization. Status values: Proposed (in this portfolio) · Deferred (intentionally not decided yet). Assumption IDs (A-nn) refer to architecture/reference-scenario.md.

Index
ID	Title	Status
ADR-001	Documentation-first portfolio, no live deployment	Proposed
ADR-002	Single Azure subscription with one resource group per environment	Proposed
ADR-003	Microsoft Entra ID as the single identity source; licence-dependent controls kept optional	Proposed
ADR-004	Defer application hosting and database product selection	Deferred
ADR-005	No VPN, Kubernetes, or multi-subscription design by default	Proposed
ADR-006	Treat Microsoft 365 data protection separately from Azure workload backup	Proposed
ADR-007	Use Infrastructure as Code (Azure Bicep) as a documented concept, not a deliverable	Proposed
ADR-001 — Documentation-first portfolio, no live deployment
Context: The portfolio author has no paid cloud, server, or software subscriptions and does not want to install a local lab. The scenario organization is assumed to already own its tenants (A-01).
Problem: How can the work demonstrate infrastructure judgment without a running environment, and without misleading readers?
Options considered:
A) Deploy a lab on free trials or local virtualization.
B) Documentation-first: architecture, diagrams, decisions, procedures.
C) Publish untested infrastructure code.
Decision: Option B.
Reasoning: Architecture quality shows in requirements analysis, trade-offs, and operational thinking, which documentation can demonstrate. Trials expire and local labs conflict with stated constraints. Untested code would imply validation that did not happen.
Advantages: No cost; reproducible by anyone; focuses on reasoning; honest.
Disadvantages: No hands-on proof; some details (real portal behaviour, quotas, edge cases) cannot be verified.
Risks: Readers may mistake design for experience. Mitigation: status labels on every document and a "Limitations" section in the README.
Revisit when: The author gains access to a sandbox environment, at which point selected designs could be implemented and clearly marked Implemented.
ADR-002 — Single Azure subscription with one resource group per environment
Context: Three environments (Development, Staging, Production) and 15 employees (A-10: limited budget).
Problem: How should Azure resources be organized so environments are manageable and access can be controlled?
Options considered:
A) One subscription, one resource group per environment (possibly more resource groups per environment for shared vs. app resources).
B) One subscription per environment.
C) One resource group for everything, separated by naming only.
Decision: Option A as the starting point.
Reasoning: Azure RBAC, tagging, and policy can be scoped to resource groups, which gives workable access boundaries with minimal administrative overhead. Option C offers no real access boundary. Option B gives stronger isolation but adds overhead that is hard to justify at this size without a specific driver.
Advantages: Simple billing view; low overhead; clear RBAC scopes; easy to understand and document.
Disadvantages: Subscription-level limits, quotas, and some policies are shared; a subscription-level mistake or compromise affects all environments.
Risks: Over-broad roles assigned at subscription scope would nullify the separation. Mitigation: assign roles at resource-group scope, restrict Production, keep subscription-level roles to a very small set of separate admin accounts.
Revisit when: Compliance or customer requirements demand stronger isolation, Production must be owned by a different team or budget, subscription quotas become limiting, or the organization grows significantly.

Note: Resource-group separation is logical separation. It is not the same as complete security isolation; see architecture/cloud-architecture.md (Phase 5).

ADR-003 — Microsoft Entra ID as the single identity source; licence-dependent controls kept optional
Context: Microsoft 365 and Azure both rely on Entra ID (A-02). Licence tiers are unknown (A-03).
Problem: How to define identity controls without assuming features the organization may not have licensed?
Options considered:
A) Design around premium capabilities (e.g. Conditional Access, Intune) as mandatory.
B) Define a baseline that works on any tier, and list premium controls as optional enhancements.
C) Use separate identity stores per system.
Decision: Option B, with Entra ID as the single identity source.
Reasoning: One identity source simplifies onboarding/offboarding and auditing. A tier-independent baseline avoids unsupported claims; premium features are documented as upgrades with the dependency stated.
Advantages: Honest about licensing; usable on a limited budget; clear upgrade path.
Disadvantages: Baseline may offer less granular control than premium options; endpoint management in particular may be limited without the relevant licence.
Risks: Licensing details change. Mitigation: every licensing statement must be verified against current official Microsoft documentation and carry a date and source.
Revisit when: Licence tiers are confirmed, security requirements increase, or the device fleet grows beyond what manual processes can manage.

Verification pending: Which MFA and baseline-protection options are available at each licence tier will be confirmed from Microsoft Learn in Phase 4. No claim is made here.

ADR-004 — Defer application hosting and database product selection
Context: The technology stack, traffic, and data requirements are unknown (A-04).
Problem: The Azure design needs hosting and data tiers, but choosing a product now would be guesswork.
Options considered:
A) Pick a default (e.g. virtual machines or containers) now.
B) Document decision criteria and a short comparison of candidate options; choose when requirements are known.
Decision: Option B — Deferred.
Reasoning: Hosting and database choices depend on the application's runtime, scaling pattern, operational skills, and cost. Defaulting to any single option would contradict the "no unjustified technology" principle.
Advantages: Avoids overengineering and unsupported assumptions; keeps the design reusable.
Disadvantages: The Azure diagram will show hosting and database as generic tiers rather than named products.
Risks: The architecture may look incomplete. Mitigation: provide a decision framework and comparison in decisions/technology-comparison.md.
Revisit when: A concrete application profile (runtime, expected load, data model, team skills) is provided.
ADR-005 — No VPN, Kubernetes, or multi-subscription design by default
Context: Small team, limited budget, and a requirement to avoid overengineering.
Problem: Which advanced components, if any, belong in the baseline?
Options considered:
A) Include them to show breadth.
B) Exclude by default; document the specific triggers that would justify adding each.
Decision: Option B.
Reasoning: Each component adds cost and operational burden. A VPN is justified only if resources require private network connectivity; Kubernetes only if container orchestration needs exist; extra subscriptions only for a concrete isolation or governance driver.
Advantages: Simpler, cheaper, easier to operate and secure.
Disadvantages: May need rework if requirements change quickly.
Risks: Under-provisioning relative to future needs. Mitigation: documented triggers and growth path.
Revisit when: Private connectivity to cloud resources is required, workloads need orchestration, or governance needs exceed one subscription.
ADR-006 — Treat Microsoft 365 data protection separately from Azure workload backup
Context: The organization uses both Microsoft 365 (Exchange Online, SharePoint Online, OneDrive) and Azure workloads.
Problem: How should backup and recovery be designed so that no data class is silently unprotected?
Options considered:
A) One generic "backup" section covering everything.
B) Separate protection approaches for Azure workloads and Microsoft 365 data, each with its own scope, retention, and restore procedure.
Decision: Option B.
Reasoning: These are different platforms with different protection mechanisms. Azure workload backup should not be assumed to cover Exchange Online, SharePoint, or OneDrive. Built-in Microsoft 365 retention and recovery features have limits that must be evaluated against business requirements.
Advantages: Makes coverage gaps visible; supports accurate restore planning.
Disadvantages: More documentation; possible additional tooling or cost for Microsoft 365 backup.
Risks: Assuming built-in features meet requirements. Mitigation: document the built-in capabilities from official sources in Phase 7 and decide on third-party backup only against defined RPO/RTO.
Revisit when: RPO/RTO or retention requirements are defined, or after a restore test reveals gaps.

RPO/RTO values are intentionally not set. They must be derived from business requirements (A-06). A successful backup job does not prove recoverability; restore testing is a required operational activity.

ADR-007 — Use Infrastructure as Code (Azure Bicep) as a documented concept, not a deliverable
Context: Repeatable infrastructure is good practice, but this portfolio cannot test code (ADR-001).
Problem: How to represent IaC without publishing unvalidated templates?
Options considered:
A) Full Bicep templates and pipelines.
B) Describe the IaC approach, module structure, and review process; include short snippets only if labelled illustrative, untested.
C) Omit IaC.
Decision: Option B.
Reasoning: Azure Bicep is Azure's native declarative IaC language and fits the Azure-first scope. Explaining how IaC fits change management and environment consistency shows understanding without implying tested code.
Advantages: Honest; relevant; avoids untested artifacts.
Disadvantages: Cannot demonstrate syntax proficiency or deployment results.
Risks: Readers expecting working code. Mitigation: clear labelling and README limitations.
Revisit when: A sandbox is available to validate templates, at which point tested modules could be added and marked Implemented.
