# How To Deploy Flask Web Applications Using uWSGI Behind Nginx on CentOS 6 4

```Python``` ```Python Frameworks``` ```Nginx``` ```CentOS```


Status: Deprecated
This article covers a version of CentOS that is no longer supported. If you are currently operating a server running CentOS 6, we highly recommend upgrading or migrating to a supported version of CentOS.
Reason:
CentOS 6 reached end of life (EOL) on November 30th, 2020 and no longer receives security patches or updates. For this reason, this guide is no longer maintained.
See Instead:
This guide might still be useful as a reference, but may not work on other CentOS releases. If available, we strongly recommend using a guide written for the version of CentOS you are using.
The following DigitalOcean tutorial may be of immediate interest, as it outlines deploying Flask applications with uWSGI and Nginx on a CentOS 7 server:

How To Serve Flask Applications with uWSGI and Nginx on CentOS 7


## Introduction



Armin Ronacher’s Flask is one of the greatest things that has ever happened in the field of web application frameworks created for Python in the past couple of years.


Flask is a minimalist but extremely functional - and powerful - framework that is hugely popular and very much extensible with a great choice of third party libraries (e.g. Flask-WTF or Flask-SQLAlchemy). This developer friendly framework is a great way to start web development using Python, especially if you are trying to learn how technical challenges are solved as well, thanks to its clean and easy-to-read codebase – waiting for you to discover.


In this DigitalOcean article, we are going to try to show you how to deploy your application and get it up-and-running in a similar fashion. We will begin with preparing the deployment server running CentOS 6.4 for Python and see how to properly work with uWSGI application server set to operate behind Nginx reverse-proxy.


# Glossary



<h3>1. Flask In Brief</h3><hr>


1. Web Application Deployment
2. WSGI In Brief
3. Using Nginx As A Reverse-Proxy
4. Python WSGI Web Application Servers
5. uWSGI In Brief

<h3>2. Preparing The System For Deployment</h3><hr>


1. Updating The System
2. Setting Up Python, pip and virtualenv
3. Preparing The System For Development
4. Downloading, Compiling And Installing Python on CentOS
5. Installing pip on CentOS Using a New Python Installation
6. Installing virtualenv on CentOS Using New Python Installation

<h3>3. Getting Started With Application Deployment</h3><hr>


1. Creating The Application Directory For Deployment
2. Creating The Virtual Environment
3. Working With The Virtual Environment
4. Downloading And Installing uWSGI Inside the Virtual Environment
5. Downloading And Installing Flask Library
6. Creating A Sample Flask Application

<h3>4. Deployment Stage: Installing And Setting Up Nginx</h3><hr>


1. Installing Nginx
2. Configuring Nginx

<h3>5. Deployment Stage: Working With uWSGI</h3><hr>


1. Running The Server

<h3>6. Further Reading</h3><hr>


# Flask In Brief



Given Flask’s nature, there is not much else to say than what we have already mentioned in the introduction section. It is a beautifully programmed, minimalist web application development library that has only two direct dependencies: Jinja2 template engine and Werkzeug WSGI toolkit.


Using Flask, it is extremely easy to create web sites that can stretch anywhere from a single file to dozens of re-usable modules (i.e. components) structured using blueprints.


In our article, we will be using a very basic, sample Flask application – strictly created to demonstrate deployment. In order to learn about packaging your application and uploading it, check out our article How to package and distribute Python Applications. If you are interested in learning more about Flask and “getting big”, you might be interested in our How to structure large Flask applications.


## Web Application Deployment



In regards to all Python WSGI web applications, deployments consists of preparing a WSGI module that contains a reference to your application object which is then used as a point of entry by the web-server to pass the requests which are to be handled by the application controllers (or views).


Here, we will be using uWSGI acting as a WSGI application server that will contain the Flask application to serve it behind Nginx. Since Nginx has native support for uWSGI’s preferred and (acclaimed) faster wire-protocol, we will set it to work accordingly.


## WSGI In Brief



Very simply put, WSGI is an interface between a web server and the application itself. It exists to ensure a standardised way between various servers and applications (frameworks) to work with each other, allowing interchangeability when necessary (e.g. switching from development to production environment), which is a must-have need nowadays.


Note: If you are interested in learning more about WSGI and Python web servers, check out our article: A Comparison of Web Servers for Python Based Web Applications.


In Flask’s case, WSGI operations are handled by the underlying Werkzeug middle-ware library.


