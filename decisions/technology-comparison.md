# Technology Comparison and Decision Framework

> **Illustrative comparison for a fictional organization.** This document does **not** select an application hosting or database product. It exists because ADR-004 defers that decision until requirements are known. Facts about Azure services are taken from official Microsoft documentation **accessed 2026-11-13**; judgments are the author's and are labelled. No prices, sizes, or performance figures are given. Related: [`architecture-decisions.md`](architecture-decisions.md), [`../architecture/cloud-architecture.md`](../architecture/cloud-architecture.md).

## 1. How to use this document

1. Gather the **inputs** in section 2 for the real application.
2. Use the **criteria** in section 3 to narrow the options.
3. Check the candidate against the **constraints** that the architecture already imposes (section 4).
4. Record the choice in a new ADR that supersedes the deferral in ADR-004.

Microsoft's own guidance states that its compute flowchart is "only a starting point": the candidate must still be evaluated against the real needs.

## 2. Inputs that are currently unknown (A-04)

| Input | Why it changes the answer |
|-------|---------------------------|
| Runtime and framework | Determines which hosting products can run it without rework |
| Application shape (web app, API, background jobs, event-driven, containerized) | Maps to different service types |
| Traffic pattern and peak load | Drives scaling needs and cost |
| Data model and volume | Drives the data product and backup approach |
| Team skills and size | A small team favors lower operational overhead |
| Availability and recovery needs (RPO/RTO) | Business requirement; not defined in this portfolio |
| Compliance or data-residency rules | May restrict regions or require isolation |
| Need for private network connectivity | Triggers a network design (ADR-005) |

## 3. Hosting options

Characteristics below are from Microsoft's compute guidance unless marked as the author's judgment.

