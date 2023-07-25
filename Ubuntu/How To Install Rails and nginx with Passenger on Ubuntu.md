# How To Install Rails and nginx with Passenger on Ubuntu

```Ubuntu``` ```Nginx``` ```Ruby on Rails```

## Introduction


Ruby on Rails is an application stack that provides web developers with a framework to quickly create a variety of web applications, and nginx is a light, high performance web server software. The two programs can be easily configured to work very well together on a virtual private server when installed through Phusion Passenger.


You can run this tutorial on your VPS as a user with sudo privileges. You can check out how to set that up here: Ubuntu Server Setup


# Step One— Install Ruby with RVM


Before we do anything else, we should run a quick update to make sure that all of the packages we download to our virtual server are up to date:


```
sudo apt-get update
```


Once that's done, we can start installing RVM,  Ruby Version Manage, on our VPSr. This is a great program that lets you use several versions of Ruby on one system; however, in this case, we will just use it to install the latest version of Ruby on the droplet.


To install RVM, open terminal and type in this command:


```
curl -L get.rvm.io | bash -s stable
```


After it is done installing, load RVM.


```
source ~/.rvm/scripts/rvm
```


In order to work, RVM has some of its own dependancies that need to be installed. You can see what these are:


```
rvm requirements
```


In the text that RVM shows you, look for this paragraph.


```
Additional Dependencies:
# For Ruby / Ruby HEAD (MRI, Rubinius, & REE), install the following:
  ruby: /usr/bin/apt-get install build-essential openssl libreadline6 libreadline6-dev curl git-core zlib1g zlib1g-dev libssl-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt-dev autoconf libc6-dev ncurses-dev automake libtool bison subversion
```


Just follow the instructions to get your system up to date with all of the required dependancies.


```
rvmsudo /usr/bin/apt-get install build-essential openssl libreadline6 libreadline6-dev curl git-core zlib1g zlib1g-dev libssl-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt-dev autoconf libc6-dev ncurses-dev automake libtool bison subversion
```


# Step Two—Install Ruby


Once you are using RVM, installing Ruby is easy.


```
rvm install 1.9.3
```


Ruby is now installed. However, since we accessed it through a program that has a variety of Ruby versions, we need to tell the system to use 1.9.3 by default.


```
rvm use 1.9.3 --default
```


#  Step Three—Install RubyGems


The next step makes sure that we have all the required components of Ruby on Rails. We can continue to use RVM to install gems; type this line into terminal.


```
 rvm rubygems current
```


# Step Four—Install Rails


Once everything is set up, it is time to install Rails.


To start, open terminal and type in:


```
gem install rails
```


This process may take a while, be patient with it. Once it finishes you will have Ruby on Rails installed on your virtual server.


Once that’s done you are all set with Ruby on Rails, and it is time to connect it to nginx


# Step Five—Install Passenger


Passenger is an effective and easy way to deploy Rails on nginx or apache. In this instance, we are going to run the nginx installation.


Once Ruby on Rails is installed, go ahead and install passenger.


```
gem install passenger 
```


#  Step Six—Install nginx 


Here is where Passenger really shines. As we are looking to install Rails on an nginx server, we only need to enter one more line into terminal:


```
rvmsudo passenger-install-nginx-module
```


And now Passenger takes over.


Passenger first checks that all of the dependancies it needs to work are installed. If you are missing any, Passenger will let you know how to install them, either with the apt-get installer on Ubuntu.


After you download any missing dependancies, restart the installation. Type: passenger-install-nginx-module once more into the command line.


Passenger offers users the choice between an automated setup or a customized one. Press 1 and enter to choose the recommended, easy, installation.


#  Step Seven—Start nginx


Passenger will take about five to ten minutes to install, configure, and optimize nginx with Ruby on Rails.


After it finishes, it will let you know about changes made to the nginx configuration file and how to deploy a Ruby on Rails application on your virtual server.


The last step is to turn start nginx, as it does not do so automatically.


```
 sudo service nginx start 
```


nginx is now on. You can see the exciting “Welcome to nginx” screen in your browser if you point it toward http://youripaddress/


# Step Eight—Connect Nginx to Your Rails Project


Once you have rails installed, open up the nginx config file


```
sudo nano /opt/nginx/conf/nginx.conf
```


Set the root to the public directory of your new rails project.


Your config should then look something like this:


```
server { 
listen 80; 
server_name example.com; 
passenger_enabled on; 
root /var/www/my_awesome_rails_app/public; 
}
```


##$ (*NB—to create your new rails project, follow these steps:
Install NodeJs if you do not yet have it:
sudo apt-get install nodejs
Create your new rails app in your preferred directory:
rails new my_awesome_rails_app



By Etel Sverdlov
