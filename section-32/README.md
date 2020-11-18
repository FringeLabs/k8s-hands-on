# Other workload types

Several other types of workloads that don't fit in the app.

## Batch Jobs

It will create a pod and K8s it will make sure it runs successfully until completion, like a process that has to be executed until it is over instead of running forever.

If you create as a Pod, you will notice that `Restart policy` will make this pod keeping restarting once the command ends. Obviously you can configure it to restart on failures or to never be restarted. The problem with this is: what happens if the job goes wrong?

Please check the K8s documentation on the workloads `Job`.


## Cron Jobs

The kind `CronJob` can be found at `batck/v1beta1` (apiVersion). It behaves exaclty like Cron tab and you aare able to schedule jobs to be executed with same rules. You can always `get` or `describe` a cronjob on `kubectl`, like `kubectl get cronjob`.


## DaemonSets

A DaemonSet ensures that all nodes run a copy of a Pod. As nodes are added to the cluster, Pods are added to them.
It may be very useful to run agents. It is rare on the microservices architecture have a service like this.

In order to create a DaemonSets you can just declare a `Deployment` and change the type to `DaemonSet`, removing the field `replicas`.


## StatefulSets Overview

Object to manage stateful applications. It is important to understand **IT'S NOT USED FOR PERSISTENCE** on k8s.
We didn't use it so far, but we have persistence. Mongo pod saves data on hard disk, outside the containers (on EBS).

Statefulset means on K8s something very specific: we treat pods like cattle, not pets. But sometimes you need set of pods with known, predictable names...and you want clients to be able to call them, **by their name**.

### Treating your pods as pets

if the reason to maintain the name is a requirement to access the app with predictable names, From K8s 1.3 and afterwards we have an object called "PetSet":
* In a `PetSet` the pods will have a predictable name: 0, 1, 2...
* The pods will always start up in sequence;
* Client can address them by name (a very useful feature);

### A PetSet with "Headless Services"

There is no syntax in the yaml for headless services. This is when the service is loadbalancing based on client requests that are referencing to a specific pods.

`So, StatefultSet is the name rebranding of PetSet`. 

## Statefulsets for Database Replication

When do use Statefulsets? When you have a database **but you want to replicate it**. Databases can't usually be replicated by simple ReplicaSets/Deployments.

It depends of the database implementation but usually or often at least, you can't just replicate a database using deployments.
To have a client accessing the database you need predictable names: `mongodb://mongo-0.mongodb` (when you know the primary) or `mongodb://mongo-0.mongodb,mongo-1.mongodb,mongo-2.mongodb`.

## Scaling out a Mongo Database

We created DB pods for example purposes, but in reality, they are hard to manage inside pods. Almost always, if I possibly can, prefer go to a hosted service.

**Note**: to make this work you need a side car (https://github.com/cvallance/mongo-k8s-sidecar). Check the page to understand the details of it to have mongo working on the example.