| Option | Type | What Microsoft states | Fits when | Main trade-off (author's judgment where noted) |
|--------|------|------------------------|-----------|------------------------------------------------|
| Azure Virtual Machines | IaaS | Most control, flexibility, and portability; you manage the VMs and networking; closest to on-premises | Full OS control or lift-and-shift is needed | Highest operational burden: patching, hardening, backup, and scaling are yours |
| Azure App Service | PaaS | Managed environment for web apps, back ends, and REST APIs; supports deployment slots | Standard web application or API with a small team | Less control over the host; fits only supported runtimes and patterns |
| Azure Container Apps | Serverless containers | Built on Kubernetes and serverless, but does not expose the underlying Kubernetes APIs | The application is containerized and orchestration details are not needed | Requires a container build and registry practice |
| Azure Functions | FaaS | Runs code triggered by events; Consumption and Flex Consumption plans are serverless | Event-driven or short-running work | Not a natural fit for long-running, stateful web workloads (author's judgment) |
| Azure Kubernetes Service | Managed Kubernetes | Direct access to the Kubernetes API and control plane | The team needs Kubernetes itself | High complexity for a 15-person organization; excluded by default (ADR-005) |

Criteria Microsoft lists for comparison include team skills and operational overhead, networking, DevOps, scalability, multiregion options, security, TLS, cost, GPU support, and suitable architecture styles. All listed services are regional, so multiregion designs need an external router or load balancer.

**Author's decision heuristic for this organization:** start from the option with the **lowest operational overhead that runs the application correctly**, and move toward more control only when a specific requirement demands it. This follows the architecture principles of proportionality and avoiding overengineering. It is not a selection.

## 4. Constraints from the existing architecture

Whichever product is selected must:

| # | Constraint | Source |
|---|------------|--------|
| 1 | Deploy one immutable artifact through Development, Staging, and Production | Overview; Phase 6 |
| 2 | Use managed identities instead of stored credentials | Cloud architecture, section 7 |
| 3 | Send logs and metrics to the shared monitoring workspace | Cloud architecture, section 9 |
| 4 | Have a recoverable data design that can be restore-tested | ADR-006; cloud architecture, section 10 |
| 5 | Run without a private network unless a trigger applies | ADR-005 |
| 6 | Support the access model (resource-group-scoped roles, no standing production access) | ADR-014 |

## 5. Data tier options

The data tier is chosen **after** the data model is known. Microsoft's Azure Backup overview names these data services among supported workloads: SQL Server, SAP HANA, SAP ASE, PostgreSQL Server, PostgreSQL Flexible Server, MySQL Flexible Server, Azure Cosmos DB, Azure Files, Azure Blobs, and Azure Data Lake Storage.

| Question | Why it matters |
|----------|----------------|
| Is the data relational with transactions and joins, or document or key-value? | Selects the product family |
| Does the application already depend on a specific database engine or dialect? | Rework cost |
| Is a managed service available for that engine, or is self-hosting required? | Operational burden |
| How is the data protected and how is it restored? | Backup coverage must be confirmed in the support matrix for the exact product |
| What are the recovery objectives? | Not defined here |
| How is access controlled and how are secrets handled? | Managed identity preferred; secrets in Key Vault |

No data product is recommended here. Managed services are preferred by default over self-managed servers on virtual machines because they reduce patching and backup burden, but this is the author's judgment and each choice needs verification of feature and backup support.

## 6. Decision log summary (decisions already made)

For quick reference. Details and reasoning are in the ADRs.

| Area | Chosen | Main alternative | Key trade-off | ADR |
|------|--------|------------------|---------------|-----|
| Subscription model | One subscription, resource group per environment | Subscription per environment | Low overhead versus stronger isolation | ADR-002 |
| Identity source | Entra ID as the single source | Separate identity stores | One place for joiners and leavers versus per-system autonomy | ADR-003 |
| MFA | Security defaults baseline | Conditional Access | No licence needed versus precise control | ADR-010 |
| Devices | Documented manual baseline | Intune | No licence needed versus central enforcement | ADR-011 |
| Administration | Separate admin accounts, emergency accounts | Daily accounts with admin roles | Safety versus convenience | ADR-012 |
| Network segmentation | Three VLANs, firewall policy point | Flat network or many VLANs | Simplicity versus finer control | ADR-008 |
| Staff Wi-Fi | Shared passphrase | 802.1X | Simplicity versus per-user control | ADR-009 |
| Remote access | No VPN by default | VPN for everything | Lower cost versus private connectivity | ADR-005 |
| Hosting and database | **Deferred** | Pick now | Honesty versus a more concrete diagram | ADR-004 |
| Microsoft 365 backup | Separate from Azure backup | One generic backup | Visible gaps versus less documentation | ADR-006 |
| IaC | Concept only | Full templates | Honesty versus demonstrated syntax | ADR-007 |
| Key Vault | One per environment | One shared vault | Strong boundary versus fewer objects | ADR-013 |
| Azure roles | Group-based, resource-group scope, minimal subscription roles | Broad subscription roles | Isolation versus convenience | ADR-014 |
| Branching | Short-lived branches, protected main | Branch per environment, GitFlow | Simplicity and build-once versus structure | ADR-015 |
| Releases | One immutable artifact, approval before Production | Rebuild per environment, auto-deploy to Production | Safety versus speed | ADR-016 |
| Pipeline identity | Federated credential per environment | Stored service-principal secret | No long-lived secret versus simpler setup | ADR-017 |

## 7. Limitations

- No product has been tested or priced.
- Service characteristics may change; re-check Microsoft documentation before deciding.
- The heuristics are the author's and are not an official recommendation.

## 8. Sources

Accessed 2026-11-13.

| Topic | Source |
|-------|--------|
| Compute service characteristics and criteria | [Azure Architecture Center — Choose an Azure compute service](https://learn.microsoft.com/en-us/azure/architecture/guide/technology-choices/compute-decision-tree) |
| Data workloads that Azure Backup supports | [Microsoft Learn — What is Azure Backup?](https://learn.microsoft.com/en-us/azure/backup/backup-overview) |
