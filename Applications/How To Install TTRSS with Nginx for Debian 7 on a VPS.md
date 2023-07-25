# How To Install TTRSS with Nginx for Debian 7 on a VPS

```Applications``` ```Nginx``` ```Debian```

## Introduction



This tutorial is going to guide you through the installation process of Tiny Tiny RSS with nginx and PostgreSQL on Debian 7.0 VPS. For setting up TTRSS, you need two essential components: a web server and a database. As a web server, we are going to use nginx and as database PostgreSQL.


# Prerequisite: Update Package List



First, you should update list of available packages.


```
sudo apt-get update

```


# Step 1: Install PHP



To install PHP and all the needed modules, use the following command.


```
sudo apt-get install php5 php5-pgsql php5-fpm php-apc php5-curl php5-cli

```


# Step 2: Install and Configure PostgreSQL



Install PostgreSQL:


```
sudo apt-get install postgresql

```


Now setup the database and user for TTRSS (replace yourpasshere with some random password. Write it down somewhere, you are going to need it later.):


```
sudo -u postgres psql
postgres=# CREATE USER "www-data" WITH PASSWORD 'yourpasshere';
postgres=# CREATE DATABASE ttrss WITH OWNER "www-data";
postgres=# \quit

```


# Step 3: Install nginx



Install and start nginx:


```
sudo apt-get install nginx
sudo service nginx start

```


To verify if nginx is running, open your web browser and go to http://your.server.ip. If you see “Welcome to Nginx” message, your nginx is installed correctly.


# Step 4: Setup TTRSS



Now head to https://github.com/gothfox/Tiny-Tiny-RSS/releases and select the version you want to install (if you are not sure about which version to select, then just get the newest one). Copy the link to tar.gz to the wget command below.


```
cd /usr/share/nginx
sudo wget -O ttrss.tar.gz http://your.link.here
sudo tar -xvzf ttrss.tar.gz
sudo rm ttrss.tar.gz
sudo mv Tiny-Tiny-RSS* ttrss
sudo chown -R www-data:www-data ttrss

```


To add the nginx config file:


```
cd /etc/nginx/sites-available
sudo nano ttrss

```


Paste following lines into the editor, press Ctrl+X and then Y to save the file.
Modify line “server_name” to match your domain name or ip.


```
server {
    listen  80; ## listen for ipv4; this line is default and implied

    root /usr/share/nginx/ttrss;
    index index.html index.htm index.php;

    access_log /var/log/nginx/ttrss_access.log;
    error_log /var/log/nginx/ttrss_error.log info;

    server_name name.here;

    location / {
        index           index.php;
    }

    location ~ \.php$ {
        try_files $uri = 404; #Prevents autofixing of path which could be used for exploit
        fastcgi_pass unix:/var/run/php5-fpm.sock;
        fastcgi_index index.php;
        include /etc/nginx/fastcgi_params;
    }

}

```


To enable this config file (and disable the default welcome page):


```
cd /etc/nginx/sites-enabled
sudo rm default
sudo ln -s ../sites-available/ttrss ttrss

```


Restart nginx:


```
sudo service nginx restart

```


Head to http://your.server.ip. You should see the Tiny Tiny RSS install page.


Fill fields as follows:


Database type: Select PostgreSQL


Username: www-data


Password: The password you used during Step 2


Database Name: ttrss


Hostname: leave blank


Port: 5432


Press “Test configuration” button, then “Initialize database” and then “Save configuration”. Now your TTRSS is configured. Go to http://your.server.ip and login to default admin account (Username: “admin” Password: “password”). In the top-right go to Actions->Preferences. You can change TTRSS settings there. It is recommended to create a new user account and use it for RSS reading instead of admin account. Also, do not forget to change your admin password to a different one from default.


# Step 5: Add Automatic Feed Update to cron



For TTRSS to periodically check and update feeds, open a text editor:


```
sudo nano /etc/crontab

```


Paste the following lines to the end of the file. This tells cron to call update.php every 30 minutes.


```
*/30 * * * * www-data /usr/bin/php /usr/share/nginx/ttrss/update.php --feeds --quiet

```


# What Now?



Congratulations! Everything is now set up to use TTRSS. That said, there are still some things you can do to improve this tool.


# Install Android client


There is an Android client available on Google Play. To install it, go to Actions -> Preferences and check “Enable API access”.


## Install Chrome Client


You can get TTRSS notification icon on https://chrome.google.com/webstore/detail/tiny-tiny-rss-notifier/pehjgkflglcdbmhkjjpfjomemgaaljeb. This add-on is going to show the amount of unread messages on the right side of your Chrome omnibox.


## Change Theme


On the TTRSS forum, there is quite a lot of themes to download. To install them, just copy the theme as CSS to /usr/share/nginx/ttrss/themes and then select it in Preferences.


