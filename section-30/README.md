# Kubernets ConfigMaps and Secrets

The best way to leverage a image is to make it configurable so we can configure it using environment variables and other techniques.

## Creating a Config Map

Let's say we have a database to be accessed by a pod and we need to share a DB url to be used in multiple pods. We could add env vars on every pod. That would work for sure. BUt what if the value changes? I would not like to go on every descriptor file and change the value. That is error prone.

The other option would be through ConfigMaps:

```
apiVersion: v1
kind: ConfigMap
metadata:
    name: global-database-config
    namespace: default
data:
    database.url: "https://dbserver.somewhere.com:3306"
    database.password: "P@ssW0rd1"
```
If we save as `database-config.yaml` and `apply` the file, we can check that through `kubectl get cm` or `kubectl describe cm global-database-config`.


## Consuming a ConfigMap as Environment Variables

How do we actually consume it? There are at least 3 ways to do it.

### The worst way

In the yaml we declare the environment variable and tell k8s to lookup the value when we apply the file.

```
spec:
  containers:
    env:
    - name: DATABASE_URL
      valueFrom:
        configMapKeyRef:
          name: global-database-config
          key: database.url
```
It is the worst because you add 4 lines of yaml at least and update individually the pods to have it working. A lot of work and not much different from the original copy & paste approach.

Anyways, you can always check the environment variables from the container inside through `kubectl exec -it <name of the pod> bash` and then using terminal commands to check the environment veriables, like `echo $DATABASE_URL` or `echo $DATABASE_PASSWORD`.


## Do changes to a ConfigMap get propagated?

Assuming the fact that we change the value of `database-config.yaml` file, if we change and apply that, the new value is not propagated to pods that are using that config. There is no mechanism on k8s to do that in the way we set things up so far. The only option by now is to delete the pod and the value is going to be updated when recreating the pod.

### Another way to do that

A common approach is to create a new configmap with a new version (using name convention) to store the new values. The downside of it is we still have to find a way to do global search and replace across the entire state in order to change the config map name reference in the deployments / pods definitions.

Again, unfortunately, we need to manually go every place and replace the references.


## How to consume multiple environments variables with envFrom

Another technique that can simplify things is to use `envFrom`:

```
spec:
  containers:
  - name: position-simulator
    envFrom:
    - configMapRef:
        name: global-database-config
```

This is much simpler. This will turn all the "data" block from the configmap into environment variables for that pod / container.
With this approach we can replace those long blocks of lines in the yaml files by `envFrom` reference.


## Mounting ConfigMaps as Volumes

If the application doesn't work with environment variables, we can moutn the content of ConfigMap as a "volume":
```
spec:
  containers:
  - name: position-simulator
    volumeMounts:
    - name: database-config-volume
      mountPath: /etc/any/directory/config

  volumes:
  - name: database-config-volume
    configMap:
      name: global-database-config-v3 
```

It's not the best yaml in the world but now you have all the values in the file system. With this approach, there will be no environment variable but we will have one file for each key we defined in the config map.

It may be useful if creating something like `database.properties`:
```
data:
  database.properties: |
    database.url=https://dbserver.somewhere.com:3306
    database.password=P@ssW0rd1
```

Following this example k8s will create a file called `database.properties` on `/etc/any/directory/config` folder path with the content we defined in the yaml.


## Creating Secrets

Another concept we have on K8s is called "secret" which allows us to store sensitive information. It is much more safe and flexible than any approach we followed so far.

Let's say we are going to add our AWS credentials in our secrets:
```
apiVersion: v1
kind: Secret
metadata:
  name: aws-credentials
data:
  accessKey: MTIzNDU2Nzg5MAo=
  secretKey: U0VDUkVUMTIzNAo=
```

**Note**: to make it work, all secrets must be stored as base64 encoded string. We can do things like `echo <string> | base64`. Otherwise, it will fail when running `apply` command. The other option is to let k8s encode that for you. If you decide to do that, you have to change `data` block to `stringData`, wrapping the values by `" "`. This way, k8s will encode it for you.


## Using Secrets

Secrets are not secure. There is no encryption around it. Actually, they are not much more secure than a ConfigMap. Obviously, by defult is quite hard to get the values printed in the terminal when running `describe` commands. However, there is no vault to store them. Actually you can retrieve values running `kubectl get secret aws-credentials -o yaml`.

**Tip**: Using RBAC (from previous secionts) helps on restric access to secrets to people. That is the big point from Secrets. Given the fact they are different objects than ConfigMaps, we can apply different access levels for both.


## Where have we already used ConfigMaps and Secrets?

When we build ELK stack (file `elastic-stack.yaml`):
* line 83
* line 101
* line 112
* line 114

And when looking on fluentd-config.yaml:
* line 14 (check the content);
* several other "files" to be created into the system based on mount volumes we used as on example on this session (with ConfigMaps)

In the chapter for alerting we used secreds for `slack_api_url` on `alertmanager.yaml`.

We can say that Secrets are ConfigMaps that are a bit harder to look at but they definitely have no special security mechanism around it.

COnsidering a scenario with thousands of pods, we should minimize the global shard data. Still, we will find scenarios where it makes sense to have data shared with all of them. ConfigMaps and Secrets are the way to go on that case.

A few projects that are interested to follow up form here:
* Spring Cloud Kubernetes: connects your java app to K8s ConfigMaps and has `hot reloading feature`';
* Spring Cloud Config: provides server and client-side capabilities for storing key-values that are used for configurations. If all your apps are Spring Boot apps, this is the way to go. Otherwise, ConfigMaps is still the way once they are agnostic to the containes technology.

***More about Spring***: https://www.youtube.com/watch?v=DiJ0Na8rWvc&t=563s


