# How to Deploy CherryPy Web Applications Behind Nginx Reverse-Proxy

```Ubuntu``` ```Python Frameworks``` ```Nginx``` ```CentOS```

## Introduction



CherryPy is an excellent framework to create web-applications and APIs of all sizes – From a “get-started-with-Python” Hello, world! to what can become one of world’s one of busiest web sites!


If you are coming from a different language, the process of getting your new application online might appear a little bit unfamiliar when you first start developing using CherryPy.


In this DigitalOcean article, we will go through two good ways of deploying an absolutely solid CherryPy based web application along with managing its dependencies using pip.


# Glossary



## 1. CherryPy and Web Application Deployment In Brief



1. Web Application Deployment
2. WSGI
3. Using Nginx As A Reverse-Proxy
4. Python (WSGI) Web Application Servers
5. CherryPy’s Application (HTTP) Server In Brief
6. uWSGI

## 2. Preparing A Simple CherryPy Application With The “app” Object Exposed



1. Creating The Application Structure
2. Edit “app/init.py” using nano
3. Edit “wsgi.py” using nano

## 3. Preparing The System For CherryPy Deployment



1. Updating The System
2. Setting up Python, pip and virtualenv
3. python-dev
4. pip
5. virtualenv
6. Downloading And Installing CherryPy
7. Downloading And Installing uWSGI

## 4. How To Handle Application Dependencies Using pip



1. Creating A List of Application Dependencies
2. Downloading From A List of Application Dependencies

## 5. Begin Deployment: Download, Install And Set Up Nginx



1. Installing Nginx
2. Configuring Nginx

## 6. Setting Up Python WSGI Web Application Servers



1. Serving Applications Using CherryPy’s Own Web-Server [*]
2. Running and Managing The CherryPy Application Server
3. Serving Applications Using uWSGI [*]
4. Running The Server

# CherryPy and Web Application Deployment In Brief



CherryPy as a whole is a minimalist Python web application development framework that is not shipped with too many components out-of-the-box, regardless of you wanting them or not. The framework handles all the core necessities you might need (e.g. sessions, caching, file uploads et al.) and leave the rest - and the choice - of what to use and how to use to be decided by you. It separates itself from other Python frameworks with its simplicity to get online using the shipped, ready to deploy HTTP/1.1-compliant, WSGI thread-pooled Web Server.


## Web Application Deployment



In regards to all Python WSGI web applications, deployments consists of preparing a WSGI module that contains a reference to your application object which is then used as a point of entrance by the web-server to pass the requests.


Note: However, in case of using CherryPy’s own server, the process becomes simpler and you don’t particularly need to worry about it.


In our article, we will see two different ways of application deployment:


- 
Using CherryPy’s default web-server, excellent for a majority of applications, and;

- 
Using another alternative application server (uWSGI) for applications requiring in-depth configuring abilities.


## WSGI



WSGI in a nutshell is an interface between a web server and the application itself. It exists to ensure a standardized way between various servers and applications (frameworks) to work with each other, allowing interchangeability when necessary (i.e. switching from development to production environment), which is a must-have need nowadays.


Note: If you are interested in learning more about WSGI and Python web servers, check out our article: A Comparison of Web Servers for Python Based Web Applications.


## Using Nginx As A Reverse-Proxy



Nginx is a very high performant web server / (reverse)-proxy. It has reached its popularity due to being light weight, relatively easy to work with and easy to extend (with add-ons / plug-ins). Thanks to its architecture, it is capable of handling a lot of requests (virtually unlimited), which - depending on your application or website load - could be really hard to tackle using some other, older alternatives.



Remember: “Handling” connections technically means not dropping them and being able to serve them with something. You still need your application and database functioning well in order to have Nginx serve clients responses that are not error messages.

## Python (WSGI) Web Application Servers



Python web application servers are [usually] either stand-alone C-based solutions or fully (or partially) Python based (i.e. pure-Python) ones.


They operate by accepting a Python module containing - as previously explained - an application callable to contain the web-application and serve it on a network.


Although some of them are highly capable servers that can be used directly, it is recommended to use Nginx in front for the reasons mentioned above (e.g. higher performance). Similarly, development servers that are usually shipped with web application frameworks are not recommended to be used in production due to their lack of functionality - with a few exceptions, of course!


Some Popular Python WSGI web servers are:


- 
CherryPy

- 
Gunicorn

- 
uWSGI

- 
waitress


## CherryPy’s Application (HTTP) Server In Brief



CherryPy’s pure Python web server is a compact solution which comes with the namesake framework. Defined by the [CherryPy] project as a “high-speed, production ready, thread pooled, generic HTTP server,” it is a modularized component which can be used to serve any Python WSGI web application.


