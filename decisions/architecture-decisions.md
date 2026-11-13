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
| ADR-008 | Three baseline VLANs (Staff, Guest, Management) with the firewall as the single inter-VLAN policy point | Proposed |
| ADR-009 | Shared-passphrase Wi-Fi for staff as the baseline, with 802.1X as the upgrade path | Proposed |
| ADR-010 | Security defaults as the MFA baseline, Conditional Access as the licence-dependent enhancement | Proposed |
| ADR-011 | Documented manual endpoint baseline, with Intune-based management as the licence-dependent upgrade | Proposed |
| ADR-012 | Separate administrative accounts and two emergency access accounts; no standing production administration for developers | Proposed |
| ADR-013 | One Key Vault per environment, Azure RBAC authorization, and managed identities for application access | Proposed |
| ADR-014 | Group-based Azure roles at resource-group scope; subscription-scope roles limited to a minimal set of administrative accounts | Proposed |

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

---

## ADR-008 — Three baseline VLANs (Staff, Guest, Management) with the firewall as the single inter-VLAN policy point

1. **Context:** A single office with about 15 employees, company-managed laptops, visitors who need internet access, and network devices that need administration (A-07, N-01, N-02).
2. **Problem:** How much segmentation is justified, and where should traffic between segments be controlled?
3. **Options considered:**
   - A) One flat network for everything.
   - B) Three VLANs (Staff, Guest, Management), with the firewall routing between them and enforcing policy.
   - C) Many VLANs (per team, per device type, plus a server VLAN) with a Layer 3 switch.
4. **Decision:** Option B. A fourth VLAN (shared devices such as a printer) is documented as optional and not deployed in the baseline.
5. **Reasoning:** Guest devices must not reach company resources, and network device management should not be reachable from ordinary devices. These two needs justify separate segments. More segments would add configuration and troubleshooting effort without a requirement behind them. Routing on the firewall gives one place to define and audit policy.
6. **Advantages:** Simple to document and audit; guest isolation; protected management plane; room to add the optional segment.
7. **Disadvantages:** All inter-VLAN traffic crosses the firewall; the firewall is a single point of failure; no isolation between staff devices.
8. **Risks:** Segmentation is only as strong as the firewall rules and the switch port and trunk configuration; a misconfiguration can silently remove isolation. *Mitigation:* explicit default-deny rules, a maintained IP and VLAN table, and periodic review of rules and port settings.
9. **Revisit when:** On-premises servers or shared devices are introduced, headcount or device count grows significantly, a compliance requirement demands finer segmentation, or internal traffic volume makes firewall routing a bottleneck.

> VLAN IDs, subnets, and rules are illustrative. See `architecture/network-architecture.md`.

---

## ADR-009 — Shared-passphrase Wi-Fi for staff as the baseline, with 802.1X as the upgrade path

1. **Context:** Staff Wi-Fi must authenticate devices. The organization is small, has no on-premises servers (N-02), and has not established whether a RADIUS service can be integrated with its identity platform.
2. **Problem:** Which Wi-Fi authentication model fits a 15-person organization without adding unjustified infrastructure?
3. **Options considered:**
   - A) Shared passphrase (WPA3-Personal, or WPA2/WPA3 transition mode for older clients).
   - B) 802.1X with a RADIUS server (WPA2/WPA3-Enterprise).
4. **Decision:** Option A as the baseline, with a documented passphrase-rotation rule at offboarding. Option B is the planned upgrade.
5. **Reasoning:** Option A needs no extra services and is realistic to operate for a small team. Option B gives per-user or per-device control but needs a RADIUS service and credential design whose compatibility with the organization's identity platform must be verified rather than assumed.
6. **Advantages (A):** Simple, no additional infrastructure, quick to roll out.
7. **Disadvantages (A):** A single secret known to everyone; no per-user attribution in logs; rotation requires touching every device.
8. **Risks:** A former employee who knows the passphrase keeps access until it is changed. *Mitigation:* rotate the passphrase at offboarding, keep the staff SSID on company-managed devices only, and keep guests on a separate SSID and VLAN.
9. **Revisit when:** Staff turnover increases, headcount grows, auditability or compliance requires per-user network authentication, or a verified RADIUS option becomes available.

> Support for specific WPA3 modes must be verified on the actual access points and clients. See `architecture/network-architecture.md` for sources.

---

