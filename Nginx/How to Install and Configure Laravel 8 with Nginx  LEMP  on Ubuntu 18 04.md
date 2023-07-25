# How to Install and Configure Laravel 8 with Nginx  LEMP  on Ubuntu 18 04

```PHP Frameworks``` ```LEMP``` ```Nginx``` ```Ubuntu 18.04``` ```PHP``` ```Laravel```

## Introduction


Laravel is an open source PHP framework that provides a set of tools and resources to build modern PHP applications. With a complete ecosystem leveraging its built-in features, Laravel’s popularity has grown rapidly in the past few years, with many developers adopting it as their framework of choice for a streamlined development process.


In this guide, you’ll install and configure a new Laravel 8 application on an Ubuntu 18.04 server with PHP 7.4, using Composer to download and manage the framework dependencies. When you’re finished, you’ll have a functional Laravel demo application pulling content from a MySQL database.


# Prerequisites


In order to complete this guide, you will first need to perform the following tasks on your Ubuntu 18.04 server:


- Create a sudo user and enable ufw. To set this up, you can follow our Initial Server Setup with Ubuntu 18.04 guide.
- Install Nginx. If you haven’t set this up yet, you can follow steps 1 and 2 of How To Install Nginx on Ubuntu 18.04.
- Install MySQL. Follow steps 1 and 2 of our guide on How to Install MySQL on Ubuntu 18.04 to set this up.
- Install PHP 7.4 or higher. To accomplish that, you can follow step 1 of our guide on How To Install PHP 7.4 and Set Up a Local Development Environment on Ubuntu 18.04.
- Install Composer. You’ll use Composer to install Laravel and its dependencies. You can install Composer by following our guide on How to Install Composer on Ubuntu 18.04.

To verify which version of PHP your server is currently using, run:


```
php -v


```


```
outputPHP 7.4.22 (cli) (built: Jul 30 2021 13:07:59) ( NTS )
Copyright (c) The PHP Group
Zend Engine v3.4.0, Copyright (c) Zend Technologies
    with Zend OPcache v7.4.22, Copyright (c), by Zend Technologies

```


Check the output to make sure your server has PHP 7.4 or higher installed, since that is the latest stable release of PHP.


# Step 1 — Installing PHP-FPM and Required PHP modules


Before you can install Laravel, you need to install a few PHP modules that are required by the framework. You’ll also need to install PHP-FPM in case you haven’t set that up yet.


Laravel requires the php-mbstring, php-xml and php-bcmath PHP modules. These PHP extensions provide extra support for dealing with character encoding, XML, and precision mathematics. Additionally, you’ll need php-mysql to be able to connect to the database via PHP code.


If this is the first time using apt in this session, you should first run the update command to update the package manager cache:


```
sudo apt update


```


You may be prompted to provide your user password to validate your sudo permissions. Once the update is finished, you can install the required packages with:


```
sudo apt install php7.4-fpm php7.4-mysql php7.4-mbstring php7.4-xml php7.4-bcmath


```


Your system is now ready to execute Laravel’s installation via Composer, but before doing so, you’ll need to set up a database for your application.


# Step 2 — Creating a Database for the Application


To practice Laravel’s basic installation and usage, you’ll create a sample travel list application to show a list of places a user would like to travel to, and a list of places that they already visited. This can be stored in a places table with a field for locations that you’ll call name and another field to mark them as visited or not visited, which you’ll call visited. Additionally, you’ll include an id field to uniquely identify each entry.


To connect to the database from the Laravel application, you’ll create a dedicated MySQL user, and grant this user full privileges over the travel_list database.


To get started, log in to the MySQL console as the root database user with:


```
sudo mysql


```


To create a new database, run the following command from your MySQL console:


```
CREATE DATABASE travel_list;


```


Now you can create a new user and grant them full privileges on the custom database you’ve just created. In this example, we’re creating a user named travel_user with the password password, though you should change this to a secure password of your choosing:


```
GRANT ALL ON travel_list.* TO 'travel_user'@'localhost' IDENTIFIED BY 'password' WITH GRANT OPTION;


```


This will give the travel_user user full privileges over the travel_list database, while preventing this user from creating or modifying other databases on your server.


Following this, exit the MySQL shell:


```
exit


```


