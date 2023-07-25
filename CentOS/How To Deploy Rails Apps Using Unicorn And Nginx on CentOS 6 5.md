# How To Deploy Rails Apps Using Unicorn And Nginx on CentOS 6 5

```Ruby on Rails``` ```Ruby``` ```CentOS```


Status: Deprecated
This article covers a version of CentOS that is no longer supported. If you are currently operating a server running CentOS 6, we highly recommend upgrading or migrating to a supported version of CentOS.
Reason:
CentOS 6 reached end of life (EOL) on November 30th, 2020 and no longer receives security patches or updates. For this reason, this guide is no longer maintained.
See Instead:
This guide might still be useful as a reference, but may not work on other CentOS releases. If available, we strongly recommend using a guide written for the version of CentOS you are using.

# Introduction



Application servers which are designed with simplicity can get you up and running in mere minutes when you are deploying your Rails based web-application. If, however, you wish to have more control of your server set-up or would like try something new that is more flexible, using a layered set of components can help you to achieve your goals – whether it is future-proofing the deployment or need of introducing third party elements such as caching servers.


In this DigitalOcean article, we are going to take a look at assembling a multi-layer deployment installation to host Rails based Ruby web applications. For this arrangement, we will use the ever-so-powerful, flexible, and extremely successful Unicorn application server running behind Nginx. Although we will be building this structure on a single server for demonstration purposes, you can easily use multiple droplets to spread things and scale out easily – both horizontally and vertically!


# Glossary



## 1. Web Application Deployment, Servers And Their Roles



1. Unicorn Application Server
2. Nginx HTTP Server Running As A Front-End Reverse-Proxy

## 2. Preparing The Deployment Server



1. Updating And Preparing The Operating System
2. Setting Up Ruby Environment and Rails
3. Installing Nginx
4. Installing Unicorn

## 3. Preparing Rails Applications For Deployment



1. Creating A Sample Application
2. Uploading Your Source Code

## 4. Configuring Servers



1. Unicorn
2. Nginx
3. Managing The Servers

## 5. Further Reading



# Web Application Deployment, Servers And Their Roles



When it comes to deploying web applications, there usually are a multitude of applications involved, set up in layers and working with each other. This kind of real-world deployment set-up differs hugely from using a singular development server, which is designed to be used just for testing purposes since they cannot work under the loads of actual website traffic due to lack of functionality and features.


Talking about features, it should be noted that there a handful of popular servers to choose from with each offering different functionality: some focusing on simplicity, some speed, and some a little bit of everything with possibility to configure options to suit complex production needs.


In this article, our choice of application server is Unicorn. Unicorn is a remarkable application server that contains your Rails app to process incoming requests, preferably after having them filtered and sent by a front-end HTTP server such as Nginx.


Nginx HTTP server, on the other hand, is designed from the ground up to act as a multi-purpose, front-facing web server. It is capable of serving static files (e.g. images, text files, etc.) extremely well, balance connections, and deal with certain exploits attempts. It acts as the first entry point of all requests, and passes them to Unicorn for the web application to process and return a response.


Note: To learn about different Ruby web-application servers and understand what “Rack” is, check out our article A Comparison of (Rack) Web Servers for Ruby Web Applications.


## Unicorn Application Server



Unicorn is a very mature web application server for Ruby/Rack based web applications. It is fully-featured, but it denies by design trying to do everything. Unicorn’s principal is doing what needs to be done by a web application server and delegating rest of the responsibilities.


Unicorn’s master process spawns workers, as per your requirements, to serve the requests. This process also monitors the workers in order to prevent memory and process related staggering issues. What this means for system administrators is that it will kill a process if, for example, it takes too much time to complete a task or memory issues occur.


As mentioned above, one of the areas in which Unicorn delegates tasks is using the operating system for load balancing. This allows the requests not to pile up against busy workers spawned.


## Nginx HTTP Server Running As A Front-End Reverse-Proxy



