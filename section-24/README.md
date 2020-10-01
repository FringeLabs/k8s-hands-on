# Request and Limits

## Memory Requests

We can define how much memory the container will need to run. Like:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: queue
spec:
  selector:
    matchLabels:
      app: queue
  replicas: 1
  template: # template for the pods
    metadata:
      labels:
        app: queue
    spec:
      containers:
      - name: queue
        image: richardchesterwood/k8s-fleetman-queue:release2
        resources:
            requests:
                memory: 300Mi       # 1Mi = 1024Ki     1Ki = 1024 bytes
                                    # 1M = 1000K        1K = 1000 bytes
```

On the example above the service will need that amount of memory available in the cluster.
It is recommended to always specify resources like that in all services in PRODUCTION. That helps cluster manager to orchestrate better the resources and which node will host your pod.

For instance: if we have a node with 1GB of memory we cannot have more than 3 instances of this pod running. The 4th pod will fail when provisioning because there will be no memory available.

A good way to know how much resources you have is running: `kubectl describe node <name of the node>`. This brings all the information about the node.

## CPU Requests

CPU concept is the same than 1 vCpu on AWS. We can set as we do for memory:
```
    spec:
      containers:
      - name: queue
        image: richardchesterwood/k8s-fleetman-queue:release2
        resources:
            requests:
                memory: 300Mi
                cpu: 100m               # 0.1 CPU or 100 milicores
```
It means the pod needs 1 CPU to work comfortably. It doesn't mean the pod is going to use full CPU. It will not affect the runtime at all. It also doesn't restrict the usage. FOr that we have limits.

## Memory and CPU Limits

To avoid running out of resources if a pod gets mad, you need to limit them in the yaml file:
```
    spec:
      containers:
      - name: queue
        image: richardchesterwood/k8s-fleetman-queue:release2
        resources:
            requests:
                memory: 300Mi
                cpu: 100m              
            limits:
                memory: 500Mi
                cpu: 200m
```

### Memory and CPU work very differently:

* Memory Limits: if the actual MEMORY usage of the container run time exceeds the limit for any reason the CONTAINER will be killed (the pod is not killed and stil will be alive). That means POD will restart the container;

* CPU Limits: if the CPU usage of the container at run time exceeds the limit the containes won't be killed, but the CPU will be "clamped". It means the CPU won't be allowed to go over the specified. The container will continue to run. It can also be called as "throttling".

