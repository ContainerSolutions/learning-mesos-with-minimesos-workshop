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

### Mesos

1. Create a default minimesos cluster and retrieve the state file
2. View the Weave Scope UI
3. Destroy the cluster. Change the amount of resources and now retrieve the state file again

### Frameworks

1. Deploy Mesos Elasticsearch

### Zookeeper

1. List Zookeeper state for Mesos Elasticsearch
  * Go to Weave Scope and login to the Zookeeper container

### Weave Scope

1. Go to http://host:4040 to see the Weave Scope UI.

Check that all minimesos containers are running.

2. Login to the Zookeeper container

Find `zkCli.sh` and start the shell.

### Marathon

1) Go to the Marathon endpoint

Click 'Create'. Click the 'Docker container settings' and fill in `nginx` as the Docker image. Now click '+Create'.

2) Check if nginx is running

NOTE: In regular Mesos you can click on the task and the link to jump to the nginx endpoint. This does not work on minimesos because the nginx container uses a different network stack than Marathon because Marathon runs in a container. In a production Mesos cluster Marathon, the Mesos Agent and the containers all use the host's network stack. An upcoming feature in Mesos called 'IP Per container' will change this situation but this is not supported yet.

3) Check the Weave Scope UI to see your nginx container

### Wordpress exercise

First clone this repository and switch to `wordpress` directory.

1. Deploy the MySQL container

Use `minimesos install` to install the MySQL container. It is running on port 3306 and accessible on the IP of the Mesos Agent.

To find the IP address of the Mesos Agent run

`$ docker inspect CONTAINER_ID | grep IP`

Go to the Mesos master UI and check that the MySQL container is in a `RUNNING` state

2. Deploy the Wordpress container 

Use `minimesos install` to install the Wordpress container. It is running on port 3306 and accessible on the IP of the Mesos Agent.

### Application

Running a web application on Mesos (node.js + mongo)
something like: https://github.com/ContainerSolutions/node-openshift-sample
2 containers: 1 for frontend (node js), 1 for backend (mongodb)

1. deploy frontend
2. deploy backend
3. link frontend and backend
4. scale up frontend
5. kill some of the frontend

## Notes

* Add links to CS blogs in slides
* Downsides of Ansible / Chef is that they create a static partitioning of the cluster.
* Try satellite for monitoring 
* Add links to papers: Mesos, Zookeeper, PAXOS
* Try Apache Cotton
