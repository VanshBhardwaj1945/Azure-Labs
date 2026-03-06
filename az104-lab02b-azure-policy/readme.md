# AZ-104 Lab 02b — Manage Governance via Azure Policy

> **Azure Administrator Certification Lab Documentation**  
> Demonstrating Azure governance enforcement through Policy assignments, tag inheritance, and resource locks — executed entirely via Azure Cloud Shell and PowerShell.

---

## Table of Contents

1. [Lab Overview](#lab-overview)
2. [Environment Details](#environment-details)
3. [Step 1 — Create a Tagged Resource Group](#step-1--create-a-tagged-resource-group)
4. [Step 2 — Assign a Tag Enforcement Policy](#step-2--assign-a-tag-enforcement-policy)
5. [Step 3 — Validate Policy Enforcement](#step-3--validate-policy-enforcement)
6. [Step 4 — Remove Policy and Assign Tag Inheritance](#step-4--remove-policy-and-assign-tag-inheritance)
7. [Step 5 — Validate Tag Inheritance](#step-5--validate-tag-inheritance)
8. [Step 6 — Apply a Resource Lock](#step-6--apply-a-resource-lock)
9. [Step 7 — Validate Lock Enforcement](#step-7--validate-lock-enforcement)
10. [Step 8 — Cleanup](#step-8--cleanup)
11. [Key Learnings](#key-learnings)
12. [Overall Result](#overall-result)

---

## Lab Overview

This lab focuses on implementing **Azure Policy** to enforce governance rules across cloud resources. The lab was completed entirely through **Azure Cloud Shell using PowerShell** to build familiarity with CLI-driven cloud administration.

Tasks completed:

- Created a resource group with a cost center tag
- Assigned a built-in policy to require tags on all resources
- Validated that non-compliant resource creation was blocked
- Replaced the enforcement policy with a tag inheritance policy
- Validated that tags automatically propagate from resource group to child resources
- Applied a resource lock to prevent accidental deletion
- Validated the lock by attempting to delete the resource group
- Cleaned up all resources via PowerShell

> **Core Governance Principles Applied:** Tag-based cost management, policy-driven compliance enforcement, and deletion protection through resource locks.

---

## Environment Details

| Setting | Value |
|---|---|
| **Resource Group** | `test` |
| **Location** | Central US |
| **Tag Key** | `Cost Center` |
| **Tag Value** | `000` |
| **Enforcement Policy** | Require a tag and its value on resources |
| **Inheritance Policy** | Inherit a tag from the resource group |
| **Lock Name** | `test-delete-lock` |
| **Lock Level** | CanNotDelete |
| **Interface Used** | Azure Cloud Shell (PowerShell) |

---

## Step 1 — Create a Tagged Resource Group

A resource group was created with a `Cost Center` tag applied at the resource group level. This tag serves as the source for later inheritance testing.

```powershell
New-AzResourceGroup -Name "test" -Location "CentralUS" -Tag @{"Cost Center"="000"}
```

**Output:**

```
ResourceGroupName : test
Location          : centralus
ProvisioningState : Succeeded
Tags              :
                    Name         Value
                    ===========  =====
                    Cost Center  000

ResourceId        : /subscriptions/***************/resourceGroups/test
```

The resource group was provisioned successfully with the tag confirmed in the output.

---

## Step 2 — Assign a Tag Enforcement Policy

A built-in Azure Policy was assigned to the resource group requiring that any resource created within it must carry the `Cost Center` tag with the value `000`. Any resource creation attempt that does not meet this requirement will be blocked.

```powershell
New-AzPolicyAssignment `
  -Name "require-costcenter-tag" `
  -DisplayName "Require Cost Center Tag Policy" `
  -Scope (Get-AzResourceGroup -Name "test").ResourceId `
  -PolicyDefinition (Get-AzPolicyDefinition | Where-Object {$_.DisplayName -eq "Require a tag and its value on resources"}) `
  -PolicyParameterObject @{ "tagName"="Cost Center"; "tagValue"="000" }
```

### Policy Assignment Details

| Setting | Value |
|---|---|
| **Assignment Name** | `require-costcenter-tag` |
| **Display Name** | Require Cost Center Tag Policy |
| **Scope** | Resource group `test` |
| **Built-in Policy** | Require a tag and its value on resources |
| **Tag Enforced** | `Cost Center` = `000` |

---

## Step 3 — Validate Policy Enforcement

With the policy active, a storage account creation was attempted inside the `test` resource group **without** the required tag. The portal blocked the operation and displayed a policy violation error.

**Result: BLOCKED**

The resource creation was denied at the Azure layer before provisioning began. This confirmed the policy was correctly enforced at the resource group scope.

> Screenshot: Policy violation message when attempting to create a storage account without the required Cost Center tag.

---

## Step 4 — Remove Policy and Assign Tag Inheritance

The enforcement policy was removed and replaced with a tag **inheritance** policy. Rather than blocking non-compliant resources, this policy automatically applies the `Cost Center` tag from the parent resource group to any child resources that are missing it.

### Remove Enforcement Policy

```powershell
Remove-AzPolicyAssignment `
  -Name "require-costcenter-tag" `
  -Scope (Get-AzResourceGroup -Name "test").ResourceId
```

### Assign Inheritance Policy

```powershell
New-AzPolicyAssignment `
  -Name "inherit-tag-policy" `
  -DisplayName "Inherit Tag From Resource Group" `
  -PolicyDefinition (Get-AzPolicyDefinition | Where-Object {$_.DisplayName -eq "Inherit a tag from the resource group"}) `
  -Scope (Get-AzResourceGroup -Name "test").ResourceId `
  -IdentityType SystemAssigned `
  -Location "CentralUS" `
  -PolicyParameterObject @{ tagName = "Cost Center" }
```

### Why SystemAssigned Identity Is Required
The inheritance policy uses **Azure Policy remediation tasks** to retroactively apply tags to existing resources. This requires a managed identity with write permissions — hence `SystemAssigned` is specified. Enforcement-only policies do not require this.

---

## Step 5 — Validate Tag Inheritance

A storage account was created inside the `test` resource group without explicitly specifying a tag. After creation, the `Cost Center: 000` tag was automatically applied to the storage account, inherited from the parent resource group.

**Result: Tag inherited successfully.**

> Screenshot: Storage account showing the Cost Center tag automatically populated after creation.

---

## Step 6 — Apply a Resource Lock

A `CanNotDelete` lock was applied to the resource group to prevent accidental deletion while the lab environment was still active.

```powershell
New-AzResourceLock `
  -LockName "test-delete-lock" `
  -LockLevel CanNotDelete `
  -ResourceGroupName "test"
```

### Lock Details

| Setting | Value |
|---|---|
| **Lock Name** | `test-delete-lock` |
| **Lock Level** | CanNotDelete |
| **Scope** | Resource group `test` |

`CanNotDelete` allows reads and modifications but blocks any delete operations on the resource group and its contents until the lock is explicitly removed.

---

## Step 7 — Validate Lock Enforcement

A deletion of the resource group was attempted while the lock was active.

```powershell
Remove-AzResourceGroup -Name "test"
```

**Confirmation prompt:**

```
Confirm
Are you sure you want to remove resource group 'test'
[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): y
```

**Result: FAILED**

```
Remove-AzResourceGroup: The scope '/subscriptions/***************/resourcegroups/test' cannot
perform delete operation because following scope(s) are locked:
'/subscriptions/***************/resourceGroups/test'.
Please remove the lock and try again.
StatusCode: 409
ReasonPhrase: Conflict
OperationID: 12f8f2be-7d96-4520-b21a-dfc045265d73
```

The deletion was blocked at the Azure layer with a `409 Conflict` status, confirming the lock was functioning correctly.

---

## Step 8 — Cleanup

The lock was removed first, then the resource group was deleted successfully.

### Remove the Lock

```powershell
Remove-AzResourceLock `
  -LockName "test-delete-lock" `
  -ResourceGroupName "test"
```

**Confirmation prompt:**

```
Confirm
Are you sure you want to delete the following lock:
/subscriptions/***************/resourceGroups/test/providers/Microsoft.Authorization/locks/test-delete-lock
[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): y
True
```

### Delete the Resource Group

```powershell
Remove-AzResourceGroup -Name "test"
```

**Confirmation prompt:**

```
Confirm
Are you sure you want to remove resource group 'test'
[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): y
True
```

Both operations returned `True`, confirming successful cleanup.

---

## Key Learnings

### 1. Azure Policy Operates at Scope
Policies assigned at the resource group level apply to all resources within it. The same policies can be applied at management group, subscription, or resource group scope depending on how broadly enforcement is needed.

### 2. Enforcement vs. Inheritance Are Different Policy Strategies
The **Require tag** policy blocks non-compliant resources outright. The **Inherit tag** policy takes a softer approach — allowing creation but automatically remediating missing tags. Choosing between them depends on whether you want to block or correct.

### 3. Remediation Policies Require a Managed Identity
Policies that modify existing resources (like tag inheritance) need a system-assigned managed identity to perform write operations on behalf of Azure Policy. This is not required for deny-only policies.

### 4. Resource Locks Are a Last Line of Defense
Even with RBAC and policy in place, a `CanNotDelete` lock provides an additional hard stop against accidental or unauthorized deletion — regardless of what permissions a user holds.

### 5. Order of Operations Matters for Cleanup
Attempting to delete a locked resource group will always fail. The lock must be explicitly removed before deletion can proceed, which is by design.

### 6. PowerShell and Cloud Shell Build Practical Fluency
Running this lab entirely through Azure Cloud Shell reinforced how CLI-driven administration works in practice — including piping `Get-AzResourceGroup` directly into a scope parameter, chaining policy lookups inline, and reading structured output to confirm state.

---

## Overall Result

This lab demonstrated a layered Azure governance workflow entirely through PowerShell:

```
Create Tagged Resource Group
        ↓
Assign Enforcement Policy → Validate Block
        ↓
Replace with Inheritance Policy → Validate Auto-Tag
        ↓
Apply Resource Lock → Validate Delete Block
        ↓
Remove Lock → Clean Up Successfully
```

**All objectives completed. Governance controls validated at each stage.**

---

*Lab completed as part of AZ-104: Microsoft Azure Administrator certification preparation.*
