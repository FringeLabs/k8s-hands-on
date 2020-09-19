# Deleting the Cluster in EKS
1. eksctl dekete cluster `fleetman`;
2. Check if the loadbalancer is gone;
3. Check if the instances are gone;
4. It may take 5 minutes (or more) to delete all the stuff. Be patient;
5. The volume has to be deleted manually (those from your pods / containers and so one). EKSCTL usually leaves the disks on EBS volumes. They cost $0.1 per GB. It's not much, but that is still something you will be paying for;
      
After that, check if your AWS account has any resource left.