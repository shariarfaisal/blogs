# How To Add ngx_pagespeed to Nginx on Ubuntu 14 04

```Ubuntu``` ```Nginx``` ```Server Optimization```

## Introduction


ngx_pagespeed, or just pagespeed, is an Nginx module designed to optimize your site automatically by reducing the size of its resources and hence the time the clients’ browsers need to load it. If you are not acquainted with it already, please check its official site.


This article will guide you through the installation and configuration of the pagespeed module for Nginx. It’s important to know that Nginx does not support Dynamic Loading of Modules available in other web servers such as Apache. Since Nginx doesn’t support this feature, you need to build Nginx from source to add the module.


Having your own custom package comes with one disadvantage — you are solely responsible for updating it when there is a new version. Take this into account when weighing the pros and cons of using ngx_pagespeed.


# Prerequisites


This guide has been written for Ubuntu 14.04. A CentOS 7 version and a Debian 8 version are available as well.


Before following this tutorial, please make sure you complete the following prerequisites:


- A Ubuntu 14.04 Droplet
- A non-root sudo user (Check out Initial Server Setup with Ubuntu 14.04 for details.)

Except otherwise noted, all of the commands that require root privileges in this tutorial should be run as a non-root user with sudo privileges.


# Step 1 — Download the Source and Its Dependencies


Before anything else, we have to make sure the list of packages available via apt-get has been updated:


```
sudo apt-get update


```


Next, we have to satisfy all the dependencies needed to run Nginx. For this purpose run the command:


```
sudo apt-get build-dep nginx


```


After that, create a folder in your home directory to download the source package for Nginx:


```
mkdir ~/custom-nginx


```


Change to this newly created directory:


```
cd ~/custom-nginx


```


Then, download the Nginx source package in this directory with the command:


```
sudo apt-get source nginx


```


To confirm we are on the same page, list the content of the folder ~/custom-nginx:


```
ls ~/custom-nginx


```


The result should look like this:


```
Output of ls ~/custom-nginxnginx-1.4.6  nginx_1.4.6-1ubuntu3.3.debian.tar.gz  nginx_1.4.6-1ubuntu3.3.dsc  nginx_1.4.6.orig.tar.gz

```


As you can see, the version of the Nginx source package is 1.4.6 at the time of writing this tutorial. To start adding the ngx_pagespeed module, you first need to go to the modules folder within the extracted folder nginx-1.4.6:


```
cd nginx-1.4.6/debian/modules


```


In this directory, download the latest ngx_pagespeed source archive from its Github repository with the command:


```
sudo wget https://github.com/pagespeed/ngx_pagespeed/archive/master.zip


```


Once the download completes, you will need the unzip utility to extract it. If you don’t already have  unzip, install it with the command:


```
sudo apt-get install unzip


```


After that extract the downloaded file with the command:


```
sudo unzip master.zip


```


This will create a new directory called ngx_pagespeed-master inside your ~/nginx-1.4.6/debian/modules directory. For convenience rename this directory to just ngx_pagespeed with the command:


```
sudo mv ngx_pagespeed-master ngx_pagespeed


```


Go inside the new ngx_pagespeed directory:


```
cd ngx_pagespeed


```


From there, download the PageSpeed Optimization Libraries (psol) which are required for the compilation:


```
sudo wget https://dl.google.com/dl/page-speed/psol/1.9.32.6.tar.gz


```


If the link to the psol archive is not working at the time you are reading this article, just skip this step. If you are missing the libraries during the compilation in the next steps, you will see an error with updated instructions for how to get the package later.


Finally, extract the psol package inside the ~/custom-nginx/nginx-1.4.6/debian/modules/ngx_pagespeed directory:


```
sudo tar -xzvf 1.9.32.6.tar.gz


```


# Step 2 — Customize the Source


At this point you are ready to customize the compilation rules and include ngx_pagespeed in the installation. For this purpose edit the file ~/custom-nginx/nginx-1.4.6/debian/rules with your favorite editor:


