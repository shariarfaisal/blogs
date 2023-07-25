# How To Use Apache as a Reverse-Proxy with mod_proxy on Ubuntu 20 04

```Apache``` ```Ubuntu``` ```Load Balancing``` ```Ubuntu 20.04```

## Introduction


A reverse proxy is a type of proxy server that takes HTTP(S) requests and transparently distributes them to one or more backend servers. Reverse proxies are useful because many modern web applications process incoming HTTP requests using backend application servers. These servers aren’t meant to be accessed by users directly, and often only support basic HTTP features.


You can use a reverse proxy to prevent these underlying application servers from being directly accessed. They can also be used to distribute the load from incoming requests to several different application servers, increasing performance at scale and providing fail-safeness. They can fill in the gaps with features the application servers don’t offer, such as caching, compression, or SSL encryption.


In this tutorial, you’ll set up Apache as a basic reverse proxy using the mod_proxy extension to redirect incoming connections to one or several backend servers running on the same network. This tutorial uses a backend written with the Flask web framework, but you can use any backend server you prefer.


# Prerequisites


To follow this tutorial, you will need:


- One Ubuntu 20.04 server set up with this initial server setup tutorial, including a sudo non-root user and a firewall.
- Apache 2 installed on your server by following Steps 1 and 2 of How To Install the Apache Web Server on Ubuntu 20.04.

# Step 1 — Enabling Necessary Apache Modules


Apache has many modules bundled with it that are available but not enabled in a fresh installation. First, you’ll need to enable the ones you’ll use in this tutorial.


The modules you need are mod_proxy itself and several of its add-on modules, which extend its functionality to support different network protocols. Specifically, you will use the following:


- mod_proxy: the main proxy module for redirecting connections. It allows Apache to act as a gateway to the underlying application servers.
- mod_proxy_http: adds support for proxying HTTP connections.
- mod_proxy_balancer and mod_lbmethod_byrequests: these add load balancing features for multiple backend servers.

To enable these four modules, execute the following command:


```
sudo a2enmod proxy proxy_http proxy_balancer lbmethod_byrequests


```


To put these changes into effect, restart Apache:


```
sudo systemctl restart apache2


```


Apache is now ready to act as a reverse proxy for HTTP requests. In the next optional step, you will create two basic backend servers. These will help verify if the configuration works properly, but if you already have your own backend application, you can skip to Step 3.


# Step 2 — Creating Backend Test Servers (Recommended)


Running some backend servers can help test if your Apache configuration is working properly. Here, you’ll make two test servers that respond to HTTP requests by printing a line of text. One server will say Hello world! and the other will say Howdy world! This will let you test load balancing between multiple services.



Note: In real-world setups, backend servers usually all return the same kind of content. However, for the purposes of this test, having the two servers return different messages allows you to check that the load balancing mechanism uses both.

Flask is a Python microframework for building web applications. This step outlines how to use Flask to create the test servers because a minimal application requires just a few lines of code. You don’t need to know Python to set these up, but if you’d like to learn, you can check out our series on How To Code in Python.


First update the package index list using apt:


```
sudo apt update


```


Then install pip, the recommended Python package manager:


```
sudo apt install python3-pip


```


Next, use pip to install Flask:


```
sudo pip3 install Flask


```


Now that all the required components are installed, create a new file that will contain the code for the first backend server in the home directory of the current user. You can do this with your preferred text editor, here we’ll use nano:


```
nano ~/backend1.py


```


Insert the following code snippet into the file:


~/backend1.py
```
from flask import Flask
app = Flask(__name__)

@app.route('/')
def home():
    return 'Hello world!'

```


The first two lines of code initialize the Flask framework. There is one function, home(), which returns a line of text (Hello world!). The @app.route('/') line above the home() function definition tells Flask to use home()'s return value as a response to HTTP requests directed at the / root URL of the application.


