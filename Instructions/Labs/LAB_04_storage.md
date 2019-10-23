---
lab:
    title: 'Blobs, Secure Access Storage, Service Endpoints and File Storage'
    module: 'Module 4: Azure Storage'
---

# Lab 04: Azure Storage

Storage Accounts, Blobs, Secure Access Storage, Service Endpoint and File Storage

## Student lab manual

## Scenario

In this lab you will use CLI commands in the Cloud Shell with the Bash interface to administer Azure Storage. You will configure storage accounts, Azure Blob storage , Secure Access Storage (SAS), a SubNet Service Endpoint and File Storage.

## Objectives

After you complete this lab, you will be able to use the Azure CLI to create and configure:

* Storage Accounts
* Blob Storage
* Secure Access Storage (SAS)
* A SubNet Service Endpoint
* File Storage

## Lab Setup

* **Estimated time**: 60 Minutes

## Instructions

### Before you start

#### Setup Task

1. **Dependency on previous labs:**
    1. Module 1: Azure Administration - **Lab: Creating Resource Groups**. EastRG and WestRG resource groups configured.
    1. Module 2: Azure Networking - **Lab: Virtual Networks and Peering**. VNets with SubNets and peering configured.
    1. Module 3: Azure compute - **Lab: Azure VM**. EastDebianVM created.

## Exercise 1: Blobs, Secure Access Storage, Service Endpoint and File Storage

The main tasks for this exercise are as follows:

1. Create and configure Blob Storage
1. Configure Blob with Secure Access Storage (SAS)
1. Configure a SubNet Service Endpoint
1. Create and Configure File Storage

### Exercise 1 - Task 1: Create a Storage Account with Blob Storage

**Create storage account variables**

1. Create random string to use for unique names

```sh
myRand=`head /dev/urandom | tr -dc a-z0-9 | head -c 6 ; echo ''`
echo "the random string append will be:  "$myRand
```

2. Set variables


```sh
# set variables
my_resource_group=WestRG
location=westus
my_storage_account=weststore$myRand
my_storage_sku=Standard_RAGRS # default
my_storage_kind=StorageV2
my_access_tier=hot
my_storage_encryption=blob

echo "Your my_storage_account name will be: " $my_storage_account
```

>Note the name of your storage account from the above commands

**Create storage account**

```sh
# create account
az storage account create \
    -n $my_storage_account \
    -g $my_resource_group \
    -l $location \
    --kind $my_storage_kind \
    --access-tier $my_access_tier \
    --sku $my_storage_sku \
    --encryption $my_storage_encryption
```

**Set Environment Variables in Bash CLI**

1. Create environment variables for
   * storage account
   * storage account connection string

```sh
# set environment variables
export AZURE_STORAGE_ACCOUNT=$my_storage_account

export AZURE_STORAGE_CONNECTION_STRING="$(az storage account show-connection-string \
-n $AZURE_STORAGE_ACCOUNT -g $my_resource_group --query connectionString -o tsv)"
```

2. Create storage account keys environment variables

```sh
# Display Keys
az storage account keys list \
    --account-name $AZURE_STORAGE_ACCOUNT \
    --resource-group $my_resource_group \
    --output table

# export AZURE_STORAGE_KEY=<storage_account_key1>
# store key1 as environment variable
export AZURE_STORAGE_KEY="$(az storage account keys list --account-name \
$AZURE_STORAGE_ACCOUNT --resource-group $my_resource_group --query [0].value -o tsv)"
```

> Tip: Capture all of the variables and environment variables commands in a text file so you can easily re-enter if the cloud shell times out after 20 minutes and the variables are lost from the temporary memory.

**Prepare a file to add to blob storage**

1. Make a file `helloAdmin.html` to upload to blob storage

```sh
# Make a file to add to the container as a blob
echo "<h1>Hello Azure Administrators</h1>">helloAdmin.html
```

2. check that the file `helloAdmin.html` is created

```sh
ls
```

**Create a public blob container in the west storage account**

1. Create a blob container with Public Access

```sh
container_name=westblobcontainerpublic

az storage container create --name $container_name --public-access blob
# --connection-string $AZURE_STORAGE_CONNECTION_STRING
```

