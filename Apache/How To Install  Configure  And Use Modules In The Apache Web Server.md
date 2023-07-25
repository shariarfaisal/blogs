# How To Install  Configure  And Use Modules In The Apache Web Server

```Apache``` ```Ubuntu``` ```Server Optimization```











# Status: Deprecated


This article covers a version of Ubuntu that is no longer supported.  If you are currently operate a server running Ubuntu 12.04, we highly recommend upgrading or migrating to a supported version of Ubuntu:


- Upgrade to Ubuntu 14.04.
- Upgrade from Ubuntu 14.04 to Ubuntu 16.04
- Migrate the server data to a supported version

Reason:
Ubuntu 12.04 reached end of life (EOL) on April 28, 2017 and no longer receives security patches or updates.  This guide is no longer maintained.


See Instead:
This guide might still be useful as a reference, but may not work on other Ubuntu releases.  If available, we strongly recommend using a guide written for the version of Ubuntu you are using.   You can use the search functionality at the top of the page to find a more recent version.


## What is Apache?


Apache is the most popular web server in the world.  It is responsible for serving over half of the active sites on the internet and can handle the needs of both large and small projects.


In this guide, we will be covering some common, useful modules that can add functionality and improve your experience when working with Apache.  They can help you optimize, secure, and monitor your server.


We will be using an Ubuntu 12.04 VPS to explore these modules, but most distributions should operate in a similar way.  Consult your distribution's documentation for Apache-specific file locations.


# PageSpeed Module


The mod_pagespeed module is an Apache enhancement that optimizes your content automatically.  It can compress data, implement caching, resize files, and remove unnecessary whitespace from configuration files.


There are binaries on the project's webpage for Ubuntu.  To download and install on a 64-bit Ubuntu system, type the following:


```
cd
wget https://dl-ssl.google.com/dl/linux/direct/mod-pagespeed-stable_current_amd64.deb
sudo dpkg -i mod-pagespeed-*.deb
sudo apt-get -f install
sudo service apache2 reload
```


On a 32-bit Ubuntu system, type:


```
cd
wget https://dl-ssl.google.com/dl/linux/direct/mod-pagespeed-stable_current_i386.deb
sudo dpkg -i mod-pagespeed-*.deb
sudo apt-get -f install
sudo service apache2 reload
```


The configuration file is located at "/etc/apache2/mods-available/pagespeed.conf".


This module is enabled when it is installed and should begin optimizing content when you reload the server, but you can configure many different optimization and monitoring functions from within the configuration file.  Follow our guide on how to configure mod_pagespeed on Ubuntu or Debian.


# Security Module


The mod_security module provides a configurable security layer that can accept or deny traffic based upon rules set by the administrator.  It is an application firewall that can prevent exposing vulnerabilities to the internet.


This module is in Ubuntu's default repositories, so it can be installed with the following command:


```
sudo apt-get install libapache2-modsecurity
```


You can enable the module with this command:


```
sudo a2enmod mod-security
```


The configuration file in the normal "/etc/apache2/mods-available" directory is called "mod-security.conf", but this only references the files in "/etct/modsecurity".


We can move the default example file into production with the following commands:


```
cd /etc/modsecurity
sudo cp modsecurity.conf-recommended modsecurity.conf
```


```
Open the configuration file with root privileges:

```


```
sudo nano modsecurity.conf
```


Read the configuration file and adjust the values based on the needs of your site.  Most of the default configuration settings are okay.  You might want to adjust the "SecRequestBodyLimit" to something a bit more permissive than the default 128 KB limit.


When you are ready to apply the settings, you can change the "SecRuleEngine" rule to read "On" instead of "DetectionOnly":


```
#SecRuleEngine DetectionOnly
SecRuleEngine On
```


This will implement your rules and begin applying them to your sites.  You will need to reload your Apache instance for these rules to take affect:


```
sudo service apache2 reload
```


# Status Module


One of the most helpful and easiest modules to configure comes pre-installed and configured when you install Apache on Ubuntu.  The mod_status module provides an overview of your server load and requests.


You can edit the configuration file in the "mods-available" directory with the following command:


```
sudo nano /etc/apache2/mods-available/status.conf
```


Under the "Location /server-status" directive, remove the "#" character from before the "192.0.2.0/24" line and add the IP address of the computer you will be using to access your web server:


```
<Location /server-status>
	SetHandler server-status
	Order deny,allow
	Deny from all
	Allow from 127.0.0.1 ::1
	Allow from Your.IP.Address.Here
</Location>
```


Once again, be sure that the IP address you input is the computer you are using to access the server, and not the server's IP address.


Reload Apache so that it can re-read the new configuration changes:


```
sudo service apache2 reload
```


Navigate to the server-status page you have defined by typing the following into your web browser:


```
Server.IP.Address.Or.Domain.Name/server-status
```


```
Apache Server Status for 192.241.167.189

Server Version: Apache/2.2.22 (Ubuntu)
Server Built: Jul 12 2013 13:37:15
Current Time: Thursday, 08-Aug-2013 16:36:48 UTC
Restart Time: Thursday, 08-Aug-2013 16:04:59 UTC
Parent Server Generation: 2
Server uptime: 31 minutes 49 seconds
Total accesses: 3 - Total Traffic: 0 kB
CPU Usage: u0 s0 cu0 cs0
. . .
```


You will be given a stats page that will give you information and text-based indications of your server's performance and load.  Rapidly refreshing the page will allow you to see how activity is shown.


# Spamhaus Module


The Spamhaus module enables you to block attackers by denying request from a blacklist of IP addresses that are known to be bad.


Once again, this module is in Ubuntu's default repositories.  Install is with the following command:


```
sudo apt-get install libapache2-mod-spamhaus
```


To configure, look at the "mod-spamhaus.conf" file within the "mods-available" directory:


```
sudo nano /etc/apache2/mods-available/mod_spamhaus.conf
```


You can configure the module to filter based on a number of different criteria.  With the "MS_METHODS" definition, you can check the IP whenever the client uses any of the HTTP methods recognized.  This can prevent those IPs from flooding the server.


The module also allows you to configure a whitelist, maintain a local version of the DNS blacklist, and adjust cache parameters.


The module should have been enabled on installation, but we can double-check and reload apache to enable the filtering with these commands:


```
sudo a2enmod mod-spamhaus
sudo service apache2 reload
```


# Rewrite Module


One of the most useful modules for Apache is mod_rewrite.  This module allows you to generate unique and easily readable urls for content requested on the server.


The module is installed on Ubuntu by default when Apache is installed, but it is not enabled.  To rectify this situation, issue the following command:


```
sudo a2enmod rewrite
sudo service apache2 reload
```


Configuration for mod_rewrite doesn't take place in the "mods-available" directory.  Instead, it uses .htaccess files or declarations within normal server configuration files to decide what to do.


The full configuration details are outside of the scope of this article.  However, we do have a detailed article on how to set up mod_rewrite here.


# Conclusion


This is by no means an exhaustive list of modules for Apache, but merely an introduction.  By now, you should be able to see some of the variety of modules available to modify the standard Apache behavior.


Keep in mind that with every piece of code that you add to a server, there exists the possibility that you are opening up vulnerabilities and creating more overhead.  Try to choose modules that are well-tested and commonly implemented.  Only enable the modules that are actively needed for your sites.


By Justin Ellingwood
