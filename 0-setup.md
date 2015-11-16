# Set up your system for the labs

The Docker Hands-on labs require that you connect to four AWS EC2 instances on their own sub-network. Docker has networked and launched the instances for you. This page explains how to set up your system to connect to these instances for the labs.

When you registered for labs, you received an email. Attached to the email you'll find:

* an `instances.txt` file
* a `.key` PEM file (Linux and Mac)
* a `.ppk` file (Windows)

The `instances.txt` file provides connection information for 4 EC2 instances. The `.key` file gives you the credentials you need to connect them.

**WARNING**: Your EC2 instances are good for an entire conference day. At the end of the conference day, everyone's instances EC2 are deleted. If you want to continue your lab experience on another day, you'll need to register again on the next conference day.


## Check your connection to your instances

Before starting the lab, you need to set up your connection key.

1. Download the key file to your machine.

2. Make note of where you download the key.

3. Open or download the `instances.txt` file.

4. Follow the AWS instructions for your operating system to setup the key:

	* For Mac, use the `chmod` command to make sure your private key file isn't publicly viewable. For example, 		if the name of your private key file is my-key-pair.pem, use the following command:
	
	`chmod 400 /path/my-key-pair.pem`

	Then, login using `ssh -i /path/my-key-pair.pem ubuntu@*********.compute-1.amazonaws.com`
	
	* For Windows, Follow the instructions below.You can skip the step *"Converting Your Private Key Using PuTTYgen*."" The PPK file attached 	        to your email is in the right format for PuTTy.
	   

	<a href="http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html" 	target="_blank">Connecting from Linux or Mac with SSH</a>

5. Log in to each instance using the **`ubuntu`** username and ssh key.

## Join the Chat Room

We have set up a chat room for the Hands-On Labs. Please go to [here](http://app.lets-chat-dceu-hol.nicolaka752c19b5407142b8.svc.tutum.io/#!/). Sign up with a new account and join the **dceu** room.

## (Optional) Get a Docker Hub account

Labs **1**, **2**, **4** and **8** require a Docker Hub account to complete them. Docker Hub accounts are completely free. To create one:

1. <a href="http://hub.docker.com" target="_blank">Open your browser to Docker Hub</a>.

2. On the right hand side of the screen supply a unique username, email address, and password

	Remember these credentials, you'll need them later.

3. Log into the email account you provided to Docker Hub.

4. Open the account confirmation email from Docker.

5. Click the link provided in the email to confirm your account and login.

	You can't log into Docker Hub until you first confirm your credentials.

## Choose a Lab

* [Introduction to Docker](1-docker-introduction.md)
* [Docker Orchestration](2-orchestration.md)
* [Docker Trusted Registry](3-dtr.md)
* [Tutum: Provision, Deploy, Manage](4-tutum-basics.md)
* [Docker Content Trust](5-content-trust.md)
* [Docker Networking](6-networking.md)
* [Understanding Docker Volumes](7-volumes.md)
* [Automated Builds with Docker Hub and GitHub](8-Automated-builds.md)
