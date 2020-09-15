## How to go through Section 5
Instructions about how to play around with it.
At this point I assum the pod was deployed on Kubernetes cluster:
```
kubectl apply -f first-pod.yaml
```
Assuming everything went well with yaml file, we can check if the pod is running properly in the cluster:
```
kubectl get all
```

After those steps we can start to play with it.

----

### Step 1 - Find Minikube IP

Run the following command:
```
minikube ip
```
It should return minikube ip adddress. You will see you can't get anything through the browser because pods are not intentionally accessible outside (for that, you would need to think about services, which is another story).

### Step 2 - Get pod details

The following command helps on finding pod details:
```
kubectl describe pod webapp
```
The events section helps to understand what happened with the pods.

### Step 3 - Executing commands in the pod

Once the pod is not exposed, we can at least execute commands inside the pod:
```
kubectl -it exec webapp sh
```
This command opens an interative shell to run commands inside the pod, like:

```
wget localhost:80
```

That should save index.html file containing the app that is served.