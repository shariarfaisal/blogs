# How To Install Linux  Apache  MySQL  PHP  LAMP  stack On CentOS 7

```LAMP Stack``` ```Getting Started``` ```CentOS```

## Introduction


A “LAMP” stack is a group of open-source software that is typically installed together to enable a server to host dynamic websites and web apps.  This term is an acronym which represents the Linux operating system, with the Apache web server.  The site data is typically stored in a MySQL database and dynamic content is processed by PHP.


On most Linux systems, you can install MySQL by downloading the mysql-server package from your system’s default package management repositories. However, on CentOS 7 the mysql-server package will actually install MariaDB, a community-developed fork of the MySQL relational database management system which works as a drop-in replacement for MySQL. Thus, this tutorial will outline how to install a LAMP stack that consists of Linux, Apache, MariaDB, and PHP on a CentOS 7 server.


# Prerequisites


Before you begin with this guide, you should have a separate, non-root user account set up on your server.  You can learn how to do this by following our initial server setup for CentOS 7 tutorial.


# Step 1 — Installing the Apache Web Server


Apache is a popular open-source web server that is used to display web pages to visitors. You can configure it to serve PHP pages.


Install Apache using CentOS’s package manager, yum.  A package manager allows you to install most software from a repository maintained by CentOS.


Type this command in your terminal to install the httpd Apache package:


```
sudo yum install httpd


```


When prompted, enter Y to confirm the Apache installation.
Once the installation is complete, start your Apache server with this command:


```
sudo systemctl start httpd


```


You can test if your server is running by entering your public IP address or your domain name in your web browser.



Note: If you are using DigitalOcean as DNS hosting provider, you can check our product docs for detailed instructions on how to set up a new domain name and point it to your server.

If you do not have a domain name pointed at your server or do not know your server’s public IP address, you can find it by running the following command:


```
ip addr show eth0 | grep inet | awk '{ print $2; }' | sed 's/\/.*$//'


```


This will print out a few different addresses. You can try each of them in your web browser.


An alternative method is to use an outside party to tell you how it sees your server.  You can do this by asking a specific server what your IP address is with this command:


```
curl http://icanhazip.com


```


Whichever method you choose, type in your IP address into your web browser to verify that your server is running.


```
http://your_server_IP_address

```


You will be presented with the default CentOS 7 Apache landing page:



You can enable Apache to start on boot with:


```
sudo systemctl enable httpd.service


```


# Step 2 — Installing MySQL (MariaDB)


With your web server up and running, you can install MariaDB. It will organize and provide access to databases where your site can store information.


To install the MariaDB software package, run:


```
sudo yum install mariadb-server


```


When the installation is complete, start MariaDB:


```
sudo systemctl start mariadb


```


You can enable MariaDB to start on boot with this command:


```
sudo systemctl enable mariadb.service


```


To improve the security of your database server, it’s recommended that you run a security script that comes pre-installed with MariaDB. This script will remove some insecure default settings and lock down access to your database system.


Start the interactive script by running:


```
sudo mysql_secure_installation


```


This script will take you through a series of prompts where you can make some changes to your MariaDB setup. The first prompt will ask you to enter the current database root password. This is not to be confused with the system root user. The database root user is an administrative user with full privileges over the database system. Because you just installed MariaDB and haven’t made any configuration changes, this password will be blank. Press ENTER at the prompt.


The next prompt asks you whether you’d like to set up a database root password. Type N and then press ENTER.


From there, you can press Y, and then ENTER, to accept the defaults for all the subsequent questions. This will remove anonymous users and the test database, disable remote root login, and load these new rules so that the server immediately respects the changes you have made.


When you’re finished, log in to the MariaDB console by entering:


```
sudo mysql


```


This connects you to the MariaDB server as the administrative database user root:


```
OutputWelcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 12
Server version: 5.5.68-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement. 

```


For increased security, it’s best to have dedicated user accounts with less expansive privileges set up for every database. This is especially important if you plan on having multiple databases hosted on your server.


To demonstrate such a setup, create a database named example_database and a user named example_user. You can replace these names with different values.


Run the following command from your MariaDB console to create a new database:


```
CREATE DATABASE example_database;


```


You can create a new user and grant them full privileges on the custom database you’ve just created. The following command defines this user’s password as password, but you should replace this value with a secure password:


```
GRANT ALL ON example_database.* TO 'example_user'@'localhost' IDENTIFIED BY 'password' WITH GRANT OPTION;


```


This command gives the example_user user full privileges over the example_database database, while preventing this user from creating or modifying other databases on your server.


Use the FLUSH statement to reload and save the privileges you just granted to example_user:


```
FLUSH PRIVILEGES;


```


Exit the MariaDB shell:


```
exit


```


You can test if the new user has the proper permissions by logging in to the MariaDB console again, but using the example_user credentials you created above:


```
mysql -u example_user -p


```


