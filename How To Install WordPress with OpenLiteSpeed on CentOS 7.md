# How To Install WordPress with OpenLiteSpeed on CentOS 7

```MySQL``` ```WordPress``` ```Control Panels``` ```CentOS```

## Introduction


WordPress is currently the most popular content management system (CMS) in the world.  It allows you to easily set up flexible blogs and websites on top of a database backend, using PHP to execute scripts and process dynamic content.  WordPress has a large online community for support and is a great way to get websites up and running quickly.


In this guide, we will focus on how to get a WordPress instance set up and running on CentOS 7 using the OpenLiteSpeed web server.


# Prerequisites


Before you begin this guide, there are some important steps that you must complete to prepare your server.


We will be running through the steps in this guide using a non-root user with sudo privileges.  To learn how to set up a user of this type, follow our initial server setup guide for CentOS 7.


This guide will not cover how to install OpenLiteSpeed or MySQL.  You can learn how to install and configure these components by following our guide on installing OpenLiteSpeed on CentOS 7.  This will also cover the MySQL installation.


When you are finished preparing your server using the guides linked to above, you can proceed with this article.


# Create a Database and Database User for WordPress


We will start by creating a database and database user for WordPress to use.


Start a MariaDB session by using the root MariaDB username:


```
mysql -u root -p


```


You will be prompted to enter the MariaDB administrative password that you selected while running the mysql_secure_installation script.  Afterwards, you will be dropped into a MariaDB prompt.


First, create a database for our application.  To keep things simple, we’ll call our database wordpress in this guide, but you can use whatever name you’d like:


```
CREATE DATABASE wordpress;


```


Next, we’ll create a database user and grant it access to manage the database that we just created.  We will call this user wordpressuser, but again, feel free to choose a different name.  Replace password in the command below with a strong password for your user:


```
GRANT ALL ON wordpress.* TO wordpressuser@localhost IDENTIFIED BY 'password';


```


Flush the changes you’ve made to make them available to the current MariaDB process:


```
FLUSH PRIVILEGES;


```


Now, exit out of the MariaDB prompt to get back to your regular shell:


```
exit


```


# Install the Necessary PHP Extensions for WordPress


With our database configured, we can go ahead and shift our focus to configuring PHP.


During the OpenLiteSpeed installation, we installed version 5.6 of OpenLiteSpeed’s custom compiled PHP processor.  To enable the functionality we need in WordPress, we’ll need to install some additional extensions.


Luckily, these are all included in OpenLiteSpeed’s repository.  Install the needed extensions by typing:


```
sudo yum install lsphp56-gd lsphp56-process lsphp56-mbstring


```


These will automatically be available to our web server’s PHP instance.


# Configure the Virtual Host for WordPress


We will be modifying the default virtual host that is already present in the OpenLiteSpeed configuration so that we can use it for our WordPress installation.


Log into OpenLiteSpeed’s administrative interface by visiting your server’s domain name or IP address followed by :7080 in your web browser:


```
https://server_domain_or_IP:7080

```


If prompted, log in using the username and password you configured for OpenLiteSpeed in the installation tutorial.


To begin, in the admin interface, select “Virtual Hosts” from the “Configuration” item in the menu bar:





On the “Example” virtual host, click the “View/Edit” link:





This will allow you to edit the configuration of your virtual host.


## Allow index.php Processing


To start, we will enable index.php files so that they can be used to process requests that aren’t handled by static files.  This will allow the main logic of WordPress to function correctly.


Start by clicking on the “General” tab for the virtual host and then clicking the “Edit” button for the “Index Files” table:





In the field for valid “Index Files”, add index.php before index.html to allow PHP index files to take precedence:





Click “Save” when you are finished.


## Configure WordPress Rewrites to Enable Permalink Support


Next, we will set up the rewrite instructions so that we can use permalinks within our WordPress installation.


To do so, click on the “Rewrite” tab for the virtual host.  In the next screen, click on the “Edit” button for the “Rewrite Control” table:





Select “Yes” under the “Enable Rewrite” option:





Click “Save” to go back to the main rewrite menu.  Click on the “Edit” button for the “Rewrite Rules” table:





Remove the rules that are already present and add the following rules to enable rewrites for WordPress:


```
RewriteRule ^/index\.php$ - [L]
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule . /index.php [L]

```


Click on the “Save” button to implement your new rewrite rules.


## Remove Unused Password Protection


The default virtual host that is included with the OpenLiteSpeed installation includes some password protected areas to showcase OpenLiteSpeed’s user authentication features.  WordPress includes its own authentication mechanisms and we will not be using the file-based authentication included in OpenLiteSpeed.  We should get rid of these in order to minimize the stray configuration fragments active on our WordPress installation.


