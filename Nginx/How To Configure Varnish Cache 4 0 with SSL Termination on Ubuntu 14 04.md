# How To Configure Varnish Cache 4 0 with SSL Termination on Ubuntu 14 04

```Ubuntu``` ```Caching``` ```Nginx```

## Introduction


In this tutorial, we will cover how to use Varnish Cache 4.0 to improve the performance of your existing web server. We will also show you a way to add HTTPS support to Varnish, with Nginx performing the SSL termination. We will assume that you already have a web application server set up, and we will use a generic LAMP (Linux, Apache, MySQL, PHP) server as our starting point.


Varnish Cache is a caching HTTP reverse proxy, or HTTP accelerator, which reduces the time it takes to serve content to a user. The main technique it uses is caching responses from a web or application server in memory, so future requests for the same content can be served without having to retrieve it from the web server. Performance can be improved greatly in a variety of environments, and it is especially useful when you have content-heavy dynamic web applications. Varnish was built with caching as its primary feature but it also has other uses, such as reverse proxy load balancing.


In many cases, Varnish works well with its defaults but keep in mind that it must be tuned to improve performance with certain applications, especially ones that use cookies. In depth tuning of Varnish is outside of the scope of this tutorial.


# Prerequisites


In this tutorial, we assume that you already have a web application server that is listening on HTTP (port 80) on its private IP address. If you do not already have a web server set up, use the following link to set up your own LAMP stack: How To Install Linux, Apache, MySQL, PHP (LAMP) stack on Ubuntu 14.04. We will refer to this server as LAMP_VPS.





You will need to create a new Ubuntu 14.04 VPS which will be used for your Varnish installation. Create a non-root user with sudo permissions by completing steps 1-4 in the initial server setup for Ubuntu 14.04 guide. We will refer to this server as Varnish_VPS.


Keep in mind that the Varnish server will be receiving user requests and should be adequately sized for the amount of traffic you expect to receive.


# Our Goal





Our goal is to set up Varnish Cache in front of our web application server, so requests can be served quickly and efficiently. After the caching is set up, we will show you how to add HTTPS support to Varnish, by utlizing Nginx to handle incoming SSL requests. After your setup is complete, both your HTTP and HTTPS traffic will see the performance benefits of caching.


Now that you have the prerequisites set up, and you know what you are trying to build, let’s get started!


# Install Varnish


The recommended way to get the latest release of Varnish 4.0 is to install the package avaiable through the official repository.


Ubuntu 14.04 comes with apt-transport-https, but just run the following command on Varnish_VPS to be sure:


```
sudo apt-get install apt-transport-https

```


Now add the Varnish GPG key to apt:


```
curl https://repo.varnish-cache.org/ubuntu/GPG-key.txt | sudo apt-key add -

```


Then add the Varnish 4.0 repository to your list of apt sources:


```
sudo sh -c 'echo "deb https://repo.varnish-cache.org/ubuntu/ trusty varnish-4.0" >> /etc/apt/sources.list.d/varnish-cache.list'

```


Finally, update apt-get and install Varnish with the following commands:


```
sudo apt-get update
sudo apt-get install varnish

```


By default, Varnish is configured to listen on port 6081 and expects your web server to be on the same server and listening on port 8080. Open a browser and go to port 6081 of your server (replace the highlighted part with your public IP address or domain):


```
http://varnish_VPS_public_IP:6081

```


Because we installed Varnish on a new VPS, visiting port 6081 on your server’s public IP address or domain name will return the following error page:





This indicates that Varnish is installed and running, but it can’t find the web server that it is supposed to be caching. Let’s configure it to use our web server as a backend now.


# Configure Varnish


First, we will configure Varnish to use our LAMP_VPS as a backend.


The Varnish configuration file is located at /etc/varnish/default.vcl. Let’s edit it now:


```
sudo vi /etc/varnish/default.vcl

```


Find the following lines:


```
backend default {
    .host = "127.0.0.1";
    .port = "8080";
}

```


And change the values of host and port match your LAMP server private IP address and listening port, respectively.  Note that we are assuming that your web application is listening on its private IP address and port 80. If this is not the case, modify the configuration to match your needs:


```
backend default {
    .host = "LAMP_VPS_private_IP";
    .port = "80";
}

```


Varnish has a feature called “grace mode” that, when enabled, instructs Varnish to serve a cached copy of requested pages if your web server backend goes down and becomes unavailable. Let’s enable that now. Find the following sub vcl_backend_response block, and add the following highlighted lines to it:


```
sub vcl_backend_response {
    set beresp.ttl = 10s;
    set beresp.grace = 1h;
}

```


This sets the grace period of cached pages to one hour, meaning Varnish will continue to serve cached pages for up to an hour if it can’t reach your web server to look for a fresh copy. This can be handy if your application server goes down and you prefer that stale content is served to users instead of an error page (like the 503 error that we’ve seen previously), while you bring your web server back up.


Save and exit the default.vcl file.


We will want to set Varnish to listen on the default HTTP port (80), so your users will be able to access your site without adding an unusual port number to your URL. This can be set in the /etc/default/varnish file. Let’s edit it now:


```
sudo vi /etc/default/varnish

```


You will see a lot of lines, but most of them are commented out. Find the following DAEMON_OPTS line (it should be uncommented already):


```
DAEMON_OPTS="-a :6081 \

```


The -a option is used to assign the address and port that Varnish will listen for requests on. Let’s change it to listen to the default HTTP port, port 80. After your modification, it should look like this:


