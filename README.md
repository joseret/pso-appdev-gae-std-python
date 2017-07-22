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
## Install python packages
```
pip install -r requirements.txt -t lib
```
## Remove lib from .gitignore


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
```


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
  + Show Signin Button (add the line inside initFirebase)

```
FriendlyChat.prototype.initFirebase = function() {
  // TODO(DEVELOPER): Initialize Firebase.
  this.signInButton.removeAttribute('hidden');
};

```

# s070 - Add firebase auth

## Initialize Firebase Auth

```
FriendlyChat.prototype.initFirebase = function() {
  // Shortcuts to Firebase SDK features.
  this.auth = firebase.auth();
  // this.database = firebase.database();
  // this.storage = firebase.storage();
  // Initiates Firebase auth and listen to auth state changes.
  this.auth.onAuthStateChanged(this.onAuthStateChanged.bind(this));
};
```

  + Add Checking for Auth State
```

FriendlyChat.prototype.onAuthStateChanged = function(user) {

  if (user) { // User is signed in!
    console.log('user', user);
    // Get profile pic and user's name from the Firebase user object.
    var profilePicUrl = user.photoURL; // Only change these two lines!
    var userName = user.displayName;
    var self = this;
    user.getToken().then(function(idToken) {
      console.log('getToken', idToken, user);
      self.userIdToken = idToken;
    });
    // Set the user's profile pic and name.
    this.userPic.style.backgroundImage = 'url(' + profilePicUrl + ')';
    this.userName.textContent = userName;

    // Show user's profile and sign-out button.
    this.userName.removeAttribute('hidden');
    this.userPic.removeAttribute('hidden');
    this.signOutButton.removeAttribute('hidden');
     // Hide sign-in button.
    this.signInButton.setAttribute('hidden', 'true');


  } else { // User is signed out!
    // Hide user's profile and sign-out button.
    this.idUserToken = null;
    this.userName.setAttribute('hidden', 'true');
    this.userPic.setAttribute('hidden', 'true');
    this.signOutButton.setAttribute('hidden', 'true');

    // Show sign-in button.
    this.signInButton.removeAttribute('hidden');

  }
};
```

  +  Initialize HTML Bindings in "constructor" - add these

```
  this.submitButton = document.getElementById('submit');

  this.userPic = document.getElementById('user-pic');
  this.userName = document.getElementById('user-name');
  this.signOutButton = document.getElementById('sign-out');
  this.signOutButton.addEventListener('click', this.signOut.bind(this));
  this.signInSnackbar = document.getElementById('must-signin-snackbar');

  var buttonTogglingHandler = this.toggleButton.bind(this);
  this.messageInput.addEventListener('keyup', buttonTogglingHandler);
  this.messageInput.addEventListener('change', buttonTogglingHandler);
```

  +  Add app manifest.json (in static folder)
```
{
  "name": "Google Cloud PSO AppDev Cloud Start",
  "short_name": "GCloud PSO AppDev CS",
  "start_url": "/static/index.html",
  "display": "standalone",
  "orientation": "portrait"
}
```

###  Make the popup work with auth (remove snippet temporarily filled the auth div)

```
  <script type="text/javascript">
  /**
  * @return {string} The reCAPTCHA rendering mode from the configuration.
  */
  function getRecaptchaMode() {
  // Quick way of checking query params in the fragment. If we add more config
  // we might want to actually parse the fragment as a query string.
    return location.hash.indexOf('recaptcha=invisible') !== -1 ? 'invisible' : 'normal';
  };

    // FirebaseUI config.
    var uiConfig = {
      // Url to redirect to after a successful sign-in.
      'signInSuccessUrl': '/static/index.html',
      'callbacks': {
        'signInSuccess': function(user, credential, redirectUrl) {
          if (window.opener) {
            // The widget has been opened in a popup, so close the window
            // and return false to not redirect the opener.
            window.close();
            return false;
          } else {
            // The widget has been used in redirect mode, so we redirect to the signInSuccessUrl.
            return true;
          }
        }
      },
      'signInOptions': [
        // TODO(developer): Remove the providers you don't need for your app.
        firebase.auth.GoogleAuthProvider.PROVIDER_ID,
        firebase.auth.EmailAuthProvider.PROVIDER_ID,
      ],
      // Terms of service url.
      'tosUrl': 'https://www.google.com'
    };

    // Initialize the FirebaseUI Widget using Firebase.
    var ui = new firebaseui.auth.AuthUI(firebase.auth());
    // The start method will wait until the DOM is loaded to include the FirebaseUI sign-in widget
    // within the element corresponding to the selector specified.
    ui.start('#firebaseui-auth-container', uiConfig);
  </script>
