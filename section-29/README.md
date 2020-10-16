# RBAC (Role Based Access Control) on a Kubernetes Cluster

So far we had assumed we only have one person managing all the things. To restrict permissions, we can use RBAC.

## A note for EKS users

If you're working with EKS, then you won't be able to use the Authentication method (certificates) that we use in this section. For that reason, I'm currently working on a new version of this section, especially for EKS. I'll push out an announcement when this new section is ready.

In brief, if you're using EKS and you need multiple users to access the cluster, then you use IAM to create users, which is a very different approach to the one I use in the following lessons.

## Defining Roles

There is a RBAC API that supports `Role` and `ClusterRole` objects management, where we can definte what we can do on cluster management.

Assuming we are going to allow someone to "read" info form resources on `default` namespace:
```
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
    name: new-joiner
    namespace: default
rules:
-   apiGroups: ["" , "apps"]  # "" indicates core API group
    resources: ["pods"]
    verbs: ["get", "list", "watch"]
```

The verb `watch` is an argument to `get` operation that keeps updating the results based on changes. This is not a refresh of information already listed. It just adds the updates to existing result list instead.

In the `apiGroups` you can specify `pods`, `services` but you **CAN'T** specify `deployments`. That is because of the `apiVersion` and the core API Group. To include you have to add `apps` which is the group we are adding the deployments. Btw, we should **NEVER** use `"*"` value in `apiGroups`. That gives access to **EVERYTHING** and we don't want to allow users to have it.

After applied with `kubectl` we can check the role content running `kubectl describe role new-joiner`.

## Defining RoleBindings

We don't defining users on k8s. Instead we create or add to the yaml files an object called `RoleBinding`:
```
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
    name: put_specific-user-or-users-into-new-joiner-role
    namespace: default
subjects:
-   kind: User
    name: francis-linux-login-name
-   kind: User
    name: another-linux-login-name
roleRef:
    kind: Role
    name: new-joiner
    apiGroups]: rbac.authorization.k8s.io
```

As previously we can `kubectl get rolebinding put_specific-user-or-users-into-new-joiner-role`. Other commands like `describe` always work.

## Setting up a "context" for the user

K8s doesn't create users and expect you do that. It is your responsability to have a tool to create and configure your users. YOu can configure with Google Accounts as with Amazon. With Minikube you can use "An admin distributing private keys" approach.

So, Kubernetes doesn't care if the user name in the file is `francis-linux-login-name`. You will have to do the setup to integrate with linux by yourself. 

Let's assume we are on AWS:
```
$ whoami                                    # this lists who you are on AWS (we are superuser here)
ec2-user

$ sudo useradd francis-linux-login-name     # we need to create the user with the name...
$ ls /home                                  # ... and confirm it worked!
$ sudo passwd francis-linux-login-name      # we set a password for the user.

$ kubectl create ns playground              # let's create a namespace where the user can do whatever he wants (not anywhere else)
$ su - francis-linux-login-name             # you can "switch" to another user profile.

$ kubectl get all                           # this fresh account will get an error
<refused access error>

$ exit                                      # jump back to superuser profile
$ kubectl config view                       # see that new joiner didn't have access because it doesn't have server endpoint configured
$ su - francis-linux-login-name             # switch back
$ kubectl config view                       # you will see the config is EMPTY. Need to configure it

$ kubectl config set-cluster <cluster name> --server=<url we copied>
$ kubectl get all                           # this is still will fail because we don't have a "context" to 
                                            # specify the cluster we are talking to. It is useful on multi-cluster environment

$ kubectl config set-context mycontext --user francis-linux-login-name --cluster <cluster name>
$ kubectl config view                       # after fixing we will see the context set
$ kubectl config use-context mycontext      # switch to the context
$ kubectl get all                           # here we have another failure: certificate failure (you didn't auth)
                                            # you need the admin user/password or a certificate.
```

**NOTE**: all of this may difer on newer versions of K8s.

## Issuing a Kubernetes signed X.509 certificate

As a super user we need to generate the key so the user can log in into k8s. 

**TIP**: Let's encrypt allows you to create free certificates as soon as you prove you own the domain! (chekc the website for more info).

