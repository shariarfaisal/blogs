# How To Secure Nginx with Let s Encrypt on Ubuntu 22 04

```Nginx``` ```Security``` ```Ubuntu``` ```Ubuntu 22.04```

## Introduction


Let’s Encrypt is a Certificate Authority (CA) that provides an accessible way to obtain and install free TLS/SSL certificates, thereby enabling encrypted HTTPS on web servers. It simplifies the process by providing a software client, Certbot, that attempts to automate most (if not all) of the required steps. Currently, the entire process of obtaining and installing a certificate is fully automated on both Apache and Nginx.


In this tutorial, you will use Certbot to obtain a free SSL certificate for Nginx on Ubuntu 22.04 and set up your certificate to renew automatically.


This tutorial will use a separate Nginx server configuration file instead of the default file. We recommend creating new Nginx server block files for each domain because it helps to avoid common mistakes and maintains the default files as a fallback configuration.


# Prerequisites


To follow this tutorial, you will need:


- 
One Ubuntu 22.04 server set up by following this initial server setup for Ubuntu 22.04 tutorial, including a sudo-enabled non-root user and a firewall.

- 
A registered domain name. This tutorial will use example.com throughout. You can purchase a domain name from Namecheap, get one for free with Freenom, or use the domain registrar of your choice.

- 
Both of the following DNS records set up for your server. If you are using DigitalOcean, please see our DNS documentation for details on how to add them.

An A record with example.com pointing to your server’s public IP address.
An A record with www.example.com pointing to your server’s public IP address.


- An A record with example.com pointing to your server’s public IP address.
- An A record with www.example.com pointing to your server’s public IP address.
- 
Nginx installed by following How To Install Nginx on Ubuntu 22.04. Be sure that you have a server block for your domain. This tutorial will use /etc/nginx/sites-available/example.com as an example.


# Step 1 — Installing Certbot


Certbot recommends using their snap package for installation. Snap packages work on nearly all Linux distributions, but they require that you’ve installed snapd first in order to manage snap packages. Ubuntu 22.04 comes with support for snaps out of the box, so you can start by making sure your snapd core is up to date:


```
sudo snap install core; sudo snap refresh core


```


If you’re working on a server that previously had an older version of certbot installed, you should remove it before going any further:


```
sudo apt remove certbot


```


After that, you can install the certbot package:


```
sudo snap install --classic certbot


```


Finally, you can link the certbot command from the snap install directory to your path, so you’ll be able to run it by just typing certbot. This isn’t necessary with all packages, but snaps tend to be less intrusive by default, so they don’t conflict with any other system packages by accident:


```
sudo ln -s /snap/bin/certbot /usr/bin/certbot


```


Now that we have Certbot installed, let’s run it to get our certificate.


# Step 2 — Confirming Nginx’s Configuration


Certbot needs to be able to find the correct server block in your Nginx configuration for it to be able to automatically configure SSL. Specifically, it does this by looking for a server_name directive that matches the domain you request a certificate for.


If you followed the server block set up step in the Nginx installation tutorial, you should have a server block for your domain at /etc/nginx/sites-available/example.com with the server_name directive already set appropriately.


To check, open the configuration file for your domain using nano or your favorite text editor:


```
sudo nano /etc/nginx/sites-available/example.com


```


Find the existing server_name line. It should look like this:


/etc/nginx/sites-available/example.com
```
...
server_name example.com www.example.com;
...

```


If it does, exit your editor and move on to the next step.


If it doesn’t, update it to match. Then save the file, quit your editor, and verify the syntax of your configuration edits:


```
sudo nginx -t


```


If you get an error, reopen the server block file and check for any typos or missing characters. Once your configuration file’s syntax is correct, reload Nginx to load the new configuration:


```
sudo systemctl reload nginx


```


Certbot can now find the correct server block and update it automatically.


Next, let’s update the firewall to allow HTTPS traffic.


# Step 3 — Allowing HTTPS Through the Firewall


If you have the ufw firewall enabled, as recommended by the prerequisite guides, you’ll need to adjust the settings to allow for HTTPS traffic. Luckily, Nginx registers a few profiles with ufw upon installation.


You can see the current setting by typing:


```
sudo ufw status


```


It will probably look like this, meaning that only HTTP traffic is allowed to the web server:


```
OutputStatus: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere                  
Nginx HTTP                 ALLOW       Anywhere                  
OpenSSH (v6)               ALLOW       Anywhere (v6)             
Nginx HTTP (v6)            ALLOW       Anywhere (v6)

```


To additionally let in HTTPS traffic, allow the Nginx Full profile and delete the redundant Nginx HTTP profile allowance:


```
sudo ufw allow 'Nginx Full'
sudo ufw delete allow 'Nginx HTTP'


```


Your status should now look like this:


```
sudo ufw status


```


```
OutputStatus: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
Nginx Full                 ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
Nginx Full (v6)            ALLOW       Anywhere (v6)

```


Next, let’s run Certbot and fetch our certificates.


# Step 4 — Obtaining an SSL Certificate


Certbot provides a variety of ways to obtain SSL certificates through plugins. The Nginx plugin will take care of reconfiguring Nginx and reloading the config whenever necessary. To use this plugin, type the following:


```
sudo certbot --nginx -d example.com -d www.example.com


```


This runs certbot with the --nginx plugin, using -d to specify the domain names we’d like the certificate to be valid for.


When running the command, you will be prompted to enter an email address and agree to the terms of service. After doing so, you should see a message telling you the process was successful and where your certificates are stored:


```
OutputIMPORTANT NOTES:
Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/your_domain/fullchain.pem
Key is saved at: /etc/letsencrypt/live/your_domain/privkey.pem
This certificate expires on 2022-06-01.
These files will be updated when the certificate renews.
Certbot has set up a scheduled task to automatically renew this certificate in the background.

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
If you like Certbot, please consider supporting our work by:
* Donating to ISRG / Let's Encrypt: https://letsencrypt.org/donate
* Donating to EFF: https://eff.org/donate-le

```


Your certificates are downloaded, installed, and loaded, and your Nginx configuration will now automatically redirect all web requests to https://. Try reloading your website and notice your browser’s security indicator. It should indicate that the site is properly secured, usually with a lock icon. If you test your server using the SSL Labs Server Test, it will get an A grade.


Let’s finish by testing the renewal process.


# Step 5 — Verifying Certbot Auto-Renewal


Let’s Encrypt’s certificates are only valid for ninety days. This is to encourage users to automate their certificate renewal process. The certbot package we installed takes care of this for us by adding a systemd timer that will run twice a day and automatically renew any certificate that’s within thirty days of expiration.


You can query the status of the timer with systemctl:


```
sudo systemctl status snap.certbot.renew.service


```


```
Output○ snap.certbot.renew.service - Service for snap application certbot.renew
     Loaded: loaded (/etc/systemd/system/snap.certbot.renew.service; static)
     Active: inactive (dead)
TriggeredBy: ● snap.certbot.renew.timer

```


To test the renewal process, you can do a dry run with certbot:


```
sudo certbot renew --dry-run


```


If you see no errors, you’re all set. When necessary, Certbot will renew your certificates and reload Nginx to pick up the changes. If the automated renewal process ever fails, Let’s Encrypt will send a message to the email you specified, warning you when your certificate is about to expire.


# Conclusion


In this tutorial, you installed the Let’s Encrypt client certbot, downloaded SSL certificates for your domain, configured Nginx to use these certificates, and set up automatic certificate renewal. If you have further questions about using Certbot, the official documentation is a good place to start.


