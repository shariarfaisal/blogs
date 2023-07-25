# How To Install Linux  Nginx  MySQL  PHP  LEMP stack  on Ubuntu 22 04

```LEMP``` ```Nginx``` ```PHP``` ```Ubuntu``` ```Ubuntu 22.04```

## Introduction


The LEMP software stack is a group of software that can be used to serve dynamic web pages and web applications written in PHP. This is an acronym that describes a Linux operating system, with an Nginx (pronounced like “Engine-X”) web server. The backend data is stored in the MySQL database and the dynamic processing is handled by PHP.


This guide demonstrates how to install a LEMP stack on an Ubuntu 22.04 server. The Ubuntu operating system takes care of the Linux portion of the stack. We will describe how to get the rest of the components up and running.


# Prerequisites


To complete this tutorial, you will need access to an Ubuntu 22.04 server as a regular, non-root sudo user, and a firewall enabled on your server. To set this up, you can follow our initial server setup guide for Ubuntu 22.04.


# Step 1 – Installing the Nginx Web Server


To display web pages to site visitors, you’re going to employ Nginx, a high-performance web server. You’ll use the APT package manager to obtain this software.


Since this is your first time using apt for this session, start off by updating your server’s package index:


```
sudo apt update


```


Following that, run apt install to install Nginx:


```
sudo apt install nginx


```


When prompted, press Y and ENTER to confirm that you want to install Nginx. Once the installation is finished, the Nginx web server will be active and running on your Ubuntu 22.04 server.


If you have the ufw firewall enabled, as recommended in our initial server setup guide, you will need to allow connections to Nginx. Nginx registers a few different UFW application profiles upon installation. To check which UFW profiles are available, run:


```
sudo ufw app list


```


```
OutputAvailable applications:
  Nginx Full
  Nginx HTTP
  Nginx HTTPS
  OpenSSH

```


It is recommended that you enable the most restrictive profile that will still allow the traffic you need. Since you haven’t configured SSL for your server in this guide, you will only need to allow regular HTTP traffic on port 80.


Enable this by running the following:


```
sudo ufw allow 'Nginx HTTP'


```


You can verify the change by checking the status:


```
sudo ufw status


```


This output displays that HTTP traffic is now allowed:


```
OutputStatus: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
Nginx HTTP                 ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
Nginx HTTP (v6)            ALLOW       Anywhere (v6)

```


With the new firewall rule added, you can test if the server is up and running by accessing your server’s domain name or public IP address in your web browser.


If you do not have a domain name pointed at your server and you do not know your server’s public IP address, you can find it by running either of the following commands:


```
ip addr show


```


```
hostname -I


```


This will print out a few IP addresses. You can try each of them in turn in your web browser.


As an alternative, you can check which IP address is accessible, as viewed from other locations on the internet:


```
curl -4 icanhazip.com


```


Write the address that you receive in your web browser and it will take you to Nginx’s default landing page:


```
http://server_domain_or_IP

```





If you receive this page, it means you have successfully installed Nginx and enabled HTTP traffic for your web server.


# Step 2 — Installing MySQL


Now that you have a web server up and running, you need to install the database system to store and manage data for your site. MySQL is a popular database management system used within PHP environments.


Again, use apt to acquire and install this software:


```
sudo apt install mysql-server


```


When prompted, confirm installation by pressing Y, and then ENTER.


When the installation is finished, it’s recommended that you run a security script that comes pre-installed with MySQL. This script will remove some insecure default settings and lock down access to your database system. Start the interactive script by running the following command:


```
sudo mysql_secure_installation


```


You will be prompted with a question asking if you want to configure the VALIDATE PASSWORD PLUGIN.



Note: Enabling this feature is something of a judgment call. If enabled, passwords which don’t match the specified criteria will be rejected by MySQL with an error. It is safe to leave validation disabled, but you should always use strong, unique passwords for database credentials.

Answer Y for yes, or anything else to continue without enabling:


```
OutputVALIDATE PASSWORD COMPONENT can be used to test passwords
and improve security. It checks the strength of password
and allows the users to set only those passwords which are
secure enough. Would you like to setup VALIDATE PASSWORD component?

Press y|Y for Yes, any other key for No: 

```


If you answer “yes”, you’ll be asked to select a level of password validation. Keep in mind that if you enter 2 for the strongest level, you will receive errors when attempting to set any password that does not contain numbers, upper and lowercase letters, and special characters:


```
OutputThere are three levels of password validation policy:

LOW    Length >= 8
MEDIUM Length >= 8, numeric, mixed case, and special characters
STRONG Length >= 8, numeric, mixed case, special characters and dictionary              file

Please enter 0 = LOW, 1 = MEDIUM and 2 = STRONG: 1

```


Regardless of whether you chose to set up the VALIDATE PASSWORD PLUGIN, your server will ask you to select and confirm a password for the MySQL root user. This is not to be confused with the system root. The database root user is an administrative user with full privileges over the database system. Even though the default authentication method for the MySQL root user dispenses the use of a password, even when one is set, you should define a strong password here as an additional safety measure. We’ll talk about this in a moment.


If you enabled password validation, you’ll be shown the password strength for the root password you entered and your server will ask if you want to continue with that password. If you are happy with your current password, press Y for “yes” at the prompt:


```
OutputEstimated strength of the password: 100 
Do you wish to continue with the password provided?(Press y|Y for Yes, any other key for No) : y

```


For the rest of the questions, press Y and hit the ENTER key at each prompt. This will remove some anonymous users and the test database, disable remote root logins, and load these new rules so that MySQL immediately respects the changes you have made.


When you’re finished, test if you’re able to log in to the MySQL console:


```
sudo mysql


```


This will connect to the MySQL server as the administrative database user root, which is inferred by the use of sudo when running this command. You should receive the following output:


```
OutputWelcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 10
Server version: 8.0.28-0ubuntu4 (Ubuntu)

Copyright (c) 2000, 2022, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>

```


To exit the MySQL console,write the following:


```
exit


```


Notice that you didn’t need to provide a password to connect as the root user, even though you have defined one when running the mysql_secure_installation script. This is because, when installed on Ubuntu, the default authentication method for the administrative MySQL user is auth_socket, rather than with a method that uses a password. This may seem like a security concern at first, but it makes the database server more secure since the only users allowed to log in as the root MySQL user are the system users with sudo privileges connecting from the console or through an application running with the same privileges. In practical terms, that means you won’t be able to use the administrative database root user to connect from your PHP application.


For increased security, it’s best to have dedicated user accounts with less expansive privileges set up for every database, especially if you plan on having multiple databases hosted on your server.



Note: Certain older releases of the native MySQL PHP library mysqlnd don’t support caching_sha2_authentication, the default authentication method for created users MySQL 8. For that reason, when creating database users for PHP applications on MySQL 8, you may need to make sure they’re configured to use mysql_native_password instead. We’ll demonstrate how to do that in Step 6.

Your MySQL server is now installed and secured. Next, you’ll install PHP, the final component in the LEMP stack.


# Step 3 – Installing PHP


You have Nginx installed to serve your content and MySQL installed to store and manage your data. Now you can install PHP to process code and generate dynamic content for the web server.


While Apache embeds the PHP interpreter in each request, Nginx requires an external program to handle PHP processing and act as a bridge between the PHP interpreter itself and the web server. This allows for better overall performance in most PHP-based websites, but it requires additional configuration. You’ll need to install php8.1-fpm, which stands for “PHP fastCGI process manager” and uses the current version of PHP (at the time of writing), to tell Nginx to pass PHP requests to this software for processing. Additionally, you’ll need php-mysql, a PHP module that allows PHP to communicate with MySQL-based databases. Core PHP packages will automatically be installed as dependencies.


To install the php8.1-fpm and php-mysql packages, run:


```
sudo apt install php8.1-fpm php-mysql


```


When prompted, press Y and ENTER to confirm the installation.


You now have your PHP components installed. Next, you’ll configure Nginx to use them.


# Step 4 — Configuring Nginx to Use the PHP Processor


When using the Nginx web server, we can create server blocks (similar to virtual hosts in Apache) to encapsulate configuration details and host more than one domain on a single server. In this guide, we’ll use your_domain as an example domain name.



Info: To learn more about setting up a domain name with DigitalOcean, read our introduction to DigitalOcean DNS.

On Ubuntu 22.04, Nginx has one server block enabled by default and is configured to serve documents out of a directory at /var/www/html. While this works well for a single site, it can become difficult to manage if you are hosting multiple sites. Instead of modifying /var/www/html, we’ll create a directory structure within /var/www for the your_domain website, leaving /var/www/html in place as the default directory to be served if a client request doesn’t match any other sites.


Create the root web directory for your_domain as follows:


```
sudo mkdir /var/www/your_domain


```


Next, assign ownership of the directory with the $USER environment variable, which will reference your current system user:


```
sudo chown -R $USER:$USER /var/www/your_domain


```


Then, open a new configuration file in Nginx’s sites-available directory using your preferred command-line editor. Here, we’ll use nano:


```
sudo nano /etc/nginx/sites-available/your_domain


```


This will create a new blank file. Insert the following bare-bones configuration:


/etc/nginx/sites-available/your_domain
```
server {
    listen 80;
    server_name your_domain www.your_domain;
    root /var/www/your_domain;

    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
     }

    location ~ /\.ht {
        deny all;
    }

}

```


Here’s what each of these directives and location blocks do:


- listen — Defines what port Nginx will listen on. In this case, it will listen on port 80, the default port for HTTP.
- root — Defines the document root where the files served by this website are stored.
- index — Defines in which order Nginx will prioritize index files for this website. It is a common practice to list index.html files with higher precedence than index.php files to allow for quickly setting up a maintenance landing page in PHP applications. You can adjust these settings to better suit your application needs.
- server_name — Defines which domain names and/or IP addresses this server block should respond for. Point this directive to your server’s domain name or public IP address.
- location / — The first location block includes a try_files directive, which checks for the existence of files or directories matching a URL request. If Nginx cannot find the appropriate resource, it will return a 404 error.
- location ~ \.php$ — This location block handles the actual PHP processing by pointing Nginx to the fastcgi-php.conf configuration file and the php8.1-fpm.sock file, which declares what socket is associated with php8.1-fpm.
- location ~ /\.ht — The last location block deals with .htaccess files, which Nginx does not process. By adding the deny all directive, if any .htaccess files happen to find their way into the document root, they will not be served to visitors.

When you’re done editing, save and close the file. If you’re using nano, you can do so by pressing CTRL+X and then Y and ENTER to confirm.


Activate your configuration by linking to the configuration file from Nginx’s sites-enabled directory:


```
sudo ln -s /etc/nginx/sites-available/your_domain /etc/nginx/sites-enabled/


```


Then, unlink the default configuration file from the /sites-enabled/ directory:


```
sudo unlink /etc/nginx/sites-enabled/default


```



Note: If you ever need to restore the default configuration, you can do so by recreating the symbolic link, like the following:
sudo ln -s /etc/nginx/sites-available/default /etc/nginx/sites-enabled/



This will tell Nginx to use the configuration next time it is reloaded. You can test your configuration for syntax errors by running the following:


```
sudo nginx -t


```


If any errors are reported, go back to your configuration file to review its contents before continuing.


When you are ready, reload Nginx to apply the changes:


```
sudo systemctl reload nginx


```


Your new website is now active, but the web root /var/www/your_domain is still empty. Create an index.html file in that location so that you can test that your new server block works as expected:


```
nano /var/www/your_domain/index.html


```


Include the following content in this file:


/var/www/your_domain/index.html
```
<html>
  <head>
    <title>your_domain website</title>
  </head>
  <body>
    <h1>Hello World!</h1>

    <p>This is the landing page of <strong>your_domain</strong>.</p>
  </body>
</html>

```


Now go to your browser and access your server’s domain name or IP address, as listed within the server_name directive in your server block configuration file:


```
http://server_domain_or_IP

```


You’ll receive a page like the following:





If you receive this page, it means your Nginx server block is working as expected.


You can leave this file in place as a temporary landing page for your application until you set up an index.php file to replace it. Once you do that, remember to remove or rename the index.html file from your document root, as it would take precedence over an index.php file by default.


Your LEMP stack is now fully configured. In the next step, you’ll create a PHP script to test that Nginx is in fact able to handle .php files within your newly configured website.


# Step 5 –Testing PHP with Nginx


Your LEMP stack should now be completely set up. You can test it to validate that Nginx can correctly hand .php files off to your PHP processor.


You can do this by creating a test PHP file in your document root. Open a new file called info.php within your document root in your preferred text editor:


```
nano /var/www/your_domain/info.php


```


Add the following lines into the new file. This is valid PHP code that will return information about your server:


/var/www/your_domain/info.php
```
<?php
phpinfo();

```


When you are finished, save and close the file.


You can now access this page in your web browser by visiting the domain name or public IP address you’ve set up in your Nginx configuration file, followed by /info.php:


```
http://server_domain_or_IP/info.php

```


You will receive a web page containing detailed information about your server:





After checking the relevant information about your PHP server through that page, it’s best to remove the file you created as it contains sensitive information about your PHP environment and your Ubuntu server. You can use rm to remove that file:


```
sudo rm /var/www/your_domain/info.php


```


You can always regenerate this file if you need it later.


# Step 6 — Testing Database Connection from PHP (Optional)


If you want to test whether PHP is able to connect to MySQL and execute database queries, you can create a test table with dummy data and query for its contents from a PHP script. Before doing so, you need to create a test database and a new MySQL user properly configured to access it.



Note: Certain older releases of the native MySQL PHP library mysqlnd don’t support caching_sha2_authentication, the default authentication method for MySQL 8, you may need to make sure they’re configured to use mysql_native_password instead.

We’ll create a database named example_database and a user named example_user, but you can replace these names with different values.


First, connect to the MySQL console using the root account:


```
sudo mysql


```


To create a new database, run the following command from your MySQL console:


```
CREATE DATABASE example_database;


```


Now you can create a new user and grant them full privileges on the custom database you’ve created.


The following command creates a new user named example_user, using mysql_native_password as the default authentication method. We’re defining this user’s password as password, but you should replace this value with a secure password of your own choosing.


```
CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY 'password';


```


Now we need to give this user permission over the example_database database:


```
GRANT ALL ON example_database.* TO 'example_user'@'%';


```


This will give the example_user user full privileges over the example_database database while preventing this user from creating or modifying other databases on your server.


Now exit the MySQL shell with the following command:


```
exit


```


You can test if the new user has the proper permissions by logging in to the MySQL console again, this time using the custom user credentials. Notice the -p flag in this command, which will prompt you for the password used when creating the example_user user:


```
mysql -u example_user -p


```


After logging in to the MySQL console, confirm that you have access to the example_database database:


```
SHOW DATABASES;


```


This will return the following output:


```
Output+--------------------+
| Database           |
+--------------------+
| example_database   |
| information_schema |
+--------------------+
2 rows in set (0.000 sec)

```


Next, we’ll create a test table named todo_list. From the MySQL console, run the following statement:


```
CREATE TABLE example_database.todo_list (
	item_id INT AUTO_INCREMENT,
	content VARCHAR(255),
	PRIMARY KEY(item_id)
);


```


Insert a few rows of content in the test table. You might want to repeat the next command a few times, using different values:


```
INSERT INTO example_database.todo_list (content) VALUES ("My first important item");


```


To confirm that the data was successfully saved to your table, run:


```
SELECT * FROM example_database.todo_list;


```


Your output should display as follows:


```
Output+---------+--------------------------+
| item_id | content                  |
+---------+--------------------------+
|       1 | My first important item  |
|       2 | My second important item |
|       3 | My third important item  |
|       4 | and this one more thing  |
+---------+--------------------------+
4 rows in set (0.000 sec)


```


After confirming that you have valid data in your test table, you can exit the MySQL console:


```
exit


```


Now you can create the PHP script that will connect to MySQL and query for your content. Create a new PHP file in your custom web root directory using your preferred editor. We’ll use nano for that:


```
nano /var/www/your_domain/todo_list.php


```


The following PHP script connects to the MySQL database and queries for the content of the todo_list table, exhibiting the results in a list. If there’s a problem with the database connection, it will throw an exception.


Add the following content to your todo_list.php script:


/var/www/your_domain/todo_list.php
```
<?php
$user = "example_user";
$password = "password";
$database = "example_database";
$table = "todo_list";

try {
  $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
  echo "<h2>TODO</h2><ol>"; 
  foreach($db->query("SELECT content FROM $table") as $row) {
    echo "<li>" . $row['content'] . "</li>";
  }
  echo "</ol>";
} catch (PDOException $e) {
    print "Error!: " . $e->getMessage() . "<br/>";
    die();
}

```


Save and close the file when you’re done editing.


You can now access this page in your web browser by visiting the domain name or public IP address configured for your website, followed by /todo_list.php:


```
http://server_domain_or_IP/todo_list.php

```


You should receive a page like the following, showing the content you’ve inserted in your test table:





That means your PHP environment is ready to connect and interact with your MySQL server.


# Conclusion


In this guide, you built a flexible foundation for serving PHP websites and applications to your visitors, using Nginx as a web server and MySQL as the database system. You can do this with an Apache web server as well, check out our tutorial on How To Install Linux, Apache, MySQL, PHP (LAMP) stack on Ubuntu 22.04. You can also secure your site with Let’s Encrypt, which provides free, trusted certificates. Learn how to do this with our guide on Let’s Encrypt for Apache.