**Upload file to public blob container**

```sh
blob_name=helloAdmin

az storage blob upload \
    --container-name $container_name \
    --file helloAdmin.html \
    --name $blob_name
    #--connection-string $AZURE_STORAGE_CONNECTION_STRING
```

**Download public file**

1. Download the file you previously uploaded
2. Ensure `helloAdmin.html` is downloaded with new name `helloAdminDownload.html`

```sh
az storage blob download \
    --container-name $container_name \
    --name $blob_name \
    --file helloAdminDownload.html
    #--account-name $AZURE_STORAGE_ACCOUNT \
```

3. Run `ls` to see file `helloAdminDownload.html` was downloaded

```sh
ls
```

**upload-batch**

1. Create an `upload` directory

```sh
# Create files in uploadfiles directory
mkdir uploadfiles
```

2. Create files to upload in the `upload` directory

```sh
# Create Files...
touch uploadfiles/myfile.html
touch uploadfiles/more.html
touch uploadfiles/hi.html
touch uploadfiles/myfile.txt
```

3. Create the `az_user_name` variable to be used in `myUploadPath` and `myDownloadPath`

> Note: Need to edit the below command to set `az_user_name`
>
> Capture the name `<name>@azure` in the cloud shell prompt
> e.g. - if the prompt in your cloud shell is `eric@Azure:~$` then set `az_user_name='eric'`

```sh
# REPLACE <name>
az_user_name=<name>
```

4. Create the upload Path and folder

```sh
# path to $home in cloud shell
myUploadPath=/home/$az_user_name/uploadfiles
```

5. Batch Upload the files to the container

```sh
# Upload html files in path
az storage blob upload-batch -d $container_name \
-s $myUploadPath -o table
```

6. List the files in the blob and check for the uploaded files

```sh
az storage blob list --container-name $container_name -o table
```

**download-batch**

1. Create a `download` directory

> Note: be sure `az_user_name` variable has been set as instructed above

2. Create the upload Path and folder

```sh
mkdir downloadfiles

myDownloadPath=/home/$az_user_name/downloadfiles
```

3. Bach download files from container

```sh
az storage blob download-batch -d $myDownloadPath -s $container_name
```

4. List the files in the download directory

```sh
# list downloadfiles
ls downloadfiles
```

**Get URL of blob and view public file**

1. Run the command
2. click the link generated to see the file

```sh
az storage blob url -c $container_name -n helloAdmin -o tsv
```

**Remove Public blob container**

1. When done reviewing clean up the Blob container

```sh
az storage container delete -n $container_name
```

### Task 2: Secure access to storage

**Create a private blob container in the west storage account**

1. Refresh variables if needed
1. **Edit** `AZURE_STORAGE_ACCOUNT` to reflect the account name generated

> Note: new value for `container_name` is `westblobcontainerprivate`

```sh
my_resource_group=WestRG
location=westus
container_name=westblobcontainerprivate

export AZURE_STORAGE_ACCOUNT=<*Enter*Storage*Account*> # $my_storage_account

export AZURE_STORAGE_KEY="$(az storage account keys list \
    --account-name $AZURE_STORAGE_ACCOUNT \
    --resource-group $my_resource_group \
    --query [0].value -o tsv)"

export AZURE_STORAGE_CONNECTION_STRING="$(az storage account \
    show-connection-string -n $AZURE_STORAGE_ACCOUNT\
     -g $my_resource_group --query connectionString -o tsv)"
```

**Create a blob container with Private Access**

1. Create a new blob container that is private

```sh
az storage container create --name $container_name --public-access blob
```

**Upload file to private blob container**

```sh
echo "<h1>Hello Azure Administrators - This is private</h1>">helloAdmin.html
blob_name=helloAdmin

az storage blob upload \
    --container-name $container_name \
    --file helloAdmin.html --name $blob_name
```

**Create Shared Access Signature (SAS) at the service level**

1. Set `end_date`

> *Note: access configured to last only 30 minutes*

```sh
# Create a 30 minute read-only SAS token on the container and store as CONTAINER_SAS_KEY
end_date=`date -u -d "30 minutes" '+%Y-%m-%dT%H:%MZ'`
```

