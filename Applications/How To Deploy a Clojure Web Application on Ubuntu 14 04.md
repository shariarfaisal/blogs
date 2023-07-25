# How To Deploy a Clojure Web Application on Ubuntu 14 04

```Ubuntu``` ```Applications``` ```Nginx```

## Introduction


There continues to be an uptick in interest around functional programming and, more specifically, programming for the web in Clojure. Many tutorials on how to build basic applications often overlook deployment details. This article will show you how to deploy a Clojure web application to a Ubuntu 14.04 Droplet.


Specifically, we will create a sample Clojure application and package it for production use, and set up the Clojure app environment on the server using Supervisor to run the app and Nginx to serve requests to it.


# Prerequisites


Before you begin this guide you’ll need the following:


- One Ubuntu 14.04 Droplet
- A non-root user account with sudo access on your server, which you can set up by following these instructions

# Step 1 — Creating and Packaging a Sample Clojure App


The very first step is to use git to grab the example Clojure project to deploy.


First, update your packages and install git on the server.


```
sudo apt-get update
sudo apt-get install git


```


Next, clone the sample project repository.


```
git clone https://github.com/do-community/do-clojure-web.git


```


This repository is the end result of following the Clojure Basic Web Development tutorial. If you like, instead of cloning this repository, you can follow that tutorial yourself.


Clojure leverages the JVM to run its code, so you need to compile your project to run it. Leiningen, a dependency management and build automation tool for Clojure apps, makes this easy. There are a couple steps to get Leiningen set up.


First, install Java.


```
sudo apt-get install openjdk-7-jre-headless


```


Next download the Leiningen install script. There is an Ubuntu package for Leiningen, but it’s very outdated.


```
sudo curl https://raw.githubusercontent.com/technomancy/leiningen/stable/bin/lein -o /usr/local/bin/lein

```


Set the permissions so any user can use the lein utility Leiningen provides.


```
sudo chmod a+x /usr/local/bin/lein


```


Now you can compile your project to run on the server with lein.


```
cd ~/do-clojure-web
lein uberjar


```


# Step 2 — Setting Up the Clojure Application Environment


We need three main pieces for this application to work correctly: Java, Supervisor, and Nginx. We installed Java in the last step, so next, we’ll install Supervisor and Nginx.


```
sudo apt-get install nginx supervisor


```


You’ll need a place to keep your Clojure web application and its log files, too.  Create that directory structure next.


```
sudo mkdir -p /var/www/do-clojure-web/app/db /var/www/logs


```


Now you can move your Clojure application file and database file into the directories you created.


```
sudo cp ~/do-clojure-web/target/do-clojure-web-0.1.0-standalone.jar /var/www/do-clojure-web/app/
sudo cp ~/do-clojure-web/db/do-clojure-web.h2.db /var/www/do-clojure-web/app/db/


```


The application will run as the user www-data on the system so it can write to our built-in database. Set the owner of the application path to www-data.


```
sudo chown -R www-data /var/www/do-clojure-web/


```


Change to the Clojure application directory.


```
cd /var/www/do-clojure-web/app/


```


In a production environment, the version number of the application will change with each update. You don’t want to have to update your system configuration each time that happens. To prevent that, create a symlink to the current running version of the application.  You’ll reference the symlink in the steps to come.


```
sudo ln -s do-clojure-web-0.1.0-standalone.jar do-clojure-web.jar


```


The application is currently configured to be accessible only via localhost, but you can still make sure that it starts without error. Do that before moving on.


```
sudo java -jar do-clojure-web.jar


```


If everything is working correctly you should get output similar to this:


Output
```
. . .
2015-06-12 04:30:17.882:INFO:oejs.Server:jetty-7.x.y-SNAPSHOT
2015-06-12 04:30:17.995:INFO:oejs.AbstractConnector:Started SelectChannelConnector@127.0.0.1:5000

```


Go ahead and stop the application for now by pressing the key combination CTRL+C.


# Step 3 — Configuring Supervisor to Run the Clojure App


