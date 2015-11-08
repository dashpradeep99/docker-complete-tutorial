# Lab 2 : Docker Orchestration

> **Description**: In this lab you explore Docker Swarm and Docker Compose which are Docker's orchestration tools.

> **Difficulty**: Intermediate

> **Time**: 20-30 mins

> **Prequisites**: 3 Nodes with Docker CS 1.9 Engine

> **Tasks**:
>	- [Task 1: Deploy a Multiservice Application on a single Engine](#task-1-deploy-a-multiservice-application-on-a-single-engine)
>	- [Task 2: Prepare your nodes for Swarm](#task-2-prepare-your-nodes-for-swarm)
>	- [Task 3: Create a Swarm](#task-3-create-a-swarm)
>	- [Task 4: Re-deploy DockChat on the Swarm Cluster](#task-4-re-deploy-dockchat-on-the-swarm-cluster)


# Learn a bit about Docker Compose and Swarm

Docker Swarm and Docker Compose are key elements of the Docker open source ecosystem. Docker Swarm is native clustering for Docker. It turns a pool of Docker hosts into a single, virtual host. Swarm serves the standard Docker API, so any tool which already communicates with a Docker daemon can use Swarm to transparently scale to multiple hosts: Dokku, Compose, Krane, Flynn, Deis, DockerUI, Shipyard, Drone, Jenkins... and, of course, the Docker client itself.

Docker Compose orchestrates building and deploying multi-service,multi-container applications. Compose relies on a configuration YAML file to build, create, and run containers on a single Docker Engine or across Docker Swarm cluster.

The compose configuration file is named `docker-compose.yml` by default. The
file has a [well-defined syntax](http://docs.docker.com/compose/compose-file/) that directs how the container is configured for runtime. Some common Compose options are `build`, `image`, `command`, `ports`, and `links`.

## Prerequisites

You will be using `node-0`,`node-1`, and `node-2`. Compose should be already installed on these machines. To verify the nodes are configured correctly, login into each node in turn and do the following.:

1. Check that Compose is installed.

        # docker-compose version

2. Ensure no containers are running and stop any containers that are.

        # docker ps

3. Check that Docker Engine uses default daemon options by ensuring `DOCKER_OPTS` is commented out in the `/etc/default/docker` file.

        # cat /etc/default/docker

4. Ensure that `DOCKER_HOST` is unset.

        # unset DOCKER_HOST

Finally, certain TCP ports are only allowed on a private AWS network. Therefore,
throughout this lab,  you must use the private network (10.X.X.X) when you
substitute the IP of the instance in certain commands/configuration files
throughout this lab.


## Task 1: Deploy a Multiservice Application on a single Engine

In this step, you  deploy a multi-service application, `DockChat`, is a simple Python+Mongo chat server. It is composed a `web` and a `db` service. This lab uses two `docker-compose` subcommands :`build` and `up`.

The `build` command creates an image for every service that has a `build` option in the `docker-compose.yml` file. The `build` does not create any container.

The `up` command runs the services described in the `docker-compose.yml` file. If any prerequisites images require a `build` the `up` command builds them before running the container.

Clone the DockChat application and deploy the services with Compose.

1. SSH into `node-0`.

2. Change to your home directory.

        # cd ~

3. Clone the DockChat repo from GitHub.

        # git clone https://github.com/nicolaka/dockchat.git

4. Change to `dockchat` directory.

        # cd dockchat/

5. List of files in the repo.

        # tree
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

    You'll notice a Dockerfile for the `web` service and a `docker-compose.yml` that describes the app's service configuration.

6. Open the `docker-compose.yml` file using your favorite text editor.

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

    You can see that there are two service descriptions: `db` and `web`. Each defines a service with certain build or runtime parameter:

    * `image:{IMAGE_NAME}` Specifies the image name to use. The image must have already been built and available either locally, on Docker on-prem registry, or on Docker Hub. This example uses the '`mongo`' image directly from Docker Hub.
    * `build:{DOCKERFILE_DIR}` Builds an image from the Dockerfile in the specified directory. In this lab, it's the current directory.
    * `command:{CMD}` Specifies the command to run in the container when it's created.
    * `ports:{HOST:CONTAINER}` Specifies a network host-port mapping
    * `links:{SERVICE:ALIAS}` Specifies container linking parameters. Container linking is a mechanism to automatically provide access, network, and environment parameters for one container to discover and communicate with another. In this example, you are linking the database container '`db`' to the '`web`' container and giving it the alias '`db`'.
    * `expose:{PORT}` Specifies which port the Docker Engine should map to inside in the container. This is optional if the `EXPOSE` parameter was already declared in the Dockerfile of that service.

7. Verify that Docker is running locally and the Docker Compose client can connect to it.

        # docker-compose ps
        Name   Command   State   Ports
        ------------------------------

8. Build the services with the `build` command.

      You should successfully build all the required application images before you run them.

        # docker-compose build
        db uses an image, skipping
        Building web
        ....
        Successfully built b858d650581e

9. Use the `up` command to launch the services.

        # docker-compose up  
        <snippit>
        Creating dockchat_db_1
        Creating dockchat_web_1
        Attaching to dockchat_db_1, dockchat_web_1
        db_1  | 2015-11-06T19:24:40.698+0000 I JOURNAL  [initandlisten] journal dir=/data/db/journal
        db_1  | 2015-11-06T19:24:40.707+0000 I JOURNAL  [initandlisten] recover : no journal files present, no recovery needed
        db_1  | 2015-11-06T19:24:41.102+0000 I JOURNAL  [initandlisten] preallocateIsFaster=true 3.02
        web_1 |  * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
        web_1 |  * Restarting with stat

    The output means that you have successfully created the two services.

10. Access the DockChat application from your web browser: `http://<your node IP address>:5000`

    You should see the following:

    ![](images/orchestration-step1-1.png)

11. Stop the application after you confirm it is working.

        # docker-compose stop && docker-compose rm -f


## Task 2: Prepare your nodes for Swarm

There are [multiple ways to create a Swarm
cluster](https://github.com/docker/swarm/tree/master/discovery). In this lab,
you use the Docker Hub token-based discovery service. You create a Swarm
Cluster with 2 nodes ( `node-1` and `node-2`). The Swarm manager is a container
running on `node-0`. The two Swarm nodes are labeled for deployment scheduling
and filtering purposes later.

Each node’s IP must be accessible from the Swarm manager. This section prepares your nodes to communicate across the same port. If Docker Engine TLS verification were enabled, it is recommended to use TCP port 2376 instead. For the remainder of these exercises, you are not using Docker TLS verification. So, this exercise ensures all Docker Engine on each node is listening on a TCP port `2375`.  

Prepare your nodes:

1. Log into `node-0`.
2. Edit the `/etc/default/docker` configuration file.
3. Add the following:

        # Use DOCKER_OPTS to modify the daemon startup options
        DOCKER_OPTS="-H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock --label=environment=staging"

4. Save and close the file.

5. Log into `node-1` and repeat the steps 1-4 above.

6. Log into `node-2`.

7. Edit the `/etc/default/docker` configuration file.

8. Add the following:

        # Add the following for node-2:
        DOCKER_OPTS="-H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock --label=environment=production"

9. Close and save the file.

10. Restart Docker on node-0 and node-1.

        # sudo service docker restart

## Task 3: Create a Swarm

When you create a Swarm by running a container with the `docker run --swarm create` option. The command returns a unique cluster token. You use this token for the remainder of the lab so, you'll store it in an environment variable.

1. Login into `node-0`.

2. Create the cluster and store the token in your environment.

        # export TOKEN=$(docker run --rm swarm create)`

  The Swarm manager runs in a container and is configured to use a different TCP port (3375) than Engine (2375).

3. Start the manager with your token.

        # docker run -d -p 3375:2375 swarm manage token://$TOKEN

4. Register the Swarm agents to the discovery service.

    Use the following command and replace with the proper node_ip and cluster_id to start an agent.

        # docker run -d swarm join --addr=<node_1_ip:2375> token://$TOKEN
        # docker run -d swarm join --addr=<node_2_ip:2375> token://$TOKEN

    >**Note**: Use `node-1` and `node-2`'s private IPs (`10.x.x.x`) and NOT their public IPs. To be safe, simply use their DNS names.

5. Confirm that the two nodes are part of the Swarm cluster:

        # docker -H tcp://0.0.0.0:3375 info
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

6. Confirm that the Docker server you're talking to is actually Swarm master ( shown in the Server Version section).

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


## Task 4: Re-deploy DockChat on the Swarm Cluster

Now that you have a Swarm cluster, you can re-deploy DockChat on the swarm.  Currently, `docker-compose` does not support build an image across all Swarm nodes. Therefore, you need to first build the image using the Docker client, tag it, push it to Docker Hub, and use that image in `docker-compose.yml` file.

Instead of running commands on a single Docker client, you direct all Docker client commands at the Swarm manager. Remember that Swarm Master is just a container running and listening on port 3375.


1. Log into `node-0` where you last ran Compose.

2. Set `DOCKER_HOST` to point at the Swarm Master's IP and TCP port.

        # export DOCKER_HOST={node-0-PrivateIP}:3375

3. Change directory to the `dockchat` directory where you cloned the repository.

        # cd dockchat

4. Log into the Docker Hub with your credentials.

        # docker login
        Username:
        WARNING: login credentials saved in /root/.docker/config.json
        Login Succeeded

5. Build the application with Docker Engine.

        # docker build -t {Your_Docker_Hub_username}/dockchat:v1 .
        ...
        Successfully built 40a0a638005a

6. Push the image to Docker Hub.

        # docker push {Your_Docker_Hub_username}/dockchat:v1
        e2a4fb18da48: Image successfully pushed

  Next, you change your Compose configuration to deploy on your swarm.

7. Open `docker-compose.yml` file with your favorite editor.  

8.  Change the `web` service to pull your image from Docker Hub by replace the `build` option with the `image` option.

        web:
          image: {Your_Docker_Hub_username}/dockchat:v1

    Constraints and affinity rules ensure Swarm selects the right nodes for
    deployment. Earlier, you labeled `node 1` as a `staging` node, and `node 2`
    as s `production` node.

9. Add `constraint` options to each service so they are deployed on the `staging` nodes.

        environment:
          constraint:environment==staging

10. Check you `docker-compose.yml` file make sure it looks like this:

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

11. Save and close the file.

    You need to specify a custom Docker Compose configuration file for your deployment instead of using the default name.

12. Rename the `docker-compose.yml` file to `staging.docker-compose.yml`.

        # mv docker-compose.yml staging.docker-compose.yml

13. Pull the images on all nodes making sure to specify a custom project name for your deployment.

     A custom project name is required to ensure unique configuration file to project mapping.

        # docker-compose -f staging.docker-compose.yml -p dockchat_staging pull
        Pulling db (mongo:latest)...
        node-2: Pulling mongo:latest... : downloaded
        node-1: Pulling mongo:latest... : downloaded
        Pulling web (nicolaka/dockchat:v1)...
        node-1: Pulling nicolaka/dockchat:v1... : downloaded
        node-2: Pulling nicolaka/dockchat:v1... : downloaded

14. Start your project an deploy it.

        # docker-compose -f staging.docker-compose.yml -p dockchat_staging up -d

        Creating dockchatstaging_db_1
        Creating dockchatstaging_web_1

15. Check the processes running for Compose.

        # docker-compose -f staging.docker-compose.yml -p dockchat_staging ps
                Name                      Command             State             Ports
        --------------------------------------------------------------------------------------
        dockchatstaging_db_1    /entrypoint.sh --smallfiles   Up      27017/tcp
        dockchatstaging_web_1   python webapp.py              Up      10.0.10.77:5000->5000/tcp

    Notice that both containers were deployed on `node-1` ( the `staging` node). This is because you constrained the deployment of both services only on the 'staging' nodes. Also, since the two services are linked, Docker Compose ensures they are deployed on the same node.

16. Repeat the process for the `production` nodes.

        # cp staging.docker-compose.yml production.docker-compose.yml `

17. Edit the `production.docker-compose.yml` and change the constraints form `staging` to `production`.

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
          image: <your_Docker_Hub_username>/dockchat:v1
          ports:
            - "5000:5000"
          links:
           - db:db
          environment:
           - "constraint:environment==production"

18. Deploy again using the `production.docker-compose.yml` :

        # docker-compose -f production.docker-compose.yml -p dockchat_production pull
        Pulling db (mongo:latest)...
        node-2: Pulling mongo:latest... : downloaded
        node-1: Pulling mongo:latest... : downloaded
        Pulling web (nicolaka/dockchat:v1)...
        node-2: Pulling nicolaka/dockchat:v1... : downloaded
        node-1: Pulling nicolaka/dockchat:v1... : downloaded

        # docker-compose -f production.docker-compose.yml -p dockchat_production up -d
        Creating dockchatproduction_db_1
        Creating dockchatproduction_web_1

        # docker-compose -f production.docker-compose.yml -p dockchat_production ps
                  Name                       Command             State             Ports
        ------------------------------------------------------------------------------------------
        dockchatproduction_db_1    /entrypoint.sh --smallfiles   Up      27017/tcp
        dockchatproduction_web_1   python webapp.py              Up      10.0.11.50:5000->5000/tcp

    Note that now the two containers were deployed on Node 2, which is the
    production node.

19. List the containers on `node-0`

        # docker ps
        CONTAINER ID        IMAGE                  COMMAND                  CREATED              STATUS              PORTS                       NAMES
        618b949413e4        nicolaka/dockchat:v1   "python webapp.py"       About a minute ago   Up About a minute   10.0.11.50:5000->5000/tcp   node-2/dockchatproduction_web_1
        30e0105aa493        mongo                  "/entrypoint.sh --sma"   About a minute ago   Up About a minute   27017/tcp                   node-2/dockchatproduction_db_1,node-2/dockchatproduction_web_1/db,node-2/dockchatproduction_web_1/db_1,node-2/dockchatproduction_web_1/dockchatproduction_db_1
        f77b3a038ba8        nicolaka/dockchat:v1   "python webapp.py"       3 minutes ago        Up 3 minutes        10.0.10.77:5000->5000/tcp   node-1/dockchatstaging_web_1
        e136e52638bd        mongo                  "/entrypoint.sh --sma"   3 minutes ago        Up 3 minutes        27017/tcp                   node-1/dockchatstaging_db_1,node-1/dockchatstaging_web_1/db,node-1/dockchatstaging_web_1/db_1,node-1/dockchatstaging_web_1/dockchatstaging_db_1
        root@node-0:~/dockchat#

## Conclusion

Congratulations, You have successfully used deployed a multi-service app using Docker Compose on both a single Docker Engine and on a Swarm Cluster.

Interested in scaling `DockChat` horizontally across multiple nodes? You can do that with the new multi-host networking introduced with Docker 1.9. Check out this [this repo](https://github.com/nicolaka/dockchat-multihost).

## Cleanup

If you are doing additional labs, please make sure to cleanup the current lab's containers on all nodes as follows:

```
docker rm -f $(docker ps -aq)
```
Additionally, remove the `DOCKER_OPTS` settings from `/etc/default/docker` on each node.


## Related information

* [Docker Swarm ](https://www.docker.com/docker-swarm)
* [Docker Compose ](https://www.docker.com/docker-compose)
* [Dockchat Repo](https://github.com/nicolaka/dockchat)
