# How To Deploy Kohana PHP Applications on a Debian 7   Ubuntu 13 VPS with Nginx and PHP-FPM

```Ubuntu``` ```PHP Frameworks``` ```PHP``` ```Debian```

## Introduction



Kohana comes as a self-contained package, with each copy forming a new base for a new web application, making things quite easy for deployment.


In this DigitalOcean article, following our previous ones on installing and getting started with Kohana, we’ll see how to prepare a VPS to deploy a Kohana based PHP web application - using Debian 7 / Ubuntu 13 as our host operating system.


Note: This is the third article in our Kohana series, focused on deploying applications built using the framework. To see the first part and learn about installing it, check out Getting Started with Kohana. To see about understanding the framework’s modules to build a web application, check out Building Web Applications with HMVC PHP5 Framework Kohana.


# Glossary



## 1. PHP Based Web-Application Deployment



## 2. Web Servers



```
1. Nginx HTTP Server and Reverse-Proxy
2. Lighttpd
3. Apache

```


## 3. PHP Processors



```
1. mod_php
2. FastCGI
3. PHP-FPM

```


## 4. Kohana in Brief



## 5. About Our Deployment



## 6. Preparing the System For Kohana Application Deployment



```
1. Updating the System
2. Installing Nginx
3. Installing MySQL 5
4. Installing PHP (PHP-FPM)

```


## 7. Configuring the System



```
1. Configuring PHP
2. Configuring Nginx

```


## 8. Deploying A Kohana Web-Application



```
1. Uploading the Code Base To The Server
2. Bootstrapping the Deployment (Installation)

```


# PHP Based Web-Application Deployment



There are a few different methods to deploy a PHP based web-application, with many more sub-configuration options being available.


The major factor and differentiator is the choice of web server. Some of the most popular ones are:


- 
Nginx HTTP Server and Reverse-Proxy

- 
Lighttpd (or lighty)

- 
Apache


There are also several different PHP processors that can be used to have the above web servers process and serve PHP files:


- 
mod_php

- 
FastCGI

- 
PHP-FPM


# Web Servers



## Nginx HTTP Server and Reverse-Proxy



Nginx is a very high performant web server / (reverse)-proxy. It has reached its popularity due to being light weight, relatively easy to work with, and easy to extend (with add-ons / plug-ins). Thanks to its architecture, it is capable of handling a lot of requests (virtually unlimited), which - depending on your application or website load - could be really hard to tackle using some other, older alternatives. It can be considered the tool to choose for serving static files such as images, scripts or style-sheets.


## Lighttpd



Lighttpd is a very speedy web server that is licensed under the permissive BSD License. It works and operates in a way that is closer to Nginx than Apache. The way it handles requests is very low on memory and CPU foot print.


## Apache



Apache is a long tried and tested, extremely powerful web server. Although it might not have its old popularity, Apache still offers many things that its new competitors do not. It also comes with a lot of modules that can be used to expand the default functionality and have Apache suit your specific deployment needs.


# PHP Processors



Web servers (mostly) are not set out to process PHP scripts - nor others based on different programming languages. To do this, they depend on external libraries, each operating in a similar-looking but in actuality very different ways. Different web servers offer different level of integrations with each - and it is highly recommended that as a person responsible of deploying an application, you perform a thorough research to have a better idea of what they do and how they do it.


## mod_php



For a long time, mod_php remained the most popular Apache module and the way-to-go choice for deploying PHP web applications. It works by embedding the PHP processor inside Apache to run PHP scripts.


<b>Advantages:</b>


- 
Extremely stable and well tested.

- 
No external dependencies are involved for processing.

- 
Extremely performant.

- 
Loads php.ini once.

- 
Supports .htaccess configurations.


## FastCGI



FastCGI works by connecting the external PHP processor installation with the web server through sockets. It is a more advanced way of doing the same thing with ye old CGI. FastCGI can be considered more secure than working with mod_php as it separates the processor from the web server process (and isolating each from possible harmful exploits).


<b>Advantages:</b>


