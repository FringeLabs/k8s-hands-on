# Operating your Cluster

This chapter is suitable for KOPS or EKS. Just a few commands are exclusive from KOPS.

## Provisioning SSD drives with StorageClass

1. create the file `storage-aws.yaml` or copy from the course folder;
2. copy to the server: `scp -i <pem file> ./section-16/storage-aws.yaml ec2-user@<BOOTSTRAP MACHINE IP>:~`;
3. ssh into the machine: `ssh -i <pem file> ec2-user@<public ip>`;
4. apply changes: `kubectl apply -f storage-aws.yaml`;
5. check if it is "running": `kubectl get pvc` -  It will be ***pending*** until it gets ***bound***;
6. the same with the volume: `kubectl get pv` - you should also be able to see it on "EBS" page inside AWS console;
7. Pay attention on "RECLAIM POLICY" that is set as "Delete" which is about deta deletion when the cluster is gone. You also should notice on AWS that created volume is "available". It is set on that way because this is not attached to anything;

## Deploying the Fleetman Workload to Kubernetes

1. copy mongo stack to bootstrap machine: `scp -i <pem file> ./section-12/mongo-stack.yaml  ec2-user@<public ip>:~`;
2. ssh into bootstrap machine (like in the session above);
3. apply changes from mongo stack file with `kubectl`;
4. note no change in the file has to be made;
5. check if container is there: `kubectl get all`;
6. check for problems: `kubectl describe <pod name>` - there should be no errors on Events section of the output command;
7. check for messages in the log to see if mongo is ok: `kubectl logs mongodb-65784d9f9d-q45mk`;
8. on EBS page (AWS) the volume now should be "in use";
9. copy `workloads.yaml` and `services.yaml` to AWS bootstrap machine;
10. ssh into the machine again;

### Small change on service files
1. NodePort is only intended for local development. We want port 80 for production for `fleetman-webapp`;
2. Open `services.yaml` on NANO or VIM;
3. Let's **DELETE** the `nodePort` line;
4. Change the `type` property value to **LoadBalancer**;
5. This can't be done or local but it is a change intended for production;
6. Remove `nodePort` from **queue** service as well and change the `type` to `ClustserIP` (an make sure the port is set as `8161`);
7. Need to do the same from **queue** change to **api-gateway**;


### Applying changes 

* Save the file and apply all changes: `kubectl apply -f .`;
* Check mongo logs again to see if app has been connected to mongo;
* With `kubectl get all` you can chekc status and get the `EXTERNAL-IP` with the link for **fleetman-webapp` and access through the browser;
* For Kops you should have a loadbalancer called `api-<clustername>-...` that is the LB for the master node. All commands go through this one. The other loadbalancers created are for the app, from wher eyou can also get the **DNS NAME** for accessing the application (it may take up to 5 minutes to get things in place and having domain working).
  
## Setting up a real Domain Name

IF you have a domain name you can set that up to yoru LoadBalancer so you don't expose your app with that "ugly" random domain created by Kops on AWS:
* You can register domain name on Route53;
* Then, it is just a matter to add an "alias" to the loadbalancer and that's it.

## Surviving Node Failure

Even in the event of a Node(or Availability Zone) failure, the web site must be accessible. It doesn't matter if reports from vehicles stop coming in, as long as service is restored withing a few minutes:

* To get more info: `kubectl get pods -o wide`;
* Let's simulate the webapp node power failure:
  * grab webapp pod "NODE" name and **TERMINATE** the node that hosts `webapp` on AWS;
  * this is going to make the webapp hang;
* Run `get pods` again and you will see a new webapp pod in place (it may take a long pause until it is back again - in my case it was less than a minute of outage until it redeployed the pod to an existing node). IOn the meantime, the new AWS node is still being created;
* A new node on AWS was created due the ASG configurations requiring at least 3 instances up and running - please check **ACTIVITY HISTORY**. We can check nodes with `kubectl get nodes`;
* Notice that K8S doesn't now if the pod should be moved to another node after to "keep the balance". So, no rebalance happens after this recovery scenario;

## Replicating Pods in Kubernetes

We failed on the requirement: ***Evem in the event of a Node(or Availability Zone) failure, the web site must be accessible***.
The secret to "fix" that is to edit the `workloads.yaml` and update the replicas from `1` to `2`. Then, let's apply the changes from that file.

Notice after running `kubectl get pods -o wide` a new pod was created on the new fresh node. That helps to match the original requirement. Now, time to kill another node (the old one hosting the webapp). After that, the webapp still will be available, however, you will notice no machine is being reported. That is because the other nodes were also hosted on that node.

Based on original requirements, that is fine, but it is also something to keep in mind when hosting your solution on K8s.

### Caveats
* queue pod is stateful and that is why it can't be replicated - ofor that is should be stateless;
* the same issue with mongo database - it is stateful and can eb replicated;
* a good solution is to migrate those services to managed services in the cloud (Amazon MQ, Atlas, etc);
* the trade-off of it is relying in another 3rd-party;
* You can rpelicate MongoDB and RabbitMQ, but the trade-off still persists;

### What happens if the master node fails?
* 