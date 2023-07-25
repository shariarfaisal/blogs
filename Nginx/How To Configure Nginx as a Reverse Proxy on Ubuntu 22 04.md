# How To Configure Nginx as a Reverse Proxy on Ubuntu 22 04

```Nginx``` ```Ubuntu 22.04```

## Introduction


A reverse proxy is the recommended method to expose an application server to the internet. Whether you are running a Node.js application in production or a minimal built-in web server with Flask, these application servers will often bind to localhost with a TCP port. This means by default, your application will only be accessible locally on the machine it resides on. While you can specify a different bind point to force access through the internet, these application servers are designed to be served from behind a reverse proxy in production environments. This provides security benefits in isolating the application server from direct internet access, the ability to centralize firewall protection, and a minimized attack plane for common threats such as denial of service attacks.


From a client’s perspective, interacting with a reverse proxy is no different from interacting with the application server directly. It is functionally the same, and the client cannot tell the difference. A client requests a resource and then receives it, without any extra configuration required by the client.


This tutorial will demonstrate how to set up a reverse proxy using Nginx, a popular web server and reverse proxy solution. You will install Nginx, configure it as a reverse proxy using the proxy_pass directive, and forward the appropriate headers from your client’s request. If you don’t have an application server on hand to test, you will optionally set up a test application with the WSGI server Gunicorn.


# Prerequisites


To complete this tutorial, you will need:


