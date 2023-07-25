# How to Install WordPress with LEMP on Ubuntu 20 04

```Ubuntu``` ```WordPress``` ```LEMP``` ```Nginx```

## Introduction


WordPress, one of the most popular content management systems (CMS) on the internet currently, allows users to set up flexible blogs and websites using a MySQL backend with PHP processing. WordPress has seen an incredible adoption rate among new and experienced engineers alike, and is a great choice for getting a website up and running efficiently. After an initial setup, almost all administration for WordPress websites can be done through its graphical interface— these features and more make WordPress a great choice for websites built to scale.


In this tutorial, you’ll focus on getting an instance of WordPress set up on a LEMP stack (Linux, Nginx, MySQL, and PHP) for an Ubuntu 20.04 server.


# Prerequisites


In order to complete this tutorial, you’ll need access to an Ubuntu 20.04 server. To successfully install WordPress with LEMP on your server, you’ll also need to perform the following tasks before starting this tutorial:


- Create a sudo user on your server: The steps in this tutorial are using a non-root user with sudo privileges.  You can create a user with sudo privileges by following our Ubuntu 20.04 initial server setup tutorial.
- Install a LEMP stack: WordPress will need a web server, a database, and PHP in order to correctly function.  Setting up a LEMP stack (Linux, Nginx, MySQL, and PHP) fulfills all of these requirements.  Follow this tutorial to install and configure this software.


Rather than setting up these components yourself, you can quickly provision Ubuntu 20.04 server that already has a LEMP stack installed with DigitalOcean’s LEMP 1-click install app.
Be aware, though, that this tutorial still assumes you have an administrative sudo user and an Nginx server block configured on your server. Even with a server provisioned with the LEMP 1-click app, you’ll need to follow Steps 1, 2, 3, and 5 of our Ubuntu 20.04 initial server setup tutorial. You’ll also need to complete Step 4 of our guide on installing the LEMP Stack on Ubuntu 20.04 to configure an Nginx server block and configure Nginx to use the PHP Processor

- Secure your site with SSL: WordPress serves dynamic content and handles user authentication and authorization.  TLS/SSL is the technology that allows you to encrypt the traffic from your site so that your connection is secure.  The way you set up SSL will depend on whether you have a domain name for your site.

If you have a domain name, the easiest way to secure your site is with Let’s Encrypt, which provides free, trusted certificates.  Follow our Let’s Encrypt guide for Nginx to set this up.
If you do not have a domain and you’re using this configuration for testing or personal use, you can use a self-signed certificate instead.  This provides the same type of encryption, but without the domain validation.  Follow our self-signed SSL guide for Nginx to get set up.


- If you have a domain name, the easiest way to secure your site is with Let’s Encrypt, which provides free, trusted certificates.  Follow our Let’s Encrypt guide for Nginx to set this up.
- If you do not have a domain and you’re using this configuration for testing or personal use, you can use a self-signed certificate instead.  This provides the same type of encryption, but without the domain validation.  Follow our self-signed SSL guide for Nginx to get set up.

When you are finished with setup, log in to your server as the sudo user to continue.


# Step 1 — Creating a MySQL Database and User for WordPress


WordPress uses MySQL to manage and store site and user information. Although you already have MySQL installed, let’s create a database and a user for WordPress to use.


To get started, log in to the MySQL root (administrative) account.  If MySQL is configured to use the auth_socket authentication plugin (which is default), you can log in to the MySQL administrative account using sudo:


```
sudo mysql


```


If you have changed the authentication method to use a password for the MySQL root account, use the following command instead:


```
mysql -u root -p


```


You will be prompted for the password you set for the MySQL root account.


Once logged in, create a separate database that WordPress can control. You can call this whatever you would like, but we will be using wordpress in this guide to keep it simple. You can create a database for WordPress by entering:


```
CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;


```



Note: Every MySQL statement must end in a semi-colon (;).  If you’ve encountered an error, check to make sure the semicolon is present.

Next, let’s create a separate MySQL user account that we will use exclusively to operate on our new database. Creating single-purpose databases and accounts is a good idea from a management and security standpoint. We’ll use the name wordpressuser in this guide — feel free to change this if you’d like.


In the following command, you are going to create an account, set a password, and grant access to the database you created. Remember to choose a strong password here:


```

CREATE USER 'wordpressuser'@'localhost' IDENTIFIED BY 'password';

GRANT ALL ON wordpress.* TO 'wordpressuser'@'localhost';


```


You now have a database and user account, each made specifically for WordPress.


With the database tasks complete, let’s exit out of MySQL by typing:


```
EXIT;


```


The MySQL session will exit, returning you to the regular Linux shell.


# Step 2 — Installing Additional PHP Extensions


When setting up the LEMP stack, it required a very minimal set of extensions to get PHP to communicate with MySQL. WordPress and many of its plugins leverage additional PHP extensions, and you’ll use a few more in this tutorial.


Let’s download and install some of the most popular PHP extensions for use with WordPress by typing:


```
sudo apt update


```


