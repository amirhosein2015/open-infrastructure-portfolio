# Cloud Architecture (Microsoft Azure)

> **Proposed, illustrative design.** Nothing here has been deployed or tested. Resource names, tags, and counts are examples. Assumption IDs (A-nn) refer to [`reference-scenario.md`](reference-scenario.md); decisions are in [`../decisions/architecture-decisions.md`](../decisions/architecture-decisions.md) (ADR-002, ADR-004, ADR-005, ADR-006, ADR-013, ADR-014).
> Statements about Azure are taken from official Microsoft documentation **accessed 2026-11-13** and must be re-verified before use. Statements marked *author's recommendation* are not Microsoft statements.

Diagram: **Diagram 4 — Azure Resource Architecture** (`diagrams/04-azure-resource-architecture.drawio`).

## 1. Purpose and scope

This document describes how Azure resources are organized for the reference organization: subscription and resource groups, environment separation, hosting and data tiers, secrets, monitoring, backup, tagging, and cost control.

**In scope:** logical Azure structure, access boundaries, decision criteria for hosting and data, Key Vault, monitoring, backup, tagging, and cost governance.

**Out of scope:** a specific hosting or database product (ADR-004, deferred), executable templates (ADR-007), pricing, sizing, and the delivery pipeline (Phase 6).

## 2. Azure-specific assumptions

| ID | Assumption | Consequence |
|----|------------|-------------|
| Z-01 | One Azure subscription linked to the organization's Entra tenant (A-02, ADR-002) | Environments are separated by resource group, not by subscription |
| Z-02 | Application stack, traffic, and data volume are unknown (A-04) | Hosting and data are shown as generic tiers |
| Z-03 | One primary Azure region | Region choice depends on users, data residency, and service availability; it is not decided here |
| Z-04 | No Azure-hosted private network is required in the baseline (ADR-005) | No virtual network design unless a trigger applies (section 8) |
| Z-05 | Production data is not copied into Development or Staging without a documented approval | Keeps non-production environments out of the sensitive-data scope. *Author's recommendation* |

## 3. Subscription and resource group structure

**Baseline (ADR-002):** one subscription, with resource groups for each environment and a small set of shared resource groups.

| Resource group (illustrative) | Purpose | Contents |
|-------------------------------|---------|----------|
| `rg-app-dev` | Development environment | Hosting tier, data tier, storage, Key Vault |
| `rg-app-stg` | Staging environment | Same shape as Production, smaller where sensible |
| `rg-app-prod` | Production environment | Hosting tier, data tier, storage, Key Vault |
| `rg-shared-ops` | Shared operations | Log Analytics workspace, Application Insights, backup vault, alert rules |

Notes:

- Microsoft states that subscriptions are units of management, billing, and scale, and that they bound scale, quota, cost, governance, security, and identity controls. It also states that development environments typically have more relaxed policies than production, and that using a separate subscription is appropriate when a workload needs a different policy set. A single subscription is therefore a **starting point**, and the conditions that end it are listed in ADR-002.
- Microsoft states that a resource group's region should match the region of its resources and that one resource group should not hold resources from multiple regions.
- The structure repeats per application. A second application adds a new set of environment resource groups, not a new design.

**Trade-off — one subscription versus one per environment**

| Option | Advantage | Disadvantage |
|--------|-----------|--------------|
| One subscription, resource groups per environment (chosen) | Low overhead; one cost view; simple RBAC scopes; easy to document | Quotas and some policies are shared; a subscription-level mistake or compromise affects every environment |
| One subscription per environment | Stronger isolation; different policies and owners per environment; separate quotas | More administrative work; cross-subscription connectivity must be designed; hard to justify at 15 people without a specific driver |

## 4. Logical separation versus complete security isolation

This distinction matters in design reviews and interviews.

| | Logical separation (this design) | Complete security isolation |
|--|----------------------------------|-----------------------------|
| What it is | Resource groups, RBAC scopes, tags, naming, and policies that keep environments apart in management | Separate subscriptions (or tenants) with their own identities, policies, quotas, and network boundaries |
| What it prevents | Accidental changes across environments; routine over-broad access | Compromise or misconfiguration in one environment affecting another at the platform level |
| What it does **not** prevent | A subscription-level administrator or compromised high-privilege identity reaching every environment; shared quotas being exhausted; shared subscription-level settings | Not applicable (but cost and effort are higher) |

