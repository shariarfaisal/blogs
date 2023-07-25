# How To Install and Update WordPress with Version Control on CentOS 7

```WordPress``` ```Git``` ```CentOS```

# Introduction


There are many ways to install the WordPress content management system. This tutorial introduces two methods for installing WordPress from a public repository: SVN or Git.


While you can install WordPress in a few different ways, e.g. using a one-click image, downloading a zip file, or using the built-in FTP service – using a repository has some unique benefits.


- Quick upgrades and downgrades to different versions of WordPress
- More secure protocols for transferring the files
- Faster updates since only the changed files are transferred

What happens if you update WordPress to the latest version and your site goes down? With SVN or Git, you can easily roll back the file changes with one command. This is impossible with the FTP updater.


## SVN or Git?


SVN stands for Apache Subversion. The official WordPress repository uses SVN:


http://core.svn.wordpress.org/


The benefit of using SVN is that you’re getting the files directly from WordPress.


Git is a somewhat more modern repository protocol. The GitHub WordPress repository is maintained by a third party, and currently gets its files from WordPress’s SVN repository:


https://github.com/WordPress/WordPress


The benefit of using Git is its more sophisticated version control. However, keep in mind that this is run by a third-party repository maintainer.


You are free to choose which system works best in your situation.


## Prerequisites


Are you ready to get started? Good!


Let’s make sure you’ve got the necessary items:


- A 1 GB Droplet running CentOS 7 (you can adapt this guide for Debian-based distros fairly easily)
- root SSH access to your server; you could also use sudo

# SVN Instructions


Follow these instructions for SVN. Skip to the Git instructions instead if you’d rather use Git.


## SVN Step One — Install LAMP


Follow this tutorial to install Apache, MySQL, and PHP on your server:


How To Install Linux, Apache, MySQL, PHP (LAMP) stack On CentOS 7


You can stop after Step Three — Install PHP.


## SVN Step Two — Install SVN


Install SVN with the following command:


```
yum install svn

```


You’ll need to answer yes to the installation and let the process complete.


Now let’s test it. Enter the following command:


```
svn

```


You should see the following message:


```
Type 'svn help' for usage.

```


## SVN Step Three — Check out WordPress


When setting up a new WordPress installation, you should note the latest stable version. The best place for this is to visit the official WordPress website.


At the time of writing, this is WordPress 4.0, so that’s what we’ll use in the examples.


Decide where you want to install WordPress. In this example we’ll use the default Apache document root, /var/www/html. You may want to set up a virtual host instead.


Check out WordPress 4.0, or the latest version, right from WordPress’s repository:


```
svn co http://core.svn.wordpress.org/tags/4.0/ /var/www/html/

```


The general form of the command is as follows:


```
svn co http://core.svn.wordpress.org/tags/[VERSION]/ [INSTALL IN THIS DIRECTORY]/

```


You’ll see a bunch of file names flash by as your server talks to WordPress’s SVN server and grabs the files while noting the version numbers. The process should end with the message Checked out revision [some number].


Example:


```
Checked out revision 29726.

```


Congratulations! You’ve just installed WordPress using SVN. Now we need to set up the database and configure WordPress.


## SVN Step Four — Configure WordPress


Follow the instructions in this WordPress installation tutorial except for the wget, tar, and rsync commands.


You should set up the database, change the wp-config.php details, and run the chown command:


```
chown -R apache:apache /var/www/html/*

```


At this point WordPress is ready to use! Visit your IP address or domain in your browser, and set your website and login details as prompted. Set it up to your liking, including any themes and plugins.


## SVN Step Five — Secure the .svn Directory


SVN uses a special directory called .svn that contain important information. In the name of security, it is best to block access to this data so it can’t be viewed by the outside world using your web server.


If you want to see what it looks like now, visit http://example.com/.svn/ in your browser, using your own domain name. It shows all the administrative files for the repository - not good! Now we’ll fix this.


First, open your Apache configuration file for editing:


