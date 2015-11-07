
# Tutorial 2 : Docker Orchestration

> **Difficulty**: Intermediate

> **Time**: 20-30 mins

> **Prequisites**: 3 Nodes with Docker CS 1.9 Engine

> **Tasks**:
> 
> * Step1: Deploy a Multiservice Application with Compose
> * Step2: Set Up a Docker Swarm cluster
> * Step3: Creating Ogranizations, Teams, and Members
> * Step4: Pushing and Pulling DTR Images
> * Step5: Storage and Logs
> * Step6: Authentication,Security and Notary Integration
> 
> Tutorial 2 : Docker Orchestration Tools (Owner: Nicola Kabar)
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

## Prerequisites

* You will be using **node-0,node-1,and node-2**
* Ensure that no containers are running on these containers.
* Ensure that Docker engine uses default daemon options( `DOCKER_OPTS` should be commented out in `/etc/default/docker` file)
* Ensure that DOCKER_HOST is unset ( `$unset DOCKER_HOST` )
* Certain TCP ports are only allowed on the private AWS network. Therefore, you would need to use the private network (10.X.X.X) when you subsitute the IP of the instance in certain commands/configuration files throughout this lab.

# Getting Started with Docker Compose and Swarm

**Background**: 

Docker Compose and Docker Swarm are key elements of the Docker Ecosystem. Docker Compose is an open-source tool to orchestrate building and deploying multi-service,multi-container applications. Compose relies on a configuration YAML file (default name is **docker-compose.yml**) to build, create, and run containers against a single Docker Engine or Docker Swagrm cluster. The YAML configuration file has standard reference commands ( full list [here](https://docs.docker.com/compose/yml/) ) that  define how the container is created at runtime.
Some of the most common commands used are: b**uild, image, command, ports, and links**. Compose should be already installed on your machine. To verify, issue the following command on all-nodes and ensure you see the currently installed version.

`node-0# docker-compose version`

## Step1: Deploy a Multiservice Application on a single Engine

In this step, you will deploy a multiservice app, **DockChat**, that is composed of two services : **web** and **db**. DockChat is a simple Python+Mongo chat server. First step is to clone the DockChat repo from GitHub on **node-0** 

` node-0# git clone https://github.com/nicolaka/dockchat.git `

Change directory to dockchat, and examine the list of files in the repo. 

```
node-0:~# cd dockchat/
node-0:~/dockchat# tree
.
├── docker-compose.yml
├── Dockerfile
├── README.md
├── requirements.txt
├── static
│   └── style.css
├── templates
│   └── form_action.html
└── webapp.py

2 directories, 7 files
```

You'll notice a Dockerfile for the '**web**' service and a **docker-compose.yml** that describes the app's service configuration. Take a look at docker-compose.yml file using your favorite text editor:

```
node-0:~/dockchat# cat docker-compose.yml
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
```

You can see that there are two service descriptions: db and web. Each of them define a service with certain build or runtime parameter:


> * **image:{IMAGE_NAME}** will specify the name of the image to use. Note: this needs to be an image that has already been built and available either locally, on Docker on-prem registry, or Docker Hub.

> * **build:{DOCKERFILE_DIR}** will build an image from the Dockerfile that is specified directory. In this case, it's the current directory. In this example, you will use the '**mongo**' image directly from Docker Hub.

> * **command:{CMD}** will specify the command to run in the container when it's created.

> * **ports:{HOST:CONTAINER}** will specify any network host-port mapping 

> * **links:{SERVICE:ALIAS}** will specify container linking parameters. Container linking is a mechanism to automatically provide access, network, and environment paramters for one container to discover and communicate with another contianer. In this example, we are linking the database container '**db**' TO the '**web**' container and giving it the alieas '**db**'.

> * **expose:{PORT}** will specify which port the Docker Engine should map to inside in the container. This is optional if the expose paramter was already declared in the Dockerfile of that service.

Before, running the app, you need to ensure that Docker is running locally and the Docker Compose client can connect to it. To verify, issue the following command and you should expect a similar output:

```
# docker-compose ps
Name   Command   State   Ports
------------------------------
```

You now can build and run the app. It is recommended to ensure that all the images that need to be built be built first before you run them. That is why **docker-compose** has the following two options :`build` and `up`. 

The `build` option builds an image for every service that has a `build:` paramter in the `docker-compose.yml` file but doesn't actually create any container from that image. The `up` option runs the services described in the `docker-compose.yml` file. The `up` option WILL build the services' images before it runs them. It is recommended to ensure that you successfully build all the requried images before running the containers from them as follows:

```
node-0:~/dockchat# docker-compose build
db uses an image, skipping
Building web
....
Successfully built b858d650581e
```

```
node-0:~/dockchat# docker-compose up  
<snippit>
Creating dockchat_db_1
Creating dockchat_web_1
Attaching to dockchat_db_1, dockchat_web_1
db_1  | 2015-11-06T19:24:40.698+0000 I JOURNAL  [initandlisten] journal dir=/data/db/journal
db_1  | 2015-11-06T19:24:40.707+0000 I JOURNAL  [initandlisten] recover : no journal files present, no recovery needed
db_1  | 2015-11-06T19:24:41.102+0000 I JOURNAL  [initandlisten] preallocateIsFaster=true 3.02
web_1 |  * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
web_1 |  * Restarting with stat
```

The output means that you have successfully created the two services. To access the app, go in your web browser to http://<your node IP address>:5000 . You should see somthing like this:

![](images/orchestration-step1-1.png)

Stop the app after you confirm it worked.

` node-0:~/dockchat# docker-compose stop && docker-compose rm -f `


## Step2: Set Up a Docker Swarm cluster

Docker Swarm is native clustering for Docker. It turns a pool of Docker hosts into a single, virtual host.

Swarm serves the standard Docker API, so any tool which already communicates with a Docker daemon can use Swarm to transparently scale to multiple hosts: Dokku, Compose, Krane, Flynn, Deis, DockerUI, Shipyard, Drone, Jenkins... and, of course, the Docker client itself.

There are multiple ways to create a Swarm cluster full details [here](https://github.com/docker/swarm/tree/master/discovery). In this exercise you will use the Docker Hub token-based discovery service. You will create a Swarm Cluster with 2 nodes ( **node-1 and node-2**). The Swarm manager will be a container running on **node-0**. The two Swarm nodes will be labeled for deployment schedluing and filtering purposes later.

Follow the below steps to create a Swarm cluster:

I. Ensure that Docker Engine on **node-1 and node-2** is listening on a TCP port.You can do that by adding the following to `/etc/default/docker` configuration file followed by restarting Docker daemon.  


```
node-0:~#sudo nano /etc/default/docker
# Add the following for node-1:

# Use DOCKER_OPTS to modify the daemon startup options
DOCKER_OPTS="-H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock --label=environment=staging"
```

```
# Add the following for node-2:
DOCKER_OPTS="-H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock --label=environment=production"

```

Then restart Docker on node-0 and node-1.

`node-0:~# sudo service docker restart`

Note: if Docker Engine TLS verification is enabled, it is recommended to use TCP port 2376 instead. For the remainder of these exercises, we will not use Docker TLS verifications.

II. Create a unique Cluster Token on from node-0.  When you run the contianer, it will output a unique cluster token. You will use this token for the remainder of the tutorial. 

`node-0:~#export TOKEN=$(docker run --rm swarm create)`

III. Configure the Swarm Manager on node-0. The manager runs in a container and is configured to use a different TCP port (3375) than engine (2375). 

```node-0:~#docker run -d -p 3375:2375 swarm manage token://$TOKEN```

IV. Register the Swarm agents to the discovery service. Do this step on node-0. The node’s IP must be accessible from the Swarm Manager. Use the following command and replace with the proper node_ip and cluster_id to start an agent.
```
node-0:~#docker run -d swarm join --addr=<node_1_ip:2375> token://$TOKEN
node-0:~#docker run -d swarm join --addr=<node_2_ip:2375> token://$TOKEN
```
Note: Please use node-1 and node-2's private IPs (10.x.x.x) and NOT their public IPs. To be safe, simply use their DNS names instead.

V. Confirm that the two nodes are part of the Swarm cluster:

```
node-0:## docker -H tcp://0.0.0.0:3375 info
Containers: 5
Images: 5
Role: primary
Strategy: spread
Filters: health, port, dependency, affinity, constraint
Nodes: 2
 node-1: ec2-52-29-79-140.eu-central-1.compute.amazonaws.com:2375
  └ Containers: 3
  └ Reserved CPUs: 0 / 1
  └ Reserved Memory: 0 B / 3.859 GiB
  └ Labels: executiondriver=native-0.2, kernelversion=3.19.0-26-generic, operatingsystem=Ubuntu 14.04.3 LTS, storagedriver=aufs
 node-2: ec2-52-29-99-160.eu-central-1.compute.amazonaws.com:2375
  └ Containers: 2
  └ Reserved CPUs: 0 / 1
  └ Reserved Memory: 0 B / 3.859 GiB
  └ Labels: executiondriver=native-0.2, kernelversion=3.19.0-26-generic, operatingsystem=Ubuntu 14.04.3 LTS, storagedriver=aufs
CPUs: 2
Total Memory: 7.718 GiB
Name: 5dc162006656

```
VI. Confirm that the Docker server you're talking to is actually Swarm master ( shown in the Server Version section).


```
# docker -H tcp://0.0.0.0:3375 version
Client:
 Version:      1.9.0
 API version:  1.21
 Go version:   go1.4.2
 Git commit:   76d6bc9
 Built:        Tue Nov  3 17:43:42 UTC 2015
 OS/Arch:      linux/amd64

Server:
 Version:      swarm/1.0.0
 API version:  1.21
 Go version:   go1.5.1
 Git commit:   087e245
 Built:
 OS/Arch:      linux/amd64


```


## Step3: Re-deploy DockChat on the Swarm Cluster

Now that we have a Swarm cluster, we can now re-deploy DockChat on the Swarm Cluster. The first step is to point all Docker client commands at the Swarm manager instead of a single engine. To do so, you need to set `DOCKER_HOST` to point at the Swarm Master's IP and TCP port. Remember that Swarm Master is just a container running and listening on port 3375.


`export DOCKER_HOST={node-0-PrivateIP}:3375`

Currently, **docker-compose** does not support build an image across all Swarm nodes. Therefore, you need to first build the image using the Docker client, tag it, push it to Docker Hub, and use that image in docker-compose.yml file.


```
node-0:~/dockchat# docker login
Username:
WARNING: login credentials saved in /root/.docker/config.json
Login Succeeded
```
```
node-0# docker build -t {Your_Docker_Hub_username}/dockchat:v1 .
...
Successfully built 40a0a638005a
```
```
node-0# docker push {Your_Docker_Hub_username}/dockchat:v1
```
You should see something like:

`e2a4fb18da48: Image successfully pushed`

Now, you would need to change `docker-compose.yml` file to deploy on Swarm. The first thing to change would be the source of the '**web**' image. You would need to change the following using your favorite text editor.

```
(-) build: .
(+) image: {Your_Docker_Hub_username}/dockchat:v1
```

We will also use constraints/affinity rules to ensure Swarm selects the right nodes for deployment. Inititally, we labeled **node 1** as a staging node, and **node 2** as s production node. We can specify in the `docker-compose.yml` file certain constraints to ensure the service is deployed on the environment we specified. Add the following Compose paramters under BOTH the services (**db** and **web**)

```
(+) environment:
(+)   - "constraint:environment==staging"
```

The final Compose file should look like this :

```
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
```
Let's rename the file `staging.docker-compose.yml` as follows:

`mv docker-compose.yml staging.docker-compose.yml`

Now, you need to pull the images on all nodes, and deploy and check the services. But this team need to specify both the custom Docker Compose YAML file **and** a custom project name for this deployment. This is a requirement to ensure unique configuration file to project mapping.

```

node-0:~/dockchat# docker-compose -f staging.docker-compose.yml -p dockchat_staging pull
Pulling db (mongo:latest)...
node-2: Pulling mongo:latest... : downloaded
node-1: Pulling mongo:latest... : downloaded
Pulling web (nicolaka/dockchat:v1)...
node-1: Pulling nicolaka/dockchat:v1... : downloaded
node-2: Pulling nicolaka/dockchat:v1... : downloaded

node-0:~/dockchat# docker-compose -f staging.docker-compose.yml -p dockchat_staging up -d

Creating dockchatstaging_db_1
Creating dockchatstaging_web_1

node-0:~/dockchat# docker-compose -f staging.docker-compose.yml -p dockchat_staging ps
        Name                      Command             State             Ports
---------------------------------------------------------------------------------------
dockchatstaging_db_1    /entrypoint.sh --smallfiles   Up      27017/tcp
dockchatstaging_web_1   python webapp.py              Up      10.0.10.77:5000->5000/tcp


```

You will notice that both containers were deployed on node-1 ( the staging node). This is because we constrained the deployment of both services only on the 'staging' nodes. Also, since the two services are linked, Docker Compose will ensure they are deployed on the same node.

Now you can do the same for the `production` deployment:

`node-0:~/dockchat#cp staging.docker-compose.yml production.docker-compose.yml `

and change the following in the `production.docker-compose.yml`:

```
(-) - "constraint:environment==staging"
(+) - "constraint:environment==production"

```
Deploy again using the `production.docker-compose.yml` :

```
node-0:~/dockchat# docker-compose -f production.docker-compose.yml -p dockchat_production pull
Pulling db (mongo:latest)...
node-2: Pulling mongo:latest... : downloaded
node-1: Pulling mongo:latest... : downloaded
Pulling web (nicolaka/dockchat:v1)...
node-2: Pulling nicolaka/dockchat:v1... : downloaded
node-1: Pulling nicolaka/dockchat:v1... : downloaded

node-0:~/dockchat# docker-compose -f production.docker-compose.yml -p dockchat_production up -d
Creating dockchatproduction_db_1
Creating dockchatproduction_web_1

node-0:~/dockchat# docker-compose -f production.docker-compose.yml -p dockchat_production ps
          Name                       Command             State             Ports
------------------------------------------------------------------------------------------
dockchatproduction_db_1    /entrypoint.sh --smallfiles   Up      27017/tcp
dockchatproduction_web_1   python webapp.py              Up      10.0.11.50:5000->5000/tcp

```

Note that now the two containers were deployed on Node 2, which is the production node.

```
ode-0:~/dockchat# docker ps
CONTAINER ID        IMAGE                  COMMAND                  CREATED              STATUS              PORTS                       NAMES
618b949413e4        nicolaka/dockchat:v1   "python webapp.py"       About a minute ago   Up About a minute   10.0.11.50:5000->5000/tcp   node-2/dockchatproduction_web_1
30e0105aa493        mongo                  "/entrypoint.sh --sma"   About a minute ago   Up About a minute   27017/tcp                   node-2/dockchatproduction_db_1,node-2/dockchatproduction_web_1/db,node-2/dockchatproduction_web_1/db_1,node-2/dockchatproduction_web_1/dockchatproduction_db_1
f77b3a038ba8        nicolaka/dockchat:v1   "python webapp.py"       3 minutes ago        Up 3 minutes        10.0.10.77:5000->5000/tcp   node-1/dockchatstaging_web_1
e136e52638bd        mongo                  "/entrypoint.sh --sma"   3 minutes ago        Up 3 minutes        27017/tcp                   node-1/dockchatstaging_db_1,node-1/dockchatstaging_web_1/db,node-1/dockchatstaging_web_1/db_1,node-1/dockchatstaging_web_1/dockchatstaging_db_1
root@node-0:~/dockchat#
```

Summary: You have successfully used deployed a multi-service app using Docker Compose on both a single engine as well as a Swarm Cluster. 