**Consequence for this design:** separation depends on correct role assignment at resource-group scope. Azure RBAC is additive: Microsoft's documentation shows that a Contributor assignment at subscription scope makes a narrower Reader assignment beneath it irrelevant. Subscription-scope roles must therefore be rare (ADR-014).

## 5. Environment characteristics

| Aspect | Development | Staging | Production |
|--------|-------------|---------|------------|
| Purpose | Build and early testing | Validate the release candidate before production | Serve users |
| Shape | Smallest that works | As close to Production as is justified | Sized to requirements (not defined here) |
| Human access | Developers and leads can change resources | Leads can change; developers deploy through the pipeline | No standing change rights for developers; changes through the pipeline after approval |
| Data | Synthetic or anonymized (Z-05) | Synthetic or anonymized | Real data |
| Secrets | Own Key Vault | Own Key Vault | Own Key Vault |
| Monitoring | Basic | Full, to test alerts | Full, with alerting and review |
| Backup | Not required unless data has value | As needed to rehearse restore | Required; restore-tested |
| Cost control | May be stopped or scaled down outside working hours where the product allows | As needed | Always on as required |

Sizing, availability targets, and retention are **not defined** here. They depend on business requirements (A-06).

## 6. Application hosting and data tiers (deferred)

ADR-004 defers the choice of product. The diagram therefore shows a generic **hosting tier** and **data tier**.

Microsoft's compute decision guidance describes a flowchart that is "only a starting point" and maps needs to services: full control to Virtual Machines, managed web hosting to App Service, event-driven work to Azure Functions, and containers to Container Apps, Container Instances, or AKS. The comparison criteria include team skills and operational overhead, networking, DevOps support, scalability, security, and cost. The page also notes that all listed services are regional.

A neutral comparison and a decision procedure are in [`../decisions/technology-comparison.md`](../decisions/technology-comparison.md).

Constraints this design places on whichever product is selected:

1. It must support deployment from a **single immutable build artifact** promoted through the three environments (Phase 6).
2. It must integrate with **managed identities** so that applications do not hold stored credentials (section 7).
3. It must send logs and metrics to the shared monitoring workspace (section 9).
4. Its data must be **recoverable** in a way that can be restore-tested (section 10).
5. It should not require a private network unless a trigger in section 8 applies.

## 7. Secrets management with Key Vault

Microsoft describes Azure Key Vault as a service for managing **secrets** (tokens, passwords, certificates, API keys), **encryption keys**, and **certificates**. It states that applications should not need to keep security information in code, and that access is authenticated through Microsoft Entra ID and authorized through **Azure RBAC** or a Key Vault access policy.

| Decision | Design |
|----------|--------|
| Vault granularity | One Key Vault **per environment** (`kv-app-dev`, `kv-app-stg`, `kv-app-prod`). *Author's recommendation.* Microsoft's page describes a vault per application, restricted to a specific application and team, and does not address environments |
| Authorization model | Azure RBAC for both vault management and data access, assigned to groups and application identities at vault scope |
| Application access | Managed identities with the narrowest role (for example, read secrets only) |
| Human access | Developers have no access to the production vault. Cloud administrators manage the vault; secret values are read only when required and the access is logged |
| Content | Application secrets, keys, and certificates only. **Not** employee passwords (security policy P-08) |
| Source control | No secrets in source code, tickets, or chat. Exposed secrets are rotated |

**Trade-off — one vault per environment versus one shared vault**

| Option | Advantage | Disadvantage |
|--------|-----------|--------------|
| Per environment (chosen) | RBAC boundary matches the environment boundary; Production secrets never share a scope with Development access | More vaults to manage; small extra operational work |
| One shared vault | Fewer objects | One scope covers every environment; harder to restrict by environment; mistakes can expose Production secrets |

Vault tier (Standard or Premium) and recovery protections such as soft delete are **not assessed here** and must be verified when the vault is designed. Vault pricing is not stated.

## 8. Networking in Azure (only when required)

The baseline has **no** virtual network design. Many managed Azure services are reachable over the internet and are protected by identity, TLS, and service-level access controls.

| Trigger | Response |
|---------|----------|
| A resource must not be reachable from the internet | Private connectivity for that resource, designed and recorded in a new ADR |
| Office staff need private access to Azure resources | Evaluate options and trade-offs in a new ADR; VPN is not assumed (ADR-005) |
| A hosting product requires a virtual network | Design the minimum network for that product |
| Connecting the office and Azure | Plan non-overlapping address ranges first (see [`network-architecture.md`](network-architecture.md), section 11). Microsoft states that virtual networks cannot span subscriptions, which matters if a multi-subscription split is later adopted |

