---
lab:
    title: 'Creating Users, Groups and Policy. Monitoring logs and alerts.'
    module: 'Module 5: Azure Identity'
---

# Lab 01: Azure Identity

Create users, groups, and policy and Monitoring logs and alerts

## Student lab manual

## Scenario

In this lab you will use CLI commands in the Cloud Shell with the Bash interface to administer Azure Identity.  You will work with Azure Role-based Access Control, Azure Policy, and review monitoring using Query Explorer.

## Objectives

After you complete this lab, you will be able to:

* Create and configure users and groups with Azure Role-based Access Control
* Create a policy restricting software installs
* Review monitoring logs and alerts in the Azure Portal using Query Explorer

## Lab Setup

* **Estimated time**: 45 Minutes

## Instructions

### Before you start

#### Setup Task

1. Configuring the Azure account you will use for this course.
2. **Module 1: Azure Administration, Lab: Creating Resource Groups** WestRG resource group configured.

## Exercise 1: Create users, groups, and policy

The main tasks for this exercise are as follows:

1. Add users and groups.
1. Create a policy restricting software installs.

### Exercise 1 - Task 1: Add users and groups

**Add User**

1. Create variable for user domain.

> With email address of eric@contoso.com the command would be `my_domain=ericcontoso.onmicrosoft.com`.
>
> * *consult instructor if needed.*

2. Edit variable for user domain.

```sh
# Create a variable for user domain
# user@contoso.com =>  usercontoso.onmicrosoft.com

my_domain=<email+service>.onmicrosoft.com
```

**Create a user account**

1. Create user account name

```sh
my_user_account=AZ010@$my_domain
```

2. Create your unique strong password (edit password!).

```sh
# Edit password to be unique (remove leading "!" or error)
az ad user create \
    --display-name AZ010Tester \
    --password !sTR0ngP@ssWorD543%* \
    --user-principal-name $my_user_account
```

3. Make note of display-name, password and --user-principal-name

**Manage users and groups**

1. List AD users.

```sh
az ad user list --output json | jq '.[] | {"userPrincipalName":.userPrincipalName, "objectId":.objectId}'
```

2. The user you created in previous steps should be listed.
3. List All role assignments.

```sh
az role assignment list --all -o table
```

4. note initial state of this list.
5. List the role assignments for resource group.

```sh
az role assignment list --resource-group WestRG --output json | jq '.[] | {"principalName":.principalName, "roleDefinitionName":.roleDefinitionName, "scope":.scope}'
```

6. Add a role (`"Owner"`) to the new user.

```sh
az role assignment create --role "Owner" --assignee $my_user_account --resource-group WestRG

#az role assignment create --role "Owner" --assignee <assignee object id> --resource-group <resource_group>
```

**Review changes in users and groups**

1. Repeat listing role assignments for resource group (note changes).

```sh
az role assignment list --resource-group WestRG --output json | jq '.[] | {"principalName":.principalName, "roleDefinitionName":.roleDefinitionName, "scope":.scope}'
```

2. List the role assignments for user you created (note changes).

```sh
az role assignment list --assignee $my_user_account -g WestRG #--output json | jq '.[] | {"principalName":.principalName, "roleDefinitionName":.roleDefinitionName, "scope":.scope}'
```

## Task 2 – Create a policy restricting software installs

**Deploy software restriction policy**

> *Review rules.json and parameters.json (used below script) on [Github](https://github.com/Azure/azure-policy/tree/master/samples/built-in-policy/require-sqlserver-version12)*

1. Create policy definition.

```sh
az policy definition create --name 'require-sqlserver-version12' \
    --display-name 'Require SQL Server version 12.0' \
    --description 'This policy ensures all SQL servers use version 12.0.' \
    --rules 'https://raw.githubusercontent.com/Azure/azure-policy/master/samples/built-in-policy/require-sqlserver-version12/azurepolicy.rules.json' \
    --params 'https://raw.githubusercontent.com/Azure/azure-policy/master/samples/built-in-policy/require-sqlserver-version12/azurepolicy.parameters.json' \
    --mode All
```

2. Scope at subscription level.

```sh
az policy assignment create --name SQL12AZ010 \
    --display-name 'Require SQL Server version 12.0 - subscription scope' \
    --scope '/subscriptions/'$subscriptionID \
    --policy 'require-sqlserver-version12'
```

3. List policy assignments.

```sh
az policy assignment list
```

4. Show your newly created policy.

```sh
az policy assignment show --name 'SQL12AZ010'
```

**Check for Compliance**

1. Return to the Azure Policy service page.
2. Select Compliance. And set Compliance State filter to “All compliance states”
3. Review the status of your policy and your definition.

## Exercise 2: Monitoring logs and alerts with Query Explorer

1. Monitoring logs and alerts

### Exercise 2 - Task 1 – Review monitoring logs and alerts

**Access the demonstration environment**

1. Navigate to the [Log Analytics Querying Demonstration](https://portal.loganalytics.io/demo) in a new browser tab.
2. Use the Query Explorer
    1. Select Query Explorer (top right).
    2. Expand Favorites and then select *All Syslog records with errors*.
    3. Notice the query is added to the editing pane. Notice the structure of the query.
    4. Run the query. Explore the records returned.
    5. Follow additional steps from instructor.
    6. As you have time experiment with other Favorites and also Saved Queries.
