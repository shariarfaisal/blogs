# How To Add ngx_pagespeed to Nginx on CentOS 7

```Nginx``` ```Server Optimization``` ```CentOS```

## Introduction


ngx_pagespeed, or just pagespeed, is an Nginx module designed to optimize your site automatically by reducing the size of its resources and hence the time the clients’ browsers need to load it. If you are not acquainted with it already, please check its official site.


This article will guide you through the installation and configuration of the pagespeed module for Nginx. It’s important to know that Nginx does not support Dynamic Loading of Modules available in other web servers such as Apache. Since Nginx doesn’t support this feature, you need to build Nginx from source to add the module.


Having your own custom package comes with one disadvantage — you are solely responsible for updating it when there is a new version. Take this into account when weighing the pros and cons of using ngx_pagespeed.


# Prerequisites


This guide has been written for CentOS 7. An Ubuntu 14.04 version and a Debian 8 version are available as well.


Before following this tutorial, please make sure you complete the following prerequisites:


- A CentOS 7 Droplet
- A non-root sudo user (Check out Initial Server Setup with CentOS 7 for details.)

Except otherwise noted, all of the commands that require root privileges in this tutorial should be run as a non-root user with sudo privileges.


# Step 1 — Download the Source and Its Dependencies


Let’s start with ensuring we have all the necessary software for the compilation and testing of Nginx. Please run the command:


```
sudo yum install wget curl unzip gcc-c++ pcre-devel zlib-devel


```


Next, create a folder in your home directory to download the source package for Nginx:


```
mkdir ~/custom-nginx


```


Change to this newly created directory:


```
cd ~/custom-nginx


```


Then, download the Nginx source package in this directory from its official site. Currently, the latest version is 1.8.0 and can be downloaded with the command:


```
sudo wget http://nginx.org/download/nginx-1.8.0.tar.gz


```


After that extract the newly downloaded package with the command:


```
sudo tar zxvf nginx-1.8.0.tar.gz


```


To confirm we are on the same page, list the content of the folder ~/custom-nginx:


```
ls ~/custom-nginx


```


The result should look like this:


```
Output of ls ~/custom-nginxnginx-1.8.0  nginx-1.8.0.tar.gz

```


As you can see, the version of the Nginx source package is 1.8.0 at the time of writing this tutorial. To start adding the ngx_pagespeed module, you first need to go to the modules folder within the extracted folder nginx-1.8.0:


```
cd nginx-1.8.0/src/http/modules/


```


In this directory, download the latest ngx_pagespeed source archive from its Github repository with the command:


```
sudo wget https://github.com/pagespeed/ngx_pagespeed/archive/master.zip


```


Once the download completes, extract it with the unzip utility like this:


```
sudo unzip master.zip


```


This will create a new directory called ngx_pagespeed-master inside your ~/custom-nginx/nginx-1.8.0/src/http/modules/ directory. For convenience rename this directory to just ngx_pagespeed with the command:


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


# Step 2 — Configure the Source and Compile It


At this point you are ready to configure the Nginx source and include the pagespeed module.  Go to the parent directory of the Nginx source:


```
cd ~/custom-nginx/nginx-1.8.0/


```


There run the configure the source with the command:


```
sudo ./configure --add-module=/home/your_user/custom-nginx/nginx-1.8.0/src/http/modules/ngx_pagespeed/ --user=nobody --group=nobody --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid


```


The most important configuration option from the above is --add-module=/home/your_user/custom-nginx/nginx-1.8.0/src/http/modules/ngx_pagespeed/. It ensures that the pagespeed module will be part of the Nginx compilation. Please make sure to replace your_user with your user in this path.


For convenience, we have customized a few other settings such as the location of the log files and the user/group under which the server should run. For more information on what can be customized please check the documentation for the compile-time options.


Once the configuration completes, start the compilation with the command:


```
sudo make


```


This will take up to ten minutes depending on your droplet’s resources. After that you can install the software with the command:


```
sudo make install


```


You can find your custom Nginx installed in the directory /usr/local/nginx. To make it even more convenient, let’s create two symbolic links. First for the configuration files:


```
sudo ln -s /usr/local/nginx/conf/ /etc/nginx


```


This command will allow you to find the configuration files in the /etc/nginx/ directory where usually configuration files reside.