Nginx is a very high performant web server / (reverse)-proxy. It has reached its popularity due to being light weight, relatively easy to work with and easy to extend (with add-ons / plug-ins). Thanks to its architecture, it is capable of handling a lot of requests (virtually unlimited), which - depending on your application or website load - could be really hard to tackle using some other, older alternatives.


Remember: “Handling” connections technically means not dropping them and being able to serve them with something. You still need your application and database functioning well in order to have Nginx serve clients responses that are not error messages.


To learn more about Nginx, you can visit its official website located at nginx.com.


# Preparing The Deployment Server



In this section, we are going to perform the following steps:


- 
Update the operating system

- 
Get the necessary basic tools for deployment

- 
Install Ruby, Rails and libraries

- 
Install Application (i.e. Unicorn) and HTTP server (Nginx)


## Updating And Preparing The Operating System



In order to install Ruby and the other necessary application (e.g. our servers), we need to first prepare the minimally shipped CentOS droplet and equip it with some development tools needed along the way.


Run the following command to update the default tools of your CentOS VPS:


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


Some of the packages we need for this tutorial (e.g. libyaml-devel, nginx etc.) are not found within the official CentOS repository. To simplify things and not to deal with manually installing them, we will add the EPEL software repository for YUM package manager to use.


```
# Enable EPEL Repository
sudo su -c 'rpm -Uvh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm'

# Update everything, once more.
yum -y update

```


Finally, we need to get curl-devel and several other tools and libraries for this tutorial (e.g. Rails needs sqlite-devel).


In order to install them, run the following:


```
yum install -y curl-devel nano sqlite-devel libyaml-devel

```


## Setting Up Ruby Environment and Rails



Note: This section is a summary of our dedicated article How To Install Ruby 2.1.0 On CentOS 6.5.


We are going to be using Ruby Version Manager (RVM) to download and install a Ruby interpreter.


Run the following two commands to install RVM and create a system environment for Ruby:


```
gpg --keyserver hkp://keys.gnupg.net --recv-keys D39DC0E3
\curl -sSL https://get.rvm.io | bash -s stable
source /etc/profile.d/rvm.sh

```


Finally, to finish installing Ruby on our system, let’s get RVM to download and install Ruby version 2.1.0:


```
rvm reload
rvm install 2.1.0

```


Since Rails needs first and foremost a JavaScript interpreter to work, we will also need to set up Node.js. For this purpose, we will be using the default system package manager YUM.


Run the following to download and install nodejs using yum:


```
yum install -y nodejs

```


Execute the following command to download and install rails using gem:


```
gem install bundler rails

```


## Installing Nginx



Since we have the EPEL repository enabled, it is possible to get Nginx using yum.


Run the following to download and install Nginx using yum:


```
yum install -y nginx

```


Note: We will be configuring this tool in the following sections.


## Installing Unicorn



There are a couple of ways to easily download Unicorn. Since it is an application-related dependency, the most logical way is to use RubyGems.


Run the following to download and install Unicorn using gem:


```
gem install unicorn

```


Note: We will see how to work with this tool in the next sections.


# Preparing Rails Applications For Deployment



Note: In this section, we are going to work with a very simple Ruby On Rails application as an example. For the actual deployment of your application, you should upload your codebase and make sure to have all of its dependencies install (i.e. bundle).


## Creating A Sample Application



Let’s begin with creating a very basic Rails application inside our home directory to serve with Unicorn.


Execute the following command to get Rails create a new application called “my_app”:


```
# Create a sample Rails application
cd  /var
mkdir www
cd www
rails new my_app

# Enter the application directory
cd my_app

# Create a sample resource
rails generate scaffold Task title:string note:text

# Create a sample database
RAILS_ENV=development rake db:migrate
RAILS_ENV=production  rake db:migrate

# Create a directory to hold the PID files
mkdir pids    

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


## Uploading Your Source Code



For the actual deployment, you will, of course, want to upload your code base to the server. For this purpose, you can either use SFTP or a graphical tool, such as FileZilla, to transfer and manage remote files securely. Likewise, you can use Git and a central repository such as Github to download and set up your code.


- 
To learn about working with SFTP, check out the article: How To Use SFTP.

- 
To learn about FileZilla, check out the article on the subject: How To Use FileZilla.


# Configuring Servers



## Unicorn



Unicorn can be configured a number of ways. For this tutorial, focusing on the key elements, we will create a file from scratch which is going to be used by Unicorn when starting the application server daemon process.


Open up a blank unicorn.rb document, which will be saved inside config/ directory:


```
nano config/unicorn.rb

```


Place the below block of code, modifying it as necessary:


```
# Set the working application directory
# working_directory "/path/to/your/app"
working_directory "/var/www/my_app"

# Unicorn PID file location
# pid "/path/to/pids/unicorn.pid"
pid "/var/www/my_app/pids/unicorn.pid"

# Path to logs
# stderr_path "/path/to/log/unicorn.log"
# stdout_path "/path/to/log/unicorn.log"
stderr_path "/var/www/my_app/log/unicorn.log"
stdout_path "/var/www/my_app/log/unicorn.log"

# Unicorn socket
listen "/tmp/unicorn.[app name].sock"
listen "/tmp/unicorn.myapp.sock"

# Number of processes
# worker_processes 4
worker_processes 2

# Time-out
timeout 30

```


Save and exit by pressing CTRL+X and confirming with Y.


Note: To simply test your application with Unicorn, you can run unicorn_rails inside the application directory.


## Nginx



Next, we need to tell Nginx how to talk to Unicorn. For this purpose, it is sufficient at this level to edit the default configuration file: default.conf and leave nginx.conf as provided – which is already set to include the default configurations.


```
nano /etc/nginx/conf.d/default.conf

```


Replace the files contents with the ones from below, again amending the necessary bits to suit your needs:


```
upstream app {
    # Path to Unicorn SOCK file, as defined previously
    server unix:/tmp/unicorn.myapp.sock fail_timeout=0;
}

server {


    listen 80;
    server_name localhost;

    # Application root, as defined previously
    root /root/my_app/public;
    
    try_files $uri/index.html $uri @app;
    
    location @app {
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_redirect off;
        proxy_pass http://app;
    }
    
    error_page 500 502 503 504 /500.html;
    client_max_body_size 4G;
    keepalive_timeout 10;
}  

```


Save and exit by pressing CTRL+X and confirming with Y.


Note: To learn more about Nginx, please refer to How to Configure Nginx Web Server on a VPS.


## Managing The Servers



After we complete configuring both servers, it is time to go online!


Let’s start the Unicorn and run it as a daemon using the configuration file:


```
# Make sure that you are inside the application directory
# i.e. /my_app
unicorn_rails -c config/unicorn.rb -D

# You can set the environment by chaining -E flag
# i.e. unicorn_rails .. .. .. -E [env. name]

```


Next, we are ready to reload and restart Nginx:


```
service nginx restart

```


And that’s it! You can now check out your deployment by going to your droplet’s IP address (or the domain name associated to it).


```
http://[Your droplet's IP addr]/tasks

# Listing tasks

# Title    Note	

# New Task

```


# Further Reading



## Firewall:



Setting up a firewall using IP Tables


## Securing SSH:



How To Protect SSH with fail2ban on Ubuntu
How To Protect SSH with fail2ban on CentOS 6


## Creating Alerts:



How To Send E-Mail Alerts on a CentOS VPS for System Monitoring


## Monitor and Watch Server Access Logs Daily:



How To Install and Use Logwatch Log Analyser and Reporter


## Optimising Unicorn Workers:



How to Optimize Unicorn Workers in a Ruby on Rails App


<div class=“author”>Submitted by: <a
href=“https://twitter.com/ostezer”>O.S. Tezer</a></div>


