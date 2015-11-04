
# Tutorial 6 : Docker Networking

> **Difficulty**: Advanced

> **Time**: 30-40 mins

> **Prequisites**: 3 Nodes with Engine Installed

> **Tasks**:
> 
> * Set up a key-value store 
> * Configure the engines with the key-value store
> * Create an overlay network
> * Run containers on different hosts and connect them to the overlay network
> * Create another overlay network
> * Reconnect one of the container to the new network
> * Expose host ports in an overlay network.

# Get started with multi-host networking

Background:

Before Docker 1.9, containers running on different hosts had to use the underlying host's TCP/IP stack to communicate between themsleves by mapping a host TCP/UDP port to a container TCP/UDP port. This architecture was limiting and unscalable. In version 1.9, Docker networking was revamped and a new native networking driver was introduced to enable multi-host networking. Mult-host networking enables containers residing on distinct hosts to communicate without relying on the host's TCP/IP stack.

This lab uses an example to explain the basics of creating a mult-host
network. Docker Engine supports this out-of-the-box through the `overlay`
network.  Unlike `bridge` networks overlay networks require some pre-existing
conditions before you can create one. These conditions are:

* Access to a key-value store. Engine supports Consul, Etcd, Zookeeper (Distributed store), and BoltDB (Local store) key-value stores.
* A cluster of hosts with connectivity to the key-value store.
* A properly configured Engine `daemon` on each host in the cluster.


## Prerequisites

* You will use all four nodes : node-0 ,node-1 ,node-2 ,and node-3
* Ensure that no containers are running on these containers.
* Ensure that Docker engine uses default daemon options( `DOCKER_OPTS` should be commented out in `/etc/default/docker` file)
* Ensure that DOCKER_HOST is unset ( `$unset DOCKER_HOST` )
* Certain TCP ports are only allowed on the private AWS network. Therefore, you would need to use the private network (10.X.X.X) when you subsitute the IP of the instance in certain commands/configuration files throughout this lab.


## Step 1: Set up a key-value store
![](images/tut6-step1.png)


An overlay network requires a key-value store. The key-value store information about the network state which includes discovery, networks, endpoints, ip-addresses, and more. Engine supports Consul, Etcd, and Zookeeper (Distributed store) key-value stores. This example uses Consul.

Log into **node-0** and run the Consul container as follows:

`docker run -d -p 8500:8500 -h consul --name consul progrium/consul -server -bootstrap`

Ensure that the container is Up and listening to port 8500

```
# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                                                            NAMES
3092b9afa96c        progrium/consul     "/bin/start -server -"   35 seconds ago      Up 34 seconds       53/tcp, 53/udp, 8300-8302/tcp, 8301-8302/udp, 8400/tcp, 0.0.0.0:8500->8500/tcp   consul
```


## Step 2: Configure the engines to use key-value store

On **node-1,node-2,and node-3**, reconfigure the Docker daemon options listen on TCP port 2375 and to use the Consul k/v store created in Step 1.

To do so, edit `/etc/default/docker` file and ensure to add the following:

`DOCKER_OPTS="-H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock --cluster-store=consul://<NODE-0-PRIVATE-IP>:8500/network --cluster-advertise=eth0:2375"`

Then you need to restart the engine for these settings to take effect:

`sudo service docker restart`

Ensure that the engine restarts successfully:

```
# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

## Step 3: Create the overlay Network

Now that the three nodes are configured to use the k/v store, you can create an overlay network on any node and it will be distributed to all the nodes.

Before we create an overlay network, let's look at the default networks created by Docker:

```
node-1#docker network ls
NETWORK ID          NAME                DRIVER
38ffb13fa56d        none                null
34c7921e8cb7        host                host
25b439c6c8ca        bridge              bridge   
```

Historically, these three networks are part of Docker's implementation. When you run a container you can use the `--net` flag to specify which network you want to run a container on. These three networks are still available to you.

The `bridge` network represents the docker0 network present in all Docker installations. Unless you specify otherwise with the docker run --net=<NETWORK> option, the Docker daemon connects containers to this network by default.

The `none` network adds a container to a container-specific network stack. That container lacks a network interface. 

The `host` network adds a container on the hosts network stack. You'll find the network configuration inside the container is identical to the host.

The new native `overlay` network driver supports multi-host networking natively out-of-the-box. This support is accomplished with the help of `libnetwork`, a built-in VXLAN-based overlay network driver, and Docker's `libkv` library.


Create your `overlay` network called "RED" with the 10.10.10.0/24 subnet.
```
$ docker network create -d overlay --subnet=10.10.10.0/24 RED
```

Check that the network is running:

```bash
node-1#docker network ls
NETWORK ID          NAME                DRIVER
7b1cda01f47f        RED                 overlay
38ffb13fa56d        none                null
34c7921e8cb7        host                host
25b439c6c8ca        bridge              bridge   
```
You can also check that the network was distributed to the other nodes:

```
node-3:~# docker network ls
NETWORK ID          NAME                DRIVER
7b1cda01f47f        RED                 overlay
1ae83a006465        host                host
501f891883cf        bridge              bridge
a141cc346b6c        none                null
```


## Run Containers on the RED network

Once your network is created, you can start a container on any of the hosts and it automatically is part of the network.


On **node-1**, run a simple **ubuntu** container named `container1`. 

`docker run -itd --name container1 --net RED ubuntu`

On **node-2**, run a simple **ubuntu** container named `container2`. 

`docker run -itd --name container2 --net RED ubuntu`


Let's take a look at the network config of container1:

```
node1#docker exec container1 ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:0A:0A:0A:02
          inet addr:10.10.10.2  Bcast:0.0.0.0  Mask:255.255.255.0
          inet6 addr: fe80::42:aff:fe0a:a02/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1450  Metric:1
          RX packets:13 errors:0 dropped:0 overruns:0 frame:0
          TX packets:8 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:1038 (1.0 KiB)  TX bytes:648 (648.0 B)

eth1      Link encap:Ethernet  HWaddr 02:42:AC:12:00:02
          inet addr:172.18.0.2  Bcast:0.0.0.0  Mask:255.255.0.0
          inet6 addr: fe80::42:acff:fe12:2/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:14 errors:0 dropped:0 overruns:0 frame:0
          TX packets:8 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:1128 (1.1 KiB)  TX bytes:648 (648.0 B)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```

You can see that `eth0` was assigned an IP from RED's `10.10.10.0/24` subnet. You will also notice `eth1` with a `172.18.0.0` address. When you create your first overlay network on any host, Docker also creates another network on each host called `docker_gwbridge`. This network is used to provide external access for containers. Every container that is part of an overlay network also gets an `eth` interface in the `docker_gwbridge` to allow it to access the external world. The `docker_gwbridge` is similar to the default `bridge` network, but unlike the `bridge` it restricts inter-container communciation. Docker will create only one
`docker_gwbridge` bridge network per host regardless of the number of overlay networks present. 

Let's take a look at another cool feature of the overlay networks, which is service discovery.

```
node-1:~# docker exec container1 cat /etc/hosts
10.10.10.2	42b58f7ea6dd
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
10.10.10.3	container2
10.10.10.3	container2.RED
```

You can see that Docker took care of adding an entry for each container that is part of the same overlay network. Therefore, to reach container2 from container1, you can simply use its name.





## Related information

* [Docker Swarm overview](https://docs.docker.com/swarm)

