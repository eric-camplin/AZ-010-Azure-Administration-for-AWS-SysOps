# Lab 01 Answer Key

## Instructions

1. Copy the below CLI script into an editor such as notepad
   1. Location section titled `# ----EDIT THESE VALUES Before Running----`
   1. Edit the subscriptionID or there will be an error
   1. save the file locally
1. Log into the azure portal, open bash Cloud Shell
1. Copy the CLI scripts from you local file and paste script into the Bash Cloud Shell
1. Some tasks can take more than 1 minute, wait for script to complete and review output
1. See instructor if errors occur

> NOTE: Student should set the default subscription as outlined in Lab 01

```sh
# AZ-010 LAB1 Solution
# SET THE FOLLOWING VALUES:
#   subscriptionID
# BEFORE running the below commands in Azure CLI Bash
# View Subscriptions using: az account list -o table

# ----------START----------

# ----EDIT THESE VALUES Before Running----
subscriptionID=[**subscription ID to use for labs**]

# ----Main Scripts----
# ----Setting default subscription----
az account set --subscription $subscriptionID
# ---Note the Default Subscription---
az account list --output table

# ----Creating WestRG----
az group create --location westus --name WestRG --output table

# ----Creating EastRG----
az group create --location eastus --name EastRG --output table

# ----Listing resource groups----
az group list -o table
```
