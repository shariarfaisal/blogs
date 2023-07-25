# How To Secure Nginx with Let s Encrypt on Ubuntu 18 04

```Let's Encrypt``` ```Nginx``` ```Ubuntu 18.04```

## Introduction


Let’s Encrypt is a Certificate Authority (CA) that provides a way to obtain and install free TLS/SSL certificates, thereby enabling encrypted HTTPS on web servers. It streamlines the process by providing a software client, Certbot, that attempts to automate most (if not all) of the required steps. Currently, the entire process of obtaining and installing a certificate is fully automated on both Apache and Nginx.


In this tutorial, you will use Certbot to obtain a free SSL certificate for Nginx on Ubuntu 18.04 and set up your certificate to renew automatically.


This tutorial will use a separate Nginx server block file instead of the default file. We recommend creating new Nginx server block files for each domain because it helps to avoid common mistakes and maintains the default files as a fallback configuration.


# Prerequisites


To follow this tutorial, you will need:


- 
One Ubuntu 18.04 server set up by following this initial server setup for Ubuntu 18.04 tutorial, including a sudo non-root user and a firewall.

- 
A fully registered domain name. This tutorial will use your_domain throughout. You can purchase a domain name on Namecheap, get one for free on Freenom, or use the domain registrar of your choice.

- 
Both of the following DNS records set up for your server. You can follow this introduction to DigitalOcean DNS for details on how to add them.

An A record with your_domain pointing to your server’s public IP address.
An A record with www.your_domain pointing to your server’s public IP address.


- An A record with your_domain pointing to your server’s public IP address.
- An A record with www.your_domain pointing to your server’s public IP address.
- 
Nginx installed by following How To Install Nginx on Ubuntu 18.04. Be sure that you have a server block for your domain. Again, this tutorial will use /etc/nginx/sites-available/your_domain as an example.


# Step 1 — Installing Certbot


The first step to using Let’s Encrypt to obtain an SSL certificate is to install the Certbot software on your server.


The Certbot project recommends that most users install the software through snap, a package manager originally developed by Canonical (the company behind Ubuntu) and now available on many Linux distributions:


```
sudo snap install --classic certbot


```


Your output will display the current version of Certbot and successful installation:


```
Output
certbot 1.21.0 from Certbot Project (certbot-eff✓) installed

```


Next, create a symbolic link to the newly installed /snap/bin/certbot executable from the /usr/bin/ directory. This will ensure that the certbot command can run correctly on your server. To do this, run the following ln command. This contains the -s flag which will create a symbolic or soft link, as opposed to a hard link:


```
sudo ln -s /snap/bin/certbot /usr/bin/certbot


```


Certbot is now ready to use, but in order for it to configure SSL for Nginx, you need to verify some of Nginx’s configuration.


# Step 2 — Confirming Nginx’s Configuration


Certbot needs to be able to find the correct server block in your Nginx configuration for it to be able to automatically configure SSL. Specifically, it does this by searching for a server_name directive that matches the domain you request a certificate for.


If you followed the recommended server block set up step in the Nginx installation tutorial, you will have a server block for your domain at /etc/nginx/sites-available/your_domain with the server_name directive already set appropriately.


To check, open the server block file for your domain using nano or your favorite text editor:


```
sudo nano /etc/nginx/sites-available/your_domain


```


Find the existing server_name line. It should look like the following:


/etc/nginx/sites-available/your_domain
```
...
server_name your_domain www.your_domain;
...

```


If it does, exit your editor and move on to the next step.


If it doesn’t, update it to match. Then save the file and quit your editor. If you’re using nano you can do this by pressing CTRL + X then Y and ENTER.


Now verify the syntax of your configuration edits:


```
sudo nginx -t


```


If you get an error, reopen the server block file and check for any typos or missing characters. Once your configuration file’s syntax is correct, reload Nginx to load the new configuration:


```
sudo systemctl reload nginx


```


Certbot can now find the correct server block and update it.


Next, you’ll update the firewall to allow HTTPS traffic.


# Step 3 — Allowing HTTPS Through the Firewall


If you have the ufw firewall enabled, as recommended by the prerequisite guides, you’ll need to adjust the settings to allow for HTTPS traffic. Luckily, Nginx registers a few profiles with ufw upon installation.


You can check the current setting by running the following:


```
sudo ufw status


```


You should receive output like this, indicating that only HTTP traffic is allowed to the web server:


```
OutputStatus: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere                  
Nginx HTTP                 ALLOW       Anywhere                  
OpenSSH (v6)               ALLOW       Anywhere (v6)             
Nginx HTTP (v6)            ALLOW       Anywhere (v6)

```


To let in additional HTTPS traffic, allow the Nginx Full profile and delete the redundant Nginx HTTP profile allowance:


```
sudo ufw allow 'Nginx Full'
sudo ufw delete allow 'Nginx HTTP'


```


Now when you run the ufw status command it will reflect these new rules:


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


Next, you’ll run Certbot and fetch your certificates.


# Step 4 — Obtaining an SSL Certificate


Certbot provides a variety of ways to obtain SSL certificates through plugins. The Nginx plugin will take care of reconfiguring Nginx and reloading the configuration whenever necessary. To use this plugin, run the following:


```
sudo certbot --nginx -d your_domain -d your_domain


```


This runs certbot with the --nginx plugin, using -d to specify the names you’d like the certificate to be valid for.


If this is your first time running certbot, you will be prompted to enter an email address and agree to the terms of service. After doing so, certbot will communicate with the Let’s Encrypt server to request a certificate for your domain. If successful, you will receive the following output:


```
Output
Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/your_domain/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/your_domain/privkey.pem
This certificate expires on 2022-01-27.
These files will be updated when the certificate renews.
Certbot has set up a scheduled task to automatically renew this certificate in the background.

Deploying certificate
Successfully deployed certificate for your_domain to /etc/nginx/sites-enabled/your_domain
Successfully deployed certificate for www.your_domain to /etc/nginx/sites-enabled/your_domain
Congratulations! You have successfully enabled HTTPS on https://your_domain and https://www.your_domain

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
If you like Certbot, please consider supporting our work by:
 * Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
 * Donating to EFF:                    https://eff.org/donate-le
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

```


Your certificates are downloaded, installed, and loaded. Try reloading your website using https:// and notice your browser’s security indicator. It should indicate that the site is properly secured, usually with a green lock icon. If you test your server using the SSL Labs Server Test, it will get an A grade.


Now that you’ve obtained your SSL certificate, the final step is to test the renewal process.


# Step 5 — Verifying Certbot Auto-Renewal


Let’s Encrypt’s certificates are only valid for ninety days. This is to encourage users to automate their certificate renewal process. The certbot package you installed takes care of this by adding a renew script to /etc/cron.d. This script runs twice a day and will automatically renew any certificate that’s within thirty days of expiration.


To test the renewal process, you can do a dry run with certbot:


```
sudo certbot renew --dry-run


```


If you don’t receive errors, you’re all set. When necessary, Certbot will renew your certificates and reload Nginx to pick up the changes. If the automated renewal process ever fails, Let’s Encrypt will send a message to the email you specified, warning you when your certificate is about to expire.


# Conclusion


In this tutorial, you installed the Let’s Encrypt client certbot, downloaded SSL certificates for your domain, configured Nginx to use these certificates, and set up automatic certificate renewal. If you have further questions about using Certbot, their documentation is a good place to start.