- An Ubuntu 22.04 server, set up according to our initial server setup guide for Ubuntu 22.04,
- The address of the application server you want to proxy, this will be referred to as app_server_address throughout the tutorial. This can be an IP address with TCP port (such as the Gunicorn default of http://127.0.0.1:8000), or a unix domain socket (such as http://unix:/tmp/pgadmin4.sock for pgAdmin). If you do not have an application server set up to test with, you will be guided through setting up a Gunicorn application which will bind to http://127.0.0.1:8000.
- A domain name pointed at your server’s public IP. This will be configured with Nginx to proxy your application server.

# Step 1 — Installing Nginx


Nginx is available for installation with apt through the default repositories. Update your repository index, then install Nginx:


```
sudo apt update
sudo apt install nginx


```


Press Y to confirm the installation. If you are asked to restart services, press ENTER to accept the defaults.


You need to allow access to Nginx through your firewall. Having set up your server according to the initial server prerequisites, add the following rule with ufw:


```
sudo ufw allow 'Nginx HTTP'


```


Now you can verify that Nginx is running:


```
systemctl status nginx


```


```
Output● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2022-08-29 06:52:46 UTC; 39min ago
       Docs: man:nginx(8)
   Main PID: 9919 (nginx)
      Tasks: 2 (limit: 2327)
     Memory: 2.9M
        CPU: 50ms
     CGroup: /system.slice/nginx.service
             ├─9919 "nginx: master process /usr/sbin/nginx -g daemon on; master_process on;"
             └─9920 "nginx: worker process

```


Next you will add a custom server block with your domain and app server proxy.


# Step 2 — Configuring your Server Block


It is recommended practice to create a custom configuration file for your new server block additions, instead of editing the default configuration directly. Create and open a new Nginx configuration file using nano or your preferred text editor:


```
sudo nano /etc/nginx/sites-available/your_domain


```


Insert the following into your new file, making sure to replace your_domain and app_server_address. If you do not have an application server to test with, default to using http://127.0.0.1:8000 for the optional Gunicorn server setup in Step 3:


/etc/nginx/sites-available/your_domain
```
server {
    listen 80;
    listen [::]:80;

    server_name your_domain www.your_domain;
        
    location / {
        proxy_pass app_server_address;
        include proxy_params;
    }
}

```


Save and exit, with nano you can do this by hitting CTRL+O then CTRL+X.


This configuration file begins with a standard Nginx setup, where Nginx will listen on port 80 and respond to requests made to your_domain and www.your_domain. Reverse proxy functionality is enabled through Nginx’s proxy_pass directive. With this configuration, navigating to your_domain in your local web browser will be the same as opening app_server_address on your remote machine. While this tutorial will only proxy a single application server, Nginx is capable of serving as a proxy for multiple servers at once. By adding more location blocks as needed, a single server name can combine multiple application servers through proxy into one cohesive web application.


All HTTP requests come with headers, which contain information about the client who sent the request. This includes details like IP address, cache preferences, cookie tracking, authorization status, and more. Nginx provides some recommended header forwarding settings you have included as proxy_params, and the details can be found in /etc/nginx/proxy_params:


/etc/nginx/proxy_params
```
proxy_set_header Host $http_host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;

```


With reverse proxies, your goal is to pass on relevant information about the client, and sometimes information about your reverse proxy server itself. There are use cases where a proxied server would want to know which reverse proxy server handled the request, but generally the important information is from the original client’s request. In order to pass on these headers and make information available in locations where it is expected, Nginx uses the proxy_set_header directive.


By default, when Nginx acts as a reverse proxy it alters two headers, strips out all empty headers, then passes on the request. The two altered headers are the Host and Connection header. There are many HTTP headers available, and you can check this detailed list of HTTP headers for more information on each of their purposes, though the relevant ones for reverse proxies will be covered here later.


Here are the headers forwarded by proxy_params and the variables it stores the data in:


- Host: This header contains the original host requested by the client, which is the website domain and port. Nginx keeps this in the $http_host variable.
- X-Forwarded-For: This header contains the IP address of the client who sent the original request. It can also contain a list of IP addresses, with the original client IP coming first, then a list of all the IP addresses of the reverse proxy servers that passed the request through. Nginx keeps this in the $proxy_add_x_forwarded_for variable.
- X-Real-IP: This header always contains a single IP address that belongs to the remote client. This is in contrast to the similar X-Forwarded-For which can contain a list of addresses. If X-Forwarded-For is not present, it will be the same as X-Real-IP.
- X-Forwarded-Proto: This header contains the protocol used by the original client to connect, whether it be HTTP or HTTPS. Nginx keeps this in the $scheme variable.

Next, enable this configuration file by creating a link from it to the sites-enabled directory that Nginx reads at startup:


```
sudo ln -s /etc/nginx/sites-available/your_domain /etc/nginx/sites-enabled/


```


You can now test your configuration file for syntax errors:


```
sudo nginx -t


```


With no problems reported, restart Nginx to apply your changes:


```
sudo systemctl restart nginx


```


Nginx is now configured as a reverse proxy for your application server, and you can access it from a local browser if your application server is running. If you have an intended application server but do not have it running, you can proceed to starting your intended application server. You can skip the remainder of this tutorial.


Otherwise, proceed to setting up a test application and server with Gunicorn in the next step.


# Step 3 — Testing your Reverse Proxy with Gunicorn (Optional)


If you had an application server prepared and running before beginning this tutorial, you can visit it in your browser now:


```
your_domain

```


However, if you don’t have an application server on hand to test your reverse proxy, you can go through the following steps to install Gunicorn along with a test application. Gunicorn is a Python WSGI server that is often paired with an Nginx reverse proxy.


Update your apt repository index and install gunicorn:


```
sudo apt update
sudo apt install gunicorn


```


You also have the option to install Gunicorn through pip with PyPI for the latest version that can be paired with a Python virtual environment, but apt is used here as a quick test bed.


Next, you’ll write a Python function to return “Hello World!” as an HTTP response that will render in a web browser. Create test.py using nano or your preferred text editor:


```
nano test.py


```


Insert the following Python code into the file:


```
def app(environ, start_response):
    start_response("200 OK", [])
    return iter([b"Hello, World!"])

```


This is the minimum required code by Gunicorn to start an HTTP response that renders a string of text in your web browser. After reviewing the code, save and close your file.


Now start your Gunicorn server, specifying the test Python module and the app function within it. Starting the server will take over your terminal:


```
gunicorn --workers=2 test:app


```


```
Output[2022-08-29 07:09:29 +0000] [10568] [INFO] Starting gunicorn 20.1.0
[2022-08-29 07:09:29 +0000] [10568] [INFO] Listening at: http://127.0.0.1:8000 (10568)
[2022-08-29 07:09:29 +0000] [10568] [INFO] Using worker: sync
[2022-08-29 07:09:29 +0000] [10569] [INFO] Booting worker with pid: 10569
[2022-08-29 07:09:29 +0000] [10570] [INFO] Booting worker with pid: 10570

```


The output confirms that Gunicorn is listening at the default address of http://127.0.0.1:8000. This is the address that you set up previously in your Nginx configuration to proxy. If not, go back to your /etc/nginx/sites-available/your_domain file and edit the app_server_address associated with the proxy_pass directive.


Open your web browser and navigate to the domain you set up with Nginx:


```
your_domain

```


Your Nginx reverse proxy is now serving your Gunicorn web application server, displaying “Hello World!”.


# Conclusion


With this tutorial you have configured Nginx as a reverse proxy to enable access to your application servers that would otherwise only be available locally. Additionally, you configured the forwarding of request headers, passing on the client’s header information.


For examples of a complete solution using Nginx as a reverse proxy, check out how to serve Flask applications with Gunicorn and Nginx on Ubuntu 22.04 or how to run a Meilisearch frontend using InstantSearch on Ubuntu 22.04.


