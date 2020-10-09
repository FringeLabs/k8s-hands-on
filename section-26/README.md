# Horizontal Pod Autoscaling

You can configure to automatic scale replicas of your Pods automatically. But not every pod should be replicated. You need to be aware of the software running inside the pod to determine if the software will still work after having an extra instance running.

## Introducing Replication and Autoscaling

To trigger autoscaling automatically, we need to install HPA (Horizontal Pod Autoscaling). With this mechanism, we can have rules based on CPU, memory and so on to manage the number of pods.

It also works in reverse. If you set CPU rule on 50%, a new pod will be created when CPU passes 50% and one or more are going to be deleted when dropping below that value.

You can manage HPA using `kubectl autoscale deployment <name_of_deployment or replica> --cpu-percent 400 --min 1 --max 4` (the value is the percentage of what was reserved for the pod - if cpu is set as 50mi we are talking here about 200mi). Personally, a better way to manage it is through YAML files.

After that you can run `kubectl get hpa` to check details.

### Working with YAML files

One way to retrieve the YAML representation of the command we ran before is through `kubectl get hpa api-gateway -o yaml` where "api-gateway" is the name of the hpa (that was inherited from the scaled deployment).

A few things can be deleted from the file, like `annotations`, `creationTimestamp`, `resourceVersion`, `uid` and `selfLink`. We should keep the entire `spec` block and remove `status` (these are current values caught on the fly - we don't need that).

***IMPORTANT***: you need to have metrics enabled on minikube to make it work.


## Testing Autoscaling

When testing autoscaling we can check events running `kubectl describe hpa <name of autoscaling>`.
For scaling down cases, it waits for 5 minutes while observing the cluster to make sure it is safe to scale down. It kicks in after being conservative, avoiding the system to enter in caotic mode scaling up and down several times.

It is safe to assume that K8s waits for consistent metrics under the threshold before scaling down any resource.
