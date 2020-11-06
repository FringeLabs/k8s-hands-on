# Other workload types

Several other types of workloads that don't fit in the app.

## Batch Jobs

It will create a pod and K8s it will make sure it runs successfully until completion, like a process that has to be executed until it is over instead of running forever.

If you create as a Pod, you will notice that `Restart policy` will make this pod keeping restarting once the command ends. Obviously you can configure it to restart on failures or to never be restarted. The problem with this is: what happens if the job goes wrong?

Please check the K8s documentation on the workloads `Job`.


