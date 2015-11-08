# Lab 8: Automated Builds with Docker Hub and GitHub

> **Difficulty**: Beginner
>
> **Time**: 30 minutes
>
> **Prerequisites**
> You will need a GitHub account and a Docker Hub account to complete this lab.

> If you do not have the required accounts, please visit the websites below to create new accounts (both services allow you to create free acounts).

> http://hub.docker.com
>
> http://www.github.com

> Note: Make sure you confirm your email address for each account you create

> **Tasks**:

> * Link your Docker Hub account to GitHub
* Fork the tutorial repo
* Clone the tutorial repo
* Create a new automated build
* Manually trigger a build
* Trigger an automated build

#What is an Automated Build?
An automated build is a Docker build that is kicked off by a change to a linked GitHub respository. By linking your GitHub repo to Docker Hub a new Docker image can be automatically built any time a changed is pushed to that repository.

#Task 1: Link your Docker Hub account to GitHub
In order for Docker Hub to kick off automated builds, it needs to be linked to your GitHub account.

1. In your web broweser, navigate to http://hub.docker.com and log into Docker Hub with your personal credentials

2. Click the down arrow next to your account name in the top right corner and select “Settings”

3. From the menu bar at the top click on “Linked Accounts & Services”

4. Click “Link GitHub”

5. Click “Select” under Public & Private (Recommended)"

6. Enter your personal GitHub credentials when prompted

	**Note***: If you are already logged into GitHub, you will not need to enter credentials*

#Task 2: Fork the tutorial repo
The first thing we're going to do is fork a copy of our demo repo into your GitHub account. This allows you make changes to the repo without affecting other users.

This repo builds an Nginx web server with a very simple index.html file

1. In a new browser tab navigate to http://www.github.com/mikegcoleman/dceu_tutorial8

2. Click the fork button in the upper right hand corner

	You should be redirected into your own version of the tutorial repository. The URL 	should be `https://github.com/<your githhub user name>/dceu_tutorial8`

#Task 3: Clone the tutorial repo
Now we're going to clone the repo from GitHub onto your local machine so you can easily modify the files later in the lab.

1. Open a terminal window and SSH into your Node-3 host with the supplied credentials (the 	username is ubuntu, you should have been provided an SSH key and docker host address)

		$ ssh -i <identity file> <user>@<Node-3 IP Addresst>

	For example:

		$ ssh -i user23.pem ubuntu@10.0.0.100


2. Inside the terminal window, clone your GitHub repo

		$ git clone https://github.com/<github username>/dceu_tutorial8.git

		Cloning into 'dceu_tutorial8'...
		remote: Counting objects: 7, done.
		remote: Compressing objects: 100% (5/5), done.
		remote: Total 7 (delta 0), reused 7 (delta 0), pack-reused 0
		Unpacking objects: 100% (7/7), done.
		Checking connectivity... done.

	**Note***: If you want to copy and paste the URL, click the clipboard under “HTTPS clone 	url” on your GitHub repo web page*

#Task 4: Create a new automated build
Now that we  have our GitHub repos setup, let's create an automated build that fires off any time a change is pushed to our GitHub repository.

1. In your web browser, navigate back to http://hub.docker.com and log in if necessary

2. Click “Create” in the upper right hand corner of the window (next to your account name) 	and select “Create Automated Build”

3. Make sure your user account is selected under “Users/Organizations”. If it isn’t, just 	click on your user account

4. From the list of repositories on the right, click on “dceu_tutorial8”

	**Note***: If you have a long list of repos, you can use the box above the repo list to 	filter the results*

5.	Type in “DockerCon EU tutorial” for the “Short Description”

6. Make sure "When Active, new pushes trigger automatic builds" is checked

	**Note***: We’ll leave the rest of the values at their default*

7. Click “Create”

	**Note***: It can take a few minutes for your automated build job to be created*

#Task 5: Manually Trigger a Build
Before we try an automated build, we're going to trigger a manual build to make sure everything is working correctly.

1. After the autmoated build is created it should automatically load the page for your new 	repository. If it didn't navigate to the Docker Hub home page, and select yoru automated 	build from your list of repositories.

2. Click "Build Settings"

3. Click "Trigger a Build"

4. Click "Build Details"

5. After a bit the "Build Status" should be done

6. Copy the "Docker Pull Command" from the right hand side of the screen

