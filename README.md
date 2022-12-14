# pullrequest-driven-deployment

This repository is sample of `pull-request driven deployment`.

## What is `pull-request driven deployment` ?

`Pull-request driven deployment` automates the work required to validate a pull request.  
It make a happy world where engineers can focus on development.

When a pull request is created or updated, the artifacts are automatically deployed to the unique URL linked to the pull request by a [CI/CD workflow](.github/workflows/cicd.yml).

This sample project is written in [Nuxt.js](https://nuxtjs.org/) and configured to automatically deploy to [Google Cloud Storage](https://cloud.google.com/storage).

You can implement `pull-request driven deployment` to your any language project and any deployment target at your fingertips.

## How can I try it?

You can try this sample project with the [Google Cloud Storage free tier](https://cloud.google.com/free/docs/free-cloud-features#storage) by following these steps:

### 1. Setup static-site hosting on the Google Cloud Storage.

```bash
# ----- SETTINGS -----
PROJECT_ID=your-project-id
SERVICE_ACCOUNT_NAME=your-service-account-name
BUCKET_NAME=your-bucket-name
BUCKET_LOCATION=us-west1
# ----- SETTINGS -----

# Create service account
gcloud iam service-accounts create $SERVICE_ACCOUNT_NAME \
    --description=$SERVICE_ACCOUNT_NAME \
    --display-name=$SERVICE_ACCOUNT_NAME

# Create service account key
SERVICE_ACCOUNT_ID=$SERVICE_ACCOUNT_NAME@$PROJECT_ID.iam.gserviceaccount.com
SERVICE_ACCOUNT_KEY_FILE=${SERVICE_ACCOUNT_NAME}_key.json
gcloud iam service-accounts keys create \
  $SERVICE_ACCOUNT_KEY_FILE \
  --iam-account=$SERVICE_ACCOUNT_ID

# Please add the contents of ${SERVICE_ACCOUNT_NAME}_key.json
# to your repository's Secret "GCP_CREDENTIALS".

# Create new bucket
gcloud storage buckets create gs://$BUCKET_NAME --location=$BUCKET_LOCATION

# Set a website configuration
gsutil web set \
  -m index.html \
  -e index.html \
  gs://$BUCKET_NAME

# Grant permissions to service account
gsutil iam ch \
  "serviceAccount:$SERVICE_ACCOUNT_ID:roles/storage.objectAdmin" \
  gs://$BUCKET_NAME

# Allow public access
gsutil iam ch \
  allUsers:objectViewer \
  gs://$BUCKET_NAME
```

### 2. Add secrets to your GitHub project

When try it out on your own repository, please set below secrets to your repository.

| Secret key      | Description                                    |
| --------------- | ---------------------------------------------- |
| GCP_CREDENTIALS | Contents of `${SERVICE_ACCOUNT_NAME}_key.json` |
| GCP_BUCKET_NAME | Set your bucket name for deployment.           |

### 3. Try `pull-request driven deployment`!

Create a new branch from your `main` branch, change your code, and create a new pull request to your `main` branch.  
Then the [CI/CD workflow](.github/workflows/cicd.yml) will triggered and the bot says as below a few minutes ago.

```
Deployed the latest code to an Google Cloud Storage.
http://storage.googleapis.com/{your-backet-name}/{your-repository-name}/{your-pullrequest-number}/
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