```

## Test Canary using Version Directly and Try to Login

* You get error because domain is not added in firebase auth domain. why?

![Domains allowed for oauth authentication](https://storage.googleapis.com/joe-cloudy-public/pso-appdev-cloudstart-assets/s070-authentication-domain.png)

## Important - I have introduced this a a bug. Will follow up.  right now open it only when testing.



# S080 - Let's add python libraries for google (and two handlers)


## Add appengine_config.py (for external libraries inclusion)

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

from google.appengine.ext import vendor

# Add any libraries installed in the "lib" folder.
vendor.add('lib')


```

## Add handlers
* just return the full path requested on get and post
* remember mime type

## Add Private Section Handler
* /private

## Add Rest Handler
* /rest


# S090 - Validate and Sync User
*  /rest/customer
*  /private/info
*  Update how ID Tokens gets refreshed in web 
[Firebase - retrieve and verify token](https://firebase.google.com/docs/auth/admin/verify-id-tokens)



## 

* Follow tutorial (https://cloud.google.com/appengine/docs/standard/python/authenticating-users-firebase-appengine#verifying_tokens_on_the_server)

* Use the sample code of a sample app but trigger it appropriately to make restful call in the app
  +  Click the Enter Button

```
// Fetch notes from the backend.
function fetchNotes() {
  $.ajax(backendHostUrl + '/notes', {
    /* Set header for the XMLHttpRequest to get data from the web server
    associated with userIdToken */
    headers: {
      'Authorization': 'Bearer ' + userIdToken
    }
  })
```

* Reponse in console.log on "success"
```
JSON-Success Object {path: "/rest/customer"}
```

* Add this to your app.yaml (and fix the name of the common firebase)
```
libraries:
- name: ssl
  version: latest

env_variables:
    FIREBASE_PROJECT_ID: 'pso-appdev-cs-1'

```
mkdir localonly
add to .gitignore

```
*  Add appropriate capture of credentials Sample
* Copy FirebaseAdmin json key to localonly and lib
```
import firebase_admin
from firebase_admin import credentials
from firebase_admin import auth as firebase_auth

cred = credentials.Certificate('path/to/serviceAccountKey.json')
default_app = firebase_admin.initialize_app(cred)
```
* Here is the diff - since there are nuances
```
--- a/main.py
+++ b/main.py
@@ -13,20 +13,110 @@
 # See the License for the specific language governing permissions and
 # limitations under the License.
 
+import os
+import sys
+
+import json
+import logging
+import requests
 import webapp2
 from urlparse import urlparse
 
+import google
+if os.getenv('SERVER_SOFTWARE', '').startswith('Google App Engine/'):
+  # Production, no need to alter anything
+  pass
+else:
+  # Local development server
+  google.__path__.pop(0)  # remove /home/<username>/.local/lib/python2.7/site-packages/google
+  # logging.warning(google.__path__)  # to inspect the final __path__ of module google
+
+google.__path__.append('./lib/google')
+import google.oauth2.id_token
+import google.auth.transport.requests
+
+import requests_toolbelt.adapters.appengine
+requests_toolbelt.adapters.appengine.monkeypatch()
+HTTP_REQUEST = google.auth.transport.requests.Request()
+
+import firebase_admin
+from firebase_admin import credentials
+from firebase_admin import auth as firebase_auth
+
+
+if os.getenv('SERVER_SOFTWARE', '').startswith('Google App Engine/'):
+  cred = credentials.Certificate('lib/pso-appdev-cs-1-2a4c1ef76af6.json')
+  # Production, no need to alter anything
+  try:
+    default_app = firebase_admin.get_app()
+  except:
+    default_app = firebase_admin.initialize_app()
+else:
+  # Local development server
+  cred = credentials.Certificate('localonly/pso-appdev-cs-1-2a4c1ef76af6.json')
+  try:
+    default_app = firebase_admin.get_app()
+  except:
+    default_app = firebase_admin.initialize_app(cred)
+
+def getHeader(key, headers):
+  print 'getAuthHeader', headers.items()
+  try:
+      for header in headers.items():
+          print 'header', header, len(header)
+          if len(header) > 1:
+            if header[0] == key:
+              return header[1]
+      print 'getAuthHeader', None
+  except:
+      return None
+  return None
+
+
+def getAuthInfo(request):
+  value = getHeader('Authorization', request.headers)
+  if value and len(value.split(' ')) > 1:
+    id_token = value.split(' ').pop()
+    if (id_token != 'undefined'):
+      claims = google.oauth2.id_token.verify_firebase_token(id_token, HTTP_REQUEST)
+      if not claims:
+         return {'auth': False, 'info': id_token, 'step': 'claims'}
+
+      decoded_token = firebase_auth.verify_id_token(id_token)
+      if not decoded_token:
+        return {'auth': False, 'info': id_token, 'step': 'firebase_decoded'}
+
+      return { 'auth' : True, 'info': decoded_token, 'step': 'success'}
+    return { 'auth' : False, 'info': None}
+  else:
+    return { 'auth' : False, 'info': None}
+
+def getClaimsFromToken(request, response):
+  result = getAuthInfo(request)
+  if (not type(result) is dict) or  (not 'auth' in result):
+    logging.error("Auth Check not returning appropriate dict - key = auth")
+    response.status = '500 - Unexpected Error'
+    return None
+  if not result['auth']:
+    logging.warning("Auth Check for token failed - [{0}]", result)
+    response.status = '401 - Unauthorized Error'
+    return None
+  return result
 
 class RestHandler(webapp2.RequestHandler):
   def get(self):
+    if not getClaimsFromToken(self.request, self.response):
+      return
     parsedUrl = urlparse(self.request.url)
     self.response.headers['Content-Type'] = 'application/json'
     self.response.write(parsedUrl.path)
 
   def post(self):
+    if not getClaimsFromToken(self.request, self.response):
+      return
+
     parsedUrl = urlparse(self.request.url)
     self.response.headers['Content-Type'] = 'application/json'
-    self.response.write(parsedUrl.path)
+    self.response.write(json.dumps({'path': parsedUrl.path}))
```


# S100 - Save/View Policies per User UI with Service Stups


## Policy has policy_id, nickname, owner (user)
* RESTful interface
* Upsert
* /rest/policy
  { 'policy_id': '', 'nickname': '', 'owner':  '' }
* returns 200 or 500

## Create ProcessService in services folder

* signatures
```
  def __init__(self, service_version):
    self.__service_version = service_version

  
  def update_policy(self, policy_info):
  
  def get_policies(self, user_id):
```

* look at the diff between s100 and s090 for UI
index.html, main.js
 

```
diff --git a/lib/__init__.py b/lib/__init__.py
new file mode 100644
index 0000000..e69de29
diff --git a/main.py b/main.py
index 3ddbbed..faac866 100644
--- a/main.py
+++ b/main.py
@@ -22,6 +22,8 @@ import requests
 import webapp2
 from urlparse import urlparse
 
+from services.processing_service import ProcessingService
+
 import google
 if os.getenv('SERVER_SOFTWARE', '').startswith('Google App Engine/'):
   # Production, no need to alter anything
@@ -46,10 +48,11 @@ from firebase_admin import auth as firebase_auth
 
 if os.getenv('SERVER_SOFTWARE', '').startswith('Google App Engine/'):
   # Production, no need to alter anything
+  cred = credentials.Certificate('services/pso-appdev-cs-1-2a4c1ef76af6.json')
   try:
     default_app = firebase_admin.get_app()
   except:
-    default_app = firebase_admin.initialize_app()
+    default_app = firebase_admin.initialize_app(cred)
 else:
   # Local development server
   cred = credentials.Certificate('localonly/pso-appdev-cs-1-2a4c1ef76af6.json')
@@ -104,19 +107,57 @@ def getClaimsFromToken(request, response):
 
 class RestHandler(webapp2.RequestHandler):
   def get(self):
-    if not getClaimsFromToken(self.request, self.response):
-      return
-    parsedUrl = urlparse(self.request.url)
+    try:
+      if not getClaimsFromToken(self.request, self.response):
+        return
+      parsedUrl = urlparse(self.request.url)
+      ps = ProcessingService('V1.0.0')
+      if parsedUrl.path == '/rest/policy':
+        result =  ps.get_policies("jose")
+        if (type(result) == dict and
+              'success' in result):
+
+          if result['success']:
+            self.response.headers['Content-Type'] = 'text/json'
+            self.response.status = result['status']
+            self.response.write(json.dumps(result['payload']))
+          else:
+            self.response.headers['Content-Type'] = 'text/json'
+            self.response.status = result['status']
+          return
+    except Exception as e:
+      logging.error(e)
+
     self.response.headers['Content-Type'] = 'application/json'
-    self.response.write(parsedUrl.path)
+    self.response.status = '500 - Unexpected Error'
 
   def post(self):
-    if not getClaimsFromToken(self.request, self.response):
-      return
+    try:
+      if not getClaimsFromToken(self.request, self.response):
+        return
+
+      parsedUrl = urlparse(self.request.url)
+
+      json_data = json.loads(self.request.body)
+      ps = ProcessingService('V1.0.0')
+      if parsedUrl.path == '/rest/policy':
+        result =  ps.update_policy("jose", json_data)
+        if (type(result) == dict and
+              'success' in result):
+
+          if result['success']:
+            self.response.headers['Content-Type'] = 'text/json'
+            self.response.status = result['status']
+            self.response.write(json.dumps(result['payload']))
+          else:
+            self.response.headers['Content-Type'] = 'text/json'
+            self.response.status = result['status']
+          return
+    except Exception as e:
+      logging.error(e)
 
-    parsedUrl = urlparse(self.request.url)
     self.response.headers['Content-Type'] = 'application/json'
-    self.response.write(json.dumps({'path': parsedUrl.path}))
+    self.response.status = '500 - Unexpected Error'
 
 
 class PrivatePageHandler(webapp2.RequestHandler):
diff --git a/services/__init__.py b/services/__init__.py
new file mode 100644
index 0000000..e69de29
diff --git a/services/processing_service.py b/services/processing_service.py
new file mode 100644
index 0000000..1a6106f
--- /dev/null
+++ b/services/processing_service.py
@@ -0,0 +1,58 @@
+import os
+import logging
+import time
+import datetime
+import random
+from google.appengine.ext import ndb
+
+def service_helper(success, status, payload):
+  logical_success = False
+  if success:
+    logical_success = True
+  return {
+    'success': logical_success,
+    'status': str(status),
+    'payload': payload
+  }
+
+class ProcessingService(object):
+
+  def __init__(self, service_version):
+    self.__service_version = str(service_version)
+
+  def update_policy(self, user_id, policy_info):
+    policy_info['owner'] = user_id
+    return service_helper(True, "201", {
+            'policy_id': '1',
+            'nickname': 'nick1',
+            'owner': user_id
+          })
+
+
+  def get_policies(self, user_id):
+    if (random.randint(1, 10) >= 4):
+      return service_helper( True, 200, {
+        'version': self.__service_version,
+        'list': [
+          {
+            'policy_id': '1',
+            'nickname': 'nick1',
+            'owner': user_id
+          },
+          {
+            'policy_id': '2',
+            'nickname': 'nick2',
+            'owner': user_id
+          },
+          {
+            'policy_id': '3',
+            'nickname': 'nick3',
+            'owner': user_id
+          },
+        ]
+      })
+    else:
+      return service_helper( True, 200, {
+        'version': self.__service_version,
+        'list': []
+      })
diff --git a/services/pso-appdev-cs-1-2a4c1ef76af6.json b/services/pso-appdev-cs-1-2a4c1ef76af6.json
new file mode 100644
index 0000000..ecf15c9
--- /dev/null
+++ b/services/pso-appdev-cs-1-2a4c1ef76af6.json
@@ -0,0 +1,12 @@
+{
+  "type": "service_account",
+  "project_id": "pso-appdev-cs-1",
+  "private_key_id": "2a4c1ef76af6475e57093ffcd9564071855514e9",
+  "private_key": "-----BEGIN PRIVATE KEY-----\nMIIEvAIBADANBgkqhkiG9w0BAQEFAASCBKYwggSiAgEAAoIBAQCYvfZ6x1YmAWV5\nOM2RLrHLqoxlDIjG+4+MQVZSI/3Hd+zmbUq9ke00w9J59ESO0BBjsxyeb1CbD7HA\n0S7VyjuI0nCgppW5aPO+B6tlmVSGi4w8iRfX1TwJWsedK01FEESWGWOSLzkmiP6m\nHawXfh8G/vVqpzuTpGgp2pPlG782vESB13hby6eYXG05rgPa1DRNexsKnD4G9SII\nWWL5r9KMANs9ILqXrLMSyHplibFDRzKpylIqjHxyzFA6MkQcYW2aETWVKRvHclE6\nPk7bmZk1bToycTkPXd0hQ/ftVIHZ+JMSwgMo5fVc4XiCF97A7sbs0yRD+GVvg6Qd\nENmCeKuhAgMBAAECggEAQ/3Dm1HielaCyhxL/YWQpX2Ms2qJ9DGE68Ul3LiivkkX\nDle2Pn6X3bYRmjHu1retpAPWCHy6n9uzn4Y+V/KG39f1RL4Cxh7+6SdW14oSgzXZ\nPhU0pOIJsIxVcRQWeFjOfxZcKXWV9h5jZKSut2JwA1g4/LnmnklACOmAjir0yjMB\nlscCHHkIzMSObuUYlTs1h+VAwgZioy3kAeS99br9ReEByIKHalML1TFDBU1Eh4wO\nBtP1wy1ZsAYtemvnwV9CAwxGpJaeNTJFckksa9Md3TiEdxLZ80MljrK8lFM1GHjl\n6AXpmBsAy5knioIUT/kZO19ERIUajp3m1hXA+KPqVQKBgQDWezUeG/W51boTAGO5\nprMX0NJyXDE+KuNEmfuHDj7xU4JORtFX3h9sUY5GgAN8FFxugNg8FKYJHCrzuVRo\n45QNUGEq3eiO6UFuqL6FT9mvyvYwegZqCSauL+IeyN7pKYY+2/MdOQZh4qRPRT8o\n8afXYf3l7mUbXTE345t1osbPpwKBgQC2Tze8AFP3EFO3PMKzEJmopyRC3Bv/u1nf\neAfGeAeKsq/jUtDfqv1ccfE3moLQLpHXe+pnGyoS3nIjK2R3iGJ/x4xhyy/sSGA7\nBtGTN+pDiNvrvS3tBpwOahm/mN+futPoAeCnYw+GXKzylqWYlDLWr8x2b62tAYxn\n0cPiVs1TdwKBgEnGobPUrEabFOFaXfNLOwlzJCCAQ9P9jqVXTiTbqpz6O7VPOM0/\ns5Ff0E/B0vEIU+8S1M59z8sMbF3fnwBhX9jgkDvdjxQxefdlhft3RwroBp0QLEqn\nES4TfHVYZQzQ4sOWht7DccWT3y8BQ8OCtFgq9dn0kcTC3p455YymTDq7AoGAQmBD\nRZLE/14VbNCVfsabe3knTaSAGTLoPOGhyxPmgwwd1+FOJTFHP8JIddsup4ddGByI\nsnOEdQxCeCWTVaX1XtqTdQOadie/yZ3o7fXcuCv7DjB5qSPP67ubllOdj7Vg88bD\nOY5ql5vkaAqLTise+2VURwbQL/4xVZdc/2plJW8CgYBCLvIrpuKyRR9Aug86oBiI\nxgD0nxJCrcUqFs4lvRcOCyaMwrh34FVni5Kry5m6G8t7uPA3z3uBSQ7m0Z08YGSC\nZ4hYkWQ+N2Gay4RskWc3nG600JSUscjstepLe1NsYmS7m6gjV0GgSDyuhd7/YK1T\n9OHTlYjRJRU94QIwbEGWYQ==\n-----END PRIVATE KEY-----\n",
+  "client_email": "firebase-adminsdk-fziu6@pso-appdev-cs-1.iam.gserviceaccount.com",
+  "client_id": "112021077141642459302",
+  "auth_uri": "https://accounts.google.com/o/oauth2/auth",
+  "token_uri": "https://accounts.google.com/o/oauth2/token",
+  "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
+  "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/firebase-adminsdk-fziu6%40pso-appdev-cs-1.iam.gserviceaccount.com"
+}
diff --git a/static/index.html b/static/index.html
index e251a74..a9dc5e8 100644
--- a/static/index.html
+++ b/static/index.html
@@ -63,7 +63,7 @@
       apiKey: "AIzaSyDQvUHhUTfQnioCjKf7lnr3kM_g2K8dz-8",
       authDomain: "pso-appdev-cs-1.firebaseapp.com",
       databaseURL: "https://pso-appdev-cs-1.firebaseio.com",
-      projectId: "pso-appdev-cs-1",
+      // projectId: "pso-appdev-cs-1",
       storageBucket: "pso-appdev-cs-1.appspot.com",
       messagingSenderId: "159748236642"
     };
@@ -118,15 +118,19 @@
               Entrar
             </button>
           </form>
+
         </div>
-      </div>
 
-      <div id="must-signin-snackbar" class="mdl-js-snackbar mdl-snackbar">
-        <div class="mdl-snackbar__text"></div>
-        <button class="mdl-snackbar__action" type="button"></button>
+        <div id="snackbar" class="mdl-js-snackbar mdl-snackbar" style="position: relative;">
+            <div id="snackbar-text" class="mdl-snackbar__text" style="position: relative;"></div>
+            <button class="mdl-snackbar__action" type="button"></button>
+        </div>
+          <div id="messages-bottom"   class="mdl-snackbar__text"  style="color: #000; display:hidden; text-align: center">
+          </div>
       </div>
 
     </div>
+
   </main>
 </div>
 
diff --git a/static/scripts/main.js b/static/scripts/main.js
index e325b02..31ef864 100644
--- a/static/scripts/main.js
+++ b/static/scripts/main.js
@@ -110,14 +110,56 @@ FriendlyChat.prototype.getWidgetUrl = function()  {
 // Signs-out of Friendly Chat.
 FriendlyChat.prototype.signOut = function() {
   // Sign out of Firebase.
+  console.log('signOut');
+
   this.auth.signOut();
+  this.userIdToken = "";
 };
 
+FriendlyChat.prototype.getInfo = function() {
+  var self = this;
+  var userIdToken = this.userIdToken;
+  $.ajax('/rest/policy', {
+  /* Set header for the XMLHttpRequest to get data from the web server
+  associated with userIdToken */
+    headers: {
+      'Authorization': 'Bearer ' + userIdToken
+    },
+    'data': JSON.stringify({'prop': 'value'}),
+    'type': 'get',
+    'dataType': 'json',
+    'success': function(json_data) {
+      console.log('JSON-Success', json_data)
+      if ( json_data['list'] && json_data['list'].length > 0 ) {
+        console.log('list', json_data['list']);
+        $("#messages").empty();
+        for(var o in json_data['list']) {
+          console.log('list_entry', o);
+          var innerInfo = $('<div  style="padding-bottom: 10px"  />');
+          innerInfo.append( $('<span style="padding-right: 10px" />').text('Numero'));
+          innerInfo.append( $('<span style="padding-right: 10px" />').text(json_data['list'][o]['policy_id']));
+          $("#messages").append(innerInfo);
+          var innerInfo = $('<div  style="padding-bottom: 10px"  />');
+          innerInfo.append( $('<span style="padding-right: 10px" />').text('Nombre'));
+          innerInfo.append( $('<span style="padding-right: 10px" />').text(json_data['list'][o]['nickname']));
+          $("#messages").append(innerInfo);
+        }
+      } else {
+        $("#messages").empty();
+        $("#messages").append($('<div  style="padding-bottom: 10px"  />').text("Ninguna Poliza"));
+      }
+    },
+    'error': function(error) {
+        console.error('submitButtonAction error in post', error);
+    },
+  });
+}
 
 FriendlyChat.prototype.submitButtonAction = function() {
     var userIdToken = this.userIdToken;
     console.log('submitButtonAction', userIdToken);
-    $.ajax('/rest/customer', {
+    var self = this;
+    $.ajax('/rest/policy', {
     /* Set header for the XMLHttpRequest to get data from the web server
     associated with userIdToken */
     headers: {
@@ -128,6 +170,9 @@ FriendlyChat.prototype.submitButtonAction = function() {
     'dataType': 'json',
     'success': function(json_data) {
       console.log('JSON-Success', json_data)
+      $('#messages-bottom').text("Actualizado")
+        .css("display","block").fadeOut(2000);
+      self.getInfo();
     },
     'error': function(error) {
         console.error('submitButtonAction error in post', error);
diff --git a/static/widget.html b/static/widget.html
index 930e283..4405714 100644
--- a/static/widget.html
+++ b/static/widget.html
@@ -54,7 +54,7 @@
         apiKey: "AIzaSyDQvUHhUTfQnioCjKf7lnr3kM_g2K8dz-8",
         authDomain: "pso-appdev-cs-1.firebaseapp.com",
         databaseURL: "https://pso-appdev-cs-1.firebaseio.com",
-        projectId: "pso-appdev-cs-1",
+        // projectId: "pso-appdev-cs-1",
         storageBucket: "pso-appdev-cs-1.appspot.com",
         messagingSenderId: "159748236642"
       };
```

