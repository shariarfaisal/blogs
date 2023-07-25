# Installing phpMyAdmin on Ajenti V

```MySQL``` ```Ubuntu``` ```Control Panels``` ```Arch Linux``` ```CentOS``` ```CoreOS``` ```Debian```

## Introduction


phpMyAdmin is a free, web-facing control panel that can access and edit MySQL databases hosted on your server. It integrates with all existing and future databases on your Ajenti server automatically.





This tutorial will show you integrate phpMyAdmin into the Ajenti control panel.


## Prerequisites


- Ajenti with Ajenti V installed (read How To Install The Ajenti Control Panel and Ajenti V on Ubuntu 14.04)
- A registered domain name that points to your Droplet (example.com is used throughout this tutorial)
- A subdomain (phpmyadmin.example.com) that resolves to your Droplet (follow the directions for setting up an A record in How To Set Up a Host Name with DigitalOcean)

# Step 1 — Creating the Website in Ajenti V


Before installing phpMyAdmin, you need to create the website for it in Ajenti.


In your browser, browse to your Ajenti control panel at https://panel.example.com:8000 (replacing example.com with your domain name), and log in. In the sidebar to the right, under the Web section, click Websites.


Under the New Website section there is a Name text field, type phpMyAdmin and click the Create button. Under the Websites section on that same page, click Manage next to the new phpMyAdmin line. On the page that appears, uncheck the box next to Maintenance mode. In the Website Files section below that, change Path from /srv/new-website to /srv/phpMyAdmin. Press the Set button next to that text field. Click Apply Changes at the bottom of the screen.


At the top of that page, click the Domains tab. Click Add and replace example.com with phpmyadmin.your_domain_name. Click Apply Changes at the bottom of the screen.


Now click the Content tab. Change the dropdown box to PHP FastCGI and click Create. This basically tells Ajenti to enable PHP for this website.


Click Apply Changes at the bottom of the screen. Configuration should now be complete. You should now be able to install phpMyAdmin.


# Step 2 — Installing phpMyAdmin


To install phpMyAdmin, first browse to www.phpmyadmin.net/downloads and download the latest version of phpMyAdmin in a .zip file format, shown highlighted in blue in the screenshot below.





Back in the Ajenti control panel, browse to File Manager in the sidebar. Use the File Manager to browse to /srv. At the bottom of the page there is an Upload section. Click the Choose File button, and select the phpMyAdmin zip file you downloaded. After it finishes uploading, click the newly added file, named something like phpMyAdmin-x.x.xx-all-languages.zip. In the modal box that appears, click the Unpack button (highlighted in blue in the screenshot below).





When it finishes, click the X button next to Terminal 0 at the top of the screen. You should be taken back to the File Manager, and there should be a folder named something similar to phpMyAdmin-x.x.xx-all-languages. Click the menu button to the right of that folder.





In the Name text box, rename the folder to phpMyAdmin, and click Save.


# Step 3 — Setting up a Database


We’re going to cover setting up a database in Ajenti, so you can login to phpMyAdmin. If you already have a database and login, feel free to skip ahead to step 4.


In the Ajenti control panel, in the sidebar there is a section called Software. Directly below that is a menu option named MySQL. Click that to access the MySQL control panel in Ajenti.





Under the Databases section, click the Create button. There you will be able to enter a name for your database. This can be anything you’d like. It won’t be seen by anybody but you.


The rest of the sections may automatically populate with default databases and users. This is fine, and you don’t need to touch them. Under the Users section, click the Create button. The Username and Password fields can be whatever you want, this is what you will login to phpMyAdmin with, so be sure to remember it or write it down. In the Hostname field, enter localhost.


That should be it, now we can start using phpMyAdmin.


# Step 4 — Logging in


To access the phpMyAdmin web interface, browse to phpmyadmin.your_domain_name in your web browser. For the Username and Password fields, enter the credentials of your MySQL user, and press Go.


# Conclusion


Congratulations! You now have phpMyAdmin installed on your server.


