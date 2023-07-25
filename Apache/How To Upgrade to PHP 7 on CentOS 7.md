# How To Upgrade to PHP 7 on CentOS 7

```Apache``` ```PHP``` ```LEMP``` ```Nginx``` ```LAMP Stack``` ```CentOS```

## Introduction


PHP 7, which was released on December 3, 2015, promises substantial speed improvements over previous versions of the language, along with new features like scalar type hinting.  This guide explains how to quickly upgrade an Apache or Nginx web server running PHP 5.x (any release) to PHP 7, using community-provided packages.



Warning: As with most major-version language releases, it’s best to wait a little while before switching to PHP 7 in production.  In the meanwhile, it’s a good time to test your applications for compatibility with the new release, perform benchmarks, and familiarize yourself with new language features.

If you have installed phpMyAdmin for database management, it is strongly recommended that you wait for official CentOS PHP 7 packages before upgrading, as phpMyAdmin packages do not yet support the upgrade.  If you’re running any other services or applications with active users, it is safest to first test this process in a staging environment.


# Prerequisites


This guide assumes that you are running PHP 5.x on CentOS 7, using either mod_php in conjunction with Apache, or PHP-FPM in conjunction with Nginx.  It also assumes that you have a non-root user configured with sudo privileges for administrative tasks.


The PHP 5 installation process is documented in these guides:


- How To Install Linux, Apache, MySQL, PHP (LAMP) stack On CentOS 7
- How To Install Linux, Nginx, MySQL, PHP (LEMP) stack On CentOS 7

# Subscribing to the IUS Community Project Repository


Since PHP 7.x is not yet packaged in official repositories for the major distributions, we’ll have to rely on a third-party source.  Several repositories offer PHP 7 RPM files.  We’ll use the IUS repository.


IUS offers an installation script for subscribing to their repository and importing associated GPG keys.  Make sure you’re in your home directory, and retrieve the script using curl:


```
cd ~
curl 'https://setup.ius.io/' -o setup-ius.sh


```


Run the script:


```
sudo bash setup-ius.sh


```


# Upgrading mod_php with Apache


This section describes the upgrade process for a system using Apache as the web server and mod_php to execute PHP code.  If, instead, you are running Nginx and PHP-FPM, skip ahead to the next section.


Begin by removing existing PHP packages.  Press y and hit Enter to continue when prompted.


```
sudo yum remove php-cli mod_php php-common


```


Install the new PHP 7 packages from IUS.  Again, press y and Enter when prompted.


```
sudo yum install mod_php70u php70u-cli php70u-mysqlnd


```


Finally, restart Apache to load the new version of mod_php:


```
sudo apachectl restart


```


You can check on the status of Apache, which is managed by the httpd systemd unit, using systemctl:


```
systemctl status httpd


```


# Upgrading PHP-FPM with Nginx


This section describes the upgrade process for a system using Nginx as the web server and PHP-FPM to execute PHP code.  If you have already upgraded an Apache-based system, skip ahead to the PHP Testing section.


Begin by removing existing PHP packages.  Press y and hit Enter to continue when prompted.


```
sudo yum remove php-fpm php-cli php-common


```


Install the new PHP 7 packages from IUS.  Again, press y and Enter when prompted.


```
sudo yum install php70u-fpm-nginx php70u-cli php70u-mysqlnd


```


Once the installation is finished, you’ll need to make a few configuration changes for both PHP-FPM and Nginx.  As configured, PHP-FPM listens for connections on a local TCP socket, while Nginx expects a Unix domain socket, which maps to a path on the filesystem.


PHP-FPM can handle multiple pools of child processes.  As configured, it provides a single pool called www, which is defined in /etc/php-fpm.d/www.conf.  Open this file with nano (or your preferred text editor):


```
sudo nano /etc/php-fpm.d/www.conf


```


Look for the block containing listen = 127.0.0.1:9000, which tells PHP-FPM to listen on the loopback address at port 9000.  Comment this line with a semicolon, and uncomment listen = /run/php-fpm/www.sock a few lines below.


