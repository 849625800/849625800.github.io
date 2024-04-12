---
layout: post
title: Docker - In a Nutshell
date: 2024-03-30 17:40 +0800
last_modified_at: 2024-04-03 18:30 +0800
math: true
tags: [docker]
toc:  true
---

# Introduction

This note will focus on several topics. 
- What Docker is? Why we use Docker? 
- What is the different between Docker and Virtual Machines? 
- Then we will have some hands-on experiments, which will aim to strangthen our understanding of the Docker Images and Containers, public and private registries. 
- Later on, We will learn how to create Docker Image (Dockerfile), understand some main Docker commands and versioning.
- Finally, we will learn the Docker workflow in a big piture.

[Github repo](https://github.com/yu-jinh/Docker-Practice) 
# What is Docker? Why is this important?

**what?**

In a word, Docker is a virtualization software that makes developing and deploying applications much easier.

- Docker can packages application with all the necessary dependencies configuration, system tools and runtime into a Docker container. It is a standardized unit, that has everything the application needs to run.

- The docker conatiner is a portable artifact which is easily shared and distributed.

**Why?** 

**Without Docker Container**, developer needs to install and configure all dependencies and setup the environment, which is a tedious process.

 - The installation is different for each OS, which even makes the configuring much more inefficient.

 - The installation has many steps, where something can easily go wrong.

 - The process highly depends on how complax the application is.

**With Docker Container**, we can package our own isolated environment. For example, we can have the PostgresSQL with all dependencies and configs. As a developer, we can get rid of the binary files to install all the environments. We can simply use a Docker command to fatch environment from the internet and run it on our computers.

If we have 10 services, we just need to run 10 docker commands for each container for each service no matter what operation system you are using.

Therefore, Docker standardize process of running any service on local development environment. Different versions of sam app can be run in a local environment without any conflicts, which is so hard to do without the support of Docker.

**Deployment without Docker:**
- **Development team** will provide the Artifacts (such as jar file and datasets) with the corresponding configuration file and handed over to Operation team.

- **Operation team** handles installing and configuring apps and its dependencies.

The problem of this approach is that we need to install and configure everything on the operation system, which can raise varies of problems during the setup process. We may also encounter the version conflict if different service depends on different version of a same library.

The other problem may cause by the miscommunication between the development team and operation team since everything is based on the configuration file, which may have human errors. If it happenes, the efficiency of the process is not guaranteed.


**Deployment with Docker:**
- **Development team** will provide Docker artifact includes everything the app needs instead of the textual configuration files. 

- **Operation team** will get rid of the tedious and error-prone configuration. The works for the operation team are inatalling the Docker runtime on the server and run Docker command to fetch and run the Docker artifacts.

# Virtual Machine Vs. Docker

Because Docker is a virtualization tool, Virtural Machines have the same functionality. So why docker is widely accepted by developers? What advantages do it have over VMs? What is the difference?

## Different of Virtualization part of Operation system 

A operation system such as linux, windows, macOS, is consisted with 2 layers:
- **OS kernel:** Communicate with hardwares. It helps allocate hardware resources like memory, cpu, etc to the applications.

- **OS Application Layer:** This layer is where the application running on.

**Docker** virtualizes the OS Application Layer. It contains the OS application layer and the services and apps installed on top of that layer such as java runtime and python etc. It use the same OS kernel of the host.

**VMs:** The virtual Machine virtualizes both the OS Kernel and OS application layer.

## Comparation

**Pros:**

- **Size:** Docker images are much smaller (MBs) than VM images (GBs).
- **Speed:** Docker comtainer run much faster than VM images because it reuse the kurnel of the host, while the VM should initialize the kernel each time.

**Cons:**

- **compability:** Since docker is reusing the kernel of the host, it it not compatable to a different operation system. For example, A docker on windows OS cannot run on a Linux machine. But VM can since it also have it's own kernel.

**Note:** Docker made an up data for Docker Desktop, which may enable the Linux based application run on other Operation systems. It uses a Hypervisor layer with a lightweight Linux distro. 

# Docker Desktop:
We can use Docker Desktop to help us run any docker container on our local machine. We can get the softwear by visiting the official Docker website, then follow the installation steps. It is always the best to refer to latest installation guide in the official documentation.

There are several parts in the Docker Desktop:
- **Docker Engine:** Main part of Docker. A server with a long-running daemon process "dockerd". It manages images and containers.

- **Docker CLI-Client** It is a command line interface ("docker") to interact with Docker Server. We can execute Docker commands to start/stop and do any other actions for containers.

- **GUI Client** The Graphical user interface to manage your containers and images.

Please choose the installation guide that match the local machine.

# Docker Images vs. Docker Containers

- **Docker Images** 

    Docker Images are executable application artifacts. Similar to zip, tar or jar file, they can be easily shared and moved. It not only includes app source code, but also has complete environment configuration including the Application (like JS app), any services needed (like node, npm), and OS Layer (Linux). Users can also add environment variables, create directories, files etc.

    It can alos be seen as an immutable template that defines how a container will be realized

- **Docker Containers**

    Docker Conatianer is a running instance of an Docker Image. From one Image, we can run multiple containers. It usually happens when we want to run multiple process of the same application for increasing the performance.
 
# Docker Registories

Docker Registories is a storage and distribution system for Docker Images. There are many official images available from applications like Redis, Mongo, Postgres etc.

Official images are maintained by the software authors or in collaboration with the Docker community.

Docker hosts one of the biggest Docker Registory, called "Docker Hub", in which we can find and share Docker images. The images are from different sources including companies, docker themselves and individual developers.

Official Images are maintained by a dedicated team who is responsible for reviewing and publishing all content in the Docker Official Images repositories. They work with the software developer or maintainers and security experts to create and manage those official images. This ensure the images are created by the best practice for both the developers side and Docker experts prespective.

# Image Versioning
With the changing of technology, the application must have different versions, therefore, different images will be created. Docker images are versioned as well, which is also known as tags. We can find the image tags with the version of library we need. In Docker Registories, the default image tag is `latest`.

# How to use Docker
**Download Image**

Use `docker pull` command in terminal.

```bash
docker pull {name}:{tag}
```

For example, if we want to download the `nginx` image with version 1.23, we can use:

```bash
docker pull nginx:1.23
```

**Run an Image**

Use `docker run` command in terminal. A container will be created.

```bash
docker run {name}:{tag}
```

`-d` allows the container running on a detached mode. The conatiner will keep running without occupying the current terminal session.

```bash
docker run -d {name}:{tag}
```
 
**Note:** we can use `run` directly without a `pull` command.

# Port Binding

How can we use/access the running container? Since the application inside the conatiner runs in a isolated Docker network, we are not able to access them so far (say, access from the broswer). We need to expose the container port to the host. That is port binding.

Usually, a container is running on a default container port, we can check it by `docker ps`. But the port is not published to the host, therefore, we should bind the port when we creating a new container. For example:

```bash
docker run -d -p 9000:80 nginx:1.23
```
This will run the nginx version 1.23 on port 80 and publish it to port 9000.

As for how to choose the publishing port id, the best practice is useing the same port as the conatiner is running.

# Start and stop containers

`docker run` create a new container every time. It doesn't reuse previous container. `docker ps` only shows the running containers, while `docker ps -a` shows all containers including the inactive ones.

- To control the existing containers, we need to show all the existing containers by `docker ps -a`

- Use `docker stop {Container id}` to stop a container.
- Use `docker start {Container id}` to start an existing container.

Instead of useing the Container id, which is hard to remember, we can use the name of the container. The name can be generated by docker automatically or specified when we run a container. For example:

```bash
docker run --name Nginx -d -p 9000:80 nginx:1.23
```

# Public and Private Docker Registories

DockerHub is the largest public registory where anyone can search and download Docker images. But If a company create their own Docker Images, this should not be accessed to the public. Therefore, a private Docker registory is needed.

Many cloud services provide private Docker Registries services 
- Amazon Web Service (AWS), Google Cloud Platform (GCD), Microsoft Azure, etc. 
- Nexus is a popular and useful artifact repository manager. 
- Docker Hub

We need to authenticate before accessing the registory.

# Docker Registory vs. Repository

**Docker Registry**

Docker Registry is a service providing storage, which can be hosted by a third party like AWS or by yourself. 

**Docker Repository**

Inside the registry, we can have multiple repositories to store your images. Each repository usually is a collection of images with the same name but different versions.

For example, Docker Hub is a Docker Registory, which is a collection of repositories. We can host private or public repositories for our applications.

# Dockerfile- Create own Images

Company need to create their own Images for application. How can we do that?

**Scenario:** Suppose you have completed your application and wanted to deploy to the server for end users. To make the deployment easier, you dicided to package all the applications such as the service, the database and other related part to Docker Images and run in a Docker Comtainer.

We need to create a `definition` of how to build our application to a Docker Image. The definition is what we called a `Dockerfile`. 

- Dockerfile is a tex tdocument that contains commands to assemble an image.
- Docker can then build an image by reading those instruction.

## Experiment
- We will write a Dockerfile for Node.js application.
- Then build a Docker image from it.

Suppose we have a `node.js` application has the following structure:
```
src
    - server.js
package.json
Dockerfile
```
**Structure of Dockerfile**

- Dockerfiles start form a parent image or "base image"
- It's a Docker image that your image is based on.

For example, if your application is based on `node.js`, we should choose the `node` Image. If it is based on `Python`, we should choose the `python` image. The following code will set up our base image.


```Dockerfile
FROM node:19-alpine
```
**`FROM`:** build this image from the specified image


Since this image is linux-based, we can run any linux commands. Next, we should install the `npm`.

```Dockerfile
RUN npm install
```

**`RUN`:** execute any command in a shell inside the conatiner environment.

To run the application, we should inlucde the application code inside. Therefore, we need to copy the code to the docker image.

```Dockerfile
COPY package.json /app/
COPY src /app/
```

**`COPY`:** Copies files or directories form `<src>` and adds them to the filesystem of the container at the path `<dest>`.

While `RUN` is executed in the container, `COPY` is executed on the host.

Before run the application, we should enter the working directory.

```Dockerfile
WORKDIR /app
```

**`WORKDIR`:** Set the working directory for all following commands, just like changing into a directory using `cd`

Finally, we should run the application, if it is the last step of the dockerfile, we use `CMD` instead of `RUN` to execute the application.

```Dockerfile
CMD ["node", "server.js"]
```

**`CMD`:** The instruction that is to be executed when a Docker container starts. There can only be one `CMD` instruction inta Dockerfile.

Here is the complete Dockerfile for creating a node.js application:

```Dockerfile
FROM node:19-alpine

COPY package.json /app/
COPY src /app/
WORKDIR /app

RUN npm install
CMD ["node", "server.js"]
```

# Build Image

Now we have a dockerfile to define our Image. Next we need to build the Docker image from the docker file. This can be done via terminal.

```Dockerfile
docker build -t node-app:1.0 .
```

`-t` or `--tag`: setting a neame and optionally a tag in the `{name}:{tag}` format.

`.` is the location of the docker image. This is the current location.

After the building, we can run the image by a `run` command:

```Dockerfile
docker run -d -p 3000:3000 node-app:1.0
```

We can check the runngin container by `docker ps`

# Docker UI Client

In docker desktop, we can do all of those above with interactive interface.

![img](/assets/post_img/2024-03-30-Docker-in-a-Nutshell/DockerUI.png)

It gives us a good overview of what containers we have, which one is active, port id and so on.

# How Docker fits in in the complete development and deployment process?
![img](/assets/post_img/2024-03-30-Docker-in-a-Nutshell/DockerCycle.png)

# Reference

Thanks for the [one hour Docker Crash Course](https://www.youtube.com/watch?v=pg19Z8LL06w) by youtuber : TechWorld with Nana