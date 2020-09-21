# KOPS - Running k8s on the AWS Cloud
Reference (github): kubernetes/kops
Guide to install Kops: https://kops.sigs.k8s.io/getting_started/install/


There is no need to install Kops using the guide. This document will cover how to do that.

## Introducing Kops - Kubernetes Operations:

The steps to have KOPs working on EC2. The entire guide can be found at: https://github.com/kubernetes/kops/blob/master/docs/getting_started/aws.md


### EC2 infrastructure setup

On AWS Console EC2 Running Instances List:
1. Click on Launch instance;
2. Pick Amazon Linux 2 AMI and pick the lowest element in the list (t2.nano or so);
3. Leave the defaults until get tags page;
4. When getting to the Tags form, add a tag `Name=bootstrap`;
5. Go next until the end and click on "Review and Launch";
6. Click on "Launch";
7. Create a keypar with the name you like;
8. Download the keypair;
9. Click on "Launch Instance" and go back to the EC2 list;
10. When having the instance ready, copy the public IP address of the machine;
    
### Kops Setup

Once you have the IP and the pem file:
1. Open a terminal;
2. Go to the folder you have the pem file;
3. Connect to EC2: `ssh -i <pem file> ec2-user@<public ip address>`
4. If you have permission issues with the key: `chmod go-rwx <pem file>`
5. Install Kops on Linux: https://github.com/kubernetes/kops/blob/master/docs/getting_started/install.md
6. Install kubectl: https://kubernetes.io/docs/tasks/tools/install-kubectl/
7. Create user using CLI or IAM in the console:
   1. Go to IAM and click on Groups;
   2. Create a group called "kops";
   3. Add the permissions requested in the docs;
   4. Create group;
   5. Go to Users page and click on "add user";
   6. User name - type `kops` and click on `Programmatic access`;
   7. Select the group "kops" and click on "Next";
   8. Skip "tags" and `Create User`;
   9. Save the keys ***carefully***;
8. Configure aws: `aws configure`;
9. Test that: `aws iam list-users`;
10. Export the aws env vars (check the tutorial on GH);
11. Skip `Configuring DNS` section - this is for very old kops versions. Recent versions use "gossip dns". Go straight to `Cluster State Storage`;

### Configuring your first Cluster

We need to setup a S3 bucket where kops will store its working data:

1. Create an s3 bucket called `kops-lesson-state-storage`;
2. Skip encryption and sharing sections for learning purposes;
3. Set cluster name: `export NAME=fleetman.k8s.local`;
4. Set store state: `export KOPS_STATE_STORE=s3://kops-lesson-state-storage`;
5. Create cluster configuration (follow GH doc);
   1. this is important because we can put each node on a different availability zone (data center);
   2. you can find availability zones running: `aws ec2 describe-availability-zones --region us-west-2`;
   3. create cluster config files for the cluster in all regions: `kops create cluster --zones=us-west-2a,us-west-2b,us-west-2c,us-west-2d ${NAME}`;
6. Need to fix the SSH presented in terminal, otherwise, nothing is going to work: `ssh-keygen -b 2048 -t rsa -f ~/.ssh/id_rsa` - keep password empty;
7. Create secrets: `kops create secret --name ${NAME} sshpublickey admin -i ~/.ssh/id_rsa.pub`;
8. Skip cluster customization;
   1. If you wanna play: `export EDITOR=nano`;
   2. Run: `kops edit cluster ${NAME}`;
   3. See the config file. You can find all the network configuration, regions, etc...;
9. Edit instance groups: `kops edit ig nodes --name ${NAME}`
   1.  maxSize=5
   2.  minSize=3
   3.  confirm: `kops get ig --name ${NAME}`
10. You can change the master type: `kops edit ig master-us-west-2a --name ${NAME}`


## Running the Cluster

Time to create the cluster: `kops update cluster ${NAME} --yes`. This is going to create a lot of resources in the account. Please delete after, when not using.

You always can see how it goes: `kops validate cluster ${NAME}`. You can keep it running until it stops to return "error listing nodes".

Need to wait until everything is deployed.

If the nodes have not joined to the cluster yet, you still need to wait (keep running `kops validate cluster`). It takes some time until everything is done,


Moving forward, check the created loadbalancer. You will see he points to the master node and that means all the commands you run through kubectl are going to pass through the load balancer. If you scale in the future adding more master nodes, they will be attached to that load balancer.

You also will notice the ASG will "protect" your nodes, keeping a minimum of nodes running for your infrastructure.

***TEST***: terminate one worker node to simulate a DC crash. See what happens refreshing the screen. Run `kubectl get nodes` for details.