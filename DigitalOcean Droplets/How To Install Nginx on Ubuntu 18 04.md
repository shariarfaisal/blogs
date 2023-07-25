# How To Install Nginx on Ubuntu 18 04

```Nginx``` ```Ubuntu 18.04``` ```DigitalOcean Droplets```

## Introduction


Nginx is one of the most popular web servers in the world and is responsible for hosting some of the largest and highest-traffic sites on the internet. It is more resource-friendly than Apache in most cases and can be used as a web server or reverse proxy.


In this guide, you’ll learn how to install Nginx on your Ubuntu 18.04 server and about important Nginx files and directories.


# Prerequisites


Before you begin this guide, you should have a regular, non-root user with sudo privileges and a basic firewall configured on your server. You can learn how to configure a regular user account by following our initial server setup guide for Ubuntu 18.04.


When you have an account available, log in as your non-root user to begin.


# Step 1 – Installing Nginx


Since Nginx is available in Ubuntu’s default repositories, it is possible to install it from these repositories using the apt packaging system.


Since this may be your first interaction with the apt packaging system in this session, update the local package index so that you have access to the most recent package listings.  Afterward, you can install nginx:


```
sudo apt update
sudo apt install nginx


```


After accepting the procedure, apt will install Nginx and any required dependencies to your server.


# Step 2 – Adjusting the Firewall


Before testing Nginx, the firewall software needs to be adjusted to allow access to the service.  Nginx registers itself as a service with ufw upon installation, making it straightforward to allow Nginx access.


List the application configurations that ufw knows how to work with by typing the following:


```
sudo ufw app list


```


Your output should be a list of the application profiles:


```
OutputAvailable applications:
  Nginx Full
  Nginx HTTP
  Nginx HTTPS
  OpenSSH

```


This list displays three profiles available for Nginx:


- Nginx Full: This profile opens both port 80 (normal, unencrypted web traffic) and port 443 (TLS/SSL encrypted traffic)
- Nginx HTTP: This profile opens only port 80 (normal, unencrypted web traffic)
- Nginx HTTPS: This profile opens only port 443 (TLS/SSL encrypted traffic)

It is recommended that you enable the most restrictive profile that will still allow the traffic you’ve configured. Since you haven’t configured SSL for your server yet in this guide, you’ll only need to allow traffic on port 80.


You can enable this by typing the following:


```
sudo ufw allow 'Nginx HTTP'


```


Then, verify the change:


```
sudo ufw status


```


You should receive a list of HTTP traffic allowed in the output:


```
OutputStatus: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere                  
Nginx HTTP                 ALLOW       Anywhere                  
OpenSSH (v6)               ALLOW       Anywhere (v6)             
Nginx HTTP (v6)            ALLOW       Anywhere (v6)

```


Now that you’ve added the appropriate firewall rule, you can check that your web server is running and able to serve content correctly.


# Step 3 – Checking your Web Server


At the end of the installation process, Ubuntu 18.04 starts Nginx. The web server should already be up and running.


Check with the systemd init system to make sure the service is running:


```
systemctl status nginx


```


```
Output● nginx.service - A high performance web server and a reverse proxy server
   Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: en
   Active: active (running) since Fri 2021-10-01 21:36:15 UTC; 35s ago
     Docs: man:nginx(8)
 Main PID: 9039 (nginx)
    Tasks: 2 (limit: 1151)
   CGroup: /system.slice/nginx.service
           ├─9039 nginx: master process /usr/sbin/nginx -g daemon on; master_pro
           └─9041 nginx: worker process

```


This output shows that the service has started successfully.  However, the best way to test this is to actually request a page from Nginx.


You can access the default Nginx landing page to confirm that the software is running properly by navigating to your server’s IP address. If you do not know your server’s IP address, you can get it a few different ways.


Try typing the following at your server’s command prompt:


```
ip addr show eth0 | grep inet | awk '{ print $2; }' | sed 's/\/.*$//'


```


You will receive a few lines. You can try each in your web browser to confirm if they work.


An alternative is running the following command, which should generate your public IP address as identified from another location on the internet:


```
curl -4 icanhazip.com


```


When you have your server’s IP address, enter it into your browser’s address bar:


```
http://your_server_ip

```


You should receive the default Nginx landing page:





This page is included with Nginx to verify that the server is running correctly.


# Step 4 – Managing the Nginx Process


Now that you have your web server up and running, let’s review some basic management commands.


To stop your web server, type the following:


```
sudo systemctl stop nginx


```


To start the web server when it is stopped, type the following:


```
sudo systemctl start nginx


```


To stop and then start the service again, type the following:


```
sudo systemctl restart nginx


```


If you are simply making configuration changes, you can often reload Nginx without dropping connections instead of restarting it. To do this, type the following:


```
sudo systemctl reload nginx


```


By default, Nginx is configured to start automatically when the server boots. If this is not what you want, you can disable this behavior by typing the following:


```
sudo systemctl disable nginx


```


To re-enable the service to start up at boot, you can type the following:


```
sudo systemctl enable nginx


```


Nginx should now start automatically when the server boots again.


# Step 5 – Setting Up Server Blocks (Recommended)


