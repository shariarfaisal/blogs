# How To Use Capistrano to Automate Deployments  Getting Started

```Miscellaneous``` ```Ruby``` ```CentOS```

## Introduction



One of the key areas of producing web-based applications, and where many major companies pride themselves, is deployment. More precisely, how to deploy. This task, which some indeed regard as a chore, might seem to add little or no direct or additional value to your project whatsoever. However, a well crafted process [of deployment] certainly helps to reduce the overheads, such as time being wasted trying to get products on-line.


Unless you have a very specific (and changing) requirements, with absolutely domain-focused needs, when the time comes to putting your application online, taking advantage of the various dedicated tools, automation methods or scripts will come to your aid to get back to your actual work of developing quicker – greatly!


In this DigitalOcean article, we will be taking a good look at Capistrano: a Ruby based remote server automation tool which can be easily used to automate mundane deployment and system management tasks. Using Capistrano, you can almost entirely automate all actions you would normally take to get your product live.


# Table of Contents



## 1. Capistrano



1. Ruby Programming Language
2. Capistrano Recipes
3. System/Server Administration
4. Application Deployment

## 2. Installing Capistrano



1. Preparing The System
2. Installing Ruby
3. Installing Capistrano

## 3. Getting Started With Capistrano



1. Capistrano Basics
2. Initiating Capistrano Inside A Project
3. Creating Users For Deploying With Capistrano

# Capistrano



Capistrano, as mentioned in our introduction, is a Ruby based, open-source server management tool. Although it might come out as just another alternative to many existing automation solutions, it is an excellent one to use thanks to its great [advanced] features.


Similar to other automation libraries, using Capistrano arbitrary functions can be performed on the virtual server without direct interference - by having Capistrano execute a script (i.e. a recipe). However, in general, you can think of this tool as your very own deployment assistant, helping you with almost anything from getting the code on your deployment machine to bootstrapping the deployment process – and it can do this on multiple systems at once or in a round-robin fashion.


Looking at many tutorials on the internet, you might get the impression that Capistrano is the perfect framework for RoR. However, despite being a Ruby focused framework (or tool), you can safely use it to handle many different kind of deployment scenarios with its recipes, including deploying PHP web-applications.


## Ruby Programming Language



Ruby is a general-purpose (i.e. not created to solve a specific problem), dynamic programming language that has gained a significant popularity with the release of Ruby-on-Rails web-application development framework.


The concise and ordered way one can use Ruby to write scripts (thanks to the way the language is designed) helped the language gain huge momentum. Coupled with the goals and mentality of RoR framework, and features it provided as an Object Oriented Programming (OOP) language (compared to competitors available at the time), Ruby became one of the most widely preferred languages in the last decade.


Capistrano, being a Ruby based tool, offers its users the possibility to take advantage of Ruby’s clean and clear syntax when compiling its recipes for deployments.


## Capistrano Recipes



Recipes in Capistrano lingo translate to files which contain operative directions for deploying (or managing) applications and servers. These recipes can be modified to support a great variety of language specific deployments that are not Ruby (or Rails) related. You can think of them as a script which Capistrano uses to perform its actions.


## System/Server Administration



If you are wondering in what situations Capistrano might come on in handy, below you can find a few examples.


System and server administration jobs (usually) include pretty much everything that relates to:


- 
Building a server

- 
Installing applications

- 
Maintenance of systems running these applications

- 
And monitoring


When you begin working with your very own VPS (which is a fully-fledged virtualised server with full control / access), things that appear as a mystery will quickly start to become familiar to you.


As your application starts to gain some popularity and things begin to grow, the need of managing multiple droplets and repeating everything over and over again ceases to become fun. When you deploy your applications and deal with their maintenance, it is only natural to expect that you will be running into some issues - especially overheads and time-wasters.


Capistrano can help them with most, if not all of them - beginning with application deployment.


## Application Deployment



Deploying an application (regardless of it being a web site, an API, or a server) usually means setting up a system from scratch (or from a snapshot taken in time), preparing it by updating everything, downloading dependencies, setting up the file structure and permissions followed by finally uploading your codebase - or downloading it using a source control manager (SCM) such as Git.


During the development process, chances are you will have commands that need to be routinely executed during each step (e.g. right before entering a deployment cycle).


