# Lab 4: Tutum: Provision, Deploy, Manage

> **Difficulty**: Beginner

> **Time**: 30 Minutes

> **Prerequisites**: Docker Hub account (Instructions to signup for a free account below), Web Browser
>
> **Tasks**

>	- [Task 1: Attaching a New Node](#task-1-attaching-a-new-node)
>	- [Task 2: Deploying a Service](#task-2-deploying-a-service)
>	- [Task 3: Deploying a Stack](#task-3-deploying-a-stack)


# What is Tutum?

Tutum is a cloud-based system for managing Docker infrastructure that allows users to easily:

- Provision any infrastructure, on-premises or in the cloud, and automatically install, configure, and cluster Docker Engines.

- Deploy Docker containers and Docker Compose-defined applications to the provisioned infrastructure from any registry (including Docker Hub).

- Manage updates and scaling of Dockerized apps and infrastructure through a GUI Dashboard, CLI, and RESTful APIs.

## Prerequisites

To complete this lab you will need a Docker Hub account. If you have an existing Docker Hub account, please proceed to “Attaching a new node” below


## Task 1: Attaching a New Node

In Tutum, Docker hosts are referred to Nodes. In this task, you curl the Tutum  agent onto one of you AWS-based docker hosts.

### Logging in to Tutum

1. Navigate to http://www.tutum.co (use `.co` and not `.com` in the address).

2. Click Login in the top right corner of the window

3. Supply your Docker Hub credentials

4. Click I understand to dismiss the information about cookie usage

5. Click the X to dismiss Bryan’s welcome message

		Tutum has a welcome tour, but we won't be using that for this lab. So, if you see the welcome tour pop up, click skip the tour in screen's bottom-right corner.

### Connecting your Docker Host to Tutum

Tutum can manage Docker applications on a wide variety of cloud platforms.
However, for this lab, you're going to use the bring your own node option. This
is not the same thing as linking an AWS account, but it works better for the
lab.

1. Click the Nodes tab.

2. Click Bring Your Own Node near the top right

3. Select the `curl` command in the middle of the window and copy it to the clipboard.

4. Using the terminal of your choice, SSH into `node-2`.

		ssh -i <username>.key ubuntu@<username-node-2 IP Address>

	> **Note**: If you get a warning that your private key is unprotected issue
	> the following command:

	>  `chmod 700 <username.key>	`

	> You may also get a warning about an RSA fingerprint. If you do, enter `yes`.

5. In terminal window to your SSH session, paste the `curl` command

		ubuntu@node-2:~$ curl -Ls https://get.tutum.co/ | sudo -H sh -s 26d1d6a99c8d4c2494c3b01cec2eb687

	You should see a lot of text scroll by ending with:

		tutum-agent start/running, process 5602
		-> Done!

		*******************************************************************************
		Tutum Agent installed successfully
		*******************************************************************************

		You can ready to deploy containers to this node using Tutum.

6. Exit out of the SSH session

		ubuntu@node-2:~$ exit
		logout
		Connection to 52.29.14.250 closed.

	>**Note**: Your IP Address will be different*

7. In your browser, click the `X` in the top right corner of the Bring Your Own Node window

8. Click the Nodes tab.

9. Click on the name of your newly deployed node.

10. On the left hand of the screen press Click to add deploy tags.

11. Type `HOL` and press Save.

## Task 2: Deploying a Service

A service in Tutum is a collection of containers performing the same role. Using
services makes it easier to scale out your applications. In the steps below,
you'll deploy the “hello world” service in a single container to the node you
imported previously.

1. Click the Services tab

2. Click Create your first service

3. Click Public Repositories.

	Use this screen to search Docker Hub for an image to deploy. There are currently over 185,000 public images available on Docker Hub. For this lab, you'll use a Jumpstart. Jumpstarts are repos the Tutum team have created to make it easy to get up and running quickly.

1. Click Jumpstarts.

2. Click on Miscellaneous.

3. Click Select next to `tutum/hello-world` repo.

4. In Deploy tags type `HOL`.

	This is the same deploy tag you just specified for you newly imported 	container.

5. Click the grey rectangle over the Ports section.

6. Click Publish.

7. Click `dynamic` and enter `80` for the port number.

8. Click the green check mark next to the port number.

9. Click Create and Deploy.

   The system displays a dialgo for your new service.

10. Click `Timeline` and then `Service Start`.

	From here you can see the progress of the service creation.

11. After you see the service has successfully started, click endpoints.

12. Click the arrow next to the service endpoint URL.

	Your Hello World website launches in a new browser tab.


## Task 3: Deploying a Stack

A stack is a collection of services that make up an application. Just like services are based on Docker Swarm, stacks are based on Docker Compose.

1. If you haven't already done so, return to Tutum in your browser.

2. Click stacks.

2. Click Create your first stack.

3. Enter `Lab` for the Stack name.

2. Underneath the empty Stackfile text box, click try ours.

	Tutum creates the following Stackfile:

		web:
  			image: tutum/quickstart-python
  			links:
   				 - "redis:redis"
  			ports:
    			- "5000:80"
		redis:
  			image: tutum/redis
  			environment:
    			- REDIS_PASS=password

	>**Note**: Stackfiles have similar syntax to Docker Compose files.

	The first line of the Stackfile specifies the name of our service, in this 	case `web`. You’re going to build a web service based on the `tutum/quickstart-python` image. You'll link the `web` service to the `redis` service (defined below), and alias the link with the name “Redis."

	You will map port `5000` on the host to port `80` in the container. This mapping allows users to access your service via port `5000`, but Docker  redirect that request to container's port `80`.

	You can use the “environment:” section to define any environment variables you need.

9. Click Create and Deploy.

 	Tutum displays pop up for your new service.

10. Click Timeline and then Stack Start.

	From here you can see the progress of your Stack being created

11. When this stack is successfully deployed, click “Endpoints”

	The application you created is a simple Python app that increments a counter each time the page is loaded. The number of visits is stored in a Redis cache.

13. Click the arrow next to the service endpoint.

	This opens a new browser tab pointing to your newly created service.

14. Refresh the page a few times to see the counter increment.

	Service endpoints load balance requests across every container listed under the Container endpoints. They load balance using round robin DNS entries. This simple service only has a single web container, but it is possible to scale your app to have multiple web containers all listed under Container endpoints*

## Conclusion

In this lab you learned how to create a new cluster of Docker Hosts on Amazon Web Services. You then deployed your first service, and then finally you deployed an application stack. Feel free to continue to explore Tutum.

Try pulling down an Nginx image down from Docker hub, and launching a website (Hint: You’ll need to be sure to map the ports correctly). If you get stuck, ask a moderator or one of your peers for assistance.
