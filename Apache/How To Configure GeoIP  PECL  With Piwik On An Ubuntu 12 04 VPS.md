# How To Configure GeoIP  PECL  With Piwik On An Ubuntu 12 04 VPS

```Apache``` ```Ubuntu``` ```Monitoring``` ```PHP``` ```Server Optimization```











# Status: Deprecated


This article covers a version of Ubuntu that is no longer supported.  If you are currently operate a server running Ubuntu 12.04, we highly recommend upgrading or migrating to a supported version of Ubuntu:


- Upgrade to Ubuntu 14.04.
- Upgrade from Ubuntu 14.04 to Ubuntu 16.04
- Migrate the server data to a supported version

Reason:
Ubuntu 12.04 reached end of life (EOL) on April 28, 2017 and no longer receives security patches or updates.  This guide is no longer maintained.


See Instead:
This guide might still be useful as a reference, but may not work on other Ubuntu releases.  If available, we strongly recommend using a guide written for the version of Ubuntu you are using.   You can use the search functionality at the top of the page to find a more recent version.


## Introduction


Piwik always offered a way to detect a user's country based on the language they were using. There is no doubt that it is not a reliable and satisfactory way to determine where your users are from; Imagine that someone located in Spain is accessing your website using a computer whose default language is English EN-US, since Piwik guesses a visitor's country based on the language they use, this visit will be detected as a US visit.


A GeoIP database can be used to accurately determine the location of your visitors based on their IP address. A PECL (PHP Extension Community Library) extension will give you a very fast solution, compared to MaxMind's PHP API where the database has to be loaded for each request.


# Assumptions


The steps in this tutorial require the user to have root privileges. You can see how to set that up in the Initial Server Setup Tutorial.


Additionally, this article assumes that you have Piwik installed and is configured in the default root directory of Apache: /var/www/piwik. You can find a Piwik installation tutorial here.


# Step One: Prerequisites


You will need to install build-essential, this package contains an informational list of packages which are considered essential for building Debian packages:


```
sudo apt-get install build-essential
```


You will need to install PEAR via apt-get to get the necessary package and distribution system that both PEAR and PECL use:


```
sudo apt-get install php-pear
```


You will need to install the php5-dev package to get the necessary PHP5 source files to compile additional modules:


```
sudo apt-get install php5-dev
```


# Step Two: Installing GeoIp


Run the following command:


```
sudo apt-get install php5-geoip php5-dev libgeoip-dev
```


Then, run the following command:


```
sudo pecl install geoip
```


# Step Three: Configuring PHP


Once the installation is complete, it will probably tell you that an extension= line cannot be found in your php.ini file. Let's find your php.ini file and add the required lines:


```
sudo nano /etc/php5/apache2/php.ini
```


This will open your php.ini, we have to add "extension=geoip.so" to the [PHP] section:


```
[PHP]
;AFTER THE PHP SECTION NOT BEFORE
extension=geoip.so
```


Now we have to add "geoip.custom_directory=/path/to/piwik/misc" to the [gd] section:


```
[gd]
;AFTER THE gd SECTION NOT BEFORE
geoip.custom_directory=/var/www/piwik/misc
```


# Step Four: Installing and renaming the GeoIP database


Download and extract the GeoIP database:


```
cd /var/www/piwik/misc
sudo wget http://geolite.maxmind.com/download/geoip/database/GeoLiteCity.dat.gz
sudo gunzip GeoLiteCity.dat.gz
```


The PECL extension won't recognize the database if it's named GeoLiteCity.dat so make sure it is named GeoIPCity.dat:


```
sudo mv GeoLiteCity.dat GeoIPCity.dat
```


Restart the Apache Web Server:


```
sudo service apache2 restart
```


# Step Five - Configure Piwik to use GeoIP PECL


Open your browser and login into your Piwik page, go to settings, Geolocation, and choose GeoIP (PECL) as your location provider.


# Step Six Optional - Updating Previous Visits and Updating the GeoIP Database


To re-evaluate all previously tracked visits with the current GeoIP database, input:


```
cd /var/www/piwik/misc/others
sudo php geoipUpdateRows.php
```


This process can take a LONG time if you have many visits to process:


```
1 rows to process in piwik_log_visit and piwik_log_conversion...
0% done...
100% done!
```


Note that the GeoLite database is updated on the first Tuesday of each month, you should update your GeoIP database regularly since providers reallocate IP address ranges very often.


That is all - congratulations!


