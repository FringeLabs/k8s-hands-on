# Deployments Overview

Deployments on k8s can be defined as ReplicaSets with an additional feature: the ability to get automatic rolling updates with **zero downtime**. But that is an oversimplification.

Deployments are "ways" to manage replicasets. IF we run:
```
kubectl apply -f section-09/pods.yaml 
kubectl get all
```
We will notice the deployment is going to create a replicaset. In the same way, the replicaset will create pods.
At the end, the pods will have in the name the "ID" if the replicaset as another for the pod itself.

It should return an output similar to the one below:
```
➜  k8s-hands-on git:(master) ✗ kubectl get all
NAME                          READY   STATUS    RESTARTS   AGE
pod/queue                     1/1     Running   0          3m3s
pod/webapp-77896f4bf8-6nwxc   1/1     Running   0          3m3s
pod/webapp-77896f4bf8-wss2s   1/1     Running   0          3m3s

NAME                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/fleetman-queue    NodePort    10.109.4.209    <none>        8161:30010/TCP   25m
service/fleetman-webapp   NodePort    10.107.122.21   <none>        80:30080/TCP     25m
service/kubernetes        ClusterIP   10.96.0.1       <none>        443/TCP          45h

NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/webapp   2/2     2            2           3m3s

NAME                                DESIRED   CURRENT   READY   AGE
replicaset.apps/webapp-77896f4bf8   2         2         2       3m3s
```

### Managing rollouts

If a rollback is needed, there are special commands for it. First, you can see current rollout status running:
```
kubectl rollout status deployment webapp
```
**NOTE**: You can use `deploy` instead of `deployment` as a short notation.

If something goes wrong, you can just use the following commands:

```
kubectl rollout history deploy webapp
kubectl rollout undo deploy webapp --to-revision=2
```

**IMPORTANT**: The revision parameter is optional. If you want to rollback one version you can always supress this parameter. By default, k8s rollback one versions. By default, K8s will "remember" the last 10 releases of a depoloyment.


### Emergency options

Rollback is a nice feature to be used on emergency. After rolling back you want to go back and carry on with yaml files to manage the deployments. Otherwise, the files will get to an inconsistent state and you may never know what is deployed/running in yhour cluster.
