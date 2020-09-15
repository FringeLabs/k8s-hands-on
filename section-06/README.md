# Section 6

These examples go further the fact Pods are not accessible from outside the cluster. They ar enot visible. That is because Pods have ephemeral lifecycle, which means, short ones.

## K8S services

It is a long-running object that has IP address and stable fixed ports. We can attach Services to PODs to expose apps outside a cluster. Service will find a suitable port for the requested "service".

POD lables are the key to map services to PODs. A "selector" on service side matches with POD labels to connect a Pod to service.

Pods come and goes. For all long running apps it is expected to have services in Kubernetes.


### Applying the changes to the cluster

The syntax to apply anything to K8s is the same:
```
kubectl apply -f ./section-06/webapp-service.yaml
```
After that, we can get all the information using `kubectl get all` and we should see a result like this:
```
NAME         READY   STATUS    RESTARTS   AGE
pod/webapp   1/1     Running   0          4h25m

NAME                      TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
service/fleetman-webapp   NodePort    10.104.8.154   <none>        80:32483/TCP   39s
service/kubernetes        ClusterIP   10.96.0.1      <none>        443/TCP        19h
```

This is not going to work unless we add a label to the pod declaration.

### Releasing service updates

To avoid downtimes when releasing updates (on this example) the simplest way is to use labels. We can create a label called `release` so it has to match both labels: the `app` and `release` values. The selector work as an **AND** operator.

I've just added a new pod definition inside the same file (separating with 3 dashes) and updated the service to have the label release label set as "0". Then I've ran these commands:
```
kubectl apply -f ./section-05/first-pod.yaml
kubectl apply -f ./section-06/webapp-service.yaml
```

To check which POD is bound to the service you can run
`kubectl describe service/fleetman-webapp` or `kubectl describe svc fleetman-webapp`. The rollout is just a matter to change the `release` label value on service definition and apply the change. ***NOTE***: browser can cache the results once it is a web app.

### Other useful commands

Very useful for troubleshooting:
* `kubectl get pods` or `kubectl get po`
* `kubectl get po --show-labels`
* `kubectl get po --show-labels -l release=0`
* `kubectl get po --show-labels -l release=0-5`