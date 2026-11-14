# Monitoring and Alerting

> **Proposed, illustrative design.** Nothing here has been configured or tested. **Thresholds, retention periods, and response times are intentionally not defined**; they depend on business requirements and on the products selected. Assumption IDs (A-nn) refer to [`../architecture/reference-scenario.md`](../architecture/reference-scenario.md); decisions are in [`../decisions/architecture-decisions.md`](../decisions/architecture-decisions.md) (ADR-020).
> Statements about Microsoft services come from official documentation **accessed 2026-11-13** and must be re-verified before use. Statements marked *author's recommendation* are not vendor statements.

## 1. Purpose and scope

This document describes what the reference organization watches, where the data goes, how alerts are designed and routed, and how the results are reviewed.

**In scope:** signals per domain, logging and retention facts, alert design, an illustrative alert catalogue, access to monitoring data, and review routines.

**Out of scope:** product selection for a network monitoring tool, numeric thresholds, on-call arrangements, and pricing.

## 2. Principles

1. **Alert on what needs a human action.** An alert without an owner and a first action is noise.
2. **Few, meaningful alerts.** Start small and add alerts when an incident shows a gap.
3. **Every alert has an owner and a route.**
4. **Logs are evidence.** Logs that matter for security and audit are retained and reviewable.
5. **Monitoring itself is monitored.** Silent failure of monitoring or backup is an incident.
6. **Thresholds follow requirements.** None are invented here.

## 3. What is monitored

| Domain | Signals | Why |
|--------|---------|-----|
| Identity | Sign-in activity, failed and risky sign-ins, changes to administrative roles and groups, any use of emergency access accounts, changes to authentication policy | Identity is the primary security perimeter (overview) |
| Microsoft 365 | Service notices relevant to the tenant, unusual mailbox or file activity, changes to retention or versioning settings | Detect abuse and protect recovery options |
| Office network | Internet link and device availability, firewall denied traffic and administrative logins, firmware status | Network architecture, section 9 |
| Azure platform | Activity log (who changed what), resource health, role-assignment changes, budget alerts | Control plane auditing and cost control |
| Application | Availability, errors, performance, dependency failures (Application Insights where supported) | Detect release problems and outages |
| Delivery pipeline | Failed builds and deployments, production approvals, changes to pipeline definitions | Change integrity |
| Backup | Backup job failures, changes to backup settings, restore test results | Recoverability |
| Secrets | Key Vault access by unexpected identities, secret or vault deletion | Secrets protection |
| Cost | Budget threshold alerts per environment | Cost governance |

## 4. Logging and retention facts

| Source | What Microsoft documents | Implication |
|--------|--------------------------|-------------|
| Microsoft Entra sign-in and audit logs | Retained **seven days** on the Free tier and **30 days** on P1 and P2. Longer retention is possible by routing the data with Azure Monitor to an Azure storage account; Microsoft Purview Audit (Premium) is mentioned as an alternative for organizations with specific E5-level licences | Security investigations need longer than seven days; the export path and the licence required are **not verified here** and must be confirmed |
| Log Analytics workspace | Tables retain data for 30 days by default (some tables 90 days); analytics retention can be extended at extra cost; charges depend on data volume and retention | Retention is set deliberately per data type (cloud architecture, section 9) |
| Azure Monitor alerts | Fired alerts are stored for **30 days** and then deleted | Alert history is not a long-term record; incident records must live elsewhere |
| Exchange mailbox audit entries | Stored in the Recoverable Items folder when mailbox auditing is enabled | Audit data has its own retention limits (backup and recovery document) |

**Retention policy for logs (to define):** the business or a compliance driver sets how long each log class is kept. No period is stated here. Until then, retain at least what the platform provides and record the gap.

*Author's recommendation:* decide early which logs must outlive the default retention (identity and administrative activity are the strongest candidates) so that evidence exists when an incident is discovered late.

## 5. Alert design