- 
Eliminates the need to involve PHP processor for static content.

- 
Eliminates the memory overhead of using a PHP processor per Apache process.

- 
Brings in an additional security layer by dividing the server and the processor.


## PHP-FPM



PHP-FPM consists of an upgrade to the FastCGI way of using PHP. It brings certain new features and a whole new way of handling requests - for the benefit of (especially) larger web sites.


<b>Advantages:</b>


- 
Adaptive process spawning.

- 
Graceful processor management.

- 
Smaller memory footprint compared to FastCGI.

- 
More configurable than FastCGI.


# Kohana in Brief



Kohana is an HMVC (Hierarchical Model View Controller) framework that offers almost all the necessary tools out-of-the-box in order to build a modern web application that can be developed rapidly, deployed and maintained easily.


As a “light” framework, Kohana is made of files that are scattered across carefully structured directories inside a single (application) one - which means each kohana package can be considered as a web application (minus the possible external dependencies).


The way Kohana is built, by design, makes this framework extremely friendly for deployment.


# About Our Deployment



In this article, we’ll be using Nginx coupled with PHP-FPM for deployment because of the features offered - and their popularity. Our database of choice here is MySQL; however, you should opt for and use another (e.g. MariaDB). Do not forget that you will need to migrate your database - see the end section for details.


# Preparing the System For Kohana Application Deployment



We will be working with a newly instantiated Ubuntu 13 VPS. For various reasons, you are highly advised to do the same and try everything out before performing them on an already active and working server setup.


## Updating the System



We will begin with updating our virtual server’s default application toolset to their latest available.


Run the following to perform this task:


```
aptitude    update
aptitude -y upgrade

```


## Installing Nginx



Execute the following to install Nginx using the package manager aptitude:


```
aptitude -y install nginx

```


Run the below command to start the server:


```
service nginx start

```


You can visit http://[your droplet's IP adde.]/ to test the Nginx installation. You should see the default welcome screen.


## Installing MySQL 5



We will again be using the aptitude package manager to install MySQL and the other applications. Run the below command to download and install MySQL.


Remember: During the installation process you will be prompted with a couple of questions regarding the root password.


```
aptitude -y install mysql-server mysql-client

```


Bootstrap everything using mysql_secure_installation:


```
# Run the following to start the process
mysql_secure_installation

# Enter the root password you have chosen during installation    
# Continue with answering the questions, confirming all.
# Ex.:
# Remove anonymous users? [Y/n] Y
# Disallow root login remotely? [Y/n] Y
# ..
# .     

```


Run the below command to restart and check the status of your MySQL installation:


```
service mysql restart
# mysql stop/waiting
# mysql start/running, process 25012

service mysql status
# mysql start/running, process 25012

```


## Installing PHP (PHP-FPM)



To process PHP pages with Nginx, we are going to use PHP-FPM. Run the following to download and install the application package:


```
aptitude -y install php5 php5-fpm php5-mysql

```


This will also install PHP5 commons.


# Configuring the System



After installing the necessary tools, it is time to get our server ready for deploying PHP web application by making the final configuration amendments.


## Configuring PHP



We will be working with the default PHP configuration file php.ini and edit it using the text editor nano to make a few small changes.


Run the following to open the file using nano:


```
nano /etc/php5/fpm/php.ini

```


Scroll down the document and find the line ;cgi.fix_pathinfo=1. Modify it similar to the following to have application paths processed securely:


```
# Replace ;cgi.fix_pathinfo=1 with:
cgi.fix_pathinfo=0

```


Save and exit by pressing CTRL+X and confirm with Y.


## Configuring Nginx



Since in this article we are looking at deploying a single web application, our modification and configuration choices are made accordingly. In order to host more than one application using Nginx, you will need to make user of server blocks. To learn more about them, check out Setting up Nginx Virtual Hosts (Server Blocks).


Let’s edit the default Nginx website configuration. Run the following to open up the file using nano:


```
nano /etc/nginx/sites-available/default

```


Copy and paste the below content:


