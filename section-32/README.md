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

