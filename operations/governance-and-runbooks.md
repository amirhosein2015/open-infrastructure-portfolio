# Governance, Ownership, and Runbooks

> **Proposed, illustrative governance model** for a fictional 15-person organization. Cadences are examples unless a vendor source is cited. Assumption IDs (A-nn) refer to [`../architecture/reference-scenario.md`](../architecture/reference-scenario.md); decisions are in [`../decisions/architecture-decisions.md`](../decisions/architecture-decisions.md).

## 1. Purpose and scope

This document defines who owns what, how resources and documentation are kept under control throughout their life, how cost is governed, which recurring tasks keep the architecture healthy, and how runbooks are written.

**In scope:** technical ownership, resource lifecycle, cost governance routine, documentation governance, periodic architecture review, recurring task calendar, and the runbook standard.

**Out of scope:** pricing, tool selection, and HR or legal processes.

## 2. Technical ownership

Every system has a **named owner role** who is accountable for it and a **backup owner** so that knowledge is not held by one person. Owners are roles (not personal names) in documents; the assignments are recorded separately.

| Area | Owner role (illustrative) | Backup | Accountable for |
|------|---------------------------|--------|-----------------|
| Identity (Entra ID), access reviews | IT Administrator | Cloud Administrator | Accounts, groups, MFA, emergency access |
| Microsoft 365 tenant | IT Administrator | Cloud Administrator | Settings, licensing, data protection settings |
| Office network | IT Administrator | Cloud Administrator | Devices, VLAN and IP plan, firewall rules, Wi-Fi |
| Company laptops | IT Administrator | Cloud Administrator | Inventory, encryption, updates, lifecycle |
| Azure subscription and shared operations | Cloud Administrator | IT Administrator | Resource groups, roles, monitoring, backup, cost |
| Development environment | Team lead | A developer lead | Resources, cost, data hygiene |
| Staging environment | Team lead | A developer lead | Parity with Production, test data |
| Production environment | Team lead with Cloud Administrator | Second lead | Availability, releases, recovery |
| Delivery pipeline and repositories | Team lead | Cloud Administrator | Branch protection, approvals, identities |
| Security policies | Management | IT Administrator | Policy content and exceptions |
| Backup and recovery objectives | Management | IT Administrator | Objectives table, restore test acceptance |

Ownership is shown in resource tags (`owner`) and in the register of runbooks.

## 3. Resource lifecycle

| Phase | Rules |
|-------|-------|
| **Request and create** | Created through the change process; named and tagged to the standard (cloud architecture, section 11); owner assigned; cost expectation stated |
| **Operate** | Monitored, backed up as required, patched, reviewed with its owner |
| **Review** | Quarterly: is it still needed, correctly sized, correctly tagged, correctly accessed |
| **Retire** | Owner approval; data-retention decision made; access removed; backups handled per retention; resources deleted; documentation and diagrams updated |

Specific rules:

- Untagged or ownerless resources are flagged for review and removed if nobody claims them.
- Development and Staging resources are reviewed first for savings; scale-down or shutdown outside working hours is considered where the product allows.
- Production deletion is restricted to the cloud administrator account and alerted (monitoring and alerting, AL-08).
- Leavers' accounts, devices, and data follow the offboarding checklist.

## 4. Cost governance routine

Policy design is in the cloud architecture (section 12). The routine is:

| Item | Detail |
|------|--------|
| Cadence | A regular review (illustratively monthly) |
| Inputs | Cost by resource group and tag; budget alerts; licence counts for Microsoft 365 and Entra |
| Participants | Cloud Administrator, IT Administrator, environment owners, Management for decisions |
| Questions | What changed? What is unused? What is oversized? What can be deferred? Do licences match active users? |
| Outputs | A short record of decisions and actions; budget adjustments |
| Budget alerts | Alert only; Microsoft states budgets do not stop resources, so a person decides what to do |
| Licences | Reconciled with the joiner and leaver records so unused licences are reclaimed |

No prices or targets are stated in this portfolio.

## 5. Documentation governance

| Rule | Detail |
|------|--------|
| Source of truth | The repository holds the architecture documents and diagram sources |
| Change path | Pull request with review; reviewer checks consistency with other documents |
| Status labels | Every document states whether content is proposed, recommended, conceptual, illustrative, or implemented |
| Diagrams | Source files (`.drawio`) and exported images are updated together |
| Updating with change | A change is complete only after the affected documents are updated (change management) |
| Review cadence | At least yearly, and after any significant incident or architecture change |
| Sensitive content | No real secrets, addresses, or employer information in the repository |
| Naming | Consistent file, resource, tag, and group naming as defined in each document |

## 6. Periodic architecture review

