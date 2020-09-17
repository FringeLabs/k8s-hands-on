# Networking and Service Discovery

Inside the same POD, all containers can refer each other using `localhost`.
Even it is acceptable to have many containers inside the same POD this is not recommended.

If you put a DB and the APP inside the same POD, both will "die" when any container inside the same POD fails.

## Networking Overview in K8s

K8s maintains his own DNS service to provide ip addresses to services/containers. This is called `kube-dns`. It is a service you don't need to configure and it is always running in the background.

If the webapp wants to access the service called "database", all that has to be done is to refer to the name `database` and the magic will happens. THe webapp service will "lookup" into dns for "database" and pick the right ip address.

When you look to the services using `kubectl` you can't see the dns in the list. That is because of a concept on k8s called **namespace**.


### Namespace

Namespaces are partitioning mechanisms to help to organize/group your pods based on architecture concern or functionalities (depends of your idea about how to group pods). You can split backend and frontend apps on different namespaces, by technology, by customer, etc...

If you don't specific a namespace, everything goes to the default namespace. A good way to understand that is through the command
`kubectl get ns` or `kubectl get namespaces`.

Another way to interact with namespaces:

```
➜  k8s-hands-on git:(master) ✗ kubectl get po -n kube-system
NAME                               READY   STATUS    RESTARTS   AGE
coredns-f9fd979d6-z4s99            1/1     Running   2          2d14h
etcd-minikube                      1/1     Running   2          2d14h
kube-apiserver-minikube            1/1     Running   2          2d14h
kube-controller-manager-minikube   1/1     Running   2          2d14h
kube-proxy-m5t7b                   1/1     Running   2          2d14h
kube-scheduler-minikube            1/1     Running   2          2d14h
storage-provisioner                1/1     Running   8          2d14h
```

or:

```
➜  k8s-hands-on git:(master) ✗ kubectl get all -n kube-system
NAME                                   READY   STATUS    RESTARTS   AGE
pod/coredns-f9fd979d6-z4s99            1/1     Running   2          2d14h
pod/etcd-minikube                      1/1     Running   2          2d14h
pod/kube-apiserver-minikube            1/1     Running   2          2d14h
pod/kube-controller-manager-minikube   1/1     Running   2          2d14h
pod/kube-proxy-m5t7b                   1/1     Running   2          2d14h
pod/kube-scheduler-minikube            1/1     Running   2          2d14h
pod/storage-provisioner                1/1     Running   8          2d14h

NAME               TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
service/kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   2d14h

NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/kube-proxy   1         1         1       1            1           kubernetes.io/os=linux   2d14h

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/coredns   1/1     1            1           2d14h

NAME                                DESIRED   CURRENT   READY   AGE
replicaset.apps/coredns-f9fd979d6   1         1         1       2d14h
```

**Q: How about the public namespace?**
***A:*** https://stackoverflow.com/questions/45929548/whats-the-kube-public-namespace-for

***It is very important to understand that everytime you are nto working with the default namespace you have to specify the namespace name on kubectl.***


## Service Discovery: accessing MySQL from a POD

1. Get pod name: `kubectl get po`
2. Exec shell inside pod: `kubectl exec -it <podname> sh`
3. Check DNS config content: `cat /etc/resolv.conf` . You can see the first IP address is the one tht matches with kube-dns ip address, which is the place where the machine is going to lookup for resolution.
4. Test dns: `nslookup database` - the last line will have the ip address of mysql server
5. Install mysql client: `apk update && apk add mysql-client`
6. Test mysql: `mysql -h database -uroot -ppassword fleetman`
7. Create a table: `create table testtable (test varchar(255));`
8. Display tables: `show tables;`
9. Exit the client (but not the shell).


### Fully Qualified Domain Name (FQDN):

While still in the shell:

1. Notice the service is not registered in the DNS as "database", but as a fqdn `database.<namespace>.svc.cluster.local`
2. That can be checked in the config as well: `cat /etc/resolv.conf`
3. Notice the long name in the file either.
