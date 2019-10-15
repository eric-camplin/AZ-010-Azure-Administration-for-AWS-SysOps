---
lab:
    title: 'Creating USers and Groups, Policy and Monitoring'
    module: 'Module 5: Azure Identity'
---

# Lab 01: Azure Identity

Create users and groups, policy and monitoring

## Student lab manual

## Scenario

In this lab you will use CLI commands in the Cloud Shell with the Bash interface to administer Azure Identity.  You will work with Azure Role-based Access Control, Azure Policy, and review monitoring using Query Explorer.

## Objectives

After you complete this lab, you will be able to:

* Create and configure users and groups
* Create a policy restricting software installs
* Review monitoring logs and alerts in the Azure Portal using Query Explorer

## Lab Setup

* **Estimated time**: 20 Minutes

## Instructions

### Before you start

#### Setup Task

1. Follow the course instructor provided steps in configuring the Azure account you will use for this course using the Azure portal.

### Exercise 1: Get started in the Azure CLI using Cloud Shell and create 2 Resource Groups

The main tasks for this exercise are as follows:

1. Add users and groups
1. Create a policy restricting software installs
1. Review monitoring logs and alerts

#### Task 1: Open the Cloud Shell

**Set subscription in the Azure Cloud Shell**

1. At the top of the portal, click the **Cloud Shell** icon to open the Cloud Shell pane.

1. At the Cloud Shell interface, select **Bash**.

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to list all subscriptions associated with the account used for your portal sign in.

```bash
az account list --output table
```

1. Review the list of subscriptions and if any is labeled as "default `true`"
1. Reset default subscription if the default is not set for your desired subscription
1. At the **Cloud Shell** command prompt, type in the following command with **your desired subscriptionID** and press **Enter** to set the default subscription.

```bash
# replace --subcription value with your subscription ID or subscription name
az account set --subscription [1111a1a1-22bb-3c33-d44d-e5e555ee5eee]
az account list --output table
```

4. Review the list of subscriptions and to ensure the proper subscription is labeled as "default `true`"

#### Task 2: Create WestRG Resource Group with CLI

1. At the **Cloud Shell** command prompt, type in the following command to create the WestRG resource group in the westus region.

```bash
az group create --location westus --name WestRG --output table
```

2. At the **Cloud Shell** command prompt, type in the following command to list available resource groups in the westus region.

```bash
az group list --output table
```

3. Verify the newly created WestRG is listed

#### Task 3: Create EastRG Resource Group with CLI

1. At the **Cloud Shell** command prompt, type in the following command to create the EastRG resource group in the eastus region.

```bash
az group create --location eastus --name EastRG --output table
```

2. At the **Cloud Shell** command prompt, type in the following command to list available resource groups in the eastus region.

```bash
az group list --output table
```

3. Verify the newly created EastRG is listed

> **Result**: In this lab you have configured your default Azure subscription and created two resource groups that we will be used further in the labs that follow.
