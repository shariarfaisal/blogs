# How To Automate Ruby On Rails Application Deployments Using Capistrano

```Ruby on Rails``` ```Ruby``` ```CentOS```

## Introduction



If you are not already fed up with repeating the same mundane tasks to update your application servers to get your project online, you probably will be eventually The joy you feel whilst developing your project tends to take a usual hit when it comes to the boring bits of system administration (e.g. uploading your codebase, amending configurations, executing commands over and over again, etc.)


But do not fear! Capistrano, the task-automation-tool, is here to help.


In this DigitalOcean article, we are going create a rock-solid server setup, running the latest version of CentOS to host Ruby-on-Rails applications using Nginx and Passenger. We will continue with learning how to automate the process of deployments - and updates - using the Ruby based automation tool Capistrano.


Note: This article builds on the knowledge from our past Capistrano article: Automating Deployments With Capistrano: Getting Started. In order to gain a good knowledge of the tool, which is highly recommended if you are going to use it, you are advised to read it before continuing with this piece. Likewise, if you would like to learn more about preparing a fresh droplet for Rails based application deployments with Passenger (and Nginx), check out the How To Deploy Rails Apps Using Passenger With Nginx article.


Note: Capistrano relies on Git for deployments. To learn more consider reading DigitalOcean community articles on the subject by clicking here.


# Glossary



## 1. Preparing The Deployment Server



1. Updating And Preparing The Operating System
2. Setting Up Ruby Environment and Rails
3. Downloading And Installing App. & HTTP Servers
4. Creating The Nginx Management Script
5. Configuring Nginx For Application Deployment
6. Downloading And Installing Capistrano
7. Creating A System User For Deployment

## 2. Preparing Rails Applications For Git-Based Capistrano Deployment



1. Creating A Basic Ruby-On-Rails Application
2. Creating A Git Repository

## 3. Working With Capistrano To Automate Deployments



1. Installing Capistrano Inside The Project Directory
2. Working With config/deploy.rb Inside The Project Directory
3. Working With config/deploy/production.rb Inside The Project Directory
4. Deploying To The Production Server

# Preparing The Deployment Server



Note: To have a better understanding of the below section, which can be considered a lengthy summary, check out the full article on the subject: How To Deploy Rails Apps Using Passenger With Nginx.


## Updating And Preparing The Operating System



Run the following command to update the default tools of your CentOS based droplet:


```
yum -y update

```


Install the bundle containing development tools by executing the following command:


```
yum groupinstall -y 'development tools'

```


Some of the packages we need for this tutorial (e.g. libyaml-devel, nginx etc.) are not found within the official CentOS repository.


Run the following to add the EPEL repository:


```
sudo su -c 'rpm -Uvh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm'

yum -y update

```


Finally, in order to install some additional libraries and tools, run the following command:


```
yum install -y curl-devel nano sqlite-devel libyaml-devel

```


## Setting Up Ruby Environment and Rails



Note: This section is a summary of our dedicated article How To Install Ruby 2.1.0 On CentOS 6.5.


Run the following two commands to install RVM and create a system environment for Ruby:


```
curl -L get.rvm.io | bash -s stable

source /etc/profile.d/rvm.sh
rvm reload
rvm install 2.1.0

```


Since Rails needs a JavaScript interpreter, we will also need to set up Node.js.


Run the following to download and install nodejs using yum:


```
yum install -y nodejs

```


Execute the following command using RubyGems’ gem to download and install rails:


```
gem install bundler rails

```


## Downloading And Installing App. & HTTP Servers



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

```


## Nginx



Note: Normally, to download and install Nginx, you could add the EPEL repository (as we have already done) and get Nginx via yum. However, to get Nginx work with Passenger, its source must be compiled with the necessary modules.


Run the following to start compiling Nginx with native Passenger module:


```
passenger-install-nginx-module

```


Once you run the command, press Enter and confirm your choice of language(s) (i.e. Ruby, in our case). You can use arrow keys and the space bar to select Ruby alone, if you wish.


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


## Configuring Nginx For Application Deployment



In this final step of configuring our servers, we need to create an Nginx server block, which roughly translates to Apache’s virtual hosts.


As you might remember seeing during Passenger’s Nginx installation, this procedure consists of adding a block of code to Nginx’s configuration file nginx.conf. By default, unless you states otherwise, this file can be found under /opt/nginx/conf/nginx.conf.


Type the following command to open up this configuration file to edit it with the text editor nano:


```
nano /opt/nginx/conf/nginx.conf

