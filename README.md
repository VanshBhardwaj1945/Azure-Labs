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

## Infrastructure as Code

| Lab | What I Did | Concepts Covered | Technologies |
|---|---|---|---|
| **[AZ-104 — Manage Resources via ARM Templates](./az104-lab03b-arm-templates)** | Created a managed disk via the Portal and exported its ARM template, simplified it to a single parameter and redeployed via Custom Deployment, redeployed via Azure PowerShell and Azure CLI by editing the template in Cloud Shell each time, then rewrote the deployment as a Bicep template and deployed via CLI — five disks total, five different methods | ARM template structure, Template parameterization, Infrastructure-as-Code, Custom Deployment, Bicep vs ARM JSON, Cloud Shell editor, Multi-tool deployment fluency | Azure Portal, Azure PowerShell, Azure CLI, Azure Cloud Shell, ARM Templates, Bicep |

---

## Networking

| Lab | What I Did | Concepts Covered | Technologies |
|---|---|---|---|
| **[AZ-104 — Implement Intersite Connectivity](./az104-lab05-intersite-connectivity)** | Deployed two VMs in separate VNets with non-overlapping address spaces, used Network Watcher to confirm no connectivity before peering, configured bidirectional VNet peering, validated cross-VNet communication via PowerShell `Test-NetConnection`, then created a perimeter subnet, route table, and custom UDR to steer traffic through a future NVA | VNet peering, Network segmentation, Network Watcher diagnostics, User-defined routes, Next hop types, Traffic steering via virtual appliance, Hub-and-spoke routing pattern | Azure Portal, Azure PowerShell, Azure Network Watcher, VNet Peering, Route Tables, NSG |

---

## Virtual Machines & Scale Sets

| Lab | What I Did | Concepts Covered | Technologies |
|---|---|---|---|
| **[AZ-104 — Manage Virtual Machines](./az104-lab08-manage-virtual-machines)** | Deployed two Windows Server VMs, resized a VM post-deployment, created and managed a data disk (attach, detach, upgrade SKU, reattach), deleted VMs via PowerShell, deployed a VMSS across 3 availability zones with a load balancer, and configured CPU-based autoscale rules | VM lifecycle management, Post-deployment resizing, Managed disk SKU upgrades, Delete options for orphan prevention, VMSS orchestration, Metric-driven autoscaling, Availability zones | Azure Portal, Azure PowerShell, Azure Cloud Shell, Azure Monitor, Load Balancer |
| **[VMSS from Custom Image](./vmss-custom-image-lab)** | Deployed a base VM, created a test file, generalized the VM with Sysprep, captured it as a custom managed image, deployed a VMSS from that image across availability zones, opened RDP via NSG inbound rule, and validated image fidelity by confirming the test file persisted on a scale set instance | VM generalization, Sysprep OOBE, Custom image capture, VMSS from private image, Flexible orchestration mode, NSG inbound rules, Load balancer NAT rules, Image fidelity validation | Azure Portal, Azure PowerShell, Windows Sysprep, NSG, Load Balancer, RDP |

---

## Certifications in Progress

| Certification | Status |
|---|---|
| AZ-900 — Microsoft Azure Fundamentals | Complete |
| AZ-104 — Microsoft Azure Administrator | In Progress |

---

*Labs are added as completed. Each folder contains full documentation of steps, configurations, and outcomes.*
