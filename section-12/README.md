# Persistence and Volumes

Currently position tracker stores everything in memory. This means it will crash eventually.
We need to store the telemetry history to a storage.

This is going to be upgraded to save the data into the database (mongodb).

## Upgrading to a Mongo Pod (History Database)

To create mongo stack: `kubectl apply -f mongo-stack.yaml`.
We will see we may have to tweak the storage section in mongo file unless the storage volume was created prior the command execution (which was what I did).

Turns out after running it, that overloaded my Minikube and things started to run sluggish.
It is important to remember that just create the mongo doesn't do anything. I had to upgrade position-tracker to release 3 and apply the changes to ugprade the service in a way it connects to the mongo database.

## Expanding the Minikube VM

If things get sluggish, you need to go to your Hypervisor and upgrade the VM memory.

Since hyperkit is just "a toolkit for embedding hypervisor capabilities in your application." it doesn't provide tools to manage the virtual machines directly.

The way you have to see all the virtual machines is:
```
ps -Af | grep hyperkit
```

In case you'll see in the previous command the "-l" flag with a tty available like this "-l com1,autopty=/Users/youruser/.minikube/machines/minikube/tty" you'll probably be able to open a serial tty like in How do I see a list of all minikube clusters running in Docker on my mac?:

```
$ ps -Af | grep hyperkit
0 35982     1   0  2:50PM ttys000    3:27.65 /usr/local/bin/hyperkit -A -u -F /Users/youruser/.minikube/machines/minikube/hyperkit.pid -c 2 -m 4000M -s 0:0,hostbridge -s 31,lpc -s 1:0,virtio-net -U 39c5590a-cdac-11ea-b300-acde48001122 -s 2:0,virtio-blk,/Users/youruser/.minikube/machines/minikube/minikube.rawdisk -s 3,ahci-cd,/Users/youruser/.minikube/machines/minikube/boot2docker.iso -s 4,virtio-rnd -l com1,autopty=/Users/youruser/.minikube/machines/minikube/tty,log=/Users/youruser/.minikube/machines/minikube/console-ring -f kexec,/Users/youruser/.minikube/machines/minikube/bzimage,/Users/youruser/.minikube/machines/minikube/initrd,earlyprintk=serial loglevel=3 console=ttyS0 console=tty0 noembed nomodeset norestore waitusb=10 systemd.legacy_systemd_cgroup_controller=yes random.trust_cpu=on hw_rng_model=virtio base host=minikube
```

They wayit worked for me was running minikube dashboard: `minikube dashboard`. It was way more interesting and doesn't have all the drawbacks Virtualbox has. On VBox you have to open the vbox client and upgrade the memory by yourself. Then, it will be just a matter to restart Minikube.

Later I will write about how to do professional monitoring of a cluster.

## Volume mounts

So far, all the data was inside a directory inside Mongo container. Which means if the POD dies (or the container) the data goes away as well. To avoid that problem, we need a persistent volume.

Personally, I think Persistent Volumes are quite complicated in the k8s documentation. The first step is to get the data stored in a folder outisde the container. That is fine because I'm still running locally. However, when moving to the cloud (like AWS) we have EBS and we can do K8s and his containers to use EBS to save and read data (which is way more complicated than my local testing).

The good part is that K8s make things transparent enabling the paltform migration w/o much headache.

**TIP**: The `mountPath` property on workloads file maps the folder inside the container where mongo stores data.

## Volumes

We usually create a separate file for describing volumes, specially ebcause the volume is very specific and has a wide set of properties depending where you are storing data (AWS, GCP, Azure, Local). So, the trick is declare the volume inside the "workload" (mongo stack on this case) file and then describe this volume on `storage.yaml` file. With that, in the future, you just need another file for creating storage on AWS, making things completely transparent to your workloads file (and also doesn't touch on your pods when that changes, avoiding containers to be re-created and so on).

Please refer to property `hostPath` on k8s documentation for more information (in a nutshell, this is an "escape" to save in the local host system folder).

## PersistentVolumeClaims

Through PersistentVolumeClaims we allow a storage to be claimed by a pod/container that is described in somewhere else (like another file). On this example the storage definition can be found at `storage.yaml` file.

That means when moving to AWS there is no need to change the workload. We just need to define a storage definition.
Please chekc the file and the comments in the file for more details.

**TIP**: k8s documentation contains very important and detailed info about how k8s behaves for each storage type regarding access modes (specified on persistent volume claim section in the file). For further info: https://kubernetes.io/docs/concepts/storage/persistent-volumes/ .

What is important to remember that we express 2 different needs when working with volumes:
1. a definition of what we need / want;
2. a definition for storages we have available;

In the file, the "claim" says I want a 20GB disk volume, which means it has to match with a sotrage that has **at least** 20GB, which is the last block in storage descriptor file. That helps on administration of the resources, specially on hybrid / multi-cloud scenarios.

## StorageClasses and Binding

As the title refers to, the property ***storageClassName*** is responsible for linking the "claim" to the volumes available. IT works as a label to mack/link both resources described into `storage.yaml` file.

To see that working it is just a matter of run the command: `kubectl apply -f storage.yaml`
***TIP***: Kubectl does not list persistent volumes unless explicitly ask for it: `kubectl get pv` or `kubectl get pvc` to return the details of the claim.

