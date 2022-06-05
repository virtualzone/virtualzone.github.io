---
title: "How to let Jenkins build Docker images"
date: 2017-06-11T11:30:03+00:00
tags:
    - docker
author: "Heiner"
aliases:
    - /2017/06/how-to-let-jenkins-build-docker-images/
---

If you’re using Jenkins as your Continuous Integration (CI) tool and Docker to build self-contained images of your application, you may ask yourself how to automatically build Docker images during Jenkins’ build job. Here’s how I did it – with Jenkins running in a Docker container itself.

So far, I’ve used the official [Jenkins Docker image](https://hub.docker.com/r/jenkins/jenkins) (the one based on Alpine). I’ve tried some of the Docker plugins for Jenkins available out there. None of them really convinced me as the setup was quite complicated. I’ve been looking for a simpler method.

To achieve this, I’ve created a custom Dockerfile which derives from the official jenkins:alpine image:

```dockerfile
FROM jenkins:alpine
USER root
RUN apk update && \
    apk add docker sudo
RUN echo "jenkins ALL=NOPASSWD: ALL" >> /etc/sudoers
USER jenkins
```

The user-switching is necessary to make sure that the package installation is performed as root (not as jenkins). Next, we update Alpine’s package repository and then install docker and sudo from Alpine’s official repository. sudo is required if your Docker host is configured to restrict Docker usage to specific users. After installing the packages, we allow the jenkins user to run sudo commands without password.

I’m using docker-compose to start my Jenkins container:

```yaml
version: '2'
services:
  jenkins:
    build: /docker/git/docker-jenkins
    volumes:
      - "/docker/storage/jenkins:/var/jenkins_home"
      - "/var/run/docker.sock:/var/run/docker.sock"
```

The build line specifies the folder to your recently created Dockerfile. I mount two volumes here:

* The first one specifies where Jenkins stores its files.
* The second mounts the docker.sock file. This is the key here. It allows the Docker executable in the Jenkins container to communicate with the Docker daemon running on the host.

After starting your Jenkins docker container (using “docker-compose up -d”), browse to your Jenkins URL and configure the job that’s to build a Docker image automatically.

Add “Execute Shell” to your “Build Steps”. Mine looks like:

```bash
sudo docker build -t docker_hub_username/image_name:latest . && \
sudo docker login -u docker_hub_username -p docker_hub_password && \
sudo docker push docker_hub_username/image_name:latest
```

These lines build the Docker image, log in to Docker Hub and push the recently built image.

## Update:

If you want to use docker-compose from your Jenkins Docker container as well, add these lines to your Dockerfile:

```dockerfile
RUN apk add py-pip
RUN pip install docker-compose
```