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

I have issued a github deploy key that has access to pull the private `crds-notebooks` repo (per [these instructions](https://cloud.google.com/build/docs/access-github-from-build)). If you have access to that key you can use the authenticated deploy once you have the key in secret manager.

First enable the [secrets api](https://console.cloud.google.com/flows/enableapi?apiid=secretmanager.googleapis.com,cloudbuild.googleapis.com&_ga=2.62677420.1826820591.1636573492-1369367446.1630458144)

Then enter the key into [secret manager](https://console.cloud.google.com/security/secret-manager?referrer=search&project=credit-risk-data-science)

```
    name: crds-notebooks-deploy-key
    upload file: /tmp/id_github
    click "create secret"
```

Ensure that cloudbuild has persmissions to access secrets,

```
PROJECT_ID=credit-risk-data-science
PROJECT_NUMBER=$(gcloud projects describe $PROJECT_ID --format='value(projectNumber)')
gcloud config set project $PROJECT_ID
gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member=serviceAccount:$PROJECT_NUMBER@cloudbuild.gserviceaccount.com \
    --role=roles/secretmanager.secretAccessor
```

And build with `gcloud builds submit --config docker/cloudbuild_deploykey.yaml`

# Cost reduction measures

## Persistent disks

By default, vertex notebooks have a boot disk and a data disk. Both seem mandatory, but you can only configure boot disk on notebook creation. The data disk is set to 100GB and is mounted to `/home/jupyter`. To reduce charges, we will reset the data disk to 10GB

```
gcloud compute instances stop --zone us-central1-b crds

gcloud compute instances detach-disk --zone us-central1-b crds --device-name data
gcloud compute disks create crds-data-small --size 10GB --type pd-standard --zone us-central1-b # 10 is min
gcloud compute instances attach-disk --zone us-central1-b crds --disk crds-data-small --device-name data
gcloud compute disks delete --quiet --zone us-central1-b crds-data
gcloud compute instances start --zone us-central1-b crds

# per: https://cloud.google.com/compute/docs/disks/add-persistent-disk#formatting
gcloud beta compute ssh --zone "us-central1-b" "crds"  --project "credit-risk-data-science"
    $ sudo lsblk
    $ sudo mkfs.ext4 -m 0 -E lazy_itable_init=0,lazy_journal_init=0,discard /dev/sdb
```

Note that the `crds-data-small` disk will persist until manually deleted (even if you delete the `crds` notebook instance.

## External IP

If you are happy proxying into the notebook via google cloud console then you don't need a public IP. This greatly increases the security of the notebook as well.

```
--no-public-ip
```
