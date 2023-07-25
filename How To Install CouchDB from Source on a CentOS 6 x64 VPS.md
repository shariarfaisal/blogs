# How To Install CouchDB from Source on a CentOS 6 x64 VPS

```NoSQL``` ```CentOS```


Status: Deprecated
This article covers a version of CentOS that is no longer supported. If you are currently operating a server running CentOS 6, we highly recommend upgrading or migrating to a supported version of CentOS.
Reason:
CentOS 6 reached end of life (EOL) on November 30th, 2020 and no longer receives security patches or updates. For this reason, this guide is no longer maintained.
See Instead:
This guide might still be useful as a reference, but may not work on other CentOS releases. If available, we strongly recommend using a guide written for the version of CentOS you are using.

## Introduction



CouchDB is a NoSQL database developed by The Apache Software Foundation that uses JSON for documents, JavaScript for MapReduce queries, and regular HTTP for an API. Often called “a database that completely embraces the web,” it’s used by many startups as well as corporations due to its flexibility and scalability.


As of this tutorial, the current stable version of CouchDB is 1.4.0.


It’s recommended to complete the Initial Server Setup with CentOS 6 tutorial before starting this one.


# Step 1 - Install the Build Tools on your VPS



In order to compile CouchDB from source, you need to install some tools and dependencies on your virtual server.


The first thing you need to do is update your packages to the latest version:


```
sudo yum -y update

```


Next, you have to install the Development Tools:


```
sudo yum -y groupinstall "Development Tools"

```


And the dependencies required to compile CouchDB: Erlang and SpiderMoney:


```
sudo yum -y install libicu-devel curl-devel ncurses-devel libtool libxslt fop java-1.6.0-openjdk java-1.6.0-openjdk-devel unixODBC unixODBC-devel openssl-devel

```


# Step 2 - Installing Erlang



Erlang is required by CouchDB. The CentOS team is not offering any official packages, so you will have to compile it from source.


First, go to www.erlang.org/download.html and download the latest source code.


```
wget http://www.erlang.org/download/otp_src_R16B02.tar.gz

```


After your download is finished, unpack the archive:


```
tar -zxvf otp_src_R16B02.tar.gz

```


Now that we have the Erlang source code unpacked, we can start compiling it:


```
cd otp_src_R16B02
./configure && make

```


Next you’ll have to install it. By default, Erlang will be installed in /usr/local:


```
sudo make install

```


# Step 3 - Installing the SpiderMonkey JS Engine



Mozilla’s SpiderMoney JavaScript Engine is required by CouchDB in order to successfully compile.


CouchDB requires Mozilla’s SpiderMoney version 1.8.5, which you can download from their FTP:


```
wget http://ftp.mozilla.org/pub/mozilla.org/js/js185-1.0.0.tar.gz

```


After the download is finished, unpack the archive:


```
tar -zxvf js185-1.0.0.tar.gz 

```


The next step is to compile and install it on your VPS:


```
cd js-1.8.5/js/src
./configure && make
sudo make install

```


# Step 4 - Installing CouchDB



After all dependencies are satisfied, installing CouchDB is pretty straight forward.


First, you’ll have to download and unpack the CouchDB source:


```
wget http://apache.osuosl.org/couchdb/source/1.4.0/apache-couchdb-1.4.0.tar.gz
tar -zxvf apache-couchdb-1.4.0.tar.gz

```


After we have the source code unpacked, we can start compiling it. This should take just a few minutes:


```
cd apache-couchdb-1.4.0
./configure && make

```


If everything is fine, we are now ready to install CouchDB:


```
sudo make install

```


# Step 5 - Setting up CouchDB



After CouchDB is installed, you have to create the CouchDB user, set the proper permissions and add the startup scripts.


Let’s start by adding the couchdb user:


```
sudo adduser --no-create-home couchdb

```


The couchdb user must to have the proper permissions to access a few directories:


```
sudo chown -R couchdb:couchdb /usr/local/var/lib/couchdb /usr/local/var/log/couchdb /usr/local/var/run/couchdb

```


Next, we’ll have to create a link for the couchdb init script to /etc/init.d:


```
sudo ln -sf /usr/local/etc/rc.d/couchdb /etc/init.d/couchdb

```


If you’d like CouchDB to start automatically at boot, add and enable the init script in chkconfig:


```
sudo chkconfig --add couchdb
sudo chkconfig couchdb on

```


By default, CouchDB can be accessed only from the VPS itself. If you’d like to access it from the web, you’ll have to change the configuration file.


Open the configuration file in an editor:


```
sudo nano /usr/local/etc/couchdb/local.ini

```


Should you need to access couchdb from the web, in the [httpd] section, look for a setting called bind_address and change it to 0.0.0.0 - this will make CouchDB bind all available addresses.


```
[httpd]
port = 5984
bind_address = 0.0.0.0

```


Now we’re ready to start CouchDB:


```
sudo service couchdb start

```


To verify that CouchDB is running, connect to it on port 5984:


```
curl http://localhost:5984

```


You should see a response like:


```
{"couchdb":"Welcome","uuid":"a9e7db070cfe85e6a770aa254c49c8c3","version":"1.4.0","vendor":{"name":"The Apache Software Foundation","version":"1.4.0"}}

```


After you confirm that your server is up and running, you can access it in a browser at http://your.DO.IP.address:5984/\_utils.


<div class=“author”>Submitted by: <a href=“http://www.liviudm.com”>Liviu Damian </a></div>