You can now test if the new user has the proper permissions by logging into the MySQL console again, this time using the custom user credentials:


```
mysql -u travel_user -p


```


Note the -p flag in this command, which will prompt you for the password used when creating the travel_user user. After logging into the MySQL console, confirm that you have access to the travel_list database:


```
SHOW DATABASES;


```


This will give you the following output:


```
Output+--------------------+
| Database           |
+--------------------+
| information_schema |
| travel_list        |
+--------------------+
2 rows in set (0.01 sec)

```


Next, create a table named places in the travel_list database. From the MySQL console, run the following statement:


```
CREATE TABLE travel_list.places (
	id INT AUTO_INCREMENT,
	name VARCHAR(255),
	visited BOOLEAN,
	PRIMARY KEY(id)
);


```


Now, populate the places table with some sample data:


```
INSERT INTO travel_list.places (name, visited) 
VALUES ("Tokyo", false),
("Budapest", true),
("Nairobi", false),
("Berlin", true),
("Lisbon", true),
("Denver", false),
("Moscow", false),
("Olso", false),
("Rio", true),
("Cincinnati", false),
("Helsinki", false);


```


To confirm that the data was successfully saved to your table, run:


```
SELECT * FROM travel_list.places;


```


You will see output similar to this:


```
Output+----+-----------+---------+
| id | name      | visited |
+----+-----------+---------+
|  1 | Tokyo     |       0 |
|  2 | Budapest  |       1 |
|  3 | Nairobi   |       0 |
|  4 | Berlin    |       1 |
|  5 | Lisbon    |       1 |
|  6 | Denver    |       0 |
|  7 | Moscow    |       0 |
|  8 | Oslo      |       0 |
|  9 | Rio       |       1 |
| 10 | Cincinnati|       0 |
| 11 | Helsinki  |       0 |
+----+-----------+---------+
11 rows in set (0.00 sec)


```


After confirming that you have valid data in your test table, you can exit the MySQL console:


```
exit


```


You’re now ready to create the application and configure it to connect to the new database.


# Step 3 — Creating a New Laravel Application


You will now create a new Laravel application using the composer create-project command. This Composer command is typically used to bootstrap new applications based on existing frameworks and content management systems.


Throughout this guide, we’ll use travel_list as an example application, but you are free to change this to something else. The travel_list application will display a list of locations pulled from a local MySQL server, intended to demonstrate Laravel’s basic configuration and confirm that you’re able to connect to the database.


First, go to your user’s home directory:


```
cd ~


```


The following command will create a new travel_list directory containing a barebones Laravel application based on default settings:


```
composer create-project --prefer-dist laravel/laravel travel_list


```


You will see output similar to this:


```
OutputCreating a "laravel/laravel" project at "./travel_list"
Installing laravel/laravel (v8.5.22)
  - Installing laravel/laravel (v8.5.22): Extracting archive
Created project in /home/sammy/travel_list
> @php -r "file_exists('.env') || copy('.env.example', '.env');"
Loading composer repositories with package information
Updating dependencies
Lock file operations: 109 installs, 0 updates, 0 removals
  - Locking asm89/stack-cors (v2.0.3)
  - Locking brick/math (0.9.2)
  - Locking dflydev/dot-access-data (v3.0.0)
  - Locking doctrine/inflector (2.0.3)
  - Locking doctrine/instantiator (1.4.0)
  - Locking doctrine/lexer (1.2.1)   
…
Package manifest generated successfully.
75 packages you are using are looking for funding.
Use the `composer fund` command to find out more!
> @php artisan key:generate --ansi
Application key set successfully.

```


When the installation is finished, access the application’s directory and run Laravel’s artisan command to verify that all components were successfully installed:


```
cd travel_list
php artisan


```


You’ll see output similar to this:


```
OutputLaravel Framework 8.52.0

Usage:
  command [options] [arguments]

Options:
  -h, --help            Display help for the given command. When no command is given display help for the list command
  -q, --quiet           Do not output any message
  -V, --version         Display this application version
      --ansi|--no-ansi  Force (or disable --no-ansi) ANSI output
  -n, --no-interaction  Do not ask any interactive question
      --env[=ENV]       The environment the command should run under
  -v|vv|vvv, --verbose  Increase the verbosity of messages: 1 for normal output, 2 for more verbose output and 3 for debug
...

```


