---
lab:
    title: 'Creating Resource Groups'
    module: 'Module 1: Azure Administration'
---

# Lab 01: Creating Resource Groups

An Azure Cloud Shell CLI Introduction

## Student lab manual

## Scenario

This lab introduces setting up the subscription you will use for the course in the Azure Portal and introduces the the Azure Cloud Shell using Azure CLI commands.  The course instructor will provide any the steps needed to configure the Azure subscription based on the particular course environment. You will run the courses first CLI commands in the Azure Cloud Shell by configuring two Resource Groups.

## Objectives

After you complete this lab, you will be able to:

* Set up a subscription using the Azure Portal (Course Instructor provides steps as needed)
* Create Resource Groups using the Azure CLI commands

## Lab Setup

* **Estimated time**: 20 Minutes

## Instructions

### Before you start

#### Setup Task

1. Follow the course instructor provided steps in configuring the Azure account you will use for this course using the Azure portal.

### Exercise 1: Get started in the Azure CLI using Cloud Shell and create 2 Resource Groups

The main tasks for this exercise are as follows:

1. Launch the Azure Cloud Shell
1. Set the default subscription that will be used for the labs
1. Create WestRG resource group using CLI commands
1. Create EastRG resource group using CLI commands

#### Task 1: Open the Cloud Shell

**Set subscription in the Azure Cloud Shell**

1. At the top of the portal, click the **Cloud Shell** icon to open the Cloud Shell pane.

1. At the Cloud Shell interface, select **Bash**.

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to list all subscriptions associated with the account used for your portal sign in.

```sh
az account list --output table
```

1. Review the list of subscriptions and if any is labeled as "default `true`"
1. Reset default subscription if the default is not set for your desired subscription
1. At the **Cloud Shell** command prompt, type in the following command with **your desired subscriptionID** and press **Enter** to set the default subscription.

```sh
# replace --subcription value with your subscription ID or subscription name
az account set --subscription [1111a1a1-22bb-3c33-d44d-e5e555ee5eee]
az account list --output table
```

4. Review the list of subscriptions and to ensure the proper subscription is labeled as "default `true`"

#### Task 2: Create WestRG Resource Group with CLI

1. At the **Cloud Shell** command prompt, type in the following command to create the WestRG resource group in the westus region.

```sh
az group create --location westus --name WestRG --output table
```

2. At the **Cloud Shell** command prompt, type in the following command to list available resource groups in the westus region.

```sh
az group list --output table
```

3. Verify the newly created WestRG is listed

#### Task 3: Create EastRG Resource Group with CLI

1. At the **Cloud Shell** command prompt, type in the following command to create the EastRG resource group in the eastus region.

```sh
az group create --location eastus --name EastRG --output table
```

2. At the **Cloud Shell** command prompt, type in the following command to list available resource groups in the eastus region.

```sh
az group list --output table
```

3. Verify the newly created EastRG is listed

> **Result**: In this lab you have configured your default Azure subscription and created two resource groups that we will be used further in the labs that follow.
