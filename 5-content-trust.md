# Tutorial 5: Docker Content Trust

**Description**: This lab is an introductory level tutorial exploring the features of Docker Content Trust. After an overview of essential concepts, participants will enable content trust, sign an image, and push the image to Docker Hub.

**Difficulty**: Easy

**Time**: 15 mins

### Tasks:

- Introductory Overview
- Create a test repository on Docker Hub
- Enable Content Trust on our client
- Sign an image and push an image to Docker Hub
- Test the unsigned and signed images

Note: In order to complete this lab you will need:

- An existing Docker Hub account
- SSH onto the provided `<username>-node-0` instance in AWS 
- Logged into Docker Hub with the Docker client

### Overview:

Content trust allows operations with a remote Docker registry to enforce client-side signing and verification of image tags. Content trust provides the ability to use digital signatures for data sent to and received from remote Docker registries. These signatures allow client-side verification of the integrity and publisher of specific image tags.

Image publishers can sign their images. Image consumers can ensure that the images they use are signed. publishers and consumers can be individuals alone or in organizations. Docker’s content trust supports users and automated processes pipelines.

Content trust is currently only available for users of the public Docker Hub. It is currently not available for the Docker Trusted Registry or for private registries.

### Create a new Docker Hub repository

Before we enable Docker Content Trust, let's create a new temporary repository on Docker Hub to use for the following examples.

On your AWS instance `<username>-node-0`, ensure you're logged into Docker hub:

```
ubuntu@node-0:~$ docker login
Username: <username>
Password: <password>
Email: <email>
WARNING: login credentials saved in /home/ubuntu/.docker/config.json
Login Succeeded
```
Pull the **busybox** image:

`docker pull busybox`

Create a new tag for the busybox image using your namespace:

`docker tag busybox <username>/dctrust:latest`

Push this unsigned image to Docker Hub:

`docker push <username>/dctrust:latest`

At this point you should have a new unsigned image repository **dctrust** (with the 'latest' tag) under your Docker Hub account at: https://hub.docker.com/r/(username)/dctrust/

### Working with Content Trust

Currently, content trust is disabled by default. To enable it, the only thing you need to do is to set the `DOCKER_CONTENT_TRUST` environment variable.

Enable content trust by running the command:

```
export DOCKER_CONTENT_TRUST=1
```

From this moment on, every Docker operation is secure. Any future Docker commands excecuted will be based on signatures.

Now that we've enabled content trust, lets try to pull the image we just pushed. You'll notice that the Docker client will not execute the command because there isn't any trust data available.

```
docker pull <username>/dctrust:latest
no trust data available
```

In order to create the trust data for an image all we need to do is push an image with `DOCKER_CONTENT_TRUST` enabled.

Create a new tag for our image:

`docker tag busybox <username>/dctrust:signed`

Note: On the next command, during the first push of our new image, Docker will prompt for two new passphrases for a **root key** and a **repository key**. More information is provided about these keys in the reference section below. For now, just enter two new passphrases when prompted.

Push our image:

`docker push <username>/dctrust:signed`

Example output:

```
ubuntu@node-0:~$ docker push kizbitz/dctrust:signed
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
```

At this point in our Docker hub repository we have two different images/tags:

- dctrust:latest - This image is still unsigned and cannot be pulled as long as content trust is enabled.
- dctrust:signed - This image has been signed and can be pulled if content trust is enabled or disabled.

Example Pulls (With content trust enabled):

```
ubuntu@node-0:~$ docker pull kizbitz/dctrust:latest
No trust data for latest
ubuntu@node-0:~$ docker pull kizbitz/dctrust:signed
Pull (1 of 1): kizbitz/dctrust:signed@sha256:b33669e38aef420ff10d4787c07afe11a679564f65c31bde39572c9196f3e143
sha256:b33669e38aef420ff10d4787c07afe11a679564f65c31bde39572c9196f3e143: Pulling from kizbitz/dctrust
Digest: sha256:b33669e38aef420ff10d4787c07afe11a679564f65c31bde39572c9196f3e143
Status: Downloaded newer image for kizbitz/dctrust@sha256:b33669e38aef420ff10d4787c07afe11a679564f65c31bde39572c9196f3e143
Tagging kizbitz/dctrust@sha256:b33669e38aef420ff10d4787c07afe11a679564f65c31bde39572c9196f3e143 as kizbitz/dctrust:signed
```

To temporarily disable content trust an a single command you can use `--disable-content-trust`:

```
ubuntu@node-0:~$ docker pull kizbitz/dctrust:latest
No trust data for latest
ubuntu@node-0:~$ docker pull --disable-content-trust kizbitz/dctrust:latest
latest: Pulling from kizbitz/dctrust
Digest: sha256:f3fefef65b040546adab73e2e6a624376373b0f12e83a5e5a921d9cd9059953c
Status: Image is up to date for kizbitz/dctrust:latest
```

### Summary and Cleanup

At this point you have successfully enabled content trust on your Docker client, signed an image, and pushed the image to Docker Hub.

To clean-up your environment, remember to:

- Delete the test repository created on your Hub account: https://hub.docker.com/r/(username)/dctrust/
- Unset the `DOCKER_CONTENT_TRUST` environment variable: `unset DOCKER_CONTENT_TRUST`

### Further Reading

For more in-depth examples and advanced concepts refer to the official documentation:

- [Content Trust Concepts](https://docs.docker.com/security/trust/content_trust/)
- [Automation Systems](https://docs.docker.com/security/trust/trust_automation/)
- [Understanding and Managing Keys](https://docs.docker.com/security/trust/trust_key_mng/)
- [Personal Sandbox](https://docs.docker.com/security/trust/trust_sandbox/)