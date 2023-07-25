# How To Install Nginx on Ubuntu 20 04 [Quickstart]

```Ubuntu``` ```Ubuntu 20.04``` ```Quickstart``` ```Nginx```

## Introduction


Nginx is one of the most popular web servers in the world and is responsible for hosting some of the largest and highest-traffic sites on the internet. It is more resource-friendly than Apache in most cases and can be used as a web server or reverse proxy.


In this guide, we’ll explain how to install Nginx on your Ubuntu 20.04 server. For a more detailed version of this tutorial, please refer to How To Install Nginx on Ubuntu 20.04.


# Step 1 – Installing Nginx


Because Nginx is available in Ubuntu’s default repositories, you can install it using the apt packaging system.


Update your local package index:


```
sudo apt update


```


Install Nginx:


```
sudo apt install nginx


```


# Step 2 – Adjusting the Firewall


If you followed the prerequisite server setup tutorial, then you have the UFW firewall enabled. Check the available ufw application profiles with the following command:


```
sudo ufw app list


```


```
OutputAvailable applications:
  Nginx Full
  Nginx HTTP
  Nginx HTTPS
  OpenSSH

```


Let’s enable the most restrictive profile that will still allow the traffic you’ve configured, permitting traffic on port 80:


```
sudo ufw allow 'Nginx HTTP'


```


Verify the change:


```
sudo ufw status


```


```
OutputStatus: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere                  
Nginx HTTP                 ALLOW       Anywhere                  
OpenSSH (v6)               ALLOW       Anywhere (v6)             
Nginx HTTP (v6)            ALLOW       Anywhere (v6)

```


# Step 3 – Checking your Web Server


Check with the systemd init system to make sure the service is running by typing:


```
systemctl status nginx


```


```
Outputnginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset:>
     Active: active (running) since Mon 2020-05-04 22:45:26 UTC; 1min 17s ago
       Docs: man:nginx(8)
   Main PID: 13255 (nginx)
      Tasks: 2 (limit: 1137)
     Memory: 4.6M
     CGroup: /system.slice/nginx.service
             ├─13255 nginx: master process /usr/sbin/nginx -g daemon on; master>
             └─13256 nginx: worker process

```


Access the default Nginx landing page to confirm that the software is running properly through your IP address:


```
http://your_server_ip

```


You should receive the default Nginx landing page:





# Step 4 – Setting Up Server Blocks (Recommended)


When using the Nginx web server, server blocks (similar to virtual hosts in Apache) can be used to encapsulate configuration details and host more than one domain from a single server. We will set up a domain called your_domain, but you should replace this with your own domain name. To learn more about setting up a domain name with DigitalOcean, please refer to our Introduction to DigitalOcean DNS.


Create the directory for your_domain, using the -p flag to create any necessary parent directories:


```
sudo mkdir -p /var/www/your_domain/html


```


Assign ownership of the directory:


```
sudo chown -R $USER:$USER /var/www/your_domain/html


```


The permissions of your web roots should be correct if you haven’t modified your umask value, but you can make sure by typing:


```
sudo chmod -R 755 /var/www/your_domain


```


Create a sample index.html page using nano or your favorite editor:


```
nano /var/www/your_domain/html/index.html


```


Inside, add the following sample HTML:


/var/www/your_domain/html/index.html
```
<html>
    <head>
        <title>Welcome to your_domain!</title>
    </head>
    <body>
        <h1>Success!  The your_domain server block is working!</h1>
    </body>
</html>

```


Save and close the file when you are finished.


Make a new server block at /etc/nginx/sites-available/your_domain:


```
sudo nano /etc/nginx/sites-available/your_domain


```


Paste in the following configuration block, updated for our new directory and domain name:


/etc/nginx/sites-available/your_domain
```
server {
        listen 80;
        listen [::]:80;

        root /var/www/your_domain/html;
        index index.html index.htm index.nginx-debian.html;

        server_name your_domain www.your_domain;

        location / {
                try_files $uri $uri/ =404;
        }
}

```


Save and close the file when you are finished.


Enable the file by creating a link from it to the sites-enabled directory:


```
sudo ln -s /etc/nginx/sites-available/your_domain /etc/nginx/sites-enabled/


```


Two server blocks are now enabled and configured to respond to requests based on their listen and server_name directives:


- your_domain: Will respond to requests for your_domain and www.your_domain.
- default: Will respond to any requests on port 80 that do not match the other two blocks.

To avoid a possible hash bucket memory problem that can arise from adding additional server names, it is necessary to adjust a single value in the /etc/nginx/nginx.conf file.  Open the file:


```
sudo nano /etc/nginx/nginx.conf


```


Find the server_names_hash_bucket_size directive and remove the # symbol to uncomment the line:


/etc/nginx/nginx.conf
```
...
http {
    ...
    server_names_hash_bucket_size 64;
    ...
}
...

```


Test for syntax errors:


```
sudo nginx -t


```


Restart Nginx to enable your changes:


```
sudo systemctl restart nginx


```


Nginx should now be serving your domain name. You can test this by navigating to http://your_domain, where you should receive something like this:





# Conclusion


Now that you have your web server installed, you have many options for the type of content to serve and the technologies you want to use to create a richer experience.


If you’d like to build out a more complete application stack, check out this article on how to configure a LEMP stack on Ubuntu 20.04.


