# How To Create a Blog with Ghost and Nginx on Ubuntu 14 04

```Ubuntu``` ```Ghost``` ```Nginx```

## Introduction


Ghost is a lightweight (~7.5MB), open source blogging platform which is really easy to use. Ghost is fully customizable. There are loads of themes available for Ghost on the Internet, free as well as paid.


In this tutorial, we will go through the steps to get Ghost setup and running on your Ubuntu 14.04 system. We will also install Nginx to proxy ports and install forever, a node package, to keep Ghost running in the background.


# Prerequisites


There is no minimum size requirement for a server to run Ghost. Consider how many visitors your blog will get and how much content you plan to share when deciding what size Droplet to create. This tutorial was tested on the smallest size DigitalOcean Droplet running Ubuntu 14.04.


Before you begin, you need the following:


- Ubuntu 14.04 Droplet
- Registered domain name pointed to the IP address for your Droplet
- A non-root user with sudo privileges

This tutorial will help you setup your domain name to point to your Droplet.


All the commands in this tutorial should be run as a non-root user. If root access is required for the command, it will be preceded by sudo. Initial Server Setup with Ubuntu 14.04 explains how to add users and give them sudo access.


# Step 1 — Install Node.js and Npm


You need to update your local package index and install the zip and wget packages. We will use them later in this tutorial.


```
sudo apt-get update
sudo apt-get install zip wget


```


Ghost requires Node.js v0.10.x (latest stable). Unstable versions of Node, like v0.12.x, are not supported. Node.js v0.10.36 and npm v2.5.0 are recommended by Ghost.org.


Install Node.js with the PPA method from this tutorial.


Once you have Node.js installed, check the version installed by running:


```
node -v


```


The output should be similar to this:


```
v0.10.38

```


Check if npm installed:


```
npm -v


```


It should output the installed version of npm if it is installed:


```
1.4.28

```


If it outputs an error that npm is not installed, install it with this command:


```
sudo apt-get install npm


```


Update npm to version 2.5.0 by running the following command:


```
sudo npm install npm@2.5.0 -g


```


Check the version of npm installed:


```
npm -v


```


The output should be:


```
2.5.0

```


# Step 2 — Install Ghost


Next we need to install Ghost. Ghost.org recommends to install Ghost in var/www/ghost, so that is where we will install it.


First, we will create a directory /var/www/ and then download the latest version of Ghost from Ghost’s GitHub repository:


```
sudo mkdir -p /var/www/
cd /var/www/
sudo wget https://ghost.org/zip/ghost-latest.zip


```


Now that we have obtained the latest version of Ghost, we have to unzip it. We’ll also change our directory to /var/www/ghost/:


```
sudo unzip -d ghost ghost-latest.zip
cd ghost/


```


Now we can install the Ghost dependencies and node modules (production dependencies only):


```
sudo npm install --production


```


Ghost is installed when this completes. We need to set up Ghost before we can start it.


# Step 3 — Setting up Ghost


Ghost’s configuration file should be located at /var/www/ghost/config.js. However, no such file is installed with Ghost. Instead, the installation includes config.example.js.


Copy the example configuration file to the proper location. Be sure to copy instead of move so you have a copy of the original configuration file in case you need to revert your changes.


```
sudo cp config.example.js config.js


```


Your URL and mail settings, which are in the production section, are the key areas of information which need modification. The URL is necessary. Otherwise, the links will take you to the default http://my-ghost-blog.com page. Ghost can function without the mail settings, but this is recommended that you add them. At the time of writing this article, Ghost only requires the mail functioning in case the user forgets their account password, so it wouldn’t do much harm not to configure mail.


Open the file for editing:


```
sudo nano config.js


```


You have to change the value of url to whatever your domain is (or you could use your server’s IP address in case you don’t want to use a domain right now). This value must be in the form of a URL. For example, http://example.com/ or http://45.55.76.126/. If this value is not formatted correctly, Ghost will not start.


Also change the value of host in the server section to 0.0.0.0.


The following shows the values that need to be changed in red:


/var/www/ghost/config.js
```
var path = require('path'),
    config;

config = {
    // ### Production
    // When running Ghost in the wild, use the production environment
    // Configure your URL and mail settings here
    production: {
        url: 'http://my-ghost-blog.com',
        mail: {
            // Your mail settings
        },
        database: {
            client: 'sqlite3',
            connection: {
                filename: path.join(__dirname, '/content/data/ghost.db')
            },
            debug: false
        },

        server: {
            // Host to be passed to node's `net.Server#listen()`
            host: '127.0.0.1',
            // Port to be passed to node's `net.Server#listen()`, for iisnode s$
            port: '2368'
        }
    },

(...)

