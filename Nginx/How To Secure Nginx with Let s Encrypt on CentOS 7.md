# How To Secure Nginx with Let s Encrypt on CentOS 7

```Security``` ```Let's Encrypt``` ```Nginx``` ```CentOS```

## Introduction


Let’s Encrypt is a new Certificate Authority (CA) that provides a way to obtain and install free TLS/SSL certificates, thereby enabling encrypted HTTPS on web servers. It streamlines the process by providing a software client, Certbot, that attempts to automate most (if not all) of the required steps. Currently, the entire process of obtaining and installing a certificate is fully automated on both Apache and Nginx web servers.


In this tutorial, we will show you how to use the certbot Let’s Encrypt client to obtain a free SSL certificate and use it with Nginx on CentOS 7. We will also show you how to automatically renew your SSL certificate.


# Prerequisites


Before following this tutorial, you’ll need a few things.


- A CentOS 7 server with a non-root user who has sudo privileges.  You can learn how to set up such a user account by following steps 1-3 in our initial server setup for CentOS 7 tutorial.
- You must own or control the registered domain name that you wish to use the certificate with. If you do not already have a registered domain name, you may register one with one of the many domain name registrars out there (e.g. Namecheap, GoDaddy, etc.).
- A DNS A Record that points your domain to the public IP address of your server. This is required because of how Let’s Encrypt validates that you own the domain it is issuing a certificate for. For example, if you want to obtain a certificate for example.com, that domain must resolve to your server for the validation process to work. Our setup will use example.com and www.example.com as the domain names, so both DNS records are required.

Once you have all of the prerequisites out of the way, let’s move on to installing the Let’s Encrypt client software.


# Step 1 — Installing the Certbot Let’s Encrypt Client


The first step to using Let’s Encrypt to obtain an SSL certificate is to install the certbot software on your server. Currently, the best way to install this is through the EPEL repository.


Enable access to the EPEL repository on your server by typing:


```
sudo yum install epel-release


```


Once the repository has been enabled, you can obtain the certbot-nginx package by typing:


```
sudo yum install certbot-nginx


```


The certbot Let’s Encrypt client is now installed and ready to use.


# Step 2 — Setting up Nginx


If you haven’t installed Nginx yet, you can do so now.  The EPEL repository should already be enabled from the previous section, so you can install Nginx by typing:


```
sudo yum install nginx


```


Then, start Nginx using systemctl:


```
sudo systemctl start nginx


```


Certbot can automatically configure SSL for Nginx, but it needs to be able to find the correct server block in your config. It does this by looking for a server_name directive that matches the domain you’re requesting a certificate for. If you’re starting out with a fresh Nginx install, you can update the default config file using vi or your favorite text editor:


```
sudo vi /etc/nginx/nginx.conf


```


Find the existing server_name line:


/etc/nginx/nginx.conf
```
server_name _;

```


Replace the _ underscore with your domain name:


/etc/nginx/nginx.conf
```
server_name example.com www.example.com;

```


Save the file and quit your editor. If you are using vi, enter :x, then y when prompted, to save and quit. Verify the syntax of your configuration edits with:


```
sudo nginx -t


```


If that runs with no errors, reload Nginx to load the new configuration:


```
sudo systemctl reload nginx


```


Certbot will now be able to find the correct server block and update it. Now we’ll update our firewall to allow HTTPS traffic.


# Step 3 — Updating the Firewall


If you have a firewall enabled, make sure port 80 and 443 are open to incoming traffic. If you are not running a firewall, you can skip ahead.


If you have a firewalld firewall running, you can open these ports by typing:


```
sudo firewall-cmd --add-service=http
sudo firewall-cmd --add-service=https
sudo firewall-cmd --runtime-to-permanent


```


If have an iptables firewall running, the commands you need to run are highly dependent on your current rule set.  For an initial rule set, you can add HTTP and HTTPS access by typing:


```
sudo iptables -I INPUT -p tcp -m tcp --dport 80 -j ACCEPT
sudo iptables -I INPUT -p tcp -m tcp --dport 443 -j ACCEPT


```


We’re now ready to run Certbot and fetch our certificates.


# Step 4 — Obtaining a Certificate


Certbot provides a variety of ways to obtain SSL certificates, through various plugins. The Nginx plugin will take care of reconfiguring Nginx and reloading the config whenever necessary:


```
sudo certbot --nginx -d example.com -d www.example.com


```


This runs certbot with the --nginx plugin, using -d to specify the names we’d like the certificate to be valid for.


If this is your first time running certbot, you will be prompted to enter an email address and agree to the terms of service. After doing so, certbot will communicate with the Let’s Encrypt server, then run a challenge to verify that you control the domain you’re requesting a certificate for. The configuration will be updated, and Nginx will reload to pick up the new settings. certbot will wrap up with a message telling you the process was successful and where your certificates are stored:


```
OutputIMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/your_domain/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/your_domain/privkey.pem
   Your certificate will expire on 2022-10-20. To obtain a new or
   tweaked version of this certificate in the future, simply run
   certbot again with the "certonly" option. To non-interactively
   renew *all* of your certificates, run "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le

```


Your certificates are downloaded, installed, and loaded. Try reloading your website using https:// and notice your browser’s security indicator. It should represent that the site is properly secured, usually with a green lock icon.


# Step 5 — Setting Up Auto Renewal


Let’s Encrypt’s certificates are only valid for ninety days. This is to encourage users to automate their certificate renewal process. We’ll need to set up a regularly run command to check for expiring certificates and renew them automatically.


To run the renewal check daily, we will use cron, a standard system service for running periodic jobs. We tell cron what to do by opening and editing a file called a crontab.


```
sudo crontab -e


```


Your text editor will open the default crontab which is an empty text file at this point. Paste in the following line, then save and close it:


crontab
```
. . .
15 3 * * * /usr/bin/certbot renew --quiet

```


The 15 3 * * * part of this line means “run the following command at 3:15 am, every day”. You may choose any time.


The renew command for Certbot will check all certificates installed on the system and update any that are set to expire in less than thirty days. --quiet tells Certbot not to output information or wait for user input.


cron will now run this command daily. All installed certificates will be automatically renewed and reloaded when they have thirty days or less before they expire.



For more information on how to create and schedule cron jobs, you can check our How to Use Cron to Automate Tasks in a VPS guide.

# Conclusion


In this tutorial we’ve installed the Let’s Encrypt client certbot, downloaded SSL certificates for our domain, configured Nginx to use these certificates, and set up automatic certificate renewal. If you have further questions about using Certbot, their documentation is a good place to start.


