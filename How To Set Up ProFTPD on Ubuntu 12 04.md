# How To Set Up ProFTPD on Ubuntu 12 04

```Linux Basics```











# Status: Deprecated


This article covers a version of Ubuntu that is no longer supported.  If you are currently operate a server running Ubuntu 12.04, we highly recommend upgrading or migrating to a supported version of Ubuntu:


- Upgrade to Ubuntu 14.04.
- Upgrade from Ubuntu 14.04 to Ubuntu 16.04
- Migrate the server data to a supported version

Reason:
Ubuntu 12.04 reached end of life (EOL) on April 28, 2017 and no longer receives security patches or updates.  This guide is no longer maintained.


See Instead:
This guide might still be useful as a reference, but may not work on other Ubuntu releases.  If available, we strongly recommend using a guide written for the version of Ubuntu you are using.   You can use the search functionality at the top of the page to find a more recent version.


## About ProFTP


ProFTPD is a popular ftp server. Because it was written as a powerful and configurable program, it is not necessarily the lightest ftp server available for virtual servers.


Warning: FTP is inherently insecure!  Consider configuring ProFTPd to use SFTP, a secure alternative to FTP implemented under SSH.


# Step One—Install ProFTP


You can quickly install ProFTP on your VPS in the command line:


```
sudo apt-get install proftpd
```


While the file is installing, you will be given the choice to run your VPS as an inetd or standalone server. Choose the standalone option.


Once the file finishes downloading, the ProFTPD server will be on your droplet. However, we still have to make a few changes to the configuration.


# Step Two—Configure ProFTP


Once ProFTPD is installed, you can make the needed adjustments in the configuration. Unlike some other FTP configurations, ProFTPD disables anonymous login from the outset and we only need to make a couple of alterations in the config file.


Open up the file:


```
sudo nano /etc/proftpd/proftpd.conf
```


Go ahead and make a few changes:


- Change the Server Name to your host name
- Uncomment the line that says Default Root. Doing so will limit users to their home directory.

Once you have finished those adjustments, you can save and exit.


Restart after you have  made all of your changes:


```
sudo service proftpd restart
```


# Step Three—Access the FTP server


Once you have installed the FTP server and configured it to your liking, you can now access it.


You can reach an FTP server in the browser by typing the domain name into the address bar and logging in with the appropriate ID. Keep in mind, you will only be able to access the user's home directory when connecting to the virtual server.


```
ftp://example.com
```


Alternatively, you can reach the FTP server through the command line by typing:


```
 ftp example.com
```


Then you can use the word, "exit," to get out of the FTP shell.


By Etel Sverdlov
