---
title:  "Deploying to Cloud Run with Terraform"
date:   2024-02-18 20:00:00 -0500
categories: github jekyll
mermaid: true
tags: chirpy
---

Clound Run is a popular choice for serverless computing on GCP.  It allows you to easily deploy and run a Docker Image with very little work.  It is much simpler than maintaing your own Kubernetes cluster.  There is even a [source code based deploy](https://cloud.google.com/run/docs/deploying-source-code) option if you don't want to make your own image.

Cloud Run is a great alternative to using App Engine and comes recommended from Google.
[GCP: App Engine vs Cloud Run](https://cloud.google.com/appengine/migration-center/run/compare-gae-with-run)

Cloud Run can be setup as a `service` to host an API or as a `job`.  In this example we'll deploy a Cloud Run service.  It will host a Scala application running the Play Framework to create a REST API.


# Building the Scala App

## Scala Prerequisites
- Git
- SBT Installed (or IntelliJ with Scala Plugin)
- JDK 11 or Greater

## Play Framework
The Play Framework is one of many popular REST API frameworks in the Scala ecosystem.  Play Framework may not be as new and shiny as some frameworks built off of aschronous composable runtimes, such as [Cats Effect and ZIO](https://softwaremill.com/cats-effect-vs-zio/), but its a good choice for those looking to stand up a REST API quickly in Scala.

## Source Code
The source code for this entire example, including both the Scala application and Terraform, is available in Github: [https://github.com/brandon-setegn/scala-play-example](https://github.com/brandon-setegn/scala-play-example)

```bash
git clone https://github.com/brandon-setegn/scala-play-example.git
```

## Building the Scala Application
After you clone the repo, either [open it in IntelliJ](https://docs.scala-lang.org/getting-started/intellij-track/getting-started-with-scala-in-intellij.html) or navigate to it from the command line.  [Open the project in SBT](https://docs.scala-lang.org/getting-started/sbt-track/getting-started-with-scala-and-sbt-on-the-command-line.html).

### Run the Application Locally
To run the project, simply enter `run` into the SBT shell.  By default, it should start the app listening on port 9000.  You can view the output at [http://localhost:9000](http://localhost:9000).  There is only one endpoint serving "Hello World" at the base URL.
```scala
# Hello World
GET   /     controllers.HelloWorldController.index()
```

### Building a Deploy Package

To build a **Linux** compatible package as a tgz file, use the **sbt** `Universal / packageZipTarball` command.  More details here: [Play: The Native Packager](https://www.playframework.com/documentation/3.0.x/Deploying#The-Native-Packager).  If you are running this project from Windows, I recommend using [Windows Subsystem for Linux](https://learn.microsoft.com/en-us/windows/wsl/about) to run a Linux shell and execute this command inside **sbt**.

```bash
Universal / packageZipTarball
```

If the build was successfull, you should see this package: `\target\universal\play-scala-rest-api-example-1.0-SNAPSHOT.tgz"` in your project folder.

### Build and Push a Docker Image
There is a `Dockerfile` located in the root of the project folder.  This will be be used to build a Docker Image that will be pushed to *Google Artifact Registry (GAR)*.  Cloud Run will run this image for us when the process is complete..

```dockerfile
FROM amazoncorretto:21.0.2

EXPOSE 8080

ADD ./target/universal/play-scala-rest-api-example-1.0-SNAPSHOT.tgz /app

CMD /app/play-scala-rest-api-example-1.0-SNAPSHOT/bin/play-scala-rest-api-example -Dhttp.port=8080
```

#### Docker Build Command and Push to GCP
Now, from the console you can build the Docker Image with the following command.  When we build this Docker Image, we will tag it with a specific name that will tell it where to go inside Google Cloud Platform (GCP).

##### Configure Docker and Google Artifact Registry
Follow the guide below to full setup Google Artifact Registry (GAR) and your local Docker authentication.  [GCP Guide: Store Docker Image in Artifact Registry](https://cloud.google.com/artifact-registry/docs/docker/store-docker-container-images)
> If you have trouble pushing your image to GAR, make sure to follow the steps to authenticate [GCP Docker Auth](https://cloud.google.com/sdk/gcloud/reference/auth/configure-docker)


The Image will be tagged `us-east1-docker.pkg.dev/{your-project-id}/cloud-run-example/scala-play-example`.  Repalce `{your-project-id}` with your [Project ID from GCP](https://support.google.com/googleapi/answer/7014113?hl=en).

This tag is important.  It specifies which location in GCP to go to, `us-east1`, and which project to go to, `{your-project-id}`.
```bash
docker build -t us-east1-docker.pkg.dev/{your-project-id}/cloud-run-example/scala-play-example .
```
If `docker build` ran succesfully, we can now push our Docker Image to GCP:
```bash
docker push us-east1-docker.pkg.dev/{your-project-id}/cloud-run-example/scala-play-example
```

**Now that the image has been pushed to GAR, we're ready to work on our `Terraform` to create our Cloud Run Service.**

# Terraform

> ⚠️**Important: Make sure not to hard code any GCP values or Environment Variables.**

## Terraform Prerequisites
This post won't cover all the intricacies of setting up your system to run `gcloud` and `terraform`.  Follow the links below for more information on how to accomplish that.
- Google Cloud Platform (GCP) account
- Project created in GCP
- Billing enabled for the project
- [Google Cloud CLI installed and configured](https://cloud.google.com/sdk/docs/install#deb)
- [Terraform installed](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli)


## Add Terraform to the Project
This sample project already has the `Terraform` files you need to deploy to Cloud Run.  All of the `Terraform` files go in the `terraform` directory in the project.  This is where you will run `Terraform` commands from the command line.

#### Terraform Project Setup
> This `Terraform` guide describes building your own local module: [https://developer.hashicorp.com/terraform/tutorials/modules/module-create](https://developer.hashicorp.com/terraform/tutorials/modules/module-create)

|File|Description|
|---|---|
|*main.tf*|will contain the main set of configuration for your module|
|*output.tf*|will contain the variable definitions for your module|
|*variables.tf*|will contain the output definitions for your module|


## Configure Terraform for Cloud Run
This post will only cover the specifics of using `Terraform` for `Cloud Run`.

### main.tf
This is our module that will be run to deploy to `Cloud Run`.  There are two sections to our `main.tf`, *service_account* and *cloud_run*.

#### Service Account Module
This section creates a *service account* in GCP to be used by our `Cloud Run` service.  This section must come before the *cloud_run* section.  Since our Cloud Run service will access a few *Secrets* in GCP to be used as environment variables, it needs the `roles/secretmanager.secretAccessor` role.

> ⚠️**Important**: This section uses the *var.project_id* variable.  This is so your `project_id` isn't checked into GitHub and visible to others.

```terraform
module "service_account" {
  source     = "terraform-google-modules/service-accounts/google"
  version    = "~> 4.2"
  project_id = var.project_id
  prefix     = "sa-cloud-run"
  names      = ["simple"]
  project_roles = ["${var.project_id}=>roles/secretmanager.secretAccessor"]
}
```

#### Cloud Run Module
This section creates the actual `Cloud Run` service.  It also uses the *var.project_id* variable to make sure your project id and other sensitive information isn't checked into GitHub.

#### Cloud Run Environment Variables and Secrets
The Scala application requires two enviornment variables to run.  These **MUST** be created in *manually* `GCP Secrets Manager` before the Cloud Run service is created.  Follow this [QuickStart: Secret Manager](https://cloud.google.com/secret-manager/docs/create-secret-quickstart) guide to learn how to create secrets in GCP.

|Environment Variable Name|Description|
|---|---|
|*APPLICATION_SECRET*|Secret key used by [Play Framework](https://www.playframework.com/documentation/3.0.x/ApplicationSecret)|
|*ALLOWED_HOST*|[Hosts allowed to connect to service](https://www.playframework.com/documentation/3.0.x/AllowedHostsFilter)|

The *APPLICATION_SECRET* can be set to any approriate length string necessary.  However,*ALLOWED_HOST* must be set to the final URL of our Cloud Run service.  This can be found by running `terraform plan` as described below.  One of the outputs will be the `service_url` needed to set the *ALLOWED_HOST* environment variable.


> Note: The docker tag used previously is the same used for the Cloud Run `image`

```terraform
module "cloud_run" {
  source  = "GoogleCloudPlatform/cloud-run/google"
  version = "~> 0.10"

  service_name          = "cloud-run-example-scala"
  project_id            = var.project_id
  location              = "us-east1"
  image                 = "us-east1-docker.pkg.dev/${var.project_id}/cloud-run-example/scala-play-example:latest"
  service_account_email = module.service_account.email
  env_secret_vars       = [
    {
      name = "APPLICATION_SECRET"
      value_from = [
        {
          secret_key_ref = {
            name = "cloud-run-example-play-secret"
            key  = "latest"
          }
        }
      ]
    },
    {
      name = "ALLOWED_HOST"
      value_from = [
        {
          secret_key_ref = {
            name = "cloud-run-example-play-host"
            key  = "latest"
          }
        }
      ]
    }]
}
```


#### IAM Resource

> ⚠️**Important**: This section grants `invoker` to `allUsers`.  This means that *ANYONE* can hit the URL of your `Cloud Run` service.  Make sure to turn off your service when done testing or your may incur extra charges.

```terraform
resource "google_cloud_run_service_iam_binding" "binding" {
  project  = module.cloud_run.project_id
  location = module.cloud_run.location
  service  = module.cloud_run.service_name
  role     = "roles/run.invoker"
  members = [
    "allUsers"
  ]
}
```

## Running Terraform
The sample project comes with the `Terraform` files required to run and deploy the `Cloud Run` service.  All you need to have completed to run the `Terraform` step is to have configured and installed `Terraform`.

### Test the Terraform Module
To simply see the output of the `Terraform` process in our console, run the following command.  You will be asked to input your GCP project id.
```bash
terraform plan
# Output below
var.project_id
  The project ID to deploy to

  Enter a value: {your-project-id}
```

### Running the Terraform Module
To actually create the `Cloud Run` service, run this command.  You will be asked to input your GCP project id.
```bash
terraform apply
# Output below
var.project_id
  The project ID to deploy to

  Enter a value: {your-project-id}
  ....
# Final Line outputted hsould be
service_url = "https://{your-unique-cloud-run-url}.app"
```

### Testing Our Cloud Run Service
Simply navigate to the `service_url` outputted from our `terraform apply` command.  You should be greated with `Hello World`.

## Cleaning Up Our Service
⚠️Make sure to clean up your project so that you are not billed for the service.
```
# Destroy the GCP resources created by Terraform
terraform destroy
```