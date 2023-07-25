# How To Deploy Sinatra Based Ruby Web-Applications On Ubuntu 13

```Apache``` ```Ubuntu``` ```Sinatra``` ```Ruby``` ```Nginx```

## Introduction



Sinatra is a concise framework that does not like to show off. It gets the job done, without forcing anything on the developer. When it comes to taking your application, developed using this wonderful little tool, the same logic applies and you are welcomed with multiple choices. Each working pretty much the same way: getting the job done  without unnecessary complexities.


In this DigitalOcean article, following our first Ruby 2.1 & Sinatra On Ubuntu 13 tutorial, we will learn a couple of different ways to take your Sinatra application out of its hidden box and share it with the world [on Ubuntu] using different (and interesting) technologies and methods.


# Glossary



## 1. Application Deployment



## 2. Application Servers



1. Rack Middleware
2. Phusion Passenger Application Server
3. Unicorn Application Server

## 3. HTTP / WWW Servers



1. Apache HTTP Server
2. Nginx HTTP Server Running As A Front-End Reverse-Proxy

## 4. Installations



1. Apache And Passenger Combination
2. Nginx And Unicorn Combination

Note: This article’s examples build on our first Sinatra & Ruby 2.1.0 on Ubuntu 13 article by convention. If you would like to learn more about getting started with the framework or how to prepare the operating system with Ruby 2.1.0 and Sinatra, consider checking it out before continuing with this piece.


# Application Deployment



Deploying an application (regardless of it being a web site, an API or a server) usually means setting up a system from scratch (or from a snapshot taken in time), preparing it by updating everything, downloading dependencies, setting up the file structure and permissions followed by finally uploading your codebase or downloading it using a source control manager (SCM) such as Git.


Following the design incentives of Sinatra, we are going to try to keep things as simple and possible and go with tried, tested, and easy to use methods for our deployment examples here. We will be working with reputable and trusted tools to handle the job and learn about their differences.


# Application Servers



The term “application server” applies to applications (i.e. servers), which (usually) contains another application (e.g. your web-application) and following certain specifications and interfaces (i.e. a common language), allows the contained application to communicate with the outside world.


These tools are, again, usually geared towards handling the business logic (performing procedures) alone and not for other HTTP / WWW operations such as sending or receiving static file assets, dealing with multiple clients, or contending with long standing connections – although, thanks to many readily available libraries, there are such application servers available as well.


In most set-ups, one or more application servers (e.g. Passenger, Unicorn, Puma, Thin etc.) are placed behind a proper HTTP / WWW server (e.g. Nginx, Apache etc.), tasked with handling and dealing with all incoming connections first, before passing it to the next level. This allows serving of assets (e.g. javascript files, images etc.) to be extremely efficient, meanwhile leveraging the capabilities of both applications to their maximum and keeping clients on-the-line (i.e. not dropping connections) and processing requests within the application layer.


Note: To learn about different Ruby web-application servers and understand what Rack is, check out our article A Comparison of (Rack) Web Servers for Ruby Web Applications.


## Rack Middleware



Rack middleware, implementing the Rack specification, works by dividing incoming HTTP requests into different pipelined stages and handles them in pieces until it sends back a response coming from your web application (controller). It has two distinct components, a Handler and an Adapter, used for communicating with web servers and applications (frameworks) respectively.


In regards to Ruby based frameworks, web-application servers do their job by implementing the Rack specification / interface and connecting to your application through config.ru file, calling (and importing) your application as an object.


## Phusion Passenger Application Server



Passenger is a mature, feature rich product which aims to cover necessary needs and areas of application deployment whilst greatly simplifying the set-up and getting-started procedures. It eliminates the traditional middleman architecture by directly integrating with the Nginx (and Apache) reverse-proxy.


This highly popular tool can be used widely in many production scenarios. The open-source version of Passenger (which is what we will be using) has a multi-process single-threaded operation mode. Its Enterprise version can be configured to work either single-threaded or multi-threaded, depending on your needs.


To learn more about Passenger, you can visit its official website located at https://www.phusionpassenger.com/.


## Unicorn Application Server



Unicorn is a very mature web application server for Ruby/Rack based web applications. It is fully-featured; however, it denies by design trying to do everything: Unicorn’s principal is doing what needs to be done by a web application server and delegating the rest of the responsibilities (e.g. the operating system).


