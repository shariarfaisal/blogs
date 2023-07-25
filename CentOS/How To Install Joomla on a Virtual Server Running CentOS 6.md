# How To Install Joomla on a Virtual Server Running CentOS 6

```Joomla``` ```CentOS```










# Status: Deprecated


This article covers a version of CentOS that is no longer supported. If you are currently operating a server running CentOS 6, we highly recommend upgrading or migrating to a supported version of CentOS.


Reason:
CentOS 6 reached end of life (EOL) on November 30th, 2020 and no longer receives security patches or updates. For this reason, this guide is no longer maintained.


See Instead:
This guide might still be useful as a reference, but may not work on other CentOS releases. If available, we strongly recommend using a guide written for the version of CentOS you are using.


## About Joomla


Joomla is a free and open source content management that uses a PHP and a backend database, such as MySQL. It offers a wide variety of features that make it an incredibly flexible content management system right out of the box.  It was created in 2005 and is currently the 2nd most popular content management site online. It now has over 10,000 addons to customize its functionality.


# Setup


The steps in this tutorial require the user to have root privileges on their virtual private server. You can see how to set that up in steps 3 and 4 of the Initial Server Setup


Before working with Joomla, you need to have LAMP installed on your virtual server. If you don't have the Linux, Apache, MySQL, PHP stack on your VPS, you can find the tutorial for setting it up here: How to Install LAMP on CentOS 6.


Once you have the user and required software, you can start installing Joomla!


# Step One—Download Joomla


To start, create a directory where you will keep your Joomla files temporarily:


```
mkdir temp
```


Switch into that directory:


```
cd temp
```


Then you can go ahead and download the most recent version of Joomla straight from their website. Currently, the latest version is 2.5.7.


```
wget http://joomlacode.org/gf/download/frsrelease/17410/76021/Joomla_2.5.7-Stable-Full_Package.tar.gz

```


This command will download the zipped Joomla package straight to your user's home directory on the virtual server. You can untar it with the following command, moving it straight into the default apache directory, /var/www :


```
sudo tar zxvf Joomla_2.5.7-Stable-Full_Package.tar.gz  -C /var/www/html
```


# Step Two—Configure the Settings


Once the Joomla files are in the web directory, we alter a couple of permissions to give access to the Joomla installer.


First create a Joomla configuration file and make it temporarily world-writeable:


```
sudo touch /var/www/html/configuration.php
sudo chmod 777 /var/www/html/configuration.php
```


After the installation is complete, we will change the permissions back down to 755, which will make it only writeable by the owner.


# Step Three—Create the Joomla Database and User


Now we need to switch gears for a moment and create a new MySQL directory for Joomla.


Go ahead and log into the MySQL Shell:


```
mysql -u root -p
```


Login using your MySQL root password. We then need to create the Joomla database, a user in that database, and give that user a new password. Keep in mind that all MySQL commands must end with semi-colon.


First, let's make the database (I'm calling mine Joomla for simplicity's sake—for a real server, however, this name is not very secure). Feel free to give it whatever name you choose:


```
CREATE DATABASE joomla;
Query OK, 1 row affected (0.00 sec)
```


Then we need to create the new user. You can replace the database, name, and password, with whatever you prefer:


```
CREATE USER juser@localhost;
Query OK, 0 rows affected (0.00 sec)
```


Set the password for your new user:


```
SET PASSWORD FOR juser@localhost= PASSWORD("password");
Query OK, 0 rows affected (0.00 sec)
```


Finish up by granting all privileges to the new user. Without this command, the Joomla installer will be able to harness the new mysql user to create the required tables:


```
GRANT ALL PRIVILEGES ON joomla.* TO juser@localhost IDENTIFIED BY 'password';
Query OK, 0 rows affected (0.00 sec)
```


Then refresh MySQL:


```
FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)
```


Exit out of the MySQL shell:


```
exit
```


Restart apache:


```
sudo service httpd restart
```


# Step Four—Access the Joomla Installer


Once you have placed the Joomla files in the correct location on your VPS, assigned the proper permissions, and set up the MySQL database and username, you can complete the remaining steps in your browser.


Access the Joomla installer going to your domain name or IP address. (eg. Example.com)


Once you have finished going through the installer, delete the installation folder per Joomla’s instructions and change the permissions on the config file:


```
sudo rm -rf /var/www/html/installation/
sudo chmod 755 /var/www/html/configuration.php
```


Visit your domain or IP address to see your new Joomla page.


By Etel Sverdlov
