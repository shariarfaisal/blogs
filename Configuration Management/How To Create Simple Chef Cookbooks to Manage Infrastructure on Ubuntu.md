# How To Create Simple Chef Cookbooks to Manage Infrastructure on Ubuntu

```Ubuntu``` ```Configuration Management``` ```Chef``` ```Nginx```

## Introduction



Chef is a configuration management system designed to allow you to automate and control vast numbers of computers in an automated, reliable, and scalable manner.


In previous tutorials, we have looked at some common Chef terminology and discussed how to install a Chef server, workstation, and nodes (with Chef 12 or Chef 11).  In this guide, we will use these guides as a jumping off point to begin talking about how to automate your environment.


In this article, we will discuss the basics of creating a Chef cookbook.  Cookbooks are the configuration units that allow us to configure and perform specific tasks within Chef on our remote nodes.  We build cookbooks and then tell Chef which nodes we want to run the steps outlined in the cookbook.


In this guide, we will assume that you are starting with the three machines that we ended the last lesson with.  You should have a server, a workstation, and at least one node to push configuration changes to.


# Basic Cookbook Concepts



Cookbooks serve as the fundamental unit of configuration and policy details that Chef uses to bring a node into a specific state.  This just means that Chef uses cookbooks to perform work and make sure things are as they should be on the node.


Cookbooks are usually used to handle one specific service, application, or functionality.  For instance, a cookbook can be created to use NTP to set and sync the node’s time with a specific server.  It may install and configure a database application.  Cookbooks are basically packages for infrastructure choices.


Cookbooks are created on the workstation and then uploaded to a Chef server.  From there, recipes and policies described within the cookbook can be assigned to nodes as part of the node’s “run-list”.  A run-list is a sequential list of recipes and roles that are run on a node by chef-client in order to bring the node into compliance with the policy you set for it.


In this way, the configuration details that you write in your cookbook are applied to the nodes you want to adhere to the scenario described in the cookbook.


Cookbooks are organized in a directory structure that is completely self-contained.  There are many different directories and files that are used for different purposes.  Let’s go over some of the more important ones now.


## Recipes



A recipe is the main workhorse of the cookbook.  A cookbook can contain more than one recipe, or depend on outside recipes.  Recipes are used to declare the state of different resources.


Chef resources describe a part of the system and its desired state.  For instance, a resource could say “the package x should be installed”.  Another resource may say “the x service should be running”.


A recipe is a list related resources that tell Chef how the system should look if it implements the recipe.  When Chef runs the recipe, it checks each resource for compliance to the declared state.  If the system matches, it moves on to the next resource, otherwise, it attempts to move the resource into the given state.


Resources can be of many different types.  You can learn about the different resource types here.  Some common ones are:


- package: Used to manage packages on a node
- service: Used to manage services on a node
- user: Manage users on the node
- group: Manage groups
- template: Manage files with embedded ruby templates
- cookbook_file: Transfer files from the files subdirectory in the cookbook to a location on the node
- file: Manage contents of a file on node
- directory: Manage directories on node
- execute: Execute a command on the node
- cron: Edit an existing cron file on the node

## Attributes



Attributes in Chef are basically settings.  Think of them as simple key-value pairs for anything you might want to use in your cookbook.


There are several different kinds of attributes that can be applied, each with a different level of precedence over the final settings that a node operates under.  At the cookbook level, we generally define the default attributes of the service or system we are configuring.  These can be overridden later by more specific values for a specific node.


When creating a cookbook, we can set attributes for our service in the attributes subdirectory of our cookbook.  We can then reference these values in other parts of our cookbook.


## Files



The files subdirectory within the cookbook contains any static files that we will be placing on the nodes that use the cookbook.


For instance, any simple configuration files that we are not likely to modify can be placed, in their entirety, in the files subdirectory.  A recipe can then declare a resource that moves the files from that directory into their final location on the node.


## Templates