Unicorn’s master process spawns workers, as per your requirements, to serve the requests. This process also monitors the workers in order to prevent memory and process related staggering issues. What this means for system administrators is that it will kill a process if (for example) it takes too much time to complete a task or memory issues occur.


As mentioned above, one of the areas in which Unicorn delegates tasks is using the operating system for load balancing. This allows the requests not to pile up against busy workers spawned.


# HTTP / WWW Servers



## Apache HTTP Server



Apache is an HTTP server that does not really need any introduction at this point. It has been world’s most popular HTTP server and it is an extremely mature, feature rich and highly configurable product that runs and works extremely well. In this article, one of the front-facing servers we will work with and use is Apache and we will see how it integrates with the Phusion Passenger application server.


## Nginx HTTP Server Running As A Front-End Reverse-Proxy



Nginx is designed from ground up to act as a multi-purpose HTTP server. It is capable of serving static files (e.g. images, text files etc.) extremely well, balance connections and deal with certain exploits attempts. It acts as the first entry point of all requests, and passes them to Passenger for the web application to process and return a response.


It is a very high performant web server / (reverse)-proxy that is relatively easy to work with and easy to extend (with add-ons and plug-ins). Thanks to its architecture, Nginx is capable of handling a lot of requests (virtually unlimited), which - depending on your application or website load - could be really hard to tackle using some other, older alternatives.


Remember: “Handling” connections technically means not dropping them and being able to serve them with something. You still need your application [server] and database functioning well in order to have Nginx serve clients’ responses that are not error messages.


To learn more about Nginx, you can visit its official website located at nginx.com.


# Installations



After deciding on which server combination you would like to work with, the next step consists of actually getting them installed and ready on the droplet you want your application to run.


## Apache And Passenger Combination



Note: Before installing Apache and Passenger, make sure to disable (or remove) Nginx if you have it installed; or configure it to not to block Apache’s way (i.e. port / socket collisions).


Note: If your droplet has less than 1 GB of RAM, you will need to perform the below simple procedure to prepare a SWAP disk space to be used as a temporary data holder (i.e. RAM substitute) for Passenger. Since DigitalOcean virtual servers come with fast SSD disks, this does not really constitute an issue whilst performing the server application installation tasks.


```
# Create a 1024 MB SWAP space
# The process should complete within less than a minute

sudo dd if=/dev/zero of=/swap bs=1M count=1024
sudo mkswap /swap
sudo swapon /swap

```


We will begin getting the Apache and Passenger combination with first preparing the system with the tools that Passenger requires.


These tools consist of the following:


1. 
Curl development headers with SSL support

2. 
Apache 2

3. 
Apache 2 development headers

4. 
Apache Portable Runtime (APR) development headers

5. 
Apache Portable Runtime Utility (APU) development headers


Let’s get them one by one, following the official Passenger documentations’ suggested ways:


```
# Install cURL development headers with SSL support:
apt-get install libcurl4-openssl-dev

# Apache 2:
apt-get install apache2-mpm-worker

# Apache 2 development headers:
apt-get install apache2-threaded-dev

# And finally, Apache PRU (APU) development headers:
apt-get install libapr1-dev

```


Next, let’s download and install Passenger using RubyGems’ gem:


```
gem install passenger

# This method of installing Passenger will use
# the available and activated Ruby interpreter.
# However, the installation will be available for
# all Ruby versions' use.

```


Once we have all the necessary elements ready, we can get the glue layer (i.e. module) that will allow Apache and Passenger to couple and work together.


Begin the installation procedure by running the following command:


```
passenger-install-apache2-module

# Here's what you can expect from the installation process:

# 1. The Apache 2 module will be installed for you.
# 2. You'll learn how to configure Apache.
# 3. You'll learn how to deploy a Ruby on Rails application.

# ..

```


Press enter to continue.


Now the installer will ask you to choose which programming languages you will be working with. Scroll down with your arrow keys and use the space bar to choose what’s appropriate for your use. Since we are aiming to deploy Sinatra, our selection will cover Ruby alone.


```
Which languages are you interested in?

Use <space> to select.
If the menu doesn't display correctly, ensure that your terminal supports UTF-8.

 ‣ ⬢  Ruby
   ⬡  Python
   ⬡  Node.js
   ⬡  Meteor

```


Once you have made your choice, press enter to advance to the next step.


Now the installer will start compiling the Apache module.


Note: This bit might take a short while – usually a couple of minutes.


## Nginx And Unicorn Combination



