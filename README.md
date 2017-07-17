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

## Step 1a - Enable Billing

```
gcloud service-management enable deploymentmanager.googleapis.com
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
```
cd ..
https://github.com/GoogleCloudPlatform/spinnaker-deploymentmanager.git
cd spinnaker-deploymentmanager

```

## Step 3b - create spinnaker
```
gcloud service-management enable deploymentmanager.googleapis.com
gcloud service-management enable appengine.googleapis.com
gcloud service-management enable compute-component.googleapis.com

export GOOGLE_PROJECT=$(gcloud config get-value project)
export DEPLOYMENT_NAME="spinnaker-deploy"
export JENKINS_PASSWORD=C1j0seC2!
gcloud deployment-manager deployments create --config config.jinja ${DEPLOYMENT_NAME} --properties jenkinsPassword:${JENKINS_PASSWORD}
```

## Step 3b -
```
export DEPLOYMENT_NAME="spinnaker-deploy"
export SPINNAKER_VM=$(gcloud compute instances list --regexp "${DEPLOYMENT_NAME}-spinnaker.+" --uri)
export JENKINS_VM=$(gcloud compute instances list --regexp "${DEPLOYMENT_NAME}-jenkins.+" --uri)
gcloud compute ssh ${SPINNAKER_VM} -- -L 8081:localhost:8081 -L 8082:$(basename $JENKINS_VM):8080
```

## Step 3c - Verify localhost:8081 for Spinnaker and localhost:8082 for Jenkins


