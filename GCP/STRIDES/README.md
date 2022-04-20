# Introduction: Fundamentals of Life Science Tools on Google Cloud

Course offered by Rui Costa, Christian Michael, Jason Ehrhart; coordinated by Amanda Tan at C.L.A.S.S. (Internet 2).
This is a hands on that starts by moving a container to Google Container Registry from Docker Hub.

# Freeform notes

## Cloud Code

**Cloud Code** is a plugin available for IntelliJ and VSCode and Google Cloud Shell: To enable the connection to GCP. 


## gcloud CLI

```
gcloud compute instances list
```

## Cloud Shell and Cloud Code

* The little button in the console
* Provisioning instance
    * Do changes persist? ***NO!!!*** It is a clean slate deal (except see below on my home directory)
    * Pro Tip: Use `top` to keep the Cloud Shell process hot while you step away for > 15 minutes (keeps the container busy)
    * There is 5GB of storage in my home directory does persist; so data uploads and so on will not evaporate on bounce
* Instance has `gcloud` pre-installed
    * Assigned to my account, not to a project
    * See the tabs at upper left of the Cloud Shell window: This is a drop-down for Projects I can access.
        * This contextualizes `gcloud` command actions
* Cloud Shell has an Editor that has Cloud Code pre-installed
    * Let's abbreviate this CSE
    * Note there is an Enable Cloud Run API
        * Formerly we had to do this in the console using the left menu "API" section; so nice to do from within CSE
        * cloud-run and k8 are the services that are enabled in this path for the API; so constrained in that sense
* 
