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

* **Estimated time**: 45 Minutes

## Instructions

### Before you start

#### Setup Task

1. **Dependency on previous labs:**
    1. Module 1: Azure Administration - **Lab: Creating Resource Groups**. EastRG and WestRG resource groups configured.
    1. Module 2: Azure Networking - **Lab: Virtual Networks and Peering**. VNets with SubNets and peering configured.
    1. Module 3: Azure compute - **Lab: Azure VM**. EastDebianVM created.

### Exercise 1: Blobs, Secure Access Storage, Service Endpoint and File Storage

The main tasks for this exercise are as follows:

1. Create and configure Blob Storage
1. Configure Blob with Secure Access Storage (SAS)
1. Configure a SubNet Service Endpoint
1. Create and Configure File Storage

#### Task 1: Create a Storage Account with Blob Storage

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
my_storage_kind=StorageV2myRand
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

1. Create Storage account and storage account connection string environment variables

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

#### Download public file

1. `helloAdmin.html` is downloaded with new name `helloAdminDownload.html`

```sh
az storage blob download \
    --container-name $container_name \
    --name $blob_name \
    --file helloAdminDownload.html
    #--account-name $AZURE_STORAGE_ACCOUNT \
```

#### Run `ls` to see file `helloAdminDownload.html` was downloaded

```bash
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

1. Create aan `download` directory

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

1. Run the command and then click the link generated.

```sh
az storage blob url -c $container_name -n helloAdmin -o tsv
```

**Remove Public blob container**

1. When done reviewing clean up the Blob container

```bash
az storage container delete -n $container_name
```

#### Task 2: Secure access to storage

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

```bash
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

2. Generate Container and Blob key variables

```sh
CONTAINER_SAS_KEY="$(az storage container generate-sas \
    --name $container_name --https-only --auth-mode key \
    --expiry $end_date --permissions r -o tsv)"

BLOB_SAS_KEY="$(az storage blob generate-sas \
    --name $blob_name \
    --container-name $container_name \
    --permissions r --expiry $end_date \
    --auth-mode key -o tsv)"
```

3. Generate full Blob URI and Test Link

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