There are a few options for managing your application as a service. The option you’ll use here is called Supervisor; it’s easier to manage and more versatile than a simple script. However, for a service that really needs to scale, check out the uWSGI documentation on running a Clojure application.


Create and edit the /etc/supervisor/conf.d/do-clojure-web.conf file.


```
sudo nano /etc/supervisor/conf.d/do-clojure-web.conf


```


Add this configuration to the file and save it.


/etc/supervisor/conf.d/do-clojure-web.conf
```
[program:do-clojure-web]
command=/usr/bin/java -jar do-clojure-web.jar
directory=/var/www/do-clojure-web/app
user=www-data
autostart=true
autorestart=true
startretries=3
redirect_stderr=true
stdout_logfile=/var/www/logs/do-clojure-web.app.log

```


This configuration is pretty straightforward. The Supervisor daemon (service) will run our application from within the /var/www/do-clojure-web/app directory. It will also make sure to log to /var/www/logs/do-clojure-web.app.log, and will try to restart the application, should it crash.


# Step 4 — Configuring Nginx as a Proxy Server


Because the Clojure web application only accepts connections from localhost on port 5000, we need to put a web server like Nginx in front of it to provide external access.  This will also be very convenient for serving static assets as you expand your application.


Edit the /etc/nginx/sites-available/default file.


```
sudo nano /etc/nginx/sites-available/default


```


Add the section highlighted in red to the file. This defines our backend for easy reference in the next configuration section.


/etc/nginx/sites-available/default
```
. . .
# Please see /usr/share/doc/nginx-doc/examples/ for more detailed examples.
##
upstream http_backend {
    server 127.0.0.1:5000;
    keepalive 32;
}
server {
    listen 80 default_server;
. . .

```


Now find the block that begins with location /. Comment out all the lines in it by adding a # at the beginning of each line.


/etc/nginx/sites-available/default
```
. . .
        # Make site accessible from http://localhost/
        server_name localhost;


        # location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                # try_files $uri $uri/ =404;
                # Uncomment to enable naxsi on this location
                # include /etc/nginx/naxsi.rules
        # }
        
        # Only for nginx-naxsi used with nginx-naxsi-ui : process denied requests
. . .

```


Then, just below this, add in the following section, which will tell Nginx to listen like a normal web server on port 80 and proxy your requests to the Clojure application.


/etc/nginx/sites-available/default
```
. . .

        # Only for nginx-naxsi used with nginx-naxsi-ui : process denied requests

    location / {
        proxy_pass http://http_backend;

        proxy_http_version 1.1;
        proxy_set_header Connection "";

        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;

        access_log /var/www/logs/do-clojure-web.access.log;
        error_log /var/www/logs/do-clojure-web.error.log;
    }

        #location /RequestDenied {
        #       proxy_pass http://127.0.0.1:8080;    
        #}


```


# Step 5 — Starting Services and Testing Access


It’s time to fire up all the pieces and make sure things are working properly. Go ahead and start up the Supervisor daemon so your Clojure application gets started.


```
sudo service supervisor start 


```


Wait about 30 seconds for it to start, and then start up the Nginx web server front end proxy.


```
sudo service nginx start


```


Visit http://your_server_ip in your browser.  You should see the example Clojure application site load.


If you’re just getting a default Nginx page, try restarting Supervisor with sudo service supervisor restart, waiting 30 seconds, and restarting Nginx with sudo service nginx restart.


Once the site has loaded, click on the Add a Location link at the top of the screen and try adding a few numeric coordinates to ensure your database access permissions are correct. For example, you can add 1 for x value and 2 for y value. This should bring you to a page that says:


add-location output
```
Added [1, 2] (id: 1) to the db. See for yourself.

```


If you click on the View All Locations link at the top of the screen, you should see a table with your new entry.


# Conclusion


You’ve just deployed a Clojure application using Leiningen, Supervisor, and Nginx! There is a lot to learn around the topic of deploying even the simplest websites and applications. The next step is to deploy your custom application, instead of the demo application used in this tutorial.


