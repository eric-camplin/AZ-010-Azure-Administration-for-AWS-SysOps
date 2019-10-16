# Lab 04 Answer Key

## Instructions

1. Copy the below CLI script into an editor such as notepad
1. Location section titled `# ----EDIT THESE VALUES Before Running----`
1. Edit the values so they represent your environment and save the file locally
1. Log into the azure portal, open bash Cloud Shell
1. Check your dependencies are in place (See comments top of script)
1. Copy the CLI scripts from you local file and paste script into the Bash Cloud Shell
1. Some tasks can take more than 1 minute, wait for script to complete and review output
1. See instructor if errors occur

> NOTE: Student should set the default subscription as outlined in Lab 01

```sh
## AZ-010 LAB4 Solution
# ---------------
# DEPENDENCIES:
#   LAB1-solution in place (WestRG & EastRG created)
#   LAB2-solution in place (VNets, SubNets & Peering created)
#   LAB3-solution in place (EastDebianVM created)
# ---------------
# WARNING: Several Steps take more than 1 minute
# Be patient and consult instructor if errors are encountered

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
