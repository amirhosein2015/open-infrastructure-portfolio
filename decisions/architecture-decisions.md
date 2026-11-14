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
| ADR-015 | Short-lived branches merged into a protected main; environments are not branches | Proposed |
| ADR-016 | Promote one immutable artifact through all environments, with manual approval before Production and a designed rollback | Proposed |
| ADR-017 | Federated deployment identity scoped per environment instead of stored pipeline secrets | Proposed |
| ADR-018 | Microsoft 365 recovery posture: built-in protections as baseline, dedicated backup product deferred until RPO and RTO exist | Deferred |
| ADR-019 | Proportionate change management (standard, normal, emergency) reusing pipeline approval, with a NIST-aligned incident lifecycle | Proposed |
| ADR-020 | Small, owner-based alerting baseline; built-in network dashboards; no self-hosted monitoring tool | Proposed |

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

> **Verification:** The MFA and baseline-protection options per licence tier are documented from Microsoft Learn in `security/identity-management.md`; Intune in Business Premium remains unverified.

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
8. **Risks:** Assuming built-in features meet requirements. *Mitigation:* built-in capabilities are documented from official sources in `operations/backup-and-recovery.md`, and the decision on a third-party backup is made on third-party backup only against defined RPO/RTO.
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

---

## ADR-015 — Short-lived branches merged into a protected main; environments are not branches

1. **Context:** About eight engineers (D-03), three environments (ADR-002), and a requirement that the same artifact reaches every environment. The Git product is not selected (A-09). GitHub's documentation states that branch protection can require approving reviews, passing status checks, resolved conversations, and can block force pushes, with availability depending on plan and repository type.
2. **Problem:** Which branching model keeps changes reviewed and releasable without heavy process?
3. **Options considered:**
   - A) Short-lived feature branches merged into a protected `main`.
   - B) Long-lived branches per environment.
   - C) GitFlow-style release and hotfix branches.
4. **Decision:** Option A. `main` is protected, always releasable, and every environment deploys an artifact built from it.
5. **Reasoning:** Option B encourages per-environment rebuilds and drift, which contradicts ADR-016. Option C adds merge overhead that a team of this size does not need. Option A keeps a single source of truth and fast integration.
6. **Advantages:** Simple; supports build-once; small merges are easier to review and revert.
7. **Disadvantages:** Relies on dependable automated checks; unfinished work must be hidden behind configuration if merged early; needs review discipline in a small team.
8. **Risks:** Rubber-stamp reviews or a bottleneck with only two leads; branch protection unavailable on a chosen plan. *Mitigation:* any engineer may approve routine changes, sensitive paths need a lead or the cloud administrator, an alternate approver is named, and the product and plan are checked against section 12 of the infrastructure design.
9. **Revisit when:** The team grows, release cadence needs release branches, a compliance requirement demands stricter controls, or a long-running parallel version must be maintained.

---

## ADR-016 — Promote one immutable artifact through all environments, with manual approval before Production and a designed rollback

1. **Context:** Production must be protected from untested change (ADR-012, ADR-014), and the hosting product is not selected (ADR-004). GitHub's documentation describes environments that can require reviewers, prevent self-review, restrict deployable branches, and scope secrets, and states that on Free, Pro, and Team plans these rules are available only for public repositories. Microsoft documents slot swapping as a rollback mechanism for App Service in specific tiers.
2. **Problem:** How should changes reach Production so that what is tested is what is deployed, a human confirms the release, and recovery is quick?
3. **Options considered:**
   - A) Promote one immutable artifact: Development, Staging, then Production after approval.
   - B) Rebuild for each environment.
   - C) Deploy automatically to Production on every merge.