Once you’re done, save and exit the file. If you’re using nano, you can do this by pressing CTRL + X, then Y and ENTER.


The second backend server is exactly the same as the first, aside from returning a different line of text. Therefore, run cp to copy the content from the first file, backend1.py into the backend2.py file:


```
cp ~/backend1.py ~/backend2.py


```


Now open the newly copied file using your preferred text editor:


```
nano ~/backend2.py


```


Update the message that returns the Hello world! message to read Howdy world! instead:


~/backend2.py
```
from flask import Flask
app = Flask(__name__)

@app.route('/')
def home():
    return 'Howdy world!'

```


After you’re done making the update, save and close the file.


Next, use the following command to start the first background server on port 8080. This also redirects Flask’s output to /dev/null because it would cloud the console output further on:


```
FLASK_APP=~/backend1.py flask run --port=8080 >/dev/null 2>&1 &


```


Here, you are preceding the flask command by setting the FLASK_APP environment variable in the same line. Environment variables are a convenient way to pass information into processes that are spawned from the shell. You can learn more about environment variables in our guide on How To Read and Set Environmental and Shell Variables on a Linux VPS.


In this case, using an environment variable ensures the setting applies only to the command being run and will not stay available afterward. This is necessary to avoid confusion because you’ll be passing another filename the same way to tell the flask command to start the second server.


Similarly, you want to run the following command to start the second server on port 8081. Note the different value for the FLASK_APP environment variable:


```
FLASK_APP=~/backend2.py flask run --port=8081 >/dev/null 2>&1 &


```


Now you can test if the two servers are running by using the curl command. Start by testing the first server. This command uses curl to connect to 127.0.0.1, a special IP address that represents localhost. This means that the following command tells your server to connect to itself and print its own response:


```
curl http://127.0.0.1:8080/


```


This will print the following response from the server:


```
OutputHello world!

```


Next, test the second server :


```
curl http://127.0.0.1:8081/


```


As before, this will print the expected response from the server:


```
OutputHowdy world!

```


In the next step, you’ll modify Apache’s configuration file to enable its use as a reverse proxy.


# Step 3 — Modifying the Default Configuration to Enable Reverse Proxy


In this section, you will set up the default Apache virtual host to serve as a reverse proxy for a single backend server or an array of load balanced backend servers.



Note: In this tutorial, you’re applying the configuration at the virtual host level. On a default installation of Apache, there is only a single, default virtual host enabled. However, you can use all those configuration fragments in other virtual hosts as well. To learn more about virtual hosts in Apache, you can read our How To Set Up Apache Virtual Hosts on Ubuntu 20.04 tutorial.
If your Apache server acts as both an HTTP and HTTPS server, your reverse proxy configuration must be placed in both the HTTP and HTTPS virtual hosts. To learn more about SSL with Apache, you can read our tutorial on How To Create a Self-Signed SSL Certificate for Apache in Ubuntu 20.04 tutorial.

Open the default Apache configuration file using your preferred text editor:


```
sudo nano /etc/apache2/sites-available/000-default.conf


```


Inside that file, you will find the <VirtualHost *:80> block starting on the first line. The following first example explains how to configure this block to reverse proxy for a single backend server, and the second example sets up a load balanced reverse proxy for multiple backend servers.


## Example 1 — Reverse Proxying a Single Backend Server


First, replace all the contents within the VirtualHost block so that your configuration file reads like the following. If you’ve followed along with the example servers in Step 2, use 127.0.0.1:8080. If you have your own application servers, use their addresses instead:


/etc/apache2/sites-available/000-default.conf
```
<VirtualHost *:80>
    ProxyPreserveHost On

    ProxyPass / http://127.0.0.1:8080/
    ProxyPassReverse / http://127.0.0.1:8080/
</VirtualHost>

```


There are three directives represented, here’s a brief overview of what they’re doing:


- ProxyPreserveHost: makes Apache pass the original Host header to the backend server. This is useful, as it makes the backend server aware of the address used to access the application.
- ProxyPass: the main proxy configuration directive. In this case, it specifies that everything under the root URL (/) should be mapped to the backend server at the given address. For example, if Apache gets a request for /example, it will connect to http://your_backend_server/example  and return the response to the original client.
- ProxyPassReverse: should have the same configuration as ProxyPass. It tells Apache to modify the response headers from the backend server. This makes sure that if the backend server returns a location redirect header, the client’s browser will be redirected to the proxy address and not the backend server address, which would not work as intended.

Once you’re done adding this content, save and exit the file.


To put these changes into effect, restart Apache:


```
sudo systemctl restart apache2


```


Now, if you access http://your_server_ip in a web browser, you will see your backend server response instead of the standard Apache welcome page. If you followed Step 2, this means you’ll see Hello world! in the browser.


## Example 2 — Load Balancing Across Multiple Backend Servers


If you have multiple backend servers, a good way to distribute the traffic across them when proxying is to use the load balancing features of mod_proxy.


First open the default Apache configuration file using your preferred text editor:


```
sudo nano /etc/apache2/sites-available/000-default.conf


```


Now replace all the contents within the VirtualHost so that your configuration file reads like the following. If you followed along with the example servers in Step 2, use 127.0.0.1:8080  and 127.0.0.1:8081 for the BalancerMember directives. If you have your own application servers, use their addresses instead:


/etc/apache2/sites-available/000-default.conf
```
<VirtualHost *:80>
<Proxy balancer://mycluster>
    BalancerMember http://127.0.0.1:8080
    BalancerMember http://127.0.0.1:8081
</Proxy>

    ProxyPreserveHost On

    ProxyPass / balancer://mycluster/
    ProxyPassReverse / balancer://mycluster/
</VirtualHost>

```


The configuration is similar to the previous one, but instead of specifying a single backend server directly, these directives are doing the following:


- Proxy: this additional Proxy block is used to define multiple servers. The block is named balancer://mycluster (the name can be freely altered) and consists of one or more BalancerMembers, which specify the underlying backend server addresses.
- ProxyPass and ProxyPassReverse: these directives use the load balancer pool named mycluster` instead of a specific server.

If you followed along with the example servers in Step 2, use 127.0.0.1:8080  and 127.0.0.1:8081 for the BalancerMember directives, as written in the block above. If you have your own application servers, use their addresses instead.


Once you’re done adding this content, save and exit the file.


To put these changes into effect, restart Apache:


```
sudo systemctl restart apache2


```


If you access http://your_server_ip in a web browser, you will see your backend servers’ responses instead of the standard Apache page. If you followed Step 2,  then refreshing the page multiple times should display Hello world! and Howdy world!, meaning the reverse proxy worked and is load balancing between both servers.



Note: To close both test servers after you no longer need them, like when you finish this tutorial, you can execute the killall flask command.

# Conclusion


Now you know how to set up Apache as a reverse proxy to one or many underlying application servers. mod_proxy can be used effectively to configure a reverse proxy to application servers written in a vast array of languages and technologies, such as Python and Django, or Ruby and Ruby on Rails. It can be also used to balance traffic between multiple backend servers for sites with lots of traffic, to provide high availability through multiple servers, or to provide secure SSL support to backend servers not supporting SSL natively.


While mod_proxy with mod_proxy_http is perhaps the most commonly used combination of modules, there are several others that support different network protocols. You didn’t use them in this tutorial, but some other popular modules include:


- mod_proxy_ftp for FTP (File Transfer Protocol).
- mod_proxy_connect for SSL tunneling.
- mod_proxy_ajp for AJP (Apache JServ Protocol), such as Tomcat-based backends.
- mod_proxy_wstunnel for web sockets.

To learn more about mod_proxy, you can read the official Apache mod_proxy documentation.


