---
lab:
    title: 'Azure VM and Scale Set'
    module: 'Module 3: Azure Compute'
---
    
# Lab 03: Azure Compute

## Student lab manual

## Scenario

In this lab you will use CLI commands in the Cloud Shell with the Bash interface to administer Azure Windows and Debian VMs within an Availability Set. Also, the Lab will create an Ubuntu Scale Set with an optional test to demonstrate scaling in and scaling out.

## Objectives

After you complete this lab, you will be able to use the Azure CLI to:

* Create a Scale Set
* Create and configure Windows and Linux VMs
* Configure an Ubuntu Scale Set

## Lab Setup

* **Estimated time**: 60 Minutes

## Instructions

### Before you start

#### Setup Task

1. **Dependency on previous labs:**
    1. Module 1: Azure Administration - **Lab: Creating Resource Groups**. EastRG and WestRG resource groups configured.
    1. Module 2: Azure Networking - **Lab: Virtual Networks and Peering**. VNets with SubNets and peering configured.

## Exercise 1: Create VMs configured within availability sets

The main tasks for this exercise are as follows:

1. Create an availability set
1. Create a Windows virtual machine
1. Create Linux Debian virtual machines

### Exercise 1 - Task 1: Create Windows virtual machine

* Create an availability set
* Create Windows Server 2016 DataCenter VM
* Configure as WebServer

**Prepare variables**

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to prepare variables for use in the following scripts.

> **Note**: create a Unique Password and write it down (remove leading "!" of password or there will be an error)

```sh
resourceGroupName='WestRG'
location='westus'
adminUserName='azuser'
adminPassword=!'UniqueP@$$w0rd-Here' # remove "!" or error + make unique
vmName='WestWinVM'
vmSize='Standard_D1'
availabilitySet='WestAS'
```

**Create availability set**

1. Type in the following command to create the **WestAS** Availability Set.

```sh
az vm availability-set create \
  --name $availabilitySet \
  --resource-group $resourceGroupName \
  --location $location
```

**Create WestWinVM VM**

1. Type in the following command to create the **WestWinVM** Windows Server VM.

```sh
az vm create --name $vmName --resource-group $resourceGroupName \
  --image win2016datacenter \
  --admin-username $adminUserName \
  --admin-password $adminPassword \
  --size $vmSize \
  --location $location \
  --availability-set $availabilitySet
  ```

**open ports**

1. Type in the following command to open ports in the WestWinVM.

```sh
az vm open-port -g WestRG -n $vmName --port 80 --priority 1500
az vm open-port -g WestRG -n $vmName --port 3389 --priority 2000
```

**Check your work**

1. Launch the Azure Portal navigate to the WestRG
2. Note the Resources, including the WestWinVM
3. Launch Azure Advisor and note the recommendations

### Task 2: Configure WestWinVM as a Web Server and allow Ping

**Allow ICMPv4-In (Bash CLI running PowerShell)**

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to allow ICMPv4-In by sending a PowerShell command to the WestWinVM Windows Server.

> **Note**: If Cloud shell has timed out you may need to refresh the variables from Task 1.

```sh
az vm extension set --publisher Microsoft.Compute \
--version 1.8 --name CustomScriptExtension \
--vm-name $vmName --resource-group $resourceGroupName \
--settings '{"commandToExecute":"powershell.exe New-NetFirewallRule –DisplayName “Allow ICMPv4-In” –Protocol ICMPv4"}'
```

**Install IIS (Bash CLI running PowerShell)**

1. Type in the following command to Install IIS sending a PowerShell command to the WestWinVM Windows Server.

```sh
az vm extension set --publisher Microsoft.Compute \
--version 1.8 \
--name CustomScriptExtension \
--vm-name $vmName \
--resource-group $resourceGroupName \
--settings '{"commandToExecute":"powershell.exe Install-WindowsFeature -name Web-Server -IncludeManagementTools"}'
```

**Check the IIS Server is running on VM public IP address**

1. Type in the following command to capture the WestWSinVM public IP address.