So, with that:
* Remember that kubectl uses the loadbalancer (HTTP API) created by kops to manage the cluster;
* This API only accepts requests signed by the certificate of that specific K8S cluster, issued by its own CA;
* At the time we created the cluster the Certificate (.crt) was generated;
* Once the new user has limited privileges, it needs its own certificated;
* As super users we own and hold this CA so we can generate the certificates;
* For this case, no cert has to be acquired.

As super user, let's generate:
```
# generating the key and then, the CSR file

$ openssl genrsa -out private-key-francis.key 2048                
$ openssl req -new -key private-key-francis.key -out req.csr -subj "/CN=francis-linux-login-name/O=francis-linux-login-name"

# to grab the cluster PK and/or the certificate you can go to the S3 Bucket.
# There is a folder called pki -> private -> ca
# you can copy to your system using AWS
$ aws s3 copy s3://chesterwood-state-storage/fleetman.k8s.local/pki/private/ca/<file>.key kubernetes.key

$ chmod 400 kubernetes.key                  # lets make it only readable by superuser
$ chmod 400 private-key-francis.key         # lets make it only readable by superuser

# now, let's pick the certificate
$ aws s3 copy s3://chesterwood-state-storage/fleetman.k8s.local/pki/issued/ca/<file>.crt kubernetes.crt

# now, generate a signed certificate for francis valid for 1 year
$ openssl x509 -req -in req.csr -CA kubernentes.crt -CAkey kubernetes.key -CAcreateserial -out francis.crt -days 365
```

After all those steps, we should have certificates in place.

## Installing the user's certificate

After giving to the user the generate files. For that, as superuser:
```
$ sudo mkdir /home/francis-linux-login-name/.certs
$ sudo mv francis.crt /home/francis-linux-login-name/.certs
$ sudo mv private-key-francis.key /home/francis-linux-login-name/.certs
$ sudo mv kubernetes.crt /home/francis-linux-login-name/.certs
$ rm kubernetes.key
$ rm kubernetes.srl
$ rm req.csr
$ sudo chown -R francis-linux-login-name:francis-linux-login-name /home/francis-linux-login-name/.certs/
```

Notice after that the private-key is read-only for francis, so no one can steal his PK.

## Allocating Access to Users

As superuser we are done. Now, as "francis":
```
$ su - francis-linux-login-name
$ kubectl get all                       # that should still fail
$ cd .certs                             # files should be there

$ kubectl config view
$ kubectl config set-credentials francis-linux-login-name --client-certificate=francis.crt --client-key=private-key-francis.key
$ kubectl config view

$ kubectl get all                       # this still should fail

$ kubectl config set-cluster fleetman.k8s.local --certificate-authority=kubernetes.crt
$ kubectl config view

$ kubectl get all                       # now it should work. Maybe withj "forbidden" messages, but that is on K8s level
```

The forbidden message happens the RoleBinding wasn't applied to K8S yet. So, as superuser, it is just a matter to apply the yaml with the definitions.

After that, all should work. For things that are Forbidden, it is just a matter to add the correct apiGroup to the role definition updating the yaml file. Example: `autoscaling` or `extensions` groups.

**Note**: Deployments are listed through `extensions` group due backward compatibility. It has been changed by this still goes through the old group, requiring both groups to be added. After that, we will be able to see the deployments list twice in the result set on `deployment.apps` group and `deployment.extensions` respectively.


## ClusterRoles and ClusterRoleBindings

Roles and Rolebindings are tied to namespaces. In our example so far, we have been using the `default` namespace. But we created `playground` for them. To achieve that level, the best strategy may be use ClusterRoles and CLusterROleBindings. Those objects work as the same, but they are not attached to the `namespace`. The are "worldwide" and are valid for the entire cluster.

It is jsut a matter to change the yaml to update the object names and remove the namespace property.
With that, francis should be able to list all the things on another namespace through `kubectl get all -n playground`.

The last job is to combine both strategies:
* create the cluster policies to give read only access to the entire cluster;
* create the role and rolebinding for specific operations to the namespace `playground`;
* 

Just need to apply the yaml and it should work.
