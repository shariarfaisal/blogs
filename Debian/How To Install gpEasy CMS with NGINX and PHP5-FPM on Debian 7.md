# How To Install gpEasy CMS with NGINX and PHP5-FPM on Debian 7

```Nginx``` ```PHP``` ```Debian```

## Introduction


This tutorial will take you through the steps needed to host gpEasy CMS on your Droplet.


gpEasy is a simple, powerful and lightweight CMS. It doesn’t require you to setup any databases, as it’s flat-file based, and allows you to edit your website on the fly with a true ‘What You See Is What You Get’ editor. It is also very easy to theme and customize!


Nginx is a lightweight but very powerful web server. It’s known to be ultimately stable and easy on server resources. PHP5-FPM stands for PHP5 FastCGI Process Manager. We will be using it together with nginx to serve php documents to visitors.


For the purpose of this tutorial, we’ll assume that both unzip and nano are installed on your VPS.


# Update packages list and upgrade server:


Login as root to server and execute:


```
apt-get update

```


Once the lists are updated we can upgrade the server by executing:


```
apt-get upgrade

```


# Install nginx and php5-fpm


Execute:


```
apt-get install nginx php5-fpm

```


# Create user for gpEasy installation


We will create a new user who will hold GPEasy installation in his home directory.


For the purpose of this tutorial we will call him gpeasy


Execute:


```
adduser gpeasy

```


Go through the steps of user creation:


```
Adding user `gpeasy' ...
Adding new group `gpeasy' (1000) ...
Adding new user `gpeasy' (1000) with group `gpeasy' ...
Creating home directory `/home/gpeasy' ...
Copying files from `/etc/skel' ...
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
Changing the user information for gpeasy
Enter the new value, or press ENTER for the default
    Full Name []:
    Room Number []:
    Work Phone []:
    Home Phone []:
    Other []:
Is the information correct? [Y/n] Y

```


# Add ‘gpeasy’ user to ‘www-data’ group


To avoid permission errors while using gpeasy, we will add our gpeasy user to www-data group.


Execute:


```
usermod -a -G www-data gpeasy

```


This command won’t output anything. If we don’t see any errors then most probably everything went well; but we can still perform a check just to be completely sure:


```
groups gpeasy | grep www-data

```


If output looks similar to this then everything went good:


```
gpeasy : gpeasy www-data

```


# Login as ‘gpeasy’ and download gpEasy CMS


To login as ‘gpeasy’ execeute:


```
login gpeasy

```


Once we are logged in, we will end up in the gpeasy home directory.


Now it’s time to download the gpEasy CMS:


```
wget -c http://gpeasy.com/Special_gpEasy?cmd=dlzip -O gpeasy.zip

```


```
[...]
HTTP request sent, awaiting response... 200 OK
Length: 2782667 (2.7M) [application/octet-stream]
Saving to: `gpeasy.zip'

100%[======================================>] 2,782,667    682K/s   in 4.9s   

2014-05-18 16:31:50 (560 KB/s) - `gpeasy.zip' saved [2782667/2782667]

```


Now that we have the zip file with gpEasy CMS inside, we will have to unpack it. Execute:


```
unzip gpeasy.zip

```


```
[...]
  inflating: gpEasy/addons/Multi Site/Addon.ini 
  inflating: gpEasy/addons/Multi Site/Install.php 
  inflating: gpEasy/addons/Multi Site/multi_site.css 

```


What we are going to do now is rename the gpEasy directory to www to avoid confusion with our home directory. Execute:


```
mv gpEasy/ www/

```


(Optional) We will setup gpEasy to not show index.php in the address bar so it looks nicer. Execute:


```
nano www/gpconfig.php

```


Add a line right below <?php containing:


```
define('gp_indexphp',false);

```


The end result should look similar to this:


```
<?php
define('gp_indexphp',false);

[...]

```


Press Ctrl+O then Enter/Return to save. Close editor by pressing Ctrl+X.


We have to give the right permissions to the data directory of gpEasy. This is needed to avoid read/write errors when using gpEasy. We will set the data folder to allow read/write/execute for the owner and the group, but disallow writing for public.


Execute:


```
chmod 775 /home/gpeasy/www/data

```


We also have to change group of the data directory of gpEasy to www-data:


```
chgrp www-data /home/gpeasy/www/data

```


We will also disable executing of the following files for everyone including owner and the group:


```
chmod 664 www/data/example_htaccess

```


```
chmod 664 www/data/index.html

```


Log out from gpeasy user by executing:


```
logout

```


# Configure nginx


First, we are going to remove the default nginx site configuration. Execute:


```
rm /etc/nginx/sites-enabled/default

```


(Optional) As we might want to point some domains to the server, it might turn out that we will need to increase hash bucket size in nginx configuration. To do so execute:


```
nano /etc/nginx/nginx.conf

```


Next, press Ctrl+W and search for line:


```
# server_names_hash_bucket_size 64;

```


Remove the # from front of this line so it looks like this:


```
server_names_hash_bucket_size 64;

```


Now press Ctrl+O then Enter/Return to save the file and Ctrl+X to close editor.


Here we will have to create the site configuration for our gpEasy installation. Execute:


```
nano /etc/nginx/sites-available/gpeasy

```


Now paste following configuration into editor:


```
# nginx/php5-fpm/gpeasy
server
{
    listen 80; # Listen ports
    #server_name yourdomain.com www.yourdomain.com; # Domain name pointed to server
    #gpeasy
    
    root /home/gpeasy/www/; # Location of gpeasy installation root
    index index.html index.htm index.php; # Default index files to try
    try_files $uri $uri/ /index.php?$args; # Rewrite rules for gpeasy (pass /request as argument to cms)

   
    #php5-fpm
    location ~ \.php$
    {
        fastcgi_pass unix:/var/run/php5-fpm.sock;
        fastcgi_index index.php;
        include fastcgi_params;
    }
   
    location ~ /\.ht
    {
            deny all;
    }
}

```


If you are going to point the domain to this website, you might want to replace yourdomain.com in the config with your actual domain name and remove the front # from this line:


```
#server_name yourdomain.com www.yourdomain.com; # Domain name pointed to server

```


Save the file by pressing Ctrl+O then Enter/Return. Close editor by pressing Ctrl+X.


# (Optional) Enable image functions for gpEasy


It’s very easy. All we have to do is install php5-gd. Execute:


```
apt-get install php5-gd

```


# 8. Enable website


We still have to enable our site configuration. Execute:


```
ln -s /etc/nginx/sites-available/gpeasy /etc/nginx/sites-enabled/gpeasy

```


Now we’ll restart php5-fpm and nginx by executing:


```
/etc/init.d/php5-fpm restart

```


```
/etc/init.d/nginx restart

```


# 9. Last steps


We have to open our favourite web browser and enter the server IP address or pointed domain name into the address bar.


gpEasy installation form should appear in the browser. We’ll complete it according to own needs and click install. Once it’s done, the installer will tell us that for security reasons we should delete /include/install/install.php. Execute following command to do it:


```
rm /home/gpeasy/www/include/install/install.php

```


At this point we can logout from our VPS:


```
logout

```


<div class=“author”>Submitted by: <a href=“http://lythve.com”>Chris L.</a></div>


