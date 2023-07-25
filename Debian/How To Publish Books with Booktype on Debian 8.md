# How To Publish Books with Booktype on Debian 8

```Miscellaneous``` ```Debian```

## An Article from Sourcefabric


## Introduction


Booktype is a specialized content management system for the production of books, including real, good-looking books you can hold in your hands.


You can produce Booktype output in PDF, EPUB, MOBI, XML, and HTML formats ready for book stores or the open web. Authors can import existing manuscripts in Word’s .docx format or as EPUBs, which are converted to Booktype’s native HTML chapter format for editing with Aloha.


Booktype is also a social environment in which authors can chat and share notes while producing books, seek assistance from others, or look for projects to contribute to. Booktype is a Django application written in Python and is Free Software licensed under the GNU Affero GPL, meaning that it can be freely downloaded, re-used and customized.


Booktype can be installed on any suitable GNU/Linux or Apple OS X server and in principle could run on Windows too, but this tutorial focuses on the recommended platform of Debian stable, version 8.2 (Jessie). While writing and editing books, authors can use any device with a modern web browser such as Mozilla Firefox or Google Chrome.


In this tutorial, we’ll go through an installation of Booktype which will enable you and your colleagues to produce PDF books for print and screen, EPUBs for digital devices, and XHTML for your website — all from a single source. If you want to get deeper into the possibilities of Booktype, this is a good place to start. This tutorial covers Booktype 2.0.


# Prerequisites


To following this tutorial, you will need:


- A fresh Debian 8.2 Droplet (A 512 MB/1 CPU Droplet will work, but a 1 GB/1 CPU Droplet is recommended for better performance)
- A non-root sudo user on the Droplet, as shown in Initial Server Setup with Debian 8
- Registered domain name
- Point booktype.yourdomainname.com to your Droplet (How To Set Up a Host Name with DigitalOcean explains how to set this up.)

# Step 1 — Setting Up The Dependencies


Before installing Booktype, you first need to install the development packages: the RabbitMQ server, the Redis server, the PostgreSQL database management system, the tidy syntax checker, and the Apache web server with its WSGI module:


```
sudo apt-get install git-core python-dev python-pip libjpeg-dev libpq-dev libxml2-dev libxslt-dev rabbitmq-server redis-server postgresql tidy apache2-mpm-prefork libapache2-mod-wsgi


```


If you wish Booktype to be able to send email notifications to authors, you will also need an SMTP mail server available. The simplest outgoing mail server setup is shown in the tutorial How To Install and Configure Postfix as a Send-Only SMTP Server on Ubuntu 14.04. The only difference for Debian 8.2 (instead of Ubuntu 14.04) is that in Step 1 you should enter the command:


```
sudo apt-get install postfix mailutils


```


rather than:


```
sudo apt-get install mailutils


```


Otherwise Debian’s default mail server, Exim, will be installed in place of Postfix. Exim is more complicated to configure and is not necessary for sending notifications from Booktype.


# Step 2 — Installing a PDF Renderer (Optional)


If you want to produce print books, you will need a renderer to convert Booktype’s HTML chapters into a single PDF file. The PHP application mPDF 6.0 is recommended, due to its extensive support for pre-press features. Before installing mPDF, you need to install the command line interpreter for PHP and the unzip utility with the command:


```
sudo apt-get install php5-cli unzip


```


Next, download mPDF, extract it in the directory /var/www/:


```
sudo wget http://mpdf1.com/repos/MPDF60.zip
sudo unzip MPDF60.zip -d /var/www/


```


The file is quite large, so it may take some time to download it.


Finally, change the owner of mPDF’s temporary directories to the Apache web server user www-data:


```
cd /var/www/mpdf60/
sudo chown www-data.www-data graph_cache/ tmp/ ttfontdata/


```


# Step 3 — Setting Up The Database


The next thing you need is a database to be available. Enter the following command to create the PostgreSQL user booktype-user:


```
sudo -u postgres createuser -SDRP booktype-user


```