Network security groups, firewalls, and private endpoints are not selected in the baseline.

## 9. Monitoring

| Component | Role |
|-----------|------|
| Azure Monitor | Platform metrics, activity logs, and alerts |
| Log Analytics workspace | Central store for logs and queries. Located in `rg-shared-ops` |
| Application Insights | Application telemetry, **where the application type and hosting product support it** |
| Entra sign-in and audit logs | Sent to the workspace so identity events are retained and alertable (security policy P-09) |
| Alerts and action groups | Notify the responsible role; defined in Phase 7 |

Log Analytics facts from Microsoft's documentation: tables retain data for **30 days by default** (some log tables 90 days); analytics retention can be extended up to **730 days** at extra cost; total retention can reach **12 years** using a lower-cost long-term state; **31 days** of analytics retention are included in the ingestion price; and charges depend on data volume and retention days. Per-GB prices are not stated here.

**Trade-off — one workspace versus one per environment**

| Option | Advantage | Disadvantage |
|--------|-----------|--------------|
| One shared workspace (baseline) | One place to query; simpler to operate and to alert on | Access to logs must be controlled carefully so developers do not read Production logs without need; mixed volumes complicate cost attribution |
| One workspace per environment | Access and retention can differ per environment; clearer costs | More objects to manage; cross-environment queries are harder |

The baseline uses one shared workspace. Revisit when access separation by environment is required. Retention periods and alert thresholds are not defined here; they are set in `operations/monitoring-and-alerting.md` (Phase 7, not yet written).

## 10. Backup and recovery for Azure workloads

Microsoft's Azure Backup overview names these protectable workloads: Azure Virtual Machines and Disks; SQL Server, SAP HANA, SAP ASE, PostgreSQL, MySQL Flexible Server, and Azure Cosmos DB; Azure Files, Blobs, and Data Lake Storage; and AKS. It describes soft delete, immutable backup, role-based access control, and storage of backup data in a Recovery Services vault isolated from the production environment.

| Point | Design |
|-------|--------|
| Scope | Only the workloads that the selected hosting and data products actually use. Coverage must be checked against the support matrix after ADR-004 is resolved |
| **Not covered by Azure Backup** | The page does **not** list Exchange Online, SharePoint Online, or OneDrive. Microsoft 365 data needs a separate approach (ADR-006) |
| Protection of backups | Use soft delete and immutability where available; give backup administration to a restricted group. The configuration details are not described on the overview page and must be verified |
| Where backups live | A backup vault in `rg-shared-ops`, separate from the resource groups it protects |
| Proof of recovery | A **successful restore test** is the only proof. Test cadence is defined in Phase 7 |
| Objectives | RPO and RTO are **not defined** here and must come from business requirements (A-06) |

A successful backup job does not prove that recovery is possible.

## 11. Tagging and naming

*Author's recommendation.* The sources reviewed for this phase did not cover tagging, so this section is design guidance rather than a cited Microsoft rule. Verify against current Azure tagging and naming guidance before use.

| Tag (illustrative) | Example value | Purpose |
|--------------------|---------------|---------|
| `environment` | `dev`, `stg`, `prod` | Filtering, cost views, policy |
| `application` | `app` | Group resources of one application |
| `owner` | A role or group, not a personal name | Who is accountable |
| `cost-center` | `engineering` | Cost allocation |
| `data-classification` | `internal`, `confidential` | Handling expectations |

Naming pattern (illustrative): `<type>-<application>-<environment>`, for example `rg-app-prod`, `kv-app-stg`. Keep names consistent and short. Some Azure resource types have naming restrictions that must be checked per type.

Rules: every resource group and resource carries the minimum tags; tags are checked during the periodic review. Enforcement through Azure Policy is an optional enhancement and is not assumed.

## 12. Cost governance

| Practice | Detail |
|----------|--------|
| Visibility | Review cost by resource group and tag on a regular cadence |
| Budgets | Microsoft documents that budgets **only alert**; "resources aren't affected, and your consumption isn't stopped". Budgets are supported at subscription and resource-group scope and can trigger action groups. Whether budgets themselves carry a charge is not stated on the page |
| Ownership | Each environment has a named owner role who responds to budget alerts |
| Right-sizing | Review utilization before increasing size; scale Development and Staging down when idle where the product allows |
| Deferred or optional | Additional environments, geo-redundancy, premium tiers, long log retention, and extended backup retention are optional and have cost; each requires a requirement behind it |

