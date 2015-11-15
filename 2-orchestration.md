
# Lab 2 : Docker Orchestration

> **Difficulty**: Advanced

> **Time**: 30-40 mins

> **Tasks**:
> 
>* [Prerequisites](#prerequisites)
> * [Task 1: Deploy a Multiservice Application (`DockChat`) with Compose](#task-1-deploy-a-multiservice-application-on-a-single-engine)
> * [Task 2: Set Up a Docker Swarm cluster](#task-2-set-up-a-docker-swarm-cluster)
> * [Task 3: Re-deploy DockChat on the Swarm Cluster](#task-3-re-deploy-dockchat-on-the-swarm-cluster)
> * [Task 4: Scale DockChat with Compose and Interlock](#task-4-scale-dockchat-with-interlock)

## Prerequisites

* You will be using **node-0,node-1,and node-2**
* Ensure that no containers are running on these nodes(`$ docker rm -f $(docker ps -q)`)
* Ensure that Docker engine uses default daemon options( `DOCKER_OPTS` should be commented out in `/etc/default/docker` file)
* Ensure that DOCKER_HOST is unset ( `$ unset DOCKER_HOST`)
* Certain TCP ports are only allowed on the private AWS network. Therefore, you would need to use the private network (10.X.X.X) when you subsitute the IP of the instance in certain commands/configuration files throughout this lab.
* Note: Some commands may require `sudo`. 

# Getting Started with Docker Compose and Swarm

**Background**:

Docker Compose and Docker Swarm are key elements of the Docker Ecosystem. Docker Compose is an open-source tool to orchestrate building and deploying multi-service,multi-container applications. Compose relies on a configuration YAML file (default name is **docker-compose.yml**) to build, create, and run containers against a single Docker Engine or Docker Swagrm cluster. The YAML configuration file has standard reference commands ( full list [here](https://docs.docker.com/compose/yml/) ) that  define how the container is created at runtime. Some of the most common commands used are: **build, image, command, ports, and links**. Compose should be already installed on your machine. To verify, issue the following command on all-nodes and ensure you see the currently installed version.

`node-0$ docker-compose version`

In this tutorial, you will go through deploying **DockChat**, the new awesome chat app. First you'll use Docker Compose to deploy it on a single Docker engine. Then you will create a Swarm cluster with staging and production engines. You'll use the Swarm cluster to deploy DockChat. Finally, you'll scale the app using **Interlock**, the event-driven Docker plugin system.


## Task 1: Deploy a Multiservice Application on a single Engine




In this step, you will deploy a multiservice app, **DockChat**, that is composed of two services : **web** and **db**. DockChat is a simple Python+Mongo chat server.


**Step 1:** First step is to clone the DockChat repo from GitHub on **node-0**

` node-0$ git clone https://github.com/nicolaka/dockchat.git `

**Step 2:** Change directory to dockchat, and examine the list of files in the repo.

```
node-0:~$ cd dockchat/
node-0:~/dockchat$ tree
.
├── docker-compose.yml
├── Dockerfile
├── README.md
├── requirements.txt
├── static
│   └── style.css
├── templates
│   └── form_action.html
└── webapp.py

2 directories, 7 files
```

You'll notice a Dockerfile for the '**web**' service and a **docker-compose.yml** that describes the app's service configuration. Take a look at docker-compose.yml file using your favorite text editor:

```
node-0:~/dockchat$ cat docker-compose.yml
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

You can see that there are two service descriptions: db and web. Each of them define a service with certain build on runtime parameter:


> * **image:{IMAGE_NAME}** will specify the name of the image to use. Note: this needs to be an image that has already been built and available either locally, on Docker Trusted Registry, or Docker Hub.

> * **build:{DOCKERFILE_DIR}** will build an image from the Dockerfile that is specified directory. In this case, it's the current directory. In this example, you will use the '**mongo**' image directly from Docker Hub.

> * **command:{CMD}** will specify the command to run in the container when it's created.

> * **ports:{HOST:CONTAINER}** will specify any network host-port mapping

> * **links:{SERVICE:ALIAS}** will specify container linking parameters. Container linking is a mechanism to automatically provide access, network, and environment paramters for one container to discover and communicate with another contianer. In this example, we are linking the database container '**db**' TO the '**web**' container and giving it the alieas '**db**'.

> * **expose:{PORT}** informs Docker that the container listens on the specified network ports at runtime.

**Step 3:** Before, running the app, you need to ensure that Docker is running locally and the Docker Compose client can connect to it. To verify, issue the following command and you should expect a similar output:

```
$ docker-compose ps
Name   Command   State   Ports
------------------------------
```

**Step 4:** You now can build and run the app. It is recommended to ensure that all the images that need to be built, be built first before you run them. That is why **docker-compose** has the following two options :`build` and `up`.

The `build` option builds an image for every service that has a `build:` paramter in the `docker-compose.yml` file but doesn't actually create any container from that image. The `up` option runs the services described in the `docker-compose.yml` file. The `up` option WILL build the services' images before it runs them. It is recommended to ensure that you successfully build all the requried images before running the containers from them as follows:

```
node-0:~/dockchat$ docker-compose build
db uses an image, skipping
Building web
....
Successfully built b858d650581e
```

```
node-0:~/dockchat$ docker-compose up  
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

The output means that you have successfully created the two containers. To access the app, go in your web browser to http://{your node-0 public IP address}:5000 . You should see somthing like this:

![](images/orchestration-step1-1.png)

**Step 5:** Stop the app after you confirm it worked by pressing **CTRL + C** . Then remove the two containers by typing:

` node-0:~/dockchat$ docker-compose stop && docker-compose rm -f `


## Task 2: Set Up a Docker Swarm cluster

**Docker Swarm** is native clustering for Docker. It turns a pool of Docker hosts into a single, virtual host.

Swarm serves the standard Docker API, so any tool which already communicates with a Docker daemon can use Swarm to transparently scale to multiple hosts: Dokku, Compose, Krane, Flynn, Deis, DockerUI, Shipyard, Drone, Jenkins... and, of course, the Docker client itself.

There are multiple ways to create a Swarm cluster full details [here](https://github.com/docker/swarm/tree/master/discovery). In this exercise you will use the Docker Hub token-based discovery service. You will create a Swarm Cluster with 2 nodes ( **node-1 and node-2**). The Swarm manager will be a container running on **node-0**. The two Swarm nodes will be labeled for deployment schedluing and filtering purposes later.

Follow the below steps to create a Swarm cluster:

**Step 1:** Ensure that Docker Engine on **node-1 and node-2** is listening on a TCP port.You can do that by adding the following to `/etc/default/docker` configuration file followed by restarting Docker daemon. We're also adding the **staging** and **production** engine labels that are used by the Swarm scheduler at deployment.


```
$ sudo nano /etc/default/docker
# Add the following for node-1:

# Use DOCKER_OPTS to modify the daemon startup options
DOCKER_OPTS="-H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock --label=environment=staging"
```

```
# Add the following for node-2:
DOCKER_OPTS="-H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock --label=environment=production"

```

**Step 2:** Then restart Docker on **node-1 and node-2**.

`$ sudo service docker restart`

Note: if Docker Engine TLS verification is enabled, it is recommended to use TCP port 2376 instead. For the remainder of these exercises, we will not use Docker TLS verifications.

**Step 3:** Create a unique cluster token on from node-0.  When you run the contianer, it will output a unique cluster token. You will use this token for the remainder of the tutorial.

`node-0:~$export TOKEN=$(docker run --rm swarm create)`

**Step 4:** Configure the Swarm Manager on node-0. The manager runs in a container and is configured to use a different TCP port (3375) than engine (2375).

```node-0:~$docker run -d -p 3375:2375 swarm manage token://$TOKEN```

**Step 5:** Register the Swarm agents to the discovery service. Do this step on node-0. The node’s IP must be accessible from the Swarm Manager. Use the following command and replace with the proper node_ip to start an agent.
```
node-0:~$docker run -d swarm join --addr=<node_1_ip:2375> token://$TOKEN
node-0:~$docker run -d swarm join --addr=<node_2_ip:2375> token://$TOKEN
```
Please use node-1 and node-2's private IPs (10.x.x.x) and NOT their public IPs. To be safe, simply use their **DNS names instead**.

**Step 6:**  Confirm that the two nodes are part of the Swarm cluster:

```
node-0:$ docker -H tcp://0.0.0.0:3375 info
Containers: 0
Images: 0
Role: primary
Strategy: spread
Filters: health, port, dependency, affinity, constraint
Nodes: 2
 node-1: ************.compute.amazonaws.com:2375
  └ Containers: 3
  └ Reserved CPUs: 0 / 1
  └ Reserved Memory: 0 B / 3.859 GiB
  └ Labels: executiondriver=native-0.2, kernelversion=3.19.0-26-generic, operatingsystem=Ubuntu 14.04.3 LTS, storagedriver=aufs
 node-2: *****************.eu-central-1.compute.amazonaws.com:2375
  └ Containers: 2
  └ Reserved CPUs: 0 / 1
  └ Reserved Memory: 0 B / 3.859 GiB
  └ Labels: executiondriver=native-0.2, kernelversion=3.19.0-26-generic, operatingsystem=Ubuntu 14.04.3 LTS, storagedriver=aufs
CPUs: 2
Total Memory: 7.718 GiB
Name: 5dc162006656

```

**Step 7:**  Confirm that the Docker server you're talking to is actually Swarm master ( shown in the server version section).


```
$ docker -H tcp://0.0.0.0:3375 version
Client:
 Version:      1.9.0
 API version:  1.21
 Go version:   go1.4.2
 Git commit:   76d6bc9
 Built:        Tue Nov  3 17:43:42 UTC 2015
 OS/Arch:      linux/amd64

Server:
 Version:      swarm/1.0.0 <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
 API version:  1.21
 Go version:   go1.5.1
 Git commit:   087e245
 Built:
 OS/Arch:      linux/amd64


```


## Task 3: Re-deploy DockChat on the Swarm Cluster

Now that we have a Swarm cluster, you can now re-deploy DockChat on the Swarm Cluster.

**Step 1 :** The first step is to point all Docker client commands at the Swarm manager instead of a single local engine on **node-0**. To do so, you need to set `DOCKER_HOST` to point at the Swarm Master's IP and TCP port. Remember that Swarm Master is just a container running and listening on port 3375.

`export DOCKER_HOST={node-0-PrivateIP}:3375`

**Step 2 :** Currently, **docker-compose** does not support build an image across all Swarm nodes. Therefore, you need to first build the image using the Docker client, tag it, push it to Docker Hub, and use that image in docker-compose.yml file.


```
node-0:~/dockchat$ docker login
Username:
WARNING: login credentials saved in /root/.docker/config.json
Login Succeeded
```
```
node-0$ docker build -t {Your_Docker_Hub_username}/dockchat:v1 .
...
Successfully built 40a0a638005a
```
```
node-0$ docker push {Your_Docker_Hub_username}/dockchat:v1
```
You should see something like:

`e2a4fb18da48: Image successfully pushed`

**Step 3 :** Now, you would need to change `docker-compose.yml` file to deploy on Swarm. The first thing to change would be the source of the '**web**' image. You would need to change the following using your favorite text editor.

```
(-) build: .
(+) image: {Your_Docker_Hub_username}/dockchat:v1
```

**Step 4 :** We will also use constraints/affinity rules to ensure Swarm selects the right nodes for deployment. Inititally, we labeled **node 1** as a staging node, and **node 2** as s production node. We can specify in the `docker-compose.yml` file certain constraints to ensure the service is deployed on the environment we specified. Add the following Compose paramters under **BOTH** the services (**db** and **web**)

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

**Step 5:** Let's rename the file `staging.docker-compose.yml`. It is recommended to have unique Docker Compose files for each deployment for better version control and deployment isolation. For example, you can easily integrate a CICD workflow that deploys to **staging** first using the `staging.docker-compose.yml` then if certain tests succeed deploy to the **production** nodes.

`mv docker-compose.yml staging.docker-compose.yml`

**Step 6:** Now, you need to pull the images on all nodes, and deploy and check the services. But this time need to specify both the custom Docker Compose YAML file **and** a custom project name for this deployment. This is a requirement to ensure unique configuration file to project mapping.

```

node-0:~/dockchat$ docker-compose -f staging.docker-compose.yml -p dockchat_staging pull
Pulling db (mongo:latest)...
node-2: Pulling mongo:latest... : downloaded
node-1: Pulling mongo:latest... : downloaded
Pulling web (nicolaka/dockchat:v1)...
node-1: Pulling nicolaka/dockchat:v1... : downloaded
node-2: Pulling nicolaka/dockchat:v1... : downloaded

node-0:~/dockchat$ docker-compose -f staging.docker-compose.yml -p dockchat_staging up -d

Creating dockchatstaging_db_1
Creating dockchatstaging_web_1

node-0:~/dockchat$ docker-compose -f staging.docker-compose.yml -p dockchat_staging ps
        Name                      Command             State             Ports
---------------------------------------------------------------------------------------
dockchatstaging_db_1    /entrypoint.sh --smallfiles   Up      27017/tcp
dockchatstaging_web_1   python webapp.py              Up      10.0.10.77:5000->5000/tcp


```

Using `docker ps`'s NAMES column, you'll notice that both containers were deployed on **node-1** (the staging node). This is because we constrained the deployment of both services only on the **'staging'** nodes. Also, since the two services are linked, Docker Compose will ensure they are deployed on the same node.

**Step 7:** Do the same for the **production** deployment:

`node-0:~/dockchat$cp staging.docker-compose.yml production.docker-compose.yml `

and change the following in the `production.docker-compose.yml`:

```
(-) - "constraint:environment==staging"
(+) - "constraint:environment==production"

```
**Step 8:** Deploy again using the `production.docker-compose.yml` :

```
node-0:~/dockchat$ docker-compose -f production.docker-compose.yml -p dockchat_production pull
Pulling db (mongo:latest)...
node-2: Pulling mongo:latest... : downloaded
node-1: Pulling mongo:latest... : downloaded
Pulling web (nicolaka/dockchat:v1)...
node-2: Pulling nicolaka/dockchat:v1... : downloaded
node-1: Pulling nicolaka/dockchat:v1... : downloaded

node-0:~/dockchat$ docker-compose -f production.docker-compose.yml -p dockchat_production up -d
Creating dockchatproduction_db_1
Creating dockchatproduction_web_1

node-0:~/dockchat$ docker-compose -f production.docker-compose.yml -p dockchat_production ps
          Name                       Command             State             Ports
------------------------------------------------------------------------------------------
dockchatproduction_db_1    /entrypoint.sh --smallfiles   Up      27017/tcp
dockchatproduction_web_1   python webapp.py              Up      10.0.11.50:5000->5000/tcp

```

Using `docker ps`'s NAMES column, note that now the two containers were deployed on Node 2, which is the production node.

```
ode-0:~/dockchat$ docker ps
CONTAINER ID        IMAGE                  COMMAND                  CREATED              STATUS              PORTS                       NAMES
618b949413e4        nicolaka/dockchat:v1   "python webapp.py"       About a minute ago   Up About a minute   10.0.11.50:5000->5000/tcp   node-2/dockchatproduction_web_1
30e0105aa493        mongo                  "/entrypoint.sh --sma"   About a minute ago   Up About a minute   27017/tcp                   node-2/dockchatproduction_db_1,node-2/dockchatproduction_web_1/db,node-2/dockchatproduction_web_1/db_1,node-2/dockchatproduction_web_1/dockchatproduction_db_1
f77b3a038ba8        nicolaka/dockchat:v1   "python webapp.py"       3 minutes ago        Up 3 minutes        10.0.10.77:5000->5000/tcp   node-1/dockchatstaging_web_1
e136e52638bd        mongo                  "/entrypoint.sh --sma"   3 minutes ago        Up 3 minutes        27017/tcp                   node-1/dockchatstaging_db_1,node-1/dockchatstaging_web_1/db,node-1/dockchatstaging_web_1/db_1,node-1/dockchatstaging_web_1/dockchatstaging_db_1
root@node-0:~/dockchat$
```
You can check out the app by going to http://{node-1-PUBLIC IP}:5000 for the **staging** deployment and to http://{node-2-PUBLIC IP}:5000 for the **production** deployment.

## Task 4:Scale DockChat with Interlock

**DockChat** is becoming very popular and it's getting a lot of traffic! You need to scale it to accomodate the surge in the traffic. Good news is that you can use Compose to scale the **web** service to handle more traffic in the **production** deployment. Scaling a service that does host-port mapping ( **web**'s TCP Port 5000) will not work because you can only have a single service listen on the host's port 5000. Thereofre, you will need to remove host-port mapping to avoid this problem. But then how will you direct traffic to the contaier? Here comes **Interlock** which is an event-driven plugin that can register new containers to a service.

In this example, you'll use Interlock to dynamically register your **web** service containers to an HAProxy load-balancing service. This way any container that is part of the **web** service can receive traffic directed at that service. Interlock can be easily deployed as a service in Compose!

**Step 1:** Stop the production deployment  ( I know that sounds silly :) but it is needed because we need to add certain environment vars to the **web** service containers.

```
node-0:~/dockchat$ docker-compose -f production.docker-compose.yml -p dockchat_production stop
Stopping dockchatproduction_web_1 ... done
Stopping dockchatproduction_db_1 ... done
```
**Step 2:** Edit the production.docker-compoe.yml to:

 * Add Interlock service using the `ehazlett/interlock:latest` image.

 * Add an JSON-formatted hostname environment variable inside the **web** service containers. This is needed for Interlock to discover these services (`- INTERLOCK_DATA={"hostname":"dockchat.com","domain":"dockchat.com"} `).

 * Remove host-port mapping for the **web** service ( add `"5000"` instead of `"5000:5000"`).


Your configruation file should look like:

```
# Mongo DB
db:
  image: mongo
  expose:
   - 27017
  command: --smallfiles
  environment:
   - "constraint:environment==production"
# Python App
web:
  image: nicolaka/dockchat:v1
  ports:
   - "5000"
  links:
   - db:db
  environment:
   - "constraint:environment==production"
   - INTERLOCK_DATA={"hostname":"dockchat.com","domain":"dockchat.com"}
interlock:
  image: ehazlett/interlock:latest
  ports:
    - "80:80"
  volumes:
    - /var/lib/docker:/etc/docker
  command: "--swarm-url tcp://$DOCKER_HOST --debug --plugin haproxy start"
```
**Step 3:** Re-deploy the service and check that all containers are up:

```
node-0:~/dockchat$ docker-compose -f production.docker-compose.yml -p dockchat_production pull
Pulling db (mongo:latest)...
node-2: Pulling mongo:latest... : downloaded
node-1: Pulling mongo:latest... : downloaded
Pulling web (nicolaka/dockchat:v1)...
node-2: Pulling nicolaka/dockchat:v1... : downloaded
node-1: Pulling nicolaka/dockchat:v1... : downloaded
Pulling interlock (ehazlett/interlock:latest)...
node-2: Pulling ehazlett/interlock:latest... : downloaded
node-1: Pulling ehazlett/interlock:latest... : downloaded


node-0:~/dockchat$ docker-compose -f production.docker-compose.yml -p dockchat_production up -d
Creating dockchatproduction_db_1
Creating dockchatproduction_web_1
Creating dockchatproduction_interlock_1

node-0:~/dockchat$ docker-compose -f production.docker-compose.yml -p dockchat_production ps
             Name                           Command               State               Ports
--------------------------------------------------------------------------------------------------------
dockchatproduction_db_1          /entrypoint.sh --smallfiles      Up      27017/tcp
dockchatproduction_interlock_1   /usr/local/bin/interlock - ...   Up      443/tcp, 10.0.11.50:80->80/tcp
dockchatproduction_web_1         python webapp.py                 Up      10.0.11.50:32769->5000/tcp

```

**Step 4:** To test the app from your browser, you need to add a new entry in your local `/etc/hosts` pointing to `dockchat.com` as follows:

```
mylaptop$ vim /etc/hosts

(+) {PUBLIC_IP_OF_NODE-2}   dockchat.com
```

Then go to `dockchat.com` from your browser and test if it works.

**staging** Finally, we need to scale the app, and that can be done easily with the `docker-compose scale` funcationality.

```
node-0:~/dockchat$ docker-compose -f production.docker-compose.yml -p dockchat_production scale web=10

Creating and starting 2 ... done
Creating and starting 3 ... done
Creating and starting 4 ... done
Creating and starting 5 ... done
Creating and starting 6 ... done
Creating and starting 7 ... done
Creating and starting 8 ... done
Creating and starting 9 ... done
Creating and starting 10 ... done


node-0:~/dockchat$ docker-compose -f production.docker-compose.yml -p dockchat_production ps

             Name                           Command               State               Ports
--------------------------------------------------------------------------------------------------------
dockchatproduction_db_1          /entrypoint.sh --smallfiles      Up      27017/tcp
dockchatproduction_interlock_1   /usr/local/bin/interlock - ...   Up      443/tcp, 10.0.11.50:80->80/tcp
dockchatproduction_web_1         python webapp.py                 Up      10.0.11.50:32769->5000/tcp
dockchatproduction_web_10        python webapp.py                 Up      10.0.11.50:32771->5000/tcp
dockchatproduction_web_2         python webapp.py                 Up      10.0.11.50:32770->5000/tcp
dockchatproduction_web_3         python webapp.py                 Up      10.0.11.50:32775->5000/tcp
dockchatproduction_web_4         python webapp.py                 Up      10.0.11.50:32777->5000/tcp
dockchatproduction_web_5         python webapp.py                 Up      10.0.11.50:32774->5000/tcp
dockchatproduction_web_6         python webapp.py                 Up      10.0.11.50:32773->5000/tcp
dockchatproduction_web_7         python webapp.py                 Up      10.0.11.50:32772->5000/tcp
dockchatproduction_web_8         python webapp.py                 Up      10.0.11.50:32776->5000/tcp
dockchatproduction_web_9         python webapp.py                 Up      10.0.11.50:32778->5000/tcp

```

Now if you go back to your browser, you'll see that with every refresh you're being served from a different **web** container.

Note: you can visit **dockchat.com/haproxy?stats** and log in with `stats/interlock` to view the HAProxy stats for each container.

## Conclusion

Congrats! You have completed the tutorial!

In this tutorial you used Docker Compose and Docker Swarm to deploy and scale a multi-service app.

Interested in scaling **DockChat** horizontally across multiple nodes ? You can do that with the new multi-host networking introduced with Docker 1.9. Check out this [this repo](https://github.com/nicolaka/dockchat-multihost).

### Share on Twitter!

<p>
<a href="http://ctt.ec/IMUFf" target=“_blank”>
<img src="http://www.wyntercon.com/wp-content/uploads/2015/04/twitter-bird-blue-on-white-small.png" width="100" height="100">
</p>

## Clean up

If you plan to do another lab, you need to cleanup your EC2 instances. Cleanup removes any environment variables, configuration changes, Docker images, and running containers. To do a clean up, log into each EC2 instance and run the following:

```bash
$ source /home/ubuntu/cleanup.sh
```

## Related information

* [Docker Swarm ](https://www.docker.com/docker-swarm)
* [Docker Compose ](https://www.docker.com/docker-compose)
* [Dockchat Repo](https://github.com/nicolaka/dockchat)
