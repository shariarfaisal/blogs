# How To Install nginx on CentOS 6 with yum

```Nginx``` ```CentOS```


Status: Deprecated
This article covers a version of CentOS that is no longer supported. If you are currently operating a server running CentOS 6, we highly recommend upgrading or migrating to a supported version of CentOS.
Reason:
CentOS 6 reached end of life (EOL) on November 30th, 2020 and no longer receives security patches or updates. For this reason, this guide is no longer maintained.
See Instead:
This guide might still be useful as a reference, but may not work on other CentOS releases. If available, we strongly recommend using a guide written for the version of CentOS you are using.
The following DigitalOcean tutorial may be of immediate interest, as it outlines installing Nginx on a CentOS 7 server:


How To Install Nginx on CentOS 7


## About Nginx


nginx is a high performance web server software. It is a much more flexible and lightweight program than apache.


# Set Up


The  steps in this tutorial require the user to have root privileges. You can see how to set that up in the  CentOS Initial Server Setup Tutorial  in steps 3 and 4.


# Step One—Install EPEL


EPEL stands for Extra Packages for Enterprise Linux. Because yum as a package manager does not include the latest version of nginx in its default repository, installing  EPEL will make sure that nginx on CentOS stays up to date. 


To install EPEL, open terminal and type in:


```
sudo yum install epel-release
```


# Step Two—Install nginx


To install nginx, open terminal and type in:


```
sudo yum install nginx
```


After you answer yes to the prompt twice (the first time relates to importing the EPEL gpg-key), nginx will finish installing on your virtual private server. 


# Step Three—Start nginx


nginx does not start on its own. To get nginx running, type:


```
sudo /etc/init.d/nginx start
```


You can confirm that nginx has installed on your VPS by directing your browser to your IP address. 


You can run the following command to reveal your server’s IP address.


```
ifconfig eth0 | grep inet | awk '{ print $2 }'
```


On the page, you will see the words, “Welcome to nginx”


Congratulations! You have now installed nginx.


# See More


Once you have nginx installed on your cloud server, you can go on to install the Lemp Stack or Set Up a FTP Server


By Etel Sverdlov