## ADR-010 — Security defaults as the MFA baseline, Conditional Access as the licence-dependent enhancement

1. **Context:** MFA is required, but the licence tier is unknown (A-03, I-03). Microsoft documents security defaults as available at no extra cost, and Conditional Access as requiring at least Microsoft Entra ID P1 (Microsoft also states that Microsoft 365 Business Premium provides Conditional Access). Sources and access date are in `security/identity-management.md`.
2. **Problem:** How should MFA be enforced without assuming a licence that may not exist?
3. **Options considered:**
   - A) Security defaults.
   - B) Conditional Access policies.
   - C) Per-user legacy MFA settings.
4. **Decision:** Option A as the baseline; Option B as the documented enhancement once licensing is confirmed. Option C is not used.
5. **Reasoning:** Security defaults require no licence and cover MFA registration, MFA for administrators, blocking of legacy authentication, and MFA for privileged activities. Conditional Access allows precise control but needs licensing and careful design. Microsoft states that the two are alternatives: security defaults must be disabled before Conditional Access replaces them.
6. **Advantages:** Free and quick; a sound starting point; clear upgrade path.
7. **Disadvantages:** No customization and no exclusions; one setting for everyone; emergency access accounts cannot be excluded.
8. **Risks:** Switching to Conditional Access can leave a gap if protections are not recreated, and a policy error can lock out administrators. *Mitigation:* recreate every protection explicitly before switching, keep emergency access accounts, and test.
9. **Revisit when:** A qualifying licence is confirmed, device-compliance conditions are needed, or exceptions per group or application become necessary.

> Licence statements must be re-verified before use. They are accurate only as of the access date recorded in the identity document.

---

## ADR-011 — Documented manual endpoint baseline, with Intune-based management as the licence-dependent upgrade

1. **Context:** Company-managed Windows laptops (A-08). Microsoft's Intune licensing page states that Intune is included only with the licences listed on its plans and pricing page, and names Microsoft 365 E3, E5, and E7 as examples. It does not name Business Premium, so no claim is made about it.
2. **Problem:** How to secure laptops without assuming that central device management is licensed?
3. **Options considered:**
   - A) A documented manual baseline (encryption, updates, antimalware, no daily local admin, inventory, lost-device procedure).
   - B) Intune-based central management.
   - C) No defined endpoint standard.
4. **Decision:** Option A now; Option B when licensing is confirmed. Option C is rejected.
5. **Reasoning:** A written baseline is achievable at 15 devices and avoids an unverified licence assumption. Central management provides enforcement and reporting but depends on licensing and adds operating effort.
6. **Advantages:** No licence dependency; the standard exists and can be audited manually; clear migration path.
7. **Disadvantages:** Depends on discipline; no central proof of compliance; remote wipe and compliance-based access are unavailable; poor scaling.
8. **Risks:** Configuration drift and undetected non-compliance. *Mitigation:* an inventory, periodic spot checks, and a short documented checklist at device preparation.
9. **Revisit when:** Intune inclusion is confirmed for the organization's licence, the device count grows, or compliance-based access is required.

---

## ADR-012 — Separate administrative accounts and two emergency access accounts; no standing production administration for developers

1. **Context:** A small team where one person may hold several roles (I-04). Production must be protected from routine changes and from compromise of a daily account.
2. **Problem:** How should administrative access be structured so privilege is limited, attributable, and recoverable?
3. **Options considered:**
   - A) Administrators use their daily account with extra roles.
   - B) Separate administrative accounts, two emergency access accounts, and no standing production administration for developers.
   - C) Time-limited elevation for all privileged access (Privileged Identity Management).
4. **Decision:** Option B as the baseline; Option C as an enhancement.
5. **Reasoning:** A daily account is exposed to phishing and malware, so administrative rights should not live on it. Emergency access accounts prevent a lockout. Developers change Production through the reviewed pipeline, not by standing rights. Microsoft states that Privileged Identity Management requires Microsoft Entra ID P2 or Entra ID Governance, so it cannot be a baseline assumption.
6. **Advantages:** Smaller blast radius; attributable actions; recoverable from lockout; no licence dependency.
7. **Disadvantages:** More accounts to manage; administrators switch accounts; permanent administrative roles remain between uses; the emergency accounts need disciplined custody and testing.
8. **Risks:** Neglected emergency accounts, or administrative accounts used for daily work. *Mitigation:* alerts on any emergency sign-in, validation at least every 90 days, regular access reviews, and a rule that administrative accounts are not used for email or browsing.
9. **Revisit when:** Privileged activity grows, a qualifying licence for time-limited elevation is available, or a compliance requirement calls for stricter privileged access control.

