# How To Dockerise And Deploy Multiple WordPress Applications On Ubuntu

```Docker``` ```Ubuntu``` ```WordPress``` ```Email```

## Introduction



WordPress has become one of the most popularly deployed and used web applications the world has ever seen. Thanks to years of constant development, it is now possible to create a nearly endless amount of different websites (or even web-applications) based on WordPress and its available plug-ins / extensions.


In this DigitalOcean article, using the Docker Linux Container Engine, we are going learn how to dockerise (i.e. package and contain) WordPress applications on Ubuntu cloud servers and discover what probably is the most simple and secure way of deploying multiple WordPress sites on a single host.


# Glossary



## 1. Docker In Brief



## 2. WordPress In Brief



## 3. Installing Docker on Ubuntu (latest)



## 4. Working With Docker



1. Command Line Interface Usage And The Daemon
2. Client Commands

## 5. Working With Dockerfiles



1. What Are Dockerfiles?
2. Dockerfile Commands Overview

## 6. Creating WordPress Containers



1. Pulling The Image
2. Creating A Publicly Accessible WordPress Container
3. Creating A Locally Accessible WordPress Container
4. Limiting The Memory Usage For Containers

# Docker In Brief



The Docker project offers higher-level tools, working together, which are built on top of some Linux kernel features with the goal of helping developers and system administrators port applications - with all of their dependencies conjointly - and get them running across systems and machines headache free.


Docker achieves this by creating safe, LXC (Linux Containers) based environments for applications called “containers” which are created using images. These bases for containers can be built either by executing commands manually by logging inside like a virtual-machine, or by automating the process through Dockerfiles.


Note: To learn more about Docker and its parts (i.e. the docker daemon, CLI, images, etc.), check out our introductory article to the project: Docker Explained: Getting Started.


# WordPress In Brief



WordPress was initially created as an easy to install and use self-publication platform (i.e. a Blogging engine). It has become extremely popular over the years, which lead to development of many 3rd party plugins, turning the tool into a full CMS (Content Management System). Based on WordPress, a lot of different types of web-sites and web-applications can be created with simplicity and deployed with ease.


WordPress is an open-source platform that is developed using the PHP programming language, which surely helped it on its way to success. PHP is currently one of the most common web-site and web-application creation languages and the choice of many companies (including Facebook).


WordPress sites rely on MySQL relational-database to keep their data and there are multiple ways to power a WordPress site given the multiple choices available to run PHP and MySQL together.


In this article, we will go with a tried-and-tested method to create WordPress installed Docker images, which will enable you to run yet-another WordPress site on any VPS, with a single command by using Docker.


# Installing Docker on Ubuntu (latest)



## Update your droplet



```
sudo apt-get    update
sudo apt-get -y upgrade

```


## Make Sure aufs Support Is Available



```
sudo apt-get install linux-image-extra-`uname -r`

```


## Add The Docker Repository Key To apt-key For Package Verification



```
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 36A1D7869245C8950F966E92D8576A8BA88D21E9

```


## Add The Docker Repository To Sources



```
sudo sh -c "echo deb http://get.docker.io/ubuntu docker main\
> /etc/apt/sources.list.d/docker.list"

```


## Update The Repository



```
sudo apt-get update

```


## Download And Install Docker



```
sudo apt-get install lxc-docker git

```


Ubuntu’s default firewall (UFW: Uncomplicated Firewall) denies all forwarding traffic by default, which is needed by docker.


## Enable Forwarding With UFW



Edit UFW configuration using the nano text editor.


```
sudo nano /etc/default/ufw

```


Scroll down and find the line beginning with DEFAULT_FORWARD_POLICY.


Replace:


```
DEFAULT_FORWARD_POLICY="DROP"

```


with:


```
DEFAULT_FORWARD_POLICY="ACCEPT"

```


Press CTRL+X and approve with Y to save and close.


## Reload The UFW



```
sudo ufw reload

```


## Allowing Remote Connections



If you are planning on using the docker daemon remotely, then you will need to allow the default Docker port 4243.


```
sudo ufw allow 4243/tcp

```


# Working With Docker



Before we begin working with docker, let’s quickly go over its available commands to refresh our memory from our first Getting Started article.


## Command Line Interface Usage And The Daemon



Upon installation, the docker daemon should be running in the background, ready to accept commands sent by the docker client. For certain situations where it might be necessary to manually run Docker, use the following.


Running the docker daemon:


```
sudo docker -d &

```


Client Usage:


```
sudo docker [option] [command] [arguments]

```


Note: Docker needs sudo privileges in order to work as it uses sockets owned by root.


## Client Commands



You can get a full list of all available commands by simply calling the client:


```
docker

```


Here is a list of all available commands as of version 0.8.0:


```
Commands:
    attach    Attach to a running container
    build     Build a container from a Dockerfile
    commit    Create a new image from a container's changes
    cp        Copy files/folders from the containers filesystem to the host path
    diff      Inspect changes on a container's filesystem
    events    Get real time events from the server
    export    Stream the contents of a container as a tar archive
    history   Show the history of an image
    images    List images
    import    Create a new filesystem image from the contents of a tarball
    info      Display system-wide information
    insert    Insert a file in an image
    inspect   Return low-level information on a container
    kill      Kill a running container
    load      Load an image from a tar archive
    login     Register or Login to the docker registry server
    logs      Fetch the logs of a container
    port      Lookup the public-facing port which is NAT-ed to PRIVATE_PORT
    ps        List containers
    pull      Pull an image or a repository from the docker registry server
    push      Push an image or a repository to the docker registry server
    restart   Restart a running container
    rm        Remove one or more containers
    rmi       Remove one or more images
    run       Run a command in a new container
    save      Save an image to a tar archive
    search    Search for an image in the docker index
    start     Start a stopped container
    stop      Stop a running container
    tag       Tag an image into a repository
    top       Lookup the running processes of a container
    version   Show the docker version information
    wait      Block until a container stops, then print its exit code

```


