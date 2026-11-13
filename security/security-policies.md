# Security Policies

> **Proposed, illustrative policy set.** These are example policies for a fictional 15-person organization, written to be adoptable and reviewable. They are not the policies of a real company and have not been applied. Licence-dependent items are labelled. See [`identity-management.md`](identity-management.md) for the supporting design and sources.

## 1. Purpose and how to read this document

This document turns the architecture into short, checkable rules. Each policy has an **owner**, a **baseline requirement**, and, where relevant, an **optional enhancement** that depends on licensing or additional tooling.

| Label | Meaning |
|-------|---------|
| **Baseline** | Expected in the reference design with no premium licence assumed |
| **Enhanced** | Needs a licence or tooling that must be confirmed first |
| **Owner** | The role accountable for keeping the policy true |

Terms such as "must" and "should" are used in their ordinary sense: **must** is required by this policy, **should** is a strong recommendation.

## 2. Policy summary

| # | Policy | Baseline | Enhanced | Owner |
|---|--------|----------|----------|-------|
| P-01 | Least privilege and access control | Group-based access; access matrix applied | Time-limited elevation (PIM) | IT Administrator |
| P-02 | Multi-factor authentication | Security defaults enabled | Conditional Access policies | IT Administrator |
| P-03 | Administrative account separation | Separate administrative accounts; two emergency accounts | PIM | IT Administrator |
| P-04 | Secure device configuration | Disk encryption, patching, antimalware, no daily local admin | Central device management | IT Administrator |
| P-05 | Patching and updates | Documented routine with timeframes | Managed update rings | IT Administrator |
| P-06 | Network segmentation | Staff, guest, and management VLANs; default-deny rules | Not required | IT Administrator |
| P-07 | Secure remote access | Identity-based access to cloud services; no VPN by default | Private connectivity only if required | IT Administrator and Cloud Administrator |
| P-08 | Secrets management | Application secrets in a key vault, never in source code | Automated rotation | Cloud Administrator |
| P-09 | Logging and audit | Sign-in, audit, and firewall logs retained and reviewed | Central log analytics | IT Administrator and Cloud Administrator |
| P-10 | Backup protection | Separate protection for Azure workloads and Microsoft 365 data; restore tests | Dedicated Microsoft 365 backup (if needed) | IT Administrator and Cloud Administrator |
| P-11 | Access reviews | Scheduled reviews | Automated access reviews | Management |
| P-12 | Joiner, mover, leaver | Documented checklist | Automation | IT Administrator and Management |
| P-13 | Acceptable use and data handling | Short staff rules; company data on company-managed devices | Data-loss prevention tooling | Management |

## 3. Policies

### P-01 Least privilege and access control

- Access is granted according to the [access control matrix](access-control-matrix.md) and through **groups**.
- No developer holds standing administrator access to Production.
- Azure roles are assigned at the narrowest practical scope, because role assignments at a higher scope apply to everything beneath it.
- Exceptions are time-limited, recorded, and removed afterwards.

### P-02 Multi-factor authentication

- All users **must** use MFA. In the baseline this is enforced by **security defaults**, which Microsoft documents as available at no extra cost.
- Legacy authentication is blocked (security defaults do this).
- Where Conditional Access is licensed and adopted, it **replaces** security defaults, and the equivalent protections must be recreated explicitly before switching, including MFA for administrators, MFA for all users, and blocking of legacy authentication.
- Phishing-resistant methods are preferred where supported.

### P-03 Administrative account separation

- Administration is performed only with a **separate administrative account**.
- Administrative accounts do not read email or browse the web for daily work.
- The number of accounts with the highest-privilege role is kept very small.
- Two **emergency access accounts** exist, are excluded from sign-in-blocking policies where the platform allows, are monitored for any use, and are tested at least every 90 days, following Microsoft's guidance.

### P-04 Secure device configuration

- Company laptops **must** have disk encryption enabled, with recovery keys stored in a controlled, documented location.
- Built-in antimalware **must** be enabled and up to date.
- Employees **must not** use a local administrator account for daily work.
- A device inventory records owner, serial number, and status.
- A lost or stolen device is reported immediately; sessions are revoked and the account is secured according to the lost-device procedure.

### P-05 Patching and updates

