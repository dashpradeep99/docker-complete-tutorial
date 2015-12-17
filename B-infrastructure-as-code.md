# Lab B: Infrastructure as Code: Image building best practices

> **Difficulty**: Easy
>
> **Time**: 20 minutes
>
> **Tasks**:
>
>* [Prerequisites](#prerequisites)
>* [Task 1: Review Dockerfile](#task-1:-review-the-unoptimized-dockerfile)
* [Task 2: Steps to optimize the image](#task-2:-steps-to-optimize-the-image)

## What is a Dockerfile?
A Dockerfile is a manifest that has instructions on what goes into an docker image. Because all commands are expressed as plain text code, the image can be reconstructed in a predictable and automated manner if necessary.

Every line in the Dockerfile will create a new layer in the image on a build. Keeping the number of layers small without impacting the readability of the Dockerfile is necessary to build optimized images.

## A Good Dockerfile = Optimized Docker image

In this lab, we'll take an existing Dockerfile, review it and then attempt to utilize best practices to ensure that it is optimized in terms of the number of layers, which will translate to higher speed of pulling down the image from a registry, as well as lower size of the overall image. Subsequent docker build invocations should adequately utilize the cache and we'll understand how to avoid some common pitfalls during the building of images. We will also use accepted conventions to mark metadata (in the form of labels) as described [here](https://docs.docker.com/engine/userguide/labels-custom-metadata/).

In other words, we'll fix a defective Dockerfile. Remember, optimization is a process, not a destination. We will therefore, use a process of incremental optimization with a subsequent validation step after every change to improve the Dockerfile. Using a version control system (Git, SVN, Perforce etc) is recommended to track changes and quantitatively measure improvements.

## Prerequisites

* Any node with Docker engine (>=1.9) installed.
* A file editor (vi, emacs, nano).
* A version control respository (Git, SVN Perforce etc) *Not shown here.*

## Task 1: Review the unoptimized Dockerfile
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
3. Create a file `index.html` inside this folder with some basic html content `<h1>Hello World</h1>` will do too.
4. Build an image using `docker build -t nginx-img:0.1 .`
5. View all layers & their sizes that go to make the image. `docker history nginx-img:0.1`
    You should see output similar to below:
   ```
  IMAGE               CREATED              CREATED BY                                      SIZE                COMMENT
  af1e357c08b1        53 seconds ago       /bin/sh -c #(nop) CMD ["nginx" "-g" "daemon o   0 B
  665b0be837a1        53 seconds ago       /bin/sh -c #(nop) LABEL release-date=2016-01-   0 B
  e7116bdcdcef        53 seconds ago       /bin/sh -c #(nop) LABEL is-beta=true            0 B
  f15b10af432f        54 seconds ago       /bin/sh -c #(nop) LABEL version=0.9.0-beta      0 B
  e9eb3d51b24e        54 seconds ago       /bin/sh -c #(nop) LABEL vendor=Acme Example C   0 B
  bb337f0c5e3c        54 seconds ago       /bin/sh -c #(nop) EXPOSE 80/tcp                 0 B
  72957a9cbf5a        54 seconds ago       /bin/sh -c #(nop) COPY file:d0411b7ac48ed3171   21 B
  d28ef2970965        54 seconds ago       /bin/sh -c rm -rf /var/lib/apt/lists/*          0 B
  7d8e6240468d        55 seconds ago       /bin/sh -c apt-get install -y nginx             18.29 MB
  042e7a731dce        About a minute ago   /bin/sh -c apt-get install -y python-all        23.59 MB
  db2a32d8c546        About a minute ago   /bin/sh -c apt-get install -y curl              11.39 MB
  06e9d2d8bd27        About a minute ago   /bin/sh -c apt-get update                       21.43 MB
  e88690a17306        About a minute ago   /bin/sh -c #(nop) MAINTAINER user@acme.com      0 B
  89d5d8e8bafb        9 days ago           /bin/sh -c #(nop) CMD ["/bin/bash"]             0 B
  e24428725dd6        9 days ago           /bin/sh -c sed -i 's/^#\s*\(deb.*universe\)$/   1.895 kB
  1796d1c62d0c        9 days ago           /bin/sh -c echo '#!/bin/sh' > /usr/sbin/polic   194.5 kB
  0bf056161913        9 days ago           /bin/sh -c #(nop) ADD file:9b5ba3935021955492   187.7 MB
   ```
  
6. Test that the image works by running a container with it: `docker run -d -p 80:80 --name www1 nginx-img:0.1`
7. Navigate to `http://<node-ip>` and you should see the rendering of the `index.html`
8. Let's clean up before we attempt to optimize the Dockerfile. `docker rm -f www1`
9. Optionally, consider checking in this working Dockerfile into a source control repository.
10. Modify the Dockerfile to reduce the number of layers & pin the version of the base image. `vi Dockerfile`

   ```
   FROM ubuntu:14.04
   MAINTAINER USER CANE <user@acme.com>
   RUN apt-get update && \
     apt-get install -y curl && \
     apt-get install -y python-all && \
     apt-get install -y nginx && \
     rm -rf /var/lib/apt/lists/*
   COPY index.html /usr/share/nginx/html
   EXPOSE 80
   LABEL vendor=Acme\ Example\ Corp. \
     version="0.9.0-beta" \
     is-beta=true \
     release-date="2016-01-01"
   CMD ["nginx", "-g", "daemon off;"]
   ```   
   
11. Let's build a new image using the updated & hopefully improved Dockerfile. We'll tag it as `0.2` as follows: `docker build -t nginx-img:0.2 .`
12. Running `docker history nginx-img:0.2` should show that the number of layers has indeed been reduced.

   ```
   IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
   cfb976c0c191        7 minutes ago       /bin/sh -c #(nop) CMD ["nginx" "-g" "daemon o   0 B
   95ae6dd917fa        7 minutes ago       /bin/sh -c #(nop) LABEL vendor=Acme Example C   0 B
   df255a6a034c        7 minutes ago       /bin/sh -c #(nop) EXPOSE 80/tcp                 0 B
   955559830d7a        7 minutes ago       /bin/sh -c #(nop) COPY file:d0411b7ac48ed3171   21 B
   5718d61aad87        7 minutes ago       /bin/sh -c apt-get update &&  apt-get install   51.19 MB
   12989f8e0228        10 minutes ago      /bin/sh -c #(nop) MAINTAINER USER CANE <user@   0 B
   89d5d8e8bafb        9 days ago          /bin/sh -c #(nop) CMD ["/bin/bash"]             0 B
   e24428725dd6        9 days ago          /bin/sh -c sed -i 's/^#\s*\(deb.*universe\)$/   1.895 kB
   1796d1c62d0c        9 days ago          /bin/sh -c echo '#!/bin/sh' > /usr/sbin/polic   194.5 kB
   0bf056161913        9 days ago          /bin/sh -c #(nop) ADD file:9b5ba3935021955492   187.7 MB
   ```
13. Ensure that every optimization step is followed by a validation step (steps 8-9) to ensure that the image is functionally adequate and does not break any of its base requirements.
14. Next, consider using a `VOLUME` to make development more flexible.