2. Generate Container key variables

```sh
CONTAINER_SAS_KEY="$(az storage container generate-sas \
    --name $container_name --https-only --auth-mode key \
    --expiry $end_date --permissions r -o tsv)"
```

3. Generate Blob key variables

```sh
BLOB_SAS_KEY="$(az storage blob generate-sas \
    --name $blob_name \
    --container-name $container_name \
    --permissions r --expiry $end_date \
    --auth-mode key -o tsv)"
```

4. Generate full Blob URI and Test Link

```sh
# generate full blob URI and test link
private_URI="$(az storage blob generate-sas \
    --name $blob_name \
    --container-name $container_name \
    --permissions r --auth-mode key \
    --expiry $end_date --https-only \
    --full-uri)"

echo $private_URI
```

5. Test the URI link generated in the previous step to ensure it worked

### Task 3: Create a subnet service endpoint

**Display the status of the default rule for the storage account**

1. Type in the following CLI command to display status of the default rule for the storage account.

```sh
az storage account show --resource-group $my_resource_group \
    -n $AZURE_STORAGE_ACCOUNT --query networkRuleSet.defaultAction
```

**Set the default rule to deny network access by default (if needed)**

1. Set rule to `Deny` if current set to `Allow`

```sh
az storage account update --resource-group $my_resource_group \
    -n $AZURE_STORAGE_ACCOUNT --default-action Deny
```

**List storage account network rules**

1. Display, and then review, the list of list of network rules.

```sh
az storage account network-rule list --resource-group $my_resource_group \
    -n $AZURE_STORAGE_ACCOUNT
```

**Enable service endpoint for Azure Storage on existing vnet (WestVNet) and subnet (WestSubnet1)**

1. Update subnet rules to enable storage on WestVNet - WestSubnet1.

```sh
az network vnet subnet update --resource-group $my_resource_group \
    --vnet-name WestVNet --name WestSubnet1 \
    --service-endpoints "Microsoft.Storage"
```

**Add a network rule for a virtual network and subnet**

> **Note**: required to previously have set the default rule to deny, or network rules have no effect.

1. Create `subnet_id` variable

```sh
subnet_id="$(az network vnet subnet show \
    --resource-group $my_resource_group \
    --vnet-name WestVNet --name WestSubnet1 \
    --query id --output tsv)"
```

2. Add network subnet rule.

```sh
az storage account network-rule add --resource-group $my_resource_group \
    -n $AZURE_STORAGE_ACCOUNT --subnet $subnet_id
```

**List storage account network rule updates**

1. Display, and again review, the list of list of network rules.

```sh
az storage account network-rule list --resource-group $my_resource_group \
    -n $AZURE_STORAGE_ACCOUNT
```

**Test the page that was accessible in previous task - the network rule should deny access**

1. Click the link to test.
2. Result should be "access denied."

```sh
echo $private_URI
```

**Clean Up storage account (west)**

1. When done reviewing, remove storage account resources

```sh
az storage account delete -n $my_storage_account -g my_resource_group
```

### Task 4 – Create file storage

**Create a eastus storage account using Azure CLI**

1. Create random string to use for unique names (if not still in memory)

```sh
myRand=`head /dev/urandom | tr -dc a-z0-9 | head -c 6 ; echo ''`
echo "the random string append will be:  "$myRand
```

2. Create variables

```sh
my_resource_group=EastRG
location=eastus
my_storage_account=eaststore$myRand
```

3. Create eastus storage account

```sh
az storage account create --name $my_storage_account \
    --resource-group $my_resource_group --location $location \
    --sku Standard_LRS --kind StorageV2
```

4. allow non-https traffic (avoids "error 13"  mounting Linux drive)

```sh
az storage account update -n $my_storage_account --https-only false
```

5. List Storage Accounts
6. Note the newly created account

```sh
az storage account list -o table
```

**Create a file share and upload a file**

1. Set environment variables

```sh
export AZURE_STORAGE_ACCOUNT=$my_storage_account
export AZURE_STORAGE_KEY="$(az storage account keys list --account-name $AZURE_STORAGE_ACCOUNT --resource-group $my_resource_group --query [0].value -o tsv)"
export AZURE_STORAGE_CONNECTION_STRING="$(az storage account show-connection-string -n $AZURE_STORAGE_ACCOUNT -g $my_resource_group --query connectionString -o tsv)"
```

