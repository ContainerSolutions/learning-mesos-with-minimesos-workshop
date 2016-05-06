# learning-mesos-with-minimesos-workshop
We suggest that you clone this git repository to your machine,
and use an editor to step throught each task; by doing it this way,
you will always know which steps you have done already.

Futhermore, you'll be using the other files in this repository during the exercises.

## Exercises

### Before you start
Before you start with the exercises, we'd like to share some known isssues and tips & tricks with you.

#### Tips & tricks
 
* Kill and remove all Docker containers
  * `docker rm -f $(docker ps -q -a)`
* Find the IP address of a container
  * `docker inspect --format '{{.NetworkSettings.IPAddress}}' "CONTAINERID"`
* print IP addresses of all running containers
  * `for i in $(docker ps -q); do echo -n $i" "; docker inspect --format '{{ .NetworkSettings.IPAddress }}' $i; done`
* creating and managing docker machine. This comes in handy if you're running OSX or Windows.
  * create `docker-machine create -d virtualbox --virtualbox-memory 8192 --virtualbox-cpu-count 1 minimesos`
  * prepare environment `eval $(docker-machine env minimesos)`
  * adjust routing table `sudo route delete 172.17.0.0/16; sudo route -n add 172.17.0.0/16 $(docker-machine ip ${DOCKER_MACHINE_NAME})`

#### minimesos known issues

* When creating a volume it might not be visible on the host because they are created on a directory structure that is not mapped from Docker Machine to the host
* All Mesos containers: Master, Agent and Marathon have their own network stack which causes some subtle issues because containers will use a different network stack than the host. See https://github.com/ContainerSolutions/minimesos/issues/401
* `minimesos` does not run on Fedora because it comes with docker from RedHat, which has slightly different API. See https://github.com/ContainerSolutions/minimesos/issues/290

### Installation

* Install minimesos if you haven't already: http://minimesos.readthedocs.io/en/latest/
  
#### CLI

* Run `minimesos help` to see what commands are available
* Create a `minimesosFile` with `minimesos init`
* Change the name of the cluster to your name, and set `mapAgentSandboxVolume` to `true`
  (just to practise chaning these settings).
  These settings can be found in the `minimesosFile` you generated in the previous step.
* Launch the cluster with `minimesos up`. This may take a bit since images have to be downloaded
* Run `docker ps` to see what kind of containers are runnning
* Run `minimesos info` to find the endpoints of the containers in the minimesos cluster 
* Look inside the `.minimesos` folder in the directory you created the minimesos cluster. What is in it?
* Traverse the `sandbox-` directory structure and find the logs for the Weave Scope task.
  (HINT: use find/tree. The file name is stdout/stderr in some directory)
* What happens if you run `minimesos init` again?
* What happens if you run `minimesos up` again?
* Run `minimesos destroy`. What does `minimesos info` say? And `docker ps`?
* Done!

#### Mesos UI & Weave Scope

* Visit the Master's UI at `$MINIMESOS_MASTER`
* Find the Weave Scope task logs in UI
* Check out the frameworks and agent tab to check if everything is working as expected
* Go to `http://<MINIMESOS_NETWORK_GATEWAY>:4040` to see the Weave Scope UI.
  * Where `<MINIMESOS_NETWORK_GATEWAY>` is localhost if you're running Docker directly on your host,
    or if you're using Docker Machine, the IP address of the Docker Machine Virtual Machine.
  * Check that all minimesos containers are running.

You can use the above commands during the next few exercises to find information about your setup.
Feel free to experiment during this exercise, destroy your cluster and make changes to the `minimesosFile`. 

### Marathon (15 minutes)
* Destroy any existing minimesos cluster and recreate it by running `minimesos up`
* Evaluate the export's again.
* Go to the Marathon endpoint, printed out two steps ago.
  * Click 'Create'. Click the 'Docker container settings' and fill in `nginx` as the Docker image. Now click '+Create'.
  * Figure out IP address of the new container. There are a few ways to do it. What are they?
* Check if nginx is running by accessing `$NEW_CONTAINER_IP:80`
  * NOTE: In regular Mesos you can click on the task and the link to jump to the nginx endpoint.
    This does not work on minimesos because the nginx container uses a different network stack than Marathon because Marathon
    runs in a container. In a production Mesos cluster Marathon, the Mesos Agent and the containers all use the host's network stack.
    An upcoming feature in Mesos called 'IP Per container' will change this situation but this is not
    supported yet: https://github.com/ContainerSolutions/minimesos/issues/420
* Check the Weave Scope UI to check if your nginx container is running
* Now destroy your container via the UI
* Now start it again
* Now scale it up to 3 instances
* Now scale it down to 1 instance

### Mesos Elasticsearch (15 minutes)

Mesos Elasticseach is a Mesos frameworks that deploys Elasticsearch on Mesos.
Checkout http://elasticsearch.mesosframeworks.com website and read about what it does.

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

Switch to the `wordpress` directory in your cloned git repository.

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
