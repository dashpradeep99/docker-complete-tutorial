#Tutorial 7 : Understanding Docker Volumes
> **Difficulty**: Intermediate

> **Time**: 30 mins

> **Prerequisites**: A machine with Docker Engine installed

> **Tasks**

> * Implement a Docker volume via the Docker client
* Understand how Docker represents volumes in the file system
* Delete a Docker volume
* Map a host directory to a Docker volume
* Use a volume to share data between containers

# What are Docker data volumes?

A data volume is a specially-designated directory within one or more containers that bypasses Docker’s storage driver and interacts directly with the host file system. Data volumes provide several useful features for persistent or shared data:

* Volumes are initialized when a container is created. If the container’s base image contains data at the specified mount point, that existing data is copied into the new volume upon volume initialization.
* Data volumes can be shared and reused among containers.
* Changes to a data volume are made directly to the host filesystem (vs. going through the storage driver)
* Changes to a data volume will not be included when you update an image.
* Data volumes persist even if the container itself is deleted.

Data volumes are designed to persist data, independent of the container’s lifecycle. Docker therefore never automatically deletes volumes when you remove a container, nor will it "garbage collect" volumes that are no longer referenced by a container.

# Task 1: Implementing a volume via the Docker client
In this task we're going to create a new container, and then add a file to it.

1. SSH into your Docker host with the supplied credentials, for example:	`ssh -i user23.pem ubuntu@ec2-52-24-109.us-west-2.compute.amazonaws.com`

2. Pull down the official Nginx image

	`$ docker pull nginx`

3. Create a new  volume named /barcelona	`$ docker volume create --name barcelona`

	**Note***: Your new volume will be created on the host in the* `/var/lib/docker/volumes`*directory*

4. Ensure your volume was created by using docker volume ls to list the volumes on your Docker host

 		$ docker volume ls
		DRIVER              VOLUME NAME
		local               barcelona	**Note***: You may see other volumes listed in addition to the one you just created*

5. Instantiate a Docker container with our barcelona volume mapped to /barcelona in 	the container, and log into the shell

	 	$ docker run -it -v barcelona:/barcelona --name volumeslab nginx /bin/bash	This command will instantiate an Nginx container and mount our volume `barcelona` 	at the at the root of the file system `/barcelona`. It will also execute `/bin/bash` 	giving you an interactive shell inside the container.

	 **_Note_***: You are now working within the shell of your running container*6. Change directory into /barcelona

		$ cd /barcelona

7. Create a file in that directory		$ touch file.txt

8. List the directory contents to make sure the file exists		$ ls
		file.txt

9. Exit the container but leave it running by hitting `Ctrl+P` (at the same time) and then hitting `Ctrl+Q` (at the same time)	**_Note_***: You are now working within the shell of your Docker host*10. Ensure your container is still running:		$ docker ps	The output should be similar to:
	
		CONTAINER ID   IMAGE   COMMAND         CREATED             STATUS                           		0124480582a2   nginx   "/bin/bash"     6 minutes ago       Up 6 minutes                           The important detail is that `STATUS` lists `Up` instead of `Exited`

# Task 2: Understand how Docker represents volume data in the file system

As mentioned above, Docker manages volumes outside of the storage driver that it uses to manage the layers of a given container. This allows for data persistence(the volume is not destroyed when the container is destroyed).In this task we’re going to take a quick look at where Docker stores volume data, and how a change to the host filesystem is immediately reflected back in the container.

1. You can use  docker volume inspect to learn details about about a given volume. In this case our barcelona volume		$ docker volume inspect barcelona	Your output should be similar to:		
		[   			{        		"Name": "barcelona",        		"Driver": "local",        		"Mountpoint": "/var/lib/docker/volumes/barcelona/_data"    		}		]	**Note***: The volume mount point above is*: `/var/lib/docker/volumes/barcelona/_data`

2. In order to examine the contents of the directory, you’ll need to elevate your user privileges:		$ sudo su

3. Change to the volumes data directory in the host filesystem		$ cd /var/lib/docker/volumes/barcelona/_data

4. List the contents of the directory		$ ls
		file.txt	Notice the file you previously created `file.txt` is listed in the host directory

5. Create a new file		$ touch file2.txt

6. Ensure both file.txt and file2.txt exist in the directory		$ ls		file2.txt file.txt

7. Log back into the shell of the running container using `docker exec`		$ docker exec -it volumeslab /bin/bash

	**Note***: You are now working within the shell of your running container*8. List the contents of the `/barcelona` directory and note that the file you created from the Docker host shell 	`file2.txt` now exists inside your running container.		$ ls /barcelona		file.txt file2.txt

9. Exit the container, and return to your docker host		$ exit

#Task 3: Deleting a volume

By default, when you destroy a container Docker does not remove any volumes associated with the container. You can, however, delete a given volume with `docker volume rm` 

1. Stop the container we have been working with previously		$ docker stop volumeslab

2. Remove the container.		$ docker rm volumeslab
		**Note***: This will not remove the volume the container was using*

3. Ensure the volume we were working with still exists		$ docker volume ls		DRIVER              VOLUME NAME		local               barcelona	**Note***: You may see other volumes listed in addition to*`barcelona`