```
server {

    listen 80 default_server;
    listen [::]:80 default_server ipv6only=on;
 
    # Set the default application path root
    # We are using my_app to host the application
    root /var/www/my_app;
    index index.php;

    # Replace the server_name with your own
    server_name localhost;

    location /
    {
        try_files $uri /index.php?$args;

        if (!-e $request_filename)
        {
            rewrite ^/(.*)$ /index.php/$1 last;
        }
    }

    location ~ /\.
    {
        deny  all;
    }

    location ~ \.php$
    {
        try_files $uri = 404;
        fastcgi_pass unix:/var/run/php5-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

}

```


Save and exit by pressing CTRL+X and confirm with Y.


Create and enter the application deployment directory /var/www:


```
mkdir /var/www
cd /var/www

```


Test the configurations and restart Nginx for the chances to take effect:


```
# Test the configurations
nginx -t

# Restart the server
service nginx restart

```


# Deploying A Kohana Web-Application



Following the configurations made, we are ready to deploy our application. The process will consist of two main steps:


1. 
Uploading the code base to the server

2. 
Modifying bootstrap.php to ensure it is set correctly.


## Uploading the Code Base To The Server



You can use SFTP or a graphical tool, such as FileZilla, to transfer and manage remote files securely.


To learn about working with SFTP, check out the article: How To Use SFTP.


To learn about FileZilla, check out the article on the subject: How To Use FileZilla.


Note: If you choose to use FileZilla, make sure to set /var/www as the default directory (this should save you some time).


In this example, we will work with the default (sample) Kohana application.


Download and prepare Kohana:


```
# Remember: The following commands assume that your current
#           working directory is set as /var/www/

wget https://github.com/kohana/kohana/releases/download/v3.3.1/kohana-v3.3.1.zip

# You might need to install *unzip* before extracting the files    
aptitude install -y unzip 

# Unzip and extract the files
unzip kohana-v3.3.1.zip -d my_app

# Remove the zip package
rm -v kohana-v3.3.1.zip

# Enter the application directory
cd my_app

```


## Bootstrapping the Deployment (Installation)



Run the following to start editing the configuration file using nano:


```
nano application/bootstrap.php

```


Edit your timezone:


```
# Find date_default_timezone_set and set your timezone
date_default_timezone_set('Europe/London');

```


Set your locale:


```
# Find setlocale and set your locale
setlocale(LC_ALL, 'en_UK.utf-8');

```


Find end edit the base_url to match the installation


```
# Find base_url and set the base application directory
# Relative to the base Apache directory (i.e. /var/www/my_app)

Kohana::init(array(
    'base_url' => '/',
));

```


Make sure to enable all necessary modules:


```
# Find Kohana::modules and uncomment them

Kohana::modules(array(
    'auth'       => MODPATH.'auth',       // Basic authentication
    'cache'      => MODPATH.'cache',      // Caching with multiple backends
    'codebench'  => MODPATH.'codebench',  // Benchmarking tool
    'database'   => MODPATH.'database',   // Database access
    'image'      => MODPATH.'image',      // Image manipulation
    'orm'        => MODPATH.'orm',        // Object Relationship Mapping
    'oauth'      => MODPATH.'oauth',      // OAuth authentication
    'pagination' => MODPATH.'pagination', // Paging of results
    'unittest'   => MODPATH.'unittest',   // Unit testing
    'userguide'  => MODPATH.'userguide',  // User guide and API documentation
));

```


Save and exit by pressing CTRL+X and confirm with Y.


Set cache and log directories writable:


```
sudo chmod -R a+rwx application/cache
sudo chmod -R a+rwx application/logs

```


And that’s it! Now you should have your Kohana web application ready to run.


Remember: You should not forget about migrating your application data (e.g. MySQL server database) to your droplet from your development machine or another server. To learn about doing this, check out How To Migrate a MySQL Database Between Two Servers and related comments for further information.


<div class=“author”>Submitted by: <a
href=“https://twitter.com/ostezer”>O.S. Tezer</a></div>


