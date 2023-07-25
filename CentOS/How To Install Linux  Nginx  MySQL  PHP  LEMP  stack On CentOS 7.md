# How To Install Linux  Nginx  MySQL  PHP  LEMP  stack On CentOS 7

```MySQL``` ```LEMP``` ```Nginx``` ```PHP``` ```CentOS```

## Introduction


A LEMP software stack is a group of open source software that is typically installed together to enable a server to host dynamic websites and web apps.  This term is actually an acronym which represents the Linux operating system, with the ENginx web server (which replaces the Apache component of a LAMP stack).  The site data is stored in a MySQL-based database, and dynamic content is processed by PHP.


In this guide, we’ll get a LEMP stack with PHP 7.4 installed on a CentOS 7 server, using MariaDB as the database management system. MariaDB works as a drop-in replacement for the original MySQL server, which in practice means you can switch to MariaDB without having to make any configuration or code changes in your application.


# Prerequisites


Before you begin with this guide, you should have a separate, non-root user account set up on your server.  You can learn how to do this by completing steps 1-4 in the initial server setup for CentOS 7.


# Step 1 — Installing Nginx


In order to display web pages to our site visitors, we are going to employ Nginx, a high-performance web server. To get the latest Nginx version, we’ll first install the EPEL repository, which contains additional software for the CentOS 7 operating system.


To add the CentOS 7 EPEL repository, run the following command:


```
sudo yum install epel-release


```


Since we are using a sudo command, these operations get executed with root privileges.  It will ask you for your regular user’s password to verify that you have permission to run commands with root privileges. You’ll also be prompted to confirm installation, so press Y to proceed.


Now that the EPEL repository is installed on your server, install Nginx using the following yum command:


```
sudo yum install nginx


```


Once the installation is finished, start the Nginx service with:


```
sudo systemctl start nginx


```


You can do a spot check right away to verify that everything went as planned by visiting your server’s public IP address in your web browser (see the note under the next heading to find out what your public IP address is if you do not have this information already):


```
Open in a web browser:http://server_domain_name_or_IP/

```


You will see the default CentOS 7 Nginx web page, which is there for informational and testing purposes.  It should look something like this:





If you see this page, then your web server is now correctly installed.


To enable Nginx to start on boot, run the following command:


```
sudo systemctl enable nginx


```


## How To Find Your Server’s Public IP Address


If you do not know what your server’s public IP address is, there are a number of ways you can find it.  Usually, this is the address you use to connect to your server through SSH.


From the command line, you can find this a few ways.  First, you can use the iproute2 tools to get your address by typing this:


```
ip addr show eth0 | grep inet | awk '{ print $2; }' | sed 's/\/.*$//'


```


This will give you one or two lines back.  They are both correct addresses, but your computer may only be able to use one of them, so feel free to try each one.


An alternative method is to use an outside party to tell you how it sees your server.  You can do this by asking a specific server what your IP address is:


```
curl http://icanhazip.com


```


Regardless of the method you use to get your IP address, you can type it into your web browser’s address bar to get to your server.


# Step 2 — Installing MariaDB


Now that we have our web server up and running, it is time to install MariaDB, a MySQL drop-in replacement.  MariaDB is a community-developed fork of the MySQL relational database management system.


Again, we can use yum to acquire and install our software.  This time, we’ll also install some other helper packages that will assist us in getting our components to communicate with each other:


```
sudo yum install mariadb-server mariadb


```


When the installation is complete, we need to start MariaDB with the following command:


```
sudo systemctl start mariadb


```


Now that our MariaDB database is running, we want to run a security script that will remove some dangerous defaults and lock down access to our database. Start the interactive script by running:


```
sudo mysql_secure_installation


```


The prompt will ask you for your current root MariaDB password. Since you just installed MariaDB, you most likely won’t have one, so leave it blank by pressing enter. Then the prompt will ask you if you want to set a root password. Go ahead and enter Y, and follow the instructions:


```
mysql_secure_installation prompts:Enter current password for root (enter for none): 
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

Set root password? [Y/n] y
New password: 
Re-enter new password: 
Password updated successfully!
Reloading privilege tables..
 ... Success!

```


For the rest of the questions, you should hit the “ENTER” key through each prompt to accept the default values.  This will remove some sample users and databases, disable remote root logins, and load these new rules so that MySQL immediately respects the changes we have made.


The last thing you will want to do is enable MariaDB to start on boot. Use the following command to do so:


```
sudo systemctl enable mariadb


```


At this point, your database system is now set up and we can move on.


# Step 3 — Installing PHP


PHP is the component of our setup that will process code to display dynamic content.  It can run scripts, connect to our MySQL databases to get information, and hand the processed content over to our web server to display.


The PHP version available by default within CentOS 7 servers is outdated, and for that reason, we’ll need to install a third-party package repository in order to obtain PHP 7+ and get it installed on your CentOS 7 server. Remi is a popular package repository providing the most up-to-date PHP releases for CentOS servers.


To install the Remi repository for CentOS 7, run:


```
sudo yum install http://rpms.remirepo.net/enterprise/remi-release-7.rpm


```


After the installation is done, you’ll need to run a command to enable the repository containing your preferred version of PHP. To check which PHP 7+ releases are available in the Remi repository, run:


```
yum --disablerepo="*" --enablerepo="remi-safe" list php[7-9][0-9].x86_64


```


You’ll see output like this:


```
OutputLoaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * remi-safe: mirrors.ukfast.co.uk
Available Packages
php70.x86_64                                              2.0-1.el7.remi                                       remi-safe
php71.x86_64                                              2.0-1.el7.remi                                       remi-safe
php72.x86_64                                              2.0-1.el7.remi                                       remi-safe
php73.x86_64                                              2.0-1.el7.remi                                       remi-safe
php74.x86_64                                              1.0-3.el7.remi                                       remi-safe
php80.x86_64                                              1.0-3.el7.remi                                       remi-safe

```


In this guide, we’ll install PHP 7.4, which is currently the most updated stable version of PHP. To enable the correct Remi package to get PHP 7.4 installed, run:


```
sudo yum-config-manager --enable remi-php74


```


Now we can proceed to use yum for installing PHP as usual. The following command will install all the required packages to get PHP 7.4 set up within Nginx and allow it to connect to MySQL-based databases:


```
sudo yum install php php-mysqlnd php-fpm


```


To confirm that PHP is available as your chosen version, run:


```
php --version


```


You’ll see output like this:


```
OutputPHP 7.4.5 (cli) (built: Apr 14 2020 12:54:33) ( NTS )
Copyright (c) The PHP Group
Zend Engine v3.4.0, Copyright (c) Zend Technologies

```


PHP is now successfully installed on your system. Next, we need to make a few adjustments to the default configuration. To facilitate editing files on CentOS, we’ll first install nano, a more user-friendly text editor than vi:


```
sudo yum install nano


```


Open the /etc/php-fpm.d/www.conf configuration file using nano or your editor of choice:


```
sudo nano /etc/php-fpm.d/www.conf


```


Now look for the user and group directives. If you are using nano, you can hit CTRL+W to search for these terms inside the open file.


/etc/php-fpm.d/www.conf
```
…
; Unix user/group of processes
; Note: The user is mandatory. If the group is not set, the default user's group
;       will be used.
; RPM: apache user chosen to provide access to the same directories as httpd
user = apache
; RPM: Keep a group allowed to write in log dir.
group = apache
…

```


You’ll notice that both the user and group variables are set to apache. We need to change these to nginx:


/etc/php-fpm.d/www.conf
```
…
; RPM: apache user chosen to provide access to the same directories as httpd
user = nginx
; RPM: Keep a group allowed to write in log dir.
group = nginx
…

```


