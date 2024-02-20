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
## Terraform Prerequisites
This post won't cover all the intricacies of setting up your system to run `gcloud` and `terraform`.  Follow the links below for more information on how to accomplish that.
- Google Cloud Platform (GCP) account
- Project created in GCP
- Billing enabled for the project
- [Google Cloud CLI installed and configured](https://cloud.google.com/sdk/docs/install#deb)
- [Terraform installed](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli)

