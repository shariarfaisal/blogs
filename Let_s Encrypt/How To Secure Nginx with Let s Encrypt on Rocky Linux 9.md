# How To Secure Nginx with Let s Encrypt on Rocky Linux 9

```Let's Encrypt``` ```Nginx``` ```Rocky Linux``` ```Rocky Linux 9``` ```Security```

## Introduction


Let’s Encrypt is a Certificate Authority (CA) that provides an accessible way to obtain and install free TLS/SSL certificates, thereby enabling encrypted HTTPS on web servers. It simplifies the process by providing a software client, Certbot, that attempts to automate most (if not all) of the required steps. Currently, the entire process of obtaining and installing a certificate is fully automated on both Apache and Nginx.


In this tutorial, you will use Certbot to obtain a free SSL certificate for Nginx on Rocky Linux 9 and set up your certificate to renew automatically.


This tutorial will use a separate Nginx server configuration file instead of the default file. You should create new Nginx server block files for each domain because it helps to avoid common mistakes and maintains the default files as a fallback configuration.


# Prerequisites


To follow this tutorial, you will need:


- 
One Rocky Linux 9 server set up by following this initial server setup for Rocky Linux 9  tutorial, including a sudo-enabled non-root user and a firewall.

- 
A registered domain name. This tutorial will use example.com throughout. You can purchase a domain name from Namecheap, get one for free with Freenom, or use the domain registrar of your choice.

- 
Both of the following DNS records set up for your server. If you are using DigitalOcean, please see our DNS documentation for details on how to add them.

An A record with example.com pointing to your server’s public IP address.
An A record with www.example.com pointing to your server’s public IP address.


- An A record with example.com pointing to your server’s public IP address.
- An A record with www.example.com pointing to your server’s public IP address.
- 
Nginx installed by following How To Install Nginx on Rocky Linux 9. Be sure that you have a server block for your domain. This tutorial will use /etc/nginx/sites-available/example.com as an example.


# Step 1 — Installing Certbot


First, you need to install the certbot software package. Log in to your Rocky Linux 8 machine as your non-root user:


```
ssh sammy@your_server_ip


```


The certbot package is not available through the package manager by default. You will need to enable the EPEL repository to install Certbot.


To add the Rocky Linux 9 EPEL repository, run the following command:


```
sudo dnf install epel-release


```


When asked to confirm the installation, type and enter y.


Now that you have access to the extra repository, install all of the required packages:


```
sudo dnf install certbot python3-certbot-nginx


```


This will install Certbot itself and the Nginx plugin for Certbot, which is needed to run the program.


The installation process will ask you about importing a GPG key. Confirm it so the installation can complete.


Now that you have Certbot installed, let’s run it to get a certificate.


# Step 2 — Confirming Nginx’s Configuration


Certbot needs to be able to find the correct server block in your Nginx configuration for it to be able to automatically configure SSL. Specifically, it does this by looking for a server_name directive that matches the domain you request a certificate for.


If you followed the server block set up step in the Nginx installation tutorial, you should have a server block for your domain at /etc/nginx/conf.d/example.com with the server_name directive already set appropriately.


To check, open the configuration file for your domain using nano or your favorite text editor:


```
sudo nano /etc/nginx/conf.d/example.com


```


Find the existing server_name line. It should look like this:


/etc/nginx/conf.d/example.com
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


# Step 3 — Updating the Firewall Rules


Since your prerequisite setup enables firewalld, you will need to adjust the firewall settings in order to allow external connections on your Nginx web server.


To check which services are already enabled, run the command:


```
sudo firewall-cmd --permanent --list-all


```


You’ll receive output like this:


```
Outputpublic
  target: default
  icmp-block-inversion: no
  interfaces:
  sources:
  services: cockpit dhcpv6-client http ssh
  ports:
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:

```


If you do not see http in the services list, enable it by running:


```
sudo firewall-cmd --permanent --add-service=http


```


To allow https traffic, run the following command:


```
sudo firewall-cmd --permanent --add-service=https


```


To apply the changes, you’ll need to reload the firewall service:


```
sudo firewall-cmd --reload


```


Now that you’ve opened up your server to https traffic, you’re ready to run Certbot and fetch your certificates.


# Step 4 — Obtaining an SSL Certificate


