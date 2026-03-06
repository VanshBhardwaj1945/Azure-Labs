
ok i followed this lab. https://github.com/MicrosoftLearning/AZ-104-MicrosoftAzureAdministrator/blob/master/Instructions/Labs/LAB_02a_Manage_Subscriptions_and_RBAC_Entra.md

6:08 PM

Searched the web
Searched the web
It looks like GitHub's rendering blocked the lab content for me. It seems like you've completed AZ-104 Lab 02a – Manage Subscriptions and RBAC (Entra ID). How can I help you with it? For example:

Did something not work as expected?
Want me to explain a concept from the lab (RBAC roles, management groups, custom roles, etc.)?
Looking to test your knowledge with some practice questions?
Want a summary/notes document of what the lab covered?
Let me know what you need!


PS /home/test> New-AzResourceGroup -Name 'Test123' -Location 'Central US' New-AzResourceGroup: The client 'testuser1@vanshbhardwaj.com' with object id 'd80798cf-6657-41c3-8126-0e45226de505' does not have authorization to perform action 'Microsoft.Resources/subscriptions/resourcegroups/

pasted

and this is what I did. can u create me a beautiful read me about what I did on this lab including a table of contents. I want it in .md format

6:10 PM

Viewed a file, created a file, read a file
Viewed a file, created a file, read a file
Here's your README! It includes:

Full table of contents with anchor links to every section
Formatted tables for environment details, role permissions, and test results
All your PowerShell commands in proper code blocks with the actual error output you got
A visual hierarchy diagram showing how management groups flow down to resources
A final summary flowchart showing the full lab workflow
Clean badges and icons throughout to make it easy to scan
It's structured so it reads great both on GitHub and as standalone documentation.

Az104 lab02a readme
Document · MD 
one small thing, take away any emojis. This has to be emant for employer to see and wanna hire me. going on my portfolio

6:11 PM

Created 2 files, viewed a file, ran a command
Created 2 files, viewed a file, ran a command
All emojis removed — clean, professional, and portfolio-ready. Good luck with the job hunt!

Az104 lab02a readme
Document · MD 
so on my github portfolio repo im making a link called "azure labs" meant for smaller labs like that repo ima have a main read me and have smaller folders with readmes explaining each one. What should i name the folder holding this readme.

6:15 PM
Something like az104-lab02a-rbac-management-groups would work well — it's descriptive, lowercase with hyphens (standard GitHub convention), and tells an employer exactly what's inside without even clicking.





Claude is AI and can make mistakes. Please double-check responses.
Az104 lab02a readme · MD
Copy

# AZ-104 Lab 02a — Manage Subscriptions and RBAC with Microsoft Entra ID

> **Azure Administrator Certification Lab Documentation**  
> Demonstrating secure cloud governance through Role-Based Access Control (RBAC), Management Groups, and permission validation.

---

## Table of Contents

