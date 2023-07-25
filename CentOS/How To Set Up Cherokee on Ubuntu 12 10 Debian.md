# How To Set Up Cherokee on Ubuntu 12 10 Debian

```Ubuntu``` ```Debian``` ```CentOS```

## About Cherokee


Cherokee is a high performance, lightweight, full-featured web server. It is compatible with SSL, FastCGI, and all modern web application frameworks like NodeJS, Rails, and Python through uWSGI. The best thing about Cherokee is that unlike Apache or Nginx, Cherokee can be administered completely through it's admin web interface.


# Setup On Ubuntu


On Ubuntu droplets, Cherokee can be installed after the Cherokee PPA has been configured.


```
sudo add-apt-repository ppa:cherokee-webserver/ppa
```


If your system does not come with add-apt-repository, try installing software-properties-common first.


```
sudo apt-get install software-properties-common
```


Afterwards, update the apt cache and install cherokee and cherokee-admin


```
sudo apt-get update
sudo apt-get install cherokee cherokee-admin
```


# Setup On Debian and Ubuntu


On Debian or Ubuntu droplets, Cherokee can be installed directly from the apt repository.


```
sudo apt-get install cherokee cherokee-admin
```


# Setup On CentOS


On CentOS droplets, Cherokee can be installed directly from the EPEL repository.


```
sudo yum install cherokee
```


# Check Cherokee Status


Now that Cherokee is installed, we can check whether it is running.


```
sudo service cherokee status
```


If Cherokee is ever having troubles, we can check its logs in the /var/log/cherokee directory. We can also verify that Cherokee is running by visiting the droplet's IP address. We should be greeted by this page.





# Administering Cherokee


The best part about using Cherokee is being able to manage all of its configurations through a simple to use web interface. The web management interface does not and should not be running by default. It can be started through the cherokee-admin command.


```
sudo cherokee-admin
```


Cherokee-admin will output the temporary credentials to use in the web interface. Copy the one time password generated.


```
Cherokee Web Server 1.2.101 (Oct 25 2012): Listening on port 127.0.0.1:9090,
TLS disabled, IPv6 enabled, using epoll, 4096 fds system limit, max. 2041
connections, caching I/O, single thread

Login:
  User:              admin
  One-time Password: tGCtsC95wdbwtBCC

Web Interface:
  URL:               http://127.0.0.1:9090/
```


Since the administration page is bound by default only to the server's local interface, we should start another SSH connection and forward port 9090 to our local machine. This allows us to access the web interface on our local machine at port 9090.


```
ssh USER_NAME@DROPLET_IP_ADDRESS -L9090:localhost:9090
```


Now if we visit localhost:9090, we will be prompted for the admin password. After logging in, we can use the Cherokee admin interface to manage all aspects of our web server, such as setting up vServers, WSGI sources, tuning, etc...