```
nano /etc/httpd/conf/httpd.conf

```


Locate the AllowOverride line in the <Directory “/var/www/html”> section. It should be the third AllowOverride line in the default configuration file. Update the setting from None to ALL. This will allow your .htaccess file to become active.


```
...
<Directory "/var/www/html">

...

    Options Indexes FollowSymLinks

...

    AllowOverride ALL

    #
    # Controls who can get stuff from this server.
    #
    Require all granted
</Directory>
...

```


Now create a new .htaccess file in the  /var/www/html/.svn/.htaccess directory:


```
nano /var/www/html/.svn/.htaccess

```


Add the following contents to the file:


```
order deny, allow
deny from all

```


Restart Apache:


```
service httpd restart

```


Now you, or anyone trying to snoop on your server, will get an Internal Server Error if they visit http://example.com/.svn/.


# SVN Step Six — Upgrade or Roll Back


New versions of WordPress will be released and you’ll want to quickly and easily update your installation to address security patches, fix bugs, and add new features. So let’s discuss how this is quickly and easily accomplished using SVN.


It’s always a good idea to make a backup.


Connect to your server with SSH, and move to your WordPress installation directory:


```
cd /var/www/html/

```


Execute this command to switch to a new version:


```
svn sw http://core.svn.wordpress.org/tags/[VERSION]/ .

```


[VERSION] is a placeholder for the actual number of the release.


The period (.) tells SVN where to check and install the files. Since we have changed to the directory containing the WordPress files, we simply used the period to tell SVN to look in the current directory. You could specify the path if you weren’t in the directory.


If the new version to be installed was 4.0.1, the command would be:


```
svn sw http://core.svn.wordpress.org/tags/4.0.1/ .

```


This is also the method for downgrading, too. So let’s say you want to return to version 3.9.2; you would do that with this command:


```
svn sw http://core.svn.wordpress.org/tags/3.9.2/ .

```


To see all the available options, check the WordPress SVN tags page.


That is how easy it is to upgrade and downgrade the core WordPress files using the SVN system. Your custom settings, like your wp-config.php file and your themes and plugins, should all stay in place. However, if you’ve modified any of the core files, you may run into problems. (That’s why you should have made a backup.)


Once you have the files, you need to let WordPress make the changes it needs in the database.


Visit http://example.com/wp-admin/


Click the Update WordPress Database button.


That’s it! You should now be on your desired version of WordPress. If your site isn’t working after the change, simply check out the version you had before.


# Git Instructions


Follow these instructions for Git. Scroll back up to the SVN instructions if you’d rather use SVN.


## Git Step One — Install LAMP


Follow this tutorial to install Apache, MySQL, and PHP on your server:


How To Install Linux, Apache, MySQL, PHP (LAMP) stack On CentOS 7


You can stop after Step Three — Install PHP.


## Git Step Two — Install Git


Install Git with the following command:


```
yum install git

```


You’ll need to answer yes to accept the download. Now let’s test it.  Enter the following command:


```
git

```


You should see the following message:


```
usage: git ...

```


## Git Step Three — Clone WordPress