```
DAEMON_OPTS="-a :80 \

```


Save and exit.


Now restart Varnish to put the changes into effect:


```
sudo service varnish restart

```


Now test it out with a web browser, by visiting your Varnish server by its public IP address, on port 80 (HTTP) this time:


```
http://varnish_VPS_public_IP

```


You should see the same thing that is served from your LAMP_VPS. In our case, it’s just a plain Apache2 Ubuntu page:





At this point, Varnish is caching our application server–hopefully will you see performance benefits in decreased response time. If you had a domain name pointing to your existing application server, you may change its DNS entry to point to your Varnish_VPS_public_IP.


Now that we have the basic caching set up, let’s add SSL support with Nginx!


# SSL Support with Nginx (Optional)


Varnish does not support SSL termination natively, so we will install Nginx for the sole purpose of handling HTTPS traffic. We will cover the steps to install and configure Nginx with a self-signed SSL certificate, and reverse proxy traffic from an HTTPS connection to Varnish over HTTP.


If you would like a more detailed explanation of setting up a self-signed SSL certificate with Nginx, refer to this link: SSL with Nginx for Ubuntu. If you want to try out a certificate from StartSSL, here is a tutorial that covers that.


Let’s install Nginx.


## Install Nginx


On Varnish_VPS, let’s install Nginx with the following apt command:


```
sudo apt-get install nginx

```


After the installation is complete, you will notice that Nginx is not running. This is because it is configured to listen on port 80 by default, but Varnish is already using that port. This is fine because we want to listen on the default HTTPS port, port 443.


Let’s generate the SSL certificate that we will use.


## Generate Self-signed SSL Certificate


On Varnish_VPS, create a directory where SSL certificate can be placed:


```
sudo mkdir /etc/nginx/ssl

```


Generate a self-signed, 2048-bit SSL key and certicate pair:


```
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/nginx/ssl/nginx.key -out /etc/nginx/ssl/nginx.crt

```


Make sure that you set common name to match your domain name. This particular certificate will expire in a year.


Now that we have our certificate in place, let’s configure Nginx to use it.


## Configure Nginx


Open the default Nginx server block configuration for editing:


```
sudo vi /etc/nginx/sites-enabled/default

```


Delete everything in the file and replace it with the following (and change the server_name to match your domain name):


```
server {
        listen 443 ssl;

        server_name example.com;
        ssl_certificate /etc/nginx/ssl/nginx.crt;
        ssl_certificate_key /etc/nginx/ssl/nginx.key;

        location / {
            proxy_pass http://127.0.0.1:80;
            proxy_set_header X-Real-IP  $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto https;
            proxy_set_header X-Forwarded-Port 443;
            proxy_set_header Host $host;
        }
}

```


Save and exit. The above configuration has a few important lines that we will explain in more detail:


- ssl_certificate: specifies SSL certificate location
- ssl_certificate_key: specifies SSL key location
- listen 443 ssl: configures Nginx to listen on port 443
- server_name: specifies your server name, and should match the common name of your SSL certificate
- proxy_pass http://127.0.0.1:80;: redirects traffic to Varnish (which is running on port 80 of 127.0.0.1 (i.e. localhost)

The other proxy_set_header lines tell Nginx to forward information, such as the original user’s IP address, along with any user requests.


Now let’s start Nginx so our server can handle HTTPS requests.


```
sudo service nginx start

```


Now test it out with a web browser, by visiting your Varnish server by its public IP address, on port 443 (HTTPS) this time:


```
https://varnish_VPS_public_IP

```


Note: If you used a self-signed certificate, you will see a warning saying something like “The site’s security certificate is not trusted”. Since you know you just created the certificate, it is safe to proceed.


Again, you should see the same application page as before. The difference is that you are actually visiting the Nginx server, which handles the SSL encryption and forwards the unencrypted request to Varnish, which treats the request like it normally does.


## Configure Backend Web Server


If your backend web server is binding to all of its network interfaces (i.e. public and private network interfaces), you will want to modify your web server configuration so it is only listening on its private interface. This is to prevent users from accessing your backend web server directly via its public IP address, which would bypass your Varnish Cache.


In Apache or Nginx, this would involve assigning the value of the listen directives to bind to the private IP address of your backend server.


# Troubleshooting Varnish


If you are having trouble getting Varnish to serve your pages properly, here are a few commands that will help you see what Varnish is doing behind the scenes.


## Stats


If you want to get an idea of how well your cache is performing, you will want to take a look at the varnishstat command. Run it like this:


```
varnishstat

```


You will a screen that looks like the following:





There is a large variety of stats that come up, and using the up/down arrows to scroll will show you a short description of each item. The cache_hit stat shows you how many requests were served with a cached result–you want this number to be as close to the total number of client requests (client_req) as possible.


Press q to quit.


## Logs


If you want to get a detailed view of how Varnish is handling each individual request, in the form of a streaming log, you will want to use the varnishlog command. Run it like this:


```
varnishlog

```


Once it is running, try and access your Varnish server via a web browser. For each request you send to Varnish, you will see a detailed output that can be used to help troubleshoot and tune your Varnish configuration.


Press CTRL + C to quit.


# Conclusion


Now that your web server has a Varnish Cache server in front of it, you will see improved performance in most cases. Remember that Varnish is very powerful and tuneable, and it may require additional tweaks to get the full benefit from it.


