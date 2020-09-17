# Microservices Architecture

## WARNING - possible resource problems!

In this section, we're going to be deploying a lot more resources to your cluster.

I've realised since recording that on some systems, there won't be enough resources (RAM) in minikube to manage the full load. For some reason, I got away with it on the videos (although in a later section, when we use Mongo, I did have to expand the RAM).

I recommend before starting this section that you set up minikube with plenty of RAM. To do this:

1. Stop minikube with `minikube stop`
2. Delete your existing minikube with `minikube delete`
3. Remove any config files with `rm -rf ~/.kube` and `rm -rf ~/.minikube`. Or, delete these folders using your file explorer. The folders are stored in your home directory, which under windows will be "c:\Users\<your username>"
4. Now restart minikube with `minikube start --memory 4096`.

This will allocate 4Gb of RAM to minikube (I'm assuming you have enough host ram to support this) and should give a much more comfortable experience in the next few sections.

## An introduction to Microservices

Just a contrast to traditional architecture (monolith apps) with integration databases (databases shared across multiple apps). It works until one business area starts to break other business areas inside the system.

You can think about a microservice as a component or subsystem that can be developed, deployed and maintained by their own w/o interfeering on other microservices and they communicate each other through well-defined interfaces. That means they are **self-contained**.

All of this implies a microservices should deal with onnly one specific area of the business.

With that, they are:
* highly cohesive: dealing one business requirement, addressing one busines concern at the time;
* loosely coupled: minimal interface (dependencies) between microservices - using messaging and events;

### About databases in microservices

Databases may be built through years, getting as big as you can imagine, violating all microservices principles one they contains multiple business areas data as many apps accessing and performing operations.

For data modeling tips, please check BoundedContext: https://www.martinfowler.com/bliki/BoundedContext.html

I don't like the word "context" but that gives an idea about modeling data on separate tables or even databases and understand the replication of this data based on the "usage" of the data.

## Fleetman Microservices - setting the scene

It is a system for transport company (like logistics). Many vehicles reporting positions at same time to the system.

The architecture:
* Browser <- NGINX (Reverse Proxy) <- API Gateway <- Position Tracker <- ActiveMQ <- Position Simulator
  

The pieces:
* ***Position Simulator***: simulates vehicles sending position from around the globe. When executed, it reads a set of files and simulates vehicles. ALl this data goes to the queue system;
* ***ActiveMQ***: the queue system, right?
* ***Position Tracker***: reads from the queue the position coordinates and do several calculations on the top of it;
* ***API Gateway***: abstracts the entire backend for front-end apps. Position Tracker can get more and mroe complicated and may be splitted into more microservices. As simple microservices could be merged. So, this layer prevents direct access to MS allowing them to get coupled - https://microservices.io/patterns/apigateway.html. NGINX can play this role as well;


## Deploying the system

* Deploying the pods, deployments and so on: `kubectl apply -f ./section-11/workloads.yaml`;
* Inspecting Pod logs: `kubectl logs position-simulator-54c465565f-rf2mr` or `kubectl logs position-simulator-54c465565f-rf2mr -f` to follow the logs. be aware if the container is restarted you are going be kicked out of the logs you are tailing.
* Deploying services: `kubectl apply -f ./section-11/services.yaml`;