- Operating system and application updates are applied on a documented routine. Target timeframes are set by the organization; none are invented here.
- Network device firmware is tracked and updated, using vendor-validated sources. This follows the CISA guidance on securing network infrastructure devices cited in the network document.
- Critical security updates are applied outside the routine when a vendor or authority marks them urgent.

### P-06 Network segmentation

- Guest and personal devices **must** use the guest network, which has internet access only.
- Management interfaces of network devices are reachable only from the management segment and a designated administration workstation.
- Inter-segment traffic is denied by default; each allowed flow is explicit. VLANs alone are not security; the firewall rules and switch configuration enforce the isolation. Detail is in [`../architecture/network-architecture.md`](../architecture/network-architecture.md).

### P-07 Secure remote access

- Employees reach Microsoft 365 and Azure over the internet using their identity and MFA.
- A VPN is **not** a universal requirement. It is introduced only if a resource actually requires private network connectivity (ADR-005).
- Network devices are not administered from the internet.

### P-08 Secrets management

- Application secrets, keys, and certificates **must not** be stored in source code, tickets, or chat.
- They are kept in a key vault service and accessed through managed identities or tightly scoped permissions (Azure design in Phase 5).
- Azure Key Vault is for **application** secrets. It is **not** an employee password manager; employees use an approved password manager.
- Any secret that is exposed is treated as compromised and rotated.

### P-09 Logging and audit

- Sign-in logs, audit logs, and firewall logs are retained and reviewed on a schedule.
- Any sign-in by an emergency access account triggers an alert and a documented review.
- Retention periods and alert thresholds are defined in the operations documents. They are not invented here.

### P-10 Backup protection

- Azure workloads and Microsoft 365 data are protected **separately**. Azure Backup is not assumed to protect Exchange Online, SharePoint Online, or OneDrive (ADR-006).
- Backups are protected from unauthorized change or deletion, with separate administrative rights where practical.
- A backup is considered proven only after a **successful restore test**.
- Recovery objectives must be derived from business requirements and are not defined in this portfolio (A-06).

### P-11 Access reviews

- Administrative roles, group memberships, Azure role assignments, guest accounts, and emergency accounts are reviewed on the schedule in the identity document.
- Findings and removals are recorded.

### P-12 Joiner, mover, leaver

- Each change follows the checklists in [`../operations/onboarding-offboarding.md`](../operations/onboarding-offboarding.md).
- A leaver's access is removed on the **last working day, or earlier if risk requires**, and shared secrets the person knew are rotated (including the staff Wi-Fi passphrase, ADR-009).

### P-13 Acceptable use and data handling

- Company data is handled only on company-managed devices and approved services.
- Staff report suspected phishing, lost devices, or unusual account activity immediately.
- Staff must not share accounts or credentials.
- Content of this policy for staff should be short, plain, and acknowledged on joining.

## 4. Baseline versus licence-dependent controls

| Control | Baseline | Needs confirmation |
|---------|----------|--------------------|
| Security defaults (MFA, legacy authentication blocking) | Yes (no licence required, per Microsoft) | — |
| Conditional Access | No | Requires at least Microsoft Entra ID P1; Microsoft also states Business Premium includes it |
| Risk-based policies | No | Microsoft Entra ID P2 |
| Privileged Identity Management | No | Microsoft Entra ID P2 or Entra ID Governance |
| Intune-based device management | No | Intune inclusion depends on the licence; not asserted for Business Premium |
| Azure RBAC | Yes | Free and included with an Azure subscription, per Microsoft |
| Firewall segmentation and guest isolation | Yes | Depends on firewall capability (N-03) |

Statements are as documented on 2026-11-13 and must be re-verified before use.

## 5. Policy governance

| Item | Rule |
|------|------|
| Ownership | Each policy has a named owner role (table in section 2) |
| Review | Reviewed at least yearly and after any significant incident or architecture change |
| Exceptions | Written, time-limited, approved by management, and tracked |
| Change | Changes follow the change management process (Phase 7) |
| Evidence | Adherence is shown by access reviews, logs, and restore-test records, not by assertion |

## 6. Limitations

- These are example policies. A real organization must adapt them to its legal, contractual, and risk context.
- No compliance framework is mapped (A-05).
- Timeframes, retention, and thresholds are deliberately not set here; they depend on business requirements.