# Working With Dockerfiles



## What Are Dockerfiles?



Dockerfiles are scripts containing commands declared successively which are to be executed, in the order given, by Docker to automatically create a new image.


These files always begin with the definition of a base image with the FROM instruction. From there on, the build process starts and each following action forms the final image with commits (i.e. saving the image state).


Dockerfiles can be used with the build command:


```
# Build an image using the Dockerfile at current location
# Tag the final image with [name] (e.g. *wordpress_img*)
# Example: sudo docker build -t [name] .
sudo docker build -t wordpress_img . 

```


Note: To learn more about Dockerfiles, check out the article: Docker Explained: Using Dockerfiles to Automate Building of Images.


## Dockerfile Commands Overview



Dockerfiles work by receiving the below instructions:


- 
ADD: Copy a file from the host into the container

- 
CMD: Set default commands to be executed, or passed to the ENTRYPOINT

- 
ENTRYPOINT: Set the default entrypoint application inside the container

- 
ENV: Set environment variable (e.g. key = value)

- 
EXPOSE: Expose a port to outside

- 
FROM: Set the base image to use

- 
MAINTAINER: Set the author / owner data of the Dockerfile

- 
RUN: Run a command and commit the ending result (container) image

- 
USER: Set the user to run the containers from the image

- 
VOLUME: Mount a directory from the host to the container

- 
WORKDIR: Set the directory for the directives of CMD to be executed


# Creating WordPress Containers



## Pulling The Image



For our tutorial, we will be using an out-of-the-box WordPress image called tutum/wordpress. This wordpress image is created using Tutum’s Wordpress Image: In order to create containers from this image, we need to pull (download) it first.


Let’s pull the image:


```
docker pull tutum/wordpress

```


This command will download the underlying base images with all modified layers.


Once the image is ready, by issuing a single command we can create dockerised WordPress instances.


## Creating A Publicly Accessible WordPress Container



Run the following command to create a container that is reachable from the outside on a port you specify (e.g. 80):


```
# Usage: docker run -p [Port Number]:80 tutum/wordpress
# Example:
docker run -p 80:80 tutum/wordpress

```


The above command will create a WordPress instance that will accept connections from the outside on the default HTTP port 80.


## Creating A Locally Accessible WordPress Container



Sometimes it might suit you the best to have containers reachable only locally. This can be useful if you decide to set up a load-balancer or another reverse-proxy to distribute connections across many WordPress instances.


Run the following command to create a locally accessible container.


```
# Allocate a port dynamically:
# Usage: docker run -p 127.0.0.1::80 tutum/wordpress
# Example:
docker run -p 127.0.0.1::80 tutum/wordpress

```


Once you execute the above command, Docker will create a container, provide you its ID and then dynamically allocate a port. You can figure out which port the container is using with the port command.


```
# Usage: docker port [container ID] [private port number]
# Example:
docker port 9af15d73fdf8a997 80

# 127.0.0.1:49156

```


In this case, the output means that the container is accessible only on the localhost on port 49156. You can use the address, provided in full, to redirect connections from a reverse-proxy.


If you would like to specify a port, just place it in-between the IP address and the private port used by the web server inside (e.g. 80):


```
# Usage: docker run -p 127.0.0.1:[local port]:80 tutum/wordpress
# Example:
docker run -p 127.0.0.1:8081:80 tutum/wordpress

```


This way, you will have a WordPress instance that is locally accessible at port 8081.


Note: In order to run your container in the background, you also need to add the -d flag after the run command:


```
docker run -d ..

```


Otherwise, you will be connected to the container where you will see the output from all the applications running.


In order to leave the container, as shown in the introduction article, you need to use the escape sequence CTRL+P immediately followed by CTRL+Q.


Using the docker ps command, you can get the list of running containers to find your newly instantiated one’s ID.


Note: Using the -name [name] arguments, you can tag a container with a name which should free you from dealing with complex container IDs:


```
docker run -d -name new_container_1 ..

```


## Limiting The Memory Usage For Containers



In order to limit the amount of memory a docker container process can use, simply set the -m [memory amount] flag with the limit.


To run a container with memory limited to 256 MBs:


```
# Example: docker run -name [name] -m [Memory (int)][memory unit (b, k, m or g)] -d (to run not to attach) -p (to set access and expose ports) [image ID]
docker run -m 64m -d -p 8082:80 tutum/wordpress

```


To confirm the memory limit, you can inspect the container:


```
# Example: docker inspect [container ID] | grep Memory
docker inspect 9a7562a361122706 | grep Memory

```


Note: The command above will grab the memory related information from the inspection output. To see all the relevant information regarding your container, opt for sudo docker inspect [container ID]. Also, please note that your Linux kernel must support swap limit capabilities for actual limitation to work.


For the full set of instructions to install and use docker, check out the docker documentation at docker.io.


<div class=“author”>Submitted by: <a
href=“https://twitter.com/ostezer”>O.S. Tezer</a></div>


