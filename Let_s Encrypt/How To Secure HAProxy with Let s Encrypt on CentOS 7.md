# How To Secure HAProxy with Let s Encrypt on CentOS 7

```Security``` ```Let's Encrypt``` ```Load Balancing``` ```CentOS```

## Introduction


Let’s Encrypt is a new Certificate Authority (CA) that provides an easy way to obtain and install free TLS/SSL certificates, thereby enabling encrypted HTTPS on web servers. It simplifies the process by providing a software client, Certbot, that attempts to automate most of the required steps. Currently the entire process of obtaining and installing a certificate is fully automated only on Apache web servers. However, Certbot can be used to easily obtain a free SSL certificate, which can be installed manually, regardless of your choice of web server software.


In this tutorial, we will show you how to use Let’s Encrypt to obtain a free SSL certificate and use it with HAProxy on CentOS 7. We will also show you how to automatically renew your SSL certificate.





# Prerequisites


Before following this tutorial, you’ll need a few things.


You should have an CentOS 7 server with a non-root user who has sudo privileges.  You can learn how to set up such a user account by following steps 1-3 in our initial server setup for CentOS 7 tutorial.


You must own or control the registered domain name that you wish to use the certificate with. If you do not already have a registered domain name, you may register one with one of the many domain name registrars out there (e.g. Namecheap, GoDaddy, etc.).


If you haven’t already, be sure to create an A Record that points your domain to the public IP address of your server. This is required because of how Let’s Encrypt validates that you own the domain it is issuing a certificate for. For example, if you want to obtain a certificate for example.com, that domain must resolve to your server for the validation process to work. Our setup will use example.com and www.example.com as the domain names, so both DNS records are required.


Once you have all of the prerequisites out of the way, let’s move on to installing Certbot, the Let’s Encrypt client software.


# Step 1 — Installing Certbot, the Let’s Encrypt Client


The first step to using Let’s Encrypt to obtain an SSL certificate is to install the certbot software on your server. Currently, the best way to install this is through the EPEL repository.


Enable access to the EPEL repository on your server by typing:


```
sudo yum install epel-release


```


Once the repository has been enabled, you can obtain the certbot package by typing:


```
sudo yum install certbot


```


The certbot Let’s Encrypt client should now be installed and ready to use.


# Step 2 — Obtaining a Certificate


Let’s Encrypt provides a variety of ways to obtain SSL certificates, through various plugins. Unlike the Apache plugin, which is covered in a different tutorial, most of the plugins will only help you with obtaining a certificate which you must manually configure your web server to use. Plugins that only obtain certificates, and don’t install them, are referred to as “authenticators” because they are used to authenticate whether a server should be issued a certificate.


We’ll show you how to use the Standalone plugin to obtain an SSL certificate.


## Verify Port 80 is Open


The Standalone plugin provides a very simple way to obtain SSL certificates. It works by temporarily running a small web server, on port 80, on your server, to which the Let’s Encrypt CA can connect and validate your server’s identity before issuing a certificate. As such, this method requires that port 80 is not in use. That is, be sure to stop your normal web server, if it’s using port 80 (i.e. http), before attempting to use this plugin.


For example, if you’re using HAProxy, you can stop it by running this command:


```
sudo systemctl stop haproxy


```


If you’re not sure if port 80 is in use, you can run this command:


```
netstat -na | grep ':80.*LISTEN'

```


If there is no output when you run this command, you can use the Standalone plugin.


## Run Certbot


Now use the Standalone plugin by running this command:


```
sudo certbot certonly --standalone --preferred-challenges http --http-01-port 80 -d example.com -d www.example.com


```


You will be prompted to enter your email address and agree to the Let’s Encrypt terms of service. Afterwards, the http challenge will run. If everything is successful, certbot will print an output message like this:


```
Output:IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at
   /etc/letsencrypt/live/example.com/fullchain.pem. Your cert
   will expire on 2017-09-06. To obtain a new or tweaked version of
   this certificate in the future, simply run certbot again. To
   non-interactively renew *all* of your certificates, run "certbot
   renew"
 - Your account credentials have been saved in your Certbot
   configuration directory at /etc/letsencrypt. You should make a
   secure backup of this folder now. This configuration directory will
   also contain certificates and private keys obtained by Certbot so
   making regular backups of this folder is ideal.
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le

```


