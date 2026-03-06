# Azure Labs

A collection of hands-on Azure labs completed while studying for the AZ-900 (Azure Fundamentals) and AZ-104 (Azure Administrator) certifications, with more to be added as I continue through the curriculum.

Each lab folder contains a dedicated README documenting the objective, steps taken, configuration details, and key takeaways.

---

## Identity & Access Management

| Lab | What I Did | Concepts Covered | Technologies |
|---|---|---|---|
| **[AZ-104 — RBAC & Management Groups](./az104-lab02a-rbac-management-groups)** | Created a management group, assigned a built-in RBAC role to a user group, built a custom role with restricted permissions, and validated enforcement via PowerShell | Role-Based Access Control, Principle of Least Privilege, Custom Role definitions, Permission inheritance through management group hierarchy | Azure Portal, Microsoft Entra ID, Azure PowerShell, IAM |
| **[AZ-104 — Governance via Azure Policy](./az104-lab02b-azure-policy)** | Created a tagged resource group, assigned a tag enforcement policy and validated it blocked non-compliant resources, replaced it with a tag inheritance policy and validated auto-tagging, applied a resource lock and validated deletion was blocked, then cleaned up entirely via PowerShell | Azure Policy, Tag enforcement, Tag inheritance, Resource locks, CanNotDelete, Managed Identity, Cloud Shell administration | Azure Portal, Azure Policy, Azure PowerShell, Azure Cloud Shell, IAM |

---



## Certifications in Progress

| Certification | Status |
|---|---|
| AZ-900 — Microsoft Azure Fundamentals | In Progress |
| AZ-104 — Microsoft Azure Administrator | In Progress |

---

*Labs are added as completed. Each folder contains full documentation of steps, configurations, and outcomes.*
