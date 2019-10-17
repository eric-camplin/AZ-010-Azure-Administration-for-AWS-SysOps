# Lab 04 Answer Key

## Instructions

1. Copy the below CLI script into an editor such as notepad
1. Location section titled `# ----EDIT THESE VALUES Before Running----`
1. **Edit the values** so they represent your environment and save the file locally
1. Log into the azure portal, open bash Cloud Shell
1. Check your dependencies are in place (See comments top of script)
1. Copy the CLI scripts from you local file and paste script into the Bash Cloud Shell
1. Some tasks can take more than 1 minute, wait for script to complete and review output
1. See instructor if errors occur

> Note regarding `EDIT THESE VALUES` Section
>
>Create the `az_user_name` variable to be used in `myUploadPath` and `myDownloadPath`
>
> Need to edit the command that follows to set `az_user_name`
>
> Capture the **name** from `<name>@azure` in the cloud shell prompt
> e.g. - if the prompt in your cloud shell is `eric@Azure:~$` then set **`az_user_name='eric'`**

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
subscriptionID=<**subscription ID to use for labs**>

az_user_name=<name>

# ----Main Scripts----
# ----Setting default subscription----
az account set --subscription $subscriptionID

# ----Create random string to use for unique names----

myRand=`head /dev/urandom | tr -dc a-z0-9 | head -c 6 ; echo ''`
echo "the random string append will be:  "$myRand

# ----Set variables----
my_resource_group=WestRG
location=westus
my_storage_account=weststore$myRand
my_storage_sku=Standard_RAGRS # default
my_storage_kind=StorageV2myRand
my_access_tier=hot
my_storage_encryption=blob

# **************************************************************
echo "Your my_storage_account name will be: " $my_storage_account
# **************************************************************

# ----Create storage account----
az storage account create \
    -n $my_storage_account \
    -g $my_resource_group \
    -l $location \
    --kind $my_storage_kind \
    --access-tier $my_access_tier \
    --sku $my_storage_sku \
    --encryption $my_storage_encryption

# ----Set Environment Variables in Bash CLI----
export AZURE_STORAGE_ACCOUNT=$my_storage_account
export AZURE_STORAGE_CONNECTION_STRING="$(az storage account show-connection-string \
-n $AZURE_STORAGE_ACCOUNT -g $my_resource_group --query connectionString -o tsv)"

# ----Create storage account keys environment variables----
# Display Keys
az storage account keys list \
    --account-name $AZURE_STORAGE_ACCOUNT \
    --resource-group $my_resource_group \
    --output table

# ----export AZURE_STORAGE_KEY=<storage_account_key1>----
# ----store key1 as environment variable----
export AZURE_STORAGE_KEY="$(az storage account keys list --account-name \
$AZURE_STORAGE_ACCOUNT --resource-group $my_resource_group --query [0].value -o tsv)"

# ----Make a file `helloAdmin.html` to upload to blob storage
echo "<h1>Hello Azure Administrators</h1>">helloAdmin.html

# ----Create a blob container with Public Access----
container_name=westblobcontainerpublic

az storage container create --name $container_name --public-access blob

# ----Upload file to public blob container----
blob_name=helloAdmin

az storage blob upload \
    --container-name $container_name \
    --file helloAdmin.html \
    --name $blob_name

# ----Download public file----
az storage blob download \
    --container-name $container_name \
    --name $blob_name \
    --file helloAdminDownload.html

# ---------upload-batch---------
mkdir uploadfiles

# ----Create files to upload in the `upload` directory----
touch uploadfiles/myfile.html
touch uploadfiles/more.html
touch uploadfiles/hi.html
touch uploadfiles/myfile.txt

# ***********************************************************
# ASSUMES REPLACED <name> in edited variables section above
# az_user_name=<name>
# ***********************************************************

# path to $home in cloud shell
myUploadPath=/home/$az_user_name/uploadfiles

# Upload html files in path
az storage blob upload-batch -d $container_name \

#----Create the upload Path and folder----
mkdir downloadfiles

#----create path for download----
myDownloadPath=/home/$az_user_name/downloadfiles

#----Bach download files from container----
az storage blob download-batch -d $myDownloadPath -s $container_name

#----Get URL of blob and view public file----
#----click the link generated to see the file
az storage blob url -c $container_name -n helloAdmin -o tsv

# ----Task 2: Secure access to storage----
# ----Create a private blob container in the west storage account----

# ----Refresh variables----
container_name=westblobcontainerprivate

# ----Storage Key----
export AZURE_STORAGE_KEY="$(az storage account keys list \
    --account-name $AZURE_STORAGE_ACCOUNT \
    --resource-group $my_resource_group \
    --query [0].value -o tsv)"

# ----Storage Connection Key----
export AZURE_STORAGE_CONNECTION_STRING="$(az storage account \
    show-connection-string -n $AZURE_STORAGE_ACCOUNT\
     -g $my_resource_group --query connectionString -o tsv)"

# ----Create a new blob container that is private
az storage container create --name $container_name --public-access blob

# ----Make & Upload file to private blob container----
echo "<h1>Hello Azure Administrators - This is private</h1>">helloAdmin.html
blob_name=helloAdmin

az storage blob upload \
    --container-name $container_name \
    --file helloAdmin.html --name $blob_name

# ----Create Shared Access Signature (SAS) at the service level----
end_date=`date -u -d "30 minutes" '+%Y-%m-%dT%H:%MZ'`