## Using Nginx As A Reverse-Proxy



Nginx is a very high performant web server / (reverse)-proxy. It has reached its popularity due to being light weight, relatively easy to work with and easy to extend (with add-ons / plug-ins). Thanks to its architecture, it is capable of handling a lot of requests (virtually unlimited), which - depending on your application or website load - could be really hard to tackle using some other, older alternatives.



Remember: “Handling” connections technically means not dropping them and being able to serve them with something. You still need your application and database functioning well in order to have Nginx serve clients responses that are not error messages.

Due to its popularity and success, we are going to deploy our Flask application running behind Nginx to benefit from its powerful features. Its native support for uWSGI application server also makes it a preferred way to go online.


## Python WSGI Web Application Servers



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


## uWSGI In Brief



Despite its very confusing naming conventions, uWSGI itself is a vast project with many components, aiming to provide a full software stack for building hosting services. One of these components, the uWSGI server, runs Python WSGI applications. It is capable of using various protocols, including its own uwsgi wire protocol, which is quasi-identical to SCGI. In order to fulfil the understandable demand to use stand-alone HTTP servers in front of application servers, NGINX and Cherokee web servers are modularised to support uWSGI’s [own] best performing uwsgi protocol to have direct control over its processes.


uWSGI Highlights


- 
uWSGI comes with a WSGI adapter and it fully supports Python applications running on WSGI.

- 
It links with libpython. It loads the application code on startup and acts like a Python interpreter. It parses the incoming requests and invokes the Python callable.

- 
It comes with direct support for popular NGINX web server (along with Cherokee and lighttpd).

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


# Preparing The System For Deployment



## Updating The System



In order to have a stable deployment server, it is crucial to keep things up-to-date and well maintained.


To ensure that we have the latest available versions of default applications, we need to update our system.


Run the following to update your CentOS system:


```
yum -y update

```


## Setting Up Python, pip and virtualenv



Note: This guide should be valid for CentOS version 6.5 as well as 5.8 and 6.4.


CentOS / RHEL, by default, comes as a very lean server. Its toolset, which is likely to be dated for your needs, is not there to run your applications but to power the server’s system tools (e.g. YUM).


In order to prepare a CentOS system, Python needs to be set up (i.e. compiled from the source) and pip / virtualenv need to be installed using that particular interpreter.


All in all, we will be working with the following Python packages:


- 
python-dev – development tools

- 
pip – to manage packages

- 
virtualenv – to create isolated, virtual environments


Note: The following is a summary (albeit being a thorough one) of our How to set up Python 2.7.6 and 3.3.3 on CentOs 6.4 article. If you would like to learn more, you are advised to check it out. In case you might want to learn more about pip and virtualenv, see our Common Python Tools: Using virtualenv And pip tutorial.


## Preparing The System For Development



CentOS distributions do not come with many of the popular applications and tools that you are likely to need - and this is an intentional design choice.


For our installations, we are going to need some libraries and tools (i.e. development [related] tools) not shipped by default.


Therefore, we need to get them downloaded and installed before we continue.


YUM Software Groups consist of bunch of commonly used tools (applications) bundled together, ready for download all at the same time via execution of a single command and stating a group name.


Note: Using YUM, you can even download multiple groups together.


In order to get necessary development tools, run the following:


```
yum groupinstall -y development

```


or;


```
yum groupinstall -y 'development tools'

```


Note: The former (shorter) version might not work on older distributions of CentOS.


To download some additional packages which are handy:


```
yum install -y zlib-devel openssl-devel sqlite-devel bzip2-devel

```


## Downloading, Compiling And Installing Python on CentOS



Note: Instructions given here can be used to download any version of Python. You will just need to replace the version stated (which is 2.7.6 in the example below) with the version you require (e.g. 3.3.3). You can install and use multiple versions at the same time. However, you will need to specify their version during the execution (i.e. instead of python, you will need to use python2.7 or python3.3)


Let’s begin with retrieving the (compressed) archive containing Python source code. We will target --version 2.7.6.


```
wget http://www.python.org/ftp/python/2.7.6/Python-2.7.6.tar.xz

```


This file is compressed using XZ library. Your system, depending on its version, might not have it. If that is the case, run the following to install XZ library:


```
 yum install xz-libs

```


Decode the XZ archive, and extracting the tar archive’s contents:


```
# Let's decode (-d) the XZ encoded tar archive:
xz -d Python-2.7.6.tar.xz


# Now we can perform the extraction:
tar -xvf Python-2.7.6.tar

```


