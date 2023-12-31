# How To Use Apache HTTP Server As Reverse-Proxy Using mod_proxy Extension

```Apache``` ```Ubuntu``` ```Debian```

## Introduction



Apache is a tried and tested HTTP server which comes with access to a very wide range of powerful extensions. Although it might not seem like the go-to choice in terms of running a reverse-proxy, system administrators who already depend on Apache for the available rich feature-set can also use it as a gateway to their application servers. In most cases, this will translate to removing an additional layer from their server set up or the need to use yet another tool just to redirect connections.


In this DigitalOcean article, we are going to see set up Apache on Ubuntu 13 and use it as a reverse-proxy to welcome incoming connections and redirect them to application server(s) running on the same network. For this purpose, we are going to use and work with the mod_proxy extension and several other related Apache modules.


# Glossary



## 1. Apache



## 2. Apache Working As A Reverse-Proxy Using mod_proxy



## 3. Installing Apache And mod_proxy



1. Updating The Operating-System
2. Getting The Essential Build Tools
3. Getting The Modules And Dependencies

## 4. Configuring Apache To Proxy Connections



1. Activating The Modules
2. Modifying The Default Configuration
3. Enabling Load-Balancing
4. Enabling SSL Support
5. Restarting Apache

# Apache



Apache HTTP server does not require an introduction, since it is probably the most famous and popular web-server that exists. It is possible to run Apache very easily on many different platforms and set ups. The application comes with a lot of third party modules to handle different kind of tasks (mod_rewrite for rule-based URL rewriting) and one of them, albeit nowadays relatively neglected, is mod_proxy: The Apache Module to implement a proxy (or gateway) for servers running on the back-end.



Tip: According to some articles, Apache’s name comes from server’s “patchy” nature - i.e. it being a collection of application patches (or modules).

Note: To learn more about Apache, you can check out the Wikipedia entry on the subject - Apache HTTP Server.


# Apache Working As A Reverse-Proxy Using mod_proxy



mod_proxy is the Apache module for redirecting connections (i.e. a gateway, passing them through). It is enabled for use just like any other module and configuration is pretty basic (or standard), in line with others. mod_proxy is not just a single module but a collection of them, with each bringing a new set of functionality.


Some of these modules are:


- 
mod_proxy: The main proxy module for Apache that manages connections and redirects them.

- 
mod_proxy_http: This module implements the proxy features for HTTP and HTTPS protocols.

- 
mod_proxy_ftp: This module does the same but for FTP protocol.

- 
mod_proxy_connect: This one is used for SSL tunnelling.

- 
mod_proxy_ajp: Used for working with the AJP protocol.

- 
mod_proxy_wstunnel: Used for working with web-sockets (i.e. WS and WSS).

- 
mod_proxy_balancer: Used for clustering and load-balancing.

- 
mod_cache: Used for caching.

- 
mod_headers: Used for managing HTTP headers.

- 
mod_deflate: Used for compression.


Note: To learn more about Apache and mod_proxy, you can check out the official Apache documentation on the subject here.


# Installing Apache And mod_proxy



Note: Instructions given here are kept brief, since chances are you already have Apache installed or know how to use it. Nonetheless, by following the steps below you can get a new Ubuntu VPS running Apache in a matter of minutes.


## Updating The Operating-System



We will begin with preparing our virtual server. We are going to first upgrade the default available components to make sure that we have everything up-to-date.


Update the software sources list and upgrade the dated applications:


```
aptitude    update
aptitude -y upgrade


```


## Getting The Essential Build Tools



Let’s continue with getting the essential package for application building - the build-essential. This package contains tools necessary to install certain things from source.


Run the following command to install build-essential package:


```
aptitude install -y build-essential


```


## Getting The Modules And Dependencies



Next, we are going to get the module and dependencies.


Run the following command to install them:


```
aptitude install -y libapache2-mod-proxy-html libxml2-dev


```


# Configuring Apache To Proxy Connections



## Activating The Modules



Before configuring Apache, we are going to enable the necessary modules that we will be using in this tutorial, or which might come in handy in the future.


First, let’s verify that all modules are correctly installed and ready to be activated.


Run the following command to get a list of available Apache modules:


```
a2enmod

# You will be presented with an output similar to:

# Your choices are: access_compat actions alias allowmethods asis auth_basic auth_digest auth_form authn_anon authn_core authn_dbd authn_dbm authn_file authn_socache authnz_ldap authz_core authz_dbd authz_dbm authz_groupfile authz_host authz_owner authz_user autoindex buffer cache cache_disk cache_socache cgi cgid charset_lite data dav dav_fs dav_lock dbd deflate dialup dir dump_io echo env expires ext_filter file_cache filter headers heartbeat heartmonitor include info lbmethod_bybusyness lbmethod_byrequests lbmethod_bytraffic lbmethod_heartbeat ldap log_debug log_forensic lua macro mime mime_magic mpm_event mpm_itk mpm_prefork mpm_worker negotiation proxy proxy_ajp proxy_balancer proxy_connect proxy_express proxy_fcgi proxy_fdpass proxy_ftp proxy_html proxy_http proxy_scgi proxy_wstunnel ratelimit reflector remoteip reqtimeout request rewrite sed session session_cookie session_crypto session_dbd setenvif slotmem_plain slotmem_shm socache_dbm socache_memcache socache_shmcb speling ssl status substitute suexec unique_id userdir usertrack vhost_alias xml2enc
# Which module(s) do you want to enable (wildcards ok)?


```


Once you are prompted with the choice of modules you desire, you can pass the below line listing the module names:


The list of modules:


```
proxy proxy_ajp proxy_http rewrite deflate headers proxy_balancer proxy_connect proxy_html


```


Or alternatively, you can run the following commands to enable the modules one by one:


```
a2enmod proxy
a2enmod proxy_http
a2enmod proxy_ajp
a2enmod rewrite
a2enmod deflate
a2enmod headers
a2enmod proxy_balancer
a2enmod proxy_connect
a2enmod proxy_html


```


Note: Some modules are likely to be enabled by default. Trying to enable them twice will just ensure that they are active.


## Modifying The Default Configuration



In this step, we are going to see how to modify the default configuration file 000-default.conf inside /etc/apache2/sites-enabled to set up “proxying” functionality.


Run the following command to edit the default Apache virtual host using the nano text editor:


```
nano /etc/apache2/sites-enabled/000-default.conf


```


Here, we will be defining a proxy virtual host using mod_virtualhost and mod_proxy together.


Copy-and-paste the below block of configuration, amending it to suit your needs:


```
    <VirtualHost *:*>
        ProxyPreserveHost On
        
        # Servers to proxy the connection, or;
        # List of application servers:
        # Usage:
        # ProxyPass / http://[IP Addr.]:[port]/
        # ProxyPassReverse / http://[IP Addr.]:[port]/
        # Example: 
        ProxyPass / http://0.0.0.0:8080/
        ProxyPassReverse / http://0.0.0.0:8080/
        
        ServerName localhost
    </VirtualHost>

```


Press CTRL+X and confirm with Y to save and exit.


Note: To learn more about virtual host configurations, you can check out the detailed Apache manual on the subject by clicking here.


## Enabling Load-Balancing



If you have multiple back-end servers, a good way to distribute the connection when proxying them is to use Apache’s load balancing features.


Start editing the virtual-host settings like the previous step, but this time using the below configuration example:


```
    <Proxy balancer://mycluster>
        # Define back-end servers:

        # Server 1
        BalancerMember http://0.0.0.0:8080/
        
        # Server 2
        BalancerMember http://0.0.0.0:8081/
    </Proxy>
    
    <VirtualHost *:*>
        # Apply VH settings as desired
        # However, configure ProxyPass argument to
        # use "mycluster" to balance the load
        
        ProxyPass / balancer://mycluster
    </VirtualHost>

```


## Enabling SSL Reverse-Proxy Support



If you are dealing with SSL connections and certificates, you will also need to enable a secondary virtual host with below settings.


Repeat the steps from the previous steps but using these configuration options:


```
    Listen 443
     
    NameVirtualHost *:443
    <VirtualHost *:443>
    
        SSLEngine On
        
        # Set the path to SSL certificate
        # Usage: SSLCertificateFile /path/to/cert.pem
        SSLCertificateFile /etc/apache2/ssl/file.pem
        
        
        # Servers to proxy the connection, or;
        # List of application servers:
        # Usage:
        # ProxyPass / http://[IP Addr.]:[port]/
        # ProxyPassReverse / http://[IP Addr.]:[port]/
        # Example: 
        ProxyPass / http://0.0.0.0:8080/
        ProxyPassReverse / http://0.0.0.0:8080/
        
        # Or, balance the load:
        # ProxyPass / balancer://balancer_cluster_name
    
    </VirtualHost>

```


## Restarting Apache



Once you are happy with your configuration, you will need to restart the cloud server for the changes to go into effect.


Execute the following command to restart Apache:


```
service apache2 restart


```


And that’s it!


You can now visit your VPS and Apache shall reverse-proxy connections to your back-end application servers.