```
sudo apt install php-curl php-gd php-intl php-mbstring php-soap php-xml php-xmlrpc php-zip


```



Note: Each WordPress plugin has its own set of requirements.  Some may require additional PHP extension packages to be installed.  Check your plugin documentation to discover its PHP requirements.  If they are available, they can be installed with apt as demonstrated above.

When you are finished installing the extensions, restart the PHP-FPM process so that the running PHP processor can leverage the newly installed features:


```
sudo systemctl restart php7.4-fpm


```


You now have all of the PHP extensions needed, installed on the server.


# Step 3 — Configuring Nginx


Next, let’s make a few adjustments to our Nginx server block files. Based on the prerequisite tutorials, you should have a configuration file for your site in the /etc/nginx/sites-available/ directory configured to respond to your server’s domain name or IP address and protected by a TLS/SSL certificate.  We’ll use /etc/nginx/sites-available/wordpress as an example here, but you should substitute the path to your configuration file where appropriate.


Additionally, we will use /var/www/wordpress as the root directory of our WordPress install in this guide.  Again, you should use the web root specified in your own configuration.



Note: It’s possible you are using the /etc/nginx/sites-available/default default configuration (with /var/www/html as your web root).  This is fine to use if you’re only going to host one website on this server.  If not, it’s best to split the necessary configuration into logical chunks, one file per site.

Open your site’s server block file with sudo privileges to begin:


```
sudo nano /etc/nginx/sites-available/wordpress


```


Within the main server block, let’s add a few location blocks.


Start by creating exact-matching location blocks for requests to /favicon.ico and /robots.txt, both of which you do not want to log requests for.


Use a regular expression location to match any requests for static files.  We will again turn off the logging for these requests and will mark them as highly cacheable, since these are typically expensive resources to serve. You can adjust this static files list to contain any other file extensions your site may use:


/etc/nginx/sites-available/wordpress
```
server {
    . . .

    location = /favicon.ico { log_not_found off; access_log off; }
    location = /robots.txt { log_not_found off; access_log off; allow all; }
    location ~* \.(css|gif|ico|jpeg|jpg|js|png)$ {
        expires max;
        log_not_found off;
    }
    . . .
}

```