---

## ADR-013 — One Key Vault per environment, Azure RBAC authorization, and managed identities for application access

1. **Context:** Applications need secrets, keys, and certificates, and Production secrets must not be reachable by the access given for Development (ADR-002, ADR-012). Microsoft describes Key Vault as a service for secrets, keys, and certificates, with authentication through Microsoft Entra ID and authorization through Azure RBAC or access policies. Its overview describes a vault per application restricted to a specific application and team, and does not address environments.
2. **Problem:** How should vaults be organized, and how should applications and people be authorized?
3. **Options considered:**
   - A) One shared vault for all environments.
   - B) One vault per environment.
   - C) One vault per application and environment.
4. **Decision:** Option B at the baseline, scaling to Option C when several applications exist. Authorization uses Azure RBAC, and applications use managed identities with the narrowest role.
5. **Reasoning:** Matching the vault boundary to the environment boundary lets role assignments keep Production secrets separate from Development access. Managed identities avoid stored credentials. The environment-level choice is the author's recommendation, not a Microsoft statement.
6. **Advantages:** Clear RBAC boundary per environment; no credentials in code or configuration; consistent with the role model.
7. **Disadvantages:** More vaults to manage than a single shared vault; some work to grant each application's identity access.
8. **Risks:** Over-broad roles on the vault, or secrets copied between environments. *Mitigation:* roles at vault scope for groups and identities, no human access to the production vault by developers, access logged, and exposed secrets rotated.
9. **Revisit when:** More applications are hosted, Production moves to its own subscription, hosting selection (ADR-004) changes how identities are issued, or a requirement calls for hardware-backed key protection or other vault features not assessed here.

> Vault tier, recovery protections, and pricing are not assessed in this portfolio. Key Vault is for application secrets and **not** for employee passwords.

---

## ADR-014 — Group-based Azure roles at resource-group scope; subscription-scope roles limited to a minimal set of administrative accounts

1. **Context:** One subscription with resource groups per environment (ADR-002). Microsoft documents that Azure RBAC is additive, that a role assignment combines a principal, a role definition, and a scope, and that scopes run from management group to subscription to resource group to resource. A broad assignment at a higher scope therefore overrides a narrower one beneath it. The access matrix left open whether the cloud administrator needs the Owner role.
2. **Problem:** How should Azure roles be assigned so that environment separation holds and administrative capability is still available?
3. **Options considered:**
   - A) Broad roles at subscription scope for the technical team.
   - B) Roles assigned to groups at resource-group scope, with a minimal set of administrative accounts holding subscription-scope rights.
   - C) Time-limited elevation for all privileged access (Privileged Identity Management).
4. **Decision:** Option B. The Owner role at subscription scope is held only by the separate administrative accounts of the cloud administrator (and at most one deputy). Everyone else receives roles at resource-group scope through groups. The pipeline identity is a managed identity or service principal scoped to the target resource group. Option C is the licence-dependent enhancement.
5. **Reasoning:** Resource-group scope keeps each environment's access separate, and group-based assignment follows role changes. Someone must be able to create resource groups and manage role assignments, so a small number of subscription-scope accounts is unavoidable in a single-subscription design. Time-limited elevation requires Microsoft Entra ID P2 or Entra ID Governance and is not assumed (ADR-012).
6. **Advantages:** Preserves the environment boundary; small, reviewable set of powerful accounts; no licence dependency.
7. **Disadvantages:** The subscription-scope accounts are powerful and permanent without elevation tooling; creating resource groups needs a higher scope than operating inside them.
8. **Risks:** Role creep and subscription-scope assignments made for convenience. *Mitigation:* quarterly review of Azure role assignments at subscription and resource-group scope, no subscription-scope roles for developers, and alerts on changes to role assignments.
9. **Revisit when:** A qualifying licence enables time-limited elevation, a narrower built-in or custom role can replace Owner for the cloud administrator (to be evaluated and verified, not assumed), headcount grows, or Production moves to its own subscription.

> Which built-in role best fits each duty must be verified against the permissions each role actually grants before use. The role names in the access matrix are illustrative.
