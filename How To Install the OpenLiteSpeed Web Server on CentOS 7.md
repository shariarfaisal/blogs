# How To Install the OpenLiteSpeed Web Server on CentOS 7

```Databases``` ```Miscellaneous``` ```MariaDB``` ```Control Panels``` ```CentOS```

## Introduction


OpenLiteSpeed is an optimized open source web server that can be used to manage and serve sites.  As far as Linux web servers are concerned, OpenLiteSpeed has some interesting features that make it a solid choice for many installations.  It features Apache compatible rewrite rules, a web administration interface, and customized PHP processing optimized for the server.


In this guide, we’ll demonstrate how to install and configure OpenLiteSpeed on CentOS 7 server.  We will also download and install MariaDB to complete the conventional setup of a web server, dynamic script processor, and database management system.


# Prerequisites and Goals


Before we begin, you should have a non-root user account configured on your server, complete with sudo privileges.  To learn how to set up an account like this, follow our CentOS 7 initial server setup guide.


This tutorial will guide you through the process of installing and configuring an OpenLiteSpeed instance on your server.  We will also install and configure MariaDB to facilitate interaction with many common web applications and services.  OpenLiteSpeed uses a customized version of PHP that is also available from the OpenLiteSpeed repos.  We will install the custom PHP package and the custom PHP extensions we need.


# Add the OpenLiteSpeed Repository


The OpenLiteSpeed project maintains a package repository for CentOS 7.  We can use this to install OpenLiteSpeed and its associated packages without having to compile the software ourselves.


We can add the repository information to our system by typing:


```
sudo rpm -ivh http://rpms.litespeedtech.com/centos/litespeed-repo-1.1-1.el7.noarch.rpm


```


This will update the list of repositories that yum references when searching for and installing packages.


# Install the Components


Now that we have access to the OpenLiteSpeed repository, we can install all of the components we need.


To start, we can install the web server itself along with MariaDB, the database management system we will be using:


```
sudo yum install openlitespeed mariadb-server


```


A version of PHP customized to work well with OpenLiteSpeed is included with the standard installation.  However, the version included is in the PHP 5.3 family.  The OpenLiteSpeed repositories include other versions of PHP customized to work with the web server.


We will install PHP version 5.6 and the PHP extension needed to connect to a MariaDB database.  The OpenLiteSpeed versions of these components will start with “ls”.  Install PHP 5.6 and the database extension by typing:


```
sudo yum install lsphp56 lsphp56-mysql


```


To see all of the extensions available for version 5.6 of the OpenLiteSpeed PHP build, use yum to search:


```
yum search lsphp56


```


If you wish to install all of the extensions for version 5.6 of OpenLiteSpeed’s PHP, you can type:


```
sudo yum install lsphp56-* --skip-broken


```


With all of our components installed, we can now take care of some configuration.


# Change the Default Admin Password for OpenLiteSpeed


First, we should change the default administration password for OpenLiteSpeed.  By default, this is set to “123456”, so we should modify this value immediately.


To change the password, execute the following script:


```
sudo /usr/local/lsws/admin/misc/admpass.sh


```


You can optionally select a username for the administrative account, or just press ENTER to accept the default value of “admin”.  Afterwards, you will have to supply and verify a password for the administrative user.  Make sure to select a strong password because the administrative login screen is open to the web by default.


# Link the New PHP Version


In the installation step, we installed version 5.6 of OpenLiteSpeed’s customized PHP processor.  However, we have not yet told the web server that this is the version of PHP we wish to use for normal operations.


We can enable version 5.6 by linking it into the location that OpenLiteSpeed calls when attempting to execute PHP code.  The file that is called is located at /usr/local/lsws/fcgi-bin/lsphp5.  Currently, that location is linked to lsphp in the same directory, which is the version of PHP installed by default by OpenLiteSpeed (5.3).


We can change the link to the version we installed by typing:


```
sudo ln -sf /usr/local/lsws/lsphp56/bin/lsphp /usr/local/lsws/fcgi-bin/lsphp5


```


The web server will now use OpenLiteSpeed’s PHP version 5.6 when processing PHP files.


# Start and Secure the MariaDB System


Next, we should start the MariaDB database system and do some simple configuration.


Start MariaDB by typing:


```
sudo systemctl start mariadb


```


Next, we’ll enable the service so that it automatically starts when our machine boots:


```
sudo systemctl enable mariadb


```


