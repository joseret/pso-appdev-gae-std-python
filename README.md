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

#  Get the starting empty github repository and instructions
```
git clone git@github.com:joseret/pso-appdev-gae-c1.git
cd pso-appdev-gae-c1
```

# Clone
```
export PSO_GIT_USER_NAME='Jose Retelny'
export PSO_GIT_EMAIL='joseret@google.com'
git config --global user.email ${PSO_GIT_EMAIL}
git config --global user.name ${PSO_GIT_USER_NAME}
git remote add upstream https://github.com/joseret/pso-appdev-gae-c1
cd pso-appdev-gae-c1
```

# Section 010 - Create Local App
```commandline
git checkout -b s-010-local-app

```
# Create main.py

```
# Copyright 2016 Google Inc.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

import webapp2


class MainPage(webapp2.RequestHandler):
    def get(self):
        self.response.headers['Content-Type'] = 'text/html'
        self.response.write('<body style=\'background-color: red\'>Hello, World!</body>')


app = webapp2.WSGIApplication([
    ('/', MainPage),
], debug=True)


```

# Create app.yaml

```
runtime: python27
api_version: 1
threadsafe: true

handlers:
- url: /.*
  script: main.app
```

# Commit changes
```commandline
git status
git add .
git commit -a -m "Section 010 - local app"
git push origin s-010-local-app 
```

## Start Local Dev Server
```commandline
 dev_appserver.py app.yaml
```

## Check App
```commandline
http://localhost:8080

```

