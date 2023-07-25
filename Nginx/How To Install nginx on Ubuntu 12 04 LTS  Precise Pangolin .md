# How To Install nginx on Ubuntu 12 04 LTS  Precise Pangolin 

```Ubuntu``` ```Nginx```











# Status: Deprecated


This article covers a version of Ubuntu that is no longer supported.  If you are currently operate a server running Ubuntu 12.04, we highly recommend upgrading or migrating to a supported version of Ubuntu:


- Upgrade to Ubuntu 14.04.
- Upgrade from Ubuntu 14.04 to Ubuntu 16.04
- Migrate the server data to a supported version

Reason:
Ubuntu 12.04 reached end of life (EOL) on April 28, 2017 and no longer receives security patches or updates.  This guide is no longer maintained.


See Instead:
This guide might still be useful as a reference, but may not work on other Ubuntu releases.  If available, we strongly recommend using a guide written for the version of Ubuntu you are using.   You can use the search functionality at the top of the page to find a more recent version.


##  About nginx


nginx is a high performance web server software.  It is a much more flexible and lightweight program than apache.


# Set Up


The  steps in this tutorial require the user to have root privileges. You can see how to set that up in the Initial Server Setup Tutorial in steps 3 and 4.


# Step One—Install nginx


To install nginx, open terminal and type in:


```
sudo apt-get install nginx
```


When prompted, say yes. nginx is now installed on your virtual private server.


# Step Two—Start nginx


nginx does not start on its own. To get nginx running on your VPS, type:


```
sudo service nginx start
```


# Step Three—RESULTS: Confirm That nginx Has Started


You can confirm that nginx has been installed as your web server by directing your browser to your IP address.


**You can run the following command to reveal your virtual server’s IP address.


```
ifconfig eth0 | grep inet | awk '{ print $2 }'
```


When you visit your IP address page in your browser, you will see the words, “Welcome to nginx”


You can see a screenshot of the utilitarian nginx welcome page here


To ensure that nginx will be up after reboots, it’s best to add it to the startup.
Type this command into terminal:


```
update-rc.d nginx defaults
```


You may see a message like:


```
System start/stop links for /etc/init.d/nginx already exist.
```


If that is the case, then nginx is set up to run on startup, and you are all set.


Congratulations! You have now installed nginx


# See More


Once you have installed nginx on your virtual private server, you can do a variety of things on your server such as Set Up Virtual Hosts or Create an SSL certificate for your site


By Etel Sverdlov
