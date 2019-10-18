# Lab 03 pt 1 Answer Key

## Instructions

1. Copy the below CLI script into an editor such as notepad
      1. Locate section titled `# ----EDIT THESE VALUES Before Running----`
      1. Edit the Password or it will cause an **ERROR** and save the CLI commands file
      1. Save the file locally
1. Log into the azure portal, open bash Cloud Shell
1. Check your dependencies are in place (See comments top of script)
1. Copy the CLI scripts from you local file and paste script into the Bash Cloud Shell
1. Some tasks can take more than 1 minute, wait for script to complete and review output
1. See instructor if errors occur

> NOTE: Student should set the default subscription as outlined in Lab 01

```sh
# AZ-010 LAB3 Solution
# ---------------
# DEPENDENCIES:
#   LAB1-solution in place (WestRG & EastRG created)
#   LAB2-solution in place (VNets, SubNets & Peering created)
# ---------------
# WARNING: Several Steps take more than 1 minute
# Be patient and consult instructor if errors are encountered

# ----------START----------

# ----EDIT THESE VALUES to be UNIQUE Before Running----
#----remove leading "!" of password or there will be an ERROR----
adminPassword=!'UniqueP@$$w0rd-Here'

# ----Set Variables----
adminUserName='azuser'
resourceGroupName='WestRG'
location='westus'
vmName='WestWinVM'
vmSize='Standard_D1'
availabilitySet='WestAS'

# ----Main Scripts----

# ----Create availability set----
az vm availability-set create \
  --name $availabilitySet \
  --resource-group $resourceGroupName \
  --location $location

# ----Create WestWinVM VM----
az vm create --name $vmName --resource-group $resourceGroupName \
  --image win2016datacenter \
  --admin-username $adminUserName \
  --admin-password $adminPassword \
  --location $location \
  --size $vmSize \
  --availability-set $availabilitySet

# wait a minute for VM
sleep 60

# ----open ports----
az vm open-port -g WestRG -n $vmName --port 80 --priority 1500
az vm open-port -g WestRG -n $vmName --port 3389 --priority 2000

# ----Allow ICMPv4-In (Bash CLI running PowerShell)----
az vm extension set --publisher Microsoft.Compute \
--version 1.8 --name CustomScriptExtension \
--vm-name $vmName --resource-group $resourceGroupName \
--settings '{"commandToExecute":"powershell.exe New-NetFirewallRule –DisplayName “Allow ICMPv4-In” –Protocol ICMPv4"}'

# ----Install IIS (Bash CLI running PowerShell)----
az vm extension set --publisher Microsoft.Compute \
--version 1.8 \
--name CustomScriptExtension \
--vm-name $vmName \
--resource-group $resourceGroupName \
--settings '{"commandToExecute":"powershell.exe Install-WindowsFeature -name Web-Server -IncludeManagementTools"}'

# ----Check the IIS Server is running on VM public IP address----
az vm show -d -g $resourceGroupName -n $vmName --query publicIps -o tsv

# ***********************************************************
# Paste above IP address in browser to see if IIS is running
# ***********************************************************

# ----Create Debian virtual machines configured with DNS----
# ----Create WestDebianVM----
az vm create \
--image credativ:Debian:8:latest \
--admin-username azuser \
--resource-group WestRG \
--vnet-name WestVNet \
--subnet WestSubnet1 \
--availability-set WestAS \
--size 'Standard_D1' \
--location westus \
--name WestDebianVM \
--generate-ssh-keys

# ----Create EastAS Availability Set----
az vm availability-set create --name EastAS --resource-group EastRG

# ----Create EastDebianVM----
az vm create \
--image credativ:Debian:8:latest \
--admin-username azuser \
--resource-group EastRG \
--vnet-name EastVNet \
--subnet EastSubnet2 \
--availability-set EastAS \
--size 'Standard_D1' \
--location eastus \
--name EastDebianVM \
--generate-ssh-keys

# ----Configure DNS of Debian Machines----
myRand=`head /dev/urandom | tr -dc a-z0-9 | head -c 6 ; echo ''`
echo "the random string append will be:  "$myRand

# ----Configure DNS----
az network public-ip update --resource-group WestRG --name WestDebianVMPublicIP --dns-name westdebianvm$myRand

az network public-ip update --resource-group EastRG --name EastDebianVMPublicIP --dns-name eastdebianvm$myRand

# ----Ensure both Debian VMs are running----
az vm get-instance-view --name WestDebianVM --resource-group WestRG --query instanceView.statuses[1] --output table

az vm get-instance-view --name EastDebianVM --resource-group EastRG --query instanceView.statuses[1] --output table

# ----Connect to WestDebianVM with SSH and ping test WestWinVM----
# ----Connect to the Debian virtual machine in WestRG----

# ----get Public and Private IP Addresses of the VMs----
WestDebianIP=$(az vm list-ip-addresses -n WestDebianVM --query "[*].virtualMachine.network.publicIpAddresses[0].ipAddress" -o tsv)

EastDebianIP=$(az vm list-ip-addresses -n EastDebianVM --query "[*].virtualMachine.network.publicIpAddresses[0].ipAddress" -o tsv)

WestWinIP=$(az vm list-ip-addresses -n WestWinVM --query "[*].virtualMachine.network.publicIpAddresses[0].ipAddress" -o tsv)

#----SSH into the WestDebianVM virtual machine----

# *******************************************
# To Start SSH Session use the command below
echo "For SSH to WestDebianVM use: " 'ssh' $adminUserName'@'$WestDebianIP
echo "EastDebianVM IP: " $EastDebianIP
echo "WestWinVM IP: " $WestWinIP
# *******************************************

# *******************************************
# --------Start SSH Session Commands--------
# --See Lab 03 Exercise 1 Task 4: Ping VMs--
# --Note above SSH connection and IP values--
# *******************************************
```

**Complete Ex. 1 Task4: SSH and Pings**

**Continue Lab03 Answer Key solution Part 2 with LAB_AK_03_azure_compute_pt2.md**
