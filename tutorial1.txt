Tutorial 1 : Intro to Docker
Difficulty: Easy
Time: 20 mins

Tasks:
Environment (Version/Info)
Images( Search/Pull)
Building Images from Dockerfile
Running Containers ( Run  / PS)
Maintaining Containers ( logs/events)
Docker Hub (Tag/Push)


Docker is an open platform for building, shipping and running distributed applications. It gives programmers, development teams and operations engineers the common toolbox they need to take advantage of the distributed and networked nature of modern applications. This tutorial will go through a number of introductory Docker tasks to introduce you to Docker concepts and operations.

The Docker platform is composed of multiple pieces, mainly they are the Docker Engine, Docker Client, and Docker Hub. Docker Engine is primarily responsible for building images and running containers. When you install Docker on your machine, both the Docker Engine and the Docker Client get installed. The Docker Client is one of the many different ways that you can communicate with the Docker Engine. Since Docker is built using a client-server architecture, you can communicate with the engine using standard API calls over HTTP/HTTPS. Docker Hub is an online registry to distribute Docker images. Docker should be installed on your machine. 

Docker Engine and Client should be installed and running in your setup. To verify that Docker is running, log in as 'ubuntu' and issue the following command:

$docker version 
Client:
 Version:      1.8.2
 API version:  1.20
 Go version:   go1.4.2
 Git commit:   0a8c2e3
 Built:        Thu Sep 10 19:19:00 UTC 2015
 OS/Arch:      linux/amd64

Server:
 Version:      1.8.2
 API version:  1.20
 Go version:   go1.4.2
 Git commit:   0a8c2e3
 Built:        Thu Sep 10 19:19:00 UTC 2015
 OS/Arch:      linux/amd64

1. Docker Environment

 If Docker is running properly, you should see both Client and Server version details. In this step, we issued a Docker Client command which communicated over the local Unix socket that the Docker Engine is listening to (by default it is /var/run/docker.sock and is owned by root). Try to issue the following commands and observe the output:

 - docker info  -> shows various info on Docker engine settings. Info like toal number of running containers, cached images, storage backend, operating system, and root directory are some of the info displayed with this command.
 - docker ps    -> shows currently running containers. You should see none as there are none currently running.
 - docker images -> shows all available images. You should see none because we have not pulled/built any images yet.

2. Images

Docker containers are constructed by sequentially mounting a set of file systems from one or more images. A Docker image is a file system layer with an optional parent image reference. 
All containers that are constructed from the same image should be identical. You can run containers from images that you built or that you pulled from teh Docker Hub. Docker Hub has millions of images that are created and shared by regular users. Additioanlly, Docker Hub hosts official images ( e.g Ubuntu,Redis, and Mongo) that are created and maintained by their respective companies. 


Try to issue the following commands and observe the output:

- docker search ubuntu -> Search Docker Hub for all images that include the word "ubuntu" in them.
- docker pull ubuntu:latest   -> Pull the image "ubuntu". 
- docker images        -> Display the local images. Notice that now you have the ubuntu image.

Here we are using the Docker Client to search Docker Hub for all images that contain the word 'ubuntu' in them. You can search Docker Hub from the web if you go to www.hub.docker.com.
You'll notice that the search results will indicate if the image is official. Official images are certified by Docker to indicate that they are built using certain standards. A list of all official images can be found here : https://github.com/docker-library/official-images/tree/master/library. 

When we performed the 'docker pull ubuntu:latest' command, we requested to pull the official ubuntu image. 'latest' is a tag used to identify the specific version of the image to pull. If the image name and tag are found in Hub, then each of the layers of that image are downloaded and cached locally. You can verify which images you have downloaded if you issue the 'docker images' command.


3. Building Images from Dockerfile

There are two ways to create a Docker Image:

	I.  Update a container created from an image and commut the results to a new image
	II. Use a Dockerfile to specify instructions to create an image.

Although the first option seems to be simpler, it makes it harder to track all changes that were committed in the new container once they're committed into a new image. The most common way used to create new images is the second option. A Dockerfile is a text file that contains all the commands, in order, needed to build a given image. The primary advtange of using Dockerfile is version control and tracibility of all steps that were used to create a particular image. Dockerfile adhere to a specific format and use a specific set of instructions to build an image. Some of the most common commands used in Dockerfile are FROM , RUN , WORKDIR, EXPOSE, and CMD. A complete guide to Dockerfiles can be found here : https://docs.docker.com/articles/dockerfile_best-practices/ .