| Capability | Core | Optional or deferrable | Cost note |
|------------|------|------------------------|-----------|
| Resource groups, tags, Azure RBAC | Yes | | Microsoft states Azure RBAC is free and included with an Azure subscription |
| Hosting and data tiers | Yes (product deferred) | | Cost depends on the product and size |
| Key Vault | Yes | | Pricing not stated here |
| Monitoring | Yes | Long retention, extra tables | Depends on data volume and retention days |
| Azure Backup | Yes for data with value | Extended retention, archive | Depends on protected size and retention |
| Virtual network, firewall, private endpoints | No | Add when a trigger applies | Not assessed |
| Additional subscriptions | No | Add when a driver applies | Not assessed |

No prices are stated or implied.

## 13. Access model for Azure (summary)

Role assignments follow [`../security/access-control-matrix.md`](../security/access-control-matrix.md) and ADR-014:

- Roles are assigned to **groups**, at **resource-group scope**.
- A small number of administrative accounts hold the roles needed to create resource groups and manage role assignments. Their count is kept minimal.
- Deployment from the pipeline uses a **managed identity or service principal** scoped to the target resource group (Phase 6).
- Developers cannot change Production directly.

## 14. Growth path

| Trigger | Extension |
|---------|-----------|
| Second application or product | Repeat the environment resource-group set; share `rg-shared-ops` |
| Different owners, policies, or compliance for Production | Move Production to its own subscription, grouped under a management group (ADR-002) |
| Private connectivity needed | Virtual network design under a new ADR |
| Need to enforce standards | Azure Policy for tags, allowed regions, and allowed resource types |
| More automation | Infrastructure as code (Bicep) with reviewed changes (ADR-007, Phase 6) |
| Larger team | Separate cloud operations from application ownership |

## 15. Limitations

- Nothing has been created in Azure, and no cost, quota, or capacity has been checked.
- The hosting product, data product, region, size, and availability targets are open.
- Backup and Key Vault protections are described at feature level; their configuration is not verified.
- Tagging and naming guidance is the author's recommendation.

## 16. Well-Architected quick check (self-review, not an official assessment)

| Pillar | Observation |
|--------|-------------|
| Reliability | Backup and restore testing required; recovery objectives deliberately undefined; no redundancy assumed |
| Security | Identity-centred access, per-environment vaults, managed identities, no standing production rights; logical (not complete) isolation acknowledged |
| Cost Optimization | Single subscription, deferred optional services, budgets and tags; no prices claimed |
| Operational Excellence | Consistent naming and tags, shared monitoring, documented ownership; IaC as a concept |
| Performance Efficiency | No sizing claims; hosting choice deferred until requirements exist |

## 17. Sources

Accessed 2026-11-13. Recommendations beyond these statements are the author's own.

| Topic | Source |
|-------|--------|
| Subscriptions as units of management, billing, scale; separate subscriptions for different policy sets; resource group region | [Cloud Adoption Framework — Subscription considerations and recommendations](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/landing-zone/design-area/resource-org-subscriptions) |
| Azure RBAC scopes, additive permissions, no extra cost | [Microsoft Learn — What is Azure RBAC?](https://learn.microsoft.com/en-us/azure/role-based-access-control/overview) |
| Key Vault purpose and access control | [Microsoft Learn — Azure Key Vault overview](https://learn.microsoft.com/en-us/azure/key-vault/general/overview) |
| Log Analytics retention | [Microsoft Learn — Manage data retention in a Log Analytics workspace](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/data-retention-configure) |
| Azure Backup workloads and protections | [Microsoft Learn — What is Azure Backup?](https://learn.microsoft.com/en-us/azure/backup/backup-overview) |
| Compute service selection | [Azure Architecture Center — Choose an Azure compute service](https://learn.microsoft.com/en-us/azure/architecture/guide/technology-choices/compute-decision-tree) |
| Budgets alert only; supported scopes | [Microsoft Learn — Create and manage budgets](https://learn.microsoft.com/en-us/azure/cost-management-billing/costs/tutorial-acm-create-budgets) |