Microsoft describes an alert rule as the combination of monitored resources, a signal, and conditions. When the conditions are met an alert fires and can start an **action group**, which can send email, SMS, or push notifications or start automated workflows. Alert types include metric, log search, activity log (including Resource Health and Service Health), and smart detection for Application Insights. Alerts can be **stateful** (fire once until resolved) or **stateless** (fire each time the condition is met, as all activity log alerts do), and an alert has a user response state of New, Acknowledged, or Closed. Alert processing rules can suppress or add action groups, for example during planned maintenance. Microsoft's page does not describe severity levels, so the priority model below is the organization's own.

| Design rule | Detail |
|-------------|--------|
| Route to a **shared** mailbox or channel, not to one person | Avoid single points of failure |
| Distinguish **urgent** from **informational** | Urgent alerts demand immediate action; informational ones are reviewed on a schedule |
| Use stateful alerts where repeated notifications add no value | Reduces alert fatigue |
| Suppress notifications during planned maintenance | With alert processing rules, linked to the change record |
| Review alerts that never fire or always fire | Remove or repair them |
| Record the first action in a runbook | See [`governance-and-runbooks.md`](governance-and-runbooks.md) |

**Cost note:** Microsoft states that metric alert rules are charged by the number of time series monitored, and that log search alerts that split by dimensions are charged by the time series created, with no additional cost for the alert if the data is already in a Log Analytics workspace. Pricing figures are not stated here.

## 6. Priority model (author's own)

Priorities link alerts to incident response. They are not Microsoft severity levels.

| Priority | Meaning (illustrative) | Examples |
|----------|-----------------------|----------|
| P1 Urgent | Significant business impact or active security compromise | Production unavailable; emergency account sign-in; suspected account takeover |
| P2 High | Degraded service or elevated risk without total loss | Repeated deployment failures; backup failures; budget overspend trend |
| P3 Normal | Minor or planned work | A single non-critical device offline; informational warnings |

Response times for each priority are **not defined** here. They are set when the organization defines its service expectations.

## 7. Alert catalogue (illustrative, thresholds not defined)

| ID | Signal | Condition (qualitative) | Route | Owner | Priority |
|----|--------|--------------------------|-------|-------|----------|
| AL-01 | Emergency access account sign-in | Any sign-in | Shared admin channel, immediate | IT Administrator | P1 |
| AL-02 | Administrative role or group membership change | Any change | Shared admin channel | IT Administrator | P2 |
| AL-03 | Sign-in anomalies | Repeated failures or unusual locations (rule design depends on the licence) | Shared admin channel | IT Administrator | P2 |
| AL-04 | Production availability | Probe failing or error rate above an agreed level (to be set) | On-call route | Cloud Administrator | P1 |
| AL-05 | Production errors or latency | Above an agreed level (to be set) | Team channel | Team lead | P2 |
| AL-06 | Deployment failure | Pipeline deployment fails | Team channel | Team lead | P2 |
| AL-07 | Azure role-assignment change | Any change at subscription or resource-group scope | Shared admin channel | Cloud Administrator | P2 |
| AL-08 | Resource deletion in Production | Delete operation in the Production resource group | Shared admin channel | Cloud Administrator | P2 |
| AL-09 | Backup job failure | Any failure | Shared admin channel | Owner of the data class | P2 |
| AL-10 | Backup configuration change | Policy or retention change | Shared admin channel | Cloud or IT Administrator | P2 |
| AL-11 | Key Vault access anomaly | Access by an unexpected identity | Shared admin channel | Cloud Administrator | P2 |
| AL-12 | Budget threshold | Threshold reached per environment (percentages to be set) | Budget owner | Environment owner | P3 |
| AL-13 | Office link or device down | Unreachable for a defined duration (to be set) | Shared admin channel | IT Administrator | P2 |
| AL-14 | Firewall administrative login or configuration change | Any | Shared admin channel | IT Administrator | P2 |

Microsoft states that budgets only alert and do not stop resources, so AL-12 is a notification that needs a human decision.

## 8. Network monitoring decision

