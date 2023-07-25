# NGINX as Reverse Proxy for Node or Angular application

```Nginx``` ```Ubuntu``` ```UNIX/Linux```

A reverse proxy is a server that retrieves resources for clients from one or more upstream servers. It typically places itself behind a firewall in a private network and forwards clients request to these upstream servers. A reverse proxy greatly improves security, performance, and reliability of any web application. Many modern web applications written in NodeJS or Angular can run with their own standalone server but they lack a number of advanced features like load balancing, security, and acceleration that most of these applications demands. NGINX with its advanced features can act as a reverse proxy while serving the request for a NodeJS or an Angular application.


# NGINX Reverse Proxy Server


In this tutorial, we will explore how NGINX can be used as a reverse proxy server for a Node or an Angular application. Below diagram gives you an overview of how reverse proxy server works and process client requests and send the response.


Nginx Reverse Proxy
# Prerequisite


- You have already installed NGINX by following our tutorial from here.

# Assumption


- The NGINX server can be accessed from public domain.
- The Node or Angular application will be running in a separate system (upstream server) in a private network and can be reached from NGINX server. Although it is very much possible to do the setups in a single system.
- The tutorial makes use of variables like SUBDOMAIN.DOMAIN.TLD and PRIVATE_IP. Replace them with your own values at appropriate places.

# NodeJS application


Assuming you have already installed NGINX in your environment, Let us create an example NodeJS application that will be accessed through NGINX reverse proxy. To start with, set up a node environment in a system residing in your private network.


## Install Node


Before proceeding with installing NodeJS and latest version of npm(node package manager), check if it is already installed or not:


```
# node --version 
# npm --version

```


If the above commands return the version of NodeJS and NPM then skip the following installation step and proceed with creating the example NodeJS application. To install NodeJS and NPM, use the following commands:


```
# apt-get install nodejs npm

```


Once installed, check the version of NodeJS and NPM again.


```
# node --version
# npm --version

```


## Create example Node application


Once NodeJS environment is ready, create an example application using ExpressJS. Therefore, create a folder for node application and install ExpressJS.


```
# mkdir node_app  
# cd node_app
# npm install express

```


Now using your favorite text editor, create app.js and add the following content into it.


```
# vi app.js
const express = require('express')
const app = express()
app.get('/', (req, res) => res.send('Hello World !'))
app.listen(3000, () => console.log('Node.js app listening on port 3000.'))

```


Run the node application using following command:


```
# node app.js

```


Make a curl query to the port number 3000 to confirm that the application is running on localhost.


```
# curl localhost:3000
Hello World !

```


At this point, NodeJS application will be running in the upstream server. In the last step, we will configure NGINX to act as a reverse proxy for the above node application. For the time being, let us proceed with creating an angular application, the steps for which are given below:


# Angular application


Angular is another JavaScript framework for developing web applications using typescript. In general, an angular application is accessed through the standalone server that is shipped along with it. But due to a few disadvantages of using this standalone server in a production environment, a reverse proxy is placed in front of an angular application to serve it better.


## Setup angular environment


Since Angular is a JavaScript framework, it requires to have Nodejs with version > 8.9 installed in the system. Therefore before proceeding with installing angular CLI, quickly setup node environment by issuing following command in the terminal.


```
# curl -sL https://deb.nodesource.com/setup_10.x | sudo bash -
# apt-get install nodejs npm

```


Now proceed with installing Angular CLI that helps us to create projects, generate application and library code for any angular application.


```
# npm install -g @angular/cli

```


The setup needed for Angular environment is now complete. In the next step, we will create an angular application.


## Create angular application


Create an Angular application using following angular CLI command:


```
# ng new angular-app

```


Change to the newly created angular directory and run the web application by specifying the host name and port number:


```
# cd angular-app
# ng serve --host PRIVATE_IP --port 3000

```


Make a curl query to the port number 3000 to confirm that the angular application is running on localhost.


```
# curl PRIVATE_IP:3000

```


At this point, the angular application will be running in your upstream server. In the next step, we will configure NGINX to act as a reverse proxy for the above angular application.


# Configure NGINX as Reverse Proxy


Navigate to the NGINX virtual host configuration directory and create a server block that will act as a reverse proxy. Remember the system where you have installed NGINX earlier can be reached via the Internet i.e. a public IP is attached to the system.


```
# cd /etc/nginx/sites-available
# vi node_or_angular_app.conf

server {  
              listen 80;
              server_name SUBDOMAIN.DOMAIN.TLD;
              location / {  
                           proxy_pass https://PRIVATE_IP:3000;  
                           proxy_http_version 1.1;  
                           proxy_set_header Upgrade $http_upgrade;  
                           proxy_set_header Connection 'upgrade';  
                           proxy_set_header Host $host;  
                           proxy_cache_bypass $http_upgrade;  
               }  
}

```


The proxy_pass directive in the above configuration makes the server block a reverse proxy. All traffic destined to the domain SUBDOMAIN.DOMAIN.TLD and those matches with root location block (/) will be forwarded to https://PRIVATE_IP:3000 where the node or angular application is running.


# NGINX Reverse Proxy for Both NodeJS and Angular App?


The above server block will act as a reverse proxy for either node or angular application. To serve both node and angular application at the same time using NGINX reverse proxy, just run them in two different port number if you intended to use the same system for both of them. It is also very much possible to use two separate upstream servers for running node and angular application. Further, you also need to create another NGINX server block with a matching values for server_name and proxy_pass directive. Recommended Read: Understanding NGINX Configuration File. Check for any syntactical error in the above server block and enable the same. Finally, reload NGINX to apply new settings.


```
# nginx -t
# cd /etc/nginx/sites-enabled
# ln -s ../sites-available/node_or_angular_app.conf .
# systemctl reload nginx

```


Now point your favorite web browser to https://SUBDOMAIN.DOMAIN.TLD, you will be greeted with a welcome message from the Node or Angular application.


Angular Welcome Page
# Summary


That’s all for configuring an NGINX reverse proxy for NodeJS or Angular application. You can now proceed with adding a free SSL certificate like Let’s Encrypt to secure your application!


