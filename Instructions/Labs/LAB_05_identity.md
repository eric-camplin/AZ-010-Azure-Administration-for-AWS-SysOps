---
lab:
    title: 'Creating Users and Groups, Policy and Monitoring'
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

* **Estimated time**: 40 Minutes

## Instructions

### Before you start

#### Setup Task

1. Follow the course instructor provided steps in configuring the Azure account you will use for this course using the Azure portal.

### Exercise 1: Get started in the Azure CLI using Cloud Shell and create 2 Resource Groups

The main tasks for this exercise are as follows:

1. Add users and groups
1. Create a policy restricting software installs
1. Review monitoring logs and alerts

#### Task 1: Task 1: Add users and groups

### Add User

#### Create a variable for user domain

```sh
my_domain=ecamplinhotmail.onmicrosoft.com
# user@hotmail.com =>  userhotmail.onmicrosoft.com
# my_domain=<email+service>.onmicrosoft.com
```

#### Create a user account

```sh
my_user_account=AZ010@$my_domain

# Create your unique strong password
az ad user create \
    --display-name AZ010Tester \
    --password sTR0ngP@ssWorD543%* \
    --user-principal-name $my_user_account
```

### Manage users and groups

#### List AD users

```sh
az ad user list --output json | jq '.[] | {"userPrincipalName":.userPrincipalName, "objectId":.objectId}'
```

#### List All role assignments

```sh
az role assignment list --all -o table
```

#### List the role assignments for resource group

```sh
az role assignment list --resource-group WestRG --output json | jq '.[] | {"principalName":.principalName, "roleDefinitionName":.roleDefinitionName, "scope":.scope}'
```

#### Add a role to a user

```sh
az role assignment create --role "Owner" --assignee $my_user_account --resource-group WestRG

#az role assignment create --role "Owner" --assignee <assignee object id> --resource-group <resource_group>
```

#### Repeat listing role assignments for resource group

```sh
az role assignment list --resource-group WestRG --output json | jq '.[] | {"principalName":.principalName, "roleDefinitionName":.roleDefinitionName, "scope":.scope}'
```

#### List the role assignments for resource group

```sh
az role assignment list --assignee $my_user_account -g WestRG #--output json | jq '.[] | {"principalName":.principalName, "roleDefinitionName":.roleDefinitionName, "scope":.scope}'
```

## Task 2 – Create a policy restricting software installs

### Deploy Policy

> *Review rules.json and parameters.json used below script on [Github](https://github.com/Azure/azure-policy/tree/master/samples/built-in-policy/require-sqlserver-version12)*

```sh
az policy definition create --name 'require-sqlserver-version12' \
    --display-name 'Require SQL Server version 12.0' \
    --description 'This policy ensures all SQL servers use version 12.0.' \
    --rules 'https://raw.githubusercontent.com/Azure/azure-policy/master/samples/built-in-policy/require-sqlserver-version12/azurepolicy.rules.json' \
    --params 'https://raw.githubusercontent.com/Azure/azure-policy/master/samples/built-in-policy/require-sqlserver-version12/azurepolicy.parameters.json' \
    --mode All

# scope at subscription level
az policy assignment create --name SQL12AZ010 \
    --display-name 'Require SQL Server version 12.0 - subscription scope' \
    --scope '/subscriptions/546bf0c4-dead-4700-9ea0-a8fd36bdbc5e' \
    --policy 'require-sqlserver-version12'
```

### Check for Compliance

1. Return to the Azure Policy service page.
2. Select Compliance. And set Compliance State filter to “All compliance states”
3. Review the status of your policy and your definition.

## Task 3 – Review monitoring logs and alerts

### Access the demonstration environment

[Log Analytics Querying Demonstration](https://portal.loganalytics.io/demo)

### Use the Query Explorer

1. Select Query Explorer (top right).
2. Expand Favorites and then select All Syslog records with errors.
3. Notice the query is added to the editing pane. Notice the structure of the query.
4. Run the query. Explore the records returned.
5. As you have time experiment with other Favorites and also Saved Queries.
