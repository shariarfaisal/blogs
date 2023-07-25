# How To Compile Nginx from Source on a CentOS 6 4 x64 VPS

```Nginx``` ```CentOS```


Status: Deprecated
This article covers a version of CentOS that is no longer supported. If you are currently operating a server running CentOS 6, we highly recommend upgrading or migrating to a supported version of CentOS.
Reason:
CentOS 6 reached end of life (EOL) on November 30th, 2020 and no longer receives security patches or updates. For this reason, this guide is no longer maintained.
See Instead:
This guide might still be useful as a reference, but may not work on other CentOS releases. If available, we strongly recommend using a guide written for the version of CentOS you are using.

## Introduction



nginx pronounced as “engine x” is an HTTP and reverse proxy server, as well as a mail proxy server.


nginx is an opensource web server which uses epoll mechanism to serve clients as opposed to Apache which uses a thread based model which delegates the requests to an instance in the thread pool. nginx is being used more over Apache because of its speed. nginx has over 13% market share and increasing continuously.


## Why compile from source


Compiling from the source code is useful when:


1. Upgrading to the latest version right after the release.
2. Fixing security vulnerabilities
3. Fixing known bugs which has been impacting your service
4. Modifying defaults like the server name
5. Applying patches with fixing known bugs
6. Software repositories are not upgraded with the latest version because of the dependency hierarchy

Its painful because:


1. You need to keep up to date with software versions
2. Your server software might depend on older versions in the dependency tree

## Modules and 3rd Party Modules


nginx has many modules which add features on top of existing VPS. Some of the famous 3rd party modules are:


- SPDY was included in nginx 1.5.0 (earlier was provided as a patch)
- google pagespeed(source) speeds up your site and reduces page load time by automatically applying web performance best practices to pages and associated assets (CSS, JavaScript, images) without requiring you to modify your existing content or workflow
- ModSecurity is an open source web application firewall to reduce known attacks on application servers.
*TCP Proxy allows nginx to proxy through tcp server instead of the default socket mode.

# Setup your VPS



## Create a new VPS or choose an existing one


Create a new VPS with your domain name. We will be using ‘example.com’ as the domain or you can use the server IP instead of the hostname.


# Setting Up Pre-requisites to Compile from Source



## Securely Login into your VPS


You can login into the VPS with your password for your host.


```
ssh root@example.com

```


## Install dependencies for nginx


We have few pre-requisites to be installed to compile which include development libraries along with source code compilers.


```
yum -y install gcc gcc-c++ make zlib-devel pcre-devel openssl-devel

```


Lets first create a directory to store our source code:


```
mkdir -p src && cd src

```


# Compiling from Source



## Downloading the source code


Lets get the current nginx version number from http://nginx.org/en/download.html


Run the following commands to download the source.


```
nginxVersion="1.5.5"
wget http://nginx.org/download/nginx-$nginxVersion.tar.gz
tar -xzf nginx-$nginxVersion.tar.gz 
ln -sf nginx-$nginxVersion nginx

```


## Preparing the nginx source


We first want to prepare nginx with necessary basic options.


For a full list of options you can look at ./configure --help


## Options for basic file path names


These options are the basic variables which we override to use default system paths at /etc/ to ensure it works simliar when installed via rpm. The user and group option are used to run the nginx worker processes in non-privileged.


```
--user
--group
--prefix
--sbin-path
--conf-path
--pid-path
--lock-path
--error-log-path
--http-log-path

```


## Other options


- 
--with-http_gzip_static_module option enables nginx to use gzip (Before serving a file from disk to a gzip-enabled client, this module will look for a precompressed file in the same location that ends in “.gz”. The purpose is to avoid compressing the same file each time it is requested.).[recommended for reducing size of information sent]

- 
--with-http_stub_status_module option enables other plugins over nginx to allow us to get the status (This module provides the ability to get some status from nginx.). [recommended for getting stats]

- 
--with-http_ssl_module - required if you want to run a HTTPS server. See How To Create a SSL Certificate on nginx for CentOS 6

- 
--with-pcre option enables to match routes via Regular Expression Matching when defining routes. [recommended, you will find more use of this once you start adding and matching routes]

- 
--with-file-aio - enables asynchronous I/O, better than the default send file option (recommended if you are allowing users to download static files)

- 
--with-http_realip_module is used for getting the IP of the client when behind a load balancer. This is useful when serving content behind CloudFlare like services.

- 
--without-http_scgi_module - Disable SCGI module (normally used when running CGI scripts)

- 
--without-http_uwsgi_module - Disable UWSGI module (normally used when running CGI scripts)

- 
--without-http_fastcgi_module - Disable FastCGI module (normally used when running CGI scripts)


## Our configuration options


```
cd nginx

./configure \
--user=nginx                          \
--group=nginx                         \
--prefix=/etc/nginx                   \
--sbin-path=/usr/sbin/nginx           \
--conf-path=/etc/nginx/nginx.conf     \
--pid-path=/var/run/nginx.pid         \
--lock-path=/var/run/nginx.lock       \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--with-http_gzip_static_module        \
--with-http_stub_status_module        \
--with-http_ssl_module                \
--with-pcre                           \
--with-file-aio                       \
--with-http_realip_module             \
--without-http_scgi_module            \
--without-http_uwsgi_module           \
--without-http_fastcgi_module

```


## Compiling the nginx source


Once we are able to configure the source which even checks for additional requirements like the compiler(gcc, g++) which we installed in the pre-requisites step:


```
 make
 make install

```


## Running the VPS


1. 
Add the user nginx to the system. This is a one time command:
 useradd -r nginx


2. 
We need to setup the file /etc/init.d/nginx to run when system starts:
 #!/bin/sh
 #
 # nginx - this script starts and stops the nginx daemin
 #
 # chkconfig:   - 85 15
 # description:  Nginx is an HTTP(S) server, HTTP(S) reverse \
 #               proxy and IMAP/POP3 proxy server
 # processname: nginx
 # config:      /etc/nginx/nginx.conf
 # pidfile:     /var/run/nginx.pid
 # user:        nginx
 
 # Source function library.
 . /etc/rc.d/init.d/functions
 
 # Source networking configuration.
 . /etc/sysconfig/network
 
 # Check that networking is up.
 [ "$NETWORKING" = "no" ] && exit 0
 
 nginx="/usr/sbin/nginx"
 prog=$(basename $nginx)
 
 NGINX_CONF_FILE="/etc/nginx/nginx.conf"
 
 lockfile=/var/run/nginx.lock
 
 start() {
     [ -x $nginx ] || exit 5
     [ -f $NGINX_CONF_FILE ] || exit 6
     echo -n $"Starting $prog: "
     daemon $nginx -c $NGINX_CONF_FILE
     retval=$?
     echo
     [ $retval -eq 0 ] && touch $lockfile
     return $retval
 }
 
 stop() {
     echo -n $"Stopping $prog: "
     killproc $prog -QUIT
     retval=$?
     echo
     [ $retval -eq 0 ] && rm -f $lockfile
     return $retval
 }
 
 restart() {
     configtest || return $?
     stop
     start
 }
 
 reload() {
     configtest || return $?
     echo -n $"Reloading $prog: "
     killproc $nginx -HUP
     RETVAL=$?
     echo
 }
 
 force_reload() {
     restart
 }
 
 configtest() {
   $nginx -t -c $NGINX_CONF_FILE
 }
 
 rh_status() {
     status $prog
 }
 
 rh_status_q() {
     rh_status >/dev/null 2>&1
 }
 
 case "$1" in
     start)
         rh_status_q && exit 0
         $1
         ;;
     stop)
         rh_status_q || exit 0
         $1
         ;;
     restart|configtest)
         $1
         ;;
     reload)
         rh_status_q || exit 7
         $1
         ;;
     force-reload)
         force_reload
         ;;
     status)
         rh_status
         ;;
     condrestart|try-restart)
         rh_status_q || exit 0
             ;;
     *)
         echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
         exit 2
 esac

Optionally, you can obtain the source from:
 wget -O /etc/init.d/nginx https://gist.github.com/sairam/5892520/raw/b8195a71e944d46271c8a49f2717f70bcd04bf1a/etc-init.d-nginx

This file should be made executable so that we can use it via ‘service nginx <command>’:
 chmod +x /etc/init.d/nginx


3. 
Set the service to start whenever the system boots:
 chkconfig --add nginx
 chkconfig --level 345 nginx on


4. 
Configure /etc/nginx/nginx.conf to set types_hash_bucket_size and server_names_hash_bucket_size which needs to be increased.
 http {
     include       mime.types;
     default_type  application/octet-stream;
 	# add the below 2 lines under http around line 20
     types_hash_bucket_size 64;
     server_names_hash_bucket_size 128;


5. 
Start the server. This will start the VPS on port 80.
 service nginx start



## Setup Complete


Visit example.com or your IP address in the browser. You will see:


```
Welcome to nginx!

```


Congratulations! Your brand new nginx server is up.


# Maintenance


Restart nginx server when nginx binary is modified:
service nginx restart


Reload nginx when nginx.conf is modified:
service nginx reload


# Configuring nginx Web Server


# Upgrading to the Latest Version


Let’s get the current nginx version from http://nginx.org/en/download.html


Run the following commands to download the source.


```
ssh root@example.com
cd ~/src/
nginxVersion="1.5.5" # set the value here from nginx website
wget http://nginx.org/download/nginx-$nginxVersion.tar.gz
tar -xzf nginx-$nginxVersion.tar.gz
rm nginx # removes the soft link
ln -sf nginx-$nginxVersion nginx

cd nginx

./configure \
--user=nginx                          \
--group=nginx                         \
--prefix=/etc/nginx                   \
--sbin-path=/usr/sbin/nginx           \
--conf-path=/etc/nginx/nginx.conf     \
--pid-path=/var/run/nginx.pid         \
--lock-path=/var/run/nginx.lock       \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--with-http_gzip_static_module        \
--with-http_stub_status_module        \
--with-http_ssl_module                \
--with-pcre                           \
--with-file-aio                       \
--with-http_realip_module             \
--without-http_scgi_module            \
--without-http_uwsgi_module           \
--without-http_fastcgi_module

make
make install

service nginx restart

```


# More Resources on nginx


Resources about nginx on DigitalOcean