```sh
az vm show -d -g $resourceGroupName -n $vmName --query publicIps -o tsv

# ----Paste above IP address in browser to see if IIS is running----
```

2. Paste the resulting IP address into a web browser to verify IIS default page is present.

### Task 3: Create Debian virtual machines configured with DNS

Create two Debian virtual servers in Cloud Shell and connect to servers with SSH

Machine 1: **WestDebianVM**

> - WestRG resource group
>   - WestVNet
>      - WestSubnet1

and

Machine 2: **EastDebianVM**

> - East resource group
>   - EastVNet
>     - EastSubnet2 subnet

**Create WestDebianVM**

1. Type in the following command to create WestDebianVM

```sh
az vm create \
--image credativ:Debian:8:latest \
--size 'Standard_D1' \
--admin-username azuser \
--resource-group WestRG \
--vnet-name WestVNet \
--subnet WestSubnet1 \
--availability-set WestAS \
--location westus \
--name WestDebianVM \
--generate-ssh-keys
```

2. Note shh key will be generated (if it doesn't exist) and saved in the `~/.ssh` directory. Type in the following command in the cloud shell to see directory contents of the `~/.ssh` directory.

```sh
ls .ssh
```

**Create EastAS Availability Set**

1. Type in the following command to create EastAS

```sh
az vm availability-set create --name EastAS --resource-group EastRG
```

**Create EastDebianVM**

1. Type in the following command to create EastDebianVM

```sh
az vm create \
--image credativ:Debian:8:latest \
--admin-username azuser \
--resource-group EastRG \
--vnet-name EastVNet \
--subnet EastSubnet2 \
--availability-set EastAS \
--location eastus \
--name EastDebianVM \
--generate-ssh-keys
```

**Configure DNS of Debian Machines**

1. Type in the following command to create random string using Bash for use in assigning unique DNS names.

```sh
myRand=`head /dev/urandom | tr -dc a-z0-9 | head -c 6 ; echo ''`
echo "the random string append will be:  "$myRand
```

**Configure DNS**

1. Type in the following command to assigning unique DNS names to the Linux Debian VMs.

```sh
az network public-ip update --resource-group WestRG --name WestDebianVMPublicIP --dns-name westdebianvm$myRand

az network public-ip update --resource-group EastRG --name EastDebianVMPublicIP --dns-name eastdebianvm$myRand
```

2. Note the DNS Names of the Linux Debian VMs.

**Ensure both Debian VMs are running**

1. Type in the following command to see the status of the Linux Debian VMs (They Should be running).

```sh
az vm get-instance-view --name WestDebianVM --resource-group WestRG --query instanceView.statuses[1] --output table

az vm get-instance-view --name EastDebianVM --resource-group EastRG --query instanceView.statuses[1] --output table
```

### Task 4: Connect to WestDebianVM with SSH and ping test WestWinVM

**Connect to the Debian virtual machine in the West resource group**

1. Type in the following command to get Public and Private IP Addresses of the VMs

```sh
az vm list-ip-addresses --resource-group WestRG

az vm list-ip-addresses --resource-group EastRG
```

2. Note the values EastDebianVM, WestDebianVM and WestWinVM IP addresses and **Record the IPs** (or find in each VM's overview page in portal).

**SSH into the WestDebianVM virtual machine**

1. Use the WestDebianVM IP address to edit the following command to start the SSH session

```sh
ssh azuser@<PUBLIC IP address of West Debian VM>
```

**Ping the Windows VM private address from the SSH session**

> *This should work since both are on the same **private** VNet*

1. Use the WestWinVM IP address to edit the following command and type in the cloud shell SSH session to ping the WestWinVM.
1. **`ctrl+c`** to end ping in bash (ssh session)

```sh
ping <PRIVATE IP address of the Windows server>
```

**Ping EastDebianVM**

1. Use the EastDebianVM IP address to edit the following command and type in the cloud shell SSH session to ping the WestWinVM.

> *This should work due to previously configured VNet peering*

2. **`ctrl+c`** to end ping in bash (ssh session)

```sh
ping <PRIVATE IP address of eastdebianvm>
```

> **Note**: type `exit` to exit SSH

## Exercise 2: Create and Test an Ubuntu Scale Set

The main tasks for this exercise are as follows:

1. Create Ubuntu scale set and autoscale profile
1. Create autoscale out and autoscale in rules
1. Test Ubuntu Scale Set (optional)

### Exercise 2 - Task 1: Create Ubuntu Scale Set

**Create a scale set resource group**

1. Type in the following command to create a resource group for your scale set

```sh
az group create --name EastScaleRG --location eastus
```

**Create a scale set**

1. Type in the following command to create scale set

```sh
az vmss create \
  --resource-group EastScaleRG \
  --name EastUbuntuServers \
  --image UbuntuLTS \
  --upgrade-policy-mode automatic \
  --instance-count 2 \
  --admin-username azuser \
  --generate-ssh-keys
```

View Portal Dashboard and note 2 servers before scale rules

> - Resource groups
>   - EastScaleRG
>     - EastUbuntuServers > Instances

**Define an autoscale profile**

1. Type in the following command to define an auto scale profile

```sh
az monitor autoscale create \
  --resource-group EastScaleRG \
  --resource EastUbuntuServers \
  --resource-type Microsoft.Compute/virtualMachineScaleSets \
  --name autoscale \
  --min-count 1 \
  --max-count 3 \
  --count 1
  ```

> *Should see message* "follow up with `az monitor autoscale rule create` to add scaling rules."

**Create a rule to autoscale out**

1. Type in the following command to create **autoscale out** rule

```sh
az monitor autoscale rule create \
  --resource-group EastScaleRG \
  --autoscale-name autoscale \
  --condition "Percentage CPU > 50 avg 2m" \
  --scale out 1
```

**Create a rule to autoscale in**

1. Type in the following command to create **autoscale in** rule

```sh
az monitor autoscale rule create \
  --resource-group EastScaleRG \
  --autoscale-name autoscale \
  --condition "Percentage CPU < 30 avg 1m" \
  --scale in 1
```

> *Note: "scale in" can occur in as little as 2 minutes so result below is expected to differ over the first few minutes*

### Task 2: Test Ubuntu Scale Set (optional task)

> This Task will generate CPU load on a sever as a test to demonstrate the scale set behavior.

**list the servers running in the scale set**

* In the first minute, the script should list 2 server instances (e.g. 0 & 1)
* Sometime after a minute only 1 server instance (e.g. 0) should be listed

1. Type in the following command to list the servers
1. **Repeat** the command after 30 seconds until only 1 server instance is listed

```sh
az vmss list-instance-connection-info \
--resource-group EastScaleRG \
--name EastUbuntuServers
```

**Connect to instance 0 using SSH**

1. Use the portal to get the IP Address of instance 0 of the scale set
1. Type in the following edited command to connect with SSH

```sh
# example: ssh azuser@13.92.224.66 -p 50000  
ssh azuser@<instance 0 IP> -p 50000
```

**Run Stress for 4 minutes (240 seconds)**

1. Type in the following command in the SSH session to install Stress application (times out in 240 seconds)

```sh
sudo apt-get -y install stress
sudo stress --cpu 10 --timeout 240 &
```

**Confirm stress in SSH session**

1. Type in the following command in SSH to monitor the stress

```sh
top
```

**exit and SSH**

```sh
# ctrl-c
exit
```

**Watch for Autoscale Out and Autoscale In**

1. Type in the following command to Run monitoring on the Autoscale

```sh
watch az vmss list-instances \
  --resource-group EastScaleRG \
  --name EastUbuntuServers \
  --output table
```

> **Note**: may take a few minutes for stress to start registering a load over 50%
>
> **Note**: ctrl+c to close the "watch"

**Clean up Scale Set Demo**

```sh
az group delete --name EastScaleRG --yes --no-wait
```
