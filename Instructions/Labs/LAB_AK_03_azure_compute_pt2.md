# Lab 03 pt 2 Answer Key

## Instructions

1. Copy the below CLI script into an editor such as notepad
1. Location section titled `# ----EDIT THESE VALUES Before Running----`
1. Edit the values so they represent your environment and save the file locally
1. Log into the azure portal, open bash Cloud Shell
1. Copy the CLI scripts from you local file and paste script into the Bash Cloud Shell
1. Some tasks can take more than 1 minute, wait for script to complete and review output
1. See instructor if errors occur

> NOTE: Student should set the default subscription as outlined in Lab 01

```sh
# AZ-010 LAB3 Solution
# ---------------
# DEPENDENCIES:
#   LAB1-solution in place (WestRG & EastRG created)
# ---------------
# WARNING: Several Steps take more than 1 minute
# Be patient and consult instructor if errors are encountered

# ----------START----------

# ----Main Scripts----

# ----Exercise 2: Create and Test an Ubuntu Scale Set----
# ----Create Ubuntu Scale Set----

# ----Create a scale set resource group----
az group create --name EastScaleRG --location eastus

# ----Create a scale set----
az vmss create \
  --resource-group EastScaleRG \
  --name EastUbuntuServers \
  --image UbuntuLTS \
  --upgrade-policy-mode automatic \
  --instance-count 2 \
  --admin-username azuser \
  --generate-ssh-keys

# ----Define an autoscale profile----
az monitor autoscale create \
  --resource-group EastScaleRG \
  --resource EastUbuntuServers \
  --resource-type Microsoft.Compute/virtualMachineScaleSets \
  --name autoscale \
  --min-count 1 \
  --max-count 3 \
  --count 1

# ----Create a rule to autoscale out----
az monitor autoscale rule create \
  --resource-group EastScaleRG \
  --autoscale-name autoscale \
  --condition "Percentage CPU > 50 avg 2m" \
  --scale out 1

# ----Create a rule to autoscale in----
az monitor autoscale rule create \
  --resource-group EastScaleRG \
  --autoscale-name autoscale \
  --condition "Percentage CPU < 30 avg 1m" \
  --scale in 1

# --------Test Ubuntu Scale Set (optional task)--------

# ----Use the following output to connect an SSH for a scale set Ubuntu server session----
az vmss list-instance-connection-info \
--resource-group EastScaleRG \
--name EastUbuntuServers

# *********************************************************************************
# ---- use the above output to Connect to Ubuntu scale set instance 0 using SSH----
# ---- Go to LAB03 EXERCISE2 TASK2 and complete steps to
# ----     * install/run stress
# ----     * view scaling
# *********************************************************************************

# ----Clean up Scale Set Demo by running below CLI command----
# az group delete --name EastScaleRG --yes --no-wait
```

> To run the final optional task to test the Ubuntu scale set
> **Capture the final above connection info output to Connect to Ubuntu scale set instance 0 using SSH**
> Then go to LAB03 EXERCISE2 TASK2 and complete steps to
>
> * install/run stress
> * view scaling
> * clean up scale set
>