7. Switch back to your terminal window and pull down your newly built image

		$ docker pull <docker hub username>/dceu_tutorial8

		Using default tag: latest
		latest: Pulling from <docker hub username>/dceu_tutorial8

		90d7a4de1ef5: Pull complete
		60b05e04b5b6: Pull complete
		95d79d93cde4: Pull complete
		Digest: sha256:eab8584e5b08227637ff7ff77b933e062f3abb031b6e423bbf465d408632eae9
		Status: Downloaded newer image for <docker hub username>/dceu_tutorial8:latest

8. Run the webserver to make sure it's executing correctly

		$ docker run -d -p 80:80 --name mywebserver <docker hub username>/dceu_tutorial8
		b9d2496ba15ebd61d288028ae45f2479267ed4c183920b3080ebe2735eca133e

9. In a new tab in your web browser, navigate to your node-3 IP address. You should see a 	web page with the text "Hola, Barcelona"

10. Switch back to your terminal window, stop and remove your running container

		$ docker stop mywebserver
		mywebserver

		$ docker rm mywebserver
		mywebserver

11. Remove the web server image

		$ docker rmi mywebserver
		mywebserver

#Task 6: Trigger an Automated build
Now that we know our build is good, we'll push a change to the GitHub repo, which will trigger an automated build of our Docker image.

1. In your terminal window, make sure you're in the directory for the tutorial repo

		$ cd ~/dceu_tutorial8

2. Change the index.html file for our webserver

		$ echo 'Docker es el mejor' > index.html

	**Note***: Make sure to use single quotes (')

3. List the content of index.html to make sure it was updated

		$ cat index.html
	Docker es el mejor

4. Add the updated file to our GitHub repo

		$ git add .

5. Commit the change

		$ git commit -m "updated index.html"
		[master 72e748d] updated index.html
	 	1 file changed, 1 insertion(+), 1 deletion(-)

6. Push our changes to the GitHub repo. This will trigger the automated build.

		$ git push origin master
		Counting objects: 3, done.
		Delta compression using up to 8 threads.
		Compressing objects: 100% (2/2), done.
		Writing objects: 100% (3/3), 277 bytes | 0 bytes/s, done.
		Total 3 (delta 1), reused 0 (delta 0)
		To https://github.com/<github username>/dceu_tutorial8.git

7. In your web browser navigate back to your demo repo on Docker Hub

8. Click "Build details" and confirm that there is a new build

9. Copy the "Docker Pull Command" from the right hand side of the screen

10. Switch back to your terminal window and pull down your newly built image

		$ docker pull <docker hub username>/dceu_tutorial8

		Using default tag: latest
		latest: Pulling from <docker hub username>/dceu_tutorial8

		90d7a4de1ef5: Pull complete
		60b05e04b5b6: Pull complete
		95d79d93cde4: Pull complete
		Digest: sha256:eab8584e5b08227637ff7ff77b933e062f3abb031b6e423bbf465d408632eae9
		Status: Downloaded newer image for <docker hub username>/dceu_tutorial8:latest

11. Run the webserver to make sure it's executing correctly

		$ docker run -d -p 80:80 --name mywebserver <docker hub username>/dceu_tutorial8
		b9d2496ba15ebd61d288028ae45f2479267ed4c183920b3080ebe2735eca133e

12. In a new tab in your web browser, navigate to your node-3 IP address. You should see a 	 web page with the text "Docker es el mejor"

#Cleaning Up

### Remove the webserver container
1. Switch back to your terminal window, stop and remove your running container

		$ docker stop mywebserver
		mywebserver

		$ docker rm mywebserver
		mywebserver

### Remove the webserver images
2. In your terminal window, remove the web server image

		$ docker rmi mywebserver
		mywebserver

### Remove the automated build (Optional)
1. In your webserver navigate to your tutorial repo on Docker Hub

2. Click "Settings" (NOT "Build Settings")

3. Click "Delete"

4. Enter the repository name `dceu_tutorial8`

5. Click "Delete"

### Remove the repo from GitHub (Optional)
1. In your webserver navigate to the demo GitHub repo

		https://github.com/<github username>/dceu_tutorial8

2. On the right hand side of the web page click "Settings"

3. Scroll down to the "Danger Zone" and click "Delete this Repository"

4. Enter the name of the respository

		<githug username>/dceu_tutorial8