This output confirms that the application files are in place, and the Laravel command-line tools are working as expected. However, you still need to configure the application to set up the database and a few other details.


# Step 4 — Configuring Laravel


The Laravel configuration files are located in a directory called config, inside the application’s root directory. Additionally, when you install Laravel with Composer, it creates an environment file. This file contains settings that are specific to the current environment the application is running, and will take precedence over the values set in regular configuration files located at the config directory. Each installation on a new environment requires a tailored environment file to define things such as database connection settings, debug options, application URL, among other items that may vary depending on which environment the application is running.



Warning: The environment configuration file contains sensitive information about your server, including database credentials and security keys. For that reason, you should never share this file publicly.

We’ll now edit the .env file to customize the configuration options for the current application environment.


Open the .env file using your command line editor of choice. Here we’ll use nano:


```
nano .env


```


Even though there are many configuration variables in this file, you don’t need to set up all of them now. The following list contains an overview of the variables that require immediate attention:


- APP_NAME: Application name, used for notifications and messages.
- APP_ENV: Current application environment.
- APP_KEY: Used for generating salts and hashes, this unique key is automatically created when installing Laravel via Composer, so you don’t need to change it.
- APP_DEBUG: Whether or not to show debug information at client side.
- APP_URL: Base URL for the application, used for generating application links.
- DB_DATABASE: Database name.
- DB_USERNAME: Username to connect to the database.
- DB_PASSWORD: Password to connect to the database.

By default, these values are configured for a local development environment that uses Homestead, a prepackaged Vagrant box provided by Laravel. We’ll change these values to reflect the current environment settings of our example application.


In case you are installing Laravel in a development  or testing environment, you can leave the APP_DEBUG option enabled, as this will give you important debug information while testing the application from a browser. The APP_ENV variable should be set to development or testing in this case.


In case you are installing Laravel in a production environment, you should disable the APP_DEBUG option, because it shows to the final user sensitive information about your application. The APP_ENV in this case should be set to production.


The following .env file sets up our example application for development:



Note: The APP_KEY variable contains a unique key that was auto generated when you installed Laravel via Composer. You don’t need to change this value. If you want to generate a new secure key, you can use the php artisan key:generate command.

/var/www/travel_list/.env
```
APP_NAME=TravelList
APP_ENV=development
APP_KEY=APPLICATION_UNIQUE_KEY_DONT_COPY
APP_DEBUG=true
APP_URL=http://domain_or_IP

LOG_CHANNEL=stack

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=travel_list
DB_USERNAME=travel_user
DB_PASSWORD=password

...

```


Adjust your variables accordingly. When you are done editing, save and close the file to keep your changes. If you’re using nano, you can do that with CTRL+X, then Y and Enter to confirm.


Your Laravel application is now set up, but we still need to configure the web server in order to be able to access it from a browser. In the next step, we’ll configure Nginx to serve your Laravel application.


# Step 5 — Setting Up Nginx


We have installed Laravel on a local folder of your remote user’s home directory, and while this works well for local development environments, it’s not a recommended practice for web servers that are open to the public internet. We’ll move the application folder to /var/www, which is the usual location for web applications running on Nginx.


First, use the mv command to move the application folder with all its contents to /var/www/travel_list:


```
sudo mv ~/travel_list /var/www/travel_list


```


Now we need to give the web server user write access to the storage and cache folders, where Laravel stores application-generated files:


```
sudo chown -R www-data.www-data /var/www/travel_list/storage
sudo chown -R www-data.www-data /var/www/travel_list/bootstrap/cache


```


The application files are now in order, but we still need to configure Nginx to serve the content. To do this, we’ll create a new virtual host configuration file at /etc/nginx/sites-available:


```
sudo nano /etc/nginx/sites-available/travel_list


```


The following configuration file contains the recommended settings for Laravel applications on Nginx:


/etc/nginx/sites-available/travel_list
```
server {
    listen 80;
    server_name server_domain_or_IP;
    root /var/www/travel_list/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Content-Type-Options "nosniff";

    index index.html index.htm index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}

```


Copy this content to your /etc/nginx/sites-available/travel_list file and, if necessary, adjust the highlighted values to align with your own configuration. Save and close the file when you’re done editing.


