# Lab 05 Answer Key

## Instructions

1. Copy the below CLI script into an editor such as notepad
   1. Locate section titled `# ----EDIT THESE VALUES Before Running----`
   1. Edit the values so they represent your environment or there will be **ERRORS**
   1. save the file locally
1. Log into the azure portal, open bash Cloud Shell
1. Check your dependencies are in place (See comments top of script)
1. Copy the CLI scripts from you local file and paste script into the Bash Cloud Shell
1. Some tasks can take more than 1 minute, wait for script to complete and review output
1. See instructor if errors occur

> Student should set the default subscription (subscriptionID) as outlined in Lab 01
>
> Create variable for user domain.
> If account email address is eric@contoso.com the command would be `my_domain=ericcontoso.onmicrosoft.com`.
>
> * *consult instructor if needed.*

```sh
# AZ-010 LAB5 Solution
# ---------------
# DEPENDENCY: LAB1-solution in place (WestRG created)
# ---------------
# SET THE FOLLOWING VALUES:
#   subscriptionID
#   my_domain
# BEFORE running the below commands in Azure CLI Bash

# ----------START----------

# +++++++++++++++++++++++++++++++++++++++++++++++++++++
# ---------EDIT THESE VALUES Before Running---------
subscriptionID=[**subscription ID to use for labs**]
# Create a variable for user domai
my_domain=[**UsernameEmaildomain.onmicrosoft.com**]
# Create unique AD User Password
password_ad_user=[**sTR0ngP@ssWorD543%**]
# +++++++++++++++++++++++++++++++++++++++++++++++++++++

# ----Main Scripts----
# ----Set default subscription----
az account set --subscription $subscriptionID

#----Create user account name (user-principal-name)----
my_user_account=AZ010@$my_domain

#----Create user account----
az ad user create \
    --display-name AZ010Tester \
    --password $password_ad_user \
    --user-principal-name $my_user_account

#----List AD Users----
az ad user list --output json | jq '.[] | {"userPrincipalName":.userPrincipalName, "objectId":.objectId}'
# ====================================================
#----note new AD user listed in above output----
# ====================================================

#----Add a role (`"Owner"`) to the new user----
az role assignment create --role "Owner" --assignee $my_user_account --resource-group WestRG

#----List the role assignments for user you created (note changes)----
az role assignment list --assignee $my_user_account -g WestRG

#----Create a policy restricting software installs----
#----Create policy definition.
az policy definition create --name 'require-sqlserver-version12' \
    --display-name 'Require SQL Server version 12.0' \
    --description 'This policy ensures all SQL servers use version 12.0.' \
    --rules 'https://raw.githubusercontent.com/Azure/azure-policy/master/samples/built-in-policy/require-sqlserver-version12/azurepolicy.rules.json' \
    --params 'https://raw.githubusercontent.com/Azure/azure-policy/master/samples/built-in-policy/require-sqlserver-version12/azurepolicy.parameters.json' \
    --mode All

#----Scope at subscription level----
az policy assignment create --name SQL12AZ010 \
    --display-name 'Require SQL Server version 12.0 - subscription scope' \
    --scope '/subscriptions/'$subscriptionID \
    --policy 'require-sqlserver-version12'

#----Show your newly created policy----
az policy assignment show --name 'SQL12AZ010'

# ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
#----Exercise 2: Monitoring logs and alerts with Query Explorer-----
# ******************************************************************
# ---------------------------MANUAL STEPS---------------------------
#----Navigate to the Log Analytics Querying Demonstration----
#---------------https://portal.loganalytics.io/demo ----------------
#-----Use the Query Explorer as directed in Exercise 2 of LAB05-----

```

> **Exercise 2 - Task 1 – Review monitoring logs and alerts**
> Access the demonstration environment**
>
> 1. Navigate to the [Log Analytics Querying Demonstration](https://portal.loganalytics.io/demo) in a new browser tab.
> 2. Use the Query Explorer
>     1. Select Query Explorer (top right).
>     2. Expand Favorites and then select *All Syslog records with errors*.
>     3. Notice the query is added to the editing pane. Notice the structure of the query.
>     4. Run the query. Explore the records returned.
>     5. Follow additional steps from instructor.
>     6. As you have time experiment with other Favorites and also Saved Queries.
