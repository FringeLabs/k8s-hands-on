# Quality of Service and Eviction

In K8s the scheduler has the role to decide which node the pod should be placed. It is a critical component of K8s with the main responsibility that a node doesn't get overloaded.

On these notes we will understand sometimes a scheduler should evict a pod.

## Understanding the scheduler

Imagine we have a node in a cluster that has total memory of 900MB and it's all available with no pods running on it.
After deploying a pod in the cluster that requests 500Mb and also limited to the same value, helping the scheduler to taking the decision when depoying to this node. The node has plenty of memory after all and fits well in this node.

Given this scenario, now the cluster has 400Mb allocatable memory. For this example, elt's assum from those 500mb the pod is using 450Mb, leaving other 450Mb free for usage. Everything is working ok and running smoothly so far.

Let's assume now we are going to deploy Pod 2 requesting 300Mb of memory (with no limit). that means it should have enough memory to run this pod on that node, which means we should have 800Mb (500 + 300) being requested from 900Mb. For this example, let's assume from 300 it is using 100Mb. At this moment, our node has 350Mb of allocatable memory.

Now, let's come with Pode 3 that has no Request or Limit, leaving no clues to the scheduler about the best strategy to allocate this pod, so it puts in the same node. It doesn't update the Requested / Allocatable memory in the scheduler "mapping table".

When  POd 3 starts it consumes 200Mb, which in practice leaves 150Mb of memory free in the cluster. The problem we don't know how things are going to change in the future while the pods re running:

* with pod 1 we know we neever exceed 500Mb. If this pod tries to do that, it will be automatically evicted and terminated and then re-scheduled (restared in the same or even in another node);
* we can't say the same on Pod 2 or Pod 3, because they have no limits. If any of them decides to use 500Mb the limit mechanism is not going to help here;
* Request and Limits have a huge impact in the way scheduler works;

To make the understanding eaiser from now, let's lable the pods:
* Pod 1 = "Nice pod"
* Pod 2 = "Quite Nice pod"
* Pod 3 = "Rude pod"

When we labeled them it is pretty clear our understanding about their behaviour in terms of resource allocation. Turns out K8s scheduler also labels the pod as well, but with a different strategy that is called QoS (Quality of Service):
* Pod 1 = "QoS: Guaranteed"
* Pod 2 = "QoS: Burstable"
* Pod 3 = "QoS: BestEffort"

These are just labels used by scheduler when deploying the pods and they will be very importante when running out of resources.

## QoS Labels

Let's see how all the Quality Of Service classes behave in the system. Before that, let's see how to prove these labels exist in the system:
* Run `kubectl describe pod <name of the pod>`
* Check the property `QoS class`

## Evictions

The labels exist to help the scheduler to evict the pod if the node is under pressure.
* The "Guaranteed" pod is not something scheduler needs to worry about, because the request and limit values are going to prevent it to harm the system in case it gets greedy;
* The "Burstable" is not that accurante. The "Request" is a hint, but there is no limit in our example. That's why it is called this way. If it goes above the requested value, it can burst. Assuming the node gets the memory overcommited at some point, the scheduler wil need to evict one or more pods to make sure the node will be able to keep running smoothly;
* In theory, "BestEffort" is not that different. The trick in practice is scheduler will start to evict them first in case of running out of resources because there is no way to predict anything about them. After that, scheduler will evict Burstable pods and the last to be avicted are the Guaranteed - those nodes are going to be re-located in other nodes;
* If there is no node available to rescheudle the pode and we are using cluster autoscaling, another node will be automatically created;
* The trick is when a Guaranteed pod starts to use more memory w/o going over limit and the over gets overcommited again. So the Burstable is the one to be re-scheduled instead of the Guaranteed one;
  
## Pod Priorities

I haven't been using this in prod so far because:
1) it is a relative new feature (not stable before 1.14);
2) please check the notes below;

K8s uses the labels to determine Pod priorities, which can be changed with the Priorities feature. That means you can set a pod priority in the yaml file. It's like set a number like `Priority: 0`. If you deploy a new pod with `Priority: 10` in a node and there is no enough resources, the first pod is going to be evicted to open room for the second pod. That means all the logic to do it for new pods doesn't consider the QoS, leaving QoS as a concept only for pods that ar ealready running. So, addijng new pods may be a problem and justify for me not using this as a feature in production.

**Note**: THe yaml file I'm talking about is not the `workloads`, but a `PrioerityClass` taht has to be defined according with the docs. There is also an interaction of Pod priority and QoS according withy the docs and how it affects the schedule. As it is mentioned, QoS is not used for new pods but K8s uses a more complex logic that will consider both when having a kubectl out of resource scenario, ranking first by wheter or not their usage of the starved resource exceeds requests, then by priority, and then by the consumptiuon of the starved compute resource relative to the Pods scheduling requests.

 `Kubectl out-of-resource` eviction does not evict Pods whose usage does not exceed their requests. If a Pod with lower priority is not exceeding its requests, it won't be evicted. Another Pod with higher priority that exceeds its requests may be evicted.

 That means it's going with QoS first and then Priority right after, leaving another level of QoS for the end.

 *** In my opinion, in a microservice architecture every microservice should have a equal priority. They should all be designe to be evictable, bringing a robust system in place. ***





