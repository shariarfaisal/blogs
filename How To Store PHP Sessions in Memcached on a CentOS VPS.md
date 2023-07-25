# How To Store PHP Sessions in Memcached on a CentOS VPS

```PHP``` ```Server Optimization``` ```CentOS```

# About Memcached


Memcached is an in-memory key-value store for small chunks of arbitrary data (strings, objects) from results of database calls, API calls, or page rendering.


## Why store sessions in Memcached?


Memcached will store sessions in memory instead of files. Because memory is way faster than reading a file your website will perform better and reduce load time.


## What's the catch?


Sessions will only be stored in memory, memory can't hold data when your VPS is turned off or gets restarted so sessions will be deleted upon shutdown.


# Setup


Before starting this tutorial, make sure you have an up and running PHP 5 installation, you can find tutorials on how to do this in the PHP help section.


Make sure you have the EPEL repository installed, you need the EPEL repository for Memcached because Memcached isn't available in the base repository.


```
rpm -Uvh http://mirrors.kernel.org/fedora-epel/6/x86_64/epel-release-6-8.noarch.rpm
```


Updating packages to the latest available version isn't required but recommended.


```
yum update
```


# Installing Memcached


Lets start with installing Memcached.


```
yum install memcached
```


After installing Memcached, open the configuration file of Memcached with VI.


```
vi /etc/sysconfig/memcached
```


You will see this:


```
PORT="11211"
USER="memcached"
MAXCONN="1024"
CACHESIZE="64"
OPTIONS=""
```


Memcached isn't protected with a password or username so anyone can access it through port 11211. We don't want this so we are going to allow only your VPS to access it by inserting some options in the 'OPTIONS=' section:


```
PORT="11211"
USER="memcached"
MAXCONN="1024"
CACHESIZE="64"
OPTIONS="-l 127.0.0.1"
```


You might want to change the cachesize; by default it is 64MB. As soon as Memcached reaches this limit, it will delete old entries to free up memory for new entries. Unless you have a very huge website, 64MB should be fine.


Lets start Memcached.


```
/etc/init.d/memcached start
```


Memcached doesn't start by default upon boot, we want it to start upon boot.


```
chkconfig --levels 235 memcached on
```


# Installing the Memcached PHP Extension


We need to install a few things, lets start with the development tools. These are required to build from the source code:


```
yum groupinstall "Development Tools"
```


After that, we are going to install a few more things. The first two are required to build the extension, and the last two are required to run / install the extension in PHP.


```
yum install zlib-devel libmemcached-devel php-pear php-pecl-memcached
```


Now we are going to install the PHP Memcached extension using PECL (PHP Extension Community Library) which we have just installed.


```
pecl install -f memcached-1.0.0
```


# Changing PHP.ini to Setup Memcached as Session Handler


The last thing to do is to configure PHP to use Memcached as session handler. To do so, you have to open /etc/php.ini with VI.


```
vi /etc/php.ini
```


Search for the '[Session]' area as displayed below (scrolling from bottom to top is easier).


```
[Session]
; Handler used to store/retrieve data.
; http://www.php.net/manual/en/session.configuration.php#ini.session.save-handler
session.save_handler = files
```


And change it to this:


```
[Session]
; Handler used to store/retrieve data.
; http://www.php.net/manual/en/session.configuration.php#ini.session.save-handler
session.save_handler = memcached
session.save_path = "127.0.0.1:11211"
```


As you can see, we've changed the session_handler to memcached and the path to our localhost on port 11211 on which Memcached operates. Now lets restart Apache to reload the PHP.ini file.


```
service httpd restart
```


All sessions are now stored in Memcached instead of files.


You may see the following error:


```
Starting httpd: httpd: apr_sockaddr_info_get() failed for memcached
httpd: Could not reliably determine the server's fully qualified domain name, using 127.0.0.1 for ServerName
```


You can resolve this by editing the apache configuration:


```
vi /etc/httpd/conf/httpd.conf
```


and uncomment the ServerName line:


```
ServerName localhost
```


# More with Memcached


Memcached is ideal for storing intensive queries which doesn't need to be in realtime on every page view but in specified time increments (ie. every 10 minutes). For more information on how to use Memcached inside your scripts, I recommend you to look at step three in this tutorial.


Submitted by: Tim Kotkamp
