Tutorial 2 : Docker Orchestration Tools (Owner: Nicola Kabar)
Description: In this lab we will explore Docker Orchestration Tools ( mainly Swarm and Compose). We will go through how to set up a Swarm cluster using multiple discovery options , how Compose works ( including Compose files best practices), and how to deploy and troubleshoot multi-service, multi-container application across the Swarm cluster. 
Status:
Link to Lab Guide: 
Difficulty: Intermediate
Time: 30 mins
Goal: Create a Swarm cluster and deploy a multi-service application on it using Docker Compose.
Tasks: 
Setup Requirements: engine, git, compose
Running a multi-service app with Compose on a single engine
Creating a Swarm cluster
Running a multi-service app with Compose on a Swarm cluster
Scaling the app with Interlock


Tutorial 2 : Docker Orchestration Tools
Difficulty: Intermediate
Time: 30 mins

Tasks:
Build a docker-compose.yml file to deploy an app on single Engine
Create a Swarm Cluster 
Deploy the multi-service app on the Swarm Cluster


1. Build a docker-compose.yml file to deploy an app on single Engine

Docker Compose and Docker Swarm are key elements of the Docker Ecosystem. Docker Compose is an open-source tool to orchestrate building and deploying multi-service,multi-container applications. Compose relies on a configuration YAML file (defacto name is docker-compose.yml) to build, create, and run containers against a single Docker Engine or Docker Swarm cluster. The YAML configuration file has standard reference commands ( full list here : https://docs.docker.com/compose/yml/) that  define how the container is created at runtime.
Some of the most common commands used are: build, image, command, ports, and links. Compose should be already installed on your machine. To verify, issue the following command and ensure you see the currently installed version.

- docker-compose version

In the following tutorial, you will deploy a multiservice app, DockChat, that is composed of two services : web and db. DockChat is a simple Python+Mongo chat server. First step is to clone the DockChat repo from GitHub:

- git clone https://github.com/nicolaka/dockchat.git

Change directory to dockchat, and examine the list of files in the repo. 

- cd dockchat ; tree

You'll notice a Dockerfile for the 'web' service and a docker-compose.yml that describes the app's service configuration. Take a look at docker-compose.yml file using your favorite text editor.

# Mongo DB
db:
  image: mongo
  expose:
    - 27017
  command: --smallfiles
# Python App
web:
  build: .
  ports:
    - "5000:5000"
  links:
   - db:db

You can see that there are two service descriptions: db and web. Each of them define a service with certain build or runtime parameter:

		image: will specify the name of the image to use. Note: this needs to be an image that has already been built and available either locally, on Docker on-prem registry, or Docker Hub.
		build: will build an image from the Dockerfile that is specified directory. In this case, it's the current directory. In this example, you will use the 'mongo' image directly from Docker Hub.
		command: will specify the command to run in the container when it's run.
		ports: will specify any network host port mapping 
		links: will specify container linking parameters. Container linking is a mechanism to automatically provide access, network, and environment paramters for one container to discover and communicate with another contianer. In this example, we are linking the database container 'db' TO the 'web' container and giving it the alieas 'db'. The format for linking is <Name of Source Container>:<Alias in Destination Container>.
		expose: willl specify which port the Docker Engine should map to inside in the container. This is optional if the expose paramter was already declared in the Dockerfile of that service.

Before, running the app, you need to ensure that Docker is running locally and the Docker Compose client can connect to it. To verify, issue the following command and you should expect a similar output:

# docker-compose ps
Name   Command   State   Ports
------------------------------

You now can build and run the app. It is recommended to ensure that all the images that need to be built be built first before you run them. That is why docker-compose has the following two options : build and up. 
The build option builds an image for every service that has a 'build:' paramter in the docker-compose.yml file but doesn't actually create any container from that image. The 'up' option runs the services described in the docker-compose.yml file. The 'up' option WILL build the services' images before it runs them. It is just recommended to ensure that you successfully build all the requried images before running the containers from them as follows:

- /dockchat# docker-compose build

<snippit>
...
Successfully built a3896de220c8

- /dockchat# docker-compose up  
<snippit>
web_1 |  * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
web_1 |  * Restarting with stat
db_1  | 2015-10-09T22:45:47.497+0000 I NETWORK  [initandlisten] connection accepted from 172.17.0.25:39123 #4 (2 connections now open)


The output means that you have successfully created the two services. To access the app, go in your web browser to http://<your node IP address>:5000 . You should see somthing like this : 

FIXME: add picture of app

Stop the app after you confirm it worked.

- /dockchat# docker-compose stop && docker-compose rm -f 



2. Create a Swarm Cluster

Swarm is a native clustering for Docker. It allows you create and access to a pool of Docker hosts using the full suite of Docker tools. There are multiple ways to create a Swarm cluster(https://github.com/docker/swarm/tree/master/discovery). In this exercise you will use the Docker Hub token-based discovery service. You will create a Swarm Cluster with 2 nodes ( Node 1 and 2). The Swarm manager will be a container running on Node 0. The two nodes will be labeled for deployment schedluing and filtering purposes later.


Step 1: Ensure that Docker Engine on Node 1 and 2 is listening on a TCP port. 
You can do that by adding the following to /etc/default/docker configuration file followed by restarting Docker daemon.  

sudo nano /etc/default/docker
# Add the following for Node 1:

# Use DOCKER_OPTS to modify the daemon startup options
DOCKER_OPTS="-H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock --label=environment=staging"

# Add the following for Node 2:
DOCKER_OPTS="-H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock --label=environment=production"


Then restart Docker on each of the nodes.
$sudo service docker restart

Note: if Docker Engine TLS verification is enabled, it is recommended to use TCP port 2376 instead. For the remainder of these exercises, we will not use Docker TLS verifications.


Step 2: Create a unique Cluster Token on Node 0. This will output a unique cluster token. You will use this Token for the remainder of the tutorial. 

- docker run --rm swarm create

Step 3: Configure the Swarm Manager on Node 0. The manager runs in a container and is configured to use a different TCP port (3375) than engine (2375). 

- docker run -d -p 3375:2375 swarm manage token://<cluster_id_from_Step_2>

Step 4: Register the Swarm agents to the discovery service. Do this step on Node 0. The node’s IP must be accessible from the Swarm Manager. Use the following command and replace with the proper node_ip and cluster_id to start an agent.

- docker run -d swarm join --addr=<node_1_ip:2375> token://<cluster_id_from_Step_2>
- docker run -d swarm join --addr=<node_2_ip:2375> token://<cluster_id_from_Step_2>

Note: Please use node 1 and node 2's private IPs (10.x.x.x) and NOT their public IPs. To be safe, simply use their DNS names instead.

Step 5: Confirm that the two nodes are part of the Swarm clust.

- docker -H tcp://0.0.0.0:3375 info

docker -H tcp://0.0.0.0:3375 info
Containers: 6
Images: 19
Role: primary
Strategy: spread
Filters: affinity, health, constraint, port, dependency
Nodes: 2
 ip-10-0-0-80: 10.0.0.X:2375
  └ Containers: 0
  └ Reserved CPUs: 0 / 2
  └ Reserved Memory: 0 B / 7.703 GiB
  └ Labels: executiondriver=native-0.2, kernelversion=3.19.0-26-generic, operatingsystem=Ubuntu 14.04.1 LTS, storagedriver=aufs
 node1: 10.0.0.X:2375
  └ Containers: 6
  └ Reserved CPUs: 0 / 2
  └ Reserved Memory: 0 B / 7.703 GiB
  └ Labels: executiondriver=native-0.2, kernelversion=3.19.0-26-generic, operatingsystem=Ubuntu 14.04.3 LTS, storagedriver=aufs
CPUs: 4
Total Memory: 15.41 GiB
Name: e6b3f712c521

- docker -H tcp://0.0.0.0:3375 version

Client:
 Version:      1.8.2
 API version:  1.20
 Go version:   go1.4.2
 Git commit:   0a8c2e3
 Built:        Wed Oct  7 17:48:28 UTC 2015
 OS/Arch:      linux/amd64

Server:
 Version:      swarm/0.4.0
 API version:  1.16
 Go version:   go1.4.2
 Git commit:   d647d82
 Built:
 OS/Arch:      linux/amd64


3. Re-deploy DockChat on the Swarm Cluster. After successfully creating the Swarm cluster, you can redeploy Dockchat. The first step is to point all Docker client commands at the Swarm manager instead of a single engine. To do so, you need to create a new environment variable called DOCKER_HOSt.

- export DOCKER_HOST=<Node_1_IP_OR_DNS>:3375
- docker info
Containers: 5
Images: 11
Role: primary
Strategy: spread
Filters: affinity, health, constraint, port, dependency
Nodes: 2
 ip-10-0-0-80: 10.0.0.80:2375
  └ Containers: 2
  └ Reserved CPUs: 0 / 2
  └ Reserved Memory: 0 B / 7.703 GiB
  └ Labels: environment=testing, executiondriver=native-0.2, kernelversion=3.19.0-26-generic, operatingsystem=Ubuntu 14.04.1 LTS, storagedriver=aufs
 node1: 10.0.0.170:2375
  └ Containers: 3
  └ Reserved CPUs: 0 / 2
  └ Reserved Memory: 0 B / 7.703 GiB
  └ Labels: com.dockercon.environment='production', executiondriver=native-0.2, kernelversion=3.19.0-26-generic, operatingsystem=Ubuntu 14.04.3 LTS, storagedriver=aufs
CPUs: 4
Total Memory: 15.41 GiB
Name: e6b3f712c521

Currently, docker-compose cannot build an image on a Swarm cluster. Therefore, you need to first build the image using the Docker Client, tag it, push it to Docker Hub, and pull it on all nodes.

- docker build -t <your_Docker_Hub_username>/dockchat:v1 .
- docker login 
( log in with your Docker Hub credentials)
- docker push <your_Docker_Hub_username>/dockchat:v1

You should see something like:

e2a4fb18da48: Image successfully pushed


Now, you would need to change docker-compose.yml file to deploy on Swarm. The first thing to change would be the source of the 'web' image. You would need to change the following using your favorite text editor.

(-) build: .
(+) image: <your_Docker_Hub_username>/dockchat:v1

We will also use constraints/affinity rules to ensure Swarm selects the right nodes for deployment. Inititally, we labeled node 1 as a staging node, and node 2 as s production node. We can specifc in the docker-compose.yml file certain constraints to ensure the service is deployed on the environment we specified. Add the following Compose paramters under BOTH the services (db and web)

(+) environment:
(+)   - "constraint:environment==staging"


The final Compose file should look like this :

# Mongo DB
db:
  image: mongo
  expose:
    - 27017
  command: --smallfiles
  environment:
   - "constraint:environment==staging"
# Python App
web:
  image: <your_Docker_Hub_username>/dockchat:v1
  ports:
    - "5000:5000"
  links:
   - db:db
  environment:
   - "constraint:environment==staging"


Now, you need to pull the images on all nodes, and deploy the services:

- docker-compose pull
- docker-compose up -d
- docker-compose ps
- docker ps

You will notice that both containers were deployed on Node 1 ( the staging node). This is because we constraing the deployment of both services only on the 'staging' nodes. Also, since the two services are linked, Docker Compose will ensure they are deployed on the same node.

Now you can stop the containers, adjust the docker-compose.yml file contraints to the prodiction nodes, and redeploy. For both the services , adjust the following:

(-) - "constraint:environment==staging"
(+) - "constraint:environment==production"

- docker-compose stop && docker-compose rm -f
- docker-compose up -d
- docker ps

Note that now the two containers were deployed on Node 2, which is the production node.


Summary: You have successfully used deployed a multi-service app using Docker Compose on both a single engine as well as a Swarm Cluster.

















