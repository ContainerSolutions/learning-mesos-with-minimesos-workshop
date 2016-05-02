# learning-mesos-with-minimesos-workshop

### Mesos

#### Basics

* What is Mesos, what problem does it solve and what can I do with it?
  * Mesos is a distributed resource manager that allows you to treat your machines as a single computer.
  * Benefits
    * Reuse the same machines for multiple workloads.
    * Instead of separate ES, Hadoop and Spark clusters use the same machines.
    * Cost savings through increased utilization.
    * Mesos applications can react dynamically by spawning or removing tasks based on the environment
* How can I deploy a framework/application?
  * Show how to run ELK from Java mode so it connects to the Master.
* How can I find application logs?
  * Explain the sandbox
* How do I shut down a framework/application? (Note, cannot reconnect with same name after shutdown)
* How can I see what is running on the cluster?
  * Show UI and state file
* How can I see how much resources the cluster has left?
  * Show UI and show state file 
* How does resource scheduling work?
  * Explain DRF
* How does leader election work?
  * Explain PAXOS
* Where can I get state information? (state.json, tasks.json and /help)
* What does it mean when I specify `cpus=1.5`?

### Advanced
  
* How do I authenticate a framework?
* How do I configure role reservations?
* How do I distribute files and binaries through the cluster?
* What is the difference between an application and a framework?
* What is IP per container?
* What is a good level of utilization?
* What happens when you fork bomb a Mesos cluster?
* How can I monitor my cluster?
* What is the replicated log?
* What happens if the master is killed?
* What is safe mode?
* What is checkpointing?
* How do I meaure resource utilization of my Mesos cluster?
 
### Frameworks

* What is a framework?
* What is the difference between scheduler, executor and task?
* What is the lifecycle of a framework?
* How do I deploy ELK on Mesos?
* What is the sandbox?
* How does sandbox garbage collection work?
* What happens is a scheduler is killed?
* Why would I want to write/use a framework?
* How do I write a framework?
** The easiest way is to use mesosframework. Second easiest way is to use Mesos Starter. Finally you can write your own using the HTTP or libmesos APIs in your favorite language.

### Zookeeper

* How can I check the contents of Zookeeper?
* What happens when a framework re-registers?
* How do I remove stale Zookeeper state?

### Service Discovery

* How do I deploy an application that requires a lookup of another service?

### Marathon

* How do I run a shell command on Marathon?
* How do I run a container on Marathon?
* How can I configure where my application gets deployed?
* How can I start applications using marathon when MM starts?
* How can I scale my simple application?
* How can I destroy?

### MiniMesos

* How do I configure resources in MM?
* How do I change the Mesos version in MM?
* How can I use MM with my integration tests?
* What is the Maven plugin?

## Exercises

### Minimesos known issues

* When creating a volume it might not be visible on the host because they are created on a directory structure that is not mapped from Docker Machine to the host
* All Mesos containers: Master, Agent and Marathon have their own network stack which causes some subtle issues because containers will use a different network stack than the host.
* Minimesos has a bug if you run on Fedora: https://github.com/ContainerSolutions/minimesos/issues/290

### Tips & tricks
 
* Kill and remove all Docker containers
  * `alias dacr='docker rm -f $(docker ps -q -a)'`
* Find the IP address of a container
  * `docker inspect --format '{{.NetworkSettings.IPAddress}}' "CONTAINERID"`

### Minimesos basics (15 minutes)

### Setup

* If you haven't already create a Docker Machine called minimesos
  * `$ docker-machine create minimesos --driver virtualbox`
* Clone the minimesos project and run `make setup` from that folder to add routing rules so the minimesos containers are reachable

#### Cli