## Hope you see this
![Demo App - Yay!](https://storage.googleapis.com/joe-cloudy-public/pso-appdev-cloudstart-assets/S010%20-%20initial-app-running-locally.png)


## Additional Goodies - the Admin Console (for local app)
```commandline
http://localhost:8000
```

### See the Admin Console
![Demo App - Admin Console - Nice!](https://storage.googleapis.com/joe-cloudy-public/pso-appdev-cloudstart-assets/s010-local-app-admin-console.png)


### Quick Change - to main.py

* Change the text in main.py
  + Notice that python autodetects changes (most of the time does not require restart of dev_server.py)
![Demo App - Small Change](https://storage.googleapis.com/joe-cloudy-public/pso-appdev-cloudstart-assets/S010-local-app-small-change.png)



# Setup Project if applicable


## Create Project

[Create Project Docs](https://cloud.google.com/sdk/gcloud/reference/projects/create)


### Instructions (jr@joecloudy.com)

```
gcloud auth login

export GOOGLE_PROJECT='pso-appdev-gae-cs2'
export GOOGLE_ORG='joecloudy.com'
export GOOGLE_ORG_ID='553683458383'

gcloud projects create ${GOOGLE_PROJECT} --organization=${GOOGLE_ORG_ID} --set-as-default

gcloud config set project ${GOOGLE_PROJECT}
gcloud config set compute/region us-central1
gcloud config set compute/zone us-central1-f

```

## Enable Billing (if needed)
[Enable Billing Instructions](https://support.google.com/cloud/answer/6293499#enable-billing)




# Deploy App (S020)

## Enable API's
```commandline
gcloud service-management enable appengine.googleapis.com
gcloud service-management enable compute-component.googleapis.com

```

## Deploy App

```commandline
gcloud app deploy app.yaml
```

## Let's see in GCP

[GCP Console](https://console.cloud.google.com/home/dashboard?project=pso-appdev-gae-cs2)



https://pso-appdev-gae-cs2.appspot.com


## Install siege

```commandline
sudo apt-get install siege
```

## Let's see it auto

```commandline
siege -c 1 -l /tmp/siege.log https://pso-appdev-gae-cs2.appspot.com 

```
### Siege 10 then 100
![Concurrent 10 then Concurrent 3](https://storage.googleapis.com/joe-cloudy-public/pso-appdev-cloudstart-assets/s020-gcp-app-under-siege-10-100.png)

### Back to -c 3
![Back to Concurrent 3](https://storage.googleapis.com/joe-cloudy-public/pso-appdev-cloudstart-assets/s020-gcp-app-siege-concurrent-3.png)



# s030 - Install Spinnaker - Without Trigger

[Spinnaker App Engine Code Lab Instructions](https://www.spinnaker.io/guides/tutorials/codelabs/appengine-source-to-prod/)



## Skip creating the app, we already did it

## Skip Github firewall

## Create Spinnaker GCE Instance

```commandline
echo PSO_CS_PROJECT
```
```commandline
gcloud compute instances create cicd-spinnaker \
    --scopes="https://www.googleapis.com/auth/cloud-platform" \
    --machine-type="n1-highmem-4" \
    --image-family="ubuntu-1404-lts" \
    --image-project="ubuntu-os-cloud" \
    --zone="us-central1-f" \
    --tags="allow-github-webhook" --project PSO_CS_PROJECT
```

## Setup firewall rule for github
```commandline
gcloud compute firewall-rules create allow-github-webhook \
    --allow="tcp:8084" \ 
    --source-ranges=$(curl -s https://api.github.com/meta | python -c "import sys, json; print ','.join(json.load(sys.stdin)['hooks'])") \
    --target-tags="allow-github-webhook"
```


## Optional to use ssh proxy for localhost (we have a dev environment VM to in same project)

```commandline
export PSO_CS_USER=jr
export PSO_CS_PROJECT='pso-appdev-gae-cs2'
gcloud config set project ${PSO_CS_PROJECT}
gcloud config set compute/region us-central1
gcloud config set compute/zone us-central1-f

gcloud config list

gcloud compute ssh ${PSO_CS_USER}@cicd-spinnaker --ssh-flag="-L 9000:localhost:9000" --ssh-flag="-L 8084:localhost:8084"
```

### Terminal inside spinnaker vm

![Terminal inside spinnaker vm](https://storage.googleapis.com/joe-cloudy-public/pso-appdev-cloudstart-assets/s030-login-install-spinnaker.png)
## Install Spinnaker 

```commandline
curl -O https://raw.githubusercontent.com/spinnaker/halyard/master/install/stable/InstallHalyard.sh

sudo bash InstallHalyard.sh


```

### Configure Spinnaker for App Egine

```commandline
export PSO_CS_USER=jr
export PSO_CS_PROJECT='pso-appdev-gae-cs2'
hal config version edit --version $(hal version latest -q)

hal config provider appengine enable

hal config provider appengine account add my-appengine-account --project $PSO_CS_PROJECT

hal config storage gcs edit --project $PSO_CS_PROJECT

hal config storage edit --type gcs

mkdir -p ~/.hal/default/service-settings
echo "host: 0.0.0.0" | tee ~/.hal/default/service-settings/gate.yml

```

### Finalize Spinnaker Installation

```commandline
sudo hal deploy apply
```


### Goto Spinnaker App (use machine ip, or instance-name or localhost for ssh proxy)
```commandline
http://localhost:9000 
```

## Configure App Engine Application in Spinnaker

* [Follow insturctions](https://www.spinnaker.io/guides/tutorials/codelabs/appengine-source-to-prod/#deploy-to-app-engine)


### Create Application (Under Actions)

* Name (psocs)
* Email (devops email)
* Repo Type (github)
* Repo Project (https://github.com/joseret/pso-appdev-gae-c1)
* Repo Project (pso-appdev-gae-c1)
* Description 

Press Create!

### Create Server Group 

* Stack (default)
* Git Repoistory URL (https://github.com/joseret/pso-appdev-gae-c1.git)
* Git Credential Type (No Credentials)
* Branch (release)

* Config Files
  + Config Filepaths (app.yaml)
  
  
  
## Create Deployment Pipeline 
* [Follow Instructions](https://www.spinnaker.io/guides/tutorials/codelabs/appengine-source-to-prod/#deployment-pipeline)

### Create Pipeline

* Pipeline Name (Deploy And Promote)

### (Skip Webhook) - Deploy Stage
* [Follow Instructions](https://www.spinnaker.io/guides/tutorials/codelabs/appengine-source-to-prod/#deploy-stage)

  + Add Server Group

### Add Stage (Edit Load Balancer)



#  Add Stage (Manual Judgement)

https://www.spinnaker.io/guides/tutorials/codelabs/appengine-source-to-prod/#manual-judgment-stage

### Add Stage - Enable Server Group

https://www.spinnaker.io/guides/tutorials/codelabs/appengine-source-to-prod/#enable-stage

### Add Stage (Wait)

https://www.spinnaker.io/guides/tutorials/codelabs/appengine-source-to-prod/#wait-stage


### Add Stage (Destroy)

https://www.spinnaker.io/guides/tutorials/codelabs/appengine-source-to-prod/#destroy-stage



### Run Manually

* Remove the manually deployed one

# S040 - Webhook

```commandline
gcloud compute firewall-rules create allow-github-webhook \
    --allow="tcp:8084" \
    --source-ranges=$(curl -s https://api.github.com/meta | python -c "import sys, json; print ','.join(json.load(sys.stdin)['hooks'])") \
    --target-tags="spinnaker-vm"
```

## Goto Network to check


## Configure Automatic Trigger

* [Follow Instreuctions](https://www.spinnaker.io/guides/tutorials/codelabs/appengine-source-to-prod/#webhook-trigger)

![Webhook Trigger](https://storage.googleapis.com/joe-cloudy-public/pso-appdev-cloudstart-assets/s040-config-webhook-automated-trigger.png)


### Find cicd-simmaker instance external ip


### Trigger Release



```
git checkout release
git merge s040-webhook
git push
```

### If it works

![You should se this](https://storage.googleapis.com/joe-cloudy-public/pso-appdev-cloudstart-assets/s040-config-webhook-github.png)



# S050 - Setup Firebase (starter spa)

### Go to Firebase console, make sure you are logged in class environment

[Firebase console](https://console.firebase.google.com/?pli=1)


### Import Google Project

* Select Pay as You Go

### Setup Basic SPA based on [Firebase Friendly Chat](https://github.com/firebase/friendlychat-web/tree/master/web-start)

Look at s050 - branch

### Pop Quiz, look at images
1. Why does http://localhost:8080/images/profile_placeholder.png work?
2. Let's change root to go to /static/index.html
3. Let's the python gae backend to respond to /app

### Trigger release

### Validate (static)
* Check Canary
* Check Console
* Check Network

What Next? 


# S060 - Setup Firebase Auth

###  Add Firebase to you web app

```javascript
<script src="https://www.gstatic.com/firebasejs/4.1.3/firebase.js"></script>
<script>
  // Initialize Firebase
  var config = {
    apiKey: "AIzaSyDQvUHhUTfQnioCjKf7lnr3kM_g2K8dz-8",
    authDomain: "pso-appdev-cs-1.firebaseapp.com",
    databaseURL: "https://pso-appdev-cs-1.firebaseio.com",
    projectId: "pso-appdev-cs-1",
    storageBucket: "pso-appdev-cs-1.appspot.com",
    messagingSenderId: "159748236642"
  };
  firebase.initializeApp(config);
</script>

## Setup Signin Method (https://firebase.corp.google.com/project/pso-appdev-cs-1/authentication/users)

![Setup Signin Methods](https://storage.googleapis.com/joe-cloudy-public/pso-appdev-cloudstart-assets/s060-fb-auth-setup-signin.png)

## Goto Get Started

### Set Email/Password

Let's use 
[Firebase UI Projects For Web](https://github.com/firebase/FirebaseUI-Web)

### Insert the Code (localized widtet)

[Languages]()https://github.com/firebase/firebaseui-web/blob/master/LANGUAGES.md)

```
<script src="https://www.gstatic.com/firebasejs/ui/2.3.0/firebase-ui-auth__{LANGUAGE_CODE}.js"></script>
<link type="text/css" rel="stylesheet" href="https://www.gstatic.com/firebasejs/ui/2.3.0/firebase-ui-auth.css" />

```

## Create a widget.html (that will be the popup)

* Same header includes for firebase from before (with this body)
```
  <body>
    <div id="firebaseui-auth-container"></div>
  </body>
```

* Create widget.html (as login screen)
  +  (Hint-use the index.html styling as much as possible)

![Something Like This](https://storage.googleapis.com/joe-cloudy-public/pso-appdev-cloudstart-assets/s060-get-ready-for-auth.png)  

* Popup widget.html

  + Enable Signin FriendlyChat "constructor"
```
  this.signInButton = document.getElementById('sign-in');
  this.signInButton.addEventListener('click', this.signInWithPopup.bind(this));
  
```

  + Popup Code
```

/**
 * @return {string} The URL of the FirebaseUI standalone widget.
 */
FriendlyChat.prototype.getWidgetUrl = function()  {
  return '/static/widget.html';
};


FriendlyChat.prototype.signInWithPopup = function() {
  window.open(this.getWidgetUrl(), 'Sign In', 'width=985,height=735');
};

```
  + Show Signin Button

```

#TBD


https://cloud.google.com/sdk/gcloud/reference/projects/create


# Step 2 - Setup Your IDE

Make sure you fix .gitignore
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