CONTAINER_SAS_KEY="$(az storage container generate-sas \
    --name $container_name --https-only --auth-mode key \
    --expiry $end_date --permissions r -o tsv)"

BLOB_SAS_KEY="$(az storage blob generate-sas \
    --name $blob_name \
    --container-name $container_name \
    --permissions r --expiry $end_date \
    --auth-mode key -o tsv)"


# ----Generate full Blob URI and Test Link----
private_URI="$(az storage blob generate-sas \
    --name $blob_name \
    --container-name $container_name \
    --permissions r --auth-mode key \
    --expiry $end_date --https-only \
    --full-uri)"

echo $private_URI
# ***********************************************************************
# ----Test link generated above----
# ***********************************************************************


# ---------Task 3: Create a subnet service endpoint---------
# ----Set the default rule to deny network access by default (if needed)----
az storage account update --resource-group $my_resource_group \
    -n $AZURE_STORAGE_ACCOUNT --default-action Deny

# ----Update subnet rules to enable storage on WestVNet - WestSubnet1.----
az network vnet subnet update --resource-group $my_resource_group \
    --vnet-name WestVNet --name WestSubnet1 \
    --service-endpoints "Microsoft.Storage"

# ----Create `subnet_id` variable
subnet_id="$(az network vnet subnet show \
    --resource-group $my_resource_group \
    --vnet-name WestVNet --name WestSubnet1 \
    --query id --output tsv)"

# ----Add network subnet rule.
az storage account network-rule add --resource-group $my_resource_group \
    -n $AZURE_STORAGE_ACCOUNT --subnet $subnet_id

# ----List storage account network rule updates----
az storage account network-rule list --resource-group $my_resource_group \
    -n $AZURE_STORAGE_ACCOUNT

# **********************************************************
# ----Test the page that was accessible in previous task
# ----network rule should **deny access**
echo $private_URI
# TEST Above Link
# *********************************************************

# ----Task 4 – Create file storage----
# ----Create a eastus storage account using Azure CLI----

# ----Create variables----
my_resource_group=EastRG
location=eastus
my_storage_account=eaststore$myRand

#----Create eastus storage account----
az storage account create --name $my_storage_account \
    --resource-group $my_resource_group --location $location \
    --sku Standard_LRS --kind StorageV2

#----allow non-https traffic (avoids "error 13"  mounting Linux drive)----
az storage account update -n $my_storage_account --https-only false

#---------Create a file share and upload a file---------
#----Set environment variables----
export AZURE_STORAGE_ACCOUNT=$my_storage_account
export AZURE_STORAGE_KEY="$(az storage account keys list --account-name $AZURE_STORAGE_ACCOUNT --resource-group $my_resource_group --query [0].value -o tsv)"
export AZURE_STORAGE_CONNECTION_STRING="$(az storage account show-connection-string -n $AZURE_STORAGE_ACCOUNT -g $my_resource_group --query connectionString -o tsv)"
#----Create `eastfiles` file share----
file_share_name=eastfiles
#----Create storage file share----
az storage share create --name $file_share_name --quota 2048
#----Create Create `myFileShareFile.html` file to upload
echo "File Shares Share Files">myFileShareFile.html
#----Upload file to share----
az storage file upload --share-name $file_share_name --source ~/myFileShareFile.html

# ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
# ******************************************************************
# ---------------------------MANUAL STEPS---------------------------
#----Mount the file share from a Linux virtual machine----
#----SSH to Linux VM----
#----Type these COMMANDS into clould shell bash to Start SSH Session----
#> east_vm_ip="$(az vm show -d -g $my_resource_group -n eastdebianvm --query publicIps -o tsv)"
#> ssh azuser@$east_vm_ip
#
#----Type these COMMANDS into the SSH Session (or see Task 4 of lab04)----
#----to make a directory and attach to Azure storage----
#ssh> mkdir -p $my_storage_account/eastfiles
#ssh> sudo apt-get update
#ssh> sudo apt-get install cifs-utils
# ==================================================================
#----Get code from the portal to attach  storage to Linux machine---
#
#    1. In the portal:
#        * open your eaststorage*123af4* (similar name) storage account.
#        * navigate to the eastfiles file share.
#    1. From the "eastfiles" blade click Connect.
#    1. Change to the **Linux** tab and copy the connection string.
#    1. Return to your EastDebianVM ssh session.
#    1. Paste the connection string in the ssh session
#
# ==================================================================
# ====Open New Azure Portal web page tab, launch the Cloud Shell====
#----Stop allowing non-https traffic now that share is mounted (CLI)
#----Type the command in to the **Azure CLI** (NOT the SSH session)
#
#> az storage account update -n $my_storage_account --https-only true
#
# ==================================================================
# =================Return to the **Linux SSH Session================
#----In Linux SSH Session continue with following commands----
#
#ssh> cd /mnt/$my_storage_account
#ssh> echo "I am from eastDebianVM">newFile.txt
#----View files in the share on the linux machine using "ls" command
#----to exit SSH session type "exit"----
#----Confirm the file (newFile.txt) is present in the Azure Portal

# =================================================================
# -----------Clean Up Storage Lab when ready-----------
#----------RETURN TO CLI (exit the ssh session)---------
#---remove the file storage account with following commands----
#> az storage account delete -n eaststore$myRand -g EastRG
#> az storage account delete -n weststore$myRand -g WestRG
#
```

> Review review the command scripts above for
> * verification steps
> * manual steps for SSH
> * Manual Clean up steps
