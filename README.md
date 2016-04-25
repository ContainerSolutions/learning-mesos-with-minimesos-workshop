# learning-mesos-with-minimesos-workshop

## Workshop structure

* Presentation - Cookbook style presentation which covers Mesos, Zookeeper, frameworks & Marathon
* Exercises - Covers the same topics but now with focused exercises using minimesos
* Wrap up - Summarize what we learned, allow for Q&A and discuss further learning

## Presentation

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

### Mesos

#### Create a default minimesos cluster and retrieve the state file

#### Destroy the cluster. Change the amount of resources and now retrieve the state file again

#### View the Weave Scope UI

### Marathon

#### Deploy nginx and connect to it 

### Check the Weave Scope UI to see your nginx container

### Frameworks

#### Deploy Mesos Elasticsearch

### Zookeeper

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
