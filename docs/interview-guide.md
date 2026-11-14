# Interview Guide

> **Purpose:** help the author explain this portfolio honestly in an interview. It is a design portfolio, **not** evidence of deployment or production experience. Nothing in the repository has been built, tested, or operated. Every answer below should keep that distinction clear.
> Documents referenced are in this repository; decisions are in [`../decisions/architecture-decisions.md`](../decisions/architecture-decisions.md).

## 1. How to describe the project

**Thirty-second version.** "This is a documentation-first architecture portfolio for a fictional 15-person software company. I designed the identity, office network, Microsoft 365, Azure, DevOps, and operations layers on paper, recorded 20 decisions with trade-offs, and used official vendor documentation with access dates. I did not deploy anything, so I describe it as a design, not as experience."

**Two-minute version.**

1. *Why documentation-first.* I could not use paid cloud services or build a lab, and I did not want to present untested code as proven. Design reasoning, trade-offs, and operating processes are things I could demonstrate honestly.
2. *What it covers.* A reference scenario with explicit assumptions; a main architecture; an office network with three VLANs; identity and security; one Azure subscription with environment resource groups; a DevOps flow that promotes one artifact; and operations (backup, monitoring, incidents, change, governance).
3. *How decisions are recorded.* Each ADR has context, options, decision, reasoning, advantages, disadvantages, risks, and revisit conditions. Some decisions are deliberately **deferred** (hosting and database product, dedicated Microsoft 365 backup).
4. *What I would do next.* Validate the design in a real or trial environment, define recovery objectives with the business, and verify the licence-dependent features.

## 2. Honest framing

| Question | Suggested answer |
|----------|------------------|
| "Did you deploy this?" | No. It is a design. Nothing was built, tested, or validated, and the repository says so. |
| "Do you have production experience with this?" | This portfolio does not claim any. It shows how I reason about design, risk, and operations. |
| "Why are RPO and RTO missing?" | They must come from business requirements. Inventing numbers would be misleading, so the document gives the process for setting them. |
| "Why no prices?" | Prices and licence terms change; I would verify them at decision time and cite the source and date. |
| "Why is the hosting product undecided?" | The scenario does not contain the requirements that would decide it (ADR-004). The design stays valid for several products. |
| "What would you verify first?" | Licence-dependent features (Conditional Access, Intune, Entra log export), then recovery objectives and the Microsoft 365 backup decision. |

## 3. Talking points by domain

### 3.1 Identity and security

- **Idea:** identity is the main security perimeter in a cloud-first company with no on-premises servers.
- **Key decisions:** Entra ID as single identity source (ADR-003); security defaults as the MFA baseline and Conditional Access as a licence-dependent option (ADR-010); separate admin accounts and two emergency access accounts (ADR-012).
- **Trade-offs:** security defaults are simple and free but less flexible than Conditional Access; separate admin accounts add friction but limit damage from a stolen daily account.
- **Be ready to explain:** the difference between Entra roles, Microsoft 365 roles, and Azure RBAC, which are separate permission systems.
- **Be careful:** do not assume Conditional Access, PIM, or Intune are available; the documents mark them as licence-dependent or unverified.

### 3.2 Office network

- **Idea:** three VLANs (Staff, Guest, Management) with the firewall as the single policy point (ADR-008).
- **Trade-offs:** simple and understandable versus finer segmentation; shared-passphrase Wi-Fi is easy but weaker than 802.1X, which is documented as the upgrade path (ADR-009); no VPN by default (ADR-005).
- **Be ready to explain:** why VLANs alone are not security, and why guest isolation needs firewall rules.

### 3.3 Azure

- **Idea:** one subscription with a resource group per environment plus a shared operations group (ADR-002).
- **Trade-offs:** resource groups give logical separation, not full isolation; a separate subscription for Production is a documented growth path.
- **Key decisions:** one Key Vault per environment with managed identities (ADR-013); group-based roles at resource-group scope (ADR-014).
- **Be ready to explain:** budgets alert but do not stop spending; tagging is my own recommendation.

### 3.4 DevOps

- **Idea:** short-lived branches into a protected main (ADR-015); build once, promote one artifact through Development and Staging to Production with approval by someone other than the requester (ADR-016); federated deployment identity instead of stored secrets (ADR-017).
- **Trade-offs:** approval gates slow releases but add a human check; rollback by redeploying the previous artifact works with any product, while slot swap is product-specific.
- **Be careful:** Bicep appears only as a concept (ADR-007). No template was run, so none is presented as working.

### 3.5 Operations

- **Backup and recovery:** a backup is proven only by a successful restore. Azure Backup does not cover Microsoft 365 data (ADR-006), and the dedicated Microsoft 365 backup choice is deferred until objectives exist (ADR-018).
- **Monitoring:** a small catalogue of alerts, each with an owner, route, and first action; no thresholds invented (ADR-020).
- **Incidents and change:** a simple lifecycle aligned with NIST SP 800-61 Rev. 3, three change types that reuse existing approval gates (ADR-019).
- **Governance:** ownership with backups, resource lifecycle, a cost review routine, and a runbook standard.

## 4. Likely questions and short answers

| Question | Short answer |
|----------|--------------|
| Why Entra ID security defaults rather than Conditional Access? | They are free and need no design effort. Conditional Access gives control but depends on licensing, which I did not assume (ADR-010). |
| What is the biggest risk in your design? | Compromise of an administrative account. Controls are separate admin accounts, emergency accounts, restricted subscription roles, alerts on role changes, and regular role reviews. |
| How would you handle a leaked secret? | Rotate immediately, check usage logs, fix the process gap, and add secret detection (incident response, section 7.2). |
| What happens if a release breaks Production? | Decide rollback or roll forward, redeploy the previous artifact, verify, communicate, and open a problem record if the cause is unclear. |
| How do you recover a deleted SharePoint file? | Recycle bin first (93 days documented); Files Restore for bulk changes (30 days). Beyond that, built-in protections end, which is why the backup decision exists. |
| How would the design grow? | Each document has a growth-path table: more VLANs or 802.1X, a separate Production subscription, Conditional Access and Intune, on-call and central monitoring. |
| What would you do differently with a budget? | Build a trial environment to test the design, then replace "proposed" with "implemented" only where I verified it. |

## 5. Trade-off summary

The one-page comparison of every decision is in [`../decisions/technology-comparison.md`](../decisions/technology-comparison.md), section 6. Use it to answer "why this and not that".

## 6. Statements to avoid

- "I built" or "I deployed" anything from this repository.
- Any claim about performance, uptime, cost, or recovery time. No such numbers exist here.
- Treating a licence-dependent feature as available.
- Describing the design as the only correct one. It is a reasoned proposal for stated assumptions.
