# How To Install And Run A Node js App On Centos 6 4 64bit

```Node.js``` ```CentOS```










# Status: Deprecated


This article covers a version of CentOS that is no longer supported. If you are currently operating a server running CentOS 6, we highly recommend upgrading or migrating to a supported version of CentOS.


Reason:
CentOS 6 reached end of life (EOL) on November 30th, 2020 and no longer receives security patches or updates. For this reason, this guide is no longer maintained.


See Instead:
This guide might still be useful as a reference, but may not work on other CentOS releases. If available, we strongly recommend using a guide written for the version of CentOS you are using.


## Introduction


This article outlines the steps necessary to run a "Hello world" in node.js + express, running on a 64bit Centos 6.4 installation. We will be building the latest version of source (at this moment, v0.10.4) from the upstream provider.


As is written on their homepage, Node.js is a platform built on Chrome's JavaScript runtime for easily building fast, scalable network applications. This is a fast, event driven platform and server-side Javascript engine used for building web applications. DigitalOceans' droplets are a cost-effective way to install and start studying server-side Javascript and bulding or deploying web applications with Node.js.


# Setup a VPS


To get started, we will need a droplet - the smallest instance will do just enough - and a SSH client (ie. Putty on Windows, Linux systems and Mac Os X usually have it out of the box). When we receive our initial root password, we can ssh into the instance. SSH into the VPS and change the root password, if you haven't already. It would probably be a good idea to also update software repository to the latest versions:


```
yum -y update
```


This will update installed software on our VPS to the latest versions.


Yum can take a few minutes, but when it's done, we need to prepare for software installation. We're going to build Node.js from the latest source available at the moment of writing this (v0.10.4). To do that, we'll need "Development Tools". It's a group of tools for compiling software from sources.


```
yum -y groupinstall "Development Tools"
```


This command will pull a "Development Tools" group with the applications needed to compile node.js.


Also, we'll install GNU screen - a piece of software that allows us to connect to our VPS, start a session and detach from it. We could disconnect and connect later, or from another workstation, and pick up where we left. It's very handy, especially during development of an app, when we want to learn stuff.


```
yum -y install screen
```


# Node.js installation


Now that we're ready to install Node.js from sources. First, we'll move to /usr/src directory - the usual place to hold software sources.


```
cd /usr/src
```


Now, we pick the latest compressed source archive from Node.js website at http://nodejs.org/download/.


```
wget http://nodejs.org/dist/v0.10.4/node-v0.10.4.tar.gz
```


We could and should replace the url and use the more recent version of node.js, if there is one. Next, we are uncompressing the source files and move into that directory.


```
tar zxf node-v0.10.4.tar.gz
cd node-v0.10.4
```


Now the source for Node.js is extracted and we're in the source directory. We can now prepare our compiler commands, by executing the configure script:


```
./configure
```


It will read the properties of our system to prepare compiler flags. Ie. it could be a system architecture (32/64bit, CPU specific flags etc). With it, we're ready to actually compile the source now. To do that, just type:


```
make
```


This is probably the most time-consuming task here: on my example VPS it took about 6 minutes and 34 seconds to complete. When we're done, we need to make this available system-wide:


```
make install
```


The latest command will place the compiled binaries in system path, so all users could use it without any further setup. By default, node binary should be installed in /usr/local/bin/node.


# Install Express.js


We now have Node.js installed and complete, we can start developing right away, deploy an already done application or we can proceed to create our Express.js application. First, we'll use npm, nodes' module manager, to install express middleware and supervisor - a helpful module that keeps our app started, monitors for file changes (ie. when we're developing the app) and restarts the VPS when needed.


UPDATE: To be able to run an executable in /usr/local/bin through sudo, you have add /usr/local/bin to your secure_path using visudo.


```
sudo visudo
```


Look for secure_path, and append the following to it: ":/usr/local/bin". Once you have done that, you're now ready to install the express and supervisor modules.


```
npm -g install express express-generator supervisor
```


npm -g install will install the express and supervisor modules from npm software repository and make it available to the whole system. The -g switch in this command means "global" - the express and supervisor commands will be available accross the whole system.


# Add non-privileged user


You should now, for security reasons, create a regular system user and run node under non-privileged account.


To do this, add the user first. You can replace "exampleuser" with whatever name your prefer.


```
useradd exampleuser
```


We have a new system user. Add a decent password for the new user:


```
passwd exampleuser
```


Log out, and log back in as the new user.


This changes our login shell from root (system user) to exampleuser (non-privileged user who can compromise the system with less damage).


# Creating an express app


Express is powerfull framework, and to create our first application, all we have to do is type:


```
express hello
```


The command will create a "hello" directory and setup some basics for a new application. Now we should enter this directory and install express dependencies:


```
cd hello && npm install
```


npm install part of the command will read all the module dependencies from a generated package.json file and install it from npm software repository.


We should start a new screen session so we can leave node app running:


```
screen
```


Finally, we can start our application with the help of supervisor that we installed earlier.


```
supervisor ./bin/www
```


Now we're able to access our first express app at your VPS IP. For example http://123.456.78.90:3000/.