Being able to script these tasks (both local and remote), in a logically organized and – most importantly – programmable manner proves to be invaluable shortly after you realise how much time is being wasted repeating the same steps constantly, rendering everything error-prone during the process.


# Installing Capistrano



Note: In this article, we are focusing on installing Capistrano on a VPS, running on CentOS 6.5 operating system. If you are working with another sort (e.g. Ubuntu), same logic will apply but you are recommended the check the official Capistrano documentation here for installation.


Note: This section, whereby we set up currently available, latest Ruby version is a summary of our dedicated article on the subject - How To Install Ruby 2.1.0 On CentOS 6.5.


## Preparing The System



In order to install Ruby (and Capistrano), we need to prepare our minimally shipped CentOS droplet, equipping it with development tools made for purposes of installing other applications and tools (e.g. a compiler to install Ruby from source).


Let’s begin with updating our system.


Run the following to update the default tools of you CentOS based droplet:


```
yum -y update

```


Install the bundle containing development tools by executing the following command:


```
yum groupinstall -y 'development tools'

```


## Installing Ruby



We are going to be using Ruby Version Manager, RVM, to download and install “rubies” (a Ruby interpreter, as referred by the RVM).


Run the following two commands to install RVM and create a system environment for Ruby:


```
curl -L get.rvm.io | bash -s stable
source /etc/profile.d/rvm.sh

```


Finally, to finish getting Ruby on our system, let’s get RVM to download and install Ruby version 2.1.0:


```
rvm reload
rvm install 2.1.0

```


In order to verify that Ruby is indeed installed and set up, run the following:


```
ruby --version 
# ruby 2.1.0p0 (2013-12-25 revision 44422) [i686-linux]

```


## Installing Capistrano



Once we have our system ready, getting Capistrano’s latest version thanks to RubyGems is a breeze.


You can simply use the following to get Capistrano version 3:


```
gem install capistrano

```


If you would like to work with the absolutely latest version, you can link to the Github repository:


```
git clone https://github.com/capistrano/capistrano.git
cd capistrano
gem build *.gemspec
gem install *.gem

```


You can verify your Capistrano installation in a similar way to Ruby’s:


```
cap --version
# Capistrano Version: 3.1.0 (Rake Version: 10.1.0)

```


# Getting Started With Capistrano



Once all the necessary components are set up and ready, we can continue with the basics of Capistrano in this last section of our getting started article.


## Capistrano Basics



The key to get working with Capistrano is committing your project to an external Git repository where it can be downloaded during deployment.


You can choose any provider such as Github to do so.


Alternatively, you can see DigitalOcean’s community articles on Git by visiting here to learn about hosting a private Git repository on a VPS or to learn about how to work with Git.


Note: As recommended by Capistrano, you should not contain any sensitive information (e.g. secure credentials for database connections) inside your repository.


## Initiating Capistrano Inside A Project



Initiating Capistrano version 3 slightly differs from version 2 and consists of the following command:


```
# Usage:
# Enter the project directory: cd [project-name]
# Initiate Capistrano:         cap install
cd  myapp
cap install

```


## Creating Users For Deploying With Capistrano



When using Capistrano for deployment, the good way of executing the recipes is by using a user other than default root. To begin with, we will create a deployers group, and give them the permission to proceed.


To add a new group to your droplet, run the following:


```
groupadd deployers

```


Now we can continue with adding users to our deployers group with privileged access.


Let’s add deployer as a deployer:


```
# Usage: sudo usermod -a -G deployers [name]
sudo usermod -a -G deployers deployer

```


Finally, to give the deployers group the permissions, run the following and edit the /etc/sudoers file:


```
nano /etc/sudoers

```


Add the following line to after the groups:


```
..
## Allows people in group wheel to run all commands
%deployers      ALL=(ALL) ALL

..

```


## Further Information



Note: To learn more about SSH and sudo, check out DigitalOcean community articles on Linux Basics.


And that’s it! We are now ready to use Capistrano for deployments. Move on with our next Capistrano articles to see how to use this tool in various deployment scenarios.


<div class=“author”>Submitted by: <a
href=“https://twitter.com/ostezer”>O.S. Tezer</a></div>


