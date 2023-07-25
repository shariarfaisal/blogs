# How To Upgrade to PHP 7 on Ubuntu 14 04

```Apache``` ```Ubuntu``` ```PHP``` ```LAMP Stack``` ```Nginx``` ```LEMP```

## Introduction


PHP 7, which was released on December 3, 2015, promises substantial speed improvements over previous versions of the language, along with new features like scalar type hinting.  This guide explains how to quickly upgrade an Apache or Nginx web server running PHP 5.x (any release) to PHP 7.



Warning: As with most major-version language releases, it’s best to wait a little while before switching to PHP 7 in production.  In the meanwhile, it’s a good time to test your applications for compatibility with the new release, perform benchmarks, and familiarize yourself with new language features.
If you’re running any services or applications with active users, it is safest to first test this process in a staging environment.

# Prerequisites


This guide assumes that you are running PHP 5.x on an Ubuntu 14.04 machine, using either mod_php in conjunction with Apache, or PHP-FPM in conjunction with Nginx.  It also assumes that you have a non-root user configured with sudo privileges for administrative tasks.


# Adding a PPA for PHP 7.0 Packages


A Personal Package Archive, or PPA, is an Apt repository hosted on Launchpad.  PPAs allow third-party developers to build and distribute packages for Ubuntu outside of the official channels.  They’re often useful sources of beta software, modified builds, and backports to older releases of the operating system.


Ondřej Surý maintains the PHP packages for Debian, and offers a PPA for PHP 7.0 on Ubuntu.  Before doing anything else, log in to your system, and add Ondřej’s PPA to the system’s Apt sources:


```
sudo add-apt-repository ppa:ondrej/php


```


You’ll see a description of the PPA, followed by a prompt to continue.  Press Enter to proceed.



Note: If your system’s locale is set to anything other than UTF-8, adding the PPA may fail due to a bug handling characters in the author’s name.  As a workaround, you can install language-pack-en-base to make sure that locales are generated, and override system-wide locale settings while adding the PPA:
sudo apt-get install -y language-pack-en-base
sudo LC_ALL=en_US.UTF-8 add-apt-repository ppa:ondrej/php



Once the PPA is installed, update the local package cache to include its contents:


```
sudo apt-get update


```


Now that we have access to packages for PHP 7.0, we can replace the existing PHP installation.


# Upgrading mod_php with Apache


This section describes the upgrade process for a system using Apache as the web server and mod_php to execute PHP code.  If, instead, you are running Nginx and PHP-FPM, skip ahead to the next section.


First, install the new packages.  This will upgrade all of the important PHP packages, with the exception of php5-mysql, which will be removed.


```
sudo apt-get install php7.0


```



Note: If you have made substantial modifications to any configuration files in /etc/php5/, those files are still in place, and can be referenced.  Configuration files for PHP 7.0 now live in /etc/php/7.0.

If you are using MySQL, make sure to re-add the updated PHP MySQL bindings:


```
sudo apt-get install php7.0-mysql


```


# Upgrading PHP-FPM with Nginx


This section describes the upgrade process for a system using Nginx as the web server and PHP-FPM to execute PHP code.


First, install the new PHP-FPM package and its dependencies:


```
sudo apt-get install php7.0-fpm


```


You’ll be prompted to continue.  Press Enter to complete the installation.


If you are using MySQL, be sure to re-install the PHP MySQL bindings:


```
sudo apt-get install php7.0-mysql


```



Note: If you have made substantial modifications to any configuration files in /etc/php5/, those files are still in place, and can be referenced.  Configuration files for PHP 7.0 now live in /etc/php/7.0.

## Updating Nginx Site(s) to Use New Socket Path


Nginx communicates with PHP-FPM using a Unix domain socket.  Sockets map to a path on the filesystem, and our PHP 7 installation uses a new path by default:





PHP 5
PHP 7




/var/run/php5-fpm.sock
/var/run/php/php7.0-fpm.sock




Open the default site configuration file with nano (or your editor of choice):


```
sudo nano /etc/nginx/sites-enabled/default


```


Your configuration may differ somewhat.  Look for a block beginning with location ~ \.php$ {, and a line that looks something like fastcgi_pass unix:/var/run/php5-fpm.sock;.  Change this to use unix:/var/run/php/php7.0-fpm.sock.


/etc/nginx/sites-enabled/default
```
server {
    listen 80 default_server;
    listen [::]:80 default_server ipv6only=on;

    root /var/www/html;
    index index.php index.html index.htm;

    server_name server_domain_name_or_IP;

    location / {
        try_files $uri $uri/ =404;
    }

    error_page 404 /404.html;
    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
        root /usr/share/nginx/html;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}

```


Exit and save the file.  In nano, you can accomplish this by pressing Ctrl-X to exit, y to confirm, and Enter to confirm the filename to overwrite.


You should repeat this process for any other virtual sites defined in /etc/nginx/sites-enabled which need to support PHP.


Now we can restart nginx:


```
sudo service nginx restart


```


# Testing PHP


With a web server configured and the new packages installed, we should be able to verify that PHP is up and running.  Begin by checking the installed version of PHP at the command line:


```
php -v


```


```
OutputPHP 7.0.0-5+deb.sury.org~trusty+1 (cli) ( NTS )
Copyright (c) 1997-2015 The PHP Group
Zend Engine v3.0.0, Copyright (c) 1998-2015 Zend Technologies
    with Zend OPcache v7.0.6-dev, Copyright (c) 1999-2015, by Zend Technologies

```


You can also create a test file in the web server’s document root.  Depending on your server and configuration, this may be one of:


- /var/www/html
- /var/www/
- /usr/share/nginx/html

Using nano, open a new file called info.php in the document root.  By default, on Apache, this would be:


```
sudo nano /var/www/html/info.php


```


On Nginx, you might instead use:


```
sudo nano /usr/share/nginx/html/info.php


```


Paste the following code:


info.php
```
<?php
phpinfo();
?>

```


Exit the editor, saving info.php.  Now, load the following address in your browser:


```
http://server_domain_name_or_IP/info.php

```


You should see PHP version and configuration info for PHP 7.  Once you’ve double-checked this, it’s safest to to delete info.php:


```
sudo rm /var/www/html/info.php


```


# Conclusion


You now have a working PHP 7 installation.  From here, you may want to check out Erika Heidi’s Getting Ready for PHP 7 blog post, and look over the official migration guide.


