# Readiness and Liveness Probes

Here we are going to talk about the concept of readiness / liveness probes.


## Why readiness probes are needed

The HPA we saw previously has one problem: in our example the client web app sends http requests to the API Gateway Service that redirects it to the API Gateway Pod. If the Gateway Pod reaches some a threshold we previouslyn defined, other instances of the pod are going to be created, resulting in multiple instances of Gateway Pod. THe problem is that knowing  every pod is going to start a container, that doesn't mean the software inside the container is ready to receive requests or if it is even working. It depends of framework and language used to build the app, as many other factors.

On the example, we are using a Spring Boot that disregarding the complexity takes between 20-30 seconds until it is ready to do the job. During this process, the rounding-robing algorithm will send requests to pods that are not ready, hanging them and causing timeouts or even errors to the client app.

## Applying Liveness and Readiness Probes

The secret is inform K8s a way to test if the service is ready to go or not:
```
spec:
    containers:
    - name: api-gateway
        image: richardchesterwood/k8s-fleetman-api-gateway:performance
        readinessProbe:
            httpGet:
                path: /
                port: 8080
        env: ...
        resources:              
            limits:
                memory: 200Mi
                cpu: 200m
```
There are a few parameters we can use to tune the probes, like delays, timeouts and others. Using `kubectl describe po` you can see the probe events of a given pod to check what is going on.

### Liveness probe

Liveness is different from Readiness probe. THe liveness probe keeps running during the entire lifetime if your pod. In case it fails based on given set of rules and thresholds, it is going to assume the app has issues and it's going to restart the container on that pod.

Please check the documentation for the details.