Note: Before installing Nginx with Unicorn, make sure to disable (or remove) Apache, or configure it to not to block Nginx (i.e. port / socket collisions).


Note: If you are constrained with system resources (e.g. amount of RAM you have available), you might want to choose this combination to deploy your application.


Nginx and Unicorn together make a great combination for web-application deployments. Unicorn, as we have discussed previously, is an amazing server that works in smart ways and make user of all available system tools to handle the job – and it does it well!


We will start the installation process by getting Nginx first. The default system package manager aptitude (or apt-get) will be able to download and install Nginx for us.


Run the following to download and install Nginx using aptitude:


```
aptitude install nginx

```


Once we have Nginx ready, the next step consists of getting Unicorn web-application server, which is made easy by RubyGems package manager.


Run the following to download and install Unicorn using gem:


```
gem install unicorn 

```


# Configuration



In this section, we are going to see how to configure both server combinations to get our applications online. You should remember that despite the marketing efforts, you will be able to deploy any number of web-applications as long as you configure them properly and have enough system resources to support them.


Note: Remember that you should choose one-or-the-other from above web-application / HTTP server combinations before continuing with (and applying) the below settings.


## Apache & Passenger Combination



After finishing installing Apache, Passenger and Passenger’s Apache module, you will be shown a message titled Almost there!



Please edit your Apache configuration file and add these lines:
LoadModule passenger_module /usr/local/rvm/gems/ruby-2.1.0/gems/passenger-4.0.37/buildout/apache2/mod_passenger.so
<IfModule mod_passenger.c>
PassengerRoot /usr/local/rvm/gems/ruby-2.1.0/gems/passenger-4.0.37
PassengerDefaultRuby /usr/local/rvm/gems/ruby-2.1.0/wrappers/ruby
</IfModule>
After you restart Apache, you are ready to deploy any number of web
applications on Apache, with a minimum amount of configuration!

So, as suggested and advised, let’s add the configuration block to our Apache configuration file.


Run the following to edit the Apache configuration file using nano text editor:


```
nano /etc/apache2/apache2.conf

```


Add the below block of text to where you see fit, without affecting other configurations:


```
LoadModule passenger_module /usr/local/rvm/gems/ruby-2.1.0/gems/passenger-4.0.37/buildout/apache2/mod_passenger.so
<IfModule mod_passenger.c>
    PassengerRoot /usr/local/rvm/gems/ruby-2.1.0/gems/passenger-4.0.37
    PassengerDefaultRuby /usr/local/rvm/gems/ruby-2.1.0/wrappers/ruby
</IfModule>

```


Save and exit by pressing CTRL+X and confirming with Y.


Note: The above configuration must be kept as is. Its parameters are defined in a way to use the relevant Ruby version (2.1.0 installed during the first Sinatra tutorial) at the relevant location (/usr/local/rvm/gems/ruby-2.1.0/..).


Next, we need to define an application as suggested by the installer:



Deploying a web application: an example
Suppose you have a web application in /somewhere. Add a virtual host to your
Apache configuration file and set its DocumentRoot to /somewhere/public:
<VirtualHost *:80>
ServerName www.yourhost.com
# !!! Be sure to point DocumentRoot to ‘public’!
DocumentRoot /somewhere/public
<Directory /somewhere/public>
# This relaxes Apache security settings.
AllowOverride all
# MultiViews must be turned off.
Options -MultiViews
</Directory>
</VirtualHost>

You can deploy yours anywhere you like; however, what can be considered a good example is using either:


- 
A sub-directory of a user, set for deployments, or;

- 
Using the general /var/www location, as we have done in the past.


Note: For this next step of configurations, we will use our sample application from the previous Sinatra/Ruby 2.1.0 tutorial.


To let Apache know about our application and get Passenger to run it, we need to define a virtual host. For this purpose, we will create a file inside the /etc/apache2/sites-available directory.


Let’s create an empty configuration file called my_app using nano:


Note: You can name this file as you see fit to match your application settings.


```
nano /etc/apache2/sites-available/my_app.conf

```


Place the contents below, modifying them to suit your application deployment directory:


```
<VirtualHost *:80>
    # ServerName [!! Your domain OR droplet's IP]
    ServerName 162.243.74.190 
            
    # !!! Be sure to point DocumentRoot to 'public'!
    # DocumentRoot [root to your app./public]
    DocumentRoot /var/www/my_app/public
    
    # Directory [root to your app./public]
    <Directory /var/www/my_app/public>
        # This relaxes Apache security settings.
        AllowOverride all
        # MultiViews must be turned off.
        Options -MultiViews
    </Directory>
    
</VirtualHost>

```


Save and exit by pressing CTRL+X and confirming with Y.


Now we can add our new site configuration to Apache:


```
# Usage sudo a2ensite [configuration name without .conf extension]
sudo a2ensite my_app

```


And reload:


```
service apache2 reload
service apache2 restart

```


Your web-application should now be alive and on-line. Visit your droplet’s IP address or the domain you have redirected and defined under the ServerName configuration:


```
http://162.243.74.190/

# Hello world!

```


Note: If you have opted for working with a domain name, you should add it to your /etc/hosts file as well with the following command:


```
nano /etc/hosts

```


And append your domain to the list:


```
# 127.0.0.1 [domain.tld]
# 127.0.0.1 [www.domain.tld]

127.0.0.1 example.com
127.0.0.1 www.example.com

```


## Nginx And Unicorn Combination



Unicorn can be configured a number of ways. For this tutorial, focusing on the key elements, we will create a file from scratch which is going to be used by Unicorn when starting the application server daemon.


Open up a blank unicorn.rb document, which will be saved inside the application directory (/var/www/my_app) directory:


```
nano unicorn.rb

```


Place the below block of code, modifying it as necessary:


```
# Set the working application directory
# working_directory "/path/to/your/app"
working_directory "/var/www/my_app"

# Unicorn PID file location
# pid "/path/to/pids/unicorn.pid"
pid "/var/www/my_app/pids/unicorn.pid"

# Path to logs
# stderr_path "/path/to/logs/unicorn.log"
# stdout_path "/path/to/logs/unicorn.log"
stderr_path "/var/www/my_app/logs/unicorn.log"
stdout_path "/var/www/my_app/logs/unicorn.log"

# Unicorn socket
# listen "/tmp/unicorn.[app name].sock"
listen "/tmp/unicorn.myapp.sock"

# Number of processes
# worker_processes 4
worker_processes 2

# Time-out
timeout 30

```


Save and exit by pressing CTRL+X and confirming with Y.


Note: To simply test your application with Unicorn, you can run unicorn inside the application directory.


Next, we need to tell Nginx how to talk to Unicorn. For this purpose, it is sufficient at this level to edit the default configuration file: default.conf and leave nginx.conf as provided – which is already set to include the default configurations.


```
# Remove the default configuration file
rm -v /etc/nginx/sites-available/default

# Create a new, blank configuration
nano /etc/nginx/conf.d/default.conf

```


Replace the files contents with the ones from below, again amending the necessary bits to suit your needs:


```
upstream app {
    # Path to Unicorn SOCK file, as defined previously
    server unix:/tmp/unicorn.myapp.sock fail_timeout=0;
}

server {


    listen 80;
    
    # Set the server name, similar to Apache's settings
    server_name localhost;

    # Application root, as defined previously
    root /var/www/my_app/public;
    
    try_files $uri/index.html $uri @app;
    
    location @app {
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_redirect off;
        proxy_pass http://app;
    }
    
    error_page 500 502 503 504 /500.html;
    client_max_body_size 4G;
    keepalive_timeout 10;

}  

```


Save and exit by pressing CTRL+X and confirming with Y.


Note: To learn more about Nginx, you can refer to How to Configure Nginx Web Server on a VPS.


Let’s start the Unicorn and run it as a daemon using the configuration file:


```
# Make sure that you are inside the application directory
# i.e. /my_app
unicorn -c unicorn.rb -D

```


Next, we are ready to reload and restart Nginx:


```
service nginx restart

```


And that’s it! You can now check out your deployment by going to your droplet’s IP address (or the domain name associated to it).


```
http://162.243.74.190/

# Hello world!

```


# Further Reading



## Firewall:



Setting up a firewall using IP Tables


## Securing SSH:



How To Protect SSH with fail2ban on Ubuntu
How To Protect SSH with fail2ban on CentOS 6


## Creating Alerts:



How To Send E-Mail Alerts on a CentOS VPS for System Monitoring


## Monitor and Watch Server Access Logs Daily:



How To Install and Use Logwatch Log Analyser and Reporter


## Optimising Unicorn Workers:



How to Optimize Unicorn Workers in a Ruby on Rails App


<div class=“author”>Submitted by: <a
href=“https://twitter.com/ostezer”>O.S. Tezer</a></div>