4. **Decision:** Option A. Production requires explicit approval from a named approver who did not request the deployment. Rollback by redeploying the previous retained artifact is the baseline; slot swap and feature flags are product-dependent additions.
5. **Reasoning:** Building once removes differences between tested and deployed binaries. Separating requester and approver adds a human check at low cost. Redeploying a retained artifact works with any product, whereas slot swap works only where the product supports it.
6. **Advantages:** Reproducible releases; clear audit trail; fast, product-neutral rollback; recorded approvals.
7. **Disadvantages:** Slower than fully automatic release; needs artifact storage and retention; approval depends on a small group; data changes may not be reversible by rolling back the application.
8. **Risks:** Approval becoming a formality, the platform lacking an enforced approval gate on the chosen plan, rollback never rehearsed. *Mitigation:* defined preconditions for approval, a recorded manual sign-off with restricted pipeline permissions as the fallback, backward-compatible data changes, and periodic rollback rehearsals in Staging.
9. **Revisit when:** Release frequency rises, the hosting product is selected and offers progressive delivery, the Git or pipeline product is chosen, or a requirement calls for stronger separation of duties.

> Retention windows, rollback triggers, and release windows are intentionally not defined.

---

## ADR-017 — Federated deployment identity scoped per environment instead of stored pipeline secrets

1. **Context:** The pipeline must deploy to Azure. Microsoft documents that GitHub Actions can authenticate to Azure through OpenID Connect, using a Microsoft Entra application or user-assigned managed identity with a federated identity credential, and its instructions store only identifiers (client, tenant, and subscription IDs), not a client secret. The page does not specify a role scope. Azure roles in this design are assigned at resource-group scope (ADR-014).
2. **Problem:** How should the pipeline authenticate without creating a long-lived credential that can leak?
3. **Options considered:**
   - A) A service principal with a stored client secret.
   - B) Federated credentials (workload identity federation), one identity per environment.
   - C) Personal credentials of an administrator.
4. **Decision:** Option B. Each environment has its own deployment identity with a narrowly scoped role at that environment's resource group. The production identity is usable only from jobs that passed the approval gate and from `main`. Option C is rejected.
5. **Reasoning:** A federated credential removes the need to store and rotate a password, and tokens are short-lived and tied to a repository. Per-environment identities keep a compromise of the Development pipeline away from Production. Personal credentials break attribution and separation of accounts.
6. **Advantages:** No long-lived pipeline password; least privilege per environment; consistent with the access model.
7. **Disadvantages:** More setup and more identities; trust configuration must be scoped precisely; depends on the pipeline product supporting federation.
8. **Risks:** An over-broad trust rule that lets unintended workflows obtain tokens, or an over-broad role assignment. *Mitigation:* restrict trust to the specific repository, branch, or environment, assign roles at resource-group scope, review identities quarterly, and alert on role-assignment changes.
9. **Revisit when:** The pipeline product is chosen and lacks federation support, Production moves to its own subscription, or the deployment model changes.

> The exact role needed for deployment must be verified against the permissions of each candidate role before use. No identity has been created.

---

## ADR-018 — Microsoft 365 recovery posture: built-in protections as baseline, dedicated backup product deferred until RPO and RTO exist

1. **Context:** Microsoft 365 data (Exchange mail, SharePoint, OneDrive) is not covered by Azure Backup (ADR-006). Microsoft documents built-in protections: Exchange Recoverable Items retention (14 days by default, up to 30), SharePoint and OneDrive recycle bins (93 days), file restore (30 days), version history, and short-term metadata backups. Microsoft also offers Microsoft 365 Backup as a pay-as-you-go service, and third-party products exist. RPO and RTO are not defined (A-06), and no prices are stated in this portfolio.
2. **Problem:** What recovery posture should the design assume for Microsoft 365 data, and when should a dedicated backup be chosen?
3. **Options considered:**
   - A) Built-in protections only, with gaps documented.
   - B) Microsoft 365 Backup (first-party, pay-as-you-go).
   - C) A third-party backup product.
