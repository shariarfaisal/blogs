# How to Run Django with mod_wsgi and Apache with a virtualenv Python environment on a Debian VPS

```Apache``` ```Django``` ```Debian```

## Introduction


Working with Django applications, and Python applications in general, is a complex matter with many tools in use. There are multiple ways of achieving the same goal and often there is no single way to do things.


One of the most popular ways of deploying Django applications to the web on a dedicated server is to use Nginx paired with Gunicorn. A great way to do that has already been described in depth in this article. It is however a quite popular scenario to host Django applications alongside existing websites served using Apache. We will try to cover the quick route to achieve that particular goal. Please note, however, that this is not a definitive guide to Django and Apache pairing and there are configuration aspects not covered here.


This text will make several assumptions:


- 
You have already set up your droplet with Debian 7.0 or later. There are many differences between different Linux distributions; therefore for the sake of clarity we will focus on a Debian server.

- 
You are at least somewhat familiar with common Python tools such as pip package manager and virtualenv for making virtual environments. These tools are wonderfully explained in this article.

- 
You are at least somewhat familiar with Django project structure, as this article is not intended to walk through using and configuring Django itself.

- 
You are familiar with basic Apache administration, as this tutorial will cover only the simple installation of the server itself and necessary configuration changes to pair Django with Apache.


# Prerequisites


Before installing new packages, it is always a good practice to update your system packages and package indexes. To do that execute:


```
apt-get update
apt-get upgrade

```


# Installing Apache


Since this text focuses on using Apache to serve the application, the server itself is necessary. To install the necessary packages execute:


```
apt-get install apache2

```


Straight after installation Apache will already be running. You can check whether the Apache web server has been properly set-up by opening your web browser and pointing it to the server IP address. You should see a simple It works! page on the screen.


# Installing pip and virtualenv


To begin working with Python and Django on a webserver, pip and virtualenv must be installed first. Pip is a Python package manager that facilitates installing Python software packages such as Django itself, whereas virtualenv makes it possible to create separate virtual environments for Python applications in order to separate libraries needed for different applications and avoid version clash between them.


To do that execute:


```
apt-get install python-pip python-virtualenv

```


This command will install pip and virtualenv from the Debian package repository. You can verify that both tools have been properly installed by running them with --version switch.


```
root@django:~# virtualenv --version
1.7.1.2
root@django:~# pip --version
pip 1.1 from /usr/lib/python2.7/dist-packages (python 2.7)
root@django:~#

```


# Creating a virtual environment using virtualenv


Upon Apache installation a /var/www directory is automatically created in which the default web server root is set up. We will put our new Django application there with all its dependencies.


Let’s create a new directory called sampleapp inside that directory and enter the new directory:


```
cd /var/www
mkdir sampleapp
cd sampleapp

```


Then let’s create a new virtual environment using virtualenv. A Python virtual environment is basically a directory in which the Python interpreter and a local instance of pip resides. The local instance of pip installs all packages inside the virtual environment. That way no installed packages pollute global Python installation and also there is no possibility of package version clash in a hypothetical scenario of two applications running two different versions of Django or any other library.


To create a new virtual environment enter:


```
virtualenv env

```


where env is the virtual environment name - it could be any other word. The output from this command should look like this:


```
root@django:/var/www/sampleapp# virtualenv env
New python executable in env/bin/python
Installing distribute.............................................................................................................................................................................................done.
Installing pip...............done.
root@django:/var/www/sampleapp#

```


The virtual environment is now ready and can be used in two different ways.


One way is to run commands using virtual environment interpreter directly. With this method it is necessary to always remember to execute the correct interpreter or pip instance, as there is a possibility to run a system-wide one.


```
root@django:/var/www/sampleapp# env/bin/pip --version
pip 1.1 from /var/www/sampleapp/env/lib/python2.7/site-packages/pip-1.1-py2.7.egg (python 2.7)
root@django:/var/www/sampleapp# env/bin/python --version
Python 2.7.3
root@django:/var/www/sampleapp#

```


The other way is to activate the environment first, using


```
source env/bin/activate

```


the environment name will then be prepended to the command line as such


```
root@django:/var/www/sampleapp# source env/bin/activate
(env)root@django:/var/www/sampleapp#

```


and all commands executed will be using local virtual environment versions


```
(env)root@django:/var/www/sampleapp# pip --version
pip 1.1 from /var/www/sampleapp/env/lib/python2.7/site-packages/pip-1.1-py2.7.egg (python 2.7)
(env)root@django:/var/www/sampleapp# python --version
Python 2.7.3
(env)root@django:/var/www/sampleapp#

```


it is easier to work that way; however it is necessary to deactivate the environment after the work is done using the following command


```
deactivate

```


it will return the shell to normal


```
(env)root@django:/var/www/sampleapp# deactivate
root@django:/var/www/sampleapp#

```