Templates are similar to files, but they are not static.  Template files end with the .erb extension, meaning that they contain embedded Ruby.


These are mainly used to substitute attribute values into the file to create the final file version that will be placed on the node.


For example, if we have an attribute that defines the default port for a service, the template file can call to insert the attribute at the point in the file where the port is declared.  Using this technique, you can easily create configuration files, while keeping the actual variables that you wish to change elsewhere.


## Metadata.rb



The metadata.rb file is used, not surprisingly, to manage the metadata about a package.  This includes things like the name of the package, a description, etc.


It also includes things like dependency information, where you can specify which cookbooks this cookbook needs to operate.  This will allow the Chef server to build the run-list for the nodes correctly and ensure that all of the pieces are transfered correctly.


# Create a Simple Cookbook



To demonstrate some of the work flow involved in working with cookbooks, we will create a cookbook of our own.  This will be a very simple cookbook that installs and configures the Nginx web server on our node.


To begin, we need to go to our ~/chef-repo directory on our workstation:


```
cd ~/chef-repo

```


Once there, we can create a cookbook by using knife.  As we mentioned in previous guides, knife is a tool used to configure most interactions with the Chef system.  We can use it to perform work on our workstation and also to connect with the Chef server or individual nodes.


The general syntax for creating a cookbook is:


<pre>
knife cookbook create <span class=“highlight”>cookbook_name</span>
</pre>


Since our cookbook will deal with installing and configuring Nginx, we will name our cookbook appropriately:


```
knife cookbook create nginx

```



```
** Creating cookbook nginx
** Creating README for cookbook: nginx
** Creating CHANGELOG for cookbook: nginx
** Creating metadata for cookbook: nginx

```


What knife does here is builds a simple structure within our cookbooks directory for our new cookbook.  We can see our cookbook structure by navigating into the cookbooks directory, and into the directory with the cookbook name.


```
cd cookbooks/nginx
ls

```



```
attributes  CHANGELOG.md  definitions  files  libraries  metadata.rb  providers  README.md  recipes  resources	templates

```


As you can see, this has created a folder and file structure that we can use to build our cookbook.  Let’s begin with the biggest chunk of the configuration, the recipe.


## Create a Simple Recipe



If we go into the recipes subdirectory, we can see that there is already a file called default.rb inside:


```
cd recipes
ls

```



```
default.rb

```


This is the recipe that will be run if you reference the “nginx” recipe.  This is where we will be adding our code.


Open the file with your text editor:


```
nano default.rb

```



```
#
# Cookbook Name:: nginx
# Recipe:: default
#
# Copyright 2014, YOUR_COMPANY_NAME
#
# All rights reserved - Do Not Redistribute
#

```


The only thing that is in this file currently is a comment header.


We can begin by planning the things that need to happen for our Nginx web server to get up and running the way that we want it to.  We do this by configuring “resources”.  Resources do not describe how to do something; they simply describe what a part of the system should look like when it is complete.


First of all, we obviously need to make sure the software is installed.  We can do this by creating a “package” resource first.


```
package 'nginx' do
  action :install
end

```


This little piece of code defines a package resource for Nginx.  The first line begins with the type of resource (package) and the name of the resource (‘nginx’).  The rest is a group of actions and parameters that declare what we want to happen with the resource.


In this resource, we see action :install.  This line tells Chef that the resource we are describing should be installed.  The node that runs this recipe will check that Nginx is installed.  If it is, it will check that off the list of things to do.  If not, it will install the program using the methods available on the client system and then check it off.


After we install the service, we probably want to adjust its current state on the node.  By default, Ubuntu does not start Nginx after installation, so we will want to change that:


```
service 'nginx' do
  action [ :enable, :start ]
end

```


Here, we see a resource of the “service” type.  This declares that for the Nginx service component (the part that allows us to manage the server with init or upstart), we want to start the service right now, and also enable it to start automatically when the machine is restarted.


