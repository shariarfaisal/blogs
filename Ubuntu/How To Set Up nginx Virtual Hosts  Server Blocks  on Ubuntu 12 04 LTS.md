# How To Set Up nginx Virtual Hosts  Server Blocks  on Ubuntu 12 04 LTS

```Ubuntu``` ```Nginx```











# Status: Deprecated


This article covers a version of Ubuntu that is no longer supported.  If you are currently operate a server running Ubuntu 12.04, we highly recommend upgrading or migrating to a supported version of Ubuntu:


- Upgrade to Ubuntu 14.04.
- Upgrade from Ubuntu 14.04 to Ubuntu 16.04
- Migrate the server data to a supported version

Reason:
Ubuntu 12.04 reached end of life (EOL) on April 28, 2017 and no longer receives security patches or updates.  This guide is no longer maintained.


See Instead:
This guide might still be useful as a reference, but may not work on other Ubuntu releases.  If available, we strongly recommend using a guide written for the version of Ubuntu you are using.   You can use the search functionality at the top of the page to find a more recent version.


## About Virtual Hosts


Virtual Hosts are used to run more than one website or domain off of a single server. Note: according to the nginx website, virtual hosts are called Server Blocks on the nginx. However, for easy comparison with apache, I'll refer to them as virtual hosts in this tutorial.


# Set Up


The  steps in this tutorial require the user to have root privileges on the virtual private server. You can see how to set that up in the Initial Server Setup Tutorial in steps 3 and 4. Furthermore, if I reference the user in a step, I’ll use the name www. You can implement whatever username suits you.


Additionally, you need to have nginx already installed on your VPS.  If this is not the case, you can download it with this command:


```
sudo apt-get install nginx
```


# Step One— Create a New Directory


The first step in creating a virtual host is to a create a directory where we will keep the new website’s information.


This location will be your Document Root in the nginx virtual configuration file later on. By adding a -p to the line of code, the command automatically generates all the parents for the new directory.


```
sudo mkdir -p /var/www/example.com/public_html
```


You will need to designate an actual DNS approved domain, or an IP address, to test that a virtual host is working. In this tutorial we will use example.com as a placeholder for a correct domain name.


However, should you want to use an unapproved domain name to test the process you will find information on how to make it work on your local computer in Step Six.


# Step Two—Grant Permissions


We need to grant ownership of the directory to the right user, instead of just keeping it on the root system. You can replace the "www-data" below with the appropriate username.


```
sudo chown -R www-data:www-data /var/www/example.com/public_html
```


Additionally, it is important to make sure that everyone is able to read our new files.


```
sudo chmod 755 /var/www
```


Now you are all done with permissions.


# Step Three— Create the Page


We need to create a new file called index.html within the directory we made earlier.


```
sudo nano /var/www/example.com/public_html/index.html
```


We can add some text to the file so we will have something to look at when the the site redirects to the virtual host.


```
<html>
  <head>
    <title>www.example.com</title>
  </head>
  <body>
    <h1>Success: You Have Set Up a Virtual Host</h1>
  </body>
</html>
```


Save and Exit


# Step Four—Create the New Virtual Host File


The next step is to create a new file that will contain all of our virtual host information.


nginx provides us with a layout for this file in the sites-available directory (/etc/nginx/sites-available), and we simply need to copy the text into a new custom file:


```
sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/example.com
```


# Step Five—Set Up the Virtual Hosts


Open up the new virtual host file— you will see all the information you need to set up virtual host within.


```
 sudo nano /etc/nginx/sites-available/example.com
```


We need to make a couple of changes in these few lines:


```
 server {
        listen   80; ## listen for ipv4; this line is default and implied
        #listen   [::]:80 default ipv6only=on; ## listen for ipv6

        root /var/www/example.com/public_html;
        index index.html index.htm;

        # Make site accessible from http://localhost/
        server_name example.com;
}
```


- Uncomment  "listen 80" so that all traffic coming in through that port will be directed toward the site
-  Change the root extension to match the directory that we made in Step One. If the document root is incorrect or absent you will not be able to set up the virtual host.
- Change the server name to your DNS approved domain name or, if you don't have one, you can use your IP address

You do not need to make any other changes to this file. Save and Exit.


The last step is to activate the host by creating a symbolic link between the sites-available directory and the sites-enabled directory. In apache, the command to accomplish this is "a2ensite"—nginx does not have an equivalent shortcut, but it's an easy command nonetheless.


```
sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/example.com
```


To both avoid the "conflicting server name error" and ensure that going to your site displays the correct information, you can delete the default nginx server block:


```
sudo rm /etc/nginx/sites-enabled/default
```


# Step Six—Restart nginx


We’ve made a lot of the changes to the configuration. Restart nginx and make the changes visible.


```
sudo service nginx restart
```


# Optional Step Seven—Setting Up the Local Hosts


If you have pointed your domain name to your server’s IP address you can skip this step—you do not need to set up local hosts. Your virtual hosts should work. However, if want to try out your new virtual hosts without having to connect to an actual domain name, you can set up local hosts on your computer alone. 
For this step, make sure you are on the computer itself, not your droplet.


To proceed with this step you need to know your computer’s administrative password, otherwise you will be required to use an actual domain name to test the virtual hosts.


If you are on a Mac or Linux, access the root user (su) on the computer and  open up your hosts file:


```
nano /etc/hosts 
```


If you are on a Windows Computer, you can find the directions to alter the host file on the Microsoft site


You can add the local hosts details to this file, as seen in the example below. As long as that line is there, directing your browser toward, say, example.com will give you all the virtual host details for the corresponding IP address.


```
# Host Database
#
# localhost is used to configure the loopback interface
# when the system is booting.  Do not change this entry.
##
127.0.0.1       localhost

#Virtual Hosts 
12.34.56.789    www.example.com 
```


However, it may be a good idea to delete these made up addresses out of the local hosts folder when you are done to avoid any future confusion. 


# Step Eight—RESULTS: See Your Virtual Host in Action


Once you have finished setting up your virtual host, you can see how it looks online. Type your domain name or ip address into the browser (ie. http://12.34.56.789) 



It should look somewhat similar to my handy screenshot


# Creating More Virtual Hosts


To add more virtual hosts, you can just repeat the process above, being careful to set up a new document root with the appropriate domain name, and then creating and activating the new virtual host file.


# See More


Once you have set up your virtual hosts, you can proceed to Create a SSL Certificate for your site or Install an FTP server


Etel Sverdlov
