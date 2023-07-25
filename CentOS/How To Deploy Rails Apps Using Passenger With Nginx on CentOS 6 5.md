# How To Deploy Rails Apps Using Passenger With Nginx on CentOS 6 5

```Ruby on Rails``` ```Nginx``` ```CentOS```


Status: Deprecated
This article covers a version of CentOS that is no longer supported. If you are currently operating a server running CentOS 6, we highly recommend upgrading or migrating to a supported version of CentOS.
Reason:
CentOS 6 reached end of life (EOL) on November 30th, 2020 and no longer receives security patches or updates. For this reason, this guide is no longer maintained.
See Instead:
This guide might still be useful as a reference, but may not work on other CentOS releases. If available, we strongly recommend using a guide written for the version of CentOS you are using.

## Introduction



Challenges never really end, especially if you are new to a certain area of computer programming. In this case, our subject matter is Rails and how to get your Ruby On Rails based web application(s) online – the simplest and quickest way possible.


After having solved the puzzles to get started working on your application using the Ruby programming language and Rails web application development framework, when it is time to share your application with the rest of the world it is possible to be confused with all the available choices and the endless amount of possible combinations that exist.


In this DigitalOcean article, we are going to show you – from start to finish – how to have a rock solid Rails application deployment (i.e. published online) using the latest available CentOS operating system renowned for its stability. This will be alongside Phusion Passenger application server, known for its simplicity and excellent features, coupled with Nginx HTTP server running in front to handle and manage connections.


Note: During this walk-through, you are advised to check out and read the content of links provided. They will help you with enhancing performance, security, et al.


# Glossary



## 1. Web Application Deployment, Servers And Their Roles



1. Phusion Passenger Application Server
2. Nginx HTTP Server Running As Reverse-Proxy

## 2. Preparing The Deployment Server



1. Updating And Preparing The Operating System
2. Setting Up Ruby Environment and Rails
3. Downloading And Installing The Server Applications

## 3. Preparing The Application For Deployment



1. Creating A Sample Application / Uploading Your Source Code
2. Creating The Nginx Management Script
3. Configuring Nginx

## 4. Further Reading



# Web Application Deployment, Servers And Their Roles



When it comes to deploying web applications, or putting them online, there are usually multiple layers of application used for the purpose. Surely a single one could do the job, but probably not very well, since they are not made fit-for-all-purposes.


In this tutorial, we’ll be using Phusion Passenger as the application server. The application servers’ job consists of containing modern web applications (e.g. Ruby Rack, Python WSGI etc.) and act as the secondary entry point of incoming web requests.


Nginx, on the other hand, is designed from ground up to act as a multi-purpose HTTP server. It is capable of serving static files (e.g. images, text files etc.) extremely well, balance connections, and deal with certain exploits attempts. It acts as the first entry point of all requests, and passes them to Passenger for the web application to process and return a response.


## Phusion Passenger Application Server



Passenger today has become the recommended server for Ruby on Rails applications. It is a mature, feature rich product which aims to cover necessary needs and areas of application deployment whilst greatly simplifying the set-up and getting-started procedures. It eliminates the traditional middleman server set up architecture by direct integration with Nginx (and Apache as well). It is also referred to as mod_rails.


Passenger is highly popular and used widely in many production scenarios. It is very easily possible to reach out and find experts, as well as have your issues addressed online.


The open-source version, which we will be using, has a multi-process single-threaded operation mode. Its Enterprise version can be configured to work either single-threaded or multi-threaded.


To learn more about Passenger, you can visit its official website located at https://www.phusionpassenger.com/.


## Nginx HTTP Server Running As Reverse-Proxy



Nginx is a very high performant web server / (reverse)-proxy. It has reached its popularity due to being light weight, relatively easy to work with, and easy to extend (with add-ons / plug-ins). Thanks to its architecture, it is capable of handling a lot of requests (virtually unlimited), which - depending on your application or website load - could be really hard to tackle using some other older alternatives.


Remember: “Handling” connections technically means not dropping them and being able to serve them with something. You still need your application and database functioning well in order to have Nginx serve clients responses that are not error messages.


Due to its popularity and success, we are going to deploy our application running behind Nginx to benefit from its powerful features.


To learn more about Nginx, you can visit its official website located at nginx.com.


# Preparing The Deployment Server



In this section, we are going to perform the following steps to obtain a rock-solid server, ready to serve your application.


- 
Update the operating system

- 
Get the necessary basic tools for deployment

- 
Install Ruby, Rails and libraries

- 
Install Application (i.e. Passenger) and HTTP server (Nginx)


## Updating And Preparing The Operating System



In order to install Ruby and the other necessary application (e.g. our servers), we need to first prepare the minimally shipped CentOS droplet and equip it with some development tools we will need along the way.


Run the following command to update the default tools of your CentOS based droplet:


```
yum -y update

# This command will update all the base applications
# that come with CentOS by default. Which are mostly
# reserved for use by the operating system. 

```


