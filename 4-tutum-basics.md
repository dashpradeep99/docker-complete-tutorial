Tutum Hands-on Lab

Objective: The goal of this lab is help you gain an understanding of Docker’s SaaS-based management platform, Tutum. 

By the end of the lab you should:
- Understand what Tutum is, and the basic capabilities it provides
- Know how to create a Docker host in AWS using Tutum
- Instantiate and scale Docker containers using Tutum
- Be able to deploy a multi-container application with Tutum

Time: It’s estimated that this lab will take 30 minutes to complete

**What is Tutum?**

Tutum is a cloud-based system for managing Docker infrastructure that allows users to easily:

- Provision any infrastructure, on-premises or in the cloud, and automatically install, configure, and cluster Docker Engines.

- Deploy Docker containers and Docker Compose-defined applications to the provisioned infrastructure from any registry (including Docker Hub).

- Manage updates and scaling of Dockerized apps and infrastructure through a GUI Dashboard, CLI, and RESTful APIs.

**Before You Begin**

In order to complete this lab you will need a Docker Hub account. 

If you have an existing Docker Hub account, please proceed to “Logging in to Tutum” below

**Creating a Docker Hub Account**

Docker hub accounts are completely free, and easy to create. 

1. Navigate to http://hub.docker.com

2. On the right hand side of the screen supply a unique username, email address and password

Note: Make sure you remember these credentials as you will need them later

3. Log into the email account you provided, open the account confirmation email from Docker, and click the link provided in the email. 

**Logging in to Tutum**

1. Navigate to http://www.tutum.co (note the .co and not .com)

2. Click “Login” in the top right corner of the window

3. Supply your Docker Hub credentials

4. Click “I understand” to dismiss the information about cookie usage

5. Click the X to dismiss Bryan’s welcome message

**Tutum Welcome Tour**

For this lab we are going to work through the Tutum welcome tour. When you log in for the first time you should be taken to the Welcome Tour screen. 

If you are not seeing the Welcome Tour screen: Click on your account name in the upper right, and choose Welcome Tour from the drop down. 

**Linking an AWS Account**

Tutum can manage Docker applications on a wide variety of cloud platforms (and even has a bring your own node option in case you want to use a platform that’s not explicitly provided). 

For this lab we’re going to be using Amazon Web Service as our cloud provider. 

1. On the Welcome Tour screen click “Add Your First Cloud Provider”

2. Read the first two message balloons, clicking “Next” on each of them

3. Click “+Add Credentials” on the Amazon Web Services line

4. Enter your supplied credentials and click “Save Credentials”

5. Click “Next” on the third message balloon

6. Click “Done” on the final message balloon

**Deploying Your First Node**

Tutum allows you to deploy new nodes easily via a web-based graphical user interface. In the case of our lab, a node cluster is one or more new EC2 instances of the same region and type acting as Docker hosts in AWS. 

1. From the Welcome Screen click on “Deploying your first node”

2. Follow the instructions on the three Message Balloons and supply a Node Cluster Name (test-node) and a Deploy Tag (test)

Note: There are several other options available for your node cluster including Region, VPC, Subnet, etc. You can also specify the number of nodes to create, and the size of the disk. 

For this lab, we’ll only create one node, and we’ll leave the rest of the settings at their defaults. 

3. Click “Launch Node Cluster”

4. Your node will begin deploying (it can take up to 10 minutes for a node to deploy).

5. Click “Done” on the message balloon to return to the Welcome Screen. 


Note: Once your node is deployed, click “Create Your First Service” on the welcome screen to continue the tour. 

**Deploying a Service**

A service in Tutum is a collection of containers performing the same role. Using services makes it easier to scale out your applications. 

Note: Tutum Services provides a graphical front end to Docker Swarm.

In the steps below we’ll deploy the “hello world” service in a single container to the node cluster we created previously. 

1. Click “Create your first service” 

