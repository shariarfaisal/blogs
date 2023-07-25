# How To Install DenyHosts on Ubuntu 12 04

```Security``` ```Ubuntu``` ```Monitoring```











# Status: Deprecated


This article covers a version of Ubuntu that is no longer supported.  If you are currently operate a server running Ubuntu 12.04, we highly recommend upgrading or migrating to a supported version of Ubuntu:


- Upgrade to Ubuntu 14.04.
- Upgrade from Ubuntu 14.04 to Ubuntu 16.04
- Migrate the server data to a supported version

Reason:
Ubuntu 12.04 reached end of life (EOL) on April 28, 2017 and no longer receives security patches or updates.  This guide is no longer maintained.


See Instead:
This guide might still be useful as a reference, but may not work on other Ubuntu releases.  If available, we strongly recommend using a guide written for the version of Ubuntu you are using.   You can use the search functionality at the top of the page to find a more recent version.


## About DenyHosts


DenyHosts is a security tool written in python that monitors server access logs to prevent brute force attacks on a virtual private server. The program works by banning IP addresses that exceed a certain number of failed login attempts.


# Step One—Install Deny Hosts


DenyHosts is very easy to install on Ubuntu


```
 sudo apt-get install denyhosts
```


Once the program has finished downloading, denyhosts is installed and configured on your virtual private server.


# Step Two—Whitelist IP Addresses


After you install DenyHosts, make sure to whitelist your own IP address. Skipping this step will put you at risk of locking yourself out of your own machine.


Open up the list of allowed hosts allowed on your VPS:


```
sudo nano /etc/hosts.allow
```


Under the description, add in any IP addresses that cannot afford to be banned from the server; you can write each one on a separate line, using this format:


```
sshd: 12.34.45.678
```


After making any changes, be sure to restart DenyHosts so that the new settings take effect on your virtual private server:


```
sudo /etc/init.d/denyhosts restart
```


# Step Three—(Optional) Configure DenyHosts


DenyHosts is ready use as soon as the installation is over.


However if you want to customize the behavior of DenyHosts on your VPS, you can make the changes within the DenyHost configuration file:


```
sudo nano /etc/denyhosts.conf
```


By Etel Sverdlov