You should also create a symbolic link to the main binary in the /usr/sbin/ directory so that you can find it more easily and include it in the startup script. This will be also needed for the startup script which we’ll use later. For this purpose run the command:


```
sudo ln -s /usr/local/nginx/sbin/nginx /usr/sbin/nginx


```


# Step 3 — Create the Startup Script


The previous installation process takes care of some mundane tasks such as creating the necessary nobody user and group, under which Nginx will run. However, you still have to create manually startup script. Luckily, for Nginx on CentOS 7 there is already one readily available on nginx.com.


First, create a new file in the directory /etc/init.d/:


```
sudo nano /etc/init.d/nginx


```


Then go to https://www.nginx.com/resources/wiki/start/topics/examples/redhatnginxinit/, copy the script, and paste it in this new file.


Finally, make this script executable by running the command:


```
sudo chmod +x /etc/init.d/nginx


```


After that you can start Nginx for the first time with the command:


```
sudo service nginx start


```


To make sure that Nginx starts and stops every time with the Droplet, add it to the default runlevels with the command:


```
sudo chkconfig nginx on


```


# Step 4 — Enable the Pagespeed Module


You now have Nginx installed. The next step is to enable the ngx_pagespeed module.


Before enabling the module, you have to create a folder, where it will cache the files for your website:


```
sudo mkdir -p /var/ngx_pagespeed_cache


```


Make sure to change the ownership of this folder to the Nginx user so that the web server can store files in it:


```
sudo chown -R nobody:nobody /var/ngx_pagespeed_cache


```


Then open the main Nginx configuration file nginx.conf in your favorite text editor like this:


```
sudo nano /etc/nginx/nginx.conf


```


In this file add the following lines to the server block and save the changes:


/etc/nginx/nginx.conf
```
##
# Pagespeed main settings

pagespeed on;
pagespeed FileCachePath /var/ngx_pagespeed_cache;

# Ensure requests for pagespeed optimized resources go to the pagespeed
# handler and no extraneous headers get set.

location ~ "\.pagespeed\.([a-z]\.)?[a-z]{2}\.[^.]{10}\.[^.]+" { add_header "" ""; }
location ~ "^/ngx_pagespeed_static/" { }
location ~ "^/ngx_pagespeed_beacon" { }

```


You can add these lines anywhere in the block, but in our example, we are adding it to the end of the block.


This is how the /etc/nginx/nginx.conf file should look now:


/etc/nginx/nginx.conf
```
...
http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        ##
        # Pagespeed main settings

        pagespeed on;
        pagespeed FileCachePath /var/ngx_pagespeed_cache;

        # Ensure requests for pagespeed optimized resources go to the pagespeed
        # handler and no extraneous headers get set.

        location ~ "\.pagespeed\.([a-z]\.)?[a-z]{2}\.[^.]{10}\.[^.]+" { add_header "" ""; }
        location ~ "^/ngx_pagespeed_static/" { }
        location ~ "^/ngx_pagespeed_beacon" { }

        location / {
            root   html;
            index  index.html index.htm;
        }
...

```


Also, make sure to add pagespeed configuration lines to every additional server block file you may have.


Finally, restart Nginx server for the changes to take effect:


```
sudo service nginx restart


```


# Step 5 — Test the Installation


To check if ngx_pagespeed module has been installed successfully, run the Nginx binary like this:


```
sudo /usr/sbin/nginx -V


```


If the installation was successful, you should see the ngx_pagespeed module listed among the other custom arguments:


```
Outputnginx version: nginx/1.8.0
...
configure arguments: --add-module=/home/your_user/custom-nginx/nginx-1.8.0/src/http/modules/ngx_pagespeed/
...


```


The above doesn’t mean yet that the pagespeed is enabled and works for your site. To confirm this you can use curl, a tool and a library for client-side URL transfers. With it check for the X-Page-Speed header like this:


```
curl -I -p http://localhost| grep X-Page-Speed


```


If the ngx_pagespeed module works fine, you should see it in the output along with its version:


```
OutputX-Page-Speed: 1.9.32.6-7321

```


If you don’t see this header, make sure that you have enabled pagespeed as per the instructions from the previous step.


# Conclusion


That’s how you can build Nginx with a custom module, pagespeed. These steps are valid for any other module that is not already available in Nginx. Furthermore, the whole process for installing a package from source is similar for other software packages you might need to customize. Just don’t forget that you will have to maintain and re-install these packages by yourself when there is a new version.


