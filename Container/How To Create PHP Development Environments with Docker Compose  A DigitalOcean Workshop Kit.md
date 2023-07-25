# How To Create PHP Development Environments with Docker Compose  A DigitalOcean Workshop Kit

```Docker``` ```PHP``` ```Laravel``` ```Container``` ```Workshop Kits```


How to Create PHP Development Environments with Docker Compose Workshop Kit Materials
This workshop kit is designed to help a technical audience become familiar with Docker Compose and learn to set up a working development environment for a Laravel application using containers.
The aim is to provide a complete set of resources for a speaker to host an event and deliver an introductory talk on Docker Compose in the context of PHP development environments. It includes:

Slides and speaker notes including commands for running an optional live demo. This talk runs for roughly 40 minutes.
A GitHub repository containing the demo app code and the additional files necessary to get a PHP development environment up and running with Docker and Docker Compose.
This tutorial, which walks a user through getting the Travellist demo Laravel application running on containers with Docker Compose.

This guide is intended to supplement the talk demo with additional detail and elucidation.

## Introduction


This tutorial, designed to accompany the slides and speaker notes for the How To Create PHP Development Environments with Docker Compose Slide Deck, will show you how to get a demo Laravel application up and running with Docker and Docker Compose, using the setup that we discuss in greater detail in our guide on How To Set Up Laravel with Docker Compose on Ubuntu 20.04.



Note: This material is intended to demonstrate how to use Docker Compose to create PHP development environments. Although our demo consists of a Laravel application running on a LEMP server, readers are encouraged to modify and adapt the included setup to suit their own needs.

# Prerequisites


To follow this tutorial, you will need:


- Access to an Ubuntu 20.04 local machine or development server with at least 1GB of RAM, as a non-root user with sudo privileges. If you’re using a remote server, it’s advisable to have an active firewall installed. To set these up, please refer to our Initial Server Setup Guide for Ubuntu 20.04.
- Docker installed on your local machine or development server, following Steps 1 and 2 of How To Install and Use Docker on Ubuntu 20.04.
- Docker Compose installed on your local machine or development server, following Step 1 of How To Install and Use Docker Compose on Ubuntu 20.04.

# Step 1 — Download the Demo Application


To get started, download release tutorial-4.0.3 of the Travellist Laravel Demo application, which contains the application files and the Docker Compose setup that are used in this workshop kit.


```
curl -L https://github.com/do-community/travellist-laravel-demo/archive/tutorial-4.0.3.zip --output travellist.zip


```


Next, install the unzip utility in case that is not yet installed on your local machine or development server:


```
sudo apt install unzip


```


Unzip the package and move into the newly created directory:


```
unzip travellist.zip
cd travellist-laravel-demo-tutorial-4.0.3


```


Now, you can run an ls command to inspect the contents of the cloned repository:


```
ls -l --group-directories-first


```


You’ll receive output like this:


ansible-laravel-demo
```
total 256
drwxrwxr-x 6 sammy sammy   4096 mei 14 16:16 app
drwxrwxr-x 3 sammy sammy   4096 mei 14 16:16 bootstrap
drwxrwxr-x 2 sammy sammy   4096 mei 14 16:16 config
drwxrwxr-x 5 sammy sammy   4096 mei 14 16:16 database
drwxrwxr-x 4 sammy sammy   4096 mei 14 16:16 docker-compose
drwxrwxr-x 5 sammy sammy   4096 mei 14 16:16 public
drwxrwxr-x 6 sammy sammy   4096 mei 14 16:16 resources
drwxrwxr-x 2 sammy sammy   4096 mei 14 16:16 routes
drwxrwxr-x 5 sammy sammy   4096 mei 14 16:16 storage
drwxrwxr-x 4 sammy sammy   4096 mei 14 16:16 tests
-rwxr-xr-x 1 sammy sammy   1686 mei 14 16:16 artisan
-rw-rw-r-- 1 sammy sammy   1501 mei 14 16:16 composer.json
-rw-rw-r-- 1 sammy sammy 181665 mei 14 16:16 composer.lock
-rw-rw-r-- 1 sammy sammy   1016 mei 14 16:16 docker-compose.yml
-rw-rw-r-- 1 sammy sammy    737 mei 14 16:16 Dockerfile
-rw-rw-r-- 1 sammy sammy   1013 mei 14 16:16 package.json
-rw-rw-r-- 1 sammy sammy   1405 mei 14 16:16 phpunit.xml
-rw-rw-r-- 1 sammy sammy    814 mei 14 16:16 readme.md
-rw-rw-r-- 1 sammy sammy    563 mei 14 16:16 server.php
-rw-rw-r-- 1 sammy sammy    538 mei 14 16:16 webpack.mix.js

```


Here are the relevant directories and files for the Docker Compose setup we’re using:


- docker-compose/ — contains files used to set up the containerized environment, such as the Nginx configuration file and the application’s MySQL dump.
- docker-compose.yml — here, we define all services we’ll need: app, web, and db. Shared volumes and networks are also set up here.
- Dockerfile — this defines a custom application image based on php-fpm. While the web and db services are based on default images, the app service requires additional setup steps, that’s why we are creating a custom image for this service container.