The final resource we will be declaring is the actual file that we will be serving.  Since this is just a simple file that we will not be modifying, we can simply declare the location where we want the file and tell it where in the cookbook to get the file:


```
cookbook_file "/usr/share/nginx/www/index.html" do
  source "index.html"
  mode "0644"
end

```


We use the “cookbook_file” resource type to tell Chef that this file is available within the cookbook itself and can be transfered as-is to the location.  In our example, we are transferring a file into Nginx’s document root.


In our case, we specify the file name that we are trying to create in the first line.  In the “source” line, we tell it the name of the file to look for within the cookbook.  Chef looks for this file within the “files/default” subdirectory in the cookbook.


The “mode” line sets the permissions on the file we are creating.  In this case, we are allowing the root user read and write permissions and everyone else read permissions.


Save and close this file when you are finished.


## Creating the Index file



As you saw above, we defined a “cookbook_file” resource which should move a file called “index.html” into the document root on the node.  We need to create this file.


We should put this file in the “files/default” subdirectory of our cookbook.  Go there now by typing:


```
cd ~/chef-repo/cookbooks/nginx/files/default

```


Inside this directory, we will create the file we referenced:


```
nano index.html

```


This file will just be a really simple HTML document meant to demonstrate that our resources have operated the way we wanted them to.


Paste this into the file:


```
<html>
  <head>
    <title>Hello there</title>
  </head>
  <body>
    <h1>This is a test</h1>
    <p>Please work!</p>
  </body>
</html>

```


Save and close the file when you are finished.


## Create a Helper Cookbook



Before we go any further, let’s preemptively solve a small problem.  When our node tries to run the cookbook that we’ve created as it is now, chances are, it will fail.


That is because it will attempt to install Nginx from the Ubuntu repositories, and the package database on our node is most likely out-of-date.  Usually, we run “sudo apt-get update” prior to running package commands.


To address this issue, we can create a simple cookbook whose only purpose is to ensure that the package database is updated.


We can do this using the same knife syntax we used before.  Let’s call this cookbook “apt”:


```
knife cookbook create apt

```


This will create the same kind of directory structure that we had when we first started with our Nginx cookbook.


Let’s cut straight to the chase and edit the default recipe for our new cookbook.


```
nano ~/chef-repo/cookbooks/apt/recipes/default.rb

```


In this file, we will declare an “execute” resource.  This is simply a way of defining a command that we want to run on the node.


Our resource looks like this:


```
execute "apt-get update" do
  command "apt-get update"
end

```


The first line gives a name for our resource.  In our case, we are calling the resource this for simplicity’s sake.  If the “command” attribute is defined (as we have done), then this is the actual command that is executed.


Since these are exactly the same, it does not matter in the slightest.


Save and close the file.


Now that we have our new cookbook, there are a number of ways that we can make sure that we execute this before our Nginx cookbook.  We could add it to the node’s run-list before the Nginx cookbook, but we can also tie it into the Nginx cookbook itself.


This is probably the better option because we will not have to remember to add the “apt” cookbook before the “nginx” cookbook on every node we want to configure for Nginx.


We need to adjust a few things in the Nginx cookbook to make this happen.  First, let’s open the Nginx recipe file again:


```
nano ~/chef-repo/cookbooks/nginx/recipes/default.rb

```


At the top of this cookbook, before the other resources that we have defined, we can read in the “apt” default recipe by typing:


<pre>
<span class=“highlight”>include_recipe “apt”</span>


package ‘nginx’ do
action :install
end


service ‘nginx’ do
action [ :enable, :start ]
end


cookbook_file “/usr/share/nginx/www/index.html” do
source “index.html”
mode “0644”
end
</pre>


Save and close the file.


The other file that we need to edit is the metadata.rb file.  This file is checked when the Chef server sends the run-list to the node, to see which other recipes should be added to the run-list.


Open the file now:


