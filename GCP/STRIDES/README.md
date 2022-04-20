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

## Notes from Jason


* Technically, a container *is* a VM, but it is a greatly reduced VM as containers have just minimum libraries 
(in read-only layers), to run what it is supposed to run.


* When you compare a container to a VM, say a VMWare VM, the VMWare VM has all of the things an OS normally has 
and is GB in size. Most Containers are measured in MB rather than GB and are much more efficient on a machine's resources.


* Which is why containers are what all of the cool kids are using these days.


## Cloud Storage

* Binary large-object storage
    * Hi-perf S3 equivalent; also called Buckets
    * By default encrypted at rest (actually you can't turn this off)
        * This rule also applies to database services
    * By default encrypted in transit from Google to endpoint
    * Standard, Nearline, Coldline, Archive storage (tiers)
        * Respective minimum durations are none, 30 days, 90 days, 365 days
    * Use cases respectively:
        * Hot
        * Access less than once per month: Data backup, long tail media
        * Access less than once per quarter
        * Access less than once per year (DR, archival requirements)

### GCP Storage Types including Cloud Storage

* Firestore (NoSQL)
* Cloud Bigtable (NoSQL wide column)
* Cloud Storage (Blobstore object: see above)
* Cloud SQL (Relational SQL for OLTP: Smallish)
* Cloud Spanner (Relational SQL for OLTP: Petabytes)
* Big QUery: Relational SQL for OLAP

Big Query is a managed service. The idea is the User only worries about how to send data in; and how to run queries, and everything else is handled for you.

Big Query has a built-in composer/editor with a RUN button to facilitate Big Query.

Because queries can be $ expensive $ do not be hesitant about learning how to write them effectively. That's a topic for another day.


## Questions I Have

* What is the best way to fall prey to a ransomware attack?