```


Save the file and exit the nano text editor by pressing CTRL+X then Y and finally ENTER.


While still in the /var/www/ghost directory, start Ghost with the following command:


```
sudo npm start --production


```


The output should be similar to this:


```
> ghost@0.6.4 start /var/www/ghost
> node index

Migrations: Database initialisation required for version 003
Migrations: Creating tables...
Migrations: Creating table: posts

[...]


```


If all goes well, you should be able to access your blog using port 2368: http://your_domain._name:2368 (or http://your_servers_ip:2368).


Press CTRL+C in your terminal to shutdown the Ghost instance.



Note: Ghost can be customized further. Ghost.org explains each configuration option in detail.

# Step 4 — Install Nginx


The next step is to install Nginx. Basically, it will allow connections on port 80 to connect through to the port that Ghost is running on. In simple words, you would be able to access your Ghost blog without adding the :2368.


Install it with the following command:


```
sudo apt-get install nginx


```


Next, we will have to configure Nginx by changing our directory to /etc/nginx and removing the default file in /etc/nginx/sites-enabled:


```
cd /etc/nginx/
sudo rm sites-enabled/default


```


We will create a new file in /etc/nginx/sites-available/ called ghost and open it with nano to edit it:


```
sudo touch /etc/nginx/sites-available/ghost
sudo nano /etc/nginx/sites-available/ghost


```


Paste the following code in the file and change the highlighted code in red to your domain name, or your servers IP address if you don’t want to add a domain now:


```
server {
    listen 80;
    server_name your_domain.tld;
    location / {
        proxy_set_header   X-Real-IP $remote_addr;
        proxy_set_header   Host      $http_host;
        proxy_pass         http://127.0.0.1:2368;
    }
}

```


We will now symlink our configuration in sites-enabled:


```
sudo ln -s /etc/nginx/sites-available/ghost /etc/nginx/sites-enabled/ghost


```


We will restart Nginx:


```
sudo service nginx restart


```


Next we will create a new user. This user would be granted only the privileges to do stuff in the directory /var/www/ghost. This is a security measure. If Ghost gets compromised, your system would be safe. This can be done by running this command:


```
sudo adduser --shell /bin/bash --gecos 'Ghost application' ghost


```


We will grant privileges:


```
sudo chown -R ghost:ghost /var/www/ghost/


```


You can now log in as the ghost user:


```
su - ghost


```


Now we would need to start Ghost:


```
cd /var/www/ghost
npm start --production


```


You should be able to access your blog on port 80 as http://<your_server_ip>/ or http://<your_domain_name>/.


# Step 5 — Keep Ghost Running with forever


The next step is to keep Ghost running in the background. forever is a node module which can be used to start Ghost in the background and monitor to make sure it stays up. If Ghost crashes, forever will automatically start another instance of Ghost.


Install forever with the following command from within your Ghost directory, i.e. /var/www/ghost. But before running the command log out of the ghost user and login to your non-root user:


```
exit
sudo npm install -g forever


```


Start Ghost as the ghost user. It also must be run from the Ghost directory:


```
su - ghost
cd /var/www/ghost
forever start index.js


```


The output should be similar to this:


```
warn:    --minUptime not set. Defaulting to: 1000ms
warn:    --spinSleepTime not set. Your script will exit if it does not stay up for at least 1000ms
info:    Forever processing file: index.js


```


By default, it loads in the development environment. This can be changed by running the following command:


```
NODE_ENV=production forever start index.js


```


forever can be stopped by running this from the Ghost directory:


```
forever stop index.js


```


## Possible Errors


For the following error message:


```
Error: SQLITE_READONLY: attempt to write a readonly database

```


Start forever as a root user (type exit to logout the current user):


```
sudo forever start index.js


```


If the last command says it cannot find ‘forever’, use the full path to the command:


```
sudo /usr/local/bin/forever start index.js


```


If you see the following error:


```
error:   Cannot start forever
error:   script /home/ghost/index.js does not exist.

```


You are not in the /var/www/ghost directory. Change to this directory and execute the command again.


# Conclusion


Congratulations! You have installed Ghost and learned how to proxy ports with Nginx. You have also learned how to keep tasks running with the forever node package.


There is a lot more you can do with Ghost. For example, a password-protected blog is one of the latest features.


Check out the other DigitalOcean tutorials on Ghost:


- How to use the DigitalOcean Ghost Application
- How To Configure and Maintain Ghost from the Command Line
- How To Change Themes and Adjust Settings in Ghost

Also visit the following to learn more:


- Ghost.org - Ghost website
- Ghost Documentation — Official Ghost documentation
- Ghost Slack Page — Ghost’s Slack page to get help from real people of the Ghost community

