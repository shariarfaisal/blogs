# How To Install Plausible Analytics on Ubuntu 22 04

```Docker``` ```Ubuntu``` ```Ubuntu 22.04```

## Introduction


Plausible Analytics is an open-source, self-hosted web analytics application written in Elixir that focuses on simplicity and privacy. It stores data about your website’s visitors in PostgreSQL and ClickHouse databases.


In this tutorial you will install Plausible using Docker Compose, then install Nginx to act as a reverse proxy for the Plausible app. Finally, you will enable secure HTTPS connections by using Certbot to download and configure SSL certificates from the Let’s Encrypt Certificate Authority.


# Prerequisites


In order to complete this tutorial, you’ll first need the following:


- An Ubuntu 22.04 server, with the UFW firewall enabled. Please read our Initial Server Setup with Ubuntu 22.04 to learn more about setting up these requirements
- Docker installed. You can use Step 1 of How To Install and Use Docker on Ubuntu 22.04 to get this done. Optionally, you may follow Step 2 of that tutorial if you want your non-root user to be able to run docker commands without using sudo
- Docker Compose installed. Follow Step 1 of How To Install and Use Docker Compose on Ubuntu 22.04 to install this software


Note: These prerequisite steps can be skipped if you’re using DigitalOcean’s One-Click Docker Image. This image will have Docker, Docker Compose, and UFW already installed and configured.
Launch a new Docker image in the region of your choice, then log in as the root user and proceed with the tutorial. Optionally, you could leave off the sudo parts of all commands, but it’s not necessary.

Finally, to enable SSL you’ll need a domain name pointed at your server’s public IP address. This should be something like example.com or plausible.example.com, for instance. If you’re using DigitalOcean, please see our DNS Quickstart for information on creating domain resources in our control panel.


When you’ve satisfied all the prerequisites, proceed to Step 1, where you’ll download and launch the Plausible software.


# Step 1 – Installing Plausible Analytics with Docker Compose


Plausible has created a Git repository with all the configuration files needed to self-host the software. Your first step will be to clone this repo to your server, update two configuration files, and then start the Plausible app and database containers.


Log into your server now.


First, use the cd command to navigate into the /opt directory:


```
cd /opt


```


Then use the git command to clone the repo from GitHub into a new directory within /opt called plausible:


```
sudo git clone https://github.com/plausible/hosting plausible


```


This will pull all the necessary configuration files into /opt/plausible. Move into the newly created directory:


```
cd plausible


```


The first file we need to edit is plausible-conf.env, a file that has a few configuration variables we need to set.


Before you open the file to edit it, generate a new random hash:


```
openssl rand 64 | base64 -w 0 ; echo


```


This uses the openssl command to generate 64 random characters, and the base64 command to base64 encode them. Copy the output to your clipboard, then open the configuration file:


```
sudo nano plausible-conf.env


```


The file contains five variables that you’ll need to fill in:


plausible-conf.env
```
ADMIN_USER_EMAIL=your_email_here
ADMIN_USER_NAME=admin_username
ADMIN_USER_PWD=admin_password
BASE_URL=https://your_domain_here
SECRET_KEY_BASE=paste_your_random_characters_here

```


Fill in the email, username, password, and base URL, then paste in the random characters you generated with openssl.



Note: The password you specify here must be at least six characters long. If you are using a bare IP address and not a domain name, make sure to precede it with http://.

Save the file (CTRL+O then ENTER in nano) and close out of your editor (CTRL+X).


There are more configuration options you can add to this file, but this minimal set will get you up and running. More information on configuring Plausible through plausible-conf.env can be found in the official Plausible Analytics self-hosting documentation.


Now you need to update the docker-compose.yml file. This file is what the docker-compose command uses to configure and launch multiple Docker containers. We need to change one option in this file: the IP that Plausible binds to.


```
sudo nano docker-compose.yml


```


Find the section that defines the Plausible container. It will start with plausible:. In that section find the ports: definition and update it to the following:


docker-compose.yml
```
    ports:
      - 127.0.0.1:8000:8000

```


This ensures that Plausible is only listening on the localhost interface, and is not publicly available. Even though you have a UFW firewall set up, due to some quirks in how Docker networking works, if you didn’t take this step your Plausible container would be accessible to the public on port 8000, and we only want it accessible through the Nginx proxy you’ll set up in the next step.


Save and close the docker-compose.yml file, then use docker-compose to download, configure, and launch the containers:


```
sudo docker compose up --detach


```


The --detach flag tells docker-compose to create the containers in the background, detached from our terminal session:


```
Output. . .
Starting plausible_plausible_events_db_1 ... done
Starting plausible_plausible_db_1        ... done
Starting plausible_mail_1                ... done
Starting plausible_plausible_1           ... done

```


The app container and all of its supporting mail and database containers should now be running. You can verify this by using the curl command to fetch the homepage of your new Plausible container running on localhost:


```
curl http://localhost:8000


```


```
Output<html><body>You are being <a href="/login">redirected</a>.</body></html>

```


If some HTML is output to your terminal, you know the server is up and running.


Next, we’ll set up Nginx to reverse proxy Plausible from localhost:8000 to the public.


# Step 2 — Installing and Configuring Nginx


Putting a web server such as Nginx in front of your elixir server can improve performance by offloading caching, compression, and static file serving to a more efficient process. We’re going to install Nginx and configure it to reverse proxy requests to Plausible, meaning it will take care of handing requests from your users to Plausible and back again.


First, refresh your package list, then install Nginx using apt:


```
sudo apt update
sudo apt install nginx


```


Allow public traffic to ports 80 and 443 (HTTP and HTTPS) using the “Nginx Full” UFW application profile:


```
sudo ufw allow "Nginx Full"


```


```
OutputRule added
Rule added (v6)

```


Next, open up a new Nginx configuration file in the /etc/nginx/sites-available directory. We’ll call ours plausible.conf but you could use a different name:


```
sudo nano /etc/nginx/sites-available/plausible.conf


```


Paste the following into the new configuration file, being sure to replace your_domain_here with the domain that you’ve configured to point to your Plausible server. This should be something like plausible.example.com, for instance:


/etc/nginx/sites-available/plausible.conf
```
server {
    listen       80;
    listen       [::]:80;
    server_name  your_domain_here;

    access_log  /var/log/nginx/plausible.access.log;
    error_log   /var/log/nginx/plausible.error.log;

    location / {
      proxy_pass http://localhost:8000;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }
}

```


This configuration is HTTP-only for now, as we’ll let Certbot take care of configuring SSL in the next step. The rest of the config sets up logging locations and then passes all traffic along to http://localhost:8000, the Plausible instance we started up in the previous step.


Save and close the file, then enable the configuration by linking it into /etc/nginx/sites-enabled/:


```
sudo ln -s /etc/nginx/sites-available/plausible.conf /etc/nginx/sites-enabled/


```


Use nginx -t to verify that the configuration file syntax is correct:


```
sudo nginx -t


```


```
Outputnginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

```


And finally, reload the nginx service to pick up the new configuration:


```
sudo systemctl reload nginx


```


Your Plausible site should now be available on plain HTTP. Load http://your_domain_here and it will look like this:





Now that you have your site up and running over HTTP, it’s time to secure the connection with Certbot and Let’s Encrypt certificates.


# Step 3 — Installing Certbot and Setting Up SSL Certificates


Thanks to Certbot and the Let’s Encrypt free certificate authority, adding SSL encryption to our Plausible app will take only two commands.


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
OutputCongratulations! You have successfully enabled https://plausible.example.com

You should test your configuration at:
https://www.ssllabs.com/ssltest/analyze.html?d=plausible.example.com
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/plausible.example.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/plausible.example.com/privkey.pem
   Your cert will expire on 2022-12-05. To obtain a new or tweaked
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


Your site is now secure and it’s safe to log in with the default user details you set up in Step 1. You will then be prompted to verify your registration, and a verification code will be emailed to the address you configured.


By default this email is sent directly from your server, which can create issues due to various spam-prevention measures. If you don’t get the email, check your spam folder. If it’s not there as well, you may need to set up more suitable SMTP details in the plausible-conf.env file. See the official Plausible self-hosting documentation for details on mail configuration.


When you successfully log in, you’ll see a prompt to get your first website set up with Plausible:





You have successfully installed and secured your Plausible analytics software.


# Conclusion


In this tutorial, you launched the Plausible Analytics app and its associated helper containers using Docker Compose, then set up an Nginx reverse proxy and secured it using Let’s Encrypt SSL certificates.


You’re now ready to set up your website and add the Plausible Analytics tracking script. Please see the official Plausible Analytics documentation for further information on using the software and setting up your site.


