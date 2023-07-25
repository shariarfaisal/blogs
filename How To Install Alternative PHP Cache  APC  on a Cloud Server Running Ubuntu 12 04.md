# How To Install Alternative PHP Cache  APC  on a Cloud Server Running Ubuntu 12 04

```Ubuntu``` ```Caching``` ```PHP```











# Status: Deprecated


This article covers a version of Ubuntu that is no longer supported.  If you are currently operate a server running Ubuntu 12.04, we highly recommend upgrading or migrating to a supported version of Ubuntu:


- Upgrade to Ubuntu 14.04.
- Upgrade from Ubuntu 14.04 to Ubuntu 16.04
- Migrate the server data to a supported version

Reason:
Ubuntu 12.04 reached end of life (EOL) on April 28, 2017 and no longer receives security patches or updates.  This guide is no longer maintained.


See Instead:
This guide might still be useful as a reference, but may not work on other Ubuntu releases.  If available, we strongly recommend using a guide written for the version of Ubuntu you are using.   You can use the search functionality at the top of the page to find a more recent version.


## About APC


APC is a great operation code caching system for PHP that can help speed up your site. PHP is a dynamic server-side scripting language that needs to be parsed, compiled and executed by the server with every page request. In many cases though, the requests produce exactly the same results which means that the cloud server has to unnecessarily repeat all these steps for each of them.


This is where APC comes into play. What it does is save the PHP opcode (operation code) in the RAM memory and if requested again, executes it from there. In essence, it bypasses the parsing and compiling steps and minimizes some unnecessary loads on the cloud server.


This tutorial will show you how to install and configure APC. It assumes you are already running your own VPS with root privileges and have LAMP stack installed on it. If you need help with getting you going on those, you can read this tutorial.


# Installing APC


To install APC, you first need to take care of a couple of dependencies. Install the these packages with the following command:


```
sudo apt-get install php-pear php5-dev make libpcre3-dev
```


Next up, you can install APC using the pecl command:


```
sudo pecl install apc
```


You will be asked a number of questions but unless you know exactly what you are enabling, go with the defaults by hitting Enter.


The next and final step of the installation is also mentioned in the terminal window. You need to edit the php.ini file and add a line at the end. Open and edit the file:


```
sudo nano /etc/php5/apache2/php.ini
```


Add the following line to the bottom of it:


```
extension = apc.so
```


Save, exit the file, and restart Apache:


```
sudo service apache2 restart
```


To see if APC is now enabled, you can check on the PHP info page. If you don't have one, you can create an empty php file in your /var/www folder:


```
nano /var/www/info.php
```


And paste in the following code:


```
<?php
phpinfo();
?>
```


Save, exit, and open that file in the browser. There you will find all sorts of information regarding the PHP installed on your cloud server, and if APC is enabled, it should show up there. It's probably not a good idea to leave that file there in production, so make sure you delete it after you are done checking.


# Configuring APC


You now have installed APC and it's running with the default options. There are at least two main configuration settings that you should know about. First, reopen the php.ini file you edited earlier:


```
sudo nano /etc/php5/apache2/php.ini
```


Below the line you pasted to enable APC, paste the following line:


```
apc.shm_size = 64
```


This will allocate 64MB from the RAM to APC for its caching purposes. Depending on your VPS's requirements but also limitations, you can increase or decrease this number.


Another line that you can paste below is the following:


```
apc.stat = 0
```


The apc.stat setting checks the script on each request to see if it was modified. If it has been modified, it will recompile it and cache the new version. This is the default behavior that comes with every APC installation. Setting it to 0 will tell APC not to check for changes in the script. It improves performance but it also means that if there are changes to the PHP script, they will not be reflected until the cloud server is restarted. Therefore setting it to 0 is only recommended on production sites where you are certain this is something you want.


Now that APC is up and running, there is a nifty little page you can use to check its status and performance. You can locate an apc.php file in the /usr/share/php/ folder. You have to move this file somewhere accessible from the browser - let's say the www folder:


```
cp /usr/share/php/apc.php /var/www
```


Now navigate to that file in the browser:


```
http://<IP_Address>/apc.php
```


You'll get some interesting statistics about APC. What you need to pay attention to is that APC has enough memory to store its information and that there is not too much fragmentation.


Additionally, a good indicator that APC it's doing its job is that the Hits rate is significantly higher than the Misses rate; the first should be somewhere over 95% after a few requests already.


# Conclusion


APC is a very easy to install and manage caching system for your sites hosted on cloud servers. If you want to continue improving site performance, you can look into installing Memcache and even installing Varnish for an even greater performance. 


Article Submitted by: Danny