First, click on the “Security” tab, and then click the “Delete” link next to “SampleProtectedArea” within the “Realms List” table:





You will be asked to confirm the deletion.  Click “Yes” to proceed:





Next, click on the “Context” tab.  In the “Context List”, delete the /protected/ context that was associated with the security realm you just deleted:





Again, you will have to confirm the deletion by clicking “Yes”.


You can safely delete any or all of the other contexts as well using the same technique.  We will not be needing them.  We specifically delete the /protected/ context because otherwise, an error would be produced due to the deletion of its associated security realm (which we just removed in the “Security” tab).


## Restart the Server to Implement the Changes


With all of the above configuration out of the way, we can now gracefully restart the OpenLiteSpeed server to enable our changes.


Go to the “Actions” item in the main menu bar and select “Graceful Restart”:





Once the server has restarted, click on the “Home” link in the menu bar.  Any errors that have occurred will be printed at the bottom of this page.  If you see errors, click on “Actions” and then “Server Log Viewer” to get more information.


# Prepare the Virtual Host and Document Root Directories


The last thing that we need to do before installing and configuring WordPress is clean up our virtual host and document root directories.  As we said in the last section, the default site has some extraneous pieces that we won’t be using for our WordPress site.


Start by moving into the virtual host root directory:


```
cd /usr/local/lsws/DEFAULT


```


If you deleted all of the entries in the “Contexts” tab in the last section, you can get rid of the cgi-bin and fsci-bin directories entirely:


```
sudo rm -rf cgi-bin fcgi-bin


```


If you have left these contexts enabled, you should at least remove any scripts currently present in these directories by typing:


```
sudo rm cgi-bin/* fcgi-bin/*


```


You may see a warning about not being able to remove fastcgi-bin/*.  This will happen if there was nothing present in that directory and is completely normal.


Next, we should remove the password and group files that previously protected our “/protected/” context.  Do this by typing:


```
sudo rm conf/ht*


```


Finally, we should clear out the present contents of our document root directory.  You can do that by typing:


```
sudo rm -rf html/*

```


We now have a clean place to transfer our WordPress files.


# Install and Configure WordPress


We are now ready to download and install WordPress.  Move to your home directory and download the latest version of WordPress by typing:


```
cd ~
wget https://wordpress.org/latest.tar.gz


```


Extract the archive and enter the directory by typing:


```
tar xzvf latest.tar.gz
cd wordpress


```


We can copy the sample WordPress configuration file to wp-config.php, the file that WordPress actually reads and processes.  This is where we will put our database connection details:


```
cp wp-config-sample.php wp-config.php


```


Open the configuration file so that we can add our database credentials:


```
nano wp-config.php


```


We need to find the settings for DB_NAME, DB_USER, and DB_PASSWORD so that WordPress can authenticate and utilized the database that we set up for it.


Fill in the values of these parameters with the information for the database you created.  It should look something like this:


```
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define('DB_NAME', 'wordpress');

/** MySQL database username */
define('DB_USER', 'wordpressuser');

/** MySQL database password */
define('DB_PASSWORD', 'password');

```


Save and close the file when you are finished.


Now, we are ready to copy the files into our document root.  To do this, type:


```
sudo cp -r ~/wordpress/* /usr/local/lsws/DEFAULT/html/


```


Give permission of the entire directory structure to the user that the web server runs under so that changes can be made through the WordPress interface:


```
sudo chown -R nobody:nobody /usr/local/lsws/DEFAULT/html


```


# Finishing the Installation Through the WordPress Interface


With the files installed, we can access our WordPress installation by going to our server’s domain name or IP address.  If you changed the port for the default site to port 80 during the OpenLiteSpeed installation in the prerequisite guide, you can access the site directly:


```
http://server_domain_or_IP

```


If you have not switched to port 80, you will have to add :8088 to the end of your address.  Consider switching to port 80 when launching your site using the instructions in the last guide:


```
http://server_domain_or_IP:8088

```


You should see the first screen of the WordPress installation interface, asking you to select a language:





Make your selection and click “Continue”.


On the next page, you will need to fill in some information about the site you are creating.  This will include the site title, an administrative username and password, the admin email account to set, as well as a decision as to whether to prohibit web crawlers:





After the installation, you will have to login using the account you just created.  Once authenticated, you will be taken to the WordPress admin dashboard, allowing you to configure your site:





Your WordPress installation should now be complete.


# Conclusion


In this guide, we’ve installed and configured a WordPress instance on CentOS 7 using the OpenLiteSpeed web server.  This configuration is ideal for many users because both WordPress and the web server itself can mainly be administered through a web browser.  This can make administration and modifications easier for those who do not always have access to an SSH session or who may not feel comfortable managing a web server completely from the command line.


