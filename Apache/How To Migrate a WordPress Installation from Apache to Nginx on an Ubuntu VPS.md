# How To Migrate a WordPress Installation from Apache to Nginx on an Ubuntu VPS

```Apache``` ```Ubuntu``` ```WordPress``` ```Nginx``` ```PHP```


Status: Deprecated
This article covers a version of Ubuntu that is no longer supported.  If you are currently operate a server running Ubuntu 12.04, we highly recommend upgrading or migrating to a supported version of Ubuntu:

Upgrade to Ubuntu 14.04.
Upgrade from Ubuntu 14.04 to Ubuntu 16.04
Migrate the server data to a supported version

Reason:
Ubuntu 12.04 reached end of life (EOL) on April 28, 2017 and no longer receives security patches or updates.  This guide is no longer maintained.
See Instead:
This guide might still be useful as a reference, but may not work on other Ubuntu releases.  If available, we strongly recommend using a guide written for the version of Ubuntu you are using.   You can use the search functionality at the top of the page to find a more recent version.

## Introduction



WordPress is a popular platform for easily starting a website or blog.  It is very flexible and can be quickly configured and modified, allowing you to focus on your content instead of the majority of the configuration.


While WordPress can be configured to operate on top of most web servers, a popular, perhaps default, choice for many people has been the Apache web server.  Apache is robust and well supported, but it sometimes runs into trouble with resource utilization when serving a large amount of guests.


Nginx, another web server, has been gaining popularity at a rapid pace due to its ease of use, flexibility, and low resource usage.  Many people are beginning to choose Nginx over Apache to host their WordPress installations for these reasons.  While there are many resources on how to install WordPress with Nginx, migrating an existing installation can sometimes seem like a daunting task.


In this guide, we will discuss how to migrate your existing WordPress installation from Apache to Nginx on an Ubuntu 12.04 server.  We assume that you have installed WordPress with Apache on Ubuntu 12.04 following this guide.  If you have installed it in a different manner, you may have to adjust some lines in your configuration.


# Install Nginx and PHP5-FPM



The first thing we need to do is install the Nginx server.  This is available in Ubuntu’s default repositories, so we can use apt-get:


```
sudo apt-get update
sudo apt-get install nginx

```


Unlike Apache, Nginx made a design choice to not include core PHP handling functionality.  Instead, Nginx decided that by passing these requests to a dedicated PHP handler, it could maintain its focus while still addressing the issue.


The standard choice for executing PHP files with Nginx is a utility called php5-fpm.  We need to install this package so that our WordPress installation can process the PHP files that it relies on:


```
sudo apt-get install php5-fpm

```


All of the necessary components are now installed.  Now, we can begin the real work of configuring our components to take over for Apache.


# Configure PHP5-FPM



We will start by configuring the PHP handler, since this is relatively straight forward and easy.


First, we will need to modify a value in our php5-fpm configuration file.  Open it now:


```
sudo nano /etc/php5/fpm/php.ini

```


Find the cgi.fix_pathinfo parameter and modify it to look like this:


```
cgi.fix_pathinfo=0

```


This is a security measure.  If this parameter were to be set to “1”, php5-fpm would attempt to serve PHP files by looking for the requested file, and then it would try to guess and return a close match if no exact match was found.


This is very insecure, because it can allow your site to expose sensitive information to outside users.  It is much safer for php5-fpm to return the file if it exactly matches the requested file, and otherwise return an error to the user.  This is what our modification accomplishes.


Save and close this file when you are finished.


Next, we need to modify another PHP configuration file so that our processor can connect with our web server correctly.


Open this file with your text editor:


```
sudo nano /etc/php5/fpm/pool.d/www.conf

```


Find the location where the listen = directive is defined.  We will modify this to use a socket for communication with the web server.  Modify it so that it reads:


```
    listen = /var/run/php5-fpm.sock

```


When you are finished, save and close the file.


