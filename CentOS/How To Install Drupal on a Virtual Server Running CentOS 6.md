# How To Install Drupal on a Virtual Server Running CentOS 6

```Drupal``` ```CentOS```










# Status: Deprecated


This article covers a version of CentOS that is no longer supported. If you are currently operating a server running CentOS 6, we highly recommend upgrading or migrating to a supported version of CentOS.


Reason:
CentOS 6 reached end of life (EOL) on November 30th, 2020 and no longer receives security patches or updates. For this reason, this guide is no longer maintained.


See Instead:
This guide might still be useful as a reference, but may not work on other CentOS releases. If available, we strongly recommend using a guide written for the version of CentOS you are using.


## About Drupal


Drupal is a free and open source content management that uses a PHP and a backend database, such as MySQL. It was created in 2001 and is the 3rd most popular content management site online. It now has over 17,000 addons to customize its functionality.


# Setup


The steps in this tutorial require the user to have root privileges on their virtual private server. You can see how to set that up in steps 3 and 4 of the Initial Server Setup


Before working with Drupal, you need to have LAMP installed on your virtual server. If you don't have the Linux, Apache, MySQL, PHP stack on your VPS, you can find the tutorial for setting it up here: How to Install LAMP on CentOS 6.


Once you have the user and required software, you can start installing Drupal!


# Step One—Download Drupal


We can download Drupal straight from their website. Currently, the latest version is 7.15


```
wget  http://ftp.drupal.org/files/projects/drupal-7.15.tar.gz
```


This command will download the zipped Drupal package straight to your user's home directory on the virtual server. You can unzip it with the following command:


```
tar zxvf drupal-7.15.tar.gz 
```


Once the file is unzipped, move it your default web directory. For Apache users, this is most likely  /var/www/html.


```
sudo mv drupal-7.15 /var/www/html
```


# Step Two—Download Additional Packages


Although the LAMP stack provided a good foundation for a server, Drupal requires a few additional packages in order to run. We should download them now:


```
sudo yum install php-mbstring php-gd php-xml
```


After the packages are installed, it’s time to start setting up Drupal itself.


# Step Three—Configure the Settings


After moving the Drupal files into the web directory, switch in to the Drupal directory:


```
cd /var/www/html/drupal-7.15
```


There are a couple of steps we need to take here:


First, copy the default settings file and rename the duplicate. Do not rename the default file—you need both files for the Drupal installation.


```
 cp sites/default/default.settings.php sites/default/settings.php
```


Second, allow the installer to write to the configuration file by updating the permissions for the file and for the settings directory:


```
chmod a+w sites/default/settings.php
```


```
chmod a+w sites/default
```


# Step Three—Create the Drupal Database and User


Now we need to switch gears for a moment and create a new MySQL directory for Drupal.


Go ahead and log into the MySQL Shell:


```
mysql -u root -p
```


Login using your MySQL root password. We then need to create a  Drupal database, a user in that database, and give that user a new password. Keep in mind that all MySQL commands must end with semi-colon.


First, let's make the database (I'm calling mine Drupal for simplicity's sake—for a real server, however, this name is not very secure). Feel free to give it whatever name you choose:


```
CREATE DATABASE drupal;
Query OK, 1 row affected (0.00 sec)
```


Then we need to create the new user. You can replace the database, name, and password, with whatever you prefer:


```
CREATE USER druser@localhost;
Query OK, 0 rows affected (0.00 sec)
```


Set the password for your new user:


```
SET PASSWORD FOR druser@localhost= PASSWORD("password");
Query OK, 0 rows affected (0.00 sec)
```


Finish up by granting all privileges to the new user. Without this command, the Drupal installer will be able to harness the new mysql user to create the required tables:


```
GRANT ALL PRIVILEGES ON drupal.* TO druser@localhost IDENTIFIED BY 'password';
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


# Step Four—Access the Drupal Installer


Once you have placed the Drupal files in the correct location on your VPS, assigned the proper permissions, and set up the MySQL database and username, you can complete the remaining steps in your browser.


Access the Drupal installer by adding /drupal-7.15/ to your site's domain or IP address (eg. example.com/drupal-7.15/)


By Etel Sverdlov
