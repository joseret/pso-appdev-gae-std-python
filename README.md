# pso-appdev-gae-std-python
Google PSO - AppDev - GAE - Standard - Python

#  Starting requirement - This Document - README.md

# Create Git Public Repository Public

* Sign in to Github
* New Repository
 + Name / Description
 


![Create git public repository](https://storage.googleapis.com/joe-cloudy-public/pso-appdev-cloudstart-assets/pso-app-dev-github-create-repository.png) 
 
 * Remember your github url 
 
[https://github.com/joseret/pso-appdev-gae-c1](https://github.com/joseret/pso-appdev-gae-c1)

# Login to your "development" environment

* vnc to development GCP Instance
* create your source directory folder

#  
```
git@github.com:joseret/pso-appdev-gae-c1.git
```

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

export GOOGLE_PROJECT='pso-appdev-gae-cs2'
export GOOGLE_ORG='joecloudy.com'
export GOOGLE_ORG_ID='553683458383'

gcloud projects create ${GOOGLE_PROJECT} --organization=${GOOGLE_ORG_ID} --set-as-default

gcloud config set project ${GOOGLE_PROJECT}

```

## Step 1a - Enable Billing

```
https://www.spinnaker.io/guides/tutorials/codelabs/appengine-source-to-prod/
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

## Step 3d - Open Firewall to
gcloud compute firewall-rules create allow-github-webhook \
    --allow="tcp:8084" \
    --source-ranges=$(curl -s https://api.github.com/meta | python -c "import sys, json; print ','.join(json.load(sys.stdin)['hooks'])") \
    --target-tags="spinnaker-vm"

## Step 3e - Setup Halyard

```
curl -O https://raw.githubusercontent.com/spinnaker/halyard/master/install/stable/InstallHalyard.sh

sudo bash InstallHalyard.sh
```

## Step 3f - Snapshot via UI (commnad below)
```
gcloud compute --project "pso-appdev-gae-cs1" disks snapshot "spinnaker-deploy-spinnaker-dqdp" --zone "us-west1-a" --description "spinnaker-deploy-spinnaker-dqdp-s2" --snapshot-names "spinnaker-deploy-spinnaker-dqdp-s2"

```

# New
```
gcloud compute firewall-rules create allow-github-webhook \
    --allow="tcp:8084" \
    --source-ranges=$(curl -s https://api.github.com/meta | python -c "import sys, json; print ','.join(json.load(sys.stdin)['hooks'])") \
    --target-tags="allow-github-webhook" --project ${GOOGLE_PROJECT}
```
```
gcloud compute instances create $USER-spinnaker \
    --scopes="https://www.googleapis.com/auth/cloud-platform" \
    --machine-type="n1-highmem-4" \
    --image-family="ubuntu-1404-lts" \
    --image-project="ubuntu-os-cloud" \
    --zone="us-west1-a" \
    --tags="allow-github-webhook"  --project ${GOOGLE_PROJECT}
```
```
gcloud compute ssh joseret-spinnaker --ssh-flag="-L 9000:localhost:9000" --zone use-west1-a --project ${GOOGLE_PROJECT} --ssh-flag="-L 8084:localhost:8084"
```
```


curl -O https://raw.githubusercontent.com/spinnaker/halyard/master/install/stable/InstallHalyard.sh

sudo bash InstallHalyard.sh

```

```
export GOOGLE_PROJECT='pso-appdev-gae-cs2'

gcloud app create --region us-central --project ${GOOGLE_PROJECT}
```

```

export GOOGLE_PROJECT='pso-appdev-gae-cs2'

hal config version edit --version $(hal version latest -q)

hal config provider appengine enable


hal config provider appengine account add my-appengine-account --project ${GOOGLE_PROJECT}

hal config storage gcs edit --project ${GOOGLE_PROJECT}

hal config storage edit --type gcs

mkdir  ~/.hal/default/service-settings/gate.yml

echo "host: 0.0.0.0" | tee ~/.hal/default/service-settings/gate.yml


```

```
sudo hal deploy apply



```
### Create Application

```
pso-appdev-gae-cs2
github
https://github.com/joseret/pso-appdev-gae-std-python





```