At least yearly, review the architecture against the Azure Well-Architected pillars (Reliability, Security, Cost Optimization, Operational Excellence, Performance Efficiency) as a practical checklist. This is a self-review and not a Microsoft assessment. Also review:

- Assumptions that may have changed (A-nn) and the conditions listed under "Revisit when" in each ADR.
- Licences and the features that depend on them.
- Whether growth triggers in each document have been reached.

## 7. Recurring task calendar (illustrative)

Cadences are examples except where a source is cited.

| Task | Cadence | Owner | Source or note |
|------|---------|-------|----------------|
| Review administrative role membership | Quarterly | Management with IT Administrator | Identity management |
| Review group and Azure role assignments | Quarterly | Group owners and Cloud Administrator | Identity management |
| Validate emergency access accounts | At least every 90 days and after staffing or subscription changes | IT Administrator | Microsoft's emergency access guidance (identity management) |
| Rotate the staff Wi-Fi passphrase | At offboarding of someone who knew it; periodically | IT Administrator | ADR-009 |
| Review firewall rules and switch port settings | Quarterly | IT Administrator | ADR-008 |
| Check firmware and patch status | Routine, with exceptions for urgent advisories | IT Administrator | Network and security policies |
| Review alerts, logs, and privileged activity | Monthly | IT and Cloud Administrators | Monitoring and alerting |
| Run restore tests | Set by data owners | Data owners | Backup and recovery |
| Rehearse a rollback in Staging | Periodically | Team lead | Infrastructure design |
| Tabletop incident exercise | Periodically | Incident lead | Incident response |
| Cost review | Monthly | Cloud Administrator | Section 4 |
| Review of policies and documents | Yearly | Management and owners | Section 5 |
| Review of the recovery objectives table | Yearly | Management | Backup and recovery |

## 8. Runbooks

A **runbook** is a short, tested set of steps for a specific recurring operation or response. It lets a different person do the task correctly.

### 8.1 Standard

| Rule | Detail |
|------|--------|
| One purpose per runbook | Easy to find and follow |
| Written to be used under stress | Short numbered steps, exact names, no assumed knowledge |
| Owned and dated | Owner, backup owner, last reviewed, last tested |
| Tested | A runbook is trusted only after someone other than the author has followed it successfully |
| Linked | From the alert, the document, or the incident playbook that uses it |
| Free of secrets | It says where a secret is kept, never the secret itself |
| Reviewed | After every use and at least yearly |

### 8.2 Template

```
# Runbook: <title>

Owner: <role>          Backup owner: <role>
Last reviewed: <date>  Last tested: <date> by <role>
Status: Proposed / Illustrative / Implemented

## Purpose
What this runbook achieves and when to use it.

## Preconditions
Access, accounts, tools, approvals needed. Which account to use.

## Steps
1. ...
2. ...
3. ...

## Verification
How to confirm it worked.

## Rollback or recovery
How to undo or recover if it fails.

## Escalation
Who to contact and when.

## Records
What to record, and where.

## Related documents
Links.
```

### 8.3 Runbook register (planned)

None of these has been written or tested, except the onboarding and offboarding checklists, which are procedures rather than tested runbooks.

| Runbook | Trigger | Owner role | Status |
|---------|---------|------------|--------|
| Onboard and offboard a user | Joiner or leaver | IT Administrator | Procedure drafted ([`onboarding-offboarding.md`](onboarding-offboarding.md)) |
| Respond to a compromised account | Alert or report | IT Administrator | Outline in incident response |
| Respond to a leaked secret | Alert or report | Cloud Administrator | Outline in incident response |
| Roll back a production release | Failed release | Team lead | Concept in infrastructure design |
| Restore a deleted file or mail item | User request | IT Administrator | Planned |
| Restore a SharePoint library or mailbox | Incident | IT Administrator | Planned |
| Restore an Azure data store | Incident | Cloud Administrator | Planned, depends on product |
| Rebuild an environment from definitions | Incident or exercise | Cloud Administrator | Planned |
| Use emergency access accounts | Administrator lockout | IT Administrator | Planned |
| Replace or restore a network device | Failure | IT Administrator | Planned |
| Handle a lost or stolen laptop | Report | IT Administrator | Planned |
| Rotate the staff Wi-Fi passphrase | Leaver or schedule | IT Administrator | Planned |

## 9. Growth path

| Trigger | Extension |
|---------|-----------|
| More teams or products | Ownership by product; a lightweight governance forum |
| Compliance or customer demand | Formal policy lifecycle, evidence, and audits |
| More automation | Runbooks converted into automated, tested workflows |
| Larger IT function | Separate roles for operations, security, and cloud platform |

## 10. Limitations

- The model has not been operated. Cadences are examples.
- Ownership assignments and runbook contents depend on the real people and tools.
- No tool is selected for registers, tickets, or runbook storage.
