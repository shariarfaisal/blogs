# How To Install and Setup Elgg on a Debian or Ubuntu VPS

```Ubuntu``` ```Applications``` ```PHP``` ```Debian```


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


Elgg is an award winning PHP engine for running your own full fledged social network. It has great community support in the form of plugins and themes made by official developers and the user community.


## Requirements


1. 
Debian 7 or Ubuntu 12.04

2. 
Apache (with rewrite module)

3. 
PHP 5.2+ (few modules required to support some of the features)

4. 
MySQL 5+


## Installation


It is a good idea to do a total system update before doing anything else.


```
apt-get update
apt-get upgrade

```


# Installing web and database server


Install apache and mysql first by doing


```
apt-get install apache2 mysql-server

```


During the mysql-server installation, you have to input a password for the root user. Using a strong password and noting it down is a good idea, as you will have to use it to create a database and a user for Elgg later.


## Configuring apache


After installing apache, the rewrite module that Elgg requires should be enabled by running:


```
a2enmod rewrite

```


The default apache configuration ignores .htaccess files in the “document root” (/var/www by default). But this is required for url rewriting in Elgg to work.


Change the apache configuration by opening it in nano, vim, or any editor you like.


```
nano /etc/apache2/sites-available/default

```


In this file, replace AllowOverride None with AllowOverride All in the Directory section defined for /var/www folder.


After the configuration file is edited and saved, do an apache server restart:


```
service apache2 restart

```


## Creating MySQL user and database


It is a good idea to create a new database user instead of using root for Elgg’s purpose. Get into the MySQL command line by running:


```
mysql -u root -p

```


Execute the following queries to create a new database, a user for Elgg, and grant the database access to this user.


```
CREATE DATABASE elgg;
CREATE USER elgguser IDENTIFIED BY 'elggpassword';
GRANT ALL ON elgg.* TO elgguser;

```


## Installing PHP


The following command installs php along with few modules Elgg needs for some of it’s features to work.


```
apt-get install php5 php5-gd php-xml-parser php5-mysql unzip

```


## Downloading Elgg


Go to your webserver’s root directory and download Elgg’s source code.


```
cd /var/www/
wget http://elgg.org/getelgg.php?forward=elgg-1.8.18.zip -O elgg.zip

```


Unzip and remove elgg.zip and unwrap the folder enclosing the code.


```
unzip elgg.zip && rm elgg.zip
mv elgg-1.8.18/* . && rmdir elgg-1.8.18

```


# Configuring Elgg for Install


## Setting up data directory


Elgg uses a data directory to store user uploaded and system generated data. Create a folder outside the webserver’s root directory and make it writeable by the webserver.


```
mkdir <span style='color: red'>/var/elggdata</span>
chown -R www-data:www-data /var/elggdata

```


www-data is apache’s user that writes to file system. The above command grants apache ownership of the data directory.


## .htaccess and settings.php


Now move the .htaccess file and the settings.php files to their places.


```
mv /var/www/htaccess_dist /var/www/.htaccess
mv /var/www/engine/settings.example.php /var/www/engine/settings.php

```


## Editing settings.php


Open up the settings.php file and fill in the database access details.


```
nano /var/www/engine/settings.php

```


Make sure to set the right values to the following variables in the file.


```
$CONFIG->dbuser = 'elgguser';

$CONFIG->dbpass = 'elggpassword';

$CONFIG->dbname = 'elgg';

$CONFIG->dbhost = 'localhost';

$CONFIG->dbprefix = ''; // or anything else you like

```


# Installing Elgg


If you followed all the steps in this article, the install will be a breeze for you. Open up the browser and navigate to /install.php of your website(‘example.com’, etc.) or your ip address (‘128.199.246.37’, etc.).


```
http://example.com/install.php
or
http://128.199.246.37/install.php

```


That’s it! You should see the installation page with instructions about various steps involved in the process. Elgg will create the database and ask for few site specific details and even creates the admin user for you in the subsequent steps.


## Getting Help


Elgg has great community support. If you are stuck anywhere just head over to the community site. There are quite a few people ready to help you with anything you may need.


<div class=“author”>Article Submitted by: <a href=“http://serenecode.org/”>GP</a></div>