4. **Decision:** Deferred. Option A is the documented baseline. The choice between B and C, or staying with A, is made only after management defines RPO, RTO, and retention for each data class and a restore test shows whether the built-in protections meet them.
5. **Reasoning:** Choosing a product before objectives exist would invent requirements and costs. Documenting the built-in limits keeps the gap visible and honest.
6. **Advantages:** No unjustified spend; the gaps are explicit; the decision is traceable to requirements.
7. **Disadvantages:** Recovery from long-undetected deletion, tenant-wide compromise, or retention beyond built-in windows may not be possible until the decision is made.
8. **Risks:** The deferral becomes permanent by neglect. *Mitigation:* the objectives table is reviewed yearly and after incidents, and restore tests are run per data owner.
9. **Revisit when:** Objectives are defined, a restore test fails, a contract or compliance driver sets retention, data volume or criticality grows, or Microsoft changes the built-in protections or the licence terms of its backup service.

> Feature behaviour and licence terms must be re-verified before use. No backup has been configured or tested.

---

## ADR-019 — Proportionate change management (standard, normal, emergency) reusing pipeline approval, with a NIST-aligned incident lifecycle

1. **Context:** A small team already has pull-request review and a production approval gate (ADR-015, ADR-016). Changes outside code (tenant settings, firewall rules, roles) have no equivalent control. NIST SP 800-61 Rev. 3 states that organizations should use whichever incident life cycle model suits them best.
2. **Problem:** How should changes and incidents be governed without slowing a 15-person organization or duplicating controls?
3. **Options considered:**
   - A) Three change types (standard, normal, emergency) with approval by area, reusing existing gates, and a simple seven-step incident lifecycle.
   - B) Heavyweight change advisory board and formal ITIL-style process.
   - C) No formal process; ad-hoc communication.
4. **Decision:** Option A.
5. **Reasoning:** Option B is disproportionate at this size. Option C leaves risky changes (lockout, exposure) unreviewed and incidents unstructured. Option A adds review only where risk justifies it.
6. **Advantages:** Light; no double approval for releases; clear emergency path; consistent records; learning loop through reviews.
7. **Disadvantages:** Depends on discipline; the classification can be misjudged; a simple register offers little reporting.
8. **Risks:** Everything classed as standard or emergency to avoid review. *Mitigation:* when in doubt treat as normal, emergency changes are reviewed afterwards, and the mix of change types is tracked.
9. **Revisit when:** The team or system count grows, a compliance or customer requirement demands stricter evidence, or change-caused incidents rise.

> Time limits, lead times, response times, and notification duties are intentionally not defined.

---

## ADR-020 — Small, owner-based alerting baseline; built-in network dashboards; no self-hosted monitoring tool

1. **Context:** Monitoring spans identity, Microsoft 365, the office network, Azure, applications, the pipeline, backups, secrets, and cost. Microsoft documents alert rules, action groups, stateful and stateless alerts, and 30-day storage of fired alerts. There are no on-premises servers (N-02). Thresholds and retention are not defined.
2. **Problem:** What monitoring baseline gives useful coverage without alert fatigue, extra cost, or a tool to host?
3. **Options considered:**
   - A) A small catalogue of owner-assigned alerts, routed to shared channels, plus built-in device dashboards for the network.
   - B) A cloud-managed network monitoring service from the device vendor.
   - C) A self-hosted monitoring platform.
   - D) Broad alerting on everything available.
4. **Decision:** Option A as baseline. Option B is an optional enhancement if available with the chosen devices and its licence and cost are verified. Options C and D are rejected for this scenario.
5. **Reasoning:** Alerts without an owner and a first action are noise. A self-hosted tool would need a server and maintenance the organization does not have. Built-in dashboards cost nothing extra and fit the size.
6. **Advantages:** Low cost and effort; clear ownership; links to runbooks and incident priorities.
7. **Disadvantages:** Limited correlation; manual review of network logs; gaps appear only after incidents reveal them.
8. **Risks:** Missed detection because the catalogue is small; alerts that silently stop working; short default log retention. *Mitigation:* add alerts after each incident, test notification routes, review alerts that never fire, and decide early which logs must outlive default retention.
9. **Revisit when:** Services or environments multiply, security needs call for central correlation, after-hours coverage is required, or compliance demands defined log retention.

> Entra log export paths and licence requirements are not verified. No alert has been created.

