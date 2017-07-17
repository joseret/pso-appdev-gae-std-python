# pso-appdev-gae-std-python
Google PSO - AppDev - GAE - Standard - Python

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


