# How To Install Zend Server 6 on a CentOS 6 4 VPS

```PHP Frameworks``` ```PHP``` ```CentOS```










# Status: Deprecated


This article covers a version of CentOS that is no longer supported. If you are currently operating a server running CentOS 6, we highly recommend upgrading or migrating to a supported version of CentOS.


Reason:
CentOS 6 reached end of life (EOL) on November 30th, 2020 and no longer receives security patches or updates. For this reason, this guide is no longer maintained.


See Instead:
This guide might still be useful as a reference, but may not work on other CentOS releases. If available, we strongly recommend using a guide written for the version of CentOS you are using.


## What the Red  Means


The lines that the user needs to enter or customize will be in red in this tutorial!


The rest should mostly be copy-and-pastable.


## About Zend Server 6


Zend Server 6 is the latest (as of this writing) production ready
server management tool from Zend for PHP. It offers many convenient
ways to manage your applications built using PHP. From the
administration panel you can view logs, configure PHP, view server
information and much more. There are many management tools available
for advanced users with the Enterprise license that can help with
managing multiple servers and much more. Best of all, the Community
Edition is free for everyone, even on a production server.


# Prerequisites


The following prerequisites have been made for the course of this
tutorial:


- A VPS running with either CentOS 6.4 x64 or x32.
- You know how to SSH into your cloud server as root or some other user
    which has the ability to install packages.
- You want to install Zend Server 6 with PHP version 5.3 or 5.4.
- You are familiar with vi for editing files.

# Step 1 - Prepare Yum


Add a repo file for the Zend repository so yum can find the
package to install:


```
vi /etc/yum.repos.d/zend.repo
```


Your file should be open in vi for editing (press i to go into
edit mode). Enter the following details for the Zend repository:


```
[Zend]
name=Zend Server 
baseurl=http://repos.zend.com/zend-server/6.0/rpm/$basearch
enabled=1
gpgcheck=1
gpgkey=http://repos.zend.com/zend.key


[Zend_noarch]
name=Zend Server - noarch
baseurl=http://repos.zend.com/zend-server/6.0/rpm/noarch
enabled=1
gpgcheck=1
gpgkey=http://repos.zend.com/zend.key

```


Now save the file by pressing the escape key followed by :wq
to write the file and close vi.


The repository file describes how you get the package for
installation.


 The name section is arbitrary and is used for
describing this repo. 


The baseurl is the location yum will
look when searching for the desired package. 


The $basesearch
variable is used by yum to find the right package for the system it is
installing on your virtual private server.


The enabled flag determines if
it is enabled for yum to use. 






The gpgcheck flag tells yum to
check the signature of the file it downloads against the gpgkey
supplied by the vendor.


The _noarch section provides the same
information for any data which is platform or architecture
independent, such as graphics or documentation.


# Step 2 - Install Zend Server


Now that yum can find the repository for Zend Server, you can
install the package of your choice. Zend Server 6 can be installed
with your choice of PHP version 5.3 or 5.4. Both commands will be
listed.


To install PHP version 5.4 run the following:


```
yum install zend-server-php-5.4
```


To install PHP version 5.3 run the following:


```
yum install zend-server-php-5.3
```


When prompted by the installation, you can enter yes to all the
prompts, unless you know what you want to do differently.


# Step 3 - Verify Installation


 Once the installation has completed, you can verify that Zend
Server has been installed correctly by visiting the web UI. For the following
examples, replace 1.1.1.1 with your VPS IP address or domain.


You can also verify Apache is running by visiting the IP address or
domain of your VPS without the Zend Server 10081 port.


```
http://1.1.1.1
```


The following page should appear:


![](https://assets.digitalocean.com/tutorial_images/ZIgDkfM.png?1)
Open a browser and go to the IP address or domain of your VPS with
the Zend Server UI port of 10081:


```
http://1.1.1.1:10081
```


The license agreement screen should appear. Read the license and then
check the agreement checkbox and click the "Next" button:


![](https://assets.digitalocean.com/tutorial_images/W2U3P16.png?1)
You will then be presented with the launch type for the Zend Server. When
choosing your launch type for Zend Server, keep the following in
mind:


Development
This launch type will display any error which occurs in PHP. This
includes warnings, fatal errors, and strict errors. Normally, you would
not want this to occur in a production environment. By displaying all
errors your users would see all these warning messages regardless of
whether or not you intend to make that information public. With your
server in this mode, you could end up with display issues when a simple
warning gets returned by some code that you were not expecting. With
that said, the Development launch type is especially handy when you
are working on an application. It can provide feedback you would
normally need to look into the error logs to find. The immediate
feedback of displaying the errors can help speed up bug tracking.


Production (Single Server)
This launch type will suppress all errors from the user and keep the
memory usage for bug traces to a minimum so you are not cluttering
your system with unneeded or overly large logs. The same errors that
would be displayed to the user in a Development launch type will now
only be accessible by finding the correct log file. This is one area
that the Zend Server admin UI becomes quite useful. Also in the
Production launch type, 127.0.0.1 is the only allowed host for
connection to the Zend debugger.


Production (Create or Join a Cluster)
This launch type allows you to create a cluster of multiple Zend
Servers or join an existing cluster. In this mode, you have more
control over multiple servers running Zend Server. All with the same
settings as a Zend Server running as a Production (Single Server)
launch type. This is also only available with an Enterprise Zend
Server license or during the 30 day trial period for any new
installation such as this. After 30 days, your Enterprise trial
license will expire and your server will run as the Community Edition.
So unless you are planning on buying an Enterprise license or you are
trying the functionality out, I would suggest not using this launch
type:


![](https://assets.digitalocean.com/tutorial_images/3dhptDM.png?1)
Next (if you did not choose Production Cluster), you will be prompted
to enter your passwords. The admin password is required. You can skip
the developer password if you are planning to use Zend Server
Community Edition:


![](https://assets.digitalocean.com/tutorial_images/ODL1M0R.png?1)
Finally, you will be presented with a summary of the configurations
you have chosen. Click submit and wait for Zend Server to launch:


![](https://assets.digitalocean.com/tutorial_images/Hn3XDYA.png?1)
If everything went smoothly, you should be at the Zend Server admin UI
welcome screen:


![](https://assets.digitalocean.com/tutorial_images/8sBmHS8.png?1)
Your Zend Server installation is complete and you can start using it
to configure PHP on your cloud server.


# Reference Paths and Files


Zend Server uses its own location for installing PHP and Apache. 
The following paths can come in handy:


Location of Zend Server installation:


```
cd /usr/local/zend
```


Location of vhost files:


```
cd /usr/local/zend/etc/sites.d
```


Apache configuration file:


```
/etc/httpd/conf/httpd.conf
```


# Reference Commands


Here are Zend Server specific commands:


Start Zend Server:


```
/usr/local/zend/bin/zendctl.sh start
```


Stop Zend Server:


```
/usr/local/zend/bin/zendctl.sh stop
```


Retart Zend Server:


```
/usr/local/zend/bin/zendctl.sh restart
```


Start Apache:


```
/usr/local/zend/bin/zendctl.sh start-apache
```


Stop Apache:


```
/usr/local/zend/bin/zendctl.sh stop-apache
```


Restart Apache:


```
/usr/local/zend/bin/zendctl.sh restart-apache
```


# Resources


To find out more about the server launch types you can visit
Zend's documentation here.


Article Submitted by: Chad Robertson
