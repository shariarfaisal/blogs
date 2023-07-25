# How To Install the Etherpad Collaborative Web Editor on Ubuntu 20 04

```Ubuntu``` ```Ubuntu 20.04``` ```Node.js``` ```Applications``` ```SQLite```

## Introduction


Etherpad is a web application that enables real-time collaborative text editing in the browser. It is written in Node.js and can be self-hosted on a variety of platforms and operating systems.


In this tutorial we will install Etherpad on an Ubuntu 20.04 server, using the SQLite database engine to store our data. We’ll also install and configure Nginx to act as a reverse proxy for the application, and we’ll fetch and install SSL certificates from the Let’s Encrypt certificate authority to enable secure HTTPS connections to our Etherpad instance.



Note: If you’d prefer to use our App Platform service to self-host Etherpad, please see our Deploy the Etherpad Collaborative Web Editor to App Platform tutorial, where we create an App Platform app to run the Etherpad Docker container, and connect it to a managed PostgreSQL database.

# Prerequisites


Before starting this tutorial, you will need the following:


- An Ubuntu 20.04 server, with a non-root, sudo-enabled user, and with the UFW firewall enabled. Please read our Initial Server Setup with Ubuntu 20.04 to learn more about setting up these requirements.
- Node.js installed, version 14 or higher. See option 2 of How To Install Node.js on Ubuntu 20.04 to learn how to install an up-to-date version of Node.js using NodeSource packages.
- A domain name pointed at your server’s public IP address. This should be something like example.com or etherpad.example.com, for instance.


Note: If you’re using DigitalOcean, our DNS Documentation can help you get your domain name set up in the control panel.

When you have the prerequisites in place, continue to Step 1, where we’ll download and configure the Etherpad application.


# Step 1 — Downloading and Configuring Etherpad


To install Etherpad, you’ll need to download the source code, install dependencies, and configure systemd to run the server.


The Etherpad maintainers recommend running the software as its own user, so your first step will be to create a new etherpad user using the adduser command:


```
sudo adduser --system --group --home /opt/etherpad etherpad


```


This creates a --system user, meaning that it can’t log in directly and has no password or shell assigned. We give it a home directory of /opt/etherpad, which is where we’ll download and configure the Etherpad software. We also create an etherpad group using the --group flag.


You now need to run a few commands as the etherpad user. To do so, you’ll use the sudo command to open a bash shell as the etherpad user. Then you’ll change directories (cd) to /opt/etherpad:


```
sudo -u etherpad bash
cd /opt/etherpad


```


Your shell prompt will update to show that you’re the etherpad user. It should look similar to etherpad@host:~$.


Now clone the Etherpad repository into /opt/etherpad using Git:


```
git clone --branch master https://github.com/ether/etherpad-lite.git .


```


This will pull the master branch of the Etherpad source code into the current directory (.). When that’s done, run Etherpad’s installDeps.sh script to install the dependencies:


```
./bin/installDeps.sh


```


This can take a minute. When it’s done, you’ll need to manually install one last dependency. We need to cd into the Etherpad src folder and install the sqlite3 package in order to use SQLite as our database.


First, change into the src directory:


```
cd src


```


Then install the sqlite3 package using npm:


```
npm install sqlite3


```


Your final task as the etherpad user is to update the Etherpad settings.json file to use SQLite for its database, and to work well with Nginx. Move back up to the /opt/etherpad directory:


```
cd /opt/etherpad


```


Then open the settings file using your favorite text editor:


```
nano settings.json


```


The file is formatted as JSON, but with extensive comments throughout explaining each setting. There’s a lot you can configure, but for now we’re interested in two values that update the database configuration:


settings.json
```
  "dbType": "dirty",
  "dbSettings": {
    "filename": "var/dirty.db"
  },

```


Scroll down and look for the dbType and dbSettings section, shown here. Update the settings to sqlite and a filename of your choice, like the following:


settings.json
```
  "dbType": "sqlite",
  "dbSettings": {
    "filename": "var/sqlite.db"
  },

```


Finally, scroll down some more, find the trustProxy setting, and update it to true:


settings.json
```
"trustProxy": true,

```


Save and close the settings file. In nano you can save and close by typing CTRL+O then ENTER to save, and CTRL+X to exit.


When that’s done, be sure to exit the etherpad user’s shell:


```
exit


```


You’ll be returned to your normal user’s shell.


Etherpad is installed and configured. Next we’ll create a systemd service to start and manage the Etherpad process.


# Step 2 — Creating a Systemd Service for Etherpad


In order to start Etherpad on boot and to manage the process using systemctl, we need to create a systemd service file. Open up the new file in your favorite text editor:


```
sudo nano /etc/systemd/system/etherpad.service


```


We’re going to create a service definition based on information in Etherpad’s documentation wiki. The How to deploy Etherpad Lite as a service page gives an example configuration that needs just a few changes to make it work for us.


Add the following content into your text editor, then save and close the file:


/etc/systemd/system/etherpad.service
```
[Unit]
Description=Etherpad, a collaborative web editor.
After=syslog.target network.target

[Service]
Type=simple
User=etherpad
Group=etherpad
WorkingDirectory=/opt/etherpad
Environment=NODE_ENV=production
ExecStart=/usr/bin/node --experimental-worker /opt/etherpad/node_modules/ep_etherpad-lite/node/server.js
Restart=always

[Install]
WantedBy=multi-user.target

```


This file gives systemd the information it needs to run Etherpad, including the user and group to run it as, and the command used to start the process (ExecStart=...).


After closing the file, reload the systemd daemon to pull in the new configuration:


```
sudo systemctl daemon-reload


```


Next, enable the etherpad service. This means the service will start up whenever your server reboots:


```
sudo systemctl enable etherpad


```


And finally, we can start the service:


```
sudo systemctl start etherpad


```


Check that the service started properly using systemctl status:


```
sudo systemctl status etherpad


```


```
Output● etherpad.service - Etherpad, a collaborative web editor.
     Loaded: loaded (/etc/systemd/system/etherpad.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2021-09-09 14:12:43 UTC; 18min ago
   Main PID: 698 (node)
      Tasks: 13 (limit: 1136)
     Memory: 152.0M
     CGroup: /system.slice/etherpad.service
             └─698 /usr/bin/node --experimental-worker /opt/etherpad/node_modules/ep_etherpad-lite/node/server.js

```


The output should indicate that the service is active (running).


Etherpad is now running, but it is unavailable to the public because port 9001 is blocked by your firewall. In the next step we’ll make Etherpad public by setting up Nginx as a reverse proxy in front of the Etherpad process.


# Step 3 — Installing and Configuring Nginx


Putting a web server such as Nginx in front of your Node.js server can improve performance by offloading caching, compression, and static file serving to a more efficient process. We’re going to install Nginx and configure it to proxy requests to Etherpad, meaning it will take care of handing requests from your users to Etherpad and back again.


First, refresh your package list, then install Nginx using apt:


```
sudo apt update
sudo apt install nginx


```


Allow traffic to ports 80 and 443 (HTTP and HTTPS) using the “Nginx Full” UFW application profile:


```
sudo ufw allow "Nginx Full"


```


```
OutputRule added
Rule added (v6)

```


Next, open up a new Nginx configuration file in the /etc/nginx/sites-available directory. We’ll call ours etherpad.conf but you could use a different name:


```
sudo nano /etc/nginx/sites-available/etherpad.conf


```


Paste the following into the new configuration file, being sure to replace your_domain_here with the domain that is pointing to your Etherpad server. This will be something like etherpad.example.com, for instance.


