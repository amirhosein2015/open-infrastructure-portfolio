# Change Management

> **Proposed, illustrative process** for a fictional 15-person organization. It has not been operated. Approval roles, windows, and thresholds are examples. Assumption IDs (A-nn) refer to [`../architecture/reference-scenario.md`](../architecture/reference-scenario.md); decisions are in [`../decisions/architecture-decisions.md`](../decisions/architecture-decisions.md) (ADR-016, ADR-019).

## 1. Purpose and scope

Change management makes sure changes to systems are understood, approved at the right level, reversible where possible, recorded, and communicated, without slowing a small team more than necessary.

**In scope:** application releases, infrastructure and Azure changes, identity and access changes, Microsoft 365 tenant settings, office network changes, device standards, and policy changes.

**Out of scope:** tool selection (a simple register is enough, see section 9), and organization-wide project management.

## 2. Principles

1. **Proportionate.** Low-risk, repeatable changes are pre-approved; risky changes get review.
2. **Every change has an owner and a record.**
3. **Rollback is planned before the change.**
4. **Existing controls are reused.** Application releases already go through pull request review and the production approval gate (infrastructure design); change management does not add a second approval for the same step.
5. **Documentation follows reality.** A change is not finished until the affected documents and diagrams are updated.

## 3. Change types

| Type | Definition | Examples | Approval |
|------|------------|----------|----------|
| **Standard** | Low risk, well understood, repeatable, with a documented procedure | Onboarding and offboarding steps; adding a user to a group within the access model; routine patching following the routine | Pre-approved; recorded |
| **Normal** | Anything that does not meet the standard criteria | Production release; new alert rule; firewall rule change; role-assignment change; infrastructure change; switching MFA from security defaults to Conditional Access | Review and approval before implementation |
| **Emergency** | Needed immediately to restore service or contain a security incident | Rollback of a failing release; blocking a compromised account | Verbal or chat approval by the incident lead; written record and review afterwards |

When in doubt, treat the change as Normal.

## 4. Approval by area

| Area | Reviewer or approver (illustrative) | Mechanism |
|------|-------------------------------------|-----------|
| Application release | Lead other than the requester | Pull request review and production approval gate (infrastructure design, ADR-016) |
| Infrastructure as code | Two reviewers including the cloud administrator for production-affecting changes | Pull request review with a change preview |
| Azure roles and subscription-level settings | Cloud Administrator plus Management for new privileged access | Change record |
| Entra and Microsoft 365 tenant settings | IT Administrator, with a second administrator reviewing high-impact changes | Change record |
| Office network (firewall rules, VLANs, Wi-Fi) | IT Administrator, with a second person reviewing rules that open access | Change record and updated network documents |
| Security policy | Management | Pull request to the policy documents |
| Backup and retention settings | Data owner and administrator | Change record; weakening a protection needs the owner's approval |

## 5. Process

| Step | Action | Owner |
|------|--------|-------|
| 1. Request | Describe the change, the reason, the systems affected, and the requested date | Requester |
| 2. Assess | Classify the type; assess risk and impact (section 6); decide whether a test is needed | Change owner |
| 3. Plan | Write the steps, the verification, and the **rollback**; name who performs it | Change owner |
| 4. Approve | Obtain approval according to section 4 | Approver |
| 5. Schedule | Choose a time that fits business needs and available people; announce to affected users | Change owner |
| 6. Implement | Carry out the steps; suppress planned alerts if needed | Implementer |
| 7. Verify | Check the result against the planned verification | Implementer and change owner |
| 8. Record | Update the change record, the documentation, and the diagrams | Change owner |
| 9. Review | For failed or emergency changes, review what happened | Change owner and approver |

## 6. Risk and impact checklist

A change is **higher risk** if any answer is yes:

- Does it touch Production, identity, networking, or backup configuration?
- Could it lock people out (for example MFA, Conditional Access, firewall rules, or administrator roles)?
- Could it expose data or widen access?
- Is it hard to reverse?
- Does it affect many users or a critical service?
- Is it being done for the first time?

For changes that could lock out administrators, confirm before starting that the emergency access accounts work and that a second administrator is available.

## 7. Emergency changes

1. The incident lead approves the change (chat or verbal) and notes the time and reason.
2. The change is performed through the pipeline where possible; direct portal changes in Production are the exception.
3. Within an agreed period afterwards the change is documented and reviewed, and any direct portal change is reflected back into the infrastructure definitions to avoid drift.
4. A post-incident review (incident response, section 10) decides whether a permanent fix is needed.

## 8. Change records

| Field | Content |
|-------|---------|
| ID and title | Unique and descriptive |
| Type | Standard, Normal, or Emergency |
| Requester and owner | Names or roles |
| Description and reason | What and why |
| Systems and users affected | Scope |
| Risk assessment | Checklist result |
| Plan, verification, rollback | Steps |
| Approver and time | Who approved and when |
| Schedule | Planned time |
| Outcome | Success, partial, failed, rolled back |
| Documentation updates | Links to the updated documents |

## 9. Tooling

A **simple register** is sufficient at this size: a ticket list, a shared table, or issues in the Git repository. No product is selected. Pull requests already serve as the record for code and infrastructure changes.

## 10. Calendar and communication

- Planned changes affecting users are announced in advance through the usual channel. Lead times are not defined here.
- Avoid releasing when nobody is available to respond.
- Periods when changes are restricted (for example around key business events) are set by management and are not defined here.

## 11. Measuring the process (author's suggestion)

Track the number of changes by type, how many failed or needed rollback, how many emergency changes occurred, and how many incidents were caused by changes. Targets are not defined here.

## 12. Growth path

| Trigger | Extension |
|---------|-----------|
| More engineers or systems | A change advisory group for high-impact changes |
| Compliance or customer demand | Stricter evidence, separation of duties, and change windows |
| Frequent standard changes | Automation of those changes |
| Many incidents from changes | Stronger testing and progressive delivery |

## 13. Limitations

- The process has not been operated. Time limits, lead times, and windows are not defined.
- Roles are illustrative and depend on who actually holds them.