Certbot provides a variety of ways to obtain SSL certificates through plugins. The Nginx plugin will take care of reconfiguring Nginx and reloading the config whenever necessary. To use this plugin, type the following:


```
sudo certbot --nginx -d example.com -d www.example.com


```


This runs certbot with the --nginx plugin, using -d to specify the domain names that you need the certificate to be valid for.


When running the command, you will be prompted to enter an email address and agree to the terms of service. After doing so, you should see a message telling you the process was successful and where your certificates are stored:


```
OutputSuccessfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/your_domain/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/your_domain/privkey.pem
This certificate expires on 2022-12-15.
These files will be updated when the certificate renews.
Certbot has set up a scheduled task to automatically renew this certificate in the background.

Deploying certificate
Successfully deployed certificate for your_domain to /etc/nginx/conf.d/your_domain.conf
Successfully deployed certificate for www.your_domain to /etc/nginx/conf.d/your_domain.conf
Congratulations! You have successfully enabled HTTPS on https://your_domain and https://www.your_domain
…

```


Your certificates are downloaded, installed, and loaded, and your Nginx configuration will now automatically redirect all web requests to https://. Try reloading your website and notice your browser’s security indicator. It should indicate that the site is properly secured, usually with a lock icon. If you test your server using the SSL Labs Server Test, it will get an A grade.


Let’s finish by testing the renewal process.


# Step 5 — Verifying Certbot Auto-Renewal


Let’s Encrypt certificates are valid for 90 days, but it’s recommended that you renew the certificates every 60 days to allow for a margin of error. The Certbot Let’s Encrypt client has a renew command that automatically checks the currently installed certificates and tries to renew them if they are less than 30 days away from the expiration date.


You can test automatic renewal for your certificates by running this command:


```
sudo certbot renew --dry-run


```


The output will be similar to this:


```
OutputSaving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Processing /etc/letsencrypt/renewal/your_domain.conf
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Cert not due for renewal, but simulating renewal for dry run
Plugins selected: Authenticator nginx, Installer nginx
Renewing an existing certificate
Performing the following challenges:
http-01 challenge for monitoring.pp.ua
Waiting for verification...
Cleaning up challenges

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
new certificate deployed with reload of nginx server; fullchain is
/etc/letsencrypt/live/your_domain/fullchain.pem
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
** DRY RUN: simulating 'certbot renew' close to cert expiry
**          (The test certificates below have not been saved.)

Congratulations, all renewals succeeded. The following certs have been renewed:
  /etc/letsencrypt/live/your_domain/fullchain.pem (success)
...

```


Notice that if you created a bundled certificate with multiple domains, only the base domain name will show in the output, but the renewal will work for all domains included in this certificate.


A practical way to ensure your certificates will not get outdated is to create a cron job that will periodically execute the automatic renewal command for you. Since the renewal first checks for the expiration date and only executes the renewal if the certificate is less than 30 days away from expiration, it is safe to create a cron job that runs every week, or even every day.


Edit the crontab to create a new job that will run the renewal twice per day. To edit the crontab for the root user, run:


```
sudo crontab -e


```


Your text editor will open the default crontab, which is an empty text file at this point. Enter insert mode by pressing i and add in the following line:


crontab
```
0 0,12 * * * python -c 'import random; import time; time.sleep(random.random() * 3600)' && certbot renew --quiet

```


When you’re finished, press ESC to leave insert mode, then :wq and ENTER to save and exit the file. To learn more about the text editor Vi and its successor Vim, check out our Installing and Using the Vim Text Editor on a Cloud Server tutorial.


This will create a new cron job that will execute at noon and midnight every day. python -c 'import random; import time; time.sleep(random.random() * 3600)' will select a random minute within the hour for your renewal tasks.


The renew command for Certbot will check all certificates installed on the system and update any that are set to expire in less than thirty days. --quiet tells Certbot not to output information or wait for user input.


More detailed information about renewal can be found in the Certbot documentation.


# Conclusion


In this guide, you installed the Let’s Encrypt client Certbot, downloaded SSL certificates for your domain, and set up automatic certificate renewal. If you have any questions about using Certbot, you can check the official Certbot documentation.


You can also check the official Let’s Encrypt blog for important updates from time to time.


