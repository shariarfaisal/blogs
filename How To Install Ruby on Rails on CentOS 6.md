# How To Install Ruby on Rails on CentOS 6

```Ruby on Rails``` ```Ruby``` ```CentOS```










# Status: Deprecated


This article covers a version of CentOS that is no longer supported. If you are currently operating a server running CentOS 6, we highly recommend upgrading or migrating to a supported version of CentOS.


Reason:
CentOS 6 reached end of life (EOL) on November 30th, 2020 and no longer receives security patches or updates. For this reason, this guide is no longer maintained.


See Instead:
This guide might still be useful as a reference, but may not work on other CentOS releases. If available, we strongly recommend using a guide written for the version of CentOS you are using.


The following DigitalOcean tutorial may be of immediate interest, as it outlines how to install Ruby on Rails on a CentOS 7 server:




- How To Install Ruby on Rails with rbenv on CentOS 7





##  About Ruby on Rails


Ruby on Rails is an application stack that provides web developers with a framework to quickly create a variety of web applications.


There are three separate processes required to install Ruby on Rails: you need to install Ruby, then the Ruby Gems, then Rails.


# Set Up


The  steps in this tutorial require the user to have root privileges. You can see how to set that up in the  Initial Server Setup Tutorial in steps 3 and 4. 


# Step One—Install Ruby


The easiest way to install Ruby on your virtual server is through the yum package installer.


```
sudo yum install ruby
```


After you say yes to the prompt, Ruby will install.


Then we need some additional Ruby dependancies.


Type the following into terminal:


```
sudo yum install gcc g++ make automake autoconf curl-devel openssl-devel zlib-devel httpd-devel apr-devel apr-util-devel sqlite-devel
sudo yum install ruby-rdoc ruby-devel
```


While it processes, the prompt may ask your permission to install the various packages. Go ahead and say yes each time.


# Step Two—Install Ruby Gems


Once you have Ruby installed, you can easily install the ruby gems.


Type this command into terminal:


```
sudo yum install rubygems
```


After you have agreed to the prompt, ruby gems should be installed on your VPS. However, if you have any issues with this process, you can use an alternate method to install the Ruby Gems.


# How to Install RubyGems from Source


If, for some reason, the yum installer doesn’t work for you, you can follow these steps to install ruby gems from the source.


To start, we will create a new directory to store the ruby code.


```
sudo mkdir ~/src
sudo cd ~/src
```


Then we can go ahead and download the ruby gems into the new folder. (You can always access the latest available gems by visiting:


http://rubygems.org/pages/download)


```
wget http://rubyforge.org/frs/download.php/69365/rubygems-1.3.6.tgz?tar -zxvf rubygems-1.3.6.tgz
cd rubygems-1.3.6
```


Finally you are ready to install the ruby gems.


```
sudo ruby setup.rb
```


After you answer yes to the prompt, the gems will finish installing.


You will now be ready to install Rails on your virtual server.


# Step Three—Install Rails


We should quickly do two updates to makes sure that everything is up-to date and set up correctly:


Check the gems:


```
sudo gem update
```


Then check the system overall:


```
sudo gem update --system
```


Once everything has processed, it is time to install Rails.


To start, open terminal and type in:


```
sudo gem install rails
```


This process may take a while, be patient with it.


If you are feeling especially antsy, you can type in:


```
sudo gem install rails -V
```


Terminal will then show you all the details of the process, so you can see that it is working.


After you answer yes to the prompt that comes up, rails will finish installing.


You have successfully installed Ruby on Rails!


# See More


Once you have set up Ruby on Rails installed on your VPS, you can proceed to Create a SSL Certificate for your site or Install an FTP server


By Etel Sverdlov
