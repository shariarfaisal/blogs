# How To Install and Configure Varnish with Apache on Ubuntu 12 04

```Apache``` ```Ubuntu``` ```Caching```











# Status: Deprecated


This article covers a version of Ubuntu that is no longer supported.  If you are currently operate a server running Ubuntu 12.04, we highly recommend upgrading or migrating to a supported version of Ubuntu:


- Upgrade to Ubuntu 14.04.
- Upgrade from Ubuntu 14.04 to Ubuntu 16.04
- Migrate the server data to a supported version

Reason:
Ubuntu 12.04 reached end of life (EOL) on April 28, 2017 and no longer receives security patches or updates.  This guide is no longer maintained.


See Instead:
This guide might still be useful as a reference, but may not work on other Ubuntu releases.  If available, we strongly recommend using a guide written for the version of Ubuntu you are using.   You can use the search functionality at the top of the page to find a more recent version.


## About Varnish


Varnish is an HTTP accelerator and a useful tool for speeding up a server, especially during a times when there is high traffic to a site. It works by redirecting visitors to static pages whenever possible and only drawing on the virtual private server itself if there is a need for an active process.


# Setup


To perform the steps in this tutorial, you will need to both have a user with sudo privileges and apache installed on your virtual private server.


To create a user with sudo privileges, go through the third and fourth steps of the initial ubuntu server setup tutorial


Apache can be installed on your VPS with a single command from the apt-get repository.


```
sudo apt-get install apache2
```


# Step One—Install Varnish


The varnish site recommends installing the varnish package through their repository.


You can start that process by grabbing the repository:


```
sudo curl http://repo.varnish-cache.org/debian/GPG-key.txt | sudo apt-key add -
```


The next step is to add the repository to the list of apt sources. Go ahead and open up that file.


```
sudo nano /etc/apt/sources.list
```


Once inside the file, add the varnish repository to the list of sources.


```
deb http://repo.varnish-cache.org/ubuntu/ lucid varnish-3.0
```


Save and exit.


Finally, update apt-get and install varnish.


```
sudo apt-get update
sudo apt-get install varnish
```


# Step Two—Configure Varnish


Once you have both apache and varnish installed, you can start to configure them to ease the load on your server from future visitors.


Varnish will serve the content on port 80, while fetching it from apache which will run on port 8080.


Let’s go ahead and start setting that up by opening the /etc/default/varnish file:


```
sudo nano /etc/default/varnish
```


Uncomment all of the lines under “DAEMON_OPTS”—under Alternative 2, and make the configuration match the following code:


```
 DAEMON_OPTS="-a :80 \
             -T localhost:6082 \
             -f /etc/varnish/default.vcl \
             -S /etc/varnish/secret \
             -s malloc,256m"
```


Once you save and exit out of that file, open up the default.vcl file:


```
sudo nano /etc/varnish/default.vcl
```


This file tells varnish where to look for the webserver content. Although Apache listens on port 80 by default, we will change the settings for it later. Within this file, we will tell varnish to look for the content on port 8080.


The configuration should like this:


```
backend default {
    .host = "127.0.0.1";
    .port = "8080";
}
```


# Step Three—Configure Apache


So far we have told varnish that apache ports will be running on 8080. However the default settings for apache are still on port 80. We will correct the discrepancy now. 
Open up the apache ports file:


```
sudo nano /etc/apache2/ports.conf
```


Change the port number for both the NameVirtualHost and the Listen line to port 8080, and the virtual host should only be accessible from the localhost. The configuration should look like this:


```
NameVirtualHost 127.0.0.1:8080
Listen 127.0.0.1:8080
```


Change these settings in the default virtual host file as well:


```
sudo nano /etc/apache2/sites-available/default
```


The Virtual Host should also be set to port 8080, and updated line looks like this:


```
 <VirtualHost 127.0.0.1:8080>
```


Save and exit the file and proceed to restart Apache and Varnish to make the changes effective.


```
sudo service apache2 restart
sudo service varnish restart
```


Accessing your domain should instantly take you to the varnish cached version, and you can see the details of varnish’s workings with this command:


```
varnishstat
```


By Etel Sverdlov