To activate the new virtual host configuration file, create a symbolic link to travel_list in sites-enabled:


```
sudo ln -s /etc/nginx/sites-available/travel_list /etc/nginx/sites-enabled/


```



Note: If you have another virtual host file that was previously configured for the same server_name used in the travel_list virtual host, you might need to deactivate the old configuration by removing the corresponding symbolic link inside /etc/nginx/sites-enabled/. For example, you may want to delete the /etc/nginx/sites-enabled/default file to disable Nginx’s default welcome page with:
sudo rm -f /etc/nginx/sites-enabled/default



To confirm that the configuration doesn’t contain any syntax errors, you can use:


```
sudo nginx -t


```


You should see output like this:


```
Outputnginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful


```


To apply the changes, reload Nginx with:


```
sudo systemctl reload nginx


```


Now go to your browser and access the application using the server’s domain name or IP address, as defined by the  server_name directive in your configuration file:


```
http://server_domain_or_IP

```


You will see a page like this:





That confirms your Nginx server is properly configured to serve Laravel. From this point, you can start building up your application on top of the skeleton provided by the default installation.


In the next step, we’ll modify the application’s main route to query for data in the database using Laravel’s DB facade.


# Step 6 — Customizing the Main Page


Assuming you’ve followed all the steps in this guide so far, you should have a working Laravel application and a database table named places containing some sample data.


We’ll now edit the main application route to query for the database and return the contents to the application’s view.


Open the main route file, routes/web.php:


```
nano routes/web.php


```


This file comes by default with the following content:


routes/web.php
```
<?php

/*
|--------------------------------------------------------------------------
| Web Routes
|--------------------------------------------------------------------------
|
| Here is where you can register web routes for your application. These
| routes are loaded by the RouteServiceProvider within a group which
| contains the "web" middleware group. Now create something great!
|
*/

Route::get('/', function () {
    return view('welcome');
});


```


Routes are defined within this file using the static method Route::get, which receives a path and a callback function as arguments.


The following code replaces the main route callback function. It makes 2 queries to the database using the visited flag to filter results. It returns the results to a view named travel_list, which we’re going to create next. Copy this content to your routes/web.php file, replacing the code that is already there:


routes/web.php
```
<?php

use Illuminate\Support\Facades\DB;

Route::get('/', function () {
  $visited = DB::select('select * from places where visited = ?', [1]);	
  $togo = DB::select('select * from places where visited = ?', [0]);

  return view('travel_list', ['visited' => $visited, 'togo' => $togo ] );
});

```


Save and close the file when you’re done editing. We’ll now create the view that will render the database results to the user. Create a new view file inside resources/views:


```
nano resources/views/travel_list.blade.php


```


The following template creates two lists of places based on the variables visited and togo. Copy this content to your new view file:


resources/views/travel_list/blade.php
```
<html>
<head>
	<title>Travel List</title>
</head>

<body>
	<h1>My Travel Bucket List</h1>
	<h2>Places I'd Like to Visit</h2>
	<ul>
	  @foreach ($togo as $newplace)
		<li>{{ $newplace->name }}</li>
	  @endforeach
	</ul>

	<h2>Places I've Already Been To</h2>
	<ul>
          @foreach ($visited as $place)
                <li>{{ $place->name }}</li>
          @endforeach
	</ul>
</body>
</html>

```


Save and close the file when you’re done. Now go to your browser and reload the application. You’ll see a page like this:





You have now a functional Laravel application pulling contents from a MySQL database.


# Conclusion


In this tutorial, you’ve set up a new Laravel application on top of a LEMP stack (Linux, Nginx, MySQL and PHP), running on an Ubuntu 18.04 server. You’ve also customized your default route to query for database content and exhibit the results in a custom view.


From here, you can create new routes and views for any additional pages your application needs. Check the official Laravel documentation for more information on routes, views, and database support. If you’re deploying to production, you should also check the optimization section for a few different ways in which you can improve your application’s performance.


For improved security, you should consider installing an TLS/SSL certificate for your server, allowing it to serve content over HTTPS. To this end, you can follow our guide on [how to secure your Nginx installation with Let’s Encrypt on Ubuntu 18.04] (https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-18-04).


Check also our Laravel tag page for more learning materials about this framework, including tech talks and tutorial series.


