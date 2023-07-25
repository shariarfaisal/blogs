# How To Secure Tomcat 10 with Apache or Nginx on Ubuntu 20 04

```Nginx``` ```Ubuntu 20.04``` ```Apache```

The author selected the Free and Open Source Fund to receive a donation as part of the Write for DOnations program.


## Introduction


Apache Tomcat is a web server and servlet container used to serve Java applications. It’s an open-source implementation of the Jakarta Servlet, Jakarta Server Pages, and other technologies of the Jakarta EE platform.


Upon installation, Tomcat serves traffic unencrypted by default, including passwords or other sensitive data. To secure your Tomcat installation, you will integrate Let’s Encrypt TLS certificates to all HTTP connections. To incorporate TLS, you can set up a reverse proxy with configured certificates, which will securely negotiate with clients and hand requests to Tomcat.


While TLS connections can be configured in Tomcat itself, it’s not recommended because Tomcat does not have the latest TLS standard and security updates available. In this tutorial, you will configure this connection with either Apache or Nginx. They are both widespread and well-tested, whereas software that automates Let’s Encrypt certificate provision, such as certbot, does not provide support for Tomcat. If you want to try both Apache and Nginx, you will need separate servers for each.


# Prerequisites


- An Ubuntu 20.04 server with a sudo non-root user and a firewall, which you can set up by following the Ubuntu 20.04 initial server setup guide.
- Tomcat 10 installed on your server, which you can set up by following How To Install Apache Tomcat 10 on Ubuntu 20.04.
- A registered domain name. This tutorial will use your_domain as an example throughout. You can purchase a domain name on Namecheap, get one for free on Freenom, or use the domain registrar of your choice.
- A DNS record pointing to your server’s public IP address. You can follow this introduction to DigitalOcean DNS for details on adding it.

# Option 1 – Using Apache as a Reverse Proxy


## Section Prerequisites


- Apache installed by following How To Install Apache on Ubuntu 20.04. Be sure that you have a virtual host file for your domain.
- Let’s Encrypt TLS certificates installed on your server for your domain. Follow the steps outlined in the Let’s Encrypt guide for Apache. When prompted, enable redirection to the secure domain.

## Step 1 – Configuring Virtual Hosts in Apache


Because you set up a basic virtual host at your domain and secured it using Let’s Encrypt in the section prerequisites, you have two virtual hosts available. You will only need to edit the one configuring the HTTPS traffic. To list them, show Apache config by running:


```
sudo apache2ctl -S


```


The output will be similar to this:


```
Output...
VirtualHost configuration:
*:443                  your_domain (/etc/apache2/sites-enabled/your_domain-le-ssl.conf:2)
*:80                   your_domain (/etc/apache2/sites-enabled/your_domain.conf:1)
ServerRoot: "/etc/apache2"
Main DocumentRoot: "/var/www/html"
Main ErrorLog: "/var/log/apache2/error.log"
Mutex watchdog-callback: using_defaults
Mutex rewrite-map: using_defaults
Mutex ssl-stapling-refresh: using_defaults
Mutex ssl-stapling: using_defaults
Mutex ssl-cache: using_defaults
Mutex default: dir="/var/run/apache2/" mechanism=default
PidFile: "/var/run/apache2/apache2.pid"
Define: DUMP_VHOSTS
Define: DUMP_RUN_CFG
User: name="www-data" id=33
Group: name="www-data" id=33

```


The file providing HTTPS configuration is /etc/apache2/sites-enabled/your_domain-le-ssl.conf. Open it for editing by running the following command, replacing your_domain with your domain name:


```
sudo nano /etc/apache2/sites-enabled/your_domain-le-ssl.conf


```


The file will look similar to this:


/etc/apache2/sites-enabled/your_domain-le-ssl.conf
```
<IfModule mod_ssl.c>
<VirtualHost *:443>
    ServerAdmin webmaster@localhost
    ServerName your_domain
    ServerAlias www.your_domain
    DocumentRoot /var/www/your_domain.conf
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

SSLCertificateFile /etc/letsencrypt/live/your_domain/fullchain.pem
SSLCertificateKeyFile /etc/letsencrypt/live/your_domain/privkey.pem
Include /etc/letsencrypt/options-ssl-apache.conf
</VirtualHost>
</IfModule>

```


Add the highlighted lines to the VirtualHost:


/etc/apache2/sites-enabled/your_domain-le-ssl.conf
```
...
    CustomLog ${APACHE_LOG_DIR}/access.log combined
    
    ProxyPreserveHost On
    ProxyPass / http://127.0.0.1:8080/
    ProxyPassReverse / http://127.0.0.1:8080/
...

```


