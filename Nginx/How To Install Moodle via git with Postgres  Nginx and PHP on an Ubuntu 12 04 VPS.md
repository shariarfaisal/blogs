# How To Install Moodle via git with Postgres  Nginx and PHP on an Ubuntu 12 04 VPS

```Ubuntu``` ```Applications``` ```PostgreSQL``` ```PHP``` ```Nginx``` ```Git```


Status: Deprecated
This article covers a version of Ubuntu that is no longer supported.  If you are currently operate a server running Ubuntu 12.04, we highly recommend upgrading or migrating to a supported version of Ubuntu:

Upgrade to Ubuntu 14.04.
Upgrade from Ubuntu 14.04 to Ubuntu 16.04
Migrate the server data to a supported version

Reason:
Ubuntu 12.04 reached end of life (EOL) on April 28, 2017 and no longer receives security patches or updates.  This guide is no longer maintained.
See Instead:
This guide might still be useful as a reference, but may not work on other Ubuntu releases.  If available, we strongly recommend using a guide written for the version of Ubuntu you are using.   You can use the search functionality at the top of the page to find a more recent version.

# About this guide



Moodle is a very popular and seasoned learning platform used by many institutions. While it is easy to set up a working moodle platform, it is a lot harder to run it smoothly for many concurrent users. Therefore some of the design choices taken in this guide try to solve this issue and offer a clean and fast solution for common use cases.


# Spin a Droplet and Create a User



## Setting Up Your VPS



Create a new droplet with Ubuntu 12.04 LTS and set up a new user that will do most of the server side work.


Log into your server via ssh (this depends on your IP and whether you enabled a root certificate for ssh):


```
ssh -l root your_server_ip

```


Now create a new user and add it to the sudo group (I’ll call the user worker in this tutorial):


```
adduser worker
usermod -a -G sudo worker

```


Now log out and try to log in as the new user:


```
exit

ssh -l worker your_server_ip

```


## Update Your VPS



It’s time to update your system and make sure it runs on the latest releases:


```
sudo apt-get update
sudo apt-get upgrade

```


During the next steps, we will install the necessary packages before we configure them to our needs.


Install nginx:


```
sudo apt-get install nginx

```


Install php, along with some modules moodle likes to have available:


```
sudo apt-get install php5-fpm php-apc php5-curl php5-gd php5-xmlrpc php5-intl

```


Install Postgres as your database server (as well as its php dependencies):


```
sudo apt-get install postgresql postgresql-contrib php5-pgsql

```


Install git


Git is great for moodle as it will help you to keep your site up to date with little to no hassle with updates.


```
sudo apt-get install git

```


Now you have all packages ready to get going. In this tutorial, we will use most preconfigured settings to keep it simple and easy to reproduce.


# Configuring your VPS



Nginx has decided that wwwroot is under /usr/share/nginx/www which is fine for us. We will now install moodle there:


```
cd /usr/share/nginx

```


The directory is not ready to be accessed by anybody, so we will enable our user sole access to it; moodle won’t write into its own directory, it will use the moodledata directory which we will setup at a later stage:


```
chown -R $USER:$USER www 
cd www  

git clone https://github.com/moodle/moodle.git

```


This will clone the github moodle repository in a directory called moodle. Now you can do many things a lot easier. For example, if you moved from another hosting provider to DigitalOcean, you can just “check out” the version you last used and then check out the latest version. Or you can just stick with stable releases. There is a whole world of simple code management open for you. For this tutorial, we will simply check out the latest stable version which is “tagged” in git. To find out which version is latest for you, do this:


```
cd moodle
git tag

```


You will get a long list of tags, the latest to this date is v2.5.2. So I will simply check this tag out because we’re starting with a clean install:


```
git checkout v2.5.2

```


Now the git magic happens. Once git has finished its job, we’re in a moodle directory that replicates version 2.5.2 stable. Perfect.


If you ever need to update, you can do so in a few simple steps:


```
git fetch
git merge
git checkout v2.6.0

```


In this example, you will have loaded a git tree that offers the tag v.2.6.0 (which at this point doesn’t yet exist). Then you continue with a database update by heading to the moodle admin page.


# Adjustments



In your moodle directory there is a file called config-dist.php. Open and edit it:


```
nano config-dist.php

```


There are a few values that need to be changed in order to work on your server (comments are stripped out for simplicity’s sake):


```
<?PHP
unset($CFG);
global $CFG;
$CFG = new stdClass();
$CFG->dbtype    = 'pgsql';
$CFG->dblibrary = 'native';
$CFG->dbhost    = 'localhost';
$CFG->dbname    = 'moodle';
$CFG->dbuser    = 'moodle';
$CFG->dbpass    = 'password';
$CFG->prefix    = 'mdl_';
$CFG->dboptions = array(
    'dbpersist' => false,
    'dbsocket'  => false,
    'dbport'    => '',   
);
$CFG->wwwroot   = 'http://domain_or_ip';
$CFG->dataroot  = '/usr/local/moodledata';
$CFG->directorypermissions = 02777;
$CFG->admin = 'admin';
require_once(dirname(__FILE__) . '/lib/setup.php');

```


This is a complete config file. After you have changed the values, you can save it as config.php to your moodle directory (make sure to use your own password and your own wwwroot).


# Further Steps



The changes just performed set the pace for these next steps. You need to set up a moodle data directory and a cache directory:


```
sudo mkdir /usr/local/moodledata
sudo mkdir /var/cache/moodle

```


Make them belong to the www-data user, which is in short nginx:


```
sudo chown www-data:www-data /usr/local/moodledata    
sudo chown www-data:www-data /var/cache/moodle

```


The first one is to store user uploads, session data and other things only moodle needs access to and which shouldn’t be accessible from the web. The cache store helps to preserve files for faster caching.


Now it’s time to set up the database for moodle. To do so use the postgres user to create a new role called moodle which then will be able to handle the moodle database you are about to create:


```
sudo su - postgres
pgsql

```


This will start a new postgres console:


```
CREATE USER moodle WITH PASSWORD 'password';
CREATE DATABASE moodle;
GRANT ALL PRIVILEGES ON DATABASE moodle to moodle;
\q

```


With these commands, you’ve created a new database user called “moodle” that you granted all the necessary rights to administer the moodle database which you also created. To return to the shell use the \q command.


Exit user postgres and then, in a last step, tell nginx how to serve your files. To do so create an nginx host file:


```
exit

sudo nano /etc/nginx/sites-available/moodle

```


It should look like this:


```
server {
    listen 80;
    root /usr/share/nginx/www/moodle;
     server_name example.com;
     # put your real site address here
     rewrite ^/(.*\.php)(/)(.*)$ /$1?file=/$3 last;

     location ^~ / {
         try_files $uri $uri/ /index.php?q=$request_uri;
         index index.php index.html index.htm;

         location ~ \.php$ {
             fastcgi_split_path_info ^(.+\.php)(/.+)$;                 
             fastcgi_pass unix:/var/run/php5-fpm.sock;
             include fastcgi_params;
         }
     }
}    

```


Enable your moodle site and remove the default symlink:


```
sudo rm /etc/nginx/sites-enabled/default
sudo ln -s /etc/nginx/sites-available/moodle /etc/nginx/sites-enabled/moodle

```


# Test Your Moodle Platform


Before we can finally start our server there is one small change that needs to be fixed involving your nginx configuration:


```
sudo nano /etc/php5/fpm/pool.d/www.conf

```


In this file locate listen = 127.0.0.1:9000 and change it to:


```
listen = /var/run/php5-fpm.sock        

```


Now start up your nginx server and php:


```
sudo service nginx start
sudo service php5-fpm start

```


After this last step you can test your new moodle platform. Point your browser to your domain or your server’s IP address.


Moodle will ask you a few questions while it installs.


# Fine Tuning Your Virtual Server



Finally, you will want to fine tune a couple more things.


Cronjobs are very important for moodle and running them offsite is not as effective as running them locally. You can add a short command to your www-data user’s cron tab:


```
sudo su www-data
crontab -e

```


This will open an editor. Add the following line, which will run the cron script every ten minutes for you:


```
    */10 * * * * php -q -f /usr/share/nginx/www/moodle/admin/cli/cron.php

```


Save and return to your own shell by typing “exit”


This is the end of the tutorial. Your moodle platform should now be blazingly fast. Enjoy.


<div class=“author”>Submitted by: <a href=“http://farhang.im”>burningTyger</a></div>


