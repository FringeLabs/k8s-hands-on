# Introduction to ReplicaSets

It is rare to use pods in production systems, managing like in the previous examples. IN production we usually manage deployments or replicasets (which is going to be covered on this section).

In reality, pods have a short period of live. Pods are very basic and disposable objects in K8s.
If a node fails or something else happens, pods are going to die. If they consume too many resources (CPU, memory) K8s is going to kill them. That means they can live shortly in real fully systems.

If you decide to manage pods directly (writing the file and deploying the pod manually with `kubectl apply`) you become responsible for keeping the pod alive. If it dies, they are not going to come back unless you bring them back.

That can be simulated killing the pod manually:
```
kubectl delete po webapp-release-0-5
```

Right after, if you try to access `<minikube ip>:300080` the app would not load. Disregarding the fact the service still exists, the pod is not coming back.

## ReplicaSets

An "extra piee of configuration" we provide to k8s to specify how many instances of the pod we want the system to make sure it will keep running. If the pod dies for any reason, k8s will spring another one.

It is not very intuitive to work with ReplicaSets at the beginning, but the idea of having "groups" and no need to write a separate file for a pod and a RS helps to clarify the concepts across the time.

With RS pods need no name in the files because they will be automatically named by replicaSets. You can name replicaSets as you would name your pods. Some people use the `-replicasets` sufix and that should be ok as well.

The replicaSet also requires selectors, which is very common to be the same keys we have in pod description.

### Applying changes

As usual:

```
kubectl apply -f ./section-08/pods.yaml
```
**NOTE**: Do some cleanup may be good. We can run `kubectl delete pods --all` or `kubectl delete po --all` to cleanup all the pods.

### Final notes

If you notice the pod names are made up. That is why we name them shortly and treat them like cattle. The name doesn't matter unless you are troubleshooting and we can avoid complex names if we name our ReplicaSets in a concise way. Also, `kubectl` allows to use `rs` as a shorthand notation so you don't neet to type `replicasets`, as it does for pods and services (`po` and `svc` respectively).

