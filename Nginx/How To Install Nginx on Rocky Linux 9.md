# How To Install Nginx on Rocky Linux 9

```Nginx``` ```Rocky Linux``` ```Rocky Linux 9```

## Introduction


Nginx is one of the most popular web servers in the world and is responsible for hosting some of the largest and highest-traffic sites on the internet. It is a lightweight choice that can be used as either a web server or reverse proxy.


In this guide, you’ll review how to install Nginx on your Rocky Linux 9 server, adjust the firewall, manage the Nginx process, and set up server blocks for hosting more than one domain from a single server.


# Prerequisites


Before you begin this guide, you should have a regular, non-root user with sudo privileges configured on your server. You can learn how to configure a regular user account by following our Initial server setup guide for Rocky Linux 9.


You will also optionally want to have registered a domain name before completing the last steps of this tutorial. To learn more about setting up a domain name with DigitalOcean, please refer to our Introduction to DigitalOcean DNS.


When you have an account available, log in as your non-root user to begin.


# Step 1 – Installing Nginx


Because Nginx is available in Rocky’s default repositories, you can install it with a single command, using the dnf package manager.


Install the nginx package with dnf install:


```
sudo dnf install nginx


```


When prompted, enter y to confirm that you want to install nginx. After that, dnf will install Nginx and any required dependencies on your server.


After the installation is finished, run the following commands to enable and start the web server:


```
sudo systemctl enable nginx
sudo systemctl start nginx


```


This will make Nginx restart automatically whenever your server reboots. Your new web server should now be up and running, but before testing it, you’ll probably need to make a change to your firewall configuration.


# Step 2 – Adjusting the Firewall


If you enabled the firewalld firewall as part of the initial server setup guide for Rocky Linux 9, you will need to adjust the firewall settings in order to allow external connections on your Nginx web server, which runs on port 80 by default.


Run the following command to permanently enable HTTP connections on port 80:


```
sudo firewall-cmd --permanent --add-service=http


```


To verify that the http firewall service was added correctly, you can run:


```
sudo firewall-cmd --permanent --list-all


```


You’ll see output like this:


```
Outputpublic
  target: default
  icmp-block-inversion: no
  interfaces: 
  sources: 
  services: cockpit dhcpv6-client http ssh
  ports: 
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 

```


To apply the changes, you’ll need to reload the firewall service:


```
sudo firewall-cmd --reload


```


Your web server should now be accessible to external visitors.


# Step 3 – Checking your Web Server


At this point, your web server should be up and running.


You can use the systemctl status command to make sure the service is running:


```
systemctl status nginx


```


```
Output● nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; vendor preset: disabled)
     Active: active (running) since Wed 2022-09-14 21:03:46 UTC; 7min ago
    Process: 18384 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
    Process: 18385 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
    Process: 18386 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
   Main PID: 18387 (nginx)
      Tasks: 3 (limit: 10938)
     Memory: 2.8M
        CPU: 43ms
     CGroup: /system.slice/nginx.service
             ├─18387 "nginx: master process /usr/sbin/nginx"
             ├─18388 "nginx: worker process"
             └─18389 "nginx: worker process"

```


As confirmed by this output, the service has started successfully. However, the best way to test this is to actually request a page from Nginx.


You can access the default Nginx landing page to confirm that the software is running properly by navigating to your server’s IP address. If you do not know your server’s IP address, you can find it by using the icanhazip.com tool, which will give you your public IP address as received from another location on the internet:


```
curl -4 icanhazip.com


```


When you have your server’s IP address, enter it into your browser’s address bar:


```
http://your_server_ip

```


You should receive the default Nginx landing page:





If you are on this page, your server is running correctly and is ready to be managed.


# Step 4 – Managing the Nginx Process


Now that you have your web server up and running, let’s review some service management commands.


To stop your web server, use systemctl stop:


```
sudo systemctl stop nginx


```


To start the web server when it is stopped, use systemctl start:


```
sudo systemctl start nginx


```


To stop and then start the service again, use systemctl restart:


```
sudo systemctl restart nginx


```


If you are only making configuration changes, Nginx can often reload without dropping connections. To do this, use systemctl reload:


```
sudo systemctl reload nginx


```


Earlier in this tutorial, you configured Nginx to start automatically when the server boots. You can disable this behavior by using systemctl disable:


```
sudo systemctl disable nginx


```


To re-enable the service to start up at boot, you can type:


```
sudo systemctl enable nginx


```


# Step 5 – Getting Familiar with Important Nginx Files and Directories