4. Elevate your privileges		$ sudo su

5. List the contents of the volume directory to see that the two files are still there		$ ls /var/lib/docker/volumes/barcelona/_data		file.txt file2.txt

6. Exit superuser		$ exit

7. Remove the volume		$ docker volume rm barcelona		barcelona

8. Ensure the volume was removed		$ docker volume ls		DRIVER              VOLUME NAME	**Note**:*The* `barcelona' *volume is no longer listed, although other volumes may still appear*

#Task 4: Map a host directory to a Docker volume

In the previous exercises we were able to manipulate data files in a Docker volume via the host file system,. However, it was a bit cumbersome: we had to learn the mount point via docker volume inspect, and elevate our privileges to create a new file. In practice, this is impractical.Thankfully there is an easier way to  make local files available inside a running container. Docker allows existing local files and directories to be mounted as volumes into a running container. This allows developers, for instance, to hot-mount code into their containers, and have changes made locally be reflected instantaneously in their container.In this example we’ll be modifying some HTML, and then showing how the changes are reflected immediately on the container-based website. Additionally, we’ll be creating our volume as part of the docker run command using the `-v` option.

1. Create a www subdirectory on your Docker host		$ mkdir ~/www

2. Change into your www directory		$ cd ~/www

3. Use the following command to create an index.html file with some content.		$ echo '<h1>Hola Barcelona!</h1>' > index.html

	**Note***: Be sure to use single quotes in the command above.*

4. Instantiate an Nginx container that maps the `~/www` directory on the host to `/usr/share/nginx/html` in the running container	**Note***:* `usr/share/nginx/html` *is the default directory for serving web content on the Nginx server*		$ docker run -d -v ~/www:/usr/share/nginx/html --name mywebserver -p 8001:80 nginx

5. Use the browser on your laptop to navigate to navigate to the newly created website at `http://<docker host>:8001/`
	**Note***: Your docker host name was provided to you when you signed up for this lab*	**Note***: You must add the port number (***8001***) to the URL as we’ve mapped port 80 inside the container to port 8001 on the docker host.*	Your website should display your "Hello, Barcelona" message

6. Return to your Docker host’s terminal window, and update the HTML of your website with the following command:		$ echo '<h1>Adios Barcelona!</h1>' > index.html	**Note***: Be sure to use single quotes in the command above.*

7. Reload the website in your browser, and notice how the change to the local filesystem is immediately reflected by the running Nginx container.

#Task 5: Use a volume to share data between containers

You can also use volumes to share data between containers. For instance, one container could be writing data to a log file, and a second container could be reading that data and providing a graphical interpretation.

In our example we’ll simply create a new file from one container in a directory that is a mount point for a named container, and then list the directory from another container.**Note**: *This example is purely for academic purposes. In a real-world deployment, it’s extremely dangerous to have multiple containers writing to the same volume without adequate application awareness and protection against conflicts. *

1. Create a new named volume `shared-data`		$ docker volume create --name shared-data		shared-data

2. Create a new container `foo`, mount the previously created named volume `shared-data` to the `/foo` directory, and log into the shell		$ docker run -it -v shared-data:/foo --name foo nginx /bin/bash	**Note***: You are now working in the shell of your docker container*

3. Create a new file in the /foo directory (which is mapped to your shared-data volume)		$ echo 'Tu dices "hola"' > /foo/file.txt	**_Note_***: Be careful to use the right single quote (‘) and double quotes (")*

4. Verify the content of your newly created file		$ cat /foo/file.txt		Tu dices "hola"

5. Exit the container, but leave it running by typing `Ctrl-P` and then `Ctrl-Q`

6. Create a second container `bar`, mount the previously created named volume `shared-data` to the /bar directory, and log into the shell		$ docker run -it -v shared-data:/bar --name bar nginx /bin/bash	**Note***: You are now working in the shell of your docker container*

7. Create a new file in the /foo directory (which is mapped to your shared-data volume)		$ echo 'Y yo digo "adios"' >> /bar/file.txt	**_Note_***: Be careful to use the right single quote (‘) and double quotes (")*	**Note***: Be sure to use >> otherwise you will overwrite your orignal file*

8. Verify the content of your newly created file		$ cat /bar/file.txt		Tu dices "hola"		Y yo digo "adios"	**Note**: *The text you created in the first container (Tu dices "hola") is still accessible in this container 	as well as the text you just added  (Y yo digo “adios”)*

9. Exit this container		$ exit
		
10. Log back into the shell of the original container using the docker exec command		$ docker exec -it foo /bin/bash	**Note***: You are now working in the shell of your docker container*

11. Check to see if the changes you made in the bar container are reflected in this container		$ cat /foo/file.txt		Tu dices "hola"		Y yo digo "adios"

12. Exit this container		$ exit

# Conclusion

In this lab we learned the basics of Docker volumes. We created a new Docker volume using the Docker client, and then explored the host file system to understand the relationship between the local file system and the mounted volume in the container. We then deleted our volume, and mapped a new volume to a specific directory on the host file system. Finally we looked at how you can share a volume between multiple containers.

Feel free to continue exploring Docker volumes.