When using the Nginx web server, server blocks (similar to virtual hosts in Apache) can be used to encapsulate configuration details and host more than one domain from a single server. We will set up a domain called your_domain, but you should replace this with your own domain name. To learn more about setting up a domain name with DigitalOcean, see our Introduction to DigitalOcean DNS.


Nginx on Ubuntu 18.04 has one server block enabled by default that is configured to serve documents out of a directory at /var/www/html. While this works well for a single site, it can become unwieldy if you are hosting multiple sites. Instead of modifying /var/www/html, let’s create a directory structure within /var/www for our your_domain site, leaving /var/www/html in place as the default directory to be served if a client request doesn’t match any other sites.


Create the directory for your_domain as follows, using the -p flag to create any necessary parent directories:


```
sudo mkdir -p /var/www/your_domain/html


```


Next, assign ownership of the directory with the $USER environment variable:


```
sudo chown -R $USER:$USER /var/www/your_domain/html


```


The permissions of your web roots should be correct if you haven’t modified your umask value, but you can make sure by typing the following:


```
sudo chmod -R 755 /var/www/your_domain


```


Next, create a sample index.html page using nano or your favorite editor:


```
nano /var/www/your_domain/html/index.html


```


Inside, add the following sample HTML:


/var/www/your_domain/html/index.html
```
<html>
    <head>
        <title>Welcome to your_domain!</title>
    </head>
    <body>
        <h1>Success! The your_domain server block is working!</h1>
    </body>
</html>

```


Save and close the file when you are finished. If you used nano, you can exit by pressing CTRL + X then Y and ENTER.


In order for Nginx to serve this content, it’s necessary to create a server block with the correct directives. Instead of modifying the default configuration file directly, make a new one at /etc/nginx/sites-available/your_domain:


```
sudo nano /etc/nginx/sites-available/your_domain


```


Add the following configuration block, which is similar to the default, but updated for your new directory and domain name:


/etc/nginx/sites-available/your_domain
```
server {
        listen 80;
        listen [::]:80;

        root /var/www/your_domain/html;
        index index.html index.htm index.nginx-debian.html;

        server_name your_domain.com www.your_domain;

        location / {
                try_files $uri $uri/ =404;
        }
}

```


Notice that we’ve updated the root configuration to the new directory, and the server_name to the domain name. Save and close the file when you are finished.


Next, enable the file by creating a link from it to the sites-enabled directory, which Nginx reads from during startup:


```
sudo ln -s /etc/nginx/sites-available/your_domain /etc/nginx/sites-enabled/


```


Two server blocks are now enabled and configured to respond to requests based on their listen and server_name directives (you can read more about how Nginx processes these directives here):


- your_domain: Will respond to requests for your_domain and www.your_domain.
- default: Will respond to any requests on port 80 that do not match the other two blocks.

To avoid a possible hash bucket memory problem that can arise from adding additional server names, it is necessary to adjust a single value in the /etc/nginx/nginx.conf file.  Open the file:


```
sudo nano /etc/nginx/nginx.conf


```


Find the server_names_hash_bucket_size directive and remove the # symbol to uncomment the line:


/etc/nginx/nginx.conf
```
...
http {
    ...
    server_names_hash_bucket_size 64;
    ...
}
...

```


Save and close the file when you are finished.


Next, test to make sure that there are no syntax errors in any of your Nginx files:


```
sudo nginx -t


```


If there aren’t any problems, restart Nginx to enable your changes:


```
sudo systemctl restart nginx


```


Nginx should now be serving your domain name. You can test this by navigating to http://your_domain, where you should see something like the following:





# Step 6 – Getting Familiar with Important Nginx Files and Directories


Now that you know how to manage the Nginx service itself, you should take a few minutes to familiarize yourself with a few important directories and files.


## Content


- /var/www/html: The actual web content, which by default only consists of the default Nginx page you saw earlier, is served out of the /var/www/html directory. This can be changed by altering Nginx configuration files.

## Server Configuration


- /etc/nginx: The Nginx configuration directory. All of the Nginx configuration files reside here.
- /etc/nginx/nginx.conf: The main Nginx configuration file. This can be modified to make changes to the Nginx global configuration.
- /etc/nginx/sites-available/: The directory where per-site server blocks can be stored. Nginx will not use the configuration files found in this directory unless they are linked to the sites-enabled directory. Typically, all server block configuration is done in this directory, and then enabled by linking to the other directory.
- /etc/nginx/sites-enabled/: The directory where enabled per-site server blocks are stored. Typically, these are created by linking to configuration files found in the sites-available directory.
- /etc/nginx/snippets: This directory contains configuration fragments that can be included elsewhere in the Nginx configuration. Potentially repeatable configuration segments are good candidates for refactoring into snippets.

## Server Logs


- /var/log/nginx/access.log: Every request to your web server is recorded in this log file unless Nginx is configured to do otherwise.
- /var/log/nginx/error.log: Any Nginx errors will be recorded in this log.

# Conclusion


Now that you have your web server installed, you have many options for the type of content to serve and the technologies you want to use to create a richer experience.


If you’d like to build out a more complete application stack, check out this article on how to configure a LEMP stack on Ubuntu 18.04.


