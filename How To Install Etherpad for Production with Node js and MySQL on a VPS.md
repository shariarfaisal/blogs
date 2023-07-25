# How To Install Etherpad for Production with Node js and MySQL on a VPS

```MySQL``` ```Node.js``` ```CentOS```










# Status: Deprecated


This article covers a version of CentOS that is no longer supported. If you are currently operating a server running CentOS 6, we highly recommend upgrading or migrating to a supported version of CentOS.


Reason:
CentOS 6 reached end of life (EOL) on November 30th, 2020 and no longer receives security patches or updates. For this reason, this guide is no longer maintained.


See Instead:
This guide might still be useful as a reference, but may not work on other CentOS releases. If available, we strongly recommend using a guide written for the version of CentOS you are using.


# Introduction


Etherpad is a real-time, multi-user collaboration tool mainly for program  development and web design. In this tutorial, we will focus on getting Etherpad  running on a CentOS 6.4 VPS (cloud server). This guide will consider that you already have a set up, if you do not, simply follow this guide here.


# Step 1 - Install the Required Libraries


Before we can install Etherpad, we need to install the required libraries and  prerequisites for doing so.


Go ahead and execute the following command either as root or by adding sudo to the start of every command.


```
yum install gzip git-core curl python openssl-devel make gcc gcc-c++ postgresql-devel && yum -y groupinstall "Development Tools"
```


After that is complete, you will need to install the Node.JS library and the NPM library. So perform the following commands:


```
cd /tmp
wget http://mirror-fpt-telecom.fpt.net/fedora/epel/6/i386/epel-release-6-8.noarch.rpm
rpm -ivh epel-release-6-8.noarch.rpm
yum install nodejs npm
```


Congratulations, Node.JS and NPM are installed. Now we can get onto installing Etherpad!


# Step 2 - Installing Etherpad


First, we will create a separate user for Etherpad. This will allow Etherpad to run independently from other users and is more secure than using root. This command will also create the user as well as a home directory.


```
useradd --create-home etherpad
```


Now, we will execute some commands so we can configure Etherpad as the newly created user.


```
su - etherpad
cd /home/etherpad
```


In order to get Etherpad going, we will need to download it first. Perform the following command to initiate the download from GitHub.


```
git clone git://github.com/ether/etherpad-lite.git
```


# Step 3 - Install MySQL for Etherpad Database


While Etherpad uses it's own flatfile database for storage, this is not recommended for production use. Because of this, we will be installing MySQL and configuring Etherpad to use as a database.


We will assume you do not have MySQL installed currently, so run the following commands either as root or using sudo:


```
yum install mysql-server
service mysqld start
chkconfig mysqld on
```


After it has installed, run these commands. Be sure to replace PASSWORD with a secure password of your choosing:


```
mysql -u root -p
create database `etherpad-lite`;
grant all privileges on `etherpad-lite`.* to 'etherpad'@'localhost' identified by 'PASSWORD';
exit
```


Now, we will need to go into the Etherpad directory, so execute the following:


```
su - etherpad
cd /home/etherpad/etherpad-lite	
cp settings.json.template settings.json

```


Open up the settings.json file  with your favorite editor.


Find the following text:


```
"sessionKey" : "",
```


Change it to:


```
"sessionKey" : "SECURESTRING",
```


To state the obvious, replace SECURESTRING with a 10-18 alpha-numerical string.


Then find:


```
"dbType" : "dirty",
  //the database specific settings
  "dbSettings" : {
                   "filename" : "var/dirty.db"
                 },
```


And comment it out like so:


```
// "dbType" : "dirty", */
  //the database specific settings
  // "dbSettings" : {
  //            	   "filename" : "var/dirty.db"
  //            	 },
```


Then find:


```
/* An Example of MySQL Configuration   "dbType" : "mysql",   "dbSettings" : {                    "user"    : "root",                    "host"    : "localhost",                    "password": "",                    "database": "store"                  },  */
```


Change it to the following (Taking care to ensure you remove the */ at the end):


```
// Etherpad MySQL Config   "dbType" : "mysql",   "dbSettings" : {                    "user"    : "etherpad",                    "host"    : "localhost",					 "port"    : "/var/lib/mysql/mysql.sock",                    "password": "YOURDBPASSWORD",                    "database": "etherpad-lite"                  },
```


Be sure to replace YOURDBPASSWORD with the password you set when creating the database. Save the file and close the editor afterwards.


Now we will need to let Etherpad install some dependencies for itself. So perform the following commands:


```
./bin/installDeps.sh
```


Once that has run through, we will need to run Etherpad for the first time so it can create the appropriate tables in the database. Run the following command:


```
./bin/run.sh
```


After Etherpad has loaded successfully, use Ctrl+C  to kill the process. We will need to modify the Etherpad database for use before running it for real:


```
mysql -u root -p
alter database `etherpad-lite` character set utf8 collate utf8_bin;
use `etherpad-lite`;
alter table `store` convert to character set utf8 collate utf8_bin;
exit
```


# Step 4 - Running Etherpad


We have installed Etherpad successfully and configured it to use MySQL. From this point, in order to run it properly, execute:


```
./bin/run.sh
```


This script will initialize Etherpad and then start the process.


Keep in mind that Etherpad will terminate when you cancel/close your SSH session window. You can use the optional step to place Etherpad into a screen session for easier manageability.


# Step 5 - Accessing Etherpad


Once you run the above script, you can access your Etherpad installation by browsing to: http://yourdomain.com:9001


You should be presented with an Etherpad page, asking you to create a pad or open an existing one.


# Step 6 - Running Etherpad in Screen (Optional)


Utilizing screen can save you valuable time in case your client terminates unexpectedly. It allows you to keep your session active and return to it at anytime, even when you are logged out or your SSH client quits unexpectedly.


To install the screen program, simply perform the following command as root (su) or as a super user (sudo)


```
yum install screen
```


After it has installed,  simply perform the following commands to run Etherpad within screen.


```
su - etherpad
cd /home/etherpad 
screen -dmS etherpad ./etherpad-lite/bin/run.sh
```


Etherpad should run immediately in the background.


In order to view your screen session, you will need to logout and login using your Etherpad user, but we must create a password for it first. Run the following command as root to create a password for the Etherpad user:


```
passwd etherpad
```


When this is completed, you can logout from your current SSH session and login as your Etherpad user.


Once logged in as your Etherpad user, run the following command to reattach to your screen session:


```
screen -r etherpad
```


To detach from screen and return to the bash prompt, simply press CtrlA+D at the same time. That is Control-A followed by a D.




NOTE: If the VPS loses power or is restarted, the screen session will be lost. You will need to run the commands again or utilize a startup script, such as the one found here.


# Step 7 - Additional Configuration


This guide just shows the basics in getting Etherpad setup. There are other things you can do to improve your Etherpad installation that are not covered here.


For more information on further configuring Etherpad, please visit the Etherpad Wiki at: https://github.com/ether/etherpad-lite/wiki.


Submitted by: Samuel Brereton