Note the -p flag in this command, which will prompt you for the password you chose when creating the example_user user. After logging in to the MariaDB console, confirm that you have access to the example_database database with this statement:


```
SHOW DATABASES;


```


Your example_database should be listed in the output:


```
Output+--------------------+
| Database           |
+--------------------+
| example_database   |
| information_schema |
+--------------------+
2 rows in set (0.000 sec)

```


To exit the MariaDB shell, type:


```
exit


```


Your database system is set up and you can move on to installing PHP.


# Step 3 — Installing PHP


You have Apache installed to serve your content and MariaDB to store and manage your data. PHP will process code to display dynamic content to the user.  In addition to the php package, you’ll need php-mysql, a PHP module that allows PHP to communicate with MySQL-based databases. Core PHP packages will automatically be installed as dependencies.


Use this command to install the php and php-mysql packages with yum:


```
sudo yum install php php-mysql


```


Restart the Apache web server to enable the PHP module you installed:


```
sudo systemctl restart httpd.service


```


Your server is now configured with all the components necessary for your LAMP stack application. The next step is to test your configuration to ensure that everything is working harmoniously.


# Step 4 — Testing PHP on your Apache Web Server


The default Apache installation on CentOS 7 will create a document root located at /var/www/html. You don’t need to make any changes to Apache’s default settings in order for PHP to work correctly within your web server.


You can, however, make an adjustment to change the default permission settings on your Apache document root folder. This allows you to create and modify files in that directory with your regular system user without the need to prefix each command with sudo.


The following command will change the ownership of the default Apache document root to a user and group called sammy, so be sure to replace the highlighted username and group in this command to reflect your system’s username and group:


```
sudo chown -R sammy.sammy /var/www/html/


```


You can create a PHP test file to ensure the web server works as expected. Use your preferred text editor to create this file. The following examples use the default vi text editor in CentOS 7.


Create a PHP file called info.php at the var/www/html directory:


```
vi /var/www/html/info.php


```


This opens a blank PHP file at the /var/www/html directory. Press I to enter into INSERT mode in the vi editor. This allows you to type and make changes within the text editor. Type the following PHP code:


/var/www/html/info.php
```
<?php phpinfo(); ?>

```


This PHP code displays information about the PHP environment running on your server. When you are finished with making your changes to this file, press the ESC key to exit out of INSERT mode in vi. Type :x –  a semicolon and the letter x in lowercase –  to save and close the file.


You can test whether your web server correctly displays PHP content by going to your server’s public IP address, followed by /info.php:


```
http://your_server_IP_address/info.php

```


A web page, similar to the one below, will be displayed in your browser:



This page gives you information about your server from the perspective of PHP.  It is useful for debugging and ensuring that your settings are being applied correctly. After checking the relevant information about your PHP server, it’s best to remove this file as it contains sensitive information about your PHP environment and your CentOS server.


You can use rm to remove this file:


```
rm /var/www/html/info.php


```


You can always recreate this page if you need to access the information again later. Next, you can test the database connection utilizing PHP.


# Step 5 – Testing Database Connection with PHP (Optional)


You can test if PHP connects to MariaDB and executes database queries by creating a test table with some test data. You can query for its contents from a PHP script.


First, connect to the MariaDB console with the database user you created in Step 2 of this guide:


```
mysql -u example_user -p


```


From the MariaDB console, run the following statement to create a table named todo_list within your example_database:


```
CREATE TABLE example_database.todo_list (
	item_id INT AUTO_INCREMENT,
	content VARCHAR(255),
	PRIMARY KEY(item_id)
);


```


The MariaDB console will notify you about changes to your table after each edit.


```
Query OK, 0 rows affected (0.00 sec)

```


Insert a few rows of content in the test table. You can repeat the next command a few times, using different values, to populate your test table:


```
INSERT INTO example_database.todo_list (content) VALUES ("My first important item");


```


To confirm that the data was successfully saved to your table, run:


```
SELECT * FROM example_database.todo_list;


```


Below is an example of the output:


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


After confirming that you have valid data in your test table, you can exit the MariaDB console:


```
exit


```


Now you can create the PHP script that will connect to MariaDB and query for your content. Create a new PHP file in your custom web root directory using your preferred editor. This example uses vi:


```
vi /var/www/html/todo_list.php


```


Add the following content by pressing I in the vi text editor, remembering to replace the example_user and password with your own:


/var/www/html/todo_list.php
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


Save and close the file when you’re done editing by pressing ESC, followed by typing :x in vi.


You can now access this page in your web browser by visiting your server’s host name or public IP address, followed by /todo_list.php:


```
http://server_host_or_IP/todo_list.php

```


Below is an example of the web page, revealing the content you’ve inserted in your test table:



# Conclusion


In this guide, you’ve built a flexible foundation for serving PHP websites and applications to your visitors, using Apache as a web server. You’ve set up Apache to handle PHP requests, and set up a MariaDB database to store your website’s data.