In the following task, you will be building a new image from a Dockerfile. The image will use the ubuntu:latest image that you just downloaded as a base image. 


- Create a new files in your local directory and name it Dockerfile.
- Start editing your file using your preferred text editor ( vim,vi, emacs, nano..etc)
- Add the following to the file:

FROM ubuntu:latest 
RUN apt-get install -y emacs  
RUN mkdir /mydir
ENV HOSTNAME mycontainer 
WORKDIR  /mydir  
CMD ["/bin/bash"]  

- Save and Exit
- Run 'docker build -t myimage:v1 .'


Image details: 

FROM ubuntu:latest  -> This image is based on ubuntu:latest
RUN apt-get install -y emacs   -> installs emacs text editor using apt-get
RUN mkdir /mydir  -> creates a directory called mydir under root directory
ENV HOSTNAME mycontainer -> creates an environment variable called HOSTNAME with a value set to 'mycontainer'
WORKDIR  /mydir -> /mydir directory is the working directory for the image
CMD ["/bin/bash"]  -> issues /bin/bash when a container is created from this image

The build process: the Docker Engine goes through the Dockerfile instructions one line at a time. Each line in creating what is called an 'intermediate' image by creating a container from the previous line, running the current command, and commiting that container into a new image. The process is repeated until the last command is sucessfully committed into an image. The last image is the final product of the build process and in our case is tagged with 'myimage:v1'. 

- Run 'docker images' to observe that now you have a new image



4. Running Containers

In this step, you will create and run a new container from the image built in Step #3. 
To do so, issue the following:

- docker run -it myimage:v1

The '-it' flags allows you to connect to the STDIN of the container's main process ( in our case, it was /bin/bash) using a pseudo tty connection that Docker engine creates. Notice that you're currently connected
to the container with a hostname of 'mycontainer' and the current working directory is set to '/mydir'.  Verify that 'emacs' is installed and '/mydir' was created by issuing:

- which emacs
- ls -la / | grep mydir

The container will run as long as PID 1 ( in your case it is /bin/bash ) is running. The moment you issue
a CTRL+C to exit the container, the container is stopped. To keep the container running , issue the following:

- CTRL + P + Q

The container is still running. You may verify using:

- docker ps

which will show the Container ID, image , creation time, status, any mapped ports, and container name of running containers ONLY. In order to see all containers ( running and stopped), you can issue the following:

- docker ps -a

Unless you specify a name for the container when you create it with the '--name' flag, Docker will generate a random name for it.

5. Maintaining Containers

Docker provides events, stats, and logging APIs to help you maintain and trouble-shoot your containers. The Stats API will provide CPU and memory stats for your container. The Logs API will provide all the logs produced by the STDOUT of the PID 1 process inside your container. The Events API will show all Docker Engine events. Issue the following and observe

- CID=$(docker ps -lq)  -> this command will provide the last created container ID which you'll use for the following tasks
- docker stats $CID
- docker logs $CID
- docker events --since 1h


6. Sharing Your Image with Docker Hub

Docker Hub is an online registry to share and get images. It is the default registry that Docker Engine uses to look for images when you try to pull them. If you haven't done so already, please create a free account with www.hub.docker.com. 

Once you  have an account, you can login. Login is required to push images to Hub. Login is not required to pull public images. You can create either public or private images on Hub.

To login into Hub, issue the following command and provide your Docker Hub credentials:

- docker login

If login is sucessful, you should see a note confirming that.

In order to push an image to Docker Hub, it needs to be properly tagged, the format to tag images that are distributed via Docker Hub is :

<Docker Hub Username>/<Image Name>:<Tag or Version>

To tag our image, please issue the following:

-docker tag myimage:v1 <YOUR_DOCKER_HUB_USERNAME>/myimage:v1

You can see a new image entry in your 'docker images'. Tagging an image doesn't duplicate it, it simply adds additional metadata and points at same image. You can confirm that by looking at the Image ID. You will notice that the Image ID is the same for 2 out of the 3 images listed. Now you are ready to push your image to Docker Hub.

- docker push <YOUR_DOCKER_HUB_USERNAME>/myimage:v1

If you go to www.hub.docker.com after a successful push, you should be able to see your image there. You can simple now pull that image from any other Docker Engine using:

- docker pull <YOUR_DOCKER_HUB_USERNAME>/myimage:v1






 

 




















