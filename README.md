# pso-appdev-gae-std-python
Google PSO - AppDev - GAE - Standard - Python

git clone git@github.com:joseret/pso-appdev-gae-std-python.git

# Step 0 - Clone
```
export PSO_GIT_USER_NAME='Jose Retelny'
export PSO_GIT_EMAIL='joseret@google.com'
git config --global user.email ${PSO_GIT_EMAIL}
git config --global user.name ${PSO_GIT_USER_NAME}
git remote add upstream https://github.com/joseret/pso-appdev-gae-std-python
```

# Step 1 - Create Project and Setup Spinnaker
## Reference

https://cloud.google.com/sdk/gcloud/reference/projects/create

## Instructions (jr@joecloudy.com)

```
gcloud auth login

export GOOGLE_PROJECT='pso-appdev-gae-cs1'
export GOOGLE_ORG='joecloudy.com'
export GOOGLE_ORG_ID='553683458383'

gcloud projects create ${GOOGLE_PROJECT} --organization=${GOOGLE_ORG_ID} --set-as-default

gcloud config set project ${GOOGLE_PROJECT}

```


# Step 2 - Setup Your IDE

Make sure you fix gitignore
```
mkdir lib
touch __init__.py

```

# Step 3 - Setup Spinnaker

## Reference
https://cloud.google.com/solutions/spinnaker-on-compute-engine

## Step 3a - Clone Spinnaker Deployment (somewhere else)

cd ..
https://github.com/GoogleCloudPlatform/spinnaker-deploymentmanager.git
cd spinakker-deploymentmanager