Inside of the existing location / block, let’s adjust the try_files list. Comment out the default setting by prepending the line with a pound sign (#) and then add the highlighted line. This way,  instead of returning a 404 error as the default option, control is passed to the  index.php file with the request arguments.


This should look something like this:


/etc/nginx/sites-available/wordpress
```
server {
    . . .
    location / {
        #try_files $uri $uri/ =404;
        try_files $uri $uri/ /index.php$is_args$args;
    }
    . . .
}

```


When you are finished, save and close the file.


Now, let’s check our configuration for syntax errors by typing:


```
sudo nginx -t


```


If no errors were reported, reload Nginx by typing:


```
sudo systemctl reload nginx


```


Next, let’s download and set up WordPress.


# Step 4 — Downloading WordPress


Now that your server software is configured, let’s download and set up WordPress. For security reasons, it is always recommended to get the latest version of WordPress directly from the project’s website.


Change into a writable directory and then download the compressed release by typing:


```
cd /tmp


```


This changes your directory to the temporary folder. Then, enter the following command to download the latest version of WordPress in a compressed file:


```
curl -LO https://wordpress.org/latest.tar.gz


```



Note:  The -LO flag is used to get directly to the source of the compressed file. -L ensures that fetching the file is successful in the case of redirects, and -O writes the output of our remote file with a local file that has the same name. To learn more about curl commands, visit How to Download Files with cURL

Extract the compressed file to create the WordPress directory structure:


```
tar xzvf latest.tar.gz


```


You will be moving these files into our document root momentarily, but before you do, let’s copy over the sample configuration file to the filename that WordPress actually reads:


```
cp /tmp/wordpress/wp-config-sample.php /tmp/wordpress/wp-config.php


```


Now, let’s copy the entire contents of the directory into our document root.  We’re using the -a flag to make sure our permissions are maintained, and a dot at the end of our source directory to indicate that everything within the directory should be copied (including hidden files):


```
sudo cp -a /tmp/wordpress/. /var/www/wordpress


```


Now that our files are in place, you’ll assign ownership to the www-data user and group.  This is the user and group that Nginx runs as, and Nginx will need to be able to read and write WordPress files in order to serve the website and perform automatic updates:


```
sudo chown -R www-data:www-data /var/www/wordpress


```


Files are now in the server’s document root and have the correct ownership, but you still need to complete some additional configuration.


# Step 5 — Setting up the WordPress Configuration File


Next, let’s make some changes to the main WordPress configuration file.


When you open the file, you’ll start by adjusting some secret keys to provide some security for our installation. WordPress provides a secure generator for these values so that you don’t have to come up with values on your own. These are only used internally, so it won’t hurt usability to have complex, secure values here.


To grab secure values from the WordPress secret key generator, type:


```
curl -s https://api.wordpress.org/secret-key/1.1/salt/


```


You will get back unique values that look something like this:



Warning: It is important that you request unique values each time.  Do NOT copy the values shown below!

```
Outputdefine('AUTH_KEY',         '1jl/vqfs<XhdXoAPz9 DO NOT COPY THESE VALUES c_j{iwqD^<+c9.k<J@4H');
define('SECURE_AUTH_KEY',  'E2N-h2]Dcvp+aS/p7X DO NOT COPY THESE VALUES {Ka(f;rv?Pxf})CgLi-3');
define('LOGGED_IN_KEY',    'W(50,{W^,OPB%PB<JF DO NOT COPY THESE VALUES 2;y&,2m%3]R6DUth[;88');
define('NONCE_KEY',        'll,4UC)7ua+8<!4VM+ DO NOT COPY THESE VALUES #`DXF+[$atzM7 o^-C7g');
define('AUTH_SALT',        'koMrurzOA+|L_lG}kf DO NOT COPY THESE VALUES  07VC*Lj*lD&?3w!BT#-');
define('SECURE_AUTH_SALT', 'p32*p,]z%LZ+pAu:VY DO NOT COPY THESE VALUES C-?y+K0DK_+F|0h{!_xY');
define('LOGGED_IN_SALT',   'i^/G2W7!-1H2OQ+t$3 DO NOT COPY THESE VALUES t6**bRVFSD[Hi])-qS`|');
define('NONCE_SALT',       'Q6]U:K?j4L%Z]}h^q7 DO NOT COPY THESE VALUES 1% ^qUswWgn+6&xqHN&%');

```


These are configuration lines that you can paste directly in your configuration file to set secure keys.  Copy the output you received now.


Now, open the WordPress configuration file:


```
sudo nano /var/www/wordpress/wp-config.php


```


Find the section that contains the dummy values for those settings.  It will look something like this:


/var/www/wordpress/wp-config.php
```
. . .

define('AUTH_KEY',         'put your unique phrase here');
define('SECURE_AUTH_KEY',  'put your unique phrase here');
define('LOGGED_IN_KEY',    'put your unique phrase here');
define('NONCE_KEY',        'put your unique phrase here');
define('AUTH_SALT',        'put your unique phrase here');
define('SECURE_AUTH_SALT', 'put your unique phrase here');
define('LOGGED_IN_SALT',   'put your unique phrase here');
define('NONCE_SALT',       'put your unique phrase here');

. . .

```


Delete those lines and paste in the values you copied from the command line:


/var/www/wordpress/wp-config.php
```
. . .

define('AUTH_KEY',         'VALUES COPIED FROM THE COMMAND LINE');
define('SECURE_AUTH_KEY',  'VALUES COPIED FROM THE COMMAND LINE');
define('LOGGED_IN_KEY',    'VALUES COPIED FROM THE COMMAND LINE');
define('NONCE_KEY',        'VALUES COPIED FROM THE COMMAND LINE');
define('AUTH_SALT',        'VALUES COPIED FROM THE COMMAND LINE');
define('SECURE_AUTH_SALT', 'VALUES COPIED FROM THE COMMAND LINE');
define('LOGGED_IN_SALT',   'VALUES COPIED FROM THE COMMAND LINE');
define('NONCE_SALT',       'VALUES COPIED FROM THE COMMAND LINE');

. . .

```


Next, let’s modify some of the database connection settings at the beginning of the file. You’ll have to adjust the database name, the database user, and the associated password that was configured within MySQL.


The other change you should make is to set the method that WordPress uses to write to the filesystem.  Since you’ve given the web server permission to write where it needs to, you can explicitly set the filesystem method to “direct”. Failure to set this with our current settings would result in WordPress prompting for FTP credentials when we perform some actions.  Add this setting below the database connection settings, or anywhere else in the file:


/var/www/wordpress/wp-config.php
```
. . .

define( 'DB_NAME', 'wordpress' );

/** MySQL database username */
define( 'DB_USER', 'wordpressuser' );

/** MySQL database password */
define( 'DB_PASSWORD', 'password' );

. . .

define( 'FS_METHOD', 'direct' );

```


Save and close the file when you’re done.


# Step 6 — Completing the Installation Through the Web Interface


Now that the server configuration is complete, you can finish up the installation through WordPress’ web interface.


In your web browser, navigate to your server’s domain name or public IP address:


```
http://server_domain_or_IP/wordpress

```


Select the language you would like to use:





Next, you will come to the main setup page.


Select a name for your WordPress site and choose a username (it is recommended not to choose something like “admin” for security purposes).  A strong password is generated automatically.  Save this password or select an alternative strong password.


Enter your email address and select whether you want to discourage search engines from indexing your site:





When you click ahead, you will be taken to a page that prompts you to log in:





Once you log in, you will be taken to the WordPress administration dashboard:





# Conclusion


WordPress should be installed and ready to use!  Some common next steps are to choose the permalinks setting for your posts (can be found in Settings > Permalinks) or to select a new theme (in Appearance > Themes).  If this is your first time using WordPress, explore the interface a bit to get acquainted with your new CMS.



If you’re looking for a one-click solution to install a WordPress Droplet, learn more about the WordPress One-Click App.

