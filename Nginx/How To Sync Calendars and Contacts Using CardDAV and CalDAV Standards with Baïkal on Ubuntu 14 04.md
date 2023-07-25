# How To Sync Calendars and Contacts Using CardDAV and CalDAV Standards with Baïkal on Ubuntu 14 04

```Apache``` ```Ubuntu``` ```Control Panels``` ```Nginx```

## Introduction


With more and more people using multiple devices (smartphones, computers, tablets, etc.) the need to keep every thing in sync keeps growing.


While syncing files is important, it’s also useful to be able to sync calendars and contacts in their native formats.


The CalDAV and CardDAV standards provide an easy way to keep all our smart things up-to-date with what we are doing, as well as how to get hold of our friends and other contacts. In this tutorial, we’ll show you how to sync calendars and contacts from a server you control, using a super simple installation of Baïkal, a PHP CalDAV and CardDAV server.



Note: If you are looking for an all-in-one solution, you may want to take a look at ownCloud instead.


Note: Baïkal is quick and easy but not really designed for large scale deployment. If you want calendar and contact syncing for a medium or large business, this solution may not work well for you.

## Prerequisites


Please make sure you have these prerequisites in place.


- Fresh Ubuntu 14.04 Droplet with SSH access
- A sudo user
- The Baïkal instructions highly recommend having a domain, preferably a subdomain, for the server. This tutorial will use the domain name dav.example.com. You can use dav.yourdomain.com. If you host your DNS with Digital Ocean, this article can help you set up that subdomain

We’ll also be installing some packages that Baïkal needs; we’ll be using an SSL certificate; and we’ll go over setting those up in the article itself. If you want to purchase an SSL certificate, you should purchase it for your Baïkal server’s domain or subdomain.


# Step 1 — Installing Baïkal


To get started, we’ll install some required packages, download the tarball of Baïkal, and then extract it.


In the examples below we’re using the latest version of Baïkal, which at the time of writing is 0.2.7, but we recommend double-checking the latest version of Baïkal before you get started. To find the latest version, go to the Baïkal site, and either click on the Download to get started button, or scroll down to the Get Baïkal section. If there is a newer version, copy the download link for the Regular package.


To get started you’ll need to SSH into your Ubuntu Droplet.


Now let’s install the packages that Baïkal will need to run. We’ll assume this is a fresh Ubuntu installation, so before we can install some packages from the repos we need to update the repo cache with apt-get update.


```
sudo apt-get update

```


Install some prerequisite packages: PHP, Apache, and SQLite.


```
sudo apt-get install apache2 php5 php5-sqlite sqlite3

```



Note: In the Baïkal installation file, the author notes that Apache can be replaced with Nginx and SQLite can be replaced with MySQL.

Now that we have the required pieces to get Baïkal working, lets get Baïkal installed! Since Baïkal is a PHP website of sorts, we’re going to download and extract it in the Apache site directory, /var/www.


```
cd /var/www
sudo wget http://baikal-server.com/get/baikal-regular-0.2.7.tgz
sudo tar -xvzf baikal-regular-0.2.7.tgz

```



Note: For those who’d like to know what we just told tar to do: x = extract, v = verbose, z = unzip, and f = file, followed by the file name.

One last step and Baïkal will be installed. Since we’ve extracted the PHP app we don’t need the tar file any more, so we’ll remove that, rename the extracted folder to something more relevant, and then make sure it is readable and writeable by the Apache user.


```
sudo rm baikal-regular-0.2.7.tgz
sudo mv baikal-regular dav.example.com
sudo chown -R www-data:www-data dav.example.com

```



Note: You can name the folder whatever you want, but it’s much easier to identify sites, if you intend to host several, if you use the website name for the website folder.

# Step 2 — Setting Up Apache


Our application is installed, and now we need to tell Apache about it. To make things easy, Baïkal actually includes its own Apache configuration file as a template. We’ll copy that file to the Apache sites-available directory and then edit it to fit our site.


```
sudo cp /var/www/dav.example.com/Specific/virtualhosts/baikal.apache2 /etc/apache2/sites-available/dav_example_com.conf

```


Using your favorite text editor, open up the dav_example_com.conf file and change all of the URLs to use your own URL, and the paths to where you stored your site. Here is what it’ll look like:


```
sudo nano /etc/apache2/sites-available/dav_example_com.conf

```


```
<VirtualHost *:80>
	DocumentRoot /var/www/dav.example.com/html
	ServerName dav.example.com

	RewriteEngine On
	RewriteRule /.well-known/carddav /card.php [R,L]
	RewriteRule /.well-known/caldav /cal.php [R,L]

	<Directory "/var/www/dav.example.com/html">
		Options None
		Options +FollowSymlinks
		AllowOverride All
	</Directory>
</VirtualHost>

```


Now we’ll need an SSL certificate.


