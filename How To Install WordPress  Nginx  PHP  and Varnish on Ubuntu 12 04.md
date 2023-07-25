# How To Install WordPress  Nginx  PHP  and Varnish on Ubuntu 12 04

```Ubuntu``` ```WordPress``` ```Nginx``` ```PHP``` ```Server Optimization``` ```Caching```











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


Varnish is an HTTP accelerator and a useful tool for speeding up a server, especially during a times when there is high traffic to a site. It works by redirecting visitors to static pages whenever possible and only drawing on the  server itself if there is a need for an active process.


# Setup


Before you start to work through this tutorial, there are a couple prerequisites. You will need a user with root privileges, the LEMP stack, and Wordpress already installed on your server.


You can run through a few of the previous tutorials to make sure that your server is up to speed:


1. To create a user with sudo privileges, go through the third and fourth steps of the Initial Ubuntu Server Setup

# Step One—Install Varnish


Once you have all of the prerequisites needed to configure varnish with wordpress, you should go ahead and start the process to install Varnish.


The varnish site recommends installing the varnish package through their repository.


You can start that process by grabbing the repository:


```
sudo curl http://repo.varnish-cache.org/debian/GPG-key.txt | sudo apt-key add -
```


The next step is to add the repository to the list of apt sources. Go ahead and open up the file.


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
sudo apt-get install varnish libvarnish-dev
```


# Step Two—Configure Varnish


Once you have both nginx and varnish installed, you can start to configure them to ease the load on your virtual private server.


Varnish will serve the content on port 80, while fetching it from nginx which will run on port 8080.


Go ahead and start setting that up by opening the /etc/default/varnish file:


```
sudo nano /etc/default/varnish
```


Find the lines under “DAEMON_OPTS”— in the Alternative 2 section, and change the port number by "-a" to 80. The configuration should match the following code:


```
 DAEMON_OPTS="-a :80 \
             -T localhost:6082 \
             -f /etc/varnish/default.vcl \
             -S /etc/varnish/secret \
             -s malloc,256m"
```


That's the only change you need to make there. Save and exit out of that file and open up the default.vcl file:


```
sudo nano /etc/varnish/default.vcl
```


This file tells varnish where to look for the webserver content. It should already be configured to have the backend (ie. nginx) listening in on port 8080.


We need to use this file for a secondary purpose. Wordpress is chock full of various cookies that make caching it very difficult. In order to have varnish work as efficiently as possible, we need to tell it to drop all the cookies that don't relate to the admin side of the Wordpress site.


Additionally, we need to tell varnish to remove the cookies that make worpdress very difficult to cache.


The beginning of the default.vcl file should look like this:


```
[...]
backend default {
    .host = "127.0.0.1";
    .port = "8080";
}

# Drop any cookies sent to Wordpress.
sub vcl_recv {
        if (!(req.url ~ "wp-(login|admin)")) {
                unset req.http.cookie;
        }
}

# Drop any cookies Wordpress tries to send back to the client.
sub vcl_fetch {
        if (!(req.url ~ "wp-(login|admin)")) {
                unset beresp.http.set-cookie;
        }
}

[...]
```


# Step Three—Configure Nginx


Although we have already configured varnish to expect that nginx ports will be running on 8080, the default settings for nginx are still on port 80. We will correct the discrepancy now.


Open up the virtual host file with the Wordpress information. In the previous Wordpress tutorial we simply called it wordpress:


```
sudo nano /etc/nginx/sites-available/wordpress
```


The Virtual Host should also be set to port 8080 and be accessible only from the localhost. The updated line looks like this:


```
[...]
server {
        listen  127.0.0.1:8080; ## listen for ipv4; this line is default and implied
        [...]
```


We need to do one last thing prior to starting varnish running on our site, and that is to remove the default enabled virtual host.


```
sudo rm /etc/nginx/sites-enabled/default
```


The template will remain in the sites-available directory, should you need it once more.


# Step Five—Restart


Once you have made all of the required changes, restart varnish and nginx.


```
sudo service nginx restart
```


```
sudo service varnish restart
```


Accessing your domain should instantly take you to the varnish cached version, and you can see the details of varnish’s workings on your VPS with this command:


```
varnishstat
```


By Etel Sverdlov