1. [Lab Overview](#lab-overview)
2. [Environment Details](#environment-details)
3. [Step 1 — Create a Management Group](#step-1--create-a-management-group)
4. [Step 2 — Assign a Built-in RBAC Role](#step-2--assign-a-built-in-rbac-role)
5. [Step 3 — Create a Custom RBAC Role](#step-3--create-a-custom-rbac-role)
6. [Step 4 — Test Permissions via PowerShell](#step-4--test-permissions-via-powershell)
7. [Step 5 — Security Validation Results](#step-5--security-validation-results)
8. [Key Learnings](#key-learnings)
9. [Cleanup](#cleanup)
10. [Overall Result](#overall-result)

---

## Lab Overview

This lab focuses on implementing **Azure Role-Based Access Control (RBAC)** to enforce least-privilege access across an Azure environment. Using the Azure Portal and PowerShell, the following tasks were completed:

- Created a **Management Group** to organize subscriptions
- Assigned a **built-in RBAC role** to a user group
- Created a **custom RBAC role** with restricted permissions
- Validated access controls by **testing permissions** as a restricted test user

> **Core Security Principle Applied:** *Principle of Least Privilege* — users are granted only the permissions they need, nothing more.

---

## Environment Details

| Setting | Value |
|---|---|
| **Subscription Name** | Help Desk Support Team |
| **Location** | Central US |
| **Resource Group** | `Staging_VMS` |
| **Test User Account** | `testuser1@vanshbhardwaj.com` |
| **Test User Object ID** | `d80798cf-6657-41c3-8126-0e45226de505` |

---

## Step 1 — Create a Management Group

### What Was Done
A **Management Group** was created to organize Azure subscriptions and serve as a top-level container for applying security policies and RBAC assignments that inherit downward.

### Portal Navigation
```
Azure Portal → Microsoft Entra ID → Management Groups → + Create
```

### Configuration

| Setting | Value |
|---|---|
| **Management Group ID** | `corp-helpdesk-support` |
| **Display Name** | Help Desk Support Team |

### Why This Matters
Management groups sit *above* subscriptions in the Azure hierarchy. Any policy or role assignment applied at the management group level is **inherited by all subscriptions and resource groups** below it — making them essential for enterprise-scale governance.

```
Management Group (corp-helpdesk-support)
    └── Subscription (Help Desk Support Team)
            └── Resource Group (Staging_VMS)
                    └── Resources (VMs, etc.)
```

---

## Step 2 — Assign a Built-in RBAC Role

### What Was Done
The **Virtual Machine Contributor** built-in role was assigned to the Help Desk group at the Management Group scope, allowing VM management without broader infrastructure access.

### Portal Navigation
```
Management Group → Access Control (IAM) → + Add → Add Role Assignment
```

### Role Assignment Details

| Setting | Value |
|---|---|
| **Role** | Virtual Machine Contributor |
| **Assigned To** | `helpdesk` group |
| **Scope** | Management Group level |

### What This Role Allows vs. Restricts

| Permission | Allowed? |
|---|---|
| Create and manage virtual machines | Yes |
| Start, stop, restart VMs | Yes |
| Modify VM configurations | Yes |
| Create storage accounts | No |
| Create resource groups | No |
| Modify network security settings | No |
| Register resource providers | No |

---

## Step 3 — Create a Custom RBAC Role

### What Was Done
A custom role was created by **cloning an existing support role** and removing the permission to register Azure resource providers — further tightening access in line with least-privilege principles.

### Portal Navigation
```
IAM → + Add → Add Custom Role
```

### Custom Role Configuration

| Setting | Value |
|---|---|
| **Role Name** | `Custom Support Request` |
| **Description** | Support role with restricted provider permissions |
| **Base Role** | Cloned from existing support role |

### Permission Modification

The following permission was **explicitly removed** from the cloned role:

```
Microsoft.Support/register/action
```

This prevents Help Desk users from registering new Azure resource providers at the platform level — a permission they have no business need for.

### Why Custom Roles Matter
Built-in roles are broad by design. Custom roles allow you to **precisely define** exactly what actions a user or group can take, minimizing the blast radius of any potential account compromise.

---

## Step 4 — Test Permissions via PowerShell

All permission tests were performed logged in as the restricted test user `testuser1@vanshbhardwaj.com`.

### Verify Current Session Context

```powershell
Get-AzContext
```

### Check Accessible Resource Groups

```powershell
Get-AzResourceGroup
```

**Result:** `Staging_VMS` was visible — confirming the user has read access to the resource group as expected.

---

### Test 1 — Attempt to Create a Storage Account

```powershell
New-AzStorageAccount `
  -ResourceGroupName 'Staging_VMS' `
  -Name TestStorage39859345839045 `
  -Location 'Central US' `
  -SkuName Standard_GRS `
  -Kind StorageV2
```

**Result: FAILED**

```
The client 'testuser1@vanshbhardwaj.com' does not have authorization to perform action
'Microsoft.Storage/storageAccounts/write'
StatusCode: 403 Forbidden
```

**Interpretation:** The Virtual Machine Contributor role does not include storage account write permissions. Access correctly denied.

---

### Test 2 — Attempt to Create a Resource Group

```powershell
New-AzResourceGroup -Name 'Test123' -Location 'Central US'
```

**Result: FAILED**

```
The client 'testuser1@vanshbhardwaj.com' with object id 'd80798cf-6657-41c3-8126-0e45226de505'
does not have authorization to perform action
'Microsoft.Resources/subscriptions/resourcegroups/write'
over scope '/subscriptions/f57cd85b-89a5-4527-ab89-36c945703d48/resourcegroups/Test123'
StatusCode: 403 Forbidden
OperationID: f90cd237-32a0-415b-a4f4-d991928e3a6e
```

**Interpretation:** Creating resource groups requires subscription-level write permissions not granted by the assigned role. Access correctly denied.

---

## Step 5 — Security Validation Results

### Summary of Test Outcomes

| Action Attempted | Expected Result | Actual Result | RBAC Working? |
|---|---|---|---|
| View resource group `Staging_VMS` | Allowed | Allowed | Yes |
| Create a storage account | Denied | Denied (403) | Yes |
| Create a new resource group | Denied | Denied (403) | Yes |

Both failures confirmed the RBAC configuration is functioning as intended. The test user could view assigned resources but was blocked from making any infrastructure-level changes outside the scope of their role.

---

## Key Learnings

### 1. Management Groups Enable Scalable Governance
Assigning roles at the management group level propagates permissions through the entire hierarchy — no need to manage access subscription by subscription.

### 2. Built-in Roles Are a Strong Starting Point
Azure's built-in roles like **Virtual Machine Contributor** provide well-scoped permission sets. They're ideal when the role aligns closely with job function.

### 3. Custom Roles Enforce True Least Privilege
When built-in roles are too broad, custom roles allow you to define permissions with precision — removing specific actions (like `Microsoft.Support/register`) that aren't needed.

### 4. Always Test Your RBAC Assignments
Assigning a role doesn't guarantee it behaves as intended. **Always validate by logging in as the target user** and attempting the actions you want allowed *and* denied.

### 5. 403 Forbidden = RBAC is Working
A `403 Forbidden` response from Azure when attempting an unauthorized action is the desired outcome — it means your access controls are properly enforced.

---

## Cleanup

To remove the management group after completing the lab:

```powershell
Remove-AzManagementGroup -GroupName corp-helpdesk-support
```

To remove the resource group if created:

```powershell
Remove-AzResourceGroup -Name 'Staging_VMS' -Force
```

---

## Overall Result

This lab successfully demonstrated a complete Azure RBAC governance workflow:

```
Create Management Group
        ↓
Assign Built-in Role (Virtual Machine Contributor)
        ↓
Create Custom Role (Custom Support Request)
        ↓
Test Permissions as Restricted User
        ↓
Validate 403 Responses — Security Enforced
```

**All objectives completed. RBAC controls validated and working as expected.**

---

*Lab completed as part of AZ-104: Microsoft Azure Administrator certification preparation.*
