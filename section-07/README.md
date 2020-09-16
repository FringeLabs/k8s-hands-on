# ActiveMQ as a Pod and Service

Goal: stand up a pod (and then, a service) for ActiveMQ.

**Image**: richardchesterwood/k8s-fleetman-queue

## Accessing activeMQ
Run `minkube ip` to grab virtual machine ip and open the following url: `http://<ip>:30010/admin`. You can use "admin/admin" to authenticate to the system.

