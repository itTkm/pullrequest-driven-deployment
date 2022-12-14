# pullrequest-driven-deployment

This repository is sample of `pull-request driven deployment`.

## What is `pull-request driven deployment` ?

`Pull-request driven deployment` automates the work required to validate a pull request.  
It make a happy world where engineers can focus on development.

When a pull request is created or updated, the artifacts are automatically deployed to the unique URL linked to the pull request by a [CI/CD workflow](.github/workflows/cicd.yml).

This sample project is written in [Nuxt.js](https://nuxtjs.org/) and configured to automatically deploy to [Static website hosting in Azure Storage](https://learn.microsoft.com/en-us/azure/storage/blobs/storage-blob-static-website).

You can implement `pull-request driven deployment` to your any language project and any deployment target at your fingertips.

## How can I try it?

You can try this sample project with Azure Blob Storage by following these steps:

### 1. Setup static website hosting on the Azure Blob Storage.

```bash
# ----- SETTINGS -----
SERVICE_PRINCIPAL_NAME=your-service-principal-name
REGION_NAME=japaneast
RESOURCE_GROUP_NAME=your-resource-group-name
STORAGE_ACCOUNT_NAME=yourstorageaccountname
# ----- SETTINGS -----

# Create resource group
az group create \
  --name $RESOURCE_GROUP_NAME \
  --location $REGION_NAME

# Create storage account
az storage account create \
  --name $STORAGE_ACCOUNT_NAME \
  --resource-group $RESOURCE_GROUP_NAME \
  --sku Standard_LRS

# Set a static website configuration
az storage blob service-properties update \
  --account-name $STORAGE_ACCOUNT_NAME \
  --static-website \
  --404-document index.html \
  --index-document index.html

# Get base url of static website hosting
az storage account show \
  --name $STORAGE_ACCOUNT_NAME \
  --resource-group $RESOURCE_GROUP_NAME \
  --query "primaryEndpoints.web" \
  --output tsv

# Please add the base url of static web site hosting
# to your repository's Secret "BASE_URL".

# Create service principal
az ad sp create-for-rbac \
  --name $SERVICE_PRINCIPAL_NAME \
  --years $((9999 - $(date "+%Y"))) \
  --sdk-auth | tee AZURE_CREDENTIALS_$SERVICE_PRINCIPAL_NAME.json

# Please add the contents of AZURE_CREDENTIALS_${SERVICE_ACCOUNT_NAME}.json
# to your repository's Secret "AZURE_CREDENTIALS".

# Grant permissions to service principal
groupId=$(az group show --name $RESOURCE_GROUP_NAME --query id --output tsv)
storageId=$(az storage account show --name $STORAGE_ACCOUNT_NAME --query id --output tsv)
clientId=$(az ad sp list \
  --display-name $SERVICE_PRINCIPAL_NAME \
  --query "[].{id:id}" \
  --output tsv)
az role assignment create --assignee $clientId --scope $storageId \
  --role "Storage Blob Data Contributor"
```

### 2. Add secrets to your GitHub project

When try it out on your own repository, please set below secrets to your repository.

| Secret key        | Description                                                  |
| ----------------- | ------------------------------------------------------------ |
| BASE_URL          | Set your base url of static website hosting.                 |
| AZURE_SA_NAME     | Set your storage account name for deployment.                |
| AZURE_CREDENTIALS | Contents of `AZURE_CREDENTIALS_${SERVICE_ACCOUNT_NAME}.json` |

### 3. Try `pull-request driven deployment`!

Create a new branch from your `main` branch, change your code, and create a new pull request to your `main` branch.  
Then the [CI/CD workflow](.github/workflows/cicd.yml) will triggered and the bot says as below a few minutes ago.

```
Deployed the latest code to Azure Static WebSite hosting in Azure Blob storage.
http://{BASE_URL}/{your-repository-name}/{your-pullrequest-number}/
```

## Build Setup

```bash
# install dependencies
$ yarn install

# serve with hot reload at localhost:3000
$ yarn dev

# build for production and launch server
$ yarn build
$ yarn start

# generate static project
$ yarn generate
```

For detailed explanation on how things work, check out the [documentation](https://nuxtjs.org).

## License

This project is licensed under the [MIT License](./LICENSE).
