# How To Install OpenX On CentOS 6

```PHP``` ```Nginx``` ```CentOS```










# Status: Deprecated


This article covers a version of CentOS that is no longer supported. If you are currently operating a server running CentOS 6, we highly recommend upgrading or migrating to a supported version of CentOS.


Reason:
CentOS 6 reached end of life (EOL) on November 30th, 2020 and no longer receives security patches or updates. For this reason, this guide is no longer maintained.


See Instead:
This guide might still be useful as a reference, but may not work on other CentOS releases. If available, we strongly recommend using a guide written for the version of CentOS you are using.


## Introduction


OpenX is a popular advertisement server written in PHP.  It has a web interface that allows you to easily manage your ad campaigns and track statistics.


## Step 1 - Create A Domain Name


## 
Having a domain name is essential. If you would like to get a free domain, you can get one from dot.tk.
For our purposes, we will register a free domain, cloudads.tk and point it to DigitalOcean name servers:
ns1.digitalocean.com (69.55.55.74)
ns2.digitalocean.com (141.0.175.217)





## Step 2 - Spin Up A New Droplet and Configure DNS


Spin up a CentOS 6.3 x64 droplet with at least 1GB of RAM and 1 CPU Core.  As your OpenX server grows, it would be best to separate database from webserver, and scale them up separately.


If you are just starting out, a single server would be sufficient for both.


![](https://assets.digitalocean.com/articles/community/CloudAds2.png)
We should also add some SWAP memory, and for our droplet we'll add 2 GB:


```
dd if=/dev/zero of=/swap bs=1024 count=2097152
mkswap /swap && chown root. /swap && chmod 0600 /swap && swapon /swap
echo /swap swap swap defaults 0 0 >> /etc/fstab
echo vm.swappiness = 0 >> /etc/sysctl.conf && sysctl -p

```


Now head over to DigitalOcean's Control Panel and click DNS (under Labs):


![](https://assets.digitalocean.com/articles/community/CloudAds4.png)
Click "Add Domain" and select the droplet you just created:


![](https://assets.digitalocean.com/articles/community/CloudAds3.png)
## Step 3 - Install OpenX On Your Droplet


First, we will add a repository for Nginx. Create /etc/yum.repos.d/nginx.repo and add the following:


```
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=0
enabled=1

```


Now we can install the necessary packages:


```
yum -y install nginx mysql-server php php-mysql php-fpm php-gd

```


## Step 4 - Modify Nginx Config


Edit /etc/nginx/conf.d/default.conf - make sure to modify server_name for your own domain:


```
server {
    listen       80;
    server_name  cloudads.tk www.cloudads.tk;

    location / {
        root   /usr/share/nginx/html/cloudads.tk;
        index  index.html index.htm index.php;
    }

    location ~ \.php$ {
        root           html;
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  /usr/share/nginx/html/cloudads.tk$fastcgi_script_name;
        include        fastcgi_params;
    }
}

```


## Step 5 - Install OpenX


Now we can begin installing OpenX.  First, enable Short Open Tags and set correct 
date.timezone for your droplet - whether it is in New York ("America/New_York") or Amsterdam ("Europe/Amsterdam").


```
echo "short_open_tag = On" >> /etc/php.ini
echo "date.timezone=America/New_York" >> /etc/php.ini
echo "session.save_path = /tmp" >> /etc/php.ini
sed -i 's/.*php_value\[session.save_path\].*/php_value\[session.save_path\] = \/tmp/g' /etc/php-fpm.d/www.conf

```


The short open tags are just a pain to troubleshoot, so you might as well have it enabled. Here we have also set save_path to /tmp - alternatively you can use Memcached


Navigate over to your domain's folder and download packages:


```
cd /usr/share/nginx/html
mkdir cloudads.tk
wget http://download.openx.org/openx-2.8.10.tar.bz2
tar jxvf openx-2.8.10.tar.bz2
mv openx-2.8.10/* cloudads.tk/
chown -R nginx. /usr/share/nginx
sed -i 's/apache/nginx/g' /etc/php-fpm.d/www.conf
service mysqld start && service php-fpm start && service nginx start 
chkconfig mysqld on && chkconfig php-fpm on

```


Make sure to set correct folder permissions:


```
cd /usr/share/nginx/html/cloudads.tk
chmod -R a+w /usr/share/nginx/html/cloudads.tk/var
chmod -R a+w /usr/share/nginx/html/cloudads.tk/var/cache
chmod -R a+w /usr/share/nginx/html/cloudads.tk/var/plugins
chmod -R a+w /usr/share/nginx/html/cloudads.tk/var/templates_compiled
chmod -R a+w /usr/share/nginx/html/cloudads.tk/plugins
chmod -R a+w /usr/share/nginx/html/cloudads.tk/www/admin/plugins
chmod -R a+w /usr/share/nginx/html/cloudads.tk/www/images

```


## Step 6 - Create Database


We will need to create a database for OpenX to use and a user.  Make sure to replace PassWord with your own value


```
mysqladmin create openx
mysql -Bse "create user 'openx'@'localhost' identified by 'PassWord'"
mysql -Bse "grant all privileges on \`openx\`.* to 'openx'@'localhost'"
mysqladmin flush-privileges

```


## Step 7 - Proceed with Web Installation


Navigate over to your droplet's IP or if DNS has already been switched over, domain name:


![](https://assets.digitalocean.com/articles/community/CloudAds6.png)
Click "I Agree" and proceed to next step:


![](https://assets.digitalocean.com/articles/community/CloudAds7.png)
You could try registering an OpenX.org account, however it seems to have timed out when we tried to do it.


The workaround is to disable outbound SSL connections temporarily and try logging in with any username/password.


You can always register for OpenX Market later from Admin Panel -> My Account -> OpenX Market -> Get Started.


For now, we have disabled outgoing SSL connections and will try any username/password:


```
iptables -I OUTPUT 1 -p tcp --dport 443 -j REJECT

```


![](https://assets.digitalocean.com/articles/community/CloudAds7-1.png)
![](https://assets.digitalocean.com/articles/community/CloudAds8.png)
Now you can enter your database credentials with password from Step 6:



Afterwards, you will set your admin username and password, and you will be done:


![](https://assets.digitalocean.com/articles/community/CloudAds11.png)
After you have finished installing OpenX, you can drop the outgoing iptables rule:


```
iptables -D OUTPUT 1

```


## Step 9 - Disable Dashboard


```
sed -i 's/dashboardEnabled.*$/dashboardEnabled=0/' /usr/share/nginx/html/cloudads.tk/var/cloudads.tk.conf.php

```


Proceed to login to OpenX Admin panel with credentials created in Step 4 of Web Installation


![](https://assets.digitalocean.com/articles/community/CloudAds15.png)
And you are all done!


By Bulat Khamitov
