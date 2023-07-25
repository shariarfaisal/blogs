# How To Install Laravel with Nginx on an Ubuntu 12 04 LTS VPS

```Ubuntu``` ```PHP Frameworks``` ```Nginx``` ```PHP```


Status: Deprecated
This article covers a version of Ubuntu that is no longer supported.  If you are currently operate a server running Ubuntu 12.04, we highly recommend upgrading or migrating to a supported version of Ubuntu:

Upgrade to Ubuntu 14.04.
Upgrade from Ubuntu 14.04 to Ubuntu 16.04
Migrate the server data to a supported version

Reason:
Ubuntu 12.04 reached end of life (EOL) on April 28, 2017 and no longer receives security patches or updates.  This guide is no longer maintained.
See Instead:
This guide might still be useful as a reference, but may not work on other Ubuntu releases.  If available, we strongly recommend using a guide written for the version of Ubuntu you are using.   You can use the search functionality at the top of the page to find a more recent version.

# Introduction



Laravel is a framework for websites in the PHP programming language. It allows developers to rapidly develop a website by abstracting common tasks used in most web projects, such as authentication, sessions, and caching. Laravel 4, the newest version of Laravel is based on an older framework called Symfony, but with a more expressive syntax. It is installed using Composer, a Dependency Manager, allowing developers to integrate even more open source PHP projects in a web project. If you want to want to read a quick introduction to Laravel, read the introduction. If you want to learn more about Composer, visit the website.


## Preparation


Let’s start off by updating the packages installed on your VPS. This makes sure no issues will arise on incompatible versions of the software. Also, make sure you run everything in this tutorial as root, and if you don’t, make sure you add sudo before every command!


```
apt-get update && apt-get upgrade

```


Hit Enter when you’re asked to confirm.


# Installation



Now we need to install the actual packages required to install Laravel. This will basically be Nginx and PHP. Because Composer is run from the command line, we do need php5-cli, and because we want to manage the connection between Nginx and PHP using the FastCGI Process Manager, we will need php5-fpm as well. Besides, Laravel requires php5-mcrypt and Composer requires git.


```
apt-get install nginx php5-fpm php5-cli php5-mcrypt git

```


This should take a while to install, but you are now ready to configure Nginx and PHP.


## Configuring Nginx


We will configure Nginx like Laravel is the only website you will run on it, basically accepting every HTTP request, no matter what the Host header contains. If you want more than one website on your VPS, please refer to this tutorial.


Make a dedicated folder for your Laravel website:


```
mkdir /var/www
mkdir /var/www/laravel

```


Open up the default virtual host file.


```
nano /etc/nginx/sites-available/default

```


The configuration should look like below:


```
server {
        listen   80 default_server;
		
        root /var/www/laravel/public/;
        index index.php index.html index.htm;

        location / {
             try_files $uri $uri/ /index.php$is_args$args;
        }

        # pass the PHP scripts to FastCGI server listening on /var/run/php5-fpm.sock
        location ~ \.php$ {
                try_files $uri /index.php =404;
                fastcgi_pass unix:/var/run/php5-fpm.sock;
                fastcgi_index index.php;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                include fastcgi_params;
        }
}

```


Now save and exit!


## Configuring PHP


We need to make a small change in the PHP configuration. Open the php.ini file:


```
nano /etc/php5/fpm/php.ini

```


Find the line, cgi.fix_pathinfo=1, and change the 1 to 0.


```
cgi.fix_pathinfo=0

```


If this number is kept as 1, the PHP interpreter will do its best to process the file that is as near to the requested file as possible. This is a possible security risk. If this number is set to 0, conversely, the interpreter will only process the exact file path — a much safer alternative. Now save it and exit nano.


We need to make another small change in the php5-fpm configuration. Open up www.conf:


```
nano /etc/php5/fpm/pool.d/www.conf

```


Find the line, listen = 127.0.0.1:9000, and change the 127.0.0.1:9000 to /var/run/php5-fpm.sock.


```
listen = /var/run/php5-fpm.sock

```


Again: save and exit!


## (Re)Starting PHP and Nginx


Now make sure that both services are restarted.


```
service php5-fpm restart
service nginx restart

```


## Installing Composer


It is now time to install Composer, this process is quite straightforward. Let’s start off by downloading Composer:


```
curl -sS https://getcomposer.org/installer | php

```


Now install it globally:


```
mv composer.phar /usr/local/bin/composer

```


## Installing Laravel


Heads Up: If you’re installing Laravel on DigitalOcean’s 512MB VPS, make sure you add a swapfile to Ubuntu to prevent it from running out of memory. You can quickly do this by issuing the following commands. This will only work during 1 session, so if you reboot during this tutorial, add the swapfile again.


```
dd if=/dev/zero of=/swapfile bs=1024 count=512k
mkswap /swapfile
swapon /swapfile

```


Finally, let’s install Laravel.


```
composer create-project laravel/laravel /var/www/laravel/ 4.1

```


# Testing



Now browse to your cloud server’s IP. You can find using:


```
/sbin/ifconfig|grep inet|head -1|sed 's/\:/ /'|awk '{print $3}'

```


It will now show you an error! What? The permissions still need to be set on the folders used for caching. Ah! Let’s do that now:


## Fixing permissions


This is really quite a easy fix:


```
chgrp -R www-data /var/www/laravel
chmod -R 775 /var/www/laravel/app/storage

```


# Wrap Up



So that it, you can now enjoy Laravel running on a fast Nginx backend! If you want to use MySQL on your Laravel installation, it is extremely easy: just issue apt-get install mysql-server and MySQL will be installed right away. For more information on using Laravel visit the website. Happy developing!


<div class=“author”>Submitted by: Wouter ten Bosch</div>


