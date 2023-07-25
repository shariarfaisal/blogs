# How To Install DenyHosts on CentOS 6

```Security``` ```Monitoring``` ```CentOS```










# Status: Deprecated


This article covers a version of CentOS that is no longer supported. If you are currently operating a server running CentOS 6, we highly recommend upgrading or migrating to a supported version of CentOS.


Reason:
CentOS 6 reached end of life (EOL) on November 30th, 2020 and no longer receives security patches or updates. For this reason, this guide is no longer maintained.


See Instead:
This guide might still be useful as a reference, but may not work on other CentOS releases. If available, we strongly recommend using a guide written for the version of CentOS you are using.


## About DenyHosts


DenyHosts is a security tool written in python that monitors server access logs to prevent brute force attacks on a virtual server. The program works by banning IP addresses that exceed a certain number of failed login attempts.


# Step One—Install Deny Hosts


We need to use a repository to install Deny Hosts on CentOS.


```
sudo rpm -Uvh http://mirror.metrocast.net/fedora/epel/6/i386/epel-release-6-8.noarch.rpm
```


```
sudo yum install denyhosts
```


Once the program has finished downloading to the VPS, denyhosts is installed and configured.


# Step Two—Whitelist IP Addresses


After you install DenyHosts, make sure to whitelist your own IP address. Skipping this step will put you at risk of locking yourself out of your own virtual private server.


Open up the list of allowed hosts:


```
nano /etc/hosts.allow
```


Under the description, add in any IP addresses that cannot afford to be banned from the server; you can write each one on a separate line, using this format:


```
sshd: 12.34.45.678
```


After making any changes, be sure to restart DenyHosts so that the new settings take effect on your virtual server:


```
/etc/init.d/denyhosts restart
```


# Step Three—(Optional) Configure DenyHosts


DenyHosts is ready use as soon as the installation is over.


However if you want to customize the behavior of DenyHosts on your server, you can make the changes within the DenyHost configuration file:


```
nano /etc/denyhosts.conf
```


By Etel Sverdlov
