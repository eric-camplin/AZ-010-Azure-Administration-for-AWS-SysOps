# Lab 02 Answer Key

## Instructions

1. Log into the azure portal, open bash Cloud Shell
1. Copy the below CLI script
1. Paste script into the Bash Cloud Shell
1. Some tasks can take more than 1 minute, wait for script to complete and review output
1. See instructor if errors occur

> NOTE: Student should set the default subscription as outlined in Lab 01

```sh
# AZ-010 LAB2 Solution
# ---------------
# DEPENDENCY: LAB1-solution in place (WestRG & EastRG created)
# ---------------
# WARNING: Several Steps take more than 1 minute
# Be patient and consult instructor if errors are encountered

# ----------START----------

echo ----Creating WestVNet and WestSubNet1----
az network vnet create \
  --resource-group WestRG \
  --name WestVNet \
  --address-prefix 10.1.0.0/16 \
  --subnet-name WestSubnet1 \
  --subnet-prefix 10.1.0.0/24

echo ----Verify WestVNet and WestSubNet1 created----
az network vnet subnet list --resource-group WestRG \
--vnet-name WestVNet --output table

echo ----Creating EastVNet and EastSubNet1----
az network vnet create \
  --resource-group EastRG \
  --name EastVNet \
  --address-prefix 10.2.0.0/16 \
  --subnet-name EastSubnet1 \
  --subnet-prefix 10.2.0.0/24

echo ----Creating EastSubNet2 on EastVNet----
az network vnet subnet create \
  --resource-group EastRG \
  --vnet-name EastVNet \
  --name EastSubnet2 \
  --address-prefix 10.2.1.0/24

echo ----Verify EastVNet created----
az network vnet list --output table

echo ----Verify EastVNet SubNets were created----
az network vnet subnet list --resource-group EastRG --vnet-name EastVNet --output table

echo ------START Network Peering West to East------
echo ----Capture EastVNet ID in a variable----
EastVNetId=$(az network vnet show \
  --resource-group EastRG \
  --name EastVNet \
  --query id --out tsv)

  echo "EastVNetId = " $EastVNetId

echo ----Peer West to East----
az network vnet peering create \
  --name WesttoEastPeering \
  --resource-group WestRG \
  --vnet-name WestVNet \
  --remote-vnet $EastVNetId \
  --allow-vnet-access

echo ----Verify state of peering----
az network vnet peering list \
  --resource-group WestRG \
  --vnet-name WestVNet \
  --output table

echo ------START Network Peering West to East------
echo ----Capture WestVNet ID in a variable----
WestVNetId=$(az network vnet show \
  --resource-group WestRG \
  --name WestVNet \
  --query id --out tsv)
  
echo "WestVNetId = " $WestVNetId

echo ----Peer East to West----
az network vnet peering create \
  --name EasttoWestPeering \
  --resource-group EastRG \
  --vnet-name EastVNet \
  --remote-vnet $WestVNetId \
  --allow-vnet-access

echo ----Verify state of peering----
az network vnet peering list \
  --resource-group EastRG \
  --vnet-name EastVNet \
  --output table
```