The three highlighted directives instruct Apache to enable two-way traffic between Tomcat and the outside world while preserving HTTP header values. When finished, save and close the file.


You have now instructed Apache to proxy traffic to your Tomcat installation, but that functionality is not enabled by default.


# Step 2 – Testing New Apache Configuration


In this step, you will enable mod_proxy and mod_proxy_http, which are Apache modules that facilitate a connection proxy. Run the following commands to enable them:


```
sudo a2enmod proxy
sudo a2enmod proxy_http


```


Then, check the configuration by typing:


```
sudo apache2ctl configtest


```


The output should end with Syntax OK. If there are any errors, review the configuration that you just modified.


Finally, restart the Apache web server process:


```
sudo systemctl restart apache2


```


You should now see your Tomcat installation secured with TLS certificates when accessing your_domain from your web browser. You can skip past the Nginx section and follow the step for restricting access to Tomcat.


# Option 2 – Using Nginx as a Reverse Proxy


## Section Prerequisites


- Nginx installed with a server block for your domain. This section will use /etc/nginx/sites-available/your_domain as an example. You can set up Nginx by following How To Install Nginx on Ubuntu 20.04.
- Let’s Encrypt TLS certificates installed on your server for your domain. Follow the steps outlined in the Let’s Encrypt guide for Nginx. When prompted, enable redirection to the secure domain.

## Step 1 – Adjusting the Nginx Server Block Configuration


In this step, you will modify the server block configuration for the domain you created in the section prerequisites to make Nginx aware of Tomcat.


Open the config file for editing with the following command:


```
sudo nano /etc/nginx/sites-available/your_domain


```


Add the following lines at the top of the file:


/etc/nginx/sites-available/your_domain
```
upstream tomcat {
    server 127.0.0.1:8080 fail_timeout=0;
}

```


The upstream block defines how to connect to Tomcat, which lets Nginx know where Tomcat is located.


Next, within the server block defined for port 443, replace the contents of the location / block with the highlighted directives:


/etc/nginx/sites-available/your_domain
```
upstream tomcat {
    server 127.0.0.1:8080 fail_timeout=0;
}

server {
...
    location / {
        include proxy_params;
        proxy_pass http://tomcat/;
    }
...

```


These two lines specify that all traffic should go to the upstream block called tomcat, which you just defined. You also order all HTTP parameters to be preserved during the proxying process. Save and close the file when finished.


You will now test this configuration by accessing Tomcat at your domain.


## Step 2 – Testing New Nginx Configuration


To test that your configuration changes did not introduce syntax errors, run this command:


```
sudo nginx -t


```


The output should look like this:


```
Outputnginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

```


If you see any errors, return to the previous step to review the configuration that you modified.


Then, restart Nginx to reload the configuration:


```
sudo systemctl restart nginx


```


You can now see your Tomcat installation when visiting the TLS secured variant of your domain in your web browser:


```
https://your_domain

```


# Restricting Access to Tomcat for either Apache or Nginx


Now that you have exposed your Tomcat installation at your domain through a proxy server with TLS certificates, you can harden your Tomcat installation by restricting access to it.


All HTTP requests to Tomcat should come through the proxy, but you can configure Tomcat to only listen for connections on the local loopback interface. This configuration ensures that outside parties cannot attempt to make requests to Tomcat directly.


Open the server.xml file for editing (located within your Tomcat configuration directory):


```
sudo nano /opt/tomcat/conf/server.xml


```


Find the Connector definition under the Service named Catalina, which looks like:


/opt/tomcat/conf/server.xml
```
...
    <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
...

```


To restrict access to the local loopback interface, specify 127.0.0.1 as the address parameter:


/opt/tomcat/conf/server.xml
```
...
    <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               address="127.0.0.1"
               redirectPort="8443" />
...

```


When finished, save and close the file.


For the changes to take place, restart Tomcat by running:


```
sudo systemctl restart tomcat


```


Your Tomcat installation should now only be accessible through your Apache or Nginx web server proxy. Apache or Nginx shields Tomcat from the Internet, so it will not be accessible at the 8080 port at your domain.


# Conclusion


In this tutorial, you set up Tomcat behind a proxy server secured with free Let’s Encrypt TLS certificates. You also disallowed direct outside access by restricting connections to Tomcat through the local loopback interface (localhost), which means that only local applications, such as Apache or Nginx, can connect.


While configuring a separate web server process might increase the software involved in serving your applications, it simplifies the process of securing Tomcat traffic. For more information about the process of proxying traffic, visit the official docs for Apache and Nginx.


