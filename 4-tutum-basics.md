# Tutum Hands-on Lab

> **Difficutly**: Beginner

> **Time**: 30 Minutes

> **Prerequisites**: Docker Hub account (Instructions to signup for a free account below), Web Browser
>
> **Tasks** 

> * Link a Docker host to Tutum
* Instantiate and scale Docker containers using Tutum
* Be able to deploy a multi-container application with Tutum

# What is Tutum?

Tutum is a cloud-based system for managing Docker infrastructure that allows users to easily:

- Provision any infrastructure, on-premises or in the cloud, and automatically install, configure, and cluster Docker Engines.

- Deploy Docker containers and Docker Compose-defined applications to the provisioned infrastructure from any registry (including Docker Hub).

- Manage updates and scaling of Dockerized apps and infrastructure through a GUI Dashboard, CLI, and RESTful APIs.

## Before You Begin

In order to complete this lab you will need a Docker Hub account.

If you have an existing Docker Hub account, please proceed to “Attaching a new node” below

**Creating a Docker Hub Account**

Docker hub accounts are completely free, and easy to create.

1. Navigate to http://hub.docker.com

2. On the right hand side of the screen supply a unique username, email address and password

	**Note***: Make sure you remember these credentials as you will need them later*

3. Log into the email account you provided, open the account confirmation email from Docker, and click the link provided in the email.

## Task 1: Attaching a New Node
In Tutum Docker Hosts are referred to Nodes. In this task we will curl the Tutum  agent on to one of our AWS-based docker hosts.

**Logging in to Tutum**

1. Navigate to http://www.tutum.co (note the .co and not .com)

2. Click “Login” in the top right corner of the window

3. Supply your Docker Hub credentials

4. Click “I understand” to dismiss the information about cookie usage

5. Click the X to dismiss Bryan’s welcome message

**Tutum Welcome Tour**

Tutum has a welcome tour, but we won't be using that for this lab. So, if you see the welcome tour pop up, simply click "skip the tour" towards the bottom right of the screen.

**Connecting your Docker Host to Tutum**

Tutum can manage Docker applications on a wide variety of cloud platforms, however fo this lab, we're going to use the bring your own node option. This is not the same thing as linking an AWS account, but it works better for our lab.

1. Click the `Nodes` tab

2. Click `+ Bring Your Own Node` near the top right

3. Select the curl command in the middle of the window and copy it to the clipboard

4. Using the terminal of your choice SSH into node 2

		ssh -i <username>.key ubuntu@<username-node-2 IP Address>

	**Note***: If you get a warning that your private key is unprotected issue the following command* `chmod 700 <username.key>`

	**Note***: If you get a warning about an RSA fingerprint enter `yes`

5. In terminal window to your SSH session, paste the curl command

		ubuntu@node-2:~$ curl -Ls https://get.tutum.co/ | sudo -H sh -s 26d1d6a99c8d4c2494c3b01cec2eb687

	You will see a lot of text scroll buy, but it should end with:

		tutum-agent start/running, process 5602
		-> Done!

		*******************************************************************************
		Tutum Agent installed successfully
		*******************************************************************************

		You can now deploy containers to this node using Tutum

6. Exit out of the SSH session

		ubuntu@node-2:~$ exit
		logout
		Connection to 52.29.14.250 closed.

	**Note***: Your IP Address will be different*

7. In your browser, click the `X` in the top right corner of the Bring Your Own Node window

8. Click the Nodes tab

9. Click on the name of your newly deployed node

10. On the left hand of the screen click `Click to add deploy tags`

11. Type `HOL` and click `Save`

## Task 2: Deploying a Service

A service in Tutum is a collection of containers performing the same role. Using services makes it easier to scale out your applications.

In the steps below we’ll deploy the “hello world” service in a single container to the node we imported previously.

1. Click the Services tab

2. Click “Create your first service”

3. Click “Public Repositories”

On this screen you can search Docker Hub for an image to deploy. There are currently over 185,000 public images available on Docker Hub.

For this lab, we’ll use a Jumpstart. Jumpstarts are repos the Tutum team have created to make it easy to get up and running quickly.

1. Click “Jumpstarts”

2. Click on “Miscellaneous”

3. Click “Select” next to tutum/hello-world repo

4. In Deploy tags type `HOL`

	Note: This is the same deploy tag we just specified for our newly imported 	container

5. Click the grey rectangle over the "Ports" section

6. Click `Publish`

7. Click `dynamic` and enter `80` as the port number

8. Click the green check mark next to the port number

9. Click `Create and Deploy`

10. A screen will pop up for your new service, click `Timeline` and then `Service Start`.

	From here you can see the progress of your service being created

11. After you see the service has successfully started, click `endpoints`

12. Click the arrow next to the service endpoint URL, and your Hello World 	website will launch in a new browser tab

13. Click the appropriate tab to return to Tutum

## Task 3: Deploying a Stack

A stack is a collection of services that make up an application. Just like services are based on Docker Swarm, stacks are based on Docker Compose.

1. Click stacks

2. Click “Create your first stack”

3. Enter `Lab` for the Stack name 

2. Underneath the empty Stackfile text box, click `try ours`

	Tutum will fill in the following Stackfile:

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

	**Note***: Stackfiles have similar syntax to Docker Compose files*

	The first line of the Stackfile specifies the name of our service, in this 	case “Web”

	We’re going to build our web service based on the “tutum/quickstart-python” image.

	We’re going to link the Web service to the Redis service (defined below), and alias the link with the name “Redis”

	We will map port 5000 on the host to port 80 in the container. Meaning people will access our service via port 5000, but Docker will redirect that request to port 80 on the container.

	You can use the “environment:” section to define any environment variables you need.


9. Click “Create and Deploy”

10. A screen will pop up for your new service, click `Timeline` and then `Stack Start`.

	From here you can see the progress of your Stack being created

11. When it's been successfully deployed, Click “Endpoints”

	**Note***: Our app is a simple Python app that will increment a counter each time the page is loaded. The number of visits is stored in a Redis cache*

13. Click the arrow next to the service endpoint. This will open a new browser tab pointing to your newly created service. Refresh the page a few times to see the counter increment.

	**Note***: Service endpoints load balance requests across every container listed under the Container endpoints. They load balance using round robin DNS entries. This simple service only has a single web container, but it is possible to scale your app to have multiple web containers all listed under Container endpoints*

14. Click the appropriate browser tab to return to Tutum

## Conclusion

In this lab you learned how to create a new cluster of Docker Hosts on Amazon Web Services. You then deployed your first service, and then finally you deployed an application stack.

Feel free to continue to explore Tutum.

Try pulling down an Nginx image down from Docker hub, and launching a website (tip: You’ll need to be sure to map the ports correctly).

If you get stuck, ask a moderator or one of your peers for assistance.
