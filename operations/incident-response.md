# Incident and Problem Management

> **Proposed, illustrative process** for a fictional 15-person organization. It has not been exercised. Response times, escalation contacts, legal notification duties, and thresholds are **intentionally not defined**. Assumption IDs (A-nn) refer to [`../architecture/reference-scenario.md`](../architecture/reference-scenario.md); decisions are in [`../decisions/architecture-decisions.md`](../decisions/architecture-decisions.md) (ADR-019, ADR-020).
> The structure follows the published guidance of NIST SP 800-61 Rev. 3 (accessed 2026-11-13). Statements marked *author's recommendation* are not NIST statements.

## 1. Purpose and scope

This document defines how the organization detects, handles, and learns from incidents, and how it removes the causes of repeated incidents (problem management).

**In scope:** roles, priority model, lifecycle, first actions by incident type, three playbook outlines, communication, evidence handling, problem management, and post-incident review.

**Out of scope:** legal or regulatory notification procedures (no framework is assumed, A-05), forensic investigation methods, on-call rota, and tooling selection.

**Definitions.**

| Term | Meaning |
|------|---------|
| Event | Any observable occurrence in systems |
| Incident | An event, or a set of events, that harms or threatens operations, security, or data, and needs a response |
| Problem | The underlying cause of one or more incidents |
| Service restoration | Returning to normal operation, which may precede root-cause removal |

## 2. Approach

NIST SP 800-61 Rev. 3 treats incident response as part of cybersecurity risk management and aligns it to the NIST Cybersecurity Framework 2.0 functions. According to the document, Govern, Identify, and Protect prepare for incidents and reduce their impact; Detect, Respond, and Recover cover finding, managing, containing, eradicating, and recovering from them; and lessons learned feed improvement across all functions. It also states that organizations should use whichever life cycle model suits them best. Among its recommendations are documenting roles and authority, keeping response plans current and aligned with business continuity plans, maintaining protected and tested backups, prioritizing incidents by risk factors such as asset criticality and functional impact rather than first-come first-served, restoring from clean backups and verifying them, and sharing lessons as soon as they are identified.

The organization uses a **simple seven-step lifecycle**, mapped to those ideas (ADR-019):

| Step | Meaning | NIST emphasis |
|------|---------|---------------|
| 1. Prepare | Roles, runbooks, monitoring, tested backups | Govern, Identify, Protect |
| 2. Detect and report | Alerts, user reports, vendor notices | Detect |
| 3. Triage | Validate, categorize, prioritize | Detect and Respond |
| 4. Contain | Stop the spread or the harm | Respond |
| 5. Eradicate and recover | Remove the cause, restore service, verify | Respond and Recover |
| 6. Close | Declare the end by criteria and record it | Recover |
| 7. Review and improve | Review, actions, problem management | Improvement |

## 3. Roles

| Role | Held by (illustrative) | Responsibility |
|------|------------------------|----------------|
| Incident lead | IT Administrator by default; Cloud Administrator for Azure incidents | Owns the incident, assigns work, keeps the record, decides escalation |
| Technical responders | Administrators and relevant developers | Investigate, contain, recover |
| Communications lead | Management | Internal and external communication, decisions on disclosure |
| Scribe | Any available person | Time-stamped notes of actions and decisions |
| Service owner | Named per service | Accepts restoration and the cause fix |

For a person acting as incident lead, privileged actions use the administrative account (identity management).

## 4. Priority model

Priority reflects impact and urgency, not arrival order. This is the organization's own model; no response times are set here.

| Priority | Criteria (illustrative) |
|----------|--------------------------|
| P1 Urgent | Production unavailable or seriously degraded; suspected active compromise; confirmed loss or exposure of sensitive data |
| P2 High | Partial degradation; credible security risk without confirmed compromise; backup or monitoring failure |
| P3 Normal | Minor or single-user issue; informational findings |

Increase the priority when more assets, more users, or a more sensitive system are affected than first thought.

## 5. Reporting

- Staff report suspected incidents immediately to the IT Administrator through the agreed channel (for example a shared mailbox or chat channel). Reporting is never penalized.
- Alerts route to the shared channel (monitoring and alerting).
- Every report creates an **incident record** with: ID, reporter, time, summary, affected systems, priority, owner, status.

## 6. First actions by incident type

| Incident type | Contain first | Then |
|---------------|---------------|------|
| Compromised account | Block sign-in; revoke sessions; reset credentials and authentication methods | Review sign-in and audit logs, mailbox rules, forwarding, and changes made; check for privilege changes |
| Suspected phishing | Remove the message where possible; warn staff | Check who clicked; treat clicked accounts as potentially compromised |
| Lost or stolen device | Revoke sessions; disable the account if risk is high; mark the device | Confirm disk encryption status from the inventory; report |
| Malware or ransomware | Isolate the device from the network; stop connected sync if needed | Identify scope; restore from clean sources; verify before reconnecting |
| Leaked secret | Rotate the secret immediately; revoke the credential | Find where it was exposed; review usage logs; fix the process |
| Production outage or bad release | Decide rollback or roll forward (infrastructure design) | Confirm recovery; record the change |
| Data deletion or overwrite | Stop further change; identify the scope and the time | Restore from the recovery source (backup and recovery) |
| Network outage | Check link, firewall, and switch; use a mobile hotspot as a stopgap | Restore configuration; contact the provider |
| Unauthorized administrative change | Remove or reverse the change; secure the account | Review role assignments; audit the account's activity |