Now that you know how to manage the Nginx service, you should take a few minutes to familiarize yourself with a few important directories and files.


## Content


- /usr/share/nginx/html: The actual web content, which by default only consists of the default Nginx page you saw earlier, is served out of the /usr/share/nginx/html directory.  This can be changed by altering the Nginx configuration files.

## Server Configuration


- /etc/nginx: The Nginx configuration directory.  All of the Nginx configuration files reside here.
- /etc/nginx/nginx.conf: The main Nginx configuration file.  This can be modified to make changes to the Nginx global configuration.
- /etc/nginx/conf.d/: This directory contains server block configuration files, where you can define the websites that are hosted within Nginx. A typical approach is to have each website in a separate file that is named after the website’s domain name, such as your_domain.conf.

## Server Logs


- /var/log/nginx/access.log: Every request to your web server is recorded in this log file unless Nginx is configured to do otherwise.
- /var/log/nginx/error.log: Any Nginx errors will be recorded in this log.

You should now be ready to configure the site to host one or more domains.


# Step 6 – Setting Up Server Blocks (Optional)


When using the Nginx web server, server blocks (similar to virtual hosts in Apache) can be used to organize configuration details and host more than one domain from a single server. On Rocky Linux 9, server blocks are defined in .conf files located at /etc/nginx/conf.d. We will set up a domain called your_domain, but you should replace this with your own domain name.


By default, Nginx on Rocky Linux 9 is configured to serve documents out of a directory at /usr/share/nginx/html. While this works well for a single site, it can become unmanageable if you are hosting multiple sites. Instead of modifying /usr/share/nginx/html, you’ll create a directory structure within /var/www for the your_domain website, leaving /usr/share/nginx/html in place as the default directory to be served if a client request doesn’t match any other sites.


Create the directory for your_domain as follows, using the -p flag to create any necessary parent directories:


```
sudo mkdir -p /var/www/your_domain/html


```


Next, assign ownership of the directory with the $USER environment variable, which should reference your current system user:


```
sudo chown -R $USER:$USER /var/www/your_domain/html


```


Now you’ll create a sample index.html page to test the server block configuration. The default text editor that comes with Rocky Linux 9 is vi. vi is an extremely powerful text editor, but it can be somewhat obtuse for users who lack experience with it. You might want to install a more user-friendly editor such as nano to facilitate editing configuration files on your Rocky Linux 9 server:


```
sudo dnf install nano


```


Next, create a sample index.html page using nano or your favorite editor:


```
nano /var/www/your_domain/html/index.html


```


Inside, add the following sample HTML:


/var/www/your_domain/html/index.html
```
<html>
    <head>
        <title>Welcome to your_domain</title>
    </head>
    <body>
        <h1>Success! Your Nginx server is successfully configured for <em>your_domain</em>. </h1>
<p>This is a sample page.</p>
    </body>
</html>

```


Save and close the file when you are finished. If you are using nano, you can save and quit by pressing CTRL + X, then when prompted, Y and then Enter.


In order for Nginx to serve this content, you’ll need to create a server block with directives that point to your custom web root. Create a new server block at /etc/nginx/conf.d/your_domain.conf:


```
sudo nano /etc/nginx/conf.d/your_domain.conf


```


Paste in the following configuration block:


/etc/nginx/conf.d/your_domain.conf
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


Notice that we’ve updated the root configuration to our new directory, and the server_name to our domain name. Save and close the file.


Two server blocks are now enabled and configured to respond to requests based on their listen and server_name directives (you can read more about how Nginx processes these directives here):


- your_domain: Will respond to requests for your_domain and www.your_domain.
- default: Will respond to any requests on port 80 that do not match the other two blocks.

Next, test to make sure that there are no syntax errors in any of your Nginx files, using nginx -t:


```
sudo nginx -t


```


If there aren’t any problems, restart Nginx to enable your changes:


```
sudo systemctl restart nginx


```


Before you can test the changes from your browser, you’ll need to update your server’s SELinux security contexts so that Nginx is allowed to serve content from the /var/www/your_domain directory.


This chcon context update will allow your custom document root to be served as HTTP content:


```
chcon -vR system_u:object_r:httpd_sys_content_t:s0 /var/www/your_domain/


```


Nginx should now be serving your domain name. You can test this by navigating to http://your_domain, where you should see something like this:





# Conclusion


Now that you have your web server installed, you have many options for the type of content to serve and the technologies you want to use to create a richer experience.


To set up HTTPS for your domain name with a free SSL certificate using Let’s Encrypt, you should move on to How To Secure Nginx with Let’s Encrypt on Rocky Linux 9.