CherryPy Web Server’s Highlights:


- 
A very compact and simple to use pure-Python solution

- 
Easy to configure, easy to use

- 
Thread-pooled and fast

- 
Allows scaling

- 
Supports SSL


## uWSGI




The following is an extract from the above mentioned DigitalOcean Python Server Comparison article.

Despite its very confusing naming conventions, uWSGI itself is a vast project with many components, aiming to provide a full software stack for building hosting services. One of these components, the uWSGI server, runs Python WSGI applications. It is capable of using various protocols, including its own uwsgi wire protocol, which is quasi-identical to SCGI. In order to fulfil the understandable demand to use stand-alone HTTP servers in front of application servers, NGINX and Cherokee web servers are modularized to support uWSGI’s [own] best performing uwsgi protocol to have direct control over its processes.


uWSGI Highlights


- 
uWSGI comes with a WSGI adapter and it fully supports Python applications running on WSGI.

- 
It links with libpython. It loads the application code on startup and acts like a Python interpreter. It parses the incoming requests and invokes the Python callable.

- 
It comes with direct support for popular NGINX web server (along with Cherokee* and lighttpd).

- 
It is written in C.

- 
Its various components can do much more than running an application, which might be handy for expansion.

- 
Currently (as of late 2013), it is actively developed and has fast release cycles.

- 
It has various engines for running applications (asynchronous and synchronous).

- 
It can mean lower memory footprint to run.


# Preparing A Simple CherryPy Application With The “app” Object Exposed



Let’s begin our deployment example with creating a new CherryPy application to use as a sample.


Note: The application example here uses virtual environment to manage the application files and its dependencies. To learn about pip and virtualenv, check out our tutorial Common Python Tools: Using virtualenv, Installing with Pip, and Managing Packages.


## Creating The Application Structure



We want to work with a simple example that should resemble a very minimalistic but actual application.


For this purpose, we can create something similar to this:


```
myy_app
  |-- wsgi.py
  |__ /app
        |-- __init__.py

```


First of all, let’s create an application folder and an application module:


```
mkdir ~/my_app
mkdir ~/my_app/app

```


Afterwards, let;s create the application file and the wsgi.py file needed for deployment:


```
touch ~/my_app/wsgi.py
touch ~/my_app/app/__init__.py

```


## Edit “app/init.py” using nano



We have created the app package to contain our exemplary application module. We can now edit the __init__.py to define it.


```
nano ~/my_app/app/__init__.py

```


Copy and paste the below script:


```
import cherrypy

class Root(object):
    @cherrypy.expose
    def index(self):
        return "Hello, world!"

```


Press CTRL+X and confirm with Y to save and exit.


## Edit “wsgi.py” using nano



The wsgi.py file will be used to expose an entry point to your application. In our example, inside this file we will import the application module (app) and run it directly using CherryPy’s WSGI web server, or, pass it on to uWSGI which will contain it and run.


```
nano ~/my_app/wsgi.py

```


Copy and paste the below contents:


```
import cherrypy
from app import Root

app = cherrypy.tree.mount(Root(), '/')

if __name__=='__main__':

    cherrypy.config.update({
        'server.socket_host': '127.0.0.1',
        'server.socket_port': 8080,
    })

    # Run the application using CherryPy's HTTP Web Server
    cherrypy.quickstart(Root())

```


Press CTRL+X and confirm with Y to save and exit.


# Preparing The System For CherryPy Deployment



## Updating The System



In order to have a stable deployment server, it is crucial to keep things up-to-date and well maintained.


To ensure that we have the latest available versions of default applications, we need to update our system.


For Debian Based Systems (i.e. Ubuntu, Debian), run the following:


```
aptitude    update
aptitude -y upgrade

```


For RHEL Based Systems (i.e. CentOS), run the following:


```
yum -y update

```


## Setting up Python, pip and virtualenv




Note for CentOS / RHEL Users:
CentOS / RHEL, by default, comes as a very lean server. Its toolset, which is likely to be dated for your needs, is not there to run your applications but to power the server’s system tools (e.g. YUM).
In order to prepare your CentOS system, Python needs to be set up (i.e. compiled from the source) and pip* / virtualenv need installing using that interpreter.
To learn about How to Set Up Python 2.7.6 and 3.3.3 on CentOS 6.4 and 5.8, with pip and virtualenv, please refer to: How to Set Up Python 2.7.6 and 3.3.3 on CentOS.

On Ubuntu and Debian, a recent version of Python interpreter which you can use comes by default. It leaves us with only a limited number of additional packages to install:


- 
python-dev (development tools),

- 
pip (to manage packages),

