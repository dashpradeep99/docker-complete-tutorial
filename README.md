# DockerCon EU 2015 Hands-On Labs (HOL)

**Background**

Welcome to DockerCon! In this repo, you will find a series of hands-on labs to help you gain experience in various Docker features, products, and solutions. Each lab should run between 15-30 minutes each and range form introductory to advanced in difficulty.

**Setup**

You should have received an email with cloud-based setup and access info. The setup is composed of (4) VMs each running Docker Engine. 

**Labs**

**Tutorial 1**: **Intro to Docker**

Description: This lab goes over basic intro-level Docker operations. We will go over Docker engine installation, image search, pulling and pushing images, creating new images from Dockerfiles, and running and troubleshooting containers.

Difficulty: Easy
Time: 20 mins
Goal: Introduce Docker basic operations, run basic containers, and interact with Docker Hub
Setup Requirements: engine, git

Tasks:

* Environment (Version/Info)
* Images( Search/Pull)
* Building Images from Dockerfile
* Running Containers ( Run  / PS)
* Maintaining Containers ( logs/events)
* Docker Hub (Tag/Push)


**Tutorial 2 : Docker Orchestration Tools**

Description: In this lab we will explore Docker Orchestration Tools ( mainly Swarm and Compose). We will go through how to set up a Swarm cluster using the token discovery option, how Compose works ( including Compose files best practices), and how to deploy and troubleshoot multi-service, multi-container application across the Swarm cluster. 

Difficulty: Intermediate
Time: 30 mins
Goal: Create a Swarm cluster and deploy a multi-service application on it using Docker Compose.


Tasks: 

* Setup Requirements: engine, git, compose
* Running a multi-service app with Compose on a single engine
* Creating a Swarm cluster
* Running a multi-service app with Compose on a Swarm cluster
* Scaling the app with Interlock

Tutorial 3: Docker Trusted Registry - DTR (Owner: Nicola Kabar)
Description: In this lab we will examine Docker Trusted Registry (DTR). We will go through DTR installation, authentication setup, storage backend configuration, monitoring, security, and other various enterprise-level supported features.
Attendees 
Status: 
Link to Lab Guide: 
Difficulty: Easy
Time: 15 mins
Goal: Install,Setup, and perform basic operations with Docker Trusted Registry
Tasks: 
Setup Requirements: engine, git
Install latest version of DTR
Setup DNS, authentication, and licenses
Trust self-signed TLS certificates
Push an image to DTR

Tutorial 4: Tutum Test Drive  (Owner: Mike Coleman)
Description: 
The goal of this lab is help you gain an understanding of Docker’s SaaS-based management platform, Tutum.

By the end of the lab you should:
 
Understand what Tutum is, and the basic capabilities it provides
Know how to create a Docker host in AWS using Tutum
Instantiate and scale Docker containers using
Be able to deploy a multi-container application with Tutum
 
Status: Guide completed, currently being tested
Link to Lab Guide: https://docs.google.com/document/d/1zmbaPm1A-9j3DVNKXTvGi7Q1WlOymAG1T6eUPPv3mOQ/edit?usp=sharing
Difficulty: Intermediate
Time: 30
Goal: Provide a basic overview of Tutum by walking customers through the welcome tour
Tasks: Link AWS account, deploy a Docker host (node cluster), create a container (service), deploy an application (stack)
Setup Requirements: Users need to have a Docker Hub account. We need to provide AWS credentials, including access key and secret key

Tutorial 5: Docker Content Trust  (Owner: Jerry Baker)
Description: In this lab we'll examine how Docker Notary feature works. We will be creating signed images, distributing them through Docker Hub, and ensuring that only trusted images can be distributed in our environment.
Status:
Link to Lab Guide: 
Difficulty: Easy
Time: 15 mins
Goal: Sign,pull and push signed images
Tasks: 
Setup Requirements: 

Tutorial 6: Docker Networking (Owner: Nicola Kabar)
Description: In this lab we'll examine how the new Docker multi-host networking works. We will create networks that spread across multiple Docker engines, how to create key/value stores to distribute network data across the hosts, and how to allow multi-host containers to communicate without host port binding.
 Status:
Link to Lab Guide: 
Difficulty: Advanced
Time: 30 mins
Goal: Understand the new networking features of Engine 1.9
Tasks:
Setup Requirements: engine, git, compose
Create multi-host networks
Deploy a multi-container application
FIXME

Tutorial 7: Understanding Docker Data Volumes (Owner: Mike Coleman)
Description: In this lab students will learn the fundamentals of using Docker data volumes. We will look at the various methods for attaching data volumes, and how they are represented in the host file system. We'll also look at how to use host-based data volumes to hot mount content into a container, and how to use host-only data containers to allow for sharing of common data between multiple containers.
Status: Lab guide has been tested, working on updating based on feedback
Link to Lab Guide: https://docs.google.com/document/d/1Mkd_PU0AFRG05o7-DwFApGjMMyMna0-5JO5dbr59mC4/edit?usp=sharing
Difficulty: Intermediate
Time: 30 mins
Goal: Understand the different scenarios for dealing with persistent data with Docker
Tasks: 
Setup Requirements: Docker engine 1.9 RC1+

Tutorial 8: Automated Builds with Docker Hub and GitHub  (Owner: Mike Coleman)
Description: In this lab we'll examine how to use GitHub and Docker hub to create a basic continuous delivery workflow. Attendees will learn how to integrate the two systems together such that when a change is pushed the GitHub repository, an image is automatically built, and pushed to Docker Hub. 
Status:
Difficulty: Intermediate
Time: 20 mins
Goal: Create a Continuous Integration use case with Docker Hub Automated Builds
Tasks:
Setup Requirements: Engine, Git, Git Hub account, Docker Hub Account
Fork and Clone the Docker-Demo repo
Create a Docker Hub Automated Repo based on the Github Repo
Make changes to the repo to create a new image in Docker Hub