The freshly created environment will be used to store all necessary dependencies, including Django and related libraries. It will also be used by Apache and mod_wsgi later on to serve the application using correct dependencies.


# Installing Django inside virtual environment


Next necessary step is to install Django inside the virtual environment. Let’s do that without activating the environment beforehand using:


```
env/bin/pip install django

```


The last messages shown after executing this command should look like this


```
Successfully installed django
Cleaning up...

```


Django is now installed inside virtual environment and is not available from within system-wide Python installation. You can verify that behavior by importing django module using both interpreters


```
root@django:/var/www/sampleapp# python
Python 2.7.3 (default, Mar 13 2014, 11:03:55)
[GCC 4.7.2] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import django
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ImportError: No module named django
>>> exit()

```


Import using system-wide interpreter failed, whereas


```
root@django:/var/www/sampleapp# env/bin/python
Python 2.7.3 (default, Mar 13 2014, 11:03:55)
[GCC 4.7.2] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import django
>>>

```


the one executed inside the virtual environment succeeded.


# Creating first Django project


To create a simple, basic example project we can use django-admin.py script as follows


```
 env/bin/django-admin.py startproject sampleapp .

```


Please note the trailing . in the command - without it the project will be created in an additional subdirectory. After executing that command a new sampleapp directory and manage.py script will be created in /var/www/sampleapp.  The manage.py script is used to execute Django commands for this particular project. One of the possible uses of manage.py is to run a test server instance to verify that everything is working as intended.


Please execute:


```
env/bin/python manage.py runserver 0.0.0.0:8000

```


This will run a test server bound to all interfaces on port 8000. The output should look like this:


```
Validating models...

0 errors found
April 08, 2014 - 12:29:31
Django version 1.6.2, using settings 'sampleapp.settings'
Starting development server at http://0.0.0.0:8000/
Quit the server with CONTROL-C.

```


If you open your server IP address with port 8000 in your browser (the address should look like http://<ip address>:8000/) you should see the It worked! example Django page. This is the result we will work towards using Apache web server instead of the built-in Django development server.


Since the Django application is working properly, we can proceed to pair the application with Apache.


# Installing mod_wsgi for Apache


The easiest and also recommended way to serve Python applications using Apache is to use mod_wsgi module. It is not installed by default with neither Python nor Apache, so we have to install an additional package.


```
apt-get install libapache2-mod-wsgi

```


The next step will be to configure default Apache virtual host that at the beginning of the article served It works! page to serve our Django application.


# Configuring mod_wsgi in a default virtual host


The idea behind configuring mod_wsgi for any other virtual host in Apache is the same as the one presented here. We will use the default virtual host for simplicity, since it is the one already provided by a clean Apache installation.


Open the default virtual host configuration file in nano editor


```
nano /etc/apache2/sites-enabled/000-default 

```


and add three following lines just below <VirtualHost *:80>


```
WSGIDaemonProcess sampleapp python-path=/var/www/sampleapp:/var/www/sampleapp/env/lib/python2.7/site-packages
WSGIProcessGroup sampleapp
WSGIScriptAlias / /var/www/sampleapp/sampleapp/wsgi.py

```


The first line spawns a WSGI daemon process called sampleapp that will be responsible for serving our Django application. The daemon name can be basically anything, but is good practice to *use descriptive names such as application names here.


If we were using global Python installation and global Django instance, the python-path directive would not be necessary. However, using virtual environment makes it obligatory to specify the alternate Python path so that mod_wsgi will know where to look for Python packages.


The path must contain two directories: the directory of Django project itself - /var/www/sampleapp - and directory of Python packages inside our virtual environment for that project - /var/www/sampleapp/env/lib/python2.7/site-packages. Directories in path definition are delimited using a colon sign.


The second line tells that particular virtual host to use the WSGI daemon created beforehand, and as such, the daemon name must match between those two. We used sampleapp in both lines.


The third line is the most important, as it tells Apache and mod_wsgi where to find WSGI configuration. The wsgi.py supplied by Django contains the barebone default configuration for WSGI for serving Django application that works just fine and changing the configuration in this file is out of this article scope.


After these changes it is necessary to restart Apache


```
service apache2 restart

```


After that, upon opening the web browser on your server IP address, without any additional ports, you should see the same Django page as before instead of initial It works! page that we have seen earlier.


That makes our configuration complete.


Please note: using the default virtual host without additional care is not the recommended way of configuring a production server. It is used for demonstration purposes.


## Further reading


The topic of configuring mod_wsgi and Django itself is vast. Many configuration aspects are application specific and difficult to explain or demonstrate without working with a real world Django application. This guide is not a complete howto for deployment of Django applications using Apache, but a quickstart guide on how to begin.


One of the best resources around is the official Django documentation. There are also great articles on DigitalOcean that can be found by using Django as a search keyword.


<div class=“author”>Article Submitted by: <a href=“http://maticomp.net”>Mateusz Papiernik</a></div>


