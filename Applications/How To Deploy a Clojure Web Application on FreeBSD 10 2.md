# How To Deploy a Clojure Web Application on FreeBSD 10 2

```Applications``` ```Nginx``` ```FreeBSD```

## Introduction


There continues to be an uptick in interest around functional programming and, more specifically, programming for the web in Clojure. Many tutorials on how to build basic applications often overlook deployment details. This article will show you how to deploy a Clojure web application to a FreeBSD 10.2 server.


Specifically, we will create a sample Clojure application and package it for production use, and set up the Clojure app environment on the server using Supervisor to run the app and Nginx to serve requests to it.



Note: As of July 1, 2022, DigitalOcean no longer supports FreeBSD Droplets through the Control Panel or API. However, you can still spin up FreeBSD Droplets using a custom image. Learn how to import a custom image to DigitalOcean by following our product documentation.

# Prerequisites


Before you begin this guide, you’ll need the following:


- One FreeBSD 10.2 server, which you can optionally customize with these instructions

# Step 1 — Creating and Packaging a Sample Clojure App


The very first step is to use git to grab the example Clojure project to deploy.


First, update your packages and install git on the server.


```
sudo pkg update
sudo pkg install git


```


Next, clone the sample project repository.


```
git clone https://github.com/do-community/do-clojure-web.git


```


This repository is the end result of following the Clojure Basic Web Development tutorial. If you like, instead of cloning this repository, you can follow that tutorial yourself.


Clojure leverages the JVM to run its code, so you need to compile your project to run it. Leiningen, a dependency management and build automation tool for Clojure apps, makes this easy.


Let’s install Leiningen now.


```
sudo pkg install leiningen


```


You’ll notice some output declaring that Java requires a couple special file system mount points. We’ll take care of this in the next step.


Now you can compile your project to run on the server with lein.


```
cd ~/do-clojure-web
lein uberjar


```


# Step 2 — Setting Up the Clojure Application Environment


We need three main pieces for this application to work correctly: Java, Supervisor, and Nginx. We installed Java as part of installing Leiningen in the last step, so next, we’ll install Supervisor and Nginx.


```
sudo pkg install nginx py27-supervisor


```


Java requires a couple special file system mount points, as mentioned in step 1. Run these two commands to make sure they are mounted.


```
sudo mount -t fdescfs fdesc /dev/fd
sudo mount -t procfs proc /proc


```


Rather than running those commands each time the system boots, we’ll make it happen automatically.  Edit the /etc/fstab file using ee or your favorite text editor.


```
sudo ee /etc/fstab


```


Make sure the end of your /etc/fstab file has the following two entries.


/etc/fstab
```
fdesc   /dev/fd     fdescfs     rw  0   0
proc    /proc       procfs      rw  0   0

```


You’ll need a place to keep your Clojure web application and its log files, too.  Create that directory structure next.


```
sudo mkdir -p /www/data/do-clojure-web/app/db/ /www/logs


```


Now you can move your Clojure application file and database file into the directories you created.


```
sudo cp ~/do-clojure-web/target/do-clojure-web-0.1.0-standalone.jar /www/data/do-clojure-web/app/
sudo cp ~/do-clojure-web/db/do-clojure-web.h2.db /www/data/do-clojure-web/app/db/


```


The application will run as the user www on the system so it can write to our built-in database.  Set the owner of the application path to www.


```
sudo chown -R www /www/data/do-clojure-web/


```


Change to the Clojure application directory.


```
cd /www/data/do-clojure-web/app/


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


There are a few options for managing your application as a service.  For a service that really needs to scale I recommend looking at the uWSGI documentation on running a Clojure application.


Create and edit the /usr/local/etc/supervisord.conf file.


```
sudo ee /usr/local/etc/supervisord.conf


```


Add this configuration to the very bottom of the file and save it.


/usr/local/etc/supervisord.conf
```
[program:do-clojure-web]
command=/usr/local/bin/java -jar do-clojure-web.jar
directory=/www/data/do-clojure-web/app
user=www
autostart=true
autorestart=true
startretries=3
redirect_stderr=true
stdout_logfile=/www/logs/do-clojure-web.app.log

```


This configuration is pretty straight forward. The Supervisor daemon (service) will run our application from within the /www/data/do-clojure-web/app/ directory. It will also make sure to log to /www/logs/do-clojure-web.app.log, and will try to restart the application, should it crash.


# Step 4 — Configuring Nginx as a Proxy Server


Because the Clojure web application only accepts connections from localhost on port 5000, we need to put a web server like Nginx in front of it to provide external access.  This will also be very convenient for serving static assets as you expand your application.


Edit the /usr/local/etc/nginx/nginx.conf file.


```
sudo ee /usr/local/etc/nginx/nginx.conf


```


First, add the upstream block, hilighted in red below, above the server block already in the file.


/usr/local/etc/nginx/nginx.conf
```
. . .
    #gzip  on;
    
    upstream http_backend {
        server 127.0.0.1:5000;
        keepalive 32;
    }

    server {
            listen       80;
            server_name  localhost;
. . .

```


Now find the block that begins with location / (inside the server block). Comment out all the lines in it by adding a # at the beginning of each line and add in the new location / section highlighted in red. This tells Nginx to listen like a normal web server on port 80 and proxy your requests to the Clojure application.


/usr/local/etc/nginx/nginx.conf
```
. . .
    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        #location / {
        #    root   /usr/local/www/nginx;
        #    index  index.html index.htm;
        #}

        location / {
            proxy_pass http://http_backend;

            proxy_http_version 1.1;
            proxy_set_header Connection "";

            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Host $http_host;

            access_log /www/logs/do-clojure-web.access.log;
            error_log /www/logs/do-clojure-web.error.log;
        }

        #error_page  404              /404.html;
. . .

```


Save and exit the file.


# Step 5 — Starting Services and Testing Access


It’s time to fire up all the pieces and make sure things are working properly.  The first step is to make sure your services are configured to start up on boot.


Edit the /etc/rc.conf file.


```
sudo ee /etc/rc.conf


```


Add these two lines to the end of the /etc/rc.conf file.


/usr/local/etc/rc.conf
```
nginx_enable="YES"
supervisord_enable="YES"

```


Save and exit the file.


Go ahead and start up the Supervisor daemon so your Clojure application gets started.


```
sudo service supervisord start 


```


Wait about 30 seconds for it to start, and then start up the Nginx web server front end proxy.


```
sudo service nginx start


```


Visit http://your_server_ip in your browser.  You should see the example Clojure application site load.


If you’re just getting a default Nginx page, try restarting Supervisor with sudo service supervisord restart, waiting 30 seconds, and restarting Nginx with sudo service nginx restart.


Once the site has loaded, click on the Add a Location link at the top of the screen and try adding a few numeric coordinates to ensure your database access permissions are correct. For example, you can add 1 for x value and 2 for y value. This should bring you to a page that says:


add-location output
```
Added [1, 2] (id: 1) to the db. See for yourself.

```


If you click on the View All Locations link at the top of the screen, you should see a table with your new entry.


# Conclusion


You’ve just deployed a Clojure application using Leiningen, Supervisor, and Nginx! There is a lot to learn around the topic of deploying even the simplest websites and applications. The next step is to deploy your custom application, instead of the demo application used in this tutorial.


