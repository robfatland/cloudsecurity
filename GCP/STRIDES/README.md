# Introduction: Fundamentals of Life Science Tools on Google Cloud

Course offered by Rui Costa, Christian Michael, Jason Ehrhart; coordinated by Amanda Tan at C.L.A.S.S. (Internet 2).
This is a hands on that starts by moving a container to Google Container Registry from Docker Hub.

# Lecture notes

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

BigQuery -- being charged for querying your data -- is cost-dangerous. So there is Jupyter cell magic `%%bigquery`
that enables us to run queries; but it is risky; whereas inside the console editor with syntax checking we can 
be sure it will run before running it in a Python shell. 

## Docker containers

Docker ***files*** describe how to build container images. This includes specifying an entry point: What to run on execution.
Images are files that contain the docker environment; which are distributable / movable / runnable.
Docker containers are images that have been executed. If the entry point is ... somehow ... "be an operating system" versus "run this code".


A key container idea is creating mount points that exist within the container and external to the container (some sort of alias) so that data can be configured externally and made available to the container; and results can be written to a container-local directory which exists simultaneously outside in the computer's environment. This gives us send/receive from the reality world of the computer to the toy reality world of the container.



## Lab 1 procedure

```
docker pull nextflow/rnaseq-nf

<lots of output>

docker images

<should list what the first command downloaded>

docker create --name rnaseq-nf nextflow/rnaseq-nf:latest

<creates a container but does not start it>

docker ps

<shows what is running: nothing> 

docker ps -a

<lists running and not-running containers including the image we downloaded above with docker pull>
```

At this point we have a configuration "change" opportunity. Note containers can be created empty; or with some basic tools; and go from there.
But in this case we have a boutique container courtesy of the nextflow organization. 


```
docker commit rnaseq-nf
docker images

<now we see two images: Note the untagged one ID which we use below>

echo "export PROJECT_ID=$(gcloud config get-value project)" >> ~/.bashrc
exec bash
```

There follows a few steps to get the image I want into the GCP container registry.

```
echo "export PROJECT_ID=$(gcloud config get-value project)" >> ~/.bashrc
exec bash
docker tag b2b18bc423fe gcr.io/$PROJECT_ID/rnaseq-nf
docker images
docker push gcr.io/$PROJECT_ID/rnaseq-nf
```

Go to container registry to verify that the `rnaseq-nf` container is present.


## Running Nextflow pipelines in Google Cloud

Now that we have the nextflow pipeline container in the GCR (Google Cloud Container Registry): We get to actually run the pipeline. 

```
gcloud services enable lifesciences.googleapis.com
gsutil mb gs://$PROJECT_ID-nextflow
echo "export PROJECT=$(gcloud config get-value project)" >> ~/.bashrc
echo "export SERVICE_ACCOUNT_NAME=nextflow-service-account" >> ~/.bashrc
exec bash
echo "export SERVICE_ACCOUNT_ADDRESS=${SERVICE_ACCOUNT_NAME}@${PROJECT}.iam.gserviceaccount.com" >> ~/.bashrc
exec bash
gcloud iam service-accounts create ${SERVICE_ACCOUNT_NAME}
gcloud projects add-iam-policy-binding ${PROJECT}     --member serviceAccount:${SERVICE_ACCOUNT_ADDRESS}     --role roles/lifesciences.workflowsRunner
gcloud projects add-iam-policy-binding ${PROJECT}     --member serviceAccount:${SERVICE_ACCOUNT_ADDRESS}     --role roles/iam.serviceAccountUser
gcloud projects add-iam-policy-binding ${PROJECT}     --member serviceAccount:${SERVICE_ACCOUNT_ADDRESS}     --role roles/serviceusage.serviceUsageConsumer
gcloud projects add-iam-policy-binding ${PROJECT}     --member serviceAccount:${SERVICE_ACCOUNT_ADDRESS}     --role roles/storage.objectAdmin
```


and then...


```
echo "export SERVICE_ACCOUNT_KEY=${SERVICE_ACCOUNT_NAME}-private-key.json" >> ~/.bashrc
exec bash
gcloud iam service-accounts keys create --iam-account=${SERVICE_ACCOUNT_ADDRESS} --key-file-type=json ${SERVICE_ACCOUNT_KEY}
echo "export SERVICE_ACCOUNT_KEY_FILE=$PWD/$SERVICE_ACCOUNT_KEY" >> ~/.bashrc
exec bash
echo "export GOOGLE_APPLICATION_CREDENTIALS=$PWD/$SERVICE_ACCOUNT_KEY" >> ~/.bashrc
exec bash
```


then...

```
echo "export NXF_VER=20.10.0" >> ~/.bashrc
echo "export NXF_MODE=google" >> ~/.bashrc
exec bash
curl https://get.nextflow.io | bash
```





## Questions I Have

* What is the best way to fall prey to a ransomware attack on GCP?
* What is `exec bash` versus `~/.bashrc`?
* What am I looking at installing / running in my laptop in order to work on GCP in this way programmatically?
    * i.e. what is required to invisibly authenticate? Is there a reference / guide for this?