Verify the codebase using ./configure:


```
# Enter the file directory:
cd Python-2.7.6

# Start the configuration (setting the installation directory)
# By default files are installed in /usr/local.
# You can modify the --prefix to modify it (e.g. for $HOME).
./configure --prefix=/usr/local    

```


Build and install Python 2.7.6:


```
# Let's build (compile) the source
# This procedure can take awhile (~a few minutes)
make && make altinstall

```


[Optional Step] Adding New Python Installation Location to PATH:



Note: If you have followed the instructions using the default settings, you should not have the need to go through this section. However, if you have chosen a different path than /usr/local to install Python, you will need to perform the following to be able to run it without explicitly stating its full [installation] path each time.

```
# Example: export PATH="[/path/to/installation]:$PATH"
export PATH="/usr/local/bin:$PATH"

```



To learn more about PATH, consider reading PATH definition at The Linux Information Project.

## Setting Up Common Python Tools pip and virtualenv



Having installed Python, we can now finalise completing the basics for application production and deployment. For this, we will set up two of the most commonly used tools: pip package manager and virtualenv environment manager.


If you are interested in learning more about these two tools or just quickly refreshing your knowledge, consider reading Common Python Tools: Using virtualenv, Installing with Pip, and Managing Packages.


## Installing pip on CentOS Using a New Python Installation



Before installing pip, we need to get its only external dependency - setuptools.


Execute the following commands to install setuptools:



This will install it for [Python] version 2.7.6

```
# Let's download the installation file using wget:
wget --no-check-certificate https://pypi.python.org/packages/source/s/setuptools/setuptools-1.4.2.tar.gz

# Extract the files from the archive:
tar -xvf setuptools-1.4.2.tar.gz

# Enter the extracted directory:
cd setuptools-1.4.2

# Install setuptools using the Python we've installed (2.7.6)
python2.7 setup.py install

```


Installing pip itself is a very simple process afterwards. We will make use of the instructions from the article mentioned above to have it downloaded and installed automatically and securely using cURL library.


Let’s download the setup files for pip and have Python (2.7) install it:



This will install it for [Python] version 2.7.6

```
curl https://raw.github.com/pypa/pip/master/contrib/get-pip.py | python2.7 -

```


## Installing virtualenv on CentOS Using New Python Installation



Run the following command to download and install virtualenv using pip:


```
pip install virtualenv

```


# Getting Started With Application Deployment



Before we begin with creating a sample application and actually downloading (and installing) our servers, we need to (not have to, but need to) create a virtual environment to contain application related libraries and data.


## Creating The Application Directory For Deployment



Let’s begin with building an application directory to contain:


- 
Our application module

- 
Virtual environment directory

- 
WSGI file that servers need


```
# Folders / Directories
mkdir ~/MyApplication     # Replace the name to suit your needs
mkdir ~/MyApplication/app # Application (module) directory

# Application Files 
touch ~/MyApplication/WSGI.py          # Server entry-point
touch ~/MyApplication/app/__init__.py  # Application file

```


## Creating The Virtual Environment



Getting started using virtual environment is very easy.


Run the following to initiate a new environment inside MyApplication directory:


```
cd ~/MyApplication
virtualenv env

```


This command will create a new directory - called env - next to the application module app.


## Working With The Virtual Environment



There are a couple of ways to work with a virtual environment:


- 
Activating the environment

- 
Explicitly stating the location of Python interpreter inside the environment.


To keep things simple, we will be following the second option and explicitly state the location of Python interpreter and pip.


## Downloading And Installing uWSGI Inside the Virtual Environment



To install uWSGI using pip, run the following:


```
~/MyApplication/env/bin/pip install uwsgi

```


This command will have uWSGI installed inside our virtual environment.


## Downloading And Installing Flask Library



To install Flask using pip, run the following:


```
~/MyApplication/env/bin/pip install flask

```


This command will have Flask installed inside our virtual environment


## Creating A Sample Flask Application



To continue with our deployment example, we need to have a sample application set up to run.


Let’s create (edit) a single page Flask application:


```
nano ~/MyApplication/app/__init__.py

```


Place the below contents to define a simple Flask app:


```
from flask import Flask
app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello!"

if __name__ == "__main__":
    app.run()

```


Save and exit by pressing CTRL+X and confirming with Y.


## Creating A Sample WSGI File That Imports The Application