* Run `minimesos help` to see what commands are available
* Create a `minimesosFile` with `minimesos init`
* Change the name of the cluster to your name, set `mapAgentSandboxVolume` to `true`
* Launch the cluster with `minimesos up`. This may take a bit since images have to be downloaded
* Run `docker ps` to see what kind of containers are runnning
* Run `minimesos info` to find the endpoints of the containers in the minimesos cluster 
* Go back to the terminal. Display the Master's state information using `minimesos state` and see if you can find the cluster name you changed
* Now retrieve the Master's state file from `$MINIMESOS_MASTER:5050/state.json`
* Find the container ID of the Mesos agent using `docker ps`. Now retrieve the state information using `minimesos state --agent <CONTAINER_ID>`
* Now retrieve the Agent's state file from `$MINIMESOS_MASTER:5051/state.json`. Note that the Master and Agent state files are quite different. Why?
* Look inside the `.minimesos` folder in the directory you created the minimesos cluster. What is in it?
* Traverse the `sandbox-` directory structure and find the logs for the Weave Scope task.
* What happens if you run `minimesos init` again?
* What happens if you run `minimesos up` again?
* Run `minimesos destroy`. What does `minimesos info` say? And `docker ps`?
* Recreate your cluster

#### Mesos UI & Weave Scope

* Visit the Master's UI at `$MINIMESOS_MASTER:5050`
* Find the Weave Scope task logs
* Check out the frameworks and agent tab to check if everthing is working as expected
* Go to `http://host:4040` to see the Weave Scope UI.
  * Check that all minimesos containers are running.

You can use the above commands during the next few exercises to find information about your setup. Feel free to experiment, destroy your cluster and make changes to the `minimesosFile`. If there are commands you think are missing let us know!

### Marathon (15 minutes)

* Go to the Marathon endpoint using the `minimesos` commands you used earlier
  * Click 'Create'. Click the 'Docker container settings' and fill in `nginx` as the Docker image. Now click '+Create'.
* Check if nginx is running by accessing `$MINIMESOS_AGENT:80`
  * NOTE: In regular Mesos you can click on the task and the link to jump to the nginx endpoint. This does not work on minimesos because the nginx container uses a different network stack than Marathon because Marathon runs in a container. In a production Mesos cluster Marathon, the Mesos Agent and the containers all use the host's network stack. An upcoming feature in Mesos called 'IP Per container' will change this situation but this is not supported yet: https://github.com/ContainerSolutions/minimesos/issues/420
* Check the Weave Scope UI to check if your nginx container is running
* Now destroy your container via the UI
* Now start it again
* Now scale it up to 3 instances
* Now scale it down to 1 instance

### Frameworks (15 minutes)

* Deploy Mesos Elasticsearch
  *  Go to https://github.com/mesos/elasticsearch, follow the 'read the docs' link and copy the Marathon JSON file
* Check to see if its running
* Go to the UI and scale up
  *  Why does it not scale up? ;-)
* Destroy your cluster, add extra agents so you have 5 and run `minimesos up` again
* Now scale up to 5 nodes
* Go to the Elasticsearch /_nodes endpoint and check that you have 5 different Elasticsearch nodes 
* Go to Weave Scope and login to the `zookeeper` container
* Find the `zkCli.sh` script and create a shell
* Now list the contents. You should see a `mesos-es` z-node which contains state information of Mesos Elasticsearch

### Wordpress (15 minutes)

First clone this repository and switch to `wordpress` directory.

* Deploy the MySQL container
  * Use `minimesos install` to install the MySQL container. It is running on port 3306 and accessible on the IP of the Mesos Agent. To find the IP address of the Mesos Agent run `$ docker inspect CONTAINER_ID | grep IP`. Go to the Mesos master UI and check that the MySQL container is in a `RUNNING` state
* Deploy the Wordpress container 
  * Use `minimesos install` to install the Wordpress container. It is running on port 3306 and accessible on the IP of the Mesos Agent.

## Notes

* Add links to CS blogs in slides
* Downsides of Ansible / Chef is that they create a static partitioning of the cluster.
* Try satellite for monitoring 
* Add links to papers: Mesos, Zookeeper, PAXOS
* Try Apache Cotton
