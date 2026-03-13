# AZ-104 Lab 03b — Manage Azure Resources Using ARM Templates

> **Azure Administrator Certification Lab Documentation**  
> Deploying Azure managed disks five different ways — using an auto-exported ARM template from the Portal, a simplified custom ARM template, Azure PowerShell, Azure CLI, and a Bicep template — to build fluency across all major Infrastructure-as-Code deployment methods.  
> ***Link to Lab Instructions:*** [GitHub Repo](https://github.com/MicrosoftLearning/AZ-104-MicrosoftAzureAdministrator/blob/master/Instructions/Labs/LAB_03b-Manage_Azure_Resources_by_Using_ARM_Templates.md)

---

## Table of Contents

1. [Lab Overview](#lab-overview)
2. [Environment Details](#environment-details)
3. [Folder Structure](#folder-structure)
4. [Step 1 — Create disk1 via Portal and Export ARM Template](#step-1--create-disk1-via-portal-and-export-arm-template)
5. [Step 2 — Edit Template and Redeploy via Custom Deployment (disk2)](#step-2--edit-template-and-redeploy-via-custom-deployment-disk2)
6. [Step 3 — Deploy disk3 via Azure PowerShell](#step-3--deploy-disk3-via-azure-powershell)
7. [Step 4 — Deploy disk4 via Azure CLI](#step-4--deploy-disk4-via-azure-cli)
8. [Step 5 — Deploy disk5 via Bicep Template](#step-5--deploy-disk5-via-bicep-template)
9. [Key Learnings](#key-learnings)
10. [Overall Result](#overall-result)

---

## Lab Overview

This lab explores **Infrastructure-as-Code (IaC)** on Azure by deploying the same type of managed disk resource five different ways, each using a different method or toolchain. The goal is to understand how ARM templates are structured, how parameters make them reusable, and how Bicep compares to ARM JSON as a more concise alternative.

Tasks completed:

- Created a managed disk via the Azure Portal and exported its auto-generated ARM template
- Simplified and edited the template, then redeployed via Portal Custom Deployment
- Edited the template again and deployed via Azure PowerShell in Cloud Shell
- Edited the template again and deployed via Azure CLI in Cloud Shell
- Modified and deployed a Bicep template via Azure CLI

> **Core Concepts Applied:** ARM template structure, parameterization, Infrastructure-as-Code, multi-tool deployment fluency (Portal, PowerShell, CLI, Bicep).

---

## Environment Details

| Setting | Value |
|---|---|
| **Resource Group** | `az104-rg3` |
| **Location** | East US |
| **Disk Size** | 32 GiB |
| **Base SKU** | Standard_LRS |
| **Bicep SKU** | StandardSSD_LRS |
| **Disks Created** | `disk1`, `disk2`, `disk3`, `disk4`, `az104-disk5` |

---

## Folder Structure

```
az104-lab03b-arm-templates/
├── README.md
├── templates/
│   ├── template.json          # Original auto-exported ARM template (disk1)
│   ├── new-template.json      # Simplified ARM template with disk_name parameter
│   └── azuredeploydisk.bicep  # Bicep template (disk5)
└── docs/
    ├── 01-disk1.png           # Custom Deployment portal showing disk_name parameter (disk2)
    ├── 02-disk3.png           # Template editor with disk_name set to disk3
    ├── 03-disk4.png           # Template editor with disk_name set to disk4
    └── 04-disk5.png           # Bicep template in Cloud Shell editor
```

---

## Step 1 — Create disk1 via Portal and Export ARM Template

`disk1` was created through the Azure Portal. After deployment, the **Export Template** feature under the Automation blade was used to capture the auto-generated ARM template representing the disk's configuration as code.

The exported template uses a verbose parameter structure with a companion parameters file. Every configurable property — disk name, location, SKU, size, encryption, network access — is exposed as a separate parameter.

**Key fields from the exported `template.json` parameters file:**

| Parameter | Value |
|---|---|
| `diskName` | `disk1` |
| `location` | `eastus` |
| `sku` | `Standard_LRS` |
| `diskSizeGb` | `32` |
| `createOption` | `empty` |
| `diskEncryptionSetType` | `EncryptionAtRestWithPlatformKey` |
| `networkAccessPolicy` | `AllowAll` |
| `publicNetworkAccess` | `Enabled` |

> This exported template is the raw starting point. The Portal generates it automatically — the next step is simplifying and parameterizing it for reuse.

---

## Step 2 — Edit Template and Redeploy via Custom Deployment (disk2)

The exported template was simplified into `new-template.json`. The verbose multi-parameter structure was stripped down to a single `disk_name` parameter with a default value, with all other settings hardcoded. This makes the template clean and reusable for any disk name at deploy time.

**Key changes from `template.json` → `new-template.json`:**

| | template.json | new-template.json |
|---|---|---|
| **Parameter name** | `disks_disk1_name` | `disk_name` |
| **Default value** | `disk1` | `disk2` |
| **Schema version** | 2015-01-01 | 2019-04-01 |
| **Parameter count** | 11 parameters | 1 parameter |

The template was deployed via **Azure Portal → Deploy a custom template → Build your own template in the editor**. The portal surfaced `disk_name` as an editable field, confirming the parameterization worked correctly.

<img src="./docs/01-disk1.png" alt="Azure Portal Custom Deployment blade showing disk_name parameter field populated with disk2" width="600"/>

---

## Step 3 — Deploy disk3 via Azure PowerShell

Cloud Shell was opened in **PowerShell** mode. The `new-template.json` file was uploaded, the `disk_name` default value was changed to `disk3` in the Cloud Shell editor, and the template was deployed using `New-AzResourceGroupDeployment`.

**Template edit — disk_name defaultValue changed to `disk3`:**

<img src="./docs/02-disk3.png" alt="new-template.json open in Cloud Shell editor with disk_name defaultValue set to disk3" width="600"/>

```powershell
New-AzResourceGroupDeployment -ResourceGroupName az104-rg3 -TemplateFile new-template.json
```

**Disks confirmed after deployment:**

```
Get-AzDisk | ft Name,ResourceGroupName,Location,DiskSizeGb,ProvisioningState

Name  ResourceGroupName  Location  DiskSizeGB  ProvisioningState
----  -----------------  --------  ----------  -----------------
disk1 AZ104-RG3          eastus    32          Succeeded
disk2 AZ104-RG3          eastus    32          Succeeded
disk3 AZ104-RG3          eastus    32          Succeeded
```

---

## Step 4 — Deploy disk4 via Azure CLI

Cloud Shell was switched to **Bash** mode. The `disk_name` default value was changed to `disk4` in the editor and deployed using the Azure CLI.

**Template edit — disk_name defaultValue changed to `disk4`:**

<img src="./docs/03-disk4.png" alt="new-template.json open in Cloud Shell editor with disk_name defaultValue set to disk4" width="600"/>

```bash
az deployment group create --resource-group az104-rg3 --template-file new-template.json
```

**Disks confirmed after deployment:**

```
az disk list --resource-group az104-rg3 --output table

Name    ResourceGroup    Location    Sku           SizeGb    ProvisioningState
------  ---------------  ----------  ------------  --------  -------------------
disk1   az104-rg3        eastus      Standard_LRS  32        Succeeded
disk2   az104-rg3        eastus      Standard_LRS  32        Succeeded
disk3   az104-rg3        eastus      Standard_LRS  32        Succeeded
disk4   az104-rg3        eastus      Standard_LRS  32        Succeeded
```

---

## Step 5 — Deploy disk5 via Bicep Template

The `azuredeploydisk.bicep` file was uploaded to Cloud Shell. Unlike the JSON templates, Bicep uses a declarative syntax with typed parameters and decorators — significantly more readable than ARM JSON. The file was edited to set the disk name to `az104-disk5`, size to `32` GiB, and SKU to `StandardSSD_LRS`.

**Bicep template in the Cloud Shell editor:**

<img src="./docs/04-disk5.png" alt="azuredeploydisk.bicep open in Cloud Shell editor showing typed parameters with decorators for disk name, size, IOPS, throughput, and location" width="600"/>

```bash
az deployment group create --resource-group az104-rg3 --template-file azuredeploydisk.bicep
```

**All five disks confirmed after deployment:**

```
az disk list --resource-group az104-rg3 --output table

Name         ResourceGroup    Location    Sku              SizeGb    ProvisioningState
-----------  ---------------  ----------  ---------------  --------  -------------------
az104-disk5  az104-rg3        eastus      StandardSSD_LRS  32        Succeeded
disk1        az104-rg3        eastus      Standard_LRS     32        Succeeded
disk2        az104-rg3        eastus      Standard_LRS     32        Succeeded
disk3        az104-rg3        eastus      Standard_LRS     32        Succeeded
disk4        az104-rg3        eastus      Standard_LRS     32        Succeeded
```

---

## Key Learnings

### 1. Exported ARM Templates Are a Reliable Starting Point
The Portal's Export Template feature generates a working ARM template from any existing resource. It is verbose by design — capturing every configurable property — but serves as an accurate baseline to simplify and parameterize.

### 2. Parameterization Is What Makes Templates Reusable
The key improvement from `template.json` to `new-template.json` was collapsing eleven parameters down to one clean `disk_name` parameter. The same file then deployed disk2, disk3, and disk4 by simply changing one default value in the editor each time.

### 3. The Same Template Works Across All Deployment Methods
`new-template.json` deployed successfully via Portal Custom Deployment, PowerShell (`New-AzResourceGroupDeployment`), and Azure CLI (`az deployment group create`) without structural changes. The toolchain is interchangeable — the template format is consistent across all three.

### 4. Bicep Is More Readable Than ARM JSON
Bicep achieves the same result as ARM JSON with significantly less boilerplate. Decorators like `@description`, `@minValue`, and `@maxValue` make parameters self-documenting inline. Bicep compiles down to ARM JSON at deploy time — it is not a separate system, just a better authoring experience.

### 5. Each Deployment Method Has a Natural Use Case
The Portal is best for one-off deployments and exploration. PowerShell fits scripted automation in Windows-centric environments. Azure CLI is preferred in Linux/bash workflows and CI/CD pipelines. Bicep is the modern standard for maintainable IaC at scale.

---

## Overall Result

This lab demonstrated deploying the same resource type five ways using progressively more code-driven approaches:

```
Portal UI → Create disk1 → Export ARM Template
        ↓
Simplify Template → Custom Deployment Portal → disk2
        ↓
Edit Template → Azure PowerShell (Cloud Shell) → disk3
        ↓
Edit Template → Azure CLI (Cloud Shell) → disk4
        ↓
Bicep Template → Azure CLI (Cloud Shell) → az104-disk5
        ↓
All 5 Disks Validated After Each Deployment
```

**All objectives completed. Five disks deployed using five different methods.**

---

*Lab completed as part of AZ-104: Microsoft Azure Administrator certification preparation.*
