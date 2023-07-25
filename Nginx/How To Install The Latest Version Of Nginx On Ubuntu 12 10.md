# How To Install The Latest Version Of Nginx On Ubuntu 12 10

```Nginx```


Status: Deprecated
This article is deprecated and no longer maintained.
Reason
Ubuntu 12.04 reached end of life (EOL) on April 28, 2017 and no longer receives security patches or updates.
See Instead
This article may still be useful as a reference, but may not follow best practices or work on this or other Ubuntu releases. We strongly recommend using a recent article written for the version of Ubuntu you are using.

How To Install Nginx on Ubuntu 16.04
How To Secure Nginx with Let’s Encrypt on Ubuntu 16.04

If you are currently operating a server running Ubuntu 12.04, we highly recommend upgrading or migrating to a supported version of Ubuntu:

How to upgrade from Ubuntu 12.04 to Ubuntu 14.04.
How to upgrade from Ubuntu 14.04 to Ubuntu 16.04
How to migrate server data to a supported version


## About Nginx


Nginx is a free, Open Source Web server. It is much more lightweight than Apache and it can be used as the main web server software or be set up as a reverse proxy for Apache.


## Setup


Before using this tutorial, you will need to SSH into your VPS, by typing into the terminal: ssh <user>@<server_ip>. The user will need to either be in root or have root privileges, otherwise the commands entered below may not work.


# Step One - Installing Dependencies


The packages that you will need to install are python-software-properties and software-properties-common (which is only necessary if you are running Ubuntu 12.10).


To install the first package dependency, python-software-properties, you will need to run the following command:


```
sudo apt-get install python-software-properties

```


If you are on Ubuntu 12.10, you should run the following command to install software-properties-common, which is another package that is necessary (without it, the add-apt-repository command used in Step Two will not be found).


```
sudo apt-get install software-properties-common

```


# Step Two - Adding the Stable Nginx Repository


To ensure that our web server software is secure to run on a VPS, we will be using the latest ‘stable’ release.



If you are developing a nginx module, or if you need to use the “bleeding edge” version, you can replace the ‘stable’ version with the ‘development’ version. However I would not recommend doing this on a VPS, as there may be bugs.

Now that we have the latest stable package installed, we can now add the repository to install the latest version of nginx:


```
sudo add-apt-repository ppa:nginx/stable

```


Note: If this command still does not work (normally on 12.10), run the following command:


```
sudo apt-get install software-properties-common

```


This will add the repository to Ubuntu and fetches the repository’s key. This is to verify that the packages have not been interfered with since they have been built.


# Step Three - Updating the Repositories


After adding a new repository, you will need to update the list:


```
sudo apt-get update

```


# Step Four - Install nginx


To install nginx or update the version you already have installed, run the following command:


```
sudo apt-get install nginx

```


# Step Five - Check That Nginx is Running


You can check to see that nginx is running by either going to your VPS’ IP address/domain, or typing in:


```
service nginx status

```


This will tell you whether nginx is currently running.


# (Step Six - if Nginx is Not Running)


If nginx is not running correctly, and/or prints out an error e.g. nginx: [emerg] bind() to [::]:80 failed (98: Address already in use), you can run:


```
netstat -tulpn

```


This will list all processes listening on ports. You should see something like this:


The highlighted number, the PID, is the number that you will use to kill the process. In this case, you would need to run kill -9 734. However the general code to copy into your terminal would be:


```
kill -9 xxxx

```


The phrase, “xxxx”, is the PID of the process you want to kill. After killing the process, you can try restarting nginx again by running:


```
service nginx start

```


Alternatively, the issue may be caused by the configuration accepting connections from both ipv4 and ipv6. In order to resolve this, edit out “listen [::]:80” in your default config file (/etc/nginx/sites-available/default) and any other server block config files that are in use.


```
sudo nano /etc/nginx/sites-available/default

```


The lines should look like this:


```
server {
        listen 80;
        #listen [::]:80 default_server;

```


## Note:


You can view more information about the PPA release from: https://launchpad.net/~nginx/+archive/stable