```
sudo nano ~/custom-nginx/nginx-1.4.6/debian/rules


```


There you have five different scenarios for building Nginx’s packages: core, full, light,  extras and naxsi. As their names suggest, common contains the common Nginx files without a server, full includes a server with the most popular modules, light creates a server with only the essential modules, extras is for a server with some extra fancy modules in it, and naxsi has in addition the naxsi module (a web application firewall).


Let’s assume that you need a light Nginx setup plus ngx_pagespeed. Thus, at the end of the  light_configure_flags configuration block add the line:


~/custom-nginx/nginx-1.4.6/debian/rules
```
--add-module=$(MODULESDIR)/ngx_pagespeed \

```


Please don’t forget to add a backslash (\) at the end of the row. The whole configuration block should look like this:


~/custom-nginx/nginx-1.4.6/debian/rules
```
config.status.light: config.env.light
        cd $(BUILDDIR_light) && ./configure  \
            $(common_configure_flags) \
            --with-http_gzip_static_module \
            --without-http_browser_module \
            --without-http_geo_module \
            --without-http_limit_req_module \
            --without-http_limit_zone_module \
            --without-http_memcached_module \
            --without-http_referer_module \
            --without-http_scgi_module \
            --without-http_split_clients_module \
            --without-http_ssi_module \
            --without-http_userid_module \
            --without-http_uwsgi_module \
            --add-module=$(MODULESDIR)/nginx-echo \
            --add-module=$(MODULESDIR)/ngx_pagespeed \
            >$@
        touch $@

```


You could add the same line to the other build scenarios too if you find a different Nginx setup more convenient.


Next, increase the source package version, since this will help you pin the package later. To achieve this, open the changelog file with a text editor:


```
sudo nano ~/custom-nginx/nginx-1.4.6/debian/changelog


```


The first line of the changelog file represents the current package version (1.4.6-1ubuntu3.3) and the Ubuntu codename (trusty). Add a custom tag such as pagespeed at the end of the version number preceded by a hyphen like this:


changelog
```
nginx (1.4.6-1ubuntu3.3-pagespeed) trusty-proposed; urgency=medium


```


# Step 3 — Build and Install Nginx with Pagespeed Module


Now that you have customized the build to include the ngx_pagespeed module, you are ready to build Nginx.


Go to the directory ~/custom-nginx/nginx-1.4.6/ with the command:


```
cd ~/custom-nginx/nginx-1.4.6/


```


From here, run the command to build the new custom Nginx binary packages:


```
sudo dpkg-buildpackage -b


```


The build process takes around 10 minutes at most. If you are worried that you might be disconnected during this time you could try using screen as described in this article.


If you have followed all the instructions, the build process should complete without any errors. To find the new custom Nginx packages go one directory up to ~/custom-nginx/ with the command:


```
cd ~/custom-nginx/


```


List the contents of the ~/custom-nginx/ directory:


```
ls ~/custom-nginx/


```


You should find a lot of .deb packages. The ones you need are called nginx-common_1.4.6-1ubuntu3.3-pagespeed_all.deb (containing the common Nginx files) and nginx-light_1.4.6-1ubuntu3.3-pagespeed_amd64.deb (containing your custom light server). The pagespeed part may vary if you have specified a different custom tag in the changelog file.


To install your custom Nginx with pagespeed module run the command:


```
sudo dpkg -i nginx-common_1.4.6-1ubuntu3.3-pagespeed_all.deb nginx-light_1.4.6-1ubuntu3.3-pagespeed_amd64.deb


```


# Step 4 — Enable the Pagespeed Module


You now have Nginx installed. The next step is to enable the ngx_pagespeed module.


Before enabling the module, you have to create a folder, where it will cache the files for your website:


```
sudo mkdir -p /var/ngx_pagespeed_cache


```


Make sure to change the ownership of this folder to the Nginx user so that the web server can store files in it:


```
sudo chown -R www-data:www-data /var/ngx_pagespeed_cache


```


Then, open the main Nginx configuration file nginx.conf in your favorite text editor like this:


```
sudo nano /etc/nginx/nginx.conf


```


In this file add the following lines to the http block and save the changes:


/etc/nginx/nginx.conf
```
##
# Pagespeed Settings
##

pagespeed on;
pagespeed FileCachePath /var/ngx_pagespeed_cache;

```


You can add these lines anywhere in the http block, but in our example, we are adding it to the end of the block.


This is how the /etc/nginx/nginx.conf file should look now:


/etc/nginx/nginx.conf
```
...
http {

        ##
        # Basic Settings
        ##

        sendfile on;
        tcp_nopush on;
        tcp_nodelay on;
        keepalive_timeout 65;
        types_hash_max_size 2048;
        # server_tokens off;

        # server_names_hash_bucket_size 64;
        # server_name_in_redirect off;

        ##
        # Pagespeed Settings
        ##
        
        pagespeed on;
        pagespeed FileCachePath /var/ngx_pagespeed_cache;
...

```


Also, you need to add pagespeed configuration lines to every server block file located in /etc/nginx/sites-available. For example, edit the /etc/nginx/sites-available/default file:


```
sudo nano /etc/nginx/sites-available/default


```


Add the following to the end of the server block:


/etc/nginx/sites-available
```
#  Ensure requests for pagespeed optimized resources go to the pagespeed
#  handler and no extraneous headers get set.
location ~ "\.pagespeed\.([a-z]\.)?[a-z]{2}\.[^.]{10}\.[^.]+" { add_header "" ""; }
location ~ "^/ngx_pagespeed_static/" { }
location ~ "^/ngx_pagespeed_beacon" { }

```


The above pagespeed configuration lines ensure that pagespeed will optimize every site’s resources.


Finally, restart Nginx server for the changes to take effect:


```
sudo service nginx restart


```


# Step 5 — Test the Installation


To check if ngx_pagespeed module has been installed successfully, run the Nginx binary like this:


```
sudo /usr/sbin/nginx -V


```


If the installation was successful, you should see the ngx_pagespeed module listed among the other modules:


```
Outputnginx version: nginx/1.4.6
...
--add-module=/home/your_user/custom-nginx/nginx-1.4.6/debian/modules/ngx_pagespeed


```


The above doesn’t mean yet that the pagespeed is enabled and works for your site. To confirm this you can use curl, a tool and a library for client-side URL transfers. If you don’t have curl already installed, then install it with the command:


```
sudo apt-get install curl


```


After that check for the X-Page-Speed header like this:


```
curl -I -p http://localhost| grep X-Page-Speed


```


If the ngx_pagespeed module works fine, you should see it in the output along with its version:


```
OutputX-Page-Speed: 1.9.32.6-7321

```


If you don’t see this header, make sure that you have enabled pagespeed as per the instructions from the previous step.


# Step 6 — Pin Your Custom Nginx Package


To prevent your custom Nginx package from being replaced in the future by apt with a more recent release of Nginx, you should pin (hold) your package from being upgraded by the following steps:


Create a new nginx file in /etc/apt/preferences.d:


```
sudo nano /etc/apt/preferences.d/nginx


```


Then paste the following lines in it and save it:


/etc/apt/preferences.d/nginx
```
Package: nginx-light
Pin: version 1.4.6-1ubuntu3.3-pagespeed
Pin-Priority: 1001

```


Please make sure to specify the Nginx package you have decided to use. In our example, it was nginx-light. Also, specify the exact version along with your custom tag like 1.4.6-1ubuntu3.3-pagespeed.


# Conclusion


That’s how you can build Nginx with a custom module, pagespeed. These steps are valid for any other module that is not already available in Nginx. Furthermore, the whole process for installing a package from source is similar for other software packages you might need to customize. Just don’t forget that you will have to maintain and re-install these packages by yourself when there is a new version.


