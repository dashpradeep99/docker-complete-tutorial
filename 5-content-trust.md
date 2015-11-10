# Lab 5 : Docker Content Trust

> **Difficulty**: Easy

> **Time**: 15 mins
>
> **Prequisites**:
>
* An existing Docker Hub account
* SSH onto the provided `<username>-node-0` instance in AWS
* Logged into Docker Hub with the Docker client

> **Tasks**:
>
* [Prerequisites](#prerequisites)
* [Task 1: Create a new Docker Hub repository](#task-1-create-a-new-docker-hub-repository)
* [Task 2: Enable Content Trust](#task-2-enable-content-trust)
* [Task 3: Push a signed image](#task-3-push-a-signed-image)
* [Task 4: Pulling images](#task-4-pulling-images)
* [Task 5: Cleanup](#task-5-cleanup)



## What is Docker Content Trust?

Content trust allows image operations with a remote Docker registry to enforce
client-side signing and verification of image tags. Content trust enables
digital signatures for data sent to and received from remote Docker registries.
These signatures allow Docker client-side interfaces to verify the integrity and
publisher of specific image tags.

Image publishers can sign their images. Image consumers can ensure that the
images they use are signed. Both publishers and consumers can be either
individuals or organizations. Docker’s content trust supports users and
automated processes pipelines.

Content trust is currently only available for users of the public Docker Hub. It
is currently not available for the Docker Trusted Registry or for private
registries.

## Prerequisites

* You will be using **node-0**
* An existing Docker Hub account
* SSH onto the provided `<username>-node-0` instance in AWS
* Logged into Docker Hub with the Docker client

## Task 1: Create a new Docker Hub repository

Before you enable Docker Content Trust, you must create a new temporary repository on Docker Hub. You'll use this temporary repository for the lab.

1. Log into your `node-0` instance.

2. Use the Docker command-line client to log into Docker Hub.

        $ docker login
        Username: <username>
        Password: <password>
        Email: <email>
        WARNING: login credentials saved in /home/ubuntu/.docker/config.json
        Login Succeeded

3. Pull the `busybox` image:

        $ docker pull busybox

4. Create a new tag for the busybox image using your namespace.

  Your Docker Hub account is your namespace.

        $ docker tag busybox <username>/dctrust:latest

5. Push this unsigned image to Docker Hub:

        $ docker push <username>/dctrust:latest

6. Open <a href="https://hub.docker.com target="_blank">Docker Hub</a> in your browser and login.

  At this point you should have a new unsigned image repository **dctrust** (with the 'latest' tag) under your Docker Hub account.

  ![On Docker Hub](images/dctrust-pushed.png)

## Task 2: Enable Content Trust

Currently, content trust is disabled by default. To enable it, set the `DOCKER_CONTENT_TRUST` environment variable in your environment as follows:

```
$ export DOCKER_CONTENT_TRUST=1
```

Once you set this environment variable, every Docker operation is secure. Any future Docker commands executed based on signatures.

Now that you've enabled content trust, try to pull the image you just pushed.

```
$ docker pull <username>/dctrust:latest
no trust data available
```
You'll notice that the Docker client did not pull the image.  This is because the image was not signed.

## Task 3: Push a signed image

To create the trust data for an image, you need push an image with
`DOCKER_CONTENT_TRUST` enabled. The first push of your new image, Docker prompts
for two new passphrases for a **root key** and a **repository key**.
1. Create a new tag for the `busybox` image:

        $ docker tag busybox <username>/dctrust:signed

2. Push the image making sure to enter two new passphrases when prompted.

        $ docker push <username>/dctrust:signed

  Example output:

        $ docker push kizbitz/dctrust:signed
        The push refers to a repository [docker.io/kizbitz/dctrust] (len: 1)
        2c5ac3f849df: Image already exists
        ab2b8a86ca6c: Image already exists
        signed: digest: sha256:b33669e38aef420ff10d4787c07afe11a679564f65c31bde39572c9196f3e143 size: 3214
        Signing and pushing trust metadata
        You are about to create a new root signing key passphrase. This passphrase
        will be used to protect the most sensitive key in your signing system. Please
        choose a long, complex passphrase and be careful to keep the password and the
        key file itself secure and backed up. It is highly recommended that you use a
        password manager to generate the passphrase and keep it safe. There will be no
        way to recover this key. You can find the key in your config directory.
        Enter passphrase for new root key with id 020aa4f:
        Repeat passphrase for new root key with id 020aa4f:
        Enter passphrase for new repository key with id docker.io/kizbitz/dctrust (ab6b057):
        Repeat passphrase for new repository key with id docker.io/kizbitz/dctrust (ab6b057):
        Finished initializing "docker.io/kizbitz/dctrust"

## Task 4: Pulling images

At this point in your Docker hub repository you have two different images/tags:

| Image            | Description                                                                      |
|------------------|----------------------------------------------------------------------------------|
| `dctrust:latest` | This image is still unsigned and cannot be pulled when content trust is enabled. |
| `dctrust:signed` | This image is signed and can be pulled if content trust is enabled or disabled.  |

At this point, you have trust enabled.

1. Pull the latest image and then the signed image.

        $ docker pull kizbitz/dctrust:latest
        No trust data for latest
        ubuntu@node-0:~$ docker pull kizbitz/dctrust:signed
        Pull (1 of 1): kizbitz/dctrust:signed@sha256:b33669e38aef420ff10d4787c07afe11a679564f65c31bde39572c9196f3e143
        sha256:b33669e38aef420ff10d4787c07afe11a679564f65c31bde39572c9196f3e143: Pulling from kizbitz/dctrust
        Digest: sha256:b33669e38aef420ff10d4787c07afe11a679564f65c31bde39572c9196f3e143
        Status: Downloaded newer image for kizbitz/dctrust@sha256:b33669e38aef420ff10d4787c07afe11a679564f65c31bde39572c9196f3e143
        Tagging kizbitz/dctrust@sha256:b33669e38aef420ff10d4787c07afe11a679564f65c31bde39572c9196f3e143 as kizbitz/dctrust:signed

2. Now pull images but temporarily disable content trust with the  `--disable-content-trust` flag.

        $ docker pull --disable-content-trust kizbitz/dctrust:latest
        latest: Pulling from kizbitz/latest
        Digest: sha256:f3fefef65b040546adab73e2e6a624376373b0f12e83a5e5a921d9cd9059953c
        Status: Image is up to date for kizbitz/dctrust:latest
        $ docker pull --disable-content-trust kizbitz/dctrust:signed
        signed: Pulling from kizbitz/dctrust
        Digest: sha256:357cb702777f1bdf9a6241e8cf9d17b05d30fc203e7e4e51464a067e826c7906
        Status: Downloaded newer image for kizbitz/dctrust:signed

## Conclusion

At this point you have successfully enabled content trust on your Docker client, signed an image, and pushed the image to Docker Hub.

### Share on Twitter!

<p>
<a href="http://ctt.ec/R3e56" target=“_blank”>
<img src="http://www.wyntercon.com/wp-content/uploads/2015/04/twitter-bird-blue-on-white-small.png" width="100" height="100">
</p>

## Clean up

If you plan to do another lab, you need to cleanup your EC2 instances. Cleanup removes any environment variables, configuration changes, Docker images, and running containers. To do a clean up, log into each EC2 instance and run the following:

```bash
$ source /home/ubuntu/cleanup.sh
```

## Related information

For more in-depth examples and advanced concepts refer to the Docker documentation:

- [Content Trust Concepts](https://docs.docker.com/security/trust/content_trust/)
- [Automation Systems](https://docs.docker.com/security/trust/trust_automation/)
- [Understanding and Managing Keys](https://docs.docker.com/security/trust/trust_key_mng/)
- [Personal Sandbox](https://docs.docker.com/security/trust/trust_sandbox/)