```


As the first step, find the http { node and append the following right after the passenger_root and passenger_ruby directives:


```
# Only for development purposes.
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
# Set the folder where you will be deploying your application.
# We are using: /home/deployer/apps/my_app
root              /home/deployer/apps/my_app/public;
passenger_enabled on;

```


Press CTRL+X and confirm with Y to save and exit.


Run the following to reload the Nginx with the new application configuration:


```
# !! Remember to create an Nginx management script
#    by following the main Rails deployment article for CentOS
#    linked at the beginning of this section.

/etc/init.d/nginx restart

```


To check the status of Nginx, you can use:


```
/etc/init.d/nginx status

```


Note: To learn more about Nginx, please refer to How to Configure Nginx Web Server on a VPS.


## Downloading And Installing Capistrano



Once we have our system ready, getting Capistrano’s latest version, thanks to RubyGems is a breeze.


You can simply use the following to get Capistrano version 3:


```
gem install capistrano

```


## Creating A System User For Deployment



In this step, we are going to create a CentOS system user to perform the actions of deployment. This is going to be the user for Capistrano to use.


Note: To keep things basic, we are going to create a deployer user with necessary privileges. For a more complete set up, consider using the groups example from the Capistrano introduction tutorial.


Create a new system user deployer:


```
adduser deployer

```


Set up deployer’s password:


```
passwd deployer

# Enter a password
# Confirm the password

```


Edit /etc/sudoers using the text editor nano:


```
nano /etc/sudoers

```


Scroll down the file and find where root is defined:


```
..

## The COMMANDS section may have other options added to it.
##
## Allow root to run any commands anywhere
root    ALL=(ALL)	ALL

..

```


Append the following right after root ALL=(ALL) ALL:


```
deployer ALL=(ALL) ALL

```


This section of the /etc/sudoers file should now look like this:


```
..

## The COMMANDS section may have other options added to it.
##
## Allow root to run any commands anywhere
root     ALL=(ALL)	ALL
deployer ALL=(ALL) ALL

..

```


Press CTRL+X and confirm with Y to save and exit.


# Preparing Rails Applications For Git-Based Capistrano Deployment



Once we have our system ready, with all the necessary applications set up and working correctly, we can move on to creating an exemplary Rails application to use as a sample.


In the second stage, we are going to create a Git repository and push the code base to a central, accessible location at Github for Capistrano to use for deployments.


Note: Here, we are creating a sample application. For the actual deployments, you should perform these actions on your own, after making sure that everything is backed up – just in case! Also, please note that you will need to run Capistrano from a different location than the server where the application needs to be deployed.


## Creating A Basic Ruby-On-Rails Application



Note: The below step is there to create a substitute Rails application to try out Capistrano.


Having Ruby and Rails already installed leaves us with just a single command to get started.


Execute the following command to get Rails create a new application called my_app:


```
# Create a sample Rails application
rails new my_app

# Enter the application directory
cd my_app

# Create a sample resource
rails generate scaffold Task title:string note:text

# Create a sample database
RAILS_ENV=development rake db:migrate

```


To test that your application is set correctly and everything is working fine, enter the app directory and run a simple server via rails s:


```
# Enter the application directory
cd my_app

# Run a simple server
rails s

# You should now be able to access it by
# visiting: http://[your droplet's IP]:3000

# In order to terminate the server process,
# Press CTRL+C

```


## Creating A Git Repository



Note: To learn more about working with Git, check out the How To Use Git Effectively tutorial at DigitalOcean community pages.


Note: In order to follow this section, you will need a Github account. Alternatively, you can set up a droplet to host your own Git repository following this DigitalOcean article on the subject. If you choose to do so, make sure to use the relevant URL on deployment files.


We are going to use the sample instructions supplied by Github to create a source repository.


Execute the following, self-explanatory commands inside the my_app directory to initiate a repository:


```
# !! These commands are to be executed on
#    your development machine, from where you will
#    deploy to your server.
#    Instructions might vary slightly depending on
#    your choice of operating system.
#
#    Make sure to set correct paths for application
#    Otherwise Nginx might not be able to locate it.

# Initiate the repository
git init

# Add all the files to the repository
git add .

# Commit the changes
git commit -m "first commit"

# Add your Github repository link 
# Example: git remote add origin git@github.com:[user name]/[proj. name].git
git remote add origin git@github.com:user123/my_app.git

