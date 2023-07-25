# How To Install Nginx on CentOS 7

```Nginx``` ```CentOS```

## Introduction


Nginx is a popular high-performance web server. This tutorial will teach you how to install and start Nginx on your CentOS 7 server.


# Prerequisites


The steps in this tutorial require a non-root user with sudo privileges. See our Initial Server Setup with CentOS 7 tutorial to learn how to set up this user.


# Step 1 — Adding the EPEL Software Repository


To add the CentOS 7 EPEL repository, first connect to your CentOS 7 machine via SSH, then use the yum command to install the extended package repository:


```
sudo yum install epel-release


```


You’ll be prompted to verify that you want to install the software. Type y then ENTER to continue.


Next, you’ll install the actual nginx software package.


# Step 2 — Installing Nginx


Now that the EPEL repository is installed on your server, install Nginx using the following yum command:


```
sudo yum install nginx


```


Again, answer yes to the verification prompt, then Nginx will finish installing.


# Step 3 — Starting Nginx


Nginx will not start automatically after it is installed. To get Nginx running, use the systemctl command:


```
sudo systemctl start nginx


```


You can check the status of the service with systemctl status:


```
sudo systemctl status nginx


```


```
Output● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: active (running) since Mon 2022-01-24 20:14:24 UTC; 5s ago
  Process: 1898 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
  Process: 1896 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
  Process: 1895 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
 Main PID: 1900 (nginx)
   CGroup: /system.slice/nginx.service
           ├─1900 nginx: master process /usr/sbin/nginx
           └─1901 nginx: worker process

Jan 24 20:14:24 centos-updates systemd[1]: Starting The nginx HTTP and reverse proxy server...
Jan 24 20:14:24 centos-updates nginx[1896]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Jan 24 20:14:24 centos-updates nginx[1896]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Jan 24 20:14:24 centos-updates systemd[1]: Started The nginx HTTP and reverse proxy server.

```


The service should be active.


If you are running a firewall, run the following commands to allow HTTP and HTTPS traffic:


```
sudo firewall-cmd --permanent --zone=public --add-service=http
sudo firewall-cmd --permanent --zone=public --add-service=https
sudo firewall-cmd --reload


```


You can do a spot check right away to verify that everything went as planned by visiting your server’s public IP address in your web browser:


```
http://server_domain_name_or_IP/

```


You will see the default CentOS 7 Nginx web page, which is there for informational and testing purposes. It should look something like this:





If you see this page, then your web server is now correctly installed.



Note: To find your server’s public IP address, find the network interfaces on your machine by typing:
ip addr


Output1. lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN

. . .
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000

. . .

You may see a number of interfaces here depending on the hardware available on your server. The lo interface is the local loopback interface, which is not the one we want. In our example above, the eth0 interface is what we want.
Once you have the interface name, you can run the following command to reveal your server’s public IP address. Substitute the interface name you found above:
ip addr show eth0 | grep inet | awk '{ print $2; }' | sed 's/\/.*$//'



Before continuing, you will probably want to enable Nginx to start when your system boots. To do so, enter the following command:


```
sudo systemctl enable nginx


```


Nginx is now installed and running.


# Step 4 — Exploring and Configuring Nginx


If you want to start serving your own pages or application through Nginx, you will want to know the locations of the Nginx configuration files and default server root directory.


## Default Server Root


The default server root directory is /usr/share/nginx/html. Files that are placed in there will be served on your web server. This location is specified in the default server block configuration file that ships with Nginx, which is located at /etc/nginx/conf.d/default.conf.


## Server Block Configuration


Any additional server blocks, known as Virtual Hosts in Apache, can be added by creating new configuration files in /etc/nginx/conf.d. Files that end with .conf in that directory will be loaded when Nginx is started.


## Nginx Global Configuration


The main Nginx configuration file is located at /etc/nginx/nginx.conf. This is where you can change settings like the user that runs the Nginx daemon processes, and the number of worker processes that get spawned when Nginx is running, among other things.


# Conclusion


Once you have Nginx installed on your CentOS 7 server, you can go on to install the full LEMP Stack on CentOS 7.