To enable our changes, we need to restart the php5-fpm service:


```
sudo service php5-fpm restart

```


# Create an Nginx Server Block Configuration



Now, we are ready to configure our Nginx server blocks.  Nginx server blocks are analogous to Apache virtual hosts in that they describe configurations for a specific site.


Like Apache, Nginx on Ubuntu configures sites through a sites-available directory and then links those sites to sites-enabled to serve them.  We will modify the default configuration file in order to serve our WordPress installation.


Open that file now:


```
sudo nano /etc/nginx/sites-available/default

```


Inside the configuration file, delete the information that is already there.  We are going to start from scratch so that we can discuss what is going on throughout the file we are building.


First, we need to create a server block.  This is the basic unit of organization for a single website.  These files are incorporated into the Nginx configuration when the server starts or reloads.  They inherit the default values, and may modify them.


```
server {

}

```


First of all, it is important to know that each Nginx directive must end with a semi-colon (;).  Failure to include this will lead to Nginx misinterpreting the file, which will probably cause the server to fail to start.


Inside the server block, we specify what port this site should listen on to get connections from users.  By default, HTTP traffic to web browsers is served on port 80, so that is where we will tell Nginx to look for connections.


We also want to specify the document root, where our WordPress files are stored by default.


If you followed the guide above, the WordPress files should be located in the /var/www directory.  If you installed in another manner, your WordPress files might be stored in /var/www/wordpress, /home/wordpress/public_html, or other locations.  Adjust the value of the root directive to match your configuration.


<pre>
server {
listen 80;


```
root <span class="highlight">/var/www</span>;

```


}
</pre>


Next, we will add a directive that tells Nginx which files to try to serve when a directory is requested.  The file that is served to describe a directory is called a directory index.  With WordPress, this file will be called index.php.  Other index files written in HTML can be used as a fallback.


We also need to specify the domain name that our server will respond to.  While this might not be important if WordPress is the only site we have configured, including this directive will allow us to add other sites without modifying this file again.  We can include alternative names to match (like the site preceded by “www”) by naming those after the initial server_name:


<pre>
server {
listen 80;


```
root <span class="highlight">/var/www</span>;
index index.php index.html index.htm;

server_name <span class="highlight">your_domain.com</span> www.<span class="highlight">your_domain.com</span>;

```


}
</pre>


The remainder of the information that we put in our server block will be further compartmentalized into location blocks.  Location blocks allow us to specify specific rules for what to do when a request matches a certain pattern.


First, we will make a location block that matches the root partition.  This will apply to files that are located in the folder you defined with the root directive.  This block will contain general rules on how to try to serve files.


The rule we will add will tell Nginx to attempt to find a file that matches the exact resource requested after the domain name.  If it is not successful with that, it should attempt to find a directory of the same name.  If this also is unsuccessful, it will pass the resource requested to a PHP script, which will try to find a good way to handle the request.


After this location block, we will define a block to actually handle the passing of PHP scripts to our php5-fpm tool.  This uses the information we defined in the php5-fpm configuration files to connect.


<pre>
server {
listen 80;


```
root <span class="highlight">/var/www</span>;
index index.php index.html index.htm;

server_name <span class="highlight">your_domain.com</span> www.<span class="highlight">your_domain.com</span>;

location / {
    try_files $uri $uri/ /index.php?q=$uri&$args;
}

location ~ \.php$ {
    try_files $uri =404;
    fastcgi_pass unix:/var/run/php5-fpm.sock;
    fastcgi_index index.php;
    include fastcgi_params;
}

```


}
</pre>


At this point, we just want to tighten some things up.  We will create some rules for how Nginx attempts to serve favicon requests and robot.txt requests (which are used by search engines to index sites).  We don’t need to log requests for this information.


After that, we have a location block that denies access to any hidden folders (denoted by a starting dot in a Linux system).  This will prevent us from serving files that may have been used for Apache configurations, like .htaccess files.  It is more secure to keep these files from our users.


