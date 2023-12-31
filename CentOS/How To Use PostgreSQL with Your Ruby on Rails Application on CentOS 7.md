# How To Use PostgreSQL with Your Ruby on Rails Application on CentOS 7

```PostgreSQL``` ```Ruby on Rails``` ```Ruby``` ```CentOS```

## Introduction


Ruby on Rails uses sqlite3 as its default database, which works great in many cases, but may not be sufficient for your application. If your application requires the scalability, centralization, and control (or any other feature) that a client/server SQL database, such as PostgreSQL or MySQL, you will need to perform a few additional steps to get it up and running.


This tutorial will show you how to set up a development Ruby on Rails environment that will allow your applications to use a PostgreSQL database, on an CentOS 7 or RHEL server. First, we will cover how to install and configure PostgreSQL. Then we’ll show you how to create a rails application that uses PostgreSQL as its database server.


# Prerequisites


This tutorial requires that have a working Ruby on Rails development environment. If you do not already have that, you may follow the tutorial in this link: How To Install Ruby on Rails with rbenv on CentOS 7.


You will also need to have access to a superuser, or sudo, account, so you can install the PostgreSQL database software.


This guide also assumes that SELinux is disabled.


Once you’re ready, let’s install PostgreSQL.


# Install PostgreSQL


If you don’t already have PostgreSQL installed, let’s do that now.


If you haven’t already done so, add the EPEL repository to yum with this command:


```
sudo yum install epel-release


```


Install PostgreSQL server and its development libraries:


```
sudo yum install postgresql-server postgresql-contrib postgresql-devel


```


PostgreSQL is installed but we still need to do some basic configuration.


Create a new PostgreSQL database cluster:


```
sudo postgresql-setup initdb


```


By default, PostgreSQL does not allow password authentication. We will change that by editing its host-based authentication configuration.


Open the HBA configuration with your favorite text editor. We will use vi:


```
sudo vi /var/lib/pgsql/data/pg_hba.conf


```


Find the lines that looks like this, near the bottom of the file:


pg_hba.conf excerpt (original)
```
host    all             all             127.0.0.1/32            ident
host    all             all             ::1/128                 ident

```


Then replace “ident” with “md5”, so they look like this:


pg_hba.conf excerpt (updated)
```
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5

```


Save and exit. PostgreSQL is now configured to allow password authentication.


Now start and enable PostgreSQL:


```
sudo systemctl start postgresql
sudo systemctl enable postgresql


```


PostgreSQL is now installed but you should create a new database user, that your Rails application will use.


# Create Database User


First, change to the postgres system user:


```
sudo su - postgres


```


Create a PostgreSQL superuser user with this command (substitute the highlighted word with your own username):


```
createuser -s pguser


```


To set a password for the database user, enter the PostgreSQL console with this command:


```
psql


```


The PostgreSQL console is indicated by the postgres=# prompt. At the PostgreSQL prompt, enter this command to set the password for the database user that you created:


```
\password pguser


```


Enter your desired password at the prompt, and confirm it.


Now you may exit the PostgreSQL console by entering this command:


```
\q


```


Now that your PostgreSQL user is set up, switch back to your normal user:


```
exit


```


Let’s create a Rails application now.


# Create New Rails Application


Create a new Rails application in your home directory. Use the -d postgresql option to set PostgreSQL as the database, and be sure to substitute the highlighted word with your application name:


```
cd ~
rails new appname -d postgresql


```


Then move into the application’s directory:


```
cd appname


```


The next step is to configure the application’s database connection.


## Configure Database Connection


The PostgreSQL user that you created will be used to create your application’s test and development databases. We need to configure the proper database settings for your application.


Open your application’s database configuration file in your favorite text editor. We’ll use vi:


```
vi config/database.yml


```


Under the default section, find the line that says “pool: 5” and add the following lines under it. It should look something like this (replace the highlighted parts with your PostgreSQL user and password):


config/database.yml excerpt
```
  host: localhost
  username: pguser
  password: pguser_password

```


Save and exit.


## Create Application Databases


Create your application’s development and test databases by using this rake command:


```
rake db:create


```


This will create two databases in your PostgreSQL server. For example, if your application’s name is “appname”, it will create databases called “appname_development” and “appname_test”.


If you get an error at this point, revisit the previous subsection (Configure Database Connection) to be sure that the host, username, and password in database.yml are correct. After ensuring that the database information is correct, try creating the application databases again.


# Test Configuration


The easiest way to test that your application is able to use the PostgreSQL database is to try to run it.


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


Visit in a web browser:
```
http://server_public_IP:3000

```


If you see the “Welcome aboard” Ruby on Rails page, your application is properly configured, and connected to the PostgreSQL database.


# Conclusion


You’re now ready to start development on your Ruby on Rails application, with PostgreSQL as the database, on CentOS 7!


Good luck!


