# Onboarding and Offboarding

> **Proposed, illustrative procedures** for a fictional 15-person organization. They have not been executed or tested. Group and account names are examples. See [`../security/identity-management.md`](../security/identity-management.md) and [`../security/access-control-matrix.md`](../security/access-control-matrix.md) for the design behind them.

## 1. Purpose

Joining, changing role, and leaving are the moments when access most often goes wrong. This document defines repeatable checklists so access follows the person's role and is removed when it is no longer needed. It supports security policy **P-12**.

## 2. Principles

1. **Role first:** decide the person's role before creating anything. Access comes from group membership.
2. **One named account** per person; no shared accounts.
3. **Separate administrative accounts** only for people who administer.
4. **Evidence:** each step is recorded (who, what, when) so reviews have something to check.
5. **Remove on time:** a leaver's access is removed on the last working day, or earlier when risk requires.

## 3. Roles in the process

| Role | Responsibility |
|------|----------------|
| Hiring manager (Management) | Requests the change, states the role, start or end date, and required access |
| IT Administrator | Performs identity, device, and Microsoft 365 steps |
| Cloud Administrator | Performs Azure access steps |
| Team lead | Confirms technical access needs and confirms handover |
| The employee | Completes setup and acknowledges the acceptable use rules |

## 4. Onboarding (joiner) checklist

**Before the first day**

| # | Step | Owner |
|---|------|-------|
| 1 | Receive a written request with role, start date, and manager | Hiring manager |
| 2 | Create the user account with a standard naming convention | IT Administrator |
| 3 | Assign licences according to role (verify licence availability first) | IT Administrator |
| 4 | Add the user to the role's groups (for example `grp-all-staff`, `grp-developers`) | IT Administrator |
| 5 | For technical roles, add Azure access through the role's group; confirm scope (Development, Staging) | Cloud Administrator |
| 6 | Prepare the company laptop: encryption enabled, updates applied, antimalware active, no local admin for daily use | IT Administrator |
| 7 | Record the device in the inventory | IT Administrator |

**First day**

| # | Step | Owner |
|---|------|-------|
| 8 | Hand over the device; the user signs in and registers MFA | IT Administrator and employee |
| 9 | Explain acceptable use and reporting rules; the employee acknowledges them | IT Administrator |
| 10 | Provide the staff Wi-Fi passphrase through a secure channel | IT Administrator |
| 11 | Introduce the approved password manager | IT Administrator |
| 12 | Confirm the user can reach what the role needs and nothing more | Team lead |

**For people who will administer**

| # | Step | Owner |
|---|------|-------|
| 13 | Create a **separate administrative account** with only the needed roles | IT Administrator |
| 14 | Add it to the appropriate admin group; record the approval | Management |
| 15 | Add the person to the next access review | IT Administrator |

## 4a. Mover (change of role)

| # | Step | Owner |
|---|------|-------|
| 1 | Written request describing the new role and the effective date | Hiring manager |
| 2 | Add the new role's groups | IT Administrator |
| 3 | **Remove** the old role's groups and any temporary access | IT Administrator |
| 4 | Adjust licences if the role requires different ones | IT Administrator |
| 5 | Update Azure access through groups | Cloud Administrator |
| 6 | Record the change | IT Administrator |

Access should be *replaced*, not *accumulated*. A mover who keeps all old access is the usual cause of excess privilege.

## 5. Offboarding (leaver) checklist

**Timing:** access is removed on the last working day, or earlier if there is a risk (for example, a dismissal with a conflict or a suspected security issue). Steps 1 to 5 can be performed in advance and scheduled.

| # | Step | Owner |
|---|------|-------|
| 1 | Receive a written notice with the leaving date and whether it is a risk case | Hiring manager |
| 2 | Identify data, mail, shared files, and responsibilities that need a new owner | Team lead |
| 3 | Transfer ownership of files, shared mailboxes, and repositories | Team lead and IT Administrator |
| 4 | Record any application secrets, service accounts, or shared credentials the person knew | IT Administrator and Cloud Administrator |
| 5 | Retrieve the company laptop and any accessories on or before the last day | IT Administrator |

**On the leaving date**

| # | Step | Owner |
|---|------|-------|
| 6 | **Block sign-in** for the account | IT Administrator |
| 7 | **Revoke active sessions** so existing sign-ins end | IT Administrator |
| 8 | Remove the user from all groups, including Azure access groups | IT Administrator and Cloud Administrator |
| 9 | Disable and later remove any **separate administrative account** | IT Administrator |
| 10 | Remove or reassign the licence only **after** the data-retention decision (step 12) | IT Administrator |
| 11 | **Rotate shared secrets** the person knew: staff Wi-Fi passphrase (ADR-009), shared credentials, and any application secrets found in step 4 | IT Administrator and Cloud Administrator |
| 12 | Apply the data-retention decision; delete or archive the account after the agreed period | IT Administrator and Management |
| 13 | Wipe and re-image the laptop; update the inventory | IT Administrator |
| 14 | Record completion and who performed each step | IT Administrator |

**Risk cases:** perform steps 6 and 7 first and immediately. A forced sign-out can still leave previously issued tokens usable for a limited time, so rotate sensitive secrets promptly.

## 6. Special cases

| Case | Handling |
|------|----------|
| Contractor or temporary worker | Same checklist with an **end date set at creation**; access reviewed at the end date |
| Person who holds an administrative role | Also remove administrative accounts and review what that account owned; if the person knew emergency access credentials, **change them** |
| Person who knew the emergency access credentials | Rotate the credentials and re-validate the accounts |
| Shared mailbox or shared files owned by the leaver | Reassign before deleting the account |
| Developer with source-control and cloud access | Remove from the repository service and Azure access groups; review recent changes and pipeline permissions |

## 7. Evidence and review

- Keep a simple log for each change: ticket or request, date, who acted, and what was done.
- During each quarterly access review, sample the log against the actual group memberships.
- Any leaver whose access was not removed on time is treated as an incident and the cause is corrected.

## 8. Limitations

- The checklists describe intent. Exact portal steps and timings depend on the tools and licences in use.
- Data retention periods and deletion timelines are not set here; they depend on legal and business requirements.
- Automation (for example, automated joiner and leaver workflows) is an optional enhancement and is not assumed.
