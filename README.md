# DockerCon EU 2015 Hands-On Labs (HOL)

![dceu](http://europe-2015.dockercon.com/assets/Docker_Euro_Largelogo-1c4ec95d91a66c91f44c831a65d2147d.png)

Welcome to DockerCon! This repo contains a series of hands-on labs to help you gain experience in various Docker features, products, and solutions. Depending on your experience, each lab requires between 15-30 minutes to complete. They range in difficulty from easy to advanced.

If you are at DockerCon EU 2015, you must register before beginning the labs. When you register, Docker Labs provisions the Amazon EC2 instances with the software you need for each lab. You'll receive an email with connection information. Begin by following this procedure here:

**[Set up your system for the labs](0-setup.md)**

Then, go onto to complete the labs.

## Contribute Your Own Labs

If you have an awesome tutorial/lab and would like to add it here. Please open a PR. We would love to add more exciting tutorials to the list!


## Lab 1. [Introduction to Docker](1-docker-introduction.md)
**Difficulty**: Easy

A basic introduction to Docker operations. You learn about Docker Engine image search, pulling and pushing images, creating new images from Dockerfiles, and running and troubleshooting containers.


## Lab 2. [Docker Orchestration Tools](2-orchestration.md)
**Difficulty**: Intermediate

You'll explore Docker Orchestration Tools ( mainly Swarm and Compose). You'll learn how to set up a Swarm cluster using the token discovery option, how Compose works ( including Compose files best practices), and how to deploy and troubleshoot multi-service, multi-container application across the Swarm cluster.

## Lab 3. [Docker Trusted Registry](3-dtr.md)
**Difficulty**: Easy

In this lab, we will examine Docker Trusted Registry (DTR). You'll go through DTR installation, authentication setup, storage backend configuration, monitoring, security, and other various enterprise-level supported features.


## Lab 4. [Tutum: Provision, Deploy, Manage](4-tutum-basics.md)
**Difficulty**: Intermediate

The goal of this lab is help you gain an understanding of Docker’s SaaS-based management platform, Tutum. By the end of the lab you should:

* understand what Tutum is, and the basic capabilities it provides
* know how to create a Docker host in AWS using Tutum
* instantiate and scale Docker containers using
* be able to deploy a multi-container application with Tutum


## Lab 5. [Docker Content Trust](5-content-trust.md)
**Difficulty**: Magic

In this lab, you'll examine how Docker Notary feature works. You'll be creating signed images, distributing them through Docker Hub, and ensuring that only trusted images can be distributed in your environment.



## Lab 6. [Docker Networking](6-networking.md)
**Difficulty**: Advanced

In this lab, you'll examine how the new Docker multi-host networking works. You'll create networks that spread across multiple Docker engines. You'll also learn how to create key/value stores to distribute network data across the hosts, and how to allow multi-host containers to communicate without host port binding.


## Lab 7. [Understanding Docker Volumes](7-volumes.md)
**Difficulty**: Intermediate

In this lab, you'll learn the fundamentals of using Docker data volumes. You'll look at the various methods for attaching data volumes, and how they are represented in the host file system. You'll also look at how to use host-based data volumes to hot mount content into a container, and how to use host-only data containers to allow for sharing of common data between multiple containers.


## Lab 8. [Automated Builds with Docker Hub and GitHub](8-Automated-builds.md)
**Difficulty**: Intermediate

In this lab, you'll examine how to use GitHub and Docker hub to create a basic continuous delivery workflow. You'll learn how to integrate the two systems together such that when a change is pushed the GitHub repository, an image is automatically built, and pushed to Docker Hub.