2. Click “Public Repositories”

On this screen you can search Docker Hub for an image to deploy. There are currently over 185,000 public images available on Docker Hub. 

For this lab, we’ll use a Jumpstart. Jumpstarts are repos the Tutum team have created to make it easy to get up and running quickly. 

3. Click “Jumpstarts”

4. Click on “Miscellaneous”

5. Click “Select” next to tutum/hello-world repo

6. Read and click “Next” on the first four message balloons

7. Enter “test” for the the Deploy Tag, and click Next in the message balloon.

Note: This is the same deploy tag we used when we created our node cluster.

8. Read and click “Next” on the subsequent message balloons to expose port 80

Note: Like we saw when we deployed our node cluster, there are several options we could set when deploying a new service. Feel freeto hover over some of the names to understand what the different options do, but for this lab we won’t be changing any of the defaults. 

9. Click “Create and Deploy”

10. You will see your Hello World service being deployed

11. Click “Next” on the message balloon

12. Click “Endpoints” to access your newly created service

13. Click “Next” two times

14. Click the arrow next to the service URL and your Hello World website will launch in a new browser tab

15. Click the appropriate tab to return to Tutum

16. Click “Done” in the message balloon. 

**Creating a Stack**

A stack is a collection of services that make up an application. Just like services are based on Docker Swarm, stacks are based on Docker Compose. 

1. Click “Create your first stack”

2. Click “Next” and the welcome tour will fill in a Stackfile for you. 

Note: Stackfiles have similar syntax to Docker Compose files.

3. Click “Next”

4. The first line of the Stackfile specifies the name of our service, in this case “Web” 

Click “Next”

5. We’re going to build our web service based on the “tutum/quickstart-python” image.


Click “Next”

6. We’re going to link the Web service to the Redis service (defined below), and alias the link with the name “Redis”

Click “Next”

7. We will map port 5000 on the host to port 80 in the container. Meaning people will access our service via port 5000, but Docker will redirect that request to port 80 on the container. 

Click “Next”

8. You can use the “environment:” section to define any environment variables you need 

Click “Next”

9. Click “Create and Deploy”

Note: Your stack is now launching. This may take a minute or two.

10. Read the next two message balloons, clicking “Next” after each one

11. Click “Endpoints”

Our app is a simple Python app that will increment a counter each time the page is loaded. The number of visits is stored in a Redis cache. 

12. Click “Next” in the next two message balloons

Note: It will take a minute or two for your service deployment to complete. You cannot move on to step 13 until it does
               
13. Click the  Screen Shot 2015-10-27 at 11.05.07 AM.png next to the service endpoint. This will open a new browser tab pointing to your newly created service. Refresh the page a few times to see the counter increment. 

Note: Service endpoints load balance requests across every container listed under the Container endpoints. They load balance using round robin DNS entries. This simple service only has a single web container, but it is possible to scale your app to have multiple web containers all listed under Container endpoints.

14. Click the appropriate browser tab to return to Tutum

15. Click the “Welcome” tab in the upper left of the window to return to the Welcome Tour home screen

**Repositories**

Repositories are collections of Docker images (databases, web servers, etc). These images can serve as a starting point to building your application

1. Click “Learn more about repositories”

2. Read the first two message balloons and click “Next”

3. Click the “Welcome” tab in the top left portion of the screen

4. Read the next three message balloons and click “Next”

5. Read the final message balloon and click “Done”

Creating repositories is beyond the scope of this lab, but we wanted to make sure you were aware of the functionality. 

**Conclusion** 

In this lab you learned how to create a new cluster of Docker Hosts on Amazon Web Services. You then deployed your first service, and then finally you deployed an application stack. 

Feel free to continue to explore Tutum. 

Try pulling down an Nginx image down from Docker hub, and launching a website (tip: You’ll need to be sure to map the ports correctly). 

If you get stuck, ask a moderator or one of your peers for assistance.
