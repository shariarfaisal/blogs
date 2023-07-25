# How To Configure the Apache Web Server on an Ubuntu or Debian VPS

```Apache``` ```Ubuntu``` ```Debian```

## Introduction


Apache is one of the most popular web servers on the internet. It is used to serve more than half of all active websites. Although there are many viable web servers that will serve your content, it is helpful to understand how Apache works because of its ubiquity.


This article will examine some general configuration files and the options that can be controlled within them. This article will follow the Ubuntu/Debian layout of Apache files, which is different from how other distributions build the configuration hierarchy.


# Prerequisites


Before you begin exploring your Apache configurations, you should have Apache installed on your server. You can learn how by following our How to Install the Apache Web Server on Ubuntu 20.04 tutorial or the How To Install the Apache Web Server on Debian 10 tutorial.


# The Apache File Hierarchy


Apache keeps its main configuration files within the /etc/apache2 folder. Executing the following command will list all of the files within this folder:


```
ls -f /etc/apache2


```


```
Outputenvars sites-available . apache2.conf .. sites-enabled mods-available ports.conf magic mods-enabled conf-enabled conf-available

```


There are a number of plaintext files and some subdirectories within this directory. Here are some useful locations to be familiar with:


- apache2.conf: This is the main configuration file for the server. Almost all configuration can be done from within this file, although it is recommended to use separate, designated files for simplicity. This file will configure defaults and be the central point of access for the server to read configuration details.
- ports.conf: This file is used to specify the ports that virtual hosts should listen on. Be sure to check that this file is correct if you are configuring SSL.
- sites-available/ and sites-enabled/: The sites-available directory contains virtual host file configurations. Configurations within this folder will establish which content gets served for which requests. This is enabled through linking to the sites-enabled directory, which stores activated virtual host configuration files. When Apache starts or reloads, it reads the configuration files and links from within the sites-enabled directory as it compiles a full configuration.
- conf-available/ and conf-enabled/: These directories house configuration fragments that are unattached to the virtual host configurations files.
- mods-enabled/ and mods-available/: These directories define modules that can be optionally loaded. The directories contain two components: files ending in .load, which contain fragments that load particular modules, and files ending in .conf, which store the configurations of these modules.

Apache configuration does not take place in a single monolithic file, but instead happens through a modular design where new files can be added and modified as needed.


# Exploring the Apache2.conf File


The main configuration details for your Apache server are held in the /etc/apache2/apache2.conf file.
This file is divided into three main sections:


- Configuration for the global Apache server process
- Configuration for the default server
- Configuration of virtual hosts.

Open this file with your preferred text editor. The following example uses nano:


```
sudo nano /etc/apache2/apache2.conf


```


In Ubuntu and Debian, this file is used to configure global definitions. The configuration of the default server and virtual hosts are handled by using the Include directive.
The Include directive allows Apache to read other configuration files into the current file at the location that the statement appears. The result is that Apache dynamically generates an overarching configuration file on startup.


Found within this file are a number of different Include and IncludeOptional statements. These directives load module definitions, the ports.conf document, the specific configuration files in the conf-enabled/ directory, and the virtual host definitions in the sites-enabled/ directory:


/etc/apache2/apache2.conf
```
…
IncludeOptional mods-enabled/*.load
IncludeOptional mods-enabled/*.conf
…
Include ports.conf
…
IncludeOptional conf-enabled/*.conf
…
IncludeOptional sites-enabled/*.conf

```


# Global Configurations


There are some options you may want to modify in the Global Configuration:


## Timeout


By default, this parameter is set to 300. This means that the server has a maximum of 300 seconds to fulfill each request.
This parameter can safely be dropped to something between 30 and 60 seconds.


## KeepAlive


This option, if set to On, will allow each connection to remain open to handle multiple requests from the same client.
If this is set to Off, each request will have to establish a new connection, which can result in significant overhead depending on your setup and traffic situation.


## MaxKeepAliveRequests


This controls how many separate requests each connection will handle before dying. Keeping this number high will allow Apache to serve content to each client more effectively.
The default setting is set to 100. Setting this value to 0 will allow Apache to serve an unlimited amount of requests for each connection.


## KeepAliveTimeout


This setting specifies how long to wait for the next request after finishing the last one. If the timeout threshold is reached, then the connection will die.
This means that the next time content is requested, the server will establish a new connection to handle the request for the content that makes up the page the client is visiting. The default is set to 5.


After examining the contents of this configuration file, you can close out of it by pressing CTRL+X.


# Multi-Processing Modules


A Multi-Processing Module (MPM) extends Apache’s modular design. MPMs are responsible for listening, directing, and handling different network requests. You can cross-reference which section your Apache installation was compiled in with using the following command:


```
apache2 -L


```


```
OutputCompiled in modules:
  core.c
  mod_so.c
  mod_watchdog.c
  http_core.c
  mod_log_config.c
  mod_logio.c
  mod_version.c
  mod_unixd.c

```


You can check the MPM type on your server with the a2query -M command:


```
a2query -M


```


```
Outputevent

```


The output reveals that the event MPM is used on this server. Your installation may have multiple to choose from, but only one can be selected.


# Virtual Host File


The default virtual host declaration can be found in a file called 000-default.conf within the sites-available/ directory. You can learn about the general format of a virtual host file by examining this file.


Open the file with the following command:


```
sudo nano /etc/apache2/sites-available/000-default.conf


```


/etc/apache2/sites-available/000-default.conf
```
<VirtualHost *:80>
…
ServerAdmin webmaster@localhost
DocumentRoot /var/www/html
…
ErrorLog ${APACHE LOG DIR}/error.log
CustomLog ${APACHE LOG DIR}/access.log combined
…

```