## 7. Playbook outlines

These outlines show the level of detail intended. Full runbooks follow the template in [`governance-and-runbooks.md`](governance-and-runbooks.md).

### 7.1 Compromised user account

1. Open an incident record and set the priority; assume P1 until scope is known.
2. Block sign-in and revoke active sessions for the account.
3. Reset the credentials and re-register authentication methods after confirming the user's identity through a separate channel.
4. Review the account's recent sign-ins, authentication changes, mailbox rules and forwarding, shared-file activity, and any role or group changes.
5. If administrative rights were involved, treat the administrative account as compromised and review changes made with it.
6. Contain: remove malicious rules, rotate any secrets the user could access.
7. Recover: restore deleted or changed data from the recovery source if needed.
8. Close with criteria: no malicious activity for an agreed period, credentials reset, findings recorded.
9. Review (section 10).

### 7.2 Leaked secret in a repository

1. Rotate or revoke the secret immediately; do not wait for the investigation.
2. Update the application through its normal change path with the new secret from the key vault.
3. Check usage logs for misuse between exposure and rotation.
4. Remove the secret from future use in the code; consider the repository history as exposed.
5. Find the cause (process gap) and add a control, such as secret detection in the pipeline.
6. Record and review.

### 7.3 Production outage after a release

1. Confirm the outage from monitoring and set the priority.
2. The lead on duty decides rollback or roll forward and records why.
3. Roll back by redeploying the previous artifact (infrastructure design, section 10), or follow the product-specific method.
4. Verify health and customer-facing function.
5. Communicate status to stakeholders.
6. Capture logs and the failing change for analysis.
7. Open a problem record if the cause is not obvious (section 9).

## 8. Communication and evidence

| Topic | Rule |
|-------|------|
| Internal updates | The incident lead posts status updates in one place at agreed intervals for P1 and P2 |
| External communication | Only the communications lead (management) speaks externally |
| Legal and contractual duties | No framework is assumed (A-05). If personal or customer data may be affected, obtain advice on notification duties early |
| Evidence | Preserve relevant logs and screenshots before making changes where this does not delay containment; record who did what and when |
| Access to the incident record | Limited to those who need it, because it may contain sensitive details |
| Third parties | Contact vendors when platform-side action is needed; record the case numbers |

NIST advises consulting legal advice before redirecting attackers to a sandbox and notes that delaying containment can be risky; this organization does not use such techniques.

## 9. Problem management

Problem management removes causes rather than symptoms.

| Step | Detail |
|------|--------|
| Trigger | Every P1; any incident that repeats; any incident without a clear cause |
| Investigate | Find the root cause using the incident timeline, logs, and change records; use methods such as asking "why" repeatedly |
| Record | A problem record linked to the incidents; known errors and workarounds are written down |
| Fix | Raise a change (change management) for the permanent fix, with an owner and due date |
| Verify | Confirm the problem no longer recurs; close the record |
| Trend | Quarterly review of incident and problem records for patterns |

## 10. Post-incident review

Held for every P1 and P2, soon after recovery. The goal is learning, not blame.

| Section | Content |
|---------|---------|
| Summary | What happened, impact, duration, priority |
| Timeline | Key events with timestamps |
| Detection | How it was found and how long it took |
| Response | What worked, what did not |
| Cause | Root cause and contributing factors |
| Actions | Specific actions with owners and due dates (changes, runbook updates, new alerts) |
| Documentation | Which documents and diagrams need updates |

Lessons are shared as soon as they are identified, in line with the NIST recommendation, and the actions are tracked to completion.

## 11. Exercises

Practice the process before it is needed. *Author's recommendation:* run a short tabletop exercise for one scenario (for example a compromised account) periodically, and a rollback rehearsal in Staging. Frequency is not defined here.

## 12. Growth path

| Trigger | Extension |
|---------|-----------|
| Larger team | Formal on-call rota and escalation tiers |
| Higher security exposure | Central security monitoring and a managed response service (evaluated for cost and licence) |
| Compliance or contractual demand | Defined reporting timelines and an evidence retention policy |
| Frequent incidents of one type | Dedicated problem investigations and automation |

## 13. Limitations

- The process has not been exercised and no incident has been handled.
- Response times, escalation contacts, and notification duties are not defined.
- NIST guidance is general; applying it to a specific organization requires judgment.

## 14. Sources

Accessed 2026-11-13.

| Topic | Source |
|-------|--------|
| Incident response life cycle, roles, prioritization, recovery, lessons learned | [NIST SP 800-61 Rev. 3 — Incident Response Recommendations and Considerations for Cybersecurity Risk Management](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r3.pdf) |
