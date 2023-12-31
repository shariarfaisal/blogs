# How To Use MySQL with Your Ruby on Rails Application on CentOS 7

```MySQL``` ```Ruby on Rails``` ```Ruby``` ```MariaDB``` ```CentOS```

## Introduction


Ruby on Rails uses sqlite3 as its default database, which works great in many cases, but may not be sufficient for your application. If your application requires the scalability, centralization, and control (or any other feature) that a client/server SQL database, such as PostgreSQL or MySQL, you will need to perform a few additional steps to get it up and running.


This tutorial will show you how to set up a development Ruby on Rails environment that will allow your applications to use a MySQL database, on an CentOS 7 or RHEL server. First, we will cover how to install MySQL and the MySQL adapter gem. Then we’ll show you how to create a rails application that uses MySQL as its database server.


# Prerequisites


This tutorial requires that have a working Ruby on Rails development environment. If you do not already have that, you may follow the tutorial in this link: How To Install Ruby on Rails with rbenv on CentOS 7.


You will also need to have access to a superuser, or sudo, account, so you can install the MySQL database software.


This guide also assumes that SELinux is disabled.


Once you’re ready, let’s install MySQL.


# Install MySQL


If you don’t already have MySQL installed, let’s do that now. CentOS uses MariaDB as a drop-in replacement for MySQL.


Install MySQL and its development libraries:


```
sudo yum install mariadb-server mariadb-devel

```


When the installation is complete, we need to start MySQL with the following command:


```
sudo systemctl start mariadb

```


Now that our MySQL database is running, we want to run a simple security script that will remove some dangerous defaults and lock down access to our database system a little bit. Start the interactive script by running:


```
sudo mysql_secure_installation

```


The prompt will ask you for your current root password. Since you just installed MySQL, you most likely won’t have one, so leave it blank by pressing enter. Then the prompt will ask you if you want to set a root password. Go ahead and hit Enter, and set the password. For the rest of the questions, you should simply hit the Enter key through each prompt to accept the default values. This will remove some sample users and databases, disable remote root logins, and load these new rules so that MySQL immediately respects the changes we have made.


The last thing you will want to do is enable MariaDB to start on boot. Use the following command to do so:


```
sudo systemctl enable mariadb.service

```


MySQL is now installed, but we still need to install the MySQL gem.


# Install MySQL Gem


Before your Rails application can connect to a MySQL server, you need to install the MySQL adapter. The mysql2 gem provides this functionality.


As the Rails user, install the mysql2 gem, like this:


```
gem install mysql2

```


Now your Rails applications can use MySQL databases.


# Create New Rails Application


Create a new Rails application in your home directory. Use the -d mysql option to set MySQL as the database, and be sure to substitute the highlighted word with your application name:


```
cd ~
rails new appname -d mysql

```


Then move into the application’s directory:


```
cd appname

```


The next step is to configure the application’s database connection.


## Configure Database Connection


If you followed the MySQL install instructions from this tutorial, you set a password for MySQL’s root user. The MySQL root login will be used to create your application’s test and development databases.


Open your application’s database configuration file in your favorite text editor. We’ll use vi:


```
vi config/database.yml

```


Under the default section, find the line that says “password:” and add the password to the end of it. It should look something like this (replace the highlighted part with your MySQL root password):


```
password: mysql_root_password

```


Save and exit.


## Create Application Databases


Create your application’s development and test databases by using this rake command:


```
rake db:create

```


This will create two databases in your MySQL server. For example, if your application’s name is “appname”, it will create databases called “appname_development” and “appname_test”.


If you get an error that says “Access denied for user ‘root’@‘localhost’ (using password: YES)Please provide the root password for your MySQL installation”, press Ctrl-c to quit. Then revisit the previous subsection (Configure Database Connection) to be sure that the password in database.yml is correct. After ensuring that the password is correct, try creating the application databases again.


# Test Configuration


The easiest way to test that your application is able to use the MySQL database is to try to run it.


For example, to run the development environment (the default), use this command:


```
rails server

```


This will start your Rails application on your localhost on port 3000.


If your Rails application is on a remote server, and you want to access it through a web browser, an easy way is to bind it to the public IP address of your server. First, look up the public IP address of your server, then use it with the rails server command like this:


```
rails server --binding=server_public_IP

```


Now you should be able to access your Rails application in a web browser via the server’s public IP address on port 3000:


```
http://server_public_IP:3000

```


If you see the “Welcome aboard” Ruby on Rails page, your application is properly configured, and connected to the MySQL database.


# Conclusion


You’re now ready to start development on your Ruby on Rails application, with MySQL as the database, on CentOS 7!


Good luck!


