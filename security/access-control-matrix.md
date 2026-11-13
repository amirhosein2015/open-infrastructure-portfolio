# Access Control Matrix

> **Proposed model, not an actual company's access configuration.** Roles, counts, and permissions are illustrative and apply to the fictional 15-person organization in [`../architecture/reference-scenario.md`](../architecture/reference-scenario.md). Nothing has been configured or tested.

## 1. Principles

1. **Least privilege:** each role receives the minimum access needed. Production is the most restricted.
2. **Groups, not individuals:** access is granted to groups ([`identity-management.md`](identity-management.md), section 4).
3. **Separate accounts:** anyone holding an administrative role uses a **separate administrative account**. The daily account holds no administrative rights.
4. **No standing production administration for developers:** developers do not receive administrator access to Production.
5. **Scope narrowly:** Azure roles are assigned at **resource-group scope**, not subscription scope, except for the very few accounts that must manage the subscription (ADR-002). Azure RBAC is additive, so a broad assignment at a higher scope overrides a narrow one beneath it.
6. **Review regularly** (identity-management section 9).

## 2. Legend

| Code | Meaning |
|------|---------|
| — | No access |
| R | Read only (view) |
| U | Use as an ordinary user |
| C | Contribute: create and change resources or content, but not manage access |
| D | Deploy through the pipeline only; no direct changes to resources |
| A | Approve (for example, a production release) |
| Admin | Administrative control of that area |
| Own | Access to own data only |

## 3. Matrix

| Area | Employee | Developer | Senior Developer / Team Lead | IT Administrator | Cloud Administrator | Management |
|------|----------|-----------|------------------------------|------------------|---------------------|------------|
| **Microsoft 365** (mail, Teams, files) | U | U | U | U (daily account); Admin (administrative account, M365 and Entra scope) | U | U |
| **Office network** (staff Wi-Fi/LAN) | U | U | U | U; Admin of network devices from the management VLAN | U | U |
| **Office network** (management VLAN) | — | — | — | Admin | — | — |
| **Development** environment | — | C | C | R | Admin | — |
| **Staging** environment | — | D | C and A | R | Admin | — |
| **Production** environment | — | R (limited, for diagnosis) | D (through the approved pipeline) and A | R | Admin | — |
| **Azure Portal** | — | R or C (Development resource group only) | C (Development and Staging resource groups) | R | Admin | R (cost and overview only) |
| **Monitoring** (dashboards, logs, alerts) | — | R (Development and Staging) | R (all environments) | Admin | Admin | R (summary dashboards) |
| **Backup management** (configure, restore) | — | — | — | Admin (Microsoft 365 data and device configurations) | Admin (Azure workloads) | — |

Notes:

- **Production access for developers:** Developers and leads change Production **only through the pipeline**, after review and approval. Direct changes and standing administrator rights are not granted. Read access for diagnosis should be limited and logged. Break-glass elevation for incidents is a documented exception (see section 4).
- **Cloud Administrator and IT Administrator:** in the illustrative counts these may be the same person. That person still uses a **daily account** for ordinary work and a **separate administrative account** for administration, never one account for both.
- **Management:** read-only views for oversight; no technical changes.
- **Backup:** restore operations are administrative and require a documented request. Who can restore which data, and the restore-test cadence, are defined in Phase 7.
- **Employee** access to Azure is none; the role uses Microsoft 365 and the office network only.

## 4. Exceptions and elevation

| Situation | Handling |
|-----------|----------|
| Urgent production issue needing more access than normal | A cloud administrator performs the action, or grants time-limited access that is removed afterwards. The reason and the time are recorded. Time-limited, approval-based elevation through Privileged Identity Management requires Microsoft Entra ID P2 or Entra ID Governance and is **not assumed** in the baseline |
| Temporary project access | Time-limited group membership with an owner and an end date |
| Lost administrator | Emergency access accounts (identity-management section 6) |

## 5. Azure role mapping (illustrative)

Built-in Azure roles are used where they fit. Microsoft's documentation shows that built-in roles range from high-level to specific and that custom roles are possible when none fits. The mapping below uses common built-in role names and must be verified against the permissions of each role before use.

| Group | Development RG | Staging RG | Production RG | Subscription scope |
|-------|----------------|------------|---------------|--------------------|
| `grp-developers` | Contributor | Reader | Reader (limited) | — |
| `grp-leads` | Contributor | Contributor | Reader | — |
| `grp-cloud-admins` | Contributor | Contributor | Contributor | Owner for the separate administrative accounts only (at most two; ADR-014) |
| `grp-it-admins` | Reader | Reader | Reader | — |
| `grp-management` | — | — | — | Reader (cost and overview only) |

Resolved in Phase 5 (see ADR-014):

- **Owner role:** the cloud administrator's separate administrative account holds Owner at subscription scope (with at most one deputy) so it can create resource groups and manage role assignments. The group-based assignments at resource-group scope are Contributor. A narrower role that replaces Owner is to be evaluated and verified, not assumed.
- **Pipeline identity:** a managed identity or service principal scoped to the target resource group (detail in Phase 6).
- **Developer Contributor on Development:** kept for the Development resource group only, subject to the quarterly review. Staging is read-only for developers and Production is read-only with logged access.

## 6. Limitations

- The matrix is a model. Real access depends on licences, product permissions, and application design.
- Role and group names are examples.
- No access has been implemented or tested.
