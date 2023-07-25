# How To Install Laravel 4 on a CentOS 6 VPS

```PHP Frameworks``` ```PHP``` ```CentOS```










# Status: Deprecated


This article covers a version of CentOS that is no longer supported. If you are currently operating a server running CentOS 6, we highly recommend upgrading or migrating to a supported version of CentOS.


Reason:
CentOS 6 reached end of life (EOL) on November 30th, 2020 and no longer receives security patches or updates. For this reason, this guide is no longer maintained.


See Instead:
This guide might still be useful as a reference, but may not work on other CentOS releases. If available, we strongly recommend using a guide written for the version of CentOS you are using.


## Introduction


Laravel 4 is a MVC framework written in the PHP programming language. It is a fairly young framework, but has quickly risen to become one of the most popular PHP frameworks available. It also has a large vibrant community behind it. To be able to use Laravel 4 you must have a VPS running at least PHP version 5.3.7, and also have installed the MCrypt PHP extension.


Laravel 4 makes extensive use of Composer, which is a dependency manager for PHP. By utilizing Composer, Laravel 4 gives you complete freedom of how to structure your applications. You can browse the many available composer packages at packagist.org.


## Installing Repositories


Let’s install the Remi & Epel repositories.


If you are running a 64bit version of a CentOS VPS, enter the following commands in the terminal:


```
rpm -Uvh http://download.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
```


```
rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-6.rpm
```


If you are running a 32bit version of a CentOS VPS, enter these commands instead:


```
rpm -Uvh http://download.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm
```


```
rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-6.rpm
```


# Installing Apache


In this step, we will install the Apache server. Enter the following in the terminal:


```
sudo yum --enablerepo=remi install httpd
```


Once installed we can start the VPS by executing:


```
sudo service httpd start
```


To verify if Apache is installed, direct your browser to your cloud server’s IP address (eg. http://12.34.56.789). You should see the Apache test page.


If you are uncertain of your cloud server's IP Address, you can issue the following command:


```
ifconfig eth0 | grep inet | awk '{ print $2 }'
```


# Installing PHP


In this step, we will install PHP and the MCrypt extension. Enter the following in the terminal:


```
sudo yum --enablerepo=remi install php php-mysql
```


Once installed we can check the version:


```
php –v
```


PHP has a wide variety of extensions you can install. To see what is available, enter the following:


```
 yum search php-
```


As mentioned in the introduction, we need to have the MCrypt extension installed. Enter the following in the terminal:


```
sudo yum --enablerepo=remi install php-mcrypt
```


# Installing MySQL


More than likely, you will need to have a database for your site. In this step we will install the MySQL database.


Enter the following:


```
sudo yum --enablerepo=remi install mysql-server
```


After it installs, we can start the service:


```
sudo service mysqld start
```


We should set a password for the root user, otherwise we would leave a big security gap in our setup. Enter the following in the terminal:


```
sudo /usr/bin/mysql_secure_installation
```


The prompt will ask you for your current root password.  Since you have just installed MySQL, you most likely won’t have one, so leave it blank by pressing Enter.


```
Enter current password for root (enter for none): 
OK, successfully used password, moving on...
```


The prompt will ask you if you want to set a root password. Go ahead and choose Y and follow the instructions. CentOS automates the process of setting up MySQL, asking you a series of yes or no questions. It’s easiest just to say Yes to all the options. At the end, MySQL will reload and implement the new changes.


```
By default, a MySQL installation has an anonymous user, allowing anyone
to log into MySQL without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y                                            
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] y
... Success!

By default, MySQL comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MySQL
installation should now be secure.

Thanks for using MySQL!

```


# Starting Services Automatically


Now that we have Apache, PHP, and MySQL installed, we should set them to run automatically when the VPS starts. Enter the following:


```
sudo chkconfig httpd on
sudo chkconfig mysqld on
```


Now that we have a LAMP Server setup we can proceed with the Laravel Installation.


# Installing Composer


The first step is to install Composer. Run the following command from the terminal:


```
curl  -k -sS https://getcomposer.org/installer | php
```


This will download and install composer. We will want to move composer to be available within our PATH. You can view these locations by entering the following in the terminal:


```
echo $PATH
```


We will put composer in our /usr/local/bin/ directory. Enter the following in the terminal:


```
sudo mv composer.phar /usr/local/bin/composer
```


Here we moved the composer.phar file and renamed it to simply “composer”.


# Installing Laravel


Now that we have Composer installed, we will proceed with installing Laravel. Issue the following command from the terminal:


```
wget https://github.com/laravel/laravel/archive/develop.zip
```


This will download a .zip file of the Laravel framework. The next step is to unzip this file. Ensure that the unzip software is installed by issuing the following command:


```
which unzip
```


If it is installed you will see it’s path. If it is not installed, you will get an error. You can install it via the following command:


```
sudo yum install unzip
```


Now that we've confirmed to have the unzip software, we can now unzip the archive. Enter the following:


```
unzip develop
```


When we unzipped the develop.zip, it creates a directory named “laravel-develop”.  You can verify this by running the following in the terminal:


```
ls
```


Move this directory to our www directory:


```
mv laravel-develop /var/www/yoursite
```


Here we have moved the directory and renamed it to: “yoursite”. Make sure you replace “yoursite” with the name of your site.


Now that we are finished with the develop.zip, we can now delete it:


```
rm  -f develop
```


After this, we will need to install the Composer dependencies for the Laravel project. First, change into the sites directory. Make sure, once again, to replace “yoursite” with the name of the site you used in the above steps:


```
cd /var/www/yoursite
```


Then issue the composer install command to install the dependencies:


```
composer install
```


(This could take a little while. Please be patient during the installation.)


Lastly, we need to change the permissions in the “app/storage” directory. Laravel needs to be able to write into this directory. Once again, note the “yoursite” part:


```
chmod –R  775 /var/www/yoursite/app/storage
```


# Setting up a Virtual Host in Apache


The final step we need to do is to setup a Virtual Host within Apache. To do this, edit the “httpd.conf” file.  You can open the file for editing with the following command:


```
sudo nano /etc/httpd/conf/httpd.conf
```


Enter the following and save the file:


```
<VirtualHost *:80>
        ServerName yoursite.com
        DocumentRoot /var/www/yoursite/public

       <Directory /var/www/yoursite/public>
          <IfModule mod_rewrite.c>
          Options -MultiViews
          RewriteEngine On
          RewriteCond %{REQUEST_FILENAME} !-f
         RewriteRule ^ index.php [L]
       </IfModule>
</Directory>
</VirtualHost>
```


Here we setup our Virtual Host and removed index.php from our requests allowing for “cleaner” URLs. Finally, restart Apache. I am sure you don’t need reminding by now, but make sure you replace “yoursite” with the site you entered in the above steps:


```
sudo service httpd restart
```


# Conclusion


If it all went well, you should now be able to visit your site and see the Laravel logo!


