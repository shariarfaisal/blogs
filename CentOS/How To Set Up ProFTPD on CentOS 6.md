# How To Set Up ProFTPD on CentOS 6

```System Tools``` ```CentOS```










# Status: Deprecated


This article covers a version of CentOS that is no longer supported. If you are currently operating a server running CentOS 6, we highly recommend upgrading or migrating to a supported version of CentOS.


Reason:
CentOS 6 reached end of life (EOL) on November 30th, 2020 and no longer receives security patches or updates. For this reason, this guide is no longer maintained.


See Instead:
This guide might still be useful as a reference, but may not work on other CentOS releases. If available, we strongly recommend using a guide written for the version of CentOS you are using.


## About ProFTPD


ProFTPD is a popular ftp server. Because it was written as a  powerful and configurable program, it is not necessarily the lightest ftp server available.


# Step One—Install ProFTPD


Before we do anything else, we need to download the EPEL repository which will allow us to install ProFTPD on our virtual private server with yum.


```
sudo rpm -Uvh http://download.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm
```


The next step is to install ProFTPD


```
sudo yum install proftpd
```


Finally, we must also download a ftp client, so that we can connect to an ftp server from the command line:


```
sudo yum install ftp
```


Once the files finish downloading, the ProFTPD server will be on your VPS. However, we still have to make a few changes to the configuration.


# Step Two—Configure ProFTPD


Once ProFTPD is installed, you can make the needed adjustments in the configuration. Unlike some other ftp configurations, ProFTPD disables anonymous login from the outset and we only need to address a small change in the config file.


Open up the file:


```
sudo vi /etc/proftpd.conf
```


Go ahead and change the Server Name to your host name.


```
ServerName                      "example.com"
```


Save and Exit from that file.


Then, to prevent any issues, add your droplet name and IP address to the hosts file:


```
sudo vi /etc/hosts
```


The line can look something like this:


```
12.34.56.789 servername
```


Restart after you have  made all of your changes:


```
sudo service proftpd restart
```


# Step Three—Access the FTP server


<p?Once you have installed the FTP server and configured it to your liking, you can now access it.





You can reach an FTP server in the browser by typing the domain name into the address bar and logging in with the appropriate ID. Keep in mind, you will only be able to access the user's home directory.


```
ftp://example.com
```


Alternatively, you can reach the FTP server through the command line by typing:


```
 ftp example.com
```


Then you can use the word, "exit," to get out of the FTP shell.


By Etel Sverdlov
