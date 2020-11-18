# Continuous Deployment on a Kubernetes Cluster

Some notes about one of the best practices on modern software development.

## Introducing CI/CD

Every team used to work on their modules and at some point we had a stage where we should integrate them, so the system could become functional. All of those components that never had spoken each other should go through the integration phase (this is a travel back to 1991).

The goal is instead of having a integration phase in the project, we should keep ***integrating continuously***. For that we should have a repository, a build process and "self-testing" support.

Today we use CI and CD terms in a interchangeable way, but the oversimplification is with CD we have the next step. CI is there to make sure the system is always buildable. CD is an extension of CI to guarantee we always have a **live** version of the system up & running with latest and greatest features.

The first successfull implementation of CI at the time was CruiseControl (Mar between 2001 - Sep 2010). Then we had Hudson and because a licensing issue, a fork of this project in 2011 became the most broad used CI tool: **Jenkins**.

**Note**: it worth to check the project called Jenkin X (https://jenkins-x.io/).

## Establishing a Github Organization

No notes that worth to add here.

## Setting a Basic Jenkins System

* Jenkins need to have the Kubernetes plugin installed amd kubernetes-cli;
* Docker Workflow may be a good call as well;
* You may want to rund docker inside Minikube for building Jenkins image if you intend to run it inside a pod  (which is the standard);

## Defining a Pipeline

* A pipeline is the key for building a project. It is a configuration to inform how to build the project;
* We need to add Github credentials into Jenkins credentials;
* The trick is also use the $REPOSITORY_TAG env var from jenkins to name containers images when building the project;
* You may need to use `envsubst` command on jenkins to replace environment variables on yaml files before applying with kubectl: `sh 'envsubst < ${WORKSPACE}/deploy/yaml | kubectl apply -f .`;

## Running a Multibranch Pipeline

* It is way better than pipeline. We can adjust to run the same pipeline for multiple branches in parallel and add the build number to the image if we want to;


## Reviewing Builds

* we can use `watch` command on linux to watch `kubectl get po` and see pods being created / terminated during build process;

## Organization Pipelines

* Jenkins allow to create a Github Organization to work with all pipelines from all projects;
* All pipelines will be created automatically;


## Continuous Deployment into a Cluster

**Important**: Don't forget to enable the webhooks on Github Organization to ensure github github and jenkins will talk each other properly. 