With MariaDB online, we can run a simple security script to set an administrative password and lock down some insecure defaults:


```
sudo mysql_secure_installation


```


First, it will ask you for the MariaDB root password.  Since we have not set one yet, just press ENTER to continue.  The very next step asks you to set a root password.  Select and confirm an administrative password for the database system.


For the remainder of the questions, you can just hit ENTER to accept the default suggestions.  This will revert some insecure settings on our database system.


# Test Out the Default Web Page and Admin Interface


The OpenLiteSpeed server should already be up and running.  If you need to start, stop, restart, or check the status of the server, use the standard service command with the lsws service name:


```
sudo service lsws status


```


In your web browser, you can check out OpenLiteSpeed’s default web page.  Navigate to your server’s domain name or IP address, followed by :8088 to specify the port:


```
http://server_domain_or_IP:8088

```


You will see a page the default OpenLiteSpeed web page, that looks like this:





If you click through the links, you should notice that many features are already installed and configured correctly.  For instance, an example CGI script is available, a customized PHP instance is up and running, custom error pages and authentication gates are configured.  Click around to explore a little.


When you are satisfied with the default site, we can move on to the administrative interface.  In your web browser, using HTTPS, navigate to your server’s domain name or IP address followed by :7080 to specify the port:


```
https://server_domain_or_IP:7080

```


You will likely see a page warning your that the SSL certificate from the server cannot be validated.  Since this is a self-signed certificate, this is expected.  Click through the options available to proceed to the site (in Chrome, you must click “Advanced” and then “Proceed to…”).


You will be prompted to enter the administrative name and password that you selected with the admpass.sh script a moment ago:





Once you correctly authenticate, you will be presented with the OpenLiteSpeed administration interface:





This is where the majority of your configuration for the web server will take place.


## Change the Port for the Default Page


To demonstrate the basic idea behind configuring options through the web interface, we will change the port that the default site is using from “8088” to the conventional port 80.


To accomplish this, you can use the “Configuration” menu item in the menu bar and select “Listeners”:





In the list of listeners, you can click the “View/Edit” button for the “Default” listener:





You can click the edit button in the top-right corner of the “Address Settings” table to modify its values:





On the next screen, change port “8088” to port “80” and click “Save”.


After the modification, you will need to restart the server, which can be accomplished through the “Actions” menu by selecting “Graceful Restart”:





The default web page should now be accessible in your browser on port “80” instead of port “8088”.  Visiting your server’s domain name or IP address without providing a port will now display the site.


## Information about Configuring OpenLiteSpeed


OpenLiteSpeed is a fully-featured web server that is primarily managed through the administrative web interface.  A full run through of how to configure your site through this interface is outside of the scope of this guide.


However, to get you started, we’ll touch on a few important points below:


- Everything associated with OpenLiteSpeed will be found under the /usr/local/lsws directory.
- The document root (where your files will be served from) for the default virtual host is located at /usr/local/lsws/DEFAULT/html.  The configuration and logs for this virtual host can be found under the /usr/local/lsws/DEFAULT directory.
- You can create new virtual hosts for different sites using the admin interface.  However, all of the directories that you will reference when setting up your configuration must be created ahead of time on your server.  OpenLiteSpeed will not create the directories by itself.
- You can set up virtual host templates for virtual hosts that share the same general format.
- Often, it is easiest to copy the default virtual host’s directory structure and configuration to use as a jumping off point for new configurations.
- The admin interface has a built-in tool tip help system for almost all fields.  There is also a “Help” menu option in the menu bar that links to the server documentation.  Consult these sources of information during configuration if you need more information.
- After modifying the configuration and doing a graceful restart, always click the “Home” button to see if any error messages were reported at the bottom of the status screen.  You can see the full error logs by clicking “Actions > Server Log Viewer”.
- If the PHP versions included in the OpenLiteSpeed repository do not suit the needs of your application, you can compile PHP yourself with the help of the admin interface.  You may need to install additional development libraries through yum depending on the PHP options you intend to use.  You can get started by going to “Actions > Compile PHP” to select options.  The interface will walk you through the process.

# Conclusion


At this point, you should have OpenLiteSpeed, a customized version of PHP, and MariaDB installed and running on a CentOS 7 server.  OpenLiteSpeed offers great performance, an easy-to-use interface, and pre-configured options for script handling.  Dive in and learn the ropes to start leveraging these capabilities to serve your sites.


