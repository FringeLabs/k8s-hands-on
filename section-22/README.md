# Temporary problems with the AlertManager

***2 September 2020***: Just to warn you, the Helm chart for Prometheus Operator has been updated in the last few days, and it has caused several variations from what you see on the videos in this section. I am working on a fix for this right now, I am aiming to get a revised version published as soon as possible this September.

In the meantime, I recommend just watching the videos in this section. When I've updated the course, you'll be able to apply a yaml file which will enable you to replicate what you see on the videos.

I'm sorry for this; it was a serious mistake to use Helm to install prometheus-operator. It illustrates a major problem with Helm - you lose the ability to control exactly what is being deployed to your cluster.

I'll post an announcement when the re-worked section is ready.


## What happens if the Master Node Crashes? (on Kops, obviously)

The loss of the master is not a big deal as soon as the master is restored.
* Go on EC2 nodes and `Terminate` master node;
* After, if you run `kubectl get all` you will notice it can't connect ot the master node;
* Services are still available. However, until you have a master node back again, you won't be able to re-schedule or orchestrate pods, specially those that die during this timeframe;
* It will be a matter of time until AWS ASG adds a new note in place adn everything will work again;
  