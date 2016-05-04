# learning-mesos-with-minimesos-workshop

## Exercises

### Before you start
Before you start with the exercises, we'd like to share some known isssues and tips & tricks with you.

#### minimesos known issues

* When creating a volume it might not be visible on the host because they are created on a directory structure that is not mapped from Docker Machine to the host
* All Mesos containers: Master, Agent and Marathon have their own network stack which causes some subtle issues because containers will use a different network stack than the host. See https://github.com/ContainerSolutions/minimesos/issues/401
* `minimesos` does not run on Fedora because it comes with docker from RedHat, which has slightly different API. See https://github.com/ContainerSolutions/minimesos/issues/290

#### Tips & tricks
 
* Kill and remove all Docker containers
  * `docker rm -f $(docker ps -q -a)`
* Find the IP address of a container
  * `docker inspect --format '{{.NetworkSettings.IPAddress}}' "CONTAINERID"`
* print IP addresses of all running containers
  * `for i in $(docker ps -q); do echo -n $i" "; docker inspect --format '{{ .NetworkSettings.IPAddress }}' $i; done`
* creating and managing docker machine
  * create `docker-machine create -d virtualbox --virtualbox-memory 8192 --virtualbox-cpu-count 1 minimesos`
  * prepare environment `eval $(docker-machine env minimesos)`
  * adjust routing table `sudo route delete 172.17.0.0/16; sudo route -n add 172.17.0.0/16 $(docker-machine ip ${DOCKER_MACHINE_NAME})`

### minimesos basics (15 minutes)

### Installation

* Install minimesos if you haven't already: http://minimesos.readthedocs.io/en/latest/
  
#### Cli

* Run `minimesos help` to see what commands are available
* Create a `minimesosFile` with `minimesos init`
* Change the name of the cluster to your name, set `mapAgentSandboxVolume` to `true`
* Launch the cluster with `minimesos up`. This may take a bit since images have to be downloaded
* Run `docker ps` to see what kind of containers are runnning
* Run `minimesos info` to find the endpoints of the containers in the minimesos cluster 
* Display the Master's state information using `minimesos state` and see if you can find the cluster name you changed
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
* Find the Weave Scope task logs in UI
* Check out the frameworks and agent tab to check if everything is working as expected
* Go to `http://${MINIMESOS_NETWORK_GATEWAY}:4040` to see the Weave Scope UI.
  * Check that all minimesos containers are running.

You can use the above commands during the next few exercises to find information about your setup. Feel free to experiment, destroy your cluster and make changes to the `minimesosFile`. If there are commands you think are missing let us know!

### Marathon (15 minutes)

* Go to the Marathon endpoint using the `minimesos` commands you used earlier
  * Click 'Create'. Click the 'Docker container settings' and fill in `nginx` as the Docker image. Now click '+Create'.
  * Figure out IP address of the new container. There are a few ways to do it. What are they?
* Check if nginx is running by accessing `$NEW_CONTAINER_IP:80`
  * NOTE: In regular Mesos you can click on the task and the link to jump to the nginx endpoint. This does not work on minimesos because the nginx container uses a different network stack than Marathon because Marathon runs in a container. In a production Mesos cluster Marathon, the Mesos Agent and the containers all use the host's network stack. An upcoming feature in Mesos called 'IP Per container' will change this situation but this is not supported yet: https://github.com/ContainerSolutions/minimesos/issues/420
* Check the Weave Scope UI to check if your nginx container is running
* Now destroy your container via the UI
* Now start it again
* Now scale it up to 3 instances
* Now scale it down to 1 instance

### Mesos Elasticsearch (15 minutes)

Mesos Elasticseach is a Mesos frameworks that deploys Elasticsearch on Mesos. Checkout http://elasticsearch.mesosframeworks.com website and read about what it does.

* Deploying the framework
  * Checkout the [Mesos Elasticsearch Marathon JSON file](elk/es.json)
  * Note that the file contains a a token `${MINIMESOS_ZOOKEEPER}` which will be replaced by the URL to the Zookeeper container
  * Which docker image is running?
  * What does this image contain?
  * Now deploy it with `minimesos install`
  * Go the the Marthon UI. You should see an Elasticsearch task with status 'Waiting'. Hover over the 'Waiting' text. What does it say?
  * Compare the amount of memory and cpu required by Elasticsearch versus the amount of memory and cpu of your `agent` in your `minimesosFile`
  * Destroy your cluster, change your `agent`'s resources in the `minimesosFile` and create a new cluster
  * Now try deploying again
* Using the Mesos Master, Marathon and Mesos Elasticsearch UIs
  * Now Open the Mesos Master UI. There should be 2 tasks: `elasticsearch` and `elasticsearch-executor`
  * Click on the 'Frameworks' tab. You should see Marathon and Elasticsearch.
  * Click on the Elasticsearch framework. Click the Web UI link on the left. Try out the different tabs and views
  * Now index some Tweets via this tutorial: https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html
  * In the Mesos Elasticsearch UI open the 'Query Browser' and find your Tweets
* Scaling up
  * In the Mesos Elasticsearch UI open the 'Scaling' tab. Change the number of nodes to 3
  *  Why does it not scale up? ;-) Check the Master logs to see what is happening
  * Destroy your cluster, add extra `agent` blocks so you have 3 and run `minimesos up` again
  * Now scale up to 3 Elasticsearch nodes
  * Go to 'Tasks' and click on an Elasticsearch endpoint. You will see the standard Elasticsearch endpoint. Now append `_nodes` to the url to view the `_nodes` endpoint. Check that it lists 3 different Elasticsearch nodes 

### Wordpress (15 minutes)

First clone this repository and switch to `wordpress` directory.

* Deploy the MySQL container
  * Use `minimesos install` to install the MySQL container. It is running on port 3306 and accessible on the container IP. 
  To find the IP address of the Mesos Agent run `$ docker inspect CONTAINER_ID | grep IP`. 
  Go to the Mesos master UI and check that the MySQL container is in a `RUNNING` state
* Deploy the Wordpress container 
  * Use `minimesos install` to install the Wordpress container. It is running on port 80 and accessible on the container IP. 
* Review content of `wordpress.json` file
  * What address does Wordpress use to find MySql?
  * How does Wordpress container resolve this address? 
  
## Experiments
  
In the last hour of the workshop you are free to experiment with minimesos by deploying your own applications. Alternatively, you can install the ELK stack. See below

### Installing ELK - ElasticSearch, LogStash and Kibana

* Clone https://github.com/ContainerSolutions/learning-mesos-with-minimesos-workshop
* Change to the `elk` directory and check the content of `minimesosFile`
  * How many agents will be running?
  * What applications will be installed after the cluster is up?
  * Review content of JSON files to install these applications
* Start the cluster and open reported URLs for Mesos Master, Marathon. Open Weave Scope UI
  * How many different frameworks run in the cluster?
  * Open UI of installed ElasticSearch framework and check address of running ES node
  * Are there any indices? Check this in ES framework UI and by querying ES node
* Update `LOGSTASH_ELASTICSEARCH_HOST` value in `logstash.json`. Replace it with IP and port of running ES node. Install LogStash
  * Check in both Marathon and Mesos Master is LogStash Scheduler is running. LogStash agent?
  * Are there any ES indices created?
* Update `ELASTICSEARCH_URL` value in `kibana.json` - update IP and port. Install Kibana
  * It is a framework? or task?
  * What Kibana IP is reported in Mesos UI? Does it match real IP of the docker container? Why?
  * Looking at running Kibana container, figure out URL of the application and open it.
* Browse data in Kibana
  * Agree on using `logstash-*` indices
  * Go to `Discover` menu item
  * Click on `Last 15 minutes` in upper right corner and enable auto-refresh
  * Execute `echo "log file -> logstash -> elastic search -> kibana" >> .minimesos/test.log`
  * Check Kibana 
* Questions:
  * Why Consul could not help to connect to Elastic Search nodes?

## Questions

### Basics

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

### minimesos

* How do I configure resources in MM?
* How do I change the Mesos version in MM?
* How can I use MM with my integration tests?
* What is the Maven plugin?
