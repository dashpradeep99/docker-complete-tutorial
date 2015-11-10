# Set up your system for the labs

The Docker Hands-on labs require that you connect to four AWS EC2 instances on their own subnetwork. Docker has networked and launched the instances for you. This page explains how to set up your system to connect to these instances for the labs.

When you registered for class, you received an email. Attached to the email you'll an `instances.txt` file and a `.key` PEM file. The `instances.txt` file provides connection information for 4 EC2 instances. The `.key` file gives you the credentials you need to connect them.

## Check your connection to your instances

Before starting the lab, you need to set up your connection key.

1. Download the key file to your machine.

2. Make note of where you download the key.

3. Open or download the `instances.txt` file.

4. Follow the AWS instructions for your operating system to setup the key.

	* <a href="http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/putty.html" target="_blank">Connecting from Windows with PuTTy</a>
	* <a href="http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html" target="_blank">Connecting from Linux or Mac with SSH</a>

5. Connect to each instance in turn to confirm your credentials.


## (Optional) Get a Docker Hub account

A few labs require a Docker Hub to complete them. Docker Hub accounts are completely free. To create one.

1. <a href="http://hub.docker.com" targe="_blank">Open your browser to Docker Hub</a>.

2. On the right hand side of the screen supply a unique username, email address, and password

	Remember these credentials, you'll need them later.

3. Log into the email account you provided to Docker Hub.

4. Open the account confirmation email from Docker.

5. Click the link provided in the email to confirm your account and login.

## Choose a Lab

* [Introduction to Docker](1-docker-introduction.md)
* [Docker Orchestration](2-orchestration.md)
* [Docker Trusted Registry](3-dtr.md)
* [Tutum: Provision, Deploy, Manage](4-tutum-basics.md)
* [Docker Content Trust](5-content-trust.md)
* [Docker Networking](6-networking.md)
* [Understanding Docker Volumes](7-volumes.md)
* [Automated Builds with Docker Hub and GitHub](8-Automated-builds.md)
