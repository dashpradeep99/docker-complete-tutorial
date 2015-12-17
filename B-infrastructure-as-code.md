# Lab B: Infrastructure as Code: Image building best practices

> **Difficulty**: Easy
>
> **Time**: 20 minutes
>
> **Tasks**:
>
>* [Prerequisites](#prerequisites)
>* [Task 1: Review Dockerfile](#review-dockerfile)
* [Task 2: Steps to optimize the image](#steps-to-optimize--the-image)

## What is a Dockerfile?
A Dockerfile is a manifest that has instructions on what goes into an docker image. Because all commands are expressed as plain text code, the image can be reconstructed in a predictable and automated manner if necessary.

Every line in the Dockerfile will create a new layer in the image on a build. Keeping the number of layers small without impacting the readability of the Dockerfile is necessary to build optimized images.

## Prerequisites

Docker engine installed and a file editor (vi, emacs, nano).

## Task 1: Review unoptimized Dockerfile
```
FROM ubuntu
MAINTAINER user@acme.com
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y python-all
RUN apt-get install -y nginx
RUN rm -rf /var/lib/apt/lists/*
COPY index.html /usr/share/nginx/html
EXPOSE 80
LABEL vendor=Acme\ Example\ Corp.
LABEL version="0.9.0-beta"
LABEL is-beta=true
LABEL release-date="2016-01-01"
CMD ["nginx", "-g", "daemon off;"]
```

## Task 2: Steps to optimize the image

1. Create a folder `nginx` under `/home/ubuntu` and navigate into it: `mkdir ~/nginx && cd ~/nginx`.
2. Create a file `Dockerfile` inside this folder using the contents from Task 1.
3. Create a file `inde,html` inside this folder with some basic html content `<h1>Hello World</h1>` will do too.
4. Build an image using `docker build -t nginx-img:0.1 .`
5. View all layers & their sizes that go to make the image. `docker history nginz-img:0.1`
6. Test that the image works by running a container with it: `docker run -d -p 80:80 nginx-img:0.1`
7. Navigate to `http://<node-ip>` and you should see the rendering of the `index.html`
8. Optimize the Dockerfile making sure to go through steps 4-7.