The default virtual host is configured to handle any request on port 80, the standard HTTP port. This is defined in the declaration header where it says *:80, meaning port 80 on any interface.
However, this does not mean that it will necessarily handle each request to the server on this port. Apache uses the most specific virtual host definition that matches the request. If there was a more specific definition, it could supersede this definition. After examining the file, you can close out of it by pressing CTRL+X.


## Virtual Host Configuration Options


The following options are set within the virtual host definition outside of any other lower level sub-declaration. They apply to the whole virtual host.
To start, open up the security.conf file within the conf-available/ directory:


```
sudo nano /etc/apache2/conf-available/security.conf


```


This file contains the Server Signature directive, which allows you to specify a contact email that should be used when there are server problems. You can change the default option from On to EMail to reveal the server admin email address. Make sure you are willing to receive the mail if you adjust this setting:


/etc/apache2/conf-available/security.conf
```
…
ServerSignature EMail
…

```


Exit the file by pressingCTRL+X. After editing a configuration file, a prompt will ask you to confirm your changes. Press Y to save the changes to your file or press N to discard them.


Within your virtual host file, you can add a ServerName directive that specifies the domain name or IP address that this request should handle. This is the option that would add specificity to the virtual host, allowing it to override the default definition if it matches the ServerName value.


Run the following command to open your virtual host file, making sure to replace the your_domain variable with your actual domain name:


```
sudo nano /etc/apache2/sites-available/your_domain.conf


```


Append your_domain to the ServerName directive:


/etc/apache2/sites-available/your_domain.conf
```
…
ServerName your_domain
…

```


Likewise, you can also make the virtual host apply to more than one name by using the ServerAlias directive. This provides alternate paths to get to the same content. A good use case for this is adding the same domain, preceded by www:


/etc/apache2/sites-available/your_domain.conf
```
…
ServerAlias www.your_domain.com
…

```


The DocumentRoot directive specifies where the content that is requested for this virtual host will be located. On Ubuntu, the default virtual host is set up to serve content out of the /var/www/ directory:


/etc/apache2/sites-available/your_domain.conf
```
…
DocumentRoot /var/www/your_domain/public_html
…

```


## Directory Definitions


Within the virtual host definition, there are definitions for how the server handles different directories within the file system. Apache will apply all of these directions in order from shortest to longest, so there is again a chance to override previous options.


Open the apache2.conf file with this command:


```
sudo nano /etc/apache2/apache2.conf


```


/etc/apache2/apache2.conf
```
…
<Directory />
        Options FollowSymLinks
        AllowOverride None
        Require all denied
</Directory>

<Directory /usr/share>
        AllowOverride None
        Require all granted
</Directory>

<Directory /var/www/>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
</Directory>
…

```


The first directory definition applies rules for the /, or root, directory. This will provide the baseline configuration for your virtual host, as it applies to all files served on the file system. Notice the directory configuration options, along with some helpful comments, contained within this file. This default configuration denies access to all content unless specified otherwise in subsequent directory definitions.


The Require directive can restrict or open access to different resources within your server.
The AllowOverride directive is used to decide whether an .htaccess file can override settings if it is placed in the content directory. This is not allowed by default, but can be useful to enable in a variety of circumstances.
After examining the contents of this file, you can close out of it by pressing CTRL+X.


## Alias and ScriptAlias Statements


Directory definitions are sometimes preceded by Alias or ScriptAlias directives.
Open your virtual host configuration file with this command and replace the your_domain variable with your domain name:


```
sudo nano /etc/apache2/sites-available/your_domain.conf


```


The Alias directive maps a URL path to a directory path. For example, in a virtual host that handles requests to your_domain the following would allow access to content within /usr/local/apache/content/ when navigating to your_domain.com/content/:


/etc/apache2/sites-available/your_domain.conf
```
Alias “/content/” “/usr/local/apache/content/”

```


The ScriptAlias directive operates in the same way, but is used to define directories that will have executable components in them:


/etc/apache2/sites-available/your_domain.conf
```
ScriptAlias "/cgi-bin/" "/usr/local/apache2/cgi-bin/"

```


Remember to define the directory with access privileges as discussed in the previous section. After completing your edits on the file, exit the file by pressing CTRL+X. If you made any changes to this file, press Y to save the changes to your file or press N to leave the file as it was before any changes to the configuration.


# Enabling Sites and Modules


Once you have a virtual host file that meets your requirements, you can use the tools included with Apache to transition it into live websites.
To create a symbolic link in the sites-enabled directory to an existing file in the sites-available directory, issue the following command. Make sure to replace your_domain with the name of your own virtual host site configuration file:


```
sudo a2ensite your_domain


```


After enabling a site, issue the following command to tell Apache to reload its configuration files, allowing the change to propagate:


```
sudo systemctl restart apache2


```


There is also a companion command for disabling a virtual host. It operates by removing the symbolic link from the sites-enabled directory. For example, with your virtual host site enabled, you can disable the default 000-default site:


```
sudo a2dissite 000-default


```


Modules can be enabled or disabled by using the a2enmod and a2dismod commands respectively. They work in the same way as the a2ensite and a2dissite versions of these commands. For example, to enable the info module, you can use the following command:


```
sudo a2enmod info


```


Likewise, you can disable a module using the a2dismod command:


```
sudo a2dismod info


```


Remember to restart Apache after modifying configuration files and enabling or disabling modules.


# Conclusion


Apache is versatile and very modular, so configuration needs will be different depending on your setup.
After reviewing some general use cases above, you should have a good understanding of what the main configuration files are used for and how they interact with each other. If you need to know about specific configuration options, the provided files are well commented and Apache provides excellent documentation. Hopefully, the configuration files will not be as intimidating now and you’ll feel more comfortable experimenting and modifying to suit your needs.