Install the bundle containing development tools by executing the following command:


```
yum groupinstall -y 'development tools'

# With more recent versions of CentOS, such as 6.5 in our case,
# you can simply run:
# yum groupinstall -y development
# instead.

# This bundle of applications contains various tools
# Such as: gcc, make, automake, binutils, git etc.

```


Some of the packages we need for this tutorial (e.g. libyaml-devel, nginx etc.) are not found within the official CentOS repository. To simplify things and not to deal with manually installing them, we will add the EPEL software repository for YUM and other package managers to use.


```
# Enable EPEL Repository
sudo su -c 'rpm -Uvh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm'

# Update everything, once more.
yum -y update

```


Finally, to get Passenger working with Nginx, which we are going to install in the next sections, we need the curl-devel library and nano text editor. Also for Rails, we are going to need sqlite-devel.


In order to install curl-devel and nano, run the following:


```
yum install -y curl-devel nano sqlite-devel libyaml-devel

```


## Setting Up Ruby Environment and Rails



Note: This section is a summary of our dedicated article How To Install Ruby 2.1.0 On CentOS 6.5.


We are going to be using Ruby Version Manager (RVM) to download and install a Ruby interpreter (or “rubies”, as referred by the RVM).


Run the following two commands to install RVM and create a system environment for Ruby:


```
curl -L get.rvm.io | bash -s stable
source /etc/profile.d/rvm.sh

```


Finally, to finish installing Ruby on our system, let’s get RVM to download and install Ruby version 2.1.0:


```
rvm reload
rvm install 2.1.0

```


After Ruby, we can use the RubyGems package manager to help us to get rest of the Ruby based tools, such as the Rails framework.


Since Rails needs first and foremost a JavaScript interpreter to work, we will also need to set up Node.js. For this purpose, we will be using the default system package manager YUM.


Run the following to download and install nodejs using yum:


```
yum install -y nodejs

```


Execute the following command using RubyGems’ gem to download and install rails:


```
gem install bundler rails

```


## Downloading And Installing The Server Applications



Note: If your VPS has less than 1 GB of RAM, you will need to perform the below simple procedure to prepare a SWAP disk space to be used as a temporary data holder (RAM substitute). Since DigitalOcean servers come with fast SSD disks, this does not really constitute an issue whilst performing the server application installation tasks.


```
# Create a 1024 MB SWAP space
sudo dd if=/dev/zero of=/swap bs=1M count=1024
sudo mkswap /swap
sudo swapon /swap

```


## Phusion Passenger



Red Hat Linux’s default package manager RPM (RPM Package Manager) ships applications contained within .rpm files. Unfortunately, in Passenger’s case, they are quite outdated. Therefore, we will be using RubyGem, once again, to download and install the latest available version of Passenger – version 4.


Use the below command to simply download and install passenger:


```
gem install passenger

# This command will fetch Passenger v4.0(.35+) for you.

```


To test Passenger is downloaded and set up correctly, try running passenger.


You should see an output similar to below:


```
Phusion Passenger Standalone, the easiest way to deploy Ruby web apps.

Available commands:

  passenger start            Start Phusion Passenger Standalone.

..

```


## Nginx



Normally, to download and install Nginx, you could add the EPEL repository and get Nginx via yum. However, to get Nginx working with Passenger, its source must be compiled with the necessary modules. Do not worry, though! Passenger comes with a handy tool to make the process as simple as executing a single command.


Run the following to start compiling Nginx with native Passenger module:


```
passenger-install-nginx-module

```


Once you run the command, press Enter and confirm your choice of language(s) (i.e. Ruby, in our case). You can use the arrow keys and space bar to select Ruby alone, if you wish.


```
Use <space> to select.
If the menu doesn't display correctly, ensure that your terminal supports UTF-8.

 ‣ ⬢  Ruby
   ⬢  Python
   ⬢  Node.js
   ⬡  Meteor

```


In the next step, choose Item 1:


```
1. Yes: download, compile and install Nginx for me. (recommended)
    The easiest way to get started. A stock Nginx 1.4.4 with Passenger
    support, but with no other additional third party modules, will be
    installed for you to a directory of your choice.

```


And press Enter to continue.


Now, Nginx source will be downloaded, compiled, and installed with Passenger support.


Note: This action might take a little while – probably longer than one would like or expect!


# Preparing The Application For Deployment



Note: In this section, we’re going to work with a very simple Ruby On Rails application as an example. For the actual deployment of your application, you should upload your codebase and make sure to have all of its dependencies installed.


## Creating A Sample Application / Uploading Your Source Code



Let’s begin with creating a very basic Rails application inside our home directory to serve with Passenger and Nginx.


Execute the following command to get Rails to create a new application called my_app inside the /var/www directory:


```
# Create a sample Rails application
cd /var
mkdir www
cd www
rails new my_app

# Enter the application directory
cd my_app

# Create a sample resource
rails generate scaffold Task title:string note:text

# Create a sample database
RAILS_ENV=development rake db:migrate

```


To test that your application is set correctly and everything is working fine, enter the app directory and run a simple server with rails s:


```
# Enter the application directory
cd /var/www/my_app

# Run a simple server
rails s

# You should now be able to access it by
# visiting: http://[your droplet's IP]:3000/tasks

# In order to terminate the server process,
# Press CTRL+C

```


Note: For the actual deployment, when you want to upload your code base to the server, you can use either SFTP or a graphical tool []such as FileZilla] to transfer and manage remote files securely.


- 
To learn about working with SFTP, check out the article: How To Use SFTP.

- 
To learn about FileZilla, check out the article on the subject: How To Use FileZilla.


## Creating The Nginx Management Script



After compiling Nginx, in order to control it with ease, we need to create a simple management script.


Run the following commands to create the script:


```
nano /etc/rc.d/init.d/nginx

```


Copy and paste the below contents:


```
#!/bin/sh
. /etc/rc.d/init.d/functions
. /etc/sysconfig/network
[ "$NETWORKING" = "no" ] && exit 0

nginx="/opt/nginx/sbin/nginx"
prog=$(basename $nginx)

NGINX_CONF_FILE="/opt/nginx/conf/nginx.conf"

lockfile=/var/lock/subsys/nginx

start() {
    [ -x $nginx ] || exit 5
    [ -f $NGINX_CONF_FILE ] || exit 6
    echo -n $"Starting $prog: "
    daemon $nginx -c $NGINX_CONF_FILE
    retval=$?
    echo
    [ $retval -eq 0 ] && touch $lockfile
    return $retval
}

stop() {
    echo -n $"Stopping $prog: "
    killproc $prog -QUIT
    retval=$?
    echo
    [ $retval -eq 0 ] && rm -f $lockfile
    return $retval
}

restart() {
    configtest || return $?
    stop
    start
}

reload() {
    configtest || return $?
    echo -n $”Reloading $prog: ”
    killproc $nginx -HUP
    RETVAL=$?
    echo
}

force_reload() {
    restart
}

configtest() {
    $nginx -t -c $NGINX_CONF_FILE
}

rh_status() {
    status $prog
}

rh_status_q() {
    rh_status >/dev/null 2>&1
}

case "$1" in
start)
rh_status_q && exit 0
$1
;;
stop)
rh_status_q || exit 0
$1
;;
restart|configtest)
$1
;;
reload)
rh_status_q || exit 7
$1
;;
force-reload)
force_reload
;;
status)
rh_status
;;
condrestart|try-restart)
rh_status_q || exit 0
;;
*)
echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
exit 2
esac

```


Press CTRL+X and confirm with Y to save and exit.


Set the mode of this management script as executable:


```
chmod +x /etc/rc.d/init.d/nginx

```


## Configuring Nginx



In this final step of configuring our servers, we need to create an Nginx server block, which roughly translates to Apache’s virtual hosts.


As you might remember seeing during Passenger’s Nginx installation, this procedure consists of adding a block of code to Nginx’s configuration file nginx.conf. By default, unless you state otherwise, this file can be found under /opt/nginx/conf/nginx.conf.


Type the following command to open up this configuration file to edit it with the text editor nano:


```
nano /opt/nginx/conf/nginx.conf

```


As the first step, find the http { node and append the following right after the passenger_root and passenger_ruby directives:


```
# Only for development purposes.
# For production environment, set it accordingly (i.e. production)
# Remove this line when you upload an actual application.
# For * TESTING * purposes only.
passenger_app_env development;

```


Scroll down the file and find server { ... Comment out the default location, i.e.:


```
..

#    location / {
#            root   html;
#            index  index.html index.htm;
#        }

..

```


And define your default application root:


```
root /var/www/my_app/public;
passenger_enabled on;

```


Press CTRL+X and confirm with Y to save and exit.


Run the following to reload the Nginx with the new application configuration:


```
/etc/init.d/nginx restart

```


To check the status of Nginx, you can use:


```
/etc/init.d/nginx status

```


In order to test your application (and our sample app), you can visit:


```
http://[Your droplet's IP addr]/tasks

# Listing tasks

# Title    Note	

# New Task

```


Note: To learn more about Nginx, please refer to How to Configure Nginx Web Server on a VPS.


# Further Reading



## Firewall



Setting up a firewall using IP Tables


## Securing SSH



How To Protect SSH with fail2ban on Ubuntu
How To Protect SSH with fail2ban on CentOS 6


## Creating Alerts



How To Send E-Mail Alerts on a CentOS VPS for System Monitoring


## Monitor and Watch Server Access Logs Daily



How To Install and Use Logwatch Log Analyzer and Reporter


<div class=“author”>Submitted by: <a
href=“https://twitter.com/ostezer”>O.S. Tezer</a></div>