Next, locate the listen directive. By default, php-fpm will listen on a specific host and port  over TCP. We want to change this setting so it listens on a local socket file, since this improves the overall performance of the server.
Change the line containing the listen directive to the following:


/etc/php-fpm.d/www.conf
```
listen = /var/run/php-fpm/php-fpm.sock;

```


Finally, we’ll need to change the owner and group settings for the socket file we just defined within the listen directive. Locate the listen.owner, listen.group and listen.mode directives. These lines are commented out by default. Uncomment them by removing the preceding ; sign at the beginning of the line. Then, change the owner and group to nginx:


/etc/php-fpm.d/www.conf
```
listen.owner = nginx
listen.group = nginx
listen.mode = 0660

```


Save and close the file when you’re done editing. If you are using nano, do so by pressing CTRL + X, then Y and ENTER.


To enable and start the php-fpm service, run:


```
sudo systemctl start php-fpm


```


Your PHP environment is now ready. Next, we’ll configure Nginx so that it sends all requests for PHP scripts to be processed by php-fpm.


# Step 4 — Configuring Nginx to Process PHP Pages


Now, we have all of the required components installed. The only configuration change we still need to do is tell Nginx to use our PHP processor for dynamic content.


Nginx has a dedicated directory where we can define each hosted website as a separate configuration file, using a server block. This is similar to Apache’s virtual hosts.


With the default installation, however, this directory is empty. We’ll create a new file to serve as the default PHP website on this server, which will override the default server block defined in the /etc/nginx/nginx.conf file.


First, open a new file in the /etc/nginx/conf.d directory:


```
sudo nano /etc/nginx/conf.d/default.conf


```


Copy the following PHP server definition block to your configuration file, and don’t forget to replace the server_name directive so that it points to your server’s domain name or IP address:


/etc/nginx/conf.d/default.conf
```
server {
    listen       80;
    server_name  server_domain_or_IP;

    root   /usr/share/nginx/html;
    index index.php index.html index.htm;

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
        fastcgi_pass unix:/var/run/php-fpm/php-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}

```


Save and close the file when you’re done.


Next, restart Nginx to apply the changes:


```
sudo systemctl restart nginx


```


Your web server is now fully set up. In the next step, we’ll test the PHP integration to Nginx.


# Step 5 — Testing PHP Processing on your Web Server


Now that your web server is set up, we can create a test PHP script to make sure Nginx is correctly handling .php scripts with the help of php-fpm.


Before creating our script, we’ll make a change to the default ownership settings on Nginx’s document root, so that our regular sudo user is able to create files in that location.


The following command will change the ownership of the default Nginx document root to a user and group called sammy, so be sure to replace the highlighted username and group in this command to reflect your system’s username and group.


```
sudo chown -R sammy.sammy /usr/share/nginx/html/


```


We’ll now create a test PHP page to make sure the web server works as expected.


Create a new PHP file called info.php at the /usr/share/nginx/html directory:


```
nano /usr/share/nginx/html/info.php


```


The following PHP code will display information about the current PHP environment running on the server:


/usr/share/nginx/html/info.php
```
<?php

phpinfo();

```


When you are finished, save and close the file.


Now we can test whether our web server can correctly display content generated by a PHP script.  Go to your browser and access your server hostname or IP address, followed by /info.php:


```
http://server_host_or_IP/info.php

```


You’ll see a page similar to this:





After checking the relevant information about your PHP server through that page, it’s best to remove the file you created as it contains sensitive information about your PHP environment and your CentOS server. You can use rm to remove that file:


```
rm /usr/share/nginx/html/info.php


```


You can always regenerate this file if you need it later.


# Conclusion


In this guide, you’ve built a flexible foundation for serving PHP websites and applications to your visitors, using Nginx as web server and the latest PHP release version. You’ve set up Nginx to handle PHP requests through php-fpm, and you also set up a MariaDB database to store your website’s data.


