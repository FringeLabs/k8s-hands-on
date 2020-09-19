# EKS - Running Kubernetes on the AWS Cloud

The user interface is quite tricky. So, these instructions are here to help on creating using command line tool using eksctl, which is way more friendly. This tool is officially recognized by Amazon as an option to manage K8s clusters.

Reference: https://github.com/weaveworks/eksctl

As done for KOPS, we can use the "bootstrap" machine for managing clusters. It is much easier running from a bootstrap machine than from your local.

## Install eksctl

1. ssh to a bootstrap machine (create one if you don't have any);
2. Install eksctl (instructions on the reference link); 
3. eksctl will use the standard aws cli interface. So you need to have it in your bootstrap machine;
4. Update AWS cli if you have to (EKS needs 1.18.x or above - at this time);
5. Configure AWS credentials in the CLI if you didn't so far;

## Configure AWS Credentials

On this tutorial I'm re-using bootstrap machine and kops groups / user.

1. Create an user and add to a group with `eks:*` policy / permissions. Just ensure it has:
   1. EC2 full access;
   2. IAM Full access;
   3. CloudFormation Full Access;
2. Create an inline policy in the group as stated on `install_eks.txt` file on course material folder (chapter 13);
3.  That EKS policy is required because so far there is no EKS permission on AWS predefined policy list;
4.  Test your permissions: `aws eks list-clusters`;
5.  Remember you need to have kubectl installed as well (and it has to match with the cluster k8s version - they have to be compatible);

## Start the cluster

Assuming that you already have kubectl, eksctl and aws cli installed and configured:
* Run: `eksctl create cluster --name fleetman --nodes-min=3`
* It will create a CloudFormation stack with all resources of the EKS cluster - it should take around 20 minutes or so;
* 