Enter the password you wish to set in the database when prompted. You will need to re-enter it for confirmation.



Note: Write down the password in a secure place. You will need it again in Step 5 — Creating a Booktype Instance.

Then create a database named booktype-db, setting booktype-user as the owner. The encoding should be the international UTF-8 character set, as indicated with the -E option.


```
sudo -u postgres createdb -E utf8 -O booktype-user booktype-db


```



Note: The command line option to create a PostgreSQL user is the letter O (-O), not the number zero.


Note: If you use a different database name or owner, write it down. You will need it later in Step 5  — Creating a Booktype Instance when editing dev.py.

Confirm that connections to the database booktype-db are allowed by checking the PostgreSQL configuration file with the nano editor:


```
sudo nano /etc/postgresql/9.4/main/pg_hba.conf


```


Near the end of the file is a section with client authentication rules. It should look like the following:


/etc/postgresql/9.4/main/pg_hba.conf
```
# TYPE  DATABASE     USER           ADDRESS     METHOD

# "local" is for Unix domain socket connections only
local   all          all                             peer
# IPv4 local connections:
host    all          all            127.0.0.1/32     md5
# IPv6 local connections:
host    all          all            ::1/128          md5

```


The section in the example above indicates that all local connections to PostgreSQL over both IPv4 and IPv6 are allowed on this server, so we’re good to go. Quit nano with Ctrl+X.


# Step 4 — Installing Booktype With Git


While a .deb package is available from the Sourcefabric apt server, GitHub contains the most up-to-date version of Booktype available. Using Git also makes it easier to follow bug fixes between releases or contribute pull requests to the Booktype project. Download a copy of Booktype 2.0 from the git repository to the /usr/local/src/booktype/ directory:


```
sudo mkdir /usr/local/src/booktype/
sudo git clone https://github.com/sourcefabric/Booktype.git --branch 2.0 --depth 1 /usr/local/src/booktype/


```


Next, install the requirements for both development and production installs so you’ll be able to use either:


```
sudo pip install -r /usr/local/src/booktype/requirements/dev.txt
sudo pip install -r /usr/local/src/booktype/requirements/prod.txt


```


# Step 5 — Creating a Booktype Instance


A single Booktype server can host multiple instances, each with its own community of authors, groups, and books. This enables you to create separate environments for specific interests rather than throwing unrelated authors and book projects together on a general purpose platform.


Create a directory for the Booktype instances such as /var/www/booktype/:


```
sudo mkdir /var/www/booktype/


```


Make sure it is owned by the www-data user which runs the web server:


```
sudo chown www-data:www-data /var/www/booktype/


```


By default, Debian 8.2 does not allow the user www-data to enter commands. You will need to edit the line for www-data in the /etc/passwd file in order to continue:


```
sudo nano /etc/passwd


```


Replace /usr/sbin/nologin with /bin/bash for the www-data user as follows:


```
www-data:x:33:33:www-data:/var/www:/bin/bash

```


Quit nano with Ctrl+X, saving the file when prompted.


Now switch to the www-data to start creating a Booktype instance:


```
sudo su www-data


```


Create the first Booktype instance with the dev profile and a postgresql database in the /var/www/booktype/instance1 directory:


```
cd /usr/local/src/booktype/scripts/
./createbooktype -p dev --check-versions --database postgresql /var/www/booktype/instance1


```


Change to the instance directory that was just created, and edit the base.py file which contains basic settings for the instance:


```
cd /var/www/booktype/instance1/
nano instance1_site/settings/base.py


```


There are several sections of this file which need to be edited to suit your installation. First, set the name and email address of the system administrator:


base.py
```
ADMINS = (
    # ('Your Name', 'sammy@example.com'),
)

```


Set the active profile to 'dev' for development, for the time being:


base.py
```
PROFILE_ACTIVE = 'dev'

```


Enter the site name of your Booktype instance:


base.py
```
BOOKTYPE_SITE_NAME = 'Your Booktype Site'

```


Enter the email addresses to use when sending notifications and reports as well as the outgoing mail server details. If you have installed Postfix on the Droplet, you can use default values of localhost and port 25 for the email server:


base.py
```
DEFAULT_FROM_EMAIL = 'robot@example.com'
REPORT_EMAIL_USER = 'sammy@example.com'

EMAIL_HOST = 'localhost'
EMAIL_PORT = 25

```


If you chose to install mPDF, enter the location of the installation directory:


base.py
```
MPDF_DIR = '/var/www/mpdf60/'

```


Enter the name of the default publisher to use if none is specified by the author:


base.py
```
DEFAULT_PUBLISHER = "Your Publishing Company"

```


If you’ve only just installed Redis and don’t use it for anything else, you can leave the defaults for REDIS STUFF as they are. If you have more than one application using the local Redis server, you will need to change the value of REDIS_DB to a number other than zero. The default for REDIS_PASSWORD is None, but if your Redis server requires a password, it must be surrounded by single or double quotes.


base.py
```
# REDIS STUFF
REDIS_HOST = 'localhost'
REDIS_PORT = 6379
REDIS_DB = 0
REDIS_PASSWORD = None

```


Set the instance time zone and default interface language code:


base.py
```
TIME_ZONE = 'Europe/Berlin'

LANGUAGE_CODE = 'en-us'

```


Authors will be able to select their own interface language from the installed Booktype localizations such as French or Spanish.


Save and exit the file.


Next, while still in the /var/www/booktype/instance1/ directory, edit the dev.py file which contains development settings for the Booktype instance:


```
nano instance1_site/settings/dev.py


```


Enter the domain name and URL for your Booktype development server:


dev.py
```
THIS_BOOKTYPE_SERVER = 'booktype.example.com'
BOOKTYPE_URL='http://booktype.example.com'

```


Set the name, user, and password for the database connection. The username booktype-user and the PostgreSQL database name booktype-db should be the same ones you used in Step 3 — Setting Up The Database.


It should be similar to the following example:


dev.py
```
DATABASES = {'default':
                   {'ENGINE': 'django.db.backends.postgresql_psycopg2',
                    'NAME': 'booktype-db',
                    'USER': 'booktype-user',
                    'PASSWORD': 'booktype-password',
                    'HOST': 'localhost',
                    'PORT': ''
                   }
            }

```


Press Ctrl+O to save the file and Ctrl+X to quit the nano editor.



Note: When your Booktype instance is ready for deployment, you’ll be able to switch to the prod profile with a different domain name and database, while keeping your development profile available for testing.

Load the environment variables:


```
. ./booktype.env


```


Initialize the database:


```
./manage.py syncdb


```


At the end of the process, you will see the following. Answer yes to create a superuser:


```
You have installed Django's auth system, and don't have any superusers defined.
Would you like to create one now? (yes/no): yes

```


Enter the required information as prompted:


```
Username (leave blank to use 'www-data'): admin
E-mail address: `sammy@example.com`
Password:
Password (again):
Superuser created successfully.

```


Collect static files from the Booktype component applications into a single directory.


```
./manage.py collectstatic


```


The server will respond:


```
You have requested to collect static files at the destination
location as specified in your settings:

    /var/www/booktype/instance1/static

This will overwrite existing files!
Are you sure you want to do this?

Type 'yes' to continue, or 'no' to cancel:

```


After typing yes and hitting the ENTER key, enter the following commands to fetch all installed Django applications and update their permissions then update the default roles for registered and anonymous users:


```
./manage.py update_permissions
./manage.py update_default_roles


```


Installation is now complete. Return to your normal non-root sudo user prompt in the terminal with the command:


```
exit


```


You are no longer entering commands as the www-data user.


# Step 6 — Configuring Apache


Copy the wsgi.apache file generated during the creation of the instance to the Apache configuration directory for virtual hosts:


```
sudo cp /var/www/booktype/instance1/conf/wsgi.apache /etc/apache2/sites-available/booktype-instance1.conf


```


Edit the virtual host configuration file for the instance:


```
sudo nano /etc/apache2/sites-available/booktype-instance1.conf


```


You should change at least the values for ServerName and SetEnv HTTP_HOST to the domain name configured for the server and ServerAdmin to the admin email address:


/etc/apache2/sites-available/booktype-instance1.conf
```
<VirtualHost *:80>

     # Change the following three lines for your server
     ServerName booktype.example.com
     SetEnv HTTP_HOST "booktype.example.com"
     ServerAdmin sammy@example.com

```


Because Debian 8.2 features Apache 2.4, you will need to uncomment Require all granted for all Location and Directory stanzas. To uncomment the Require all granted lines, remove the # character at the beginning of each of the lines:


/etc/apache2/sites-available/booktype-instance1.conf
```
     <Location "/">
       #Require all granted
       Options FollowSymLinks
     </Location>

     Alias /static/ "/var/www/booktype/instance1/static/"
     <Directory "/var/www/booktype/instance1/static/">
       #Require all granted
       Options -Indexes
     </Directory>

     Alias /data/ "/var/www/booktype/instance1/data/"
     <Directory "/var/www/booktype/instance1/data/">
       #Require all granted
       Options -Indexes
     </Directory>

```


Press Ctrl+O to save the file and Ctrl+X to quit the nano editor.


Then disable the default Apache configuration and enable the Booktype virtual host for the instance with the commands:


```
sudo a2dissite 000-default.conf
sudo a2ensite booktype-instance1.conf


```


Restart the Apache webserver to enable the changes with the command:


```
sudo service apache2 restart


```


You should now be able to browse your Booktype instance, at the URL of the ServerName defined in the VirtualHost configuration such as booktype.example.com. Click the top of the Django debug toolbar to hide it (this toolbar will not be present when using the prod profile).






Note: You can select an interface language from the drop-down menu in the top right corner of the browser window.

Sign in to Booktype using the superuser account details that you created earlier (admin in our example).





After signing in, the gravatar associated with the superuser email address, if you have one, will be displayed in the People and My Profile boxes.





# Step 7 — Running Celery With Supervisor


Celery is the task queue used by Booktype servers. Once you have installed Booktype, you are likely to need a process monitor to keep the Celery workers running in case of any crashes or reboots. You can install supervisord with the command:


```
sudo apt-get install supervisor


```


The supervisord program automatically starts up after installation and is configured to start automatically on the next reboot of the server.


Now, we have to create a configuration file to use with Booktype and Celery with the command:


```
sudo nano /etc/supervisor/conf.d/booktype-instance1.conf


```


For the first Booktype instance in /var/www/booktype/instance1 with ten workers, the contents of the file booktype-instance1.conf should be similar to:


/etc/supervisor/conf.d/booktype-instance1.conf
```
[program:celeryd]
command=/var/www/booktype/instance1/manage.py celery worker --concurrency=10 -l info
autostart=true
autorestart=true
startretries=3
stderr_logfile=/var/www/booktype/instance1/logs/booktype-celery.error.log
stdout_logfile=/var/www/booktype/instance1/logs/booktype-celery.output.log
user=www-data

```


After saving the booktype-instance1.conf file with Ctrl+O and exiting nano with Ctrl+X, enable the updates to the supervisord configuration with the commands:


```
sudo supervisorctl reread
sudo supervisorctl update


```


The supervisorctl program can also be used to check that supervisord is running celeryd:


```
sudo supervisorctl


```


The output of this command should be similar to:


```
Output of sudo supervisorctlceleryd                          RUNNING    pid 24182, uptime 0:13:19

```


You should also see the following prompt:


```
supervisor>

```


Type the following command to exit from supervisorctl:


```
quit


```


# Conclusion


Now you and your team have everything you need to start writing and publishing books together! Please read the official Booktype manual for usage details.