First, figure out which version of WordPress you want to install. The best place for this is to visit the [official WordPress website] (http://www.wordpress.org).


At the time of writing, this is WordPress 4.0, so that’s what we’ll use in the examples.


Decide where you want to install WordPress. In this example we’ll use the default Apache document root, /var/www/html. If you want to set up a virtual host, you can do that instead.


Clone the latest version of WordPress from the GitHub repository:


```
git clone git://github.com/WordPress/WordPress /var/www/html/

```


The general form of the command is as follows:


```
git clone git://github.com/WordPress/WordPress [INSTALL IN THIS DIRECTORY]/

```


You’ll see some messages such as Cloning in… along with, but not limited to, Receiving objects: and Receiving deltas: with some information. You now have a complete working development copy of WordPress, including past production runs.


However, we want the latest production (stable) version. First move to the WordPress directory on your server:


```
cd /var/www/html/

```


Check out WordPress 4.0, or the latest stable version, with the following command:


```
git checkout 4.0

```


The general form of the command is as follows:


```
git checkout [VERSION]

```


Git will display some information along with something like HEAD is now at 8422210... Tag 4.0, which indicates the file versions were successfully changed; in this case to 4.0.


Congratulations! You’ve just installed WordPress using Git.


Now we need to set up the database and configure WordPress.


## Git Step Four — Configure WordPress


Follow the instructions in this WordPress installation tutorial, but without the wget, tar, and rsync commands.


You do need to set up the database, change the wp-config.php details, and run the chown command:


```
chown -R apache:apache /var/www/html/*

```


At this point WordPress is ready to use! Visit your IP address or domain in your browser, and set your website and login details as prompted. You can add themes, plugins, and content as you like.


## Git Step Five — Secure the .git Directory


Git uses a special directory called .git that contain important information. You should block web access to this directory for security’s sake.


If you want to see what it looks like now, visit http://example.com/.git/ in your browser, using your own domain name. It should list the files in the directory, which is a security issue.


First, open your Apache configuration file for editing:


```
nano /etc/httpd/conf/httpd.conf

```


Locate the AllowOverride line in the <Directory “/var/www/html”> section. It should be the third AllowOverride line in the default configuration file. Update the setting from None to ALL. This will allow your .htaccess file to become active.


```
...
<Directory "/var/www/html">

...

    Options Indexes FollowSymLinks

...

    AllowOverride ALL

    #
    # Controls who can get stuff from this server.
    #
    Require all granted
</Directory>
...

```


Now create a new .htaccess file in the /var/www/html/.git/.htaccess directory:


```
nano /var/www/html/.git/.htaccess

```


Add the following contents to the file:


```
order deny, allow
deny from all

```


Restart Apache:


```
service httpd restart

```


Now you, or anyone trying to snoop on your server, will get an Internal Server Error if they visit http://example.com/.git/.


## Git Step Six — Upgrade or Roll Back


Now it’s time to upgrade WordPress. You’ll want to keep up with security patches, bug fixes, and new features. So let’s discuss how to upgrade with Git.


It’s always a good idea to make a backup.


Connect to your server with SSH, and move to your WordPress installation directory:


```
cd /var/www/html/

```


Fetch the latest files from the third-party WordPress repository:


```
git fetch -p git://github.com/WordPress/WordPress

```


The -p switch tells Git to remove any old versions that are no longer in the repository. This helps keep your files in sync with the remote server.


Execute this command to check out a new version:


```
git checkout [VERSION]

```


[VERSION] is a placeholder for the actual number of the release. If the new version to be installed was 4.0.1, the command would be:


```
git checkout 4.0.1

```


This is also the method for downgrading, too. If you want to return to version 3.9.2; you would do that with this command:


```
git checkout 3.9.2

```


To see all the available options, check the branch dropdown and the Tags tab on the repository page.


That’s it! With Git, your custom settings, like your wp-config.php file and your themes and plugins, should stay the same. However, if you’ve modified any of the core files, you may run into problems; hence the need for a backup.


Once you have the files, you need to let WordPress make the changes it needs in the database.


Visit http://example.com/wp-admin/.


Click the Update WordPress Database button.


That’s it! You should now be on your desired version of WordPress. If your site isn’t working after the change, simply check out the version you had before.


# Conclusion


If you made it to the end of this tutorial you should have a basic understanding of setting up WordPress using the SVN and/or Git system(s). It is important to note that this method will back up the core WordPress system, but your custom themes and plugins will require a different approach.


Now that you have learned how to manage WordPress with version control, you’ll probably never want to go back. This is so much faster, easier, and safer. You don’t need to store any FTP information in your WordPress installation. Also, you can easily and quickly revert back to previous versions if the need arises, something that the FTP method makes more difficult.


This guide is not a replacement for a good backup system, so make sure you have good backups, too.