You will want to note the path and expiration date of your certificate, which was highlighted in the example output above.



Note: If your domain is routing through a DNS service like CloudFlare, you will need to temporarily disable it until you have obtained the certificate.

## Certificate Files


After obtaining the cert, you will have the following PEM-encoded files:


- cert.pem: Your domain’s certificate
- chain.pem: The Let’s Encrypt chain certificate
- fullchain.pem: cert.pem and chain.pem combined
- privkey.pem: Your certificate’s private key

It’s important that you are aware of the location of the certificate files that were just created, so you can use them in your web server configuration. The files themselves are placed in a subdirectory in /etc/letsencrypt/archive. However, Certbot creates symbolic links to the most recent certificate files in the /etc/letsencrypt/live/your_domain_name directory.


You can check that the files exist by running this command (substituting in your domain name):


```
sudo ls /etc/letsencrypt/live/your_domain_name


```


The output should be the four previously mentioned certificate files.


## Combine fullchain.pem and privkey.pem


When configuring HAProxy to perform SSL termination, so it will encrypt traffic between itself and the end user, you must combine fullchain.pem and privkey.pem into a single file.


First, create the directory where the combined file will be placed, /etc/haproxy/certs:


```
sudo mkdir -p /etc/haproxy/certs


```


Next, create the combined file with this cat command (substitute the highlighted example.com with your domain name):


```
DOMAIN='example.com' sudo -E bash -c 'cat /etc/letsencrypt/live/$DOMAIN/fullchain.pem /etc/letsencrypt/live/$DOMAIN/privkey.pem > /etc/haproxy/certs/$DOMAIN.pem'


```


Secure access to the combined file, which contains the private key, with this command:


```
sudo chmod -R go-rwx /etc/haproxy/certs


```


Now we’re ready to use the SSL cert and private key with HAProxy.


# Step 3 — Installing HAProxy


This step covers the installation of HAProxy. If it’s already installed on your server, skip this step.


Install HAProxy with yum:


```
sudo yum install haproxy


```


HAProxy is now installed but needs to be configured.


# Step 4 — Configuring HAProxy


This section will show you how to configure basic HAProxy with SSL setup. It also covers how to configure HAProxy to allow us to auto-renew our Let’s Encrypt certificate.


Open haproxy.cfg in a text editor:


```
sudo vi /etc/haproxy/haproxy.cfg


```


Keep this file open as we edit it in the next several sections.


## Global Section


Add this line to the global section to configure the maximum size of temporary DHE keys that are generated:


haproxy.cfg — 1 of 5
```
   tune.ssl.default-dh-param 2048

```


## Frontend Sections


Now we’re ready to define our frontend sections.



Note: The default HAProxy configuration includes a frontend and several backends. Feel free to delete them as we will not be using them.

The first thing we want to add is a frontend to handle incoming HTTP connections, and send them to a default backend (which we’ll define later). At the end of the file, let’s add a frontend called www-http. Be sure to replace haproxy_public_IP with the public IP address of your HAProxy server:


haproxy.cfg — 2 of 5
```
frontend www-http
   bind haproxy_www_public_IP:80
   reqadd X-Forwarded-Proto:\ http
   default_backend www-backend

```


Next, we will add a frontend to handle incoming HTTPS connections. At the end of the file, add a frontend called www-https. Be sure to replace haproxy_www_public_IP with the public IP of your HAProxy server. Also, you will need to replace example.com with your domain name (which should correspond to the certificate file you created earlier):


haproxy.cfg — 3 of 5
```
frontend www-https
   bind haproxy_www_public_IP:443 ssl crt /etc/haproxy/certs/example.com.pem
   reqadd X-Forwarded-Proto:\ https
   acl letsencrypt-acl path_beg /.well-known/acme-challenge/
   use_backend letsencrypt-backend if letsencrypt-acl
   default_backend www-backend

```


This frontend uses an ACL (letsencrypt-acl) to send Let’s Encrypt validation requests (for /.well-known/acme-challenge) to the letsencrypt-backend backend, which will enable us to renew the certificate without stopping the HAProxy service. All other requests will be forwarded to the www-backend, which is the backend that will serve our web application or site.


