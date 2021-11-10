# Credit risk data science environment

This has details on the environment used to run [CRDS notebooks](https://github.com/semiformal-net/crds-notebooks)

## Docker

The `docker` directory contains a Dockerfile that is meant to run in GCP as a Vertex notebook. The directory contains a cloud run script to build and deploy it in GCP.

### Prerequisites

- Visit \url{https://console.cloud.google.com/} and signup for google cloud
- Create a new project called "credit-risk-data-science"
- Enable Container Registry API
- In google cloud shell run:

```
PROJECT_ID=credit-risk-data-science
PROJECT_NUMBER=$(gcloud projects describe $PROJECT_ID --format='value(projectNumber)')
# open https://console.cloud.google.com/cloud-build/settings and click "enable API"
# grant cloud run admin to cloud build service account
gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member=serviceAccount:$PROJECT_NUMBER@cloudbuild.gserviceaccount.com \
    --role=roles/run.admin
gcloud iam service-accounts add-iam-policy-binding \
    $PROJECT_NUMBER-compute@developer.gserviceaccount.com \
    --member=serviceAccount:$PROJECT_NUMBER@cloudbuild.gserviceaccount.com \
    --role=roles/iam.serviceAccountUser
gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member=serviceAccount:$PROJECT_NUMBER@cloudbuild.gserviceaccount.com \
    --role=roles/notebooks.admin
```

Now clone this repo,
- `git clone https://github.com/semiformal-net/crds-ml-environment.git`
- Enter the docker directory: `cd crds-ml-environment/docker`
- [Install the gcloud SDK on your machine](https://cloud.google.com/sdk/docs/install)
- Now build and deploy the CRDS container in GCP: `gcloud builds submit`

## Local

Local environment can be set up with conda.

- Install MiniConda from conda.io
- Install git
- Run the following code:
```
conda env create -f crds_conda.yml
git clone https://github.com/semiformal-net/crds-notebooks
cd crds-notebooks
conda activate crds
```
# Prepare data in your environment

Once you are in your new Jupyter environment click on *Terminal* and run

```
cd ~jupyter
git clone https://github.com/semiformal-net/crds-notebooks.git
```

Start by running the `prepare_auto_data.ipynb` notebook to prepare the data we will use in the book. Other notebooks will be referenced in the text when it is time to open them.

# Grab notebooks automatically

I have issued a github deploy key that has access to pull the `crds-notebooks` repo. If you have access to that key you can use the authenticated deploy once you have the key in secret manager.

Enable the [secrets api](https://console.cloud.google.com/security/secret-manager?referrer=search&project=credit-risk-data-science)

```
    name: crds-notebooks-deploy-key
    upload file: /tmp/id_github
    click "create secret"
```

Build with `cloudbuild_deploykey.yaml`
