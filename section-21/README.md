# Monitoring K8s Cluster with Prometheus and Grafana

This is about monitoring your underlying hardware  to see how Kubernetes is behaving. 

## Monitoring a Cluster

Every cloud provide offers options, of course.
We can monitor all the hardware using AWS built-in tools. It is like ELK stack. There are many tools to do the job, but ELK is agnostic to the cloud provider. For that I'm going to use:

* Prometheus: for monitoring;
* Grafana: for visualization of those metrics;

The goal is to plug them to K8s and install them may be quite tricky. The easiest way to do it is using the Helm paclage manager.

## Helm Package Manager

Useful links:
* https://helm.sh/
* https://github.com/helm/helm

I don't use that quite often (like a day-to-day tool) but for sur ehas a place for it.
Helm allows you install an entire K8s application using a single line of command.

To install Helm binary and make it work you will need first have a "Helm Pod" on your system to communicate with kubectl and your cluster.

Helm setup:
1. Go to Github releases: https://github.com/helm/helm/releases
2. Copy the link from Linux amd64;
3. Uncompress: `tar zxvf <file.tar.gz>`
4. Move binary: `sudo mv linux-amd64/helm /usr/local/bin/`
5. Remove GZ file and extracted folder;
6. Test: `helm version`

### Initialize a Helm Chart Repository

The course uses a very old verison of Helm that has no `init` command. The instructions below I got from Helm page itself:
* Add official helm repo: `helm repo add stable https://kubernetes-charts.storage.googleapis.com/`
* The course uses 2.9.x, which still has init commant. It was removed on 3.0
* Notes about Helm Tiller and versions above 3.0:https://github.com/helm/helm/issues/6996

```
Helm init has been removed from helm 3.0 :-). You don't need it anymore. There is no more Tiller and the client directories are initialised automatically when you start using helm.
```

### Charts

Visit: https://github.com/helm/charts/tree/master/stable

A Chart is a Helm package. It contains all of the resource definitions necessary to run an application, tool, or service inside of a Kubernetes cluster. Think of it like the Kubernetes equivalent of a Homebrew formula, an Apt dpkg, or a Yum RPM file.

A Repository is the place where charts can be collected and shared. It's like Perl's CPAN archive or the Fedora Package Database, but for Kubernetes packages.

A Release is an instance of a chart running in a Kubernetes cluster. One chart can often be installed many times into the same cluster. And each time it is installed, a new release is created. Consider a MySQL chart. If you want two databases running in your cluster, you can install that chart twice. Each one will have its own release, which will in turn have its own release name.

With these concepts in mind, we can now explain Helm like this:

Helm installs charts into Kubernetes, creating a new release for each installation. And to find new charts, you can search Helm chart repositories.

Let's test=: 
* updating repo: `helm repo update`
* installing mysql: `helm install my-special-installation stable/mysql --set mysqlPassword=password`
* check that helm shows all the resources created;
* it has all the commands to connect to the pod, get the client and connect to the db;
* You can list the deployment with `helm ls`
* And delete it: `helm uninstall my-special-installation`

**NOTE**: Helm v3 doesn't have the flag name. Instead, it is passed as parameter.You also don't need the "fix helm privileges.txt" file instructions because we don't have "tiller" on v3 anymore. Also, `--purge` is the default behaviour for deleting. Please check `https://stackoverflow.com/questions/51594064/what-is-the-purpose-of-helm-delete-purge` if you'd like to keep history.


## Errata - steps needed to get a full set of data from Prometheus

If not doing that, we will only get a few things. So, we need to:
* Edit cluster definition: `kops edit cluster --name ${NAME}`
* Find `kubelet:` block;
* Add a subproperty `authenticationTokenWebhook: true`
* Add a subproperty `authorizationMode: Webhook`
* Save the file adn exit editor;
* Update cluster: `kops update cluster --yes`
* Roll changes to each node: `kops rolling-update cluster --yes` (it takes several minutes);
  