In a normal scenario, the app folder we have created would contain the main application module - which we summarised in a single file. This application module, alongside the app object, would be imported by a WSGI file to be served. In this step, we are going to create that WSGI file, which will import the application, and provide it to the uWSGI application server in the next step.


Let’s create (edit) the WSGI.py file:


```
nano ~/MyApplication/WSGI.py

```


Place the below contents:


```
from app import app

if __name__ == "__main__":
    app.run()

```


# Deployment Stage: Installing And Setting Up Nginx



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
# Enable EPEL Repository
sudo su -c 'rpm -Uvh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm'

# Download and install Nginx
sudo yum install -y nginx

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


## Configuring Nginx



After choosing and setting up a web server to run our application, we can continue with doing the same with Nginx and prepare it to talk with the back-end server(s) [running the WSGI app].


To achieve this, we need to modify Nginx’s configuration file: “nginx.conf”


Run the following command to open up nginx.conf and edit it using nano text editor:


```
sudo nano /etc/nginx/nginx.conf

```


You can replace the file with the following example configuration to get Nginx work as a reverse-proxy, talking to your application.


Copy and paste the below example configuration:



Note: To learn about incorporating SSL support, please read this article first: Creating an SSL certificate on Nginx.

In our example below, we are taking advantage of Nginx’s native integration with the uWSGI application server through its uwsgi wire-protocol.


Example configuration for web applications:


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
    upstream uwsgicluster {
    
        server 127.0.0.1:8080;
        # server 127.0.0.1:8081;
        # ..
        # .
    
    }

    # Configuration for Nginx
    server {
    
        # Running port
        listen 80;

        # Settings to by-pass for static files 
        location ^~ /static/  {
        
            # Example:
            # root /full/path/to/application/static/file/dir;
            root /app/static/;
        
        }
        
        # Serve a static file (ex. favico) outside static dir.
        location = /favico.ico  {
        
            root /app/favico.ico;
        
        }

        # Proxying connections to application servers
        location / {
        
            include            uwsgi_params;
            uwsgi_pass         uwsgicluster;

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


Note: To learn more about Nginx, please refer to our article: How to Configure Nginx Web Server on a VPS.


# Setting Up Python WSGI Web Application Servers



Note: To learn more about deploying Python web applications using uWSGI, check out our article: How to deploy using uWSGI web server.


## Serving Applications Using uWSGI



In this section, we will see how a Python WSGI application works with uWSGI web server. What uWSGI needs, just like other servers, is for your application to provide it with an entry point (i.e. an app object). During launch, this callable, alongside configuration variables, are passed to uWSGI and it starts to do its job. When a request arrives, it processes it and passes it to your application’s controller to handle.


## Running The Server



uWSGI has a lot of options and configurations with many possible ways of using them thanks to its flexibility. Without complicating things from the start, we will begin with working with it as simply as possible and continuing thereon with more advanced methods.


Note: Make sure to be in the “my_app” folder before executing the below commands as otherwise uwsgi will not be able to find wsgi.py nor import the application object app.


Simple usage example:


```
# Enter the application directory
cd ~/MyApplication

# Run uWSGI Installed inside the virtual environment
env/bin/uwsgi [option] [option 2] .. -w [wsgi file with app. callable]

```


To run uWSGI to start serving the application from wsgi.py, run the following:


```
cd ~/MyApplication
env/bin/uwsgi --socket 127.0.0.1:8080 -w WSGI:app

# To get uWSGI communicate with Nginx using HTTP:
# env/bin/uwsgi --socket 127.0.0.1:8080 --protocol=http -w WSGI:app

```



This will run the server on the foreground. If you would like to stop it, press CTRL+C.

To run the server in the background, run the following:


```
env/bin/uwsgi --socket 127.0.0.1:8080 -w WSGI:app &

```



When you run an application in the background, you will need to use a process manager (e.g. htop) to kill (or stop) it. See the section below for more details.

And that’s it! After connecting your application server with Nginx, you can now visit it by going to your droplet’s IP address using your favourite browser.


```
http://[your droplet's IP adde.]/

# Hello, world!

```


# Further Reading



If you would like to learn more about Python web-application deployments, you are recommended to check out our following articles on the subject for a better general understanding:


- How to Deploy Python WSGI Applications Using uWSGI Web Server with Nginx
- A Comparison of Web Servers for Python Based Web Applications

<div class=“author”>Submitted by: <a
href=“https://twitter.com/ostezer”>O.S. Tezer</a></div>