You can either create or purchase your certificate. We’ll assume that you followed the linked SSL tutorial, and that your key and certificate are in the /etc/apache2/ssl directory and called apache.crt and apache.key. Please replace these with the paths to your own certificate and key as appropriate.


Now we need to tell Apache how to use the SSL certificate. For this we need to combine the default SSL config file (default-ssl.conf) with our Baïkal config file, and name it dav_example_com-ssl.conf. Below is an example of what that will look like, with all the comments taken out.


```
sudo nano /etc/apache2/sites-available/dav_example_com-ssl.conf

```


```
<IfModule mod_ssl.c>
	<VirtualHost _default_:443>
		ServerAdmin webmaster@localhost

		DocumentRoot /var/www/dav.example.com/html
		ServerName dav.example.com

        	RewriteEngine On
        	RewriteRule /.well-known/carddav /card.php [R,L]
        	RewriteRule /.well-known/caldav /cal.php [R,L]

		<Directory "/var/www/dav.example.com/html">
			Options None
			Options +FollowSymlinks
			AllowOverride All
		</Directory>

		ErrorLog ${APACHE_LOG_DIR}/error.log
		CustomLog ${APACHE_LOG_DIR}/access.log combined

		SSLEngine on

		SSLCertificateFile	  /etc/apache2/ssl/apache.crt
		SSLCertificateKeyFile /etc/apache2/ssl/apache.key

		<FilesMatch "\.(cgi|shtml|phtml|php)$">
				SSLOptions +StdEnvVars
		</FilesMatch>
		<Directory /usr/lib/cgi-bin>
				SSLOptions +StdEnvVars
		</Directory>

		BrowserMatch "MSIE [2-6]" \
				nokeepalive ssl-unclean-shutdown \
				downgrade-1.0 force-response-1.0
		BrowserMatch "MSIE [17-9]" ssl-unclean-shutdown

	</VirtualHost>
</IfModule>

```


We are on the home stretch. We have the site installed and the appropriate Apache configs created. Now we need to tell Apache to enable the rewrite module, enable the sites, and then finally restart to get the new settings loaded.


```
sudo a2enmod rewrite
sudo a2ensite dav_example_com
sudo a2ensite dav_example_com-ssl
sudo service apache2 restart

```


# Step 3 — Configuring Baïkal


We have one last thing to do on the command line and the rest can be done in a web browser. Baïkal uses a file called ENABLE_INSTALL to enable the final step of the installation. Before we open up the web browser, let’s make sure that this file exists. We’ll use touch to create the file if it isn’t there, and if it’s already there, all we do is update the modification date.


```
sudo touch /var/www/dav.example.com/Specific/ENABLE_INSTALL

```


That’s it! We are ready to open a browser and finish the setup of Baïkal. In your favorite browser navigate to https://dav.example.com.





Once you’re there, you will be presented with a screen with options. Set your time zone using the dropdown menu, create a new admin password (you’ll have to enter it twice), and leave everything else with the default settings.


Click the Save changes button.


On the next screen you can choose the default SQLite settings or enable MySQL support.





If you chose to use MySQL, you can enable that support. (Using MySQL as a backend will give this tool a greater capacity and increased performance, but if this DAV server is just for you, your family and friends, or a small business, SQLite should do just fine.)


For this example, we’ll leave the SQLite defaults enabled, and click the Save changes button on this page, too.


Then you’ll see the option to Start using Baïkal; click this button.





You’ll be taken to the Baïkal home page.



Note: If you see the default Apache website instead of your Baïkal website, you need to disable the default Apache website and restart Apache. Things should start working now.
sudo a2dissite 000-default.conf
sudo service apache2 reload


# Step 4 — Creating a User


After running through the initial setup, all that is left is creating a user, and then connecting your clients to start syncing.


To create a user, log in to the Baïkal website using the username admin and the password you set during the configuration step above.


The first page of the application is the Dashboard. It shows you what is enabled and running and some basic stats, like the number of users, calendars, and contacts.


Creating a user is a three click process.


1. At the top of the page, click on the link Users and resources
2. Now click on the button on the right, + Add user
3. Fill out all the fields and then click the Save changes button





Note: There aren’t any requirements on the server side for the formatting of the username, but some clients may complain if the username doesn’t look like an email address, like: sammy@example.com

# Troubleshooting


If you run into any problems, such as your admin password not being accepted, then there are a few commands you can run to reset the app, allowing you to set it up again. To do so, you’ll need to SSH back into your Droplet to run the following commands.


Don’t do this unless you want to reset the server.


```
cd /var/www/dav.example.com/Specific/
sudo rm config*.php
sudo touch ENABLE_INSTALL

```


Now you can jump back into the web browser and go through the app setup wizard again, and hopefully this time everything works.


## Conclusion


Congratulations! You’ve installed a CalDAV and CardDAV syncing server with a GUI control panel. At this point you can configure your clients to connect to server. When doing so, use https://dav.example.com as your host name.