- 
virtualenv (to create isolated, virtual


Note: Before continuing with installation of certain applications, especially if building from source, you might need to install essential development tools build-essential using the following command:


```
aptitude install -y build-essential

```


## python-dev



python-dev is an operating-system level package which contains extended development tools for building Python modules.


Run the following command to install python-dev using aptitude:


```
aptitude install -y python-dev

# You might need python2.7-dev
# aptitude install -y python2.7-dev

```


## pip



pip is a package manager which will help us to install the application packages that we need.


Run the following commands to install pip:


```
curl https://bitbucket.org/pypa/setuptools/raw/bootstrap/ez_setup.py | python -
curl https://raw.github.com/pypa/pip/master/contrib/get-pip.py | python -
export PATH="/usr/local/bin:$PATH"

```



You might need sudo privileges.

## virtualenv



It is best to contain a Python application within its own environment together with all of its dependencies. An environment can be best described (in simple terms) as an isolated location (a directory) where everything resides. For this purpose, a tool called virtualenv is used.


Run the following to install virtualenv using pip:


```
sudo pip install virtualenv

```


## Downloading And Installing CherryPy



CherryPy framework can be installed using the pip package manager.


Run the following to install cherrypy using pip:


```
# Install CherryPy Framework and HTTP Web-Server
pip install cherrypy

```


## Downloading And Installing uWSGI



It is always the recommended way to contain all application related elements, as much as possible, together inside the virtual environment. So we will download and install uWSGI as such.


If you are not working inside an environment, uWSGI will be installed globally (i.e. available systemwide). This is not recommended – always opt for using virtualenv.


To install uWSGI using pip, run the following:


```
pip install uwsgi

```


# How To Handle Application Dependencies Using pip



Since it is highly likely that you have started the development process on a local machine, when deploying your application, you will need to make sure that all of its dependencies are installed (inside your *virtual environment).


## Creating A List of Application Dependencies



The simplest way to get the dependencies on the production environment  is by using pip. With a single command, it is capable of generating all the packages (or dependencies) you have installed (within your activated environment, if not, globally on your system) and again with a single command, it allows you to have them all downloaded and installed.


Note: This section contains information which is to be executed on your local development machine or from wherever you want to generate the list of application dependencies. This file should be placed inside your application directory and uploaded to your server.


Using “pip” to create a list of installed packages:


```
pip freeze > requirements.txt

```


This command will create a file called requirements.txt which contains the list of all installed packages. If you run it within a virtualenv, the list will consist of packages installed inside the environment only. Otherwise, all packages, installed globally will be listed.


## Downloading From A List of Application Dependencies



Using pip to install packages from a list:


Note: This section contains information which is to be executed on your production (i.e. deployment) machine / environment.


```
pip install -r requirements.txt

```


This command will download and install all the listed packages. If you are working within an activated environment, the files will be downloaded there. Otherwise, they will be installed globally - which is not the recommended way for the reasons explained in the previous sections.


# Begin Deployment: Download, Install And Set Up Nginx



Regardless of the choice of server, our CherryPy application will go online behind Nginx for the reasons we have mentioned in the previous sections. So, let us download and configure Nginx first and continue with working application servers.


Example of a Basic Server Architecture:


```
Client Request ----> Nginx (Reverse-Proxy)
                        |
                       /|\                           
                      | | `-> App. Server I.   127.0.0.1:8080 # Our example
                      |  `--> App. Server II.  127.0.0.1:8082
                       `----> App. Server III. 127.0.0.1:8083

```


## Installing Nginx




Note for CentOS / RHEL Users:
The below instructions will not work on CentOS systems. Please see the instructions here for CentOS.

Run the following command to install Nginx using aptitude:


```
sudo aptitude install nginx

```


To run Nginx, use the following:


```
sudo service nginx start

```


To stop Nginx, use the following:


```
sudo service nginx stop

```


To restart Nginx, use the following:


```
# After each time you reconfigure Nginx, a restart
# or reload is needed for the new settings to come
# into effect.  
sudo service nginx restart

```


Note: To learn more about Nginx on Ubuntu, please refer to our article: How to Install Nginx on Ubuntu 12.04.


## Configuring Nginx



Note: Below is a shorter tutorial on using Nginx as a reverse proxy. To learn more about Nginx, check out How to Configure Nginx Web Server on a VPS.


After choosing and setting up a web server to run our application, we can continue with doing the same with Nginx and prepare it to talk with the back-end server(s) [running the WSGI app].


To achieve this, we need to modify Nginx’s configuration file: nginx.conf


Run the following command to open up nginx.conf and edit it using nano text editor:


```
sudo nano /etc/nginx/nginx.conf

```


You can replace the file with the following example configuration to get Nginx work as a reverse-proxy, talking to your application.


Copy and paste the below example configuration:


```
worker_processes 1;

events {

    worker_connections 1024;

}

http {

    sendfile on;
    
    gzip              on;
    gzip_http_version 1.0;
    gzip_proxied      any;
    gzip_min_length   500;
    gzip_disable      "MSIE [1-6]\.";
    gzip_types        text/plain text/xml text/css
                      text/comma-separated-values
                      text/javascript
                      application/x-javascript
                      application/atom+xml;

    # Configuration containing list of application servers
    upstream app_servers {
    
        server 127.0.0.1:8080;
        # server 127.0.0.1:8081;
        # ..
        # .
    
    }

    # Configuration for Nginx
    server {
    
        # Running port
        listen 80;

        # Settings to serve static files 
        location ^~ /static/  {
        
            # Example:
            # root /full/path/to/application/static/file/dir;
            root /app/static/;
        
        }
        
        # Serve a static file (ex. favico)
        # outside /static directory
        location = /favico.ico  {
        
            root /app/favico.ico;
        
        }

        # Proxy connections to the application servers
        # app_servers
        location / {
        
            proxy_pass         http://app_servers;
            proxy_redirect     off;
            proxy_set_header   Host $host;
            proxy_set_header   X-Real-IP $remote_addr;
            proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header   X-Forwarded-Host $server_name;
        
        }
    }
}

```


When you are done modifying the configuration, press CTRL+X and confirm with Y to save and exit. You will need to restart Nginx for changes to come into effect.


Run the following to restart Nginx:


```
sudo service nginx stop
sudo service nginx start

```


# Setting Up Python WSGI Web Application Servers



Having created a sample application and made our way through managing dependencies, we are ready to begin with the last stage of deployment: setting up servers.


As mentioned above, in this article we will focus on using CherryPy and uWSGI web application servers behind Nginx.


## Serving Applications Using CherryPy’s Own Web-Server [*]



CherryPy’s pure Python web server is a compact solution which comes with the framework. It is defined by the project as a “high-speed, production ready, thread pooled, generic HTTP server.”


Since we have developed using the framework, our program inside the wsgi.py is already prepared to start serving upon running it.


Our settings for CherryPy matching our configuration of Nginx:


```
# ..

cherrypy.config.update({
    'server.socket_host': '127.0.0.1',
    'server.socket_port': 8080,
})

# ..

```


## Running and Managing The CherryPy Application Server



To start serving your application, you just need to execute server.py using your Python installation.


Run the following to start the server as configured:


```
python ~/my_app/wsgi.py

```


This will run the server on the foreground. If you would like to stop it, press CTRL+C.


To run the server in the background, use the following:


```
python ~/my_app/wsgi.py &

```


When you run an application in the background, you will need to use a process manager (e.g. htop) to kill (or stop) it.


## Serving Applications Using uWSGI [*]



Albeit being extremely capable and powerful, CherryPy’s own HTTP server is not for all set ups or deployments. If you require the ability to tune a lot of options to match your desired configuration settings, uWSGI might be the solution for you.


## Running The Server



uWSGI has a lot of options and configurations with many possible ways of using them thanks to its flexibility. Without complicating things from the start, we will begin with working with it as simply as possible and continuing thereon with more advanced methods.


Note: Make sure to be in the my_app folder before executing the below commands as otherwise uwsgi will not be able to find wsgi.py nor import the application object app.


Simple usage example:


```
uwsgi [option] [option 2] .. -w [wsgi file with app. callable]

```


To run uWSGI to start serving the application from wsgi.py, run the following:


```
uwsgi --socket 127.0.0.1:8080 --protocol=http -w wsgi:app

```



This will run the server on the foreground. If you would like to stop it, press CTRL+C.

To run the server in the background, run the following:


```
uwsgi --socket 127.0.0.1:8080 --protocol=http -w wsgi:app &

```



When you run an application in the background, you will need to use a process manager (e.g. htop) to kill (or stop) it. See the section below for more details.

And that’s it! After connecting your application server with Nginx, you can now visit it by going to your droplet’s IP address using your favourite browser.


```
http://[your droplet's IP adde.]/

# Hello, world!

```


# Further Reading



If you would like to learn more about Python web-application deployments, you are recommended to check out our following articles on the subject for a better general understanding:


- 
How to Deploy Python WSGI Applications Using uWSGI Web Server with Nginx

- 
A Comparison of Web Servers for Python Based Web Applications


<div class=“author”>Submitted by: <a
href=“https://twitter.com/ostezer”>O.S. Tezer</a></div>


