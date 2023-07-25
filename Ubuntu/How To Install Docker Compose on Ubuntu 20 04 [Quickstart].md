# How To Install Docker Compose on Ubuntu 20 04 [Quickstart]

```Docker``` ```Ubuntu``` ```Container``` ```Quickstart```

## Introduction


In this quickstart guide, we’ll install Docker Compose on an Ubuntu 20.04 server.


For a more detailed version of this tutorial, with more explanations of each step, please refer to How To Install and Use Docker Compose on Ubuntu 20.04.


# Prerequisites


To follow this guide, you’ll need access to an Ubuntu 20.04 server or local machine as a sudo user, and Docker installed on this system.


# Step 1 — Download Docker Compose


Start by confirming the most recent Docker Compose release available in their releases page. At the time of this writing, the most current stable version is 1.26.0.


Run the following command to download Docker Compose and make this software globally accessible on your system as docker-compose:


```
sudo curl -L "https://github.com/docker/compose/releases/download/1.26.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose


```


# Step 2 — Set Up Executable Permissions


Next, set  the correct permissions to make sure the docker-compose command is executable:


```
sudo chmod +x /usr/local/bin/docker-compose


```


To verify that the installation was successful, run:


```
docker-compose --version


```


You’ll see output similar to this:


```
Outputdocker-compose version 1.26.0, build 8a1c60f6

```


Docker Compose is now successfully installed on your system.


# Related Tutorials


Here are links to more detailed guides related to this tutorial:


- Initial Server Setup on Ubuntu 20.04
- How To Install and Use Docker Compose on Ubuntu 20.04
- How To Install and Use Docker on Ubuntu 20.04
- How To Install and Set Up Laravel with Docker Compose on Ubuntu 20.04