# Create an RSA/SSH key
# Follow the on-screen instructions
ssh-keygen -t rsa

# View the contents of the key and add it to your Github
# by copy-and-pasting from the current remote session by
# visiting: https://github.com/settings/ssh
# To learn more about the process,
# visit: https://help.github.com/articles/generating-ssh-keys
cat /root/.ssh/id_rsa.pub

# Set your Github information
# Username:
# Usage: git config --global user.name "[your username]"
git config --global user.name "user123"

# Email:
# Usage: git config --global user.email "[your email]"
git config --global user.email "user123@domain.tld"

# Push the project's source code to your Github account
git push -u origin master

```


# Working With Capistrano To Automate Deployments



As you will remember from our first Capistrano article, the way to begin using the library is by installing it inside the project directory. In this section, we will see how to do that, followed by creating files that are needed to set servers.


## Installing Capistrano Inside The Project Directory



Another simple step in our article is installing the Capistrano files. The below command will scaffold some directories and files to be used by the tool for the deployment.


Run the following to initiate (i.e. install) Capistrano files:


```
cap install

# mkdir -p config/deploy
# create config/deploy.rb
# create config/deploy/staging.rb
# create config/deploy/production.rb
# mkdir -p lib/capistrano/tasks
# Capified

```


## Working With config/deploy.rb Inside The Project Directory



The file deploy.rb contains arguments and settings relevant to the deployment server(s). Here, we will tell Capistrano to which server(s) we would like to connect and deploy and how.


Note: When editing the file (or defining the configurations), you can either comment them out or add the new lines. Make sure to not to have some example settings overriding the ones you are appending.


Run the following to edit the file using nano text editor:


```
nano config/deploy.rb

```


Add the below block of code, modifying it to suit your own settings:


```
# Define the name of the application
set :application, 'my_app'

# Define where can Capistrano access the source repository
# set :repo_url, 'https://github.com/[user name]/[application name].git'
set :scm, :git
set :repo_url, 'https://github.com/user123/my_app.git'

# Define where to put your application code
set :deploy_to, "/home/deployer/apps/my_app"

set :pty, true

set :format, :pretty

# Set the post-deployment instructions here.
# Once the deployment is complete, Capistrano
# will begin performing them as described.
# To learn more about creating tasks,
# check out:
# http://capistranorb.com/

# namespace: deploy do

#   desc 'Restart application'
#   task :restart do
#     on roles(:app), in: :sequence, wait: 5 do
#       # Your restart mechanism here, for example:
#       execute :touch, release_path.join('tmp/restart.txt')
#     end
#   end

#   after :publishing, :restart

#   after :restart, :clear_cache do
#     on roles(:web), in: :groups, limit: 3, wait: 10 do
#       # Here we can do anything such as:
#       # within release_path do
#       #   execute :rake, 'cache:clear'
#       # end
#     end
#   end

# end

```


Press CTRL+X and confirm with Y to save and exit.


## Working With config/deploy/production.rb Inside The Project Directory



Note: Similar to deploy.rb, you will need to make some amendments to the production.rb file. You are better modifying the code instead of appending the below block.


Run the following to edit the file using nano text editor:


```
nano config/deploy/production.rb

```


Enter your server’s settings, similar to below:


```
# Define roles, user and IP address of deployment server
# role :name, %{[user]@[IP adde.]}
role :app, %w{deployer@162.243.74.190}
role :web, %w{deployer@162.243.74.190}
role :db,  %w{deployer@162.243.74.190}

# Define server(s)
server '162.243.74.190', user: 'deployer', roles: %w{web}

# SSH Options
# See the example commented out section in the file
# for more options.
set :ssh_options, {
    forward_agent: false,
    auth_methods: %w(password),
    password: 'user_deployers_password',
    user: 'deployer',
}

```


Press CTRL+X and confirm with Y to save and exit.


## Deploying To The Production Server



Once we are done with the settings, it is time to deploy.


Run the following code on your development machine to deploy to the production server. As defined in the above files, Capistrano will:


- 
Connect to the deployment server

- 
Download the application source

- 
Perform the deployment actions (i.e. get passenger restart the application)


```
cap production deploy

```


To learn more about Capistrano and what it can do, consider reading the Capistrano documentation.


<div class=“author”>Submitted by: <a
href=“https://twitter.com/ostezer”>O.S. Tezer</a></div>