/etc/nginx/sites-available/etherpad.conf
```
server {
    listen       80;
    listen       [::]:80;
    server_name  your_domain_here;

    access_log  /var/log/nginx/etherpad.access.log;
    error_log   /var/log/nginx/etherpad.error.log;

    location / {
        proxy_pass         http://127.0.0.1:9001;
        proxy_buffering    off;
        proxy_set_header   Host $host;
        proxy_pass_header  Server;

        # proxy headers
        proxy_set_header    X-Real-IP $remote_addr;
        proxy_set_header    X-Forwarded-For $remote_addr;
        proxy_set_header    X-Forwarded-Proto $scheme;
        proxy_http_version  1.1;

        # websocket proxying
        proxy_set_header  Upgrade $http_upgrade;
        proxy_set_header  Connection "upgrade";
    }
}

```


This configuration is loosely based on a configuration provided on the Etherpad wiki. It is HTTP-only for now, as we’ll let Certbot take care of configuring SSL in the next step. The rest of the config sets up logging locations and then passes all traffic along to http://127.0.0.1:9001, the Etherpad instance we started up in the previous step. We also set various headers that are required for well-behaved proxying and for websockets (persistent HTTP connections that enable real-time two-way communication) to work through a proxy.


Save and close the file, then enable the configuration by linking it into /etc/nginx/sites-enabled/:


```
sudo ln -s /etc/nginx/sites-available/etherpad.conf /etc/nginx/sites-enabled/


```


Use nginx -t to verify that the configuration file syntax is correct:


```
sudo nginx -t


```


```
[secondary_lable Output]
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

```


And finally, reload the nginx service to pick up the new configuration:


```
sudo systemctl reload nginx


```


Your Etherpad site should now be available on plain HTTP, and it will look something like this:





Now that we have our site up and running over HTTP, it’s time to secure the connection with Certbot and Let’s Encrypt certificates.


# Step 4 — Installing Certbot and Setting Up SSL Certificates


Thanks to Certbot and the Let’s Encrypt free certificate authority, adding SSL encryption to our Etherpad app will take only two commands.


First, install Certbot and its Nginx plugin:


```
sudo apt install certbot python3-certbot-nginx


```


Next, run certbot in --nginx mode, and specify the same domain you used in the Nginx server_name config:


```
sudo certbot --nginx -d your_domain_here


```


You’ll be prompted to agree to the Let’s Encrypt terms of service, and to enter an email address.


Afterwards, you’ll be asked if you want to redirect all HTTP traffic to HTTPS. It’s up to you, but this is generally recommended and safe to do.


After that, Let’s Encrypt will confirm your request and Certbot will download your certificate:


```
OutputCongratulations! You have successfully enabled https://etherpad.example.com

You should test your configuration at:
https://www.ssllabs.com/ssltest/analyze.html?d=etherpad.example.com
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/etherpad.example.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/etherpad.example.com/privkey.pem
   Your cert will expire on 2021-12-06. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot again
   with the "certonly" option. To non-interactively renew *all* of
   your certificates, run "certbot renew"
 - Your account credentials have been saved in your Certbot
   configuration directory at /etc/letsencrypt. You should make a
   secure backup of this folder now. This configuration directory will
   also contain certificates and private keys obtained by Certbot so
   making regular backups of this folder is ideal.
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le

```


Certbot will automatically reload Nginx to pick up the new configuration and certificates. Reload your site and it should switch you over to HTTPS automatically if you chose the redirect option.





You’re done! Try out your new Etherpad editor and invite some collaborators.


# Conclusion


In this tutorial we set up Etherpad, with Nginx and Let’s Encrypt SSL certificates. Your Etherpad is now ready to use, but there’s more configuration you may want to do, including adding authenticated users, adding plugins, and customizing the user interface through skins.


Your SQLite-backed Etherpad instance will be able to handle a moderate number of active users, but if you anticipate very high traffic, you may want to look into configuring a MySQL or PostgreSQL database instead.


All of these configuration options are documented on the official Etherpad wiki.


