# How To Install Docker On Ubuntu 13 04 x64 VPS

```Docker``` ```Ubuntu```



Status: Deprecated
This article is deprecated and no longer maintained.
Reason:
This article targets a version of Ubuntu that is no longer supported.
See Instead:
An updated version of this article is available at  How To Install and Use Docker on Ubuntu 16.04

## Introduction


In case you're not familiar with Docker, here is the summary of it and its functionality:



  Docker is an open-source engine which automates the deployment of applications as highly portable,
  self-sufficient containers which are independent of hardware,
  language, framework, packaging system and hosting provider.

In this tutorial, we're going to build and install Docker on a fresh Ubuntu 13.04 VPS (cloud server) instance from DigitalOcean.


# Why Build it?


Docker is still in Alpha, and as such, there's considerable changes and fixes daily. Plus, if you find anything you'd like to improve or fix yourself, it's much easier to contribute if you're already building from source!


# Step 1: Create the Virtual Private Server


Docker currently only supports Ubuntu 12.04, 12.10, and 13.04, and only in 64bit architecture. It's currently slightly simpler to install Docker on Ubuntu 13.04 x64.


![](https://assets.digitalocean.com/articles/docker/img1.png)
Create the VPS and once it's done, go ahead and SSH in.


# Step 2: Setup the VPS


First, set up a user account, so that we have a place to keep Go and Docker (The user's Home directory). Here, replace USER with your own username.


```
sudo useradd -m -d /home/USER -s /bin/bash -U USER

```


Give yourself a password with:


```
passwd USER

```


For the purposes of this tutorial, we'll give this user all privileges for use of the sudo command; let's add a group called admin and add our new user to it. This will allow them use of the sudo command.


```
groupadd admin && usermod -a -G admin USER

```


Login with:


```
su USER

```


We'll use our home directory to store both Go and the Docker repository, so enter it now with:


```
cd ~/

```


Next, we need to install some dependencies.


```
sudo apt-get update
sudo apt-get install linux-image-extra-`uname -r`

```


When confronted with the following screen, make sure you keep the local version currently installed.


![](https://assets.digitalocean.com/articles/docker/img2.png)
# Step 3: Install Go


Docker requires Go 1.1, which we are now going to download, as currently aptitude will install Go 1.0.2.


```
wget http://go.googlecode.com/files/go1.1.1.linux-amd64.tar.gz
tar xf go1.1.1.linux-amd64.tar.gz
rm go1.1.1.linux-amd64.tar.gz

```


We need to add some environmental varibles to our .profile to define where our Go installation lives, with


```
echo "export GOROOT=\$HOME/go" >> ~/.profile
echo "PATH=$PATH:\$GOROOT/bin" >> ~/.profile
source ~/.profile

```


You should now be able to see the currently installed version of Go. This should give Go version go1.1.1 linux/amd64 or similar.


We now need create a folder and add some more environment variables to our .profile.


```
mkdir ~/gocode
echo "export GOPATH=\$HOME/gocode" >> ~/.profile
echo "PATH=\$PATH:\$GOPATH/bin" >> ~/.profile
source ~/.profile

```


The documentation refers to the $GOPATH variable as:



  a colon-separated list of paths inside which Go code, package objects, and executables may be found.

Our $GOPATH is where we're going to be storing all of the Docker source and dependencies. Later on, this will let us build Docker with the Go install command, which is handy.


# Step 4: Install Docker


Install the other dependencies for Docker:


```
sudo apt-get install lxc curl xz-utils git mercurial

```


Create the folder structure for building:


```
mkdir -p $GOPATH/src/github.com/dotcloud

```


Clone the Docker repository from github:


```
cd $GOPATH/src/github.com/dotcloud
git clone https://github.com/dotcloud/docker.git

```


We can now use Go's helpful go get function to download and install the packages and dependencies we need to build Docker:


```
cd $GOPATH/src/github.com/dotcloud/docker
go get -v github.com/dotcloud/docker/...

```


This will also install Docker. To allow us to run the Docker executable as root without specifying the full path, we can symlink it to /usr/local/bin with:


```
sudo ln -s $GOPATH/bin/docker /usr/local/bin/docker

```


Now run Docker:


```
sudo docker -d &

```


After the previous command has been executed and started, hitting Enter will keep Docker running in the background. Let's test it out! First, pull the base container image:


```
docker pull base

```


Once it's downloaded, test launch a new container with:


```
docker run -t base /bin/echo "Hello, world."

```


Updating Docker is now as simple as stopping the instance, deleting the binary, pulling the repository, using Go install, and restarting Docker:


```
sudo kill $(cat /var/run/docker.pid)
rm $GOPATH/bin/docker
cd $GOPATH/src/github.com/dotcloud/docker && git pull origin master
go install -v github.com/dotcloud/docker/...
sudo docker -d &

```


# Conclusion


If you're still new to Docker, there's already a few great resources to get you started:
The Docker Index is a place you can find community-made Docker images, ready to Docker pull into your new installation.
If you'd prefer to build a few Dockerfiles yourself, there's a good collection to be found here.


 Submitted by  Fareed Dudhia 