```
nano ~/chef-repo/cookbooks/nginx/metadata.rb

```


At the bottom of the file, you can add this line:


<pre>
name             ‘nginx’
maintainer       ‘YOUR_COMPANY_NAME’
maintainer_email ‘YOUR_EMAIL’
license          ‘All rights reserved’
description      ‘Installs/Configures nginx’
long_description IO.read(File.join(File.dirname(FILE), ‘README.md’))
version          ‘0.1.0’


<span class=“highlight”>depends “apt”</span>
</pre>


With that finished, our Nginx cookbook now relies on our apt cookbook to take care of the package database update.


# Add the Cookbook to your Node



Now that our basic cookbooks are complete, we can upload them to our chef server.


We can do that individually by typing:


```
knife cookbook upload apt
knife cookbook upload nginx

```


Or, we can upload everything by typing:


```
knife cookbook upload -a

```


Either way, our recipes will be uploaded to the Chef server.


Now, we can modify the run-list of our nodes.   We can do this easily by typing:


<pre>
knife node edit <span class=“highlight”>name_of_node</span>
</pre>


If you need to find the name of your available nodes, you can type:


```
knife node list

```



```
client1

```


For our purposes, when we type this, we get a file that looks like this:


```
knife node edit client1

```



```
{
  "name": "client1",
  "chef_environment": "_default",
  "normal": {
    "tags": [

    ]
  },
  "run_list": [

  ]
}

```


You may need to set your EDITOR environmental variable before this works.  You can do this by typing:


<pre>
export EDITOR=<span class=“highlight”>name_of_editor</span>
</pre>


As you can see, this is a simple JSON document that describes some aspects of our node.  We can see a “run_list” array, which is currently empty.


We can add our Nginx cookbook to that array using the format:


<pre>
“recipe[<span class=“highlight”>name_of_recipe</span>]”
</pre>


When we are finished, our file should look like this:


```
{
  "name": "client1",
  "chef_environment": "_default",
  "normal": {
    "tags": [

    ]
  },
  "run_list": [
    "recipe[nginx]"
  ]
}

```


Save and close the file to implement the new settings.


Now, we can SSH into our node and run the Chef client software.  This will cause the client to check into the Chef server.  Once it does this, it will see the new run-list that has been assigned it.


SSH into your node and then run this:


```
sudo chef-client

```



```
Starting Chef Client, version 11.8.2
resolving cookbooks for run list: ["nginx"]
Synchronizing Cookbooks:
  - apt
  - nginx
Compiling Cookbooks...
Converging 4 resources
Recipe: apt::default
  * execute[apt-get update] action run
    - execute apt-get update

Recipe: nginx::default
  * package[nginx] action install (up to date)
  * service[nginx] action enable
    - enable service service[nginx]

  * service[nginx] action start (up to date)
  * cookbook_file[/usr/share/nginx/www/index.html] action create (up to date)
Chef Client finished, 2 resources updated

```


As you can see, our apt cookbook was sent over and run as well, even though it wasn’t in the run-list we created.  That is because Chef intelligently resolved dependencies and modified the actual run-list before executing it on the node.


Note: There are various methods of ensuring that one cookbook or recipe is run before another.  Adding a dependency is only one choice, and other methods may be preferred.


We can verify that this works by going to our node’s IP address or domain name:


<pre>
http://<span class=“highlight”>node_domain_or_IP</span>
</pre>


You should see something that looks like this:





Congratulations, you have configured your first node using Chef cookbooks!


# Conclusion



Although this was a very simple example that probably didn’t save you much time over configuring your server manually, hopefully you can begin to see the possibilities of this method of building infrastructure.


Not only does it allow for rapid deployment and configuration of different kinds of servers, it ensures that you know the exact configuration of all of your machines.  This lets you validate and test your infrastructure, and also gives you the framework you need to quickly redeploy your infrastructure on a whim.


<div class=“author”>By Justin Ellingwood</div>


