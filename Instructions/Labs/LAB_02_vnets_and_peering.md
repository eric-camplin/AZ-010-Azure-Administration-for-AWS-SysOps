---
lab:
    title: 'Azure Virtual Networks and Peering'
    module: 'Module 2: Azure Networks'
---
    
# Lab 02: Azure Cloud Shell CLI Intro - Create Resource Groups

# Student lab manual

## Scenario

In this lab you will use CLI commands in the Cloud Shell with the Bash interface to administer Virtual Networks (VNet). You will create VNets that span resource groups and regions, as well as configure VNet peering.

## Objectives

After you complete this lab, you will be able to use the Azure CLI to:

* Create and configure VNets with SubNets in desired resource groups.
* Configure VNet peering between VNets.

## Lab Setup

* **Estimated time**: 30 Minutes

## Instructions

### Before you start

#### Setup Task

1. EastRG and WestRG resource groups configured from **Module 1: Azure Administration, Lab: Creating Resource Groups**.

### Exercise 1: Create Virtual Networks with SubNets

The main tasks for this exercise are as follows:

1. Create West VNet with SubNet.
1. Verify West NetWork and Subnet was created.
1. Create East VNet with SubNets.
1. Verify East NetWork and Subnets were created.

#### Task 1 Create West VNet with SubNet

1. At the **Cloud Shell** command prompt, type in the following command to create the WestVNet virtual network with WestSubNet1 subnet.

```bash
az network vnet create \
  --resource-group WestRG \
  --name WestVNet \
  --address-prefix 10.1.0.0/16 \
  --subnet-name WestSubnet1 \
  --subnet-prefix 10.1.0.0/24
```

2. Verify West Network and Subnet was created.

```bash
az network vnet list --output table
```

3. Verify West Network and Subnet were created.

```bash
az network vnet subnet list --resource-group WestRG --vnet-name WestVNet --output table
```

#### Task 2 Create East VNet with SubNets

1. At the **Cloud Shell** command prompt, type in the following command to create the EastVNet virtual network with EastSubNet1 subnet.

```bash
az network vnet create \
  --resource-group EastRG \
  --name EastVNet \
  --address-prefix 10.2.0.0/16 \
  --subnet-name EastSubnet1 \
  --subnet-prefix 10.2.0.0/24
```

2. You can add additional SubNets to an existing VNet.
3. At the **Cloud Shell** command prompt, type in the following command to create the EastVNet EastSubNet2 subnet.

```bash
az network vnet subnet create \
  --resource-group EastRG \
  --vnet-name EastVNet \
  --name EastSubnet2 \
  --address-prefix 10.2.1.0/24
```

4. Verify the East NetWork (EastVNet) was created

```bash
az network vnet list --output table
```

5. Verify East Network Subnets were created

```bash
az network vnet subnet list --resource-group EastRG --vnet-name EastVNet --output table
```

#### Task 3: Create Peering Network West to East

1. Create Peering between West and East VNets using `remote-vnet-id` CLI command
1. At the **Cloud Shell** command prompt, type in the following command to Capture the EastVNet ID in a variable.

```bash
EastVNetId=$(az network vnet show \
  --resource-group EastRG \
  --name EastVNet \
  --query id --out tsv)

  echo "EastVNetId = " $EastVNetId
```

3. Type in the following command to peer WestVNet to EastVNet

```bash
az network vnet peering create \
  --name WesttoEastPeering \
  --resource-group WestRG \
  --vnet-name WestVNet \
  --remote-vnet $EastVNetId \
  --allow-vnet-access
```

> *Note: using Cloud Shell `--remote-vnet-id $EastVNetId` is not used and throws a warning but may be required in older CLI Shells*

4. Type in the following command to Verify State of peering

```bash
az network vnet peering list \
  --resource-group WestRG \
  --vnet-name WestVNet \
  --output table
  ```

#### Task 4: Create Peering Network East to West

1. Capture the WestVNet ID in a variable

```bash
WestVNetId=$(az network vnet show \
  --resource-group WestRG \
  --name WestVNet \
  --query id --out tsv)
  
echo "WestVNetId = " $WestVNetId
```

2. Type in the following command to peer EastVNet to WestVNet

```bash
az network vnet peering create \
  --name EasttoWestPeering \
  --resource-group EastRG \
  --vnet-name EastVNet \
  --remote-vnet $WestVNetId \
  --allow-vnet-access
```

3. Type in the following command to Verify State of peering

```bash
az network vnet peering list \
  --resource-group EastRG \
  --vnet-name EastVNet \
  --output table
  ```

> **Result**: In this lab you have configured East and West VirtualNetworks and Subnets and created peering between the networks (East to West and West to East). These resources will be used further in the labs that follow.
