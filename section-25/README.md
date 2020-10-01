# Metrics Profiling in K8s

In order to have a good balance / right values on Requests and Limits, the ability to do profiling is very helpful on kubernetes.

## Enabling metrics server

The command to do that is very simple: `kubectl top pod`. You can also use `kubectl top node`. You will see those commands don't work out of the box unless you enable the metric server which is quite hard to enable in the cloud. But in minikube is quite simple. It is an add-on to minikube.

* Run `minikube addons list`. You should find `mtrics-server`;
* To enable it: `minikube addons enable metrics-server`;
* It requires some time (minutes) to collect the metrics. After a least 1 minute you should be able to rund the commands;
* You should be able to run at least this command: `kubectl get all -n kube-system`;
* Notice that after `kubectl apply -f .` you also will need to wait until you have the pod metrics available;
* As you add more podes you can see the values changing for the node;
  
## Viewing matrics on the Dashboard

There is another minikube addon that can be quite handy: `dashboard`.
Let's enable it: `minikube addons enable dashboard`. Then, run `minikube dashboard`. You can change deployments, delete or make new ones and do several operations we usually do through `kubectl`.

We can even open logs through the UI as scale replicas. A lot of operations can be easier through the dashboard. IN prod there are better solutions but it can be used in local with no issues.

***Note***: currently there is a problem with the dashboard on displaying graphics and charts. There is an open ticket on github from people asking to integrate it with Metrics API.

## Tuning Java Spring Boot Apps - Heap Restriction

Notice we didn' specify any java memory limitation.

### Java 7, 8, 9

If no `-Xmx` is specified, java will use up to 1/4 of the host RAM as its maximum heap. This means regardless of the size of your Node, you might only get 4 containers running. From Java 10 onwards, Java recognizes its running inside a container and set the defaults to more reasonable values.  FOr more discussions about the ideal heap size: https://spring.io/blog/2015/12/10/spring-boot-memory-performance 

For now 50MB should be enough for the examples we have on this course.
