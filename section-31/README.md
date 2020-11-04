# Ingress Controllers

All the network traffic is routed by load balancer in the cloud platform provider. But there is a big problem which is when you want to expose more than one service. That requires having one load balancer per service in the cloud. Of course there is a huge advantage on having a seperate hardware for each service, but the disadvantage is that LB are expensive resources.

In a situation where you'd like to have 10, 50 or 100 services exposed it may be interesting reuse the same LB for the group of services that are part of the same project.

## Introducing Ingress

So far on this example we have been using a "Classic Load Balancer". But on AWS we have something called "Application Load Balancer" which allows to match the ports and making all of it more flexible. The problem is that K8s is agnostic to the cloud and doesn't play well with Application Load Balancers. We also dn't have ALB available in all cloud providers.

The other point is that ALB is not required at all on K8s. There is a general solution called **Ingress Controller** (eg Nginx), a special service on K8s that is plugabble and we don't need to write ourselves and is going to do the job. Based on the configuration we provide it will make routing decisions.

## Defining Routing Rules

***IMPORTANT***: In order to use Ingress with Minikube we need to enable the addon called "ingress".

It takes some time until it is created: `kubectl get po -n kube-system`
You should see `nginx` and `default-http-backend` pods being created. You will notice a service for `default-http-backend` but no service for ingress controller.

So, we need to get rid from the default routing from the Ingress Controller to Default  Backend so we can do our own routing. Let's create our `ingress.yaml` file:

```
# from v1.14 k8s version - always check the docs first
# apiVersion: networking.k8s.io/v1beta1
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: basic-routing
spec:
  rules:
    - host: fleetman.com
      http:
        paths:
          - path: /
            backend:
              serviceName: fleetman-webapp
              servicePort: 80          
```
After applying the yaml you can check that using `kubectl get ingress` or `kubectl describe ingress basic-routing`.


**Note**: we cannot use ip address to identify host. We need domain names. If you don't have. you can simulate them editing `etc.hosts`:

```
<ip from minikube ip command>   fleetman.com
<ip from minikube ip command>   queue.fleetman.com
```

## Adding Routes

Now, it is time to add multiple services:
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: basic-routing

spec:
  rules:
    - host: queue.fleetman.com
      http:
        paths:
          - path: /
            backend:
              serviceName: fleetman-webapp
              servicePort: 80
    - host:
      http:
        paths:
          - path: /
            backend:
              serviceName: fleetman-queue
              servicePort: 8161
```
Just need to apply the file and make sure you have domains on `/etc/hosts` file.

**Note**: if doing anything serious with Ingress please bookmark the NGINX Ingress Controller page. There are several other features (like authentication) that can be useful to protect resources.

## Authentication

You can go a bit further and also add authentication. There is support for different kinds of authentication.
On this example we are going to do `Basic Auth`. We could also use external oauth, client certificate and other techniques.

Please check the link for more details: https://kubernetes.github.io/ingress-nginx/
After following the guideline:

1. Run `kubectl create secret generic mycredentials --from-file auth`
2. Add configuration in the ingress file (annotations);
3. Apply ingress file changes;
4. Try to access `queue.fleetman.com`;


### Ingress File changes

```
metadata:
  name: basic-routing
  annotations:
    nginx.ingress.kubernetes.io/auth-type: basic
    nginx.ingress.kubernetes.io/auth-secret: mycredentials
    nginx.ingress.kubernetes.io/auth-realm: "Get lost unless you have a password"
```

**Note**: All the communication here happens with no certificate and is not secure (HTTPS).

We could also split the file in 2:
- `ingress-secure.yaml`
- `ingress-public.yaml`

This way, we can force login in one system and make another public. You are able to have multiple yaml files for your ingress controller.


## Running Ingress on AWS

The problem when using in the cloud is that you have to install Ingress Controller itself. The documentation page has a `Deployment` section with the instructions about how to install it on AWS: https://kubernetes.github.io/ingress-nginx/deploy/#aws

**Note**: instead of installing `mandatory.yaml` file from the guide, download the file first and save it. It will create a namespace called `ingress-nginx` with a few resources inside. With the file `service` it is recommended to have it once you may need to edit. The `patch-configmap` file is not used on this example but it is heavily used for TLS/HTTPS implementation.

You also doesn't need to have nodePort on any of the pods. Actually, you can have all set as ClusterIP once the entire traffic will be managed by ingress nginx.


## Testing the Ingress Rules

Once we don't have the domain, we can replace the minikube ip address by the AWS load balancer ip address in our machine `/etc/hosts` file. To turn out cluster into a production system we just need a domain and use Route53 to point it to our load balancer.

The last thing to be addressed is to make this cluster HTTPS. Obviously, we should also think on having a separate authenticator for Oauth2.


## (Extra) setting up HTTPS with TLS termination at the load balancer

How can you setup HTTPS with TLS: https://www.youtube.com/watch?v=gEzCKNA-nCg&feature=youtu.be