The network architecture deferred the monitoring approach to this document. The baseline is **built-in device dashboards and scheduled checks** (availability checks of the link, firewall, switch, and access points; log review; configuration and firmware tracking). A **cloud-managed vendor service** is an optional enhancement if it is available with the chosen devices, with the licence and cost verified. A **self-hosted monitoring tool** is not assumed, because there are no on-premises servers (N-02). See ADR-020.

## 9. Access to monitoring data

| Role | Access (illustrative) |
|------|-----------------------|
| Developers | Read access to Development and Staging telemetry |
| Senior developers and leads | Read access to all environments' application telemetry |
| IT Administrator | Administration of identity and Microsoft 365 monitoring |
| Cloud Administrator | Administration of Azure monitoring and alert rules |
| Management | Summary dashboards |

Microsoft documents built-in roles **Monitoring Contributor** (create and manage alerts) and **Monitoring Reader** (view alerts and resources), and states that creating an alert rule needs read permission on the target resource, write permission on the resource group where the rule is created, and read permission on any associated action group. Role names and the mapping to groups must be verified before use. Access to production logs is limited, because logs can contain sensitive data.

## 10. Review routines

| Routine | Frequency (illustrative) | Owner |
|---------|--------------------------|-------|
| Review of fired alerts, and of alerts that never fire | Monthly | IT Administrator and Cloud Administrator |
| Review of privileged and emergency account activity | Monthly | IT Administrator |
| Review of denied inter-zone traffic and device logins | Monthly | IT Administrator |
| Review of cost and budget alerts | Monthly | Environment owners |
| Review of dashboards and alert catalogue after each incident | After every P1 and P2 | Incident lead |
| Test of the notification route (a deliberate test alert) | At least when routes change | Alert owner |

Frequencies are illustrative.

## 11. Growth path

| Trigger | Extension |
|---------|-----------|
| More services or environments | Standard alert templates deployed through infrastructure as code |
| Security needs grow | Central security monitoring with correlation, evaluated against licence and cost |
| Compliance or customer demand | Defined log retention and evidence reporting |
| After-hours coverage required | On-call rota and escalation policy |
| Hosting product selected | Product-specific health signals added |

## 12. Limitations

- No alert, dashboard, or log export has been created or tested.
- Thresholds, retention, response times, and on-call arrangements are not defined.
- Entra log export paths and their licence requirements, Microsoft 365 service-notice mechanisms, and rule design for sign-in anomalies are not verified.
- Product selection for network monitoring is open.

## 13. Well-Architected quick check (self-review, not an official assessment)

| Pillar | Observation |
|--------|-------------|
| Reliability | Availability signals and backup monitoring; no objectives invented |
| Security | Identity, privilege, and secret-access alerts; log retention gap acknowledged |
| Cost Optimization | Small alert set; alert and log cost drivers named; budgets monitored |
| Operational Excellence | Owners, routes, runbooks, and review routines defined |
| Performance Efficiency | Application performance signals included; thresholds deliberately undefined |

## 14. Sources

Accessed 2026-11-13. Recommendations beyond these statements are the author's own.

| Topic | Source |
|-------|--------|
| Entra log retention by licence and export options | [Microsoft Learn — Microsoft Entra data retention](https://learn.microsoft.com/en-us/entra/identity/monitoring-health/reference-reports-data-retention) |
| Alert types, action groups, alert states, processing rules, retention, roles, cost drivers | [Microsoft Learn — Overview of Azure Monitor alerts](https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/alerts-overview) |
| Log Analytics retention | [Microsoft Learn — Manage data retention in a Log Analytics workspace](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/data-retention-configure) |
| Budgets only alert | [Microsoft Learn — Create and manage budgets](https://learn.microsoft.com/en-us/azure/cost-management-billing/costs/tutorial-acm-create-budgets) |
| Exchange mailbox audit entries in Recoverable Items | [Microsoft Learn — Recoverable Items folder in Exchange Online](https://learn.microsoft.com/en-us/exchange/security-and-compliance/recoverable-items-folder/recoverable-items-folder) |
