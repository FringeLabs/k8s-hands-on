# Logging a Kubernetes Cluster

A quick walkthrough about log inspection.

## Introducing the ELK / ElasticStack (and Manual log inspection)

* when working locally with minikube manual inspection makes all the sense;
* in production, if a pod is restarted the logs are gone - that is where ELK stack joins the party;
* you can go with fluentd or Logstash. On this example we will sue fluentd (that also supports several data sources);
* Elastic is going to store the log data providing search capabilities;
* Kibana is going to give visualization capabilities for Elastic data;
* There are docker contaienrs for all of them. However, we will need a fluentd on every node of the cluster so it can discover the logs;
* we could also request an elasticearch outside of the cluster and install Libana outside the cluster as any web app;
* we are going to do the quick and dirty approach: deploy directly on K8s cluster:
  * we are going to put one FluentD on each of the 3 nodes;
  * 2 elastic instances on 2 nodes (one is replicaset) to avoid losing data;
  * 1 node with Kibana in 1 node (no need to replicate);
  * the downside is as we add more pods to the existing cluster we do need to be mindful we will be potentially overloading these nodes;

## Installing the Stack to Kubernetes

For this section the FluentD config can be found at: https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/fluentd-elasticsearch

The required config files are combined already and attached to the yaml to make things easier. Some notes:
* `DaemonSet`: acts like a ReplicaSet, but you don't need to specify the amount of replicase because it is going to spin up one instance per node;
* `StatefulSet`: it will make sure the pods created will have "stable" names instead of random names (like elasticsearch-logging-0 and elasticsearch-logging-1);
* There is a `volumeClaimTemplate` for keeping Elastic data store so we don't lose data when restarting the cluster;
* Copy files to bootstrap machine using **SCP**;
* Apply changes with `kubectl`;
* You will notice we don't list the new pods using `kubectl get po`. That is because the ELK pods are on a different namespace and the others ar eon default namespace. A test that can be done is running `kubectl get po -n kube-system`, which where the new pods were placed;
* Find Kibana url: `kubectl get svc -n kube-system`;
* Access the url: `<domain>:5601`.

## Kibana - first look

How to setup Kibana to work with Elastic and FluentD:
* Go to `Management` and click on `Index Patterns;
* Fill pattern field as "logstash*" and click on `Next step`;
* Pick `@timestamp` field in the dropdown list and click on "Create Index pattern". It will display a list of fields found on log messages received by fluentd so far;
* Click on `Discover` menu and see log messages;
* Notice it contains important references like containers name and host where the log message came from;

## Setting Filters and Refreshes

* If you click on the `kubernetes.namespace_name` on the left it will expand and show the source of the messages from each namespace in K8S;
* Clicking on `add a filter` on the top allows to build filters based on the fields on the left;
* We can create a filter `kubernetes.namespace_name is default` to ignore messages from ELK stack and FluentD;
  
## Demo: analysing a system failure

Let's create some chaos and see what happens:
1. list pods: `kubectl get po`;
2. simulate permanate crash changing number of replicat do zero for "queue":
   1. change workloads file;
   2. edit replicas to 0;
   3. apply changes with kubectl;
3. see thet after some time the number of log messages increased a lot;
4. We can see that quite often we have `Queue unavailable - backing off`. Something is happening;

## Kibana Dashboards

Other useful features:
* save search filters: `Queue Problems`
* You can create vizualization with `Gauge`and pick `Queue Problems`;
* We can change metric ranges on `Options`;
* If we rollback the replica change, in 15 minutes or so we are going to see the number decrease in the gauge chart (or u se relative timeframe instead of `Last 15 minutes`);
* You can save the visialization as `Queue Emergency Gauge`;
* After, you can create a dashboard and add it - you can save if you want to;
* You can play with `Line Visualization` for counting. Pick `Date Histogram` as X-Axis.
* Save it as `Queue error by time`;
* Add it to the dashboard - save the dashboard as `Queue Monitoring`;
  
