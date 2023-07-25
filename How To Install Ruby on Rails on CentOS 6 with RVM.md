# How To Install Ruby on Rails on CentOS 6 with RVM

```Ruby on Rails``` ```Ruby``` ```CentOS```










# Status: Deprecated


This article covers a version of CentOS that is no longer supported. If you are currently operating a server running CentOS 6, we highly recommend upgrading or migrating to a supported version of CentOS.


Reason:
CentOS 6 reached end of life (EOL) on November 30th, 2020 and no longer receives security patches or updates. For this reason, this guide is no longer maintained.


See Instead:
This guide might still be useful as a reference, but may not work on other CentOS releases. If available, we strongly recommend using a guide written for the version of CentOS you are using.


The following DigitalOcean tutorial may be of immediate interest, as it outlines installing Ruby on Rails (albeit, with rbenv) on a CentOS 7 server:




- How To Install Ruby on Rails with rbenv on CentOS 7





## About Ruby on Rails


Ruby on Rails is an application stack that provides developers with a framework to quickly create a variety of web applications. Ruby on Rails does take a little while to install on a virtual private server, but luckily there are a lot of helpful tools to make this process as easy as possible.


You can run this tutorial on your droplet as a user with sudo privileges. You can check out how to set that up here, in steps 3 and 4: CentOS Server Setup


# Step One— Install Ruby with RVM


Before we do anything else, we should run a quick update to make sure that all of the packages we download are up to date:


```
sudo yum update
```


Once that's done, we can start installing RVM,  Ruby Version Manager. This is a great program that lets you use several versions of Ruby on one VPS; however, in this case, we will just use it to install the latest version of Ruby on the droplet.


If you do not have curl on your system, you can start by installing it:


```
sudo yum install curl
```


To install RVM, open terminal and type in this command:


```
curl -L get.rvm.io | bash -s stable
```


After it is done installing, load RVM.


```
# If you ran the installer as root, run:
source /usr/local/rvm/rvm.sh
# If you installed it through a user with access to sudo:
source ~/.rvm/rvm.sh
```


In order to work, RVM has some of its own dependancies that need to be installed. You can see what these are:


```
rvm requirements
```


In the text that RVM shows you, look for this paragraph.


```
Additional Dependencies:
# For Ruby / Ruby HEAD (MRI, Rubinius, & REE), install the following:
  ruby: yum install -y gcc-c++ patch readline readline-devel zlib zlib-devel libyaml-devel libffi-devel openssl-devel make bzip2 autoconf automake libtool bison iconv-devel ## NOTE: For centos >= 5.4 iconv-devel is provided by glibc
```


Go ahead and download the recommended dependancies, being careful not to use sudo. Instead, we should use rvmsudo:


```
rvmsudo yum install -y gcc-c++ patch readline readline-devel zlib zlib-devel libyaml-devel libffi-devel openssl-devel make bzip2 autoconf automake libtool bison iconv-devel
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


This process may take a while, be patient with it. Once it finishes you will have Ruby on Rails installed on your droplet.


# See More


Once you have installed Ruby on Rails on your VPS, you can proceed to Create a SSL Certificate for your site or Install an FTP server


By Etel Sverdlov
