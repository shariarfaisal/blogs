# How To Secure Nginx with Let s Encrypt on Rocky Linux 8

```Let's Encrypt``` ```Nginx``` ```Rocky Linux``` ```Rocky Linux 8```

## Introduction


Let’s Encrypt is a certificate authority (CA) that provides free certificates for Transport Layer Security (TLS) encryption. It simplifies the process of creation, validation, signing, installation, and renewal of certificates by providing a software client—Certbot.


In this tutorial you’ll set up a TLS/SSL certificate from Let’s Encrypt on a Rocky Linux 8 server running Nginx as a web server. Additionally, you will automate the certificate renewal process using a cron job.


# Prerequisites


In order to complete this guide, you will need:


- One Rocky Linux 8 server set up by following the Rocky Linux 8 Initial Server Setup guide, including a non-root user with sudo privileges and a firewall.
- Nginx installed on the Rocky Linux 8 server with a configured server block. You can learn how to set this up by following our tutorial How To Install Nginx on Rocky Linux 8.
- A fully registered domain name. This tutorial will use your_domain as an example throughout. You can purchase a domain name on Namecheap, get one for free on Freenom, or use the domain registrar of your choice.
- Both of the following DNS records set up for your server. You can follow this introduction to DigitalOcean DNS for details on how to add them.

An A record with your_domain pointing to your server’s public IP address.
An A record with www.your_domain pointing to your server’s public IP address.


- An A record with your_domain pointing to your server’s public IP address.
- An A record with www.your_domain pointing to your server’s public IP address.

# Step 1 — Installing the Certbot Let’s Encrypt Client


First, you need to install the certbot software package. Log in to your Rocky Linux 8 machine as your non-root user:


```
ssh sammy@your_server_ip


```


The certbot package is not available through the package manager by default. You will need to enable the EPEL repository to install Certbot.


To add the Rocky Linux 8 EPEL repository, use dnf install:


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


You have now installed the Let’s Encrypt client, but before obtaining certificates, you need to make sure that all required ports are open. To do this, you will update your firewall settings in the next step.


# Step 2 — Updating the Firewall Rules


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


# Step 3 — Obtaining a Certificate


Now you can request an SSL certificate for your domain.


When generating the SSL Certificate for Nginx using the certbot Let’s Encrypt client, the client will automatically obtain and install a new SSL certificate that is valid for the domains provided as parameters.


If you want to install a single certificate that is valid for multiple domains or subdomains, you can pass them as additional parameters to the command. The first domain name in the list of parameters will be the base domain used by Let’s Encrypt to create the certificate, and for that reason you will pass the top-level domain name as first in the list, followed by any additional subdomains or aliases:


```
sudo certbot --nginx -d your_domain -d www.your_domain


```


This runs certbot with the --nginx plugin, and the base domain will be your_domain. To execute the interactive installation and obtain a certificate that covers only a single domain, run the certbot command with:


```
sudo certbot --nginx -d your_domain


```


The certbot utility can also prompt you for domain information during the certificate request procedure. To use this functionality, call certbot without any domains:


```
sudo certbot --nginx


```


You will receive a step-by-step guide to customize your certificate options. Certbot will ask you to provide an email address for lost key recovery and notices and to agree to the terms of service. If you did not specify your domains on the command line, Certbot will look for a server_name directive and will give you a list of the domain names found. If your server block files do not specify the domain they serve explicitly using the server_name directive, Certbot will ask you to provide domain names manually.


For better security, Certbot will automatically configure redirecting all traffic on port 80 to 443.


When the installation successfully finishes, you will receive a message similar to this:


```
OutputIMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/your_domain/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/your_domain/privkey.pem
   Your cert will expire on 2022-11-14. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot again
   with the "certonly" option. To non-interactively renew *all* of
   your certificates, run "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le


```


The generated certificate files will be available within a subdirectory named after your base domain in the /etc/letsencrypt/live directory.


Now that you have finished using Certbot, you can check your SSL certificate status. Verify the status of your SSL certificate by opening the following link in your preferred web browser (don’t forget to replace your_domain with your base domain):


```
https://www.ssllabs.com/ssltest/analyze.html?d=your_domain

```


This site contains an SSL test from SSL Labs, which will start automatically. At the time of this writing, default settings will give an A rating.


You can now access your website using the https prefix. However, you must renew certificates periodically to keep this setup working. In the next step, you will automate this renewal process.


# Step 4 — Setting Up Auto-Renewal


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


