# How To Set Up vsftpd on CentOS 6

```Security``` ```CentOS```










# Status: Deprecated


This article covers a version of CentOS that is no longer supported. If you are currently operating a server running CentOS 6, we highly recommend upgrading or migrating to a supported version of CentOS.


Reason:
CentOS 6 reached end of life (EOL) on November 30th, 2020 and no longer receives security patches or updates. For this reason, this guide is no longer maintained.


See Instead:
This guide might still be useful as a reference, but may not work on other CentOS releases. If available, we strongly recommend using a guide written for the version of CentOS you are using.


## About vsftpd


Warning: FTP is inherently insecure.  If you must use FTP, consider securing your FTP connection with SSL/TLS.  Otherwise, it is best to use SFTP, a secure alternative to FTP.


The first two letters of vsftpd stand for "very secure" and the program was built to have strongest protection against possible FTP vulnerabilities.


# Step One—Install vsftpd


You can quickly install vsftpd on your virtual private server in the command line:


```
sudo yum install vsftpd
```


We also need to install the FTP client, so that we can connect to an FTP server:


```
sudo yum install ftp
```


Once the files finish downloading, vsftpd will be on your VPS. Generally speaking, the virtual private server is already configured with a reasonable amount of security. However, it does provide access to anonymous users.


# Step Two—Configure VSFTP


Once VSFTP is installed, you can adjust the configuration.


Open up the configuration file:


```
sudo vi /etc/vsftpd/vsftpd.conf
```


One primary change you need to make is to change the Anonymous_enable to No:


```
anonymous_enable=NO
```


Prior to this change, vsftpd allowed anonymous, unidentified users to access the VPS's files. This is useful if you are seeking to distribute information widely, but may be considered a serious security issue in most other cases. 
After that, uncomment the local_enable option, changing it to yes.



```
local_enable=YES
```


Finish up by uncommenting command to chroot_local_user. When this line is set to Yes, all the local users will be jailed within their chroot and will be denied access to any other part of the server.


```
chroot_local_user=YES
```


Finish up by restarting vsftpd:


```
sudo service vsftpd restart
```


In order to ensure that vsftpd runs at boot, run chkconfig:


```
chkconfig vsftpd on
```


# Step Three—Access the FTP server


Once you have installed the FTP server and configured it to your liking, you can now access it.


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
