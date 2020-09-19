# Deleting the cluster in KOPS

Instruction about deleting / restarting the cluster in Kops on AWS.

## Deleting the cluster

Do not delete manually because it has ASG and other protection mechanisms that may re-create resources, doing a mess. The proper wai to shutdown a k8s cluster in kops is:
1. Run: `kops delete cluster --name ${NAME} --yes`
2. Do not touch anything! let the process finishes;
3. Terminate bootstrap instance (or just stop if you intend to use it later);
4. If you removed bootstrap instance, feel free to destroy bootstrap EBS volume;
5. Navigate on AWS to remove resourvces left (IAM groups, users, policies);


## Restarting the Cluster

If you destroy the cluster but you kept bootstrap machine:
1. start bootstrap machine;
2. pick the public ip address of bootstrap machine;
3. connect to the machine using ssh;
4. run `history` command and:
   1. ensure you export again the env vars `NAME` and `KOPS_STATE_STORE`;
   2. run from history `kops create cluster` with all availability zones;
   3. run `kops edit ig` command suggested at the end of previous command to edit maxSize and minSize for the cluster;
   4. run `kops update cluster` from history;
   5. at the end, run the suggested command: `kops validate cluster`
   6. waits until it is done and cluster is ready;
5. Now, you are just good to go and deploy your app using `kubectl apply -f .`;
6. Ensure your webapp is exposed in the loadbalance running `kubectl get all` and check if there is an external id / address;
7. It takes a few minutes until the app is available on LoadBalancer value returned by `kubectl describe <service>`