2. Create `eastfiles` file share

```sh
file_share_name=eastfiles

az storage share create --name $file_share_name --quota 2048
```

3. Create `myFileShareFile.html` file to upload

```sh
echo "File Shares Share Files">myFileShareFile.html
```

4. Upload file to share

```sh
az storage file upload --share-name $file_share_name --source ~/myFileShareFile.html
```

5. Output list of files in your Azure file storage
6. Note `myFileShareFile.html` is now in your file storage

```sh
az storage file list --share-name $file_share_name -o table
```

**Mount the file share from a Linux virtual machine**

1. SSH to Linux VM

```sh
east_vm_ip="$(az vm show -d -g $my_resource_group -n eastdebianvm --query publicIps -o tsv)"

ssh azuser@$east_vm_ip
```

> Note: Run commands in the VM SSH session to prep for storage attach

2. create a directory on the Linux VM for eastfiles

```sh
mkdir -p $my_storage_account/eastfiles
```

3. Install cifs-utils on Linux VM

```sh
sudo apt-get update

sudo apt-get install cifs-utils
```

4. Get code from the portal to attach the storage to the Linux machine

    1. In the portal:
        * open your eaststorage*123af4* (similar name) storage account.
        * navigate to the eastfiles file share.
    1. From the "eastfiles" blade click Connect.
    1. Change to the **Linux** tab and copy the connection string.
    1. Return to your EastDebianVM ssh session.
    1. Paste the connection string in the ssh session

> ```sh
> # connection string from the portal will look similar to this
> sudo mkdir /mnt/eaststorage123af4
> if [ ! -d "/etc/smbcredentials" ]; then
> sudo mkdir /etc/smbcredentials
> fi
> if [ ! -f "/etc/smbcredentials/eaststorage123af4.cred" ]; then
>     sudo bash -c 'echo "username=eaststorage123af4" >> /etc/smbcredentials/eaststorage123af4.cred'
>     sudo bash -c 'echo "password=EEBKMxGPWyTHe5CKEU58d1PCEdU2gOMHWb4nRZM07RlT5dpk6yiY0nFHCeN4HQOqNr8HLuKhQqa05m4qrAXJrg==" >> /etc/smbcredentials/eaststorage123af4.cred'
> fi
> sudo chmod 600 /etc/smbcredentials/eaststorage123af4.cred
> 
> sudo bash -c 'echo "//eaststorage123af4.file.core.windows.net/eastfiles /mnt/eaststorage123af4 cifs nofail,vers=3.0,credentials=/etc/smbcredentials/eaststorage123af4.cred,dir_mode=0777,file_mode=0777,serverino" >> /etc/fstab'
> 
> sudo mount -t cifs //eaststorage123af4.file.core.windows.net/eastfiles /mnt/eaststorage123af4 -o vers=3.0,credentials=/etc/smbcredentials/eaststorage123af4.cred,dir_mode=0777,file_mode=0777,serverino
> ```

**Verify files are shared to VM from Azure and the other way around**

1. Stop allowing non-https traffic now that share is mounted (CLI)
    * Open a **new Azure Portal web page** instance and launch the Cloud Shell
    * Type the command in to the **Azure CLI** (not the SSH session)

```sh
az storage account update -n $my_storage_account --https-only true
```

> Note: Return to the **Linux SSH Session** for the following commands

2. In the Linux SSH Session Navigate to the mount directory

```sh
# from eastDebianVM SSH
cd /mnt/$my_storage_account
```

3. View file being shared to VM from Azure file storage

```sh
# see shared file
ls
```

4. Add a new file from the VM

```sh
# create new file from VM
echo "I am from eastDebianVM">newFile.txt
```

5. `exit` SSH session

```sh
# from VM ssh session
exit
```

6. Confirm the file (newFile.txt) is present in the Azure Portal to confirm the sharing works both directions.

**Clean Up file storage resources**

1. remove the storage account

```sh
az storage account delete -n $my_storage_account -g my_resource_group
```
