# Running K8s on the AWS Cloud

**WARNING**: Everytime we stop working, the advice is to drop the cluster so I don't pay for resources after testing them. As an example, after running for 24 hours the example cluster was around 8-10 US dollars.

## Managing a Cluster in the Cloud

The plan is to have a master node and 3 worker nodes, so the master can decide where the pod shoould be deployed among the nodes.

### AWS

* KOPS support: it is older option offered on AWS. It is the easiest way to get your production grade high-available k8s clusters (according with k8s documentation);
* EKS: the big deal os AWS is that is part of AWS itself. It is built-in AWS and they claim to be the most safe way to run k8s on AWS;

There is no right or wrong here, having pros and cons. More and more people ar eusing EKS and migrating from KOPs (on AWS). Maybe the massive difference is the way we deal with the **master node*. In KOPS we have manage the master node by ourselves. When working with EKS, you don't see the master node! It will run in background and restricted to the AWS.

The thing is from time to time an administrator may have to terminate the master node. Even whe  it crashes, worker nodes keep working fine. Temporarially you can use kubectl but as soon as you have a new master node in KOPS, things are back and you can manage it again. With EKS, once you don't see it, you can't terminate it and you can't do anything with it. However, the advantage is you don't have to manage it and be worried about it. AWS will take care of it for you.

### EKS vs KOPS - Which to Choose?

The difference between two are quite marginal and your project will succeed disregarding your choice.
Here goes my opinion about it:

#### KOPS - Pros:
* it is well respected and heavily used (a long history - stable for a long time);
* it is easy to use (maybe because I know it better?!);
  
#### KOPS - Cons:
* you are responsible for managing the Master (like alerts for high CPU consumption);
* By default you only get a single master - if it fails, you won't be able to manage your cluster until you have a new one;
* it feels like more worj than EKS (the work is mor elike the same, but I usually get this feedback);

#### EKS - Pros:
* It has gained a lot of ground recently (and not it is winning the race), having more market share;
* Using `eksctl` it is quite simple to use;
* No management of the Master required;

#### EKS - Cons:
* It needs a third party tool to make it usable (eksctl) - it used to be a hell to manage it before having this tool available;
* The GUI is (by mid july so far) very poor - this is not a showstopper but AWS has not been improving the user interface. Even that you use CLI most of the times, it is nice to have a good GUI available;
* Might feel like you are tied into AWS forever (of course you can move from there, like to KOPS or Google)

Maybe EKS wins because it is gaining a lot of traction (even his development quite slow).

***Bonus***: EKS integrates perfectly with AWS Fargate, bringing serverless capabilities for your cluster. That is a H-U-G-E advantage, not having to spend time managing underlying infrastructure like we would do with EC2.

### Pricing Differences - EKS vs Kops (by mid 2020)

Another aspect that is important to analyse is the cost difference between those choices.

In both choices, you will have to add the cost for an additional node (the master node) and loadbalancer for KOPS. On EKS, they exist, but they are "hidden". But it is not just that. EKS also provides you "control pane" with it.

#### Kops - Running Cost
* Depends on the Node Type you choose;
* For a m3.medium (about $600 / year) - a good reasonable medium cluster size (something about $2 for 2 days of course);
* But you also need a LoadBalancer at $200 / year;
* About $800/y for a single master control pane;
* All of it + the worker nodes;

#### EKS - Running Cost
* A set fee for the entire control pane;
* In 2019 it used to be way more expensive than Kops (like twice the price) $0.10 per hour of control pane per cluster (mid 2020);
* About $876 / year (mid 2020) - insignificant difference for most projects;
* This gives multiple masters - and this is a nice thing to have and there is no additional cost for it. With multiple masters Kops could cost 3 times more;
* All of it + the worker nodes;