Finally, we prevent any PHP files from being run or accessed from within the uploads or files directory.  This can prevent our server from executing malicious code.


Our completed file should look like this:


<pre>
server {
listen 80;


```
root <span class="highlight">/var/www</span>;
index index.php index.html index.htm;

server_name <span class="highlight">your_domain.com</span> www.<span class="highlight">your_domain.com</span>;

location / {
    try_files $uri $uri/ /index.php?q=$uri&$args;
}

location ~ \.php$ {
    try_files $uri =404;
    fastcgi_pass unix:/var/run/php5-fpm.sock;
    fastcgi_index index.php;
    include fastcgi_params;
}

location = /favicon.ico {
    log_not_found off;
    access_log off;
}

location = /robots.txt {
    allow all;
    log_not_found off;
    access_log off;
}

location ~ /\. {
    deny all;
}

location ~* /(?:uploads|files)/.*\.php$ {
    deny all;
}

```


}
</pre>


When you are finished, save and close the file.  This file should already be linked to the sites-enabled directory, so it should be active when we start Nginx.


# Make the Switch



Currently, Apache is serving our files on port 80.  That means that Nginx cannot bind and listen to that port.  We will need to stop Apache before we can enable Nginx to take over.


In some cases, you may wish to keep Apache around if it is involved in serving other content on your server.  You may have multiple sites and are only transitioning a single site to Nginx.  If this is the case, you need to disable all of the references to port 80 in your Apache configuration.


Some files that you will need to check are:


```
/etc/apache2/ports.conf
/etc/apache2/apache2.conf
/etc/apache2/httpd.conf
/etc/apache2/sites-enabled/    ## Search all sites in this directory

```


If you are going to continue serving sites with Apache and end up having to change the Apache port from 80 to something else, you will have to proxy these through Nginx in order for them to be accessible from the internet.  This is outside of the scope of this tutorial, but you can find relevant information in this article.


If you still need Apache running to serve some content, after you switch the port in the configuration files, you can simply reload Apache when you turn on Nginx.  This will switch your port 80 content to Nginx, while allowing Apache to continue to function for other content:


```
sudo service apache2 reload && sudo service nginx start

```


If, however, you have no further need for Apache, and you have transitioned all of your content over to Nginx, you can turn Apache off before loading Nginx:


```
sudo service apache2 stop && sudo service nginx start

```


It is important that Apache either relinquish control over port 80, or be stopped entirely prior to starting Nginx.  Failure to do this can result in your Nginx initialization not completing because the port it is told to use is not available.


If your transition is successful and your content is all being served with Nginx, you can uninstall Apache and all of the dependencies that are not being used.


You can find these by typing:


```
dpkg --get-selections | grep apache

```



```
apache2                         install
apache2-mpm-prefork             install
apache2-utils                   install
apache2.2-bin                   install
apache2.2-common                install
libapache2-mod-auth-mysql       install
libapache2-mod-php5             install

```


You can then uninstall these with typing something like:


```
sudo apt-get remove apache2 apache2-mpm-prefork apache2-utils apache2.2-bin apache2.2-common libapache2-mod-auth-mysql libapache2-mod-php5

```


You can also take care of dependencies that are no longer needed by typing:


```
sudo apt-get autoremove

```


# Conclusion



Your WordPress should now be up and running again, this time backed by Nginx.  While both Apache and Nginx operate in similar ways, especially when backing a full-featured front-end like WordPress, Nginx can often do the same work with fewer resources.


While this may not lead to a faster site necessarily, it becomes very important when your site receives a high amount of traffic.  Using less resources means that your web server can continue to function well while under a heavier load.  This can be important if your content gets reposted and begins to see an influx of traffic.  Nginx will help you deal with that traffic without allocating additional resources.


<div class=“author”>By Justin Ellingwood</div>