## Backend Sections


After you are finished configuring the frontends, add the www-backend backend by adding the following lines. Be sure to replace the highlighted words with the respective private IP addresses of your web servers (adjust the number of server lines to match how many backend servers you have):


haproxy.cfg — 4 of 5
```
backend www-backend
   redirect scheme https if !{ ssl_fc }
   server www-1 www_1_private_IP:80 check
   server www-2 www_2_private_IP:80 check

```


Any traffic that this backend receives will be balanced across its server entries, over HTTP (port 80).


Lastly, add the letsencrypt-backend backend, by adding these lines


haproxy.cfg — 5 of 5
```
backend letsencrypt-backend
   server letsencrypt 127.0.0.1:54321

```


This backend, which only handles Let’s Encrypt ACME challenges that are used for certificate requests and renewals, sends traffic to the localhost on port 54321. We’ll use this port instead of 80 and 443 when we renew our Let’s Encrypt SSL certificate.


Now we’re ready to start HAProxy:


```
sudo systemctl start haproxy


```



Note: If you’re having trouble with the haproxy.cfg configuration file, check out this GitHub Gist for an example.

The Let’s Encrypt TLS/SSL certificate is now in place, and we’re ready to set up the auto-renewal script. At this point, you should test that the TLS/SSL certificate works by visiting your domain in a web browser.


# Step 5 — Setting Up Auto Renewal


Let’s Encrypt certificates are valid for just 90 days, so it’s important to automate the renewal process.


A practical way to ensure your certificates won’t get outdated is to create a cron job that will automatically handle the renewal process for you. The cronjob will run certbot daily and renew the certificates if they’re within thirty days of expiring. certbot will also run a special renew-hook script after any successful renewal. We’ll use this renewal script to update our combined .pem file and reload haproxy.


Let’s create that script now, then test it.


## Create a Renewal Script


Open up a new file in /usr/local/bin as root:


```
sudo vi /usr/local/bin/renew.sh


```


This will be a new blank text file. Paste in the following short script, being sure to update the highlighted domain name with your own:


```
#!/bin/sh

SITE=example.com

# move to the correct let's encrypt directory
cd /etc/letsencrypt/live/$SITE

# cat files to make combined .pem for haproxy
cat fullchain.pem privkey.pem > /etc/haproxy/certs/$SITE.pem

# reload haproxy
systemctl reload haproxy

```


Save and close the file. This script moves into the correct Let’s Encrypt directory, runs the cat command to concatenate the two .pem files into one, then reloads haproxy.


Next, make the script executable:


```
sudo chmod u+x /usr/local/bin/renew.sh


```


Then run the script:


```
sudo /usr/local/bin/renew.sh


```


It should run without error. Next we’ll update Certbot and configure it to run this renewal script.


## Update certbot Configs


The certbot renew command that we’ll use to renew our certificates reads a config file that was created the first time we ran certbot. We need to open this file and update the port that certbot uses to run its standalone http server so it does’t conflict with haproxy (which is already listening on ports 80 and 443). Open the config file in a text editor:


```
sudo vi /etc/letsencrypt/renewal/example.com.conf


```


We need to change the http01_port line, so it reads like this:


example.com.conf
```
http01_port = 54321

```


Save and close the file. Now test the renewal process, specifying --dry-run so we don’t actually renew anything:


```
sudo certbot renew --dry-run


```


Certbot will listen on port 54321 for the renewal challenge, and haproxy will proxy the request from port 80 to 54321.


## Create a Cron Job


Next, we will edit the crontab to create a new job that will run the certbot renew command every day. To edit the crontab for the root user, run:


```
sudo crontab -e


```


Add the following to the bottom of the file:


crontab entry
```
30 2 * * * /usr/bin/certbot renew --renew-hook "/usr/local/bin/renew.sh" >> /var/log/le-renewal.log

```


Save and exit. This will create a new cron job that will execute the certbot renew command every day at 2:30 am. The output produced by the command will be piped to a log file located at /var/log/le-renewal.log. If the certificate is actually renewed, the --renew-hook script will run to create the combined PEM file and reload haproxy.


# Conclusion


That’s it! HAProxy is now using a free Let’s Encrypt TLS/SSL certificate to securely serve HTTPS traffic.