All remaining files are part of the application.


# Step 2 — Set Up the Application’s .env File


You’ll now create a new .env file using the included .env.example file as base. Because Laravel uses a dot env file that is also supported by Docker Compose, the values set here will be available at build time when you bring your environment up, and will be used to set up the database service container.


```
cp .env.example .env


```


For reference, this is what the included .env file looks like. Because these settings are being applied to an isolated development environment, there is no need to change database credentials in this file, but you are free to do so if you would like.


.env
```
APP_NAME=Travellist
APP_ENV=dev
APP_KEY=
APP_DEBUG=true
APP_URL=http://localhost:8000

LOG_CHANNEL=stack

DB_CONNECTION=mysql
DB_HOST=db
DB_PORT=3306
DB_DATABASE=travellist
DB_USERNAME=travellist_user
DB_PASSWORD=password

BROADCAST_DRIVER=log
CACHE_DRIVER=file
QUEUE_CONNECTION=sync
SESSION_DRIVER=cookie
SESSION_LIFETIME=120

REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379

MAIL_DRIVER=smtp
MAIL_HOST=smtp.mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null

AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=

PUSHER_APP_ID=
PUSHER_APP_KEY=
PUSHER_APP_SECRET=
PUSHER_APP_CLUSTER=mt1

MIX_PUSHER_APP_KEY="${PUSHER_APP_KEY}"
MIX_PUSHER_APP_CLUSTER="${PUSHER_APP_CLUSTER}"

```


Once you are satisfied with your .env file, you should move onto running Docker Compose, as outlined in the next session.


# Step 3 — Run Docker Compose


Once you have your .env file in place, you can bring your environment up with:


```
docker-compose up -d


```


This will execute Docker Compose in detached mode, which means it will run in the background. This command may take a few moments to run when you execute it for the first time, since it will download and build the app service image.


```
OutputCreating network "travellist-laravel-demo-tutorial-403_travellist" with driver "bridge"
Creating travellist-db    ... done
Creating travellist-app   ... done
Creating travellist-nginx ... done

```


To verify the status of your services, you can run:


```
docker-compose ps


```


You’ll receive output like this:


```
Output      Name                    Command               State          Ports        
--------------------------------------------------------------------------------
travellist-app     docker-php-entrypoint php-fpm    Up      9000/tcp            
travellist-db      docker-entrypoint.sh mysqld      Up      3306/tcp, 33060/tcp 
travellist-nginx   /docker-entrypoint.sh ngin ...   Up      0.0.0.0:8000->80/tcp

```


Your containerized PHP development environment is up and running, but there are still a couple steps required so that you can access the application from your browser. We’ll set everything up in the next and final step.


# Step 4 — Finish Setting Up the Application


Now that you have a development environment able to handle PHP scripts, you can install the application dependencies using composer. To execute commands inside containers, you can use the docker-compose exec command, followed by the name of the service container and the command you want to execute.


The following command will run composer install on the app service container, where PHP is installed:


```
docker-compose exec app composer install


```


After the dependencies are installed, you’ll need to generate a unique application key using the artisan Laravel command line tool:


```
docker-compose exec app php artisan key:generate


```


```
OutputApplication key set successfully.

```


You can now access the demo application by pointing your browser to localhost if you are running this setup on a local machine, or your remote server’s domain name or IP address if you are running this on a development server. The server expects connections on port 8000, so be sure to include :8000 in the address:


```
http://localhost:8000

```


You will see a page like this:





With this page displaying on your browser you have successfully set up your application.


# Docker Compose Quick Reference


In this section, you’ll find a brief reference of the main Docker Compose commands used to manage a containerized environment. These should be executed from the same directory where you have set up your docker-compose.yml file.


## build


Builds any custom images associated with the current docker-compose.yml file, without bringing the environment up.


```
docker-compose build


```


## up


Brings the environment up. Custom images will be automatically built when not cached, and when you make changes to the referenced Dockerfile.


```
docker-compose up


```


## ps


Similar to docker ps, shows the status of active services associated with the current docker-compose.yml file.


```
docker-compose ps


```


## exec


Executes a command on the specified service.


```
docker-compose exec service-name command


```


## stop


Stops the active environment, while keeping any allocated resources: containers, volumes, and networks.


```
docker-compose stop


```


## start


Brings up an environment that was previously stopped with the stop command.


```
docker-compose start


```


## logs


Shows latest logs from active services.


```
docker-compose logs


```


## top


Shows processes running on the specified service.


```
docker-compose top service-name

```


## down


Brings the containerized environment down along with any allocated resources.


```
docker-compose down

```


For more information about each available Docker Compose command, please refer to their official documentation.


# Conclusion


This guide complements the How To Create PHP Development Environments with Docker Compose Workshop Kit’s slides and speaker notes, and is accompanied by a demo GitHub repository containing all necessary files to follow up with the demo component of this workshop.


For a more in-depth guide on containerized PHP environments with Docker Compose, please refer to our tutorial on How To Install and Set Up Laravel with Docker Compose on Ubuntu 20.04.