/etc/php-fpm.d/www.conf
```
; The address on which to accept FastCGI requests.
; Valid syntaxes are:
;   'ip.add.re.ss:port'    - to listen on a TCP socket to a specific IPv4 address on
;                            a specific port;
;   '[ip:6:addr:ess]:port' - to listen on a TCP socket to a specific IPv6 address on
;                            a specific port;
;   'port'                 - to listen on a TCP socket to all addresses
;                            (IPv6 and IPv4-mapped) on a specific port;
;   '/path/to/unix/socket' - to listen on a unix socket.
; Note: This value is mandatory.
;listen = 127.0.0.1:9000
; WARNING: If you switch to a unix socket, you have to grant your webserver user
;          access to that socket by setting listen.acl_users to the webserver user.
listen = /run/php-fpm/www.sock

```


Next, look for the block containing listen.acl_users values, and uncomment listen.acl_users = nginx:


/etc/php-fpm.d/www.conf
```
; When POSIX Access Control Lists are supported you can set them using
; these options, value is a comma separated list of user/group names.
; When set, listen.owner and listen.group are ignored
;listen.acl_users = apache,nginx
;listen.acl_users = apache
listen.acl_users = nginx
;listen.acl_groups =

```


Exit and save the file.  In nano, you can accomplish this by pressing Ctrl-X to exit, y to confirm, and Enter to confirm the filename to overwrite.


Next, make sure that Nginx is using the correct socket path to handle PHP files.  Start by opening /etc/nginx/conf.d/default.conf:


```
sudo nano /etc/nginx/conf.d/php-fpm.conf


```


php-fpm.conf defines an upstream, which can be referenced by other Nginx configuration directives.  Inside of the upstream block, use a # to comment out server 127.0.0.1:9000;, and uncomment server unix:/run/php-fpm/www.sock;:


/etc/nginx/conf.d/php-fpm.conf
```
# PHP-FPM FastCGI server
# network or unix domain socket configuration

upstream php-fpm {
        #server 127.0.0.1:9000;
        server unix:/run/php-fpm/www.sock;
}

```


Exit and save the file, then open /etc/nginx/conf.d/default.conf:


```
sudo nano /etc/nginx/conf.d/default.conf


```


Look for a block beginning with location ~ \.php$ {.  Within this block, look for the fastcgi_pass directive.  Comment out or delete this line, and replace it with fastcgi_pass php-fpm, which will reference the upstream defined in php-fpm.conf:


/etc/nginx/conf.d/default.conf
```
  location ~ \.php$ {
      try_files $uri =404;
      fastcgi_split_path_info ^(.+\.php)(/.+)$;
      # fastcgi_pass unix:/var/run/php-fpm/php-fpm.sock;
      fastcgi_pass php-fpm;
      fastcgi_index index.php;
      fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
      include fastcgi_params;
  }

```


Exit and save the file, then restart PHP-FPM and Nginx so that the new configuration directives take effect:


```
sudo systemctl restart php-fpm
sudo systemctl restart nginx


```


You can check on the status of each service using systemctl:


```
systemctl status php-fpm
systemctl status nginx


```


# Testing PHP


With a web server configured and the new packages installed, we should be able to verify that PHP is up and running.  Begin by checking the installed version of PHP at the command line:


```
php -v


```


Output
```
PHP 7.0.1 (cli) (built: Dec 18 2015 16:35:26) ( NTS )
Copyright (c) 1997-2015 The PHP Group
Zend Engine v3.0.0, Copyright (c) 1998-2015 Zend Technologies

```


You can also create a test file in the web server’s document root.  Although its location depends on your server configuration, the document root is typically set to one of these directories:


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

```


Exit the editor, saving info.php.  Now, load the following address in your browser:


```
http://server_domain_name_or_IP/info.php

```


You should see the PHP 7 information page, which lists the running version and configuration.  Once you’ve double-checked this, it’s safest to delete info.php:


```
sudo rm /var/www/html/info.php


```


You now have a working PHP 7 installation.  From here, you may want to check out Erika Heidi’s Getting Ready for PHP 7 blog post, and look over the official migration guide.


