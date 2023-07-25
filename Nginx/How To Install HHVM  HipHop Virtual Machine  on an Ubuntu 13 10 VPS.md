# How To Install HHVM  HipHop Virtual Machine  on an Ubuntu 13 10 VPS

```Apache``` ```Ubuntu``` ```PHP``` ```Nginx```

## Introduction


This article will walk through the steps required to install HHVM on Ubuntu 13.10 on a Digital Ocean VPS. The article will also illustrate how HHVM can be used by creating:


1. 
A command line ‘Hello World’ script in PHP

2. 
A web based ‘Hello World’ script written in PHP and served by the HHVM server


# Prerequisites


The only prerequisite for this tutorial is a VPS with Ubuntu 13.10 x64 installed. Note that HHVM doesn’t support any 32 bit operating system and they have no plans to add support for 32 bit operating systems.


You will need to execute commands from the command line which you can do in one of two ways:


1. 
Use SSH to access the droplet.

2. 
Use the ‘Console Access’ from the Digital Ocean Droplet Management Panel


# What is HHVM?


HipHop Virtual Machine (HHVM) is a virtual machine developed and open sourced by Facebook to process and execute programs and scripts written in PHP. Facebook developed HHVM because the regular Zend+Apache combination isn’t as efficient to serve large applications built in PHP.


According to their website, HHVM has realized over a 9x increase in web request throughput and over a 5x reduction in memory consumption for Facebook compared with the Zend PHP engine + APC (which is the current way of hosting a large majority of PHP applications).


# Installing HHVM


Installing HHVM is quite straightforward and shouldn’t take more than a few minutes. Executing the following 4 commands from the command line will have HHVM installed and ready:


```
wget -O - http://dl.hhvm.com/conf/hhvm.gpg.key | apt-key add -
echo deb http://dl.hhvm.com/ubuntu saucy main | tee /etc/apt/sources.list.d/hhvm.list
apt-get update
apt-get install hhvm

```


To confirm that HHVM has been installed, type the following command:


```
hhvm --help

```


This will show details of how the hhvm command can be used from the command line. Here is a sample screenshot that illustrates this:





# Test a ‘Hello World’ script in PHP From the Command Line using HHVM


Type the following command on the command line:


```
cat > hello_world.php

```


This will create a file named hello_world.php and allow you to enter its contents. Type in (or copy and paste) the following code and then press Ctrl + D to save the file.


```
<?php

echo "\nHello World\n\n";

```


Note: If you are familiar with editors like nano or vim, you can use those to create and save this file.


Once this file has been created, hhvm can be used to execute it using the following command:


```
hhvm hello_world.php

```


Here is a screenshot to illustrate the creation and execution of this Hello World script:





# Test a ‘Hello World’ script in PHP Using the HHVM Server (accessible from the browser)


First create a directory (public) that will serve as the public folder and contain the PHP file. Note that you can name this directory anything and place it anywhere. Execute the following commands to create this directory and enter it:


```
mkdir public
cd public

```


Now type the following command on the command line:


```
cat > hello.php

```


This will create a file named hello.php in the public directory and allow you to enter its contents. Type in (or copy and paste) the following code and then press Ctrl + D to save the file.


```
<?php

echo '<h1>Hello World</h1>';

```


Note: If you are familiar with editors like nano or vim, you can use those to create and save this file.


Once this file has been created, hhvm can be used from the command line to start the server in the following manner:


```
hhvm -m server

```


Here is a screenshot to illustrate the creation and execution of this Hello World script:





This command will start the server (on port 80) and the hello.php file can now be accessed from the browser. If the IP address of the droplet is 128.199.212.7, this newly created file can be accessed at:


```
http://128.199.212.7/hello.php

```


Note: Replace 128.199.212.7 with the IP address or domain name used by your droplet.


Accessing this URL should display a webpage similar to the one in the following screenshot:





# Porting Your PHP Application to HHVM


In order to start serving a PHP application using HHVM instead of Zend/Apache on port 80 (the standard port for websites), the Apache service needs to be stopped. This can be done using the following command:


```
service apache2 stop

```


This command will stop Apache and free up port 80 for HHVM to use. The next step is to start the HHVM server in the root directory of your PHP application. The most common location of this root directory on Ubuntu is /var/www.


Switch to this directory using the following command:


```
cd /var/www

```


Once you are in this directory, all you need to do is start the HHVM server:


```
hhvm -m server

```


This command will start the HHVM server, which will begin serving your PHP application from the current directory.


# Using HHVM in the FastCGI Mode


Starting with version 3.0, HHVM can no longer be used in the server mode. This section will help you configure HHVM in the FastCGI mode with the Apache and Nginx servers.


## With Apache


Configuring HHVM to work in the FastCGI mode with Apache is extremely simple. All you need to do is execute the following following script:


```
/usr/share/hhvm/install_fastcgi.sh

```


Running this script configures Apache to start using HHVM to process the PHP code. It’ll also restart the Apache server so you don’t have to do anything else.


## With Nginx


If you are using Nginx with PHP-FPM, you’ll have to modify the configuration file to disable the use of PHP-FPM. This file is normally located at /etc/nginx/sites-available/default


Look for the following section and make sure it’s all commented (by adding a # at the beginning of each line)


```
# pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
#
#location ~ \.php$ {
#       fastcgi_split_path_info ^(.+\.php)(/.+)$;
#       # NOTE: You should have "cgi.fix_pathinfo = 0;" in php.ini
#
#       # With php5-cgi alone:
#       fastcgi_pass 127.0.0.1:9000;
#       # With php5-fpm:
#       fastcgi_pass unix:/var/run/php5-fpm.sock;
#       fastcgi_index index.php;
#       include fastcgi_params;
#}

```


After doing this, execute the following script:


```
/usr/share/hhvm/install_fastcgi.sh

```


Executing this script configures Nginx to start using HHVM to process the PHP code. It’ll also restart the Nginx server so you don’t have to do anything else.


## Confirming that Apache/Nginx is Using HHVM


After you have configured your server to start using HHVM, it’s always a good idea to confirm that the server (Apache or Nginx) is indeed using HHVM to process PHP.


You can do this by creating a test PHP file, let’s say info.php and putting it in the public folder of your server (typically /var/www for Apache and /usr/share/nginx/html for Nginx). Now put the following content in this file:


```
<?php

echo  defined('HHVM_VERSION')?'Using HHVM':'Not using HHVM';

```


Now if everything is set up fine, when you access this file in the browser, you should see the following message:



Using HHVM

# Important Note


HHVM has incorporated a lot of commonly used PHP extensions, making it easy to port a large number of applications without much fuss. However, if an application uses a PHP extension that hasn’t been incorporated yet, choosing HHVM will break the application. The complete list of PHP extensions that have been ported over to HHVM can be found here


## Final Word


If you have reached this far, you are now well equipped to start using HHVM to serve your PHP based websites. Being several times more efficient than the regular Zend PHP engine + APC combination, HHVM can help you serve a lot more visitors to your website with much lower hardware requirements!


<div class=“author”>Submitted by: <a href=“http://javascript.asia”>Jay</a></div>


