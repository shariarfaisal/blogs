# How To Set Up the code-server Cloud IDE Platform on Ubuntu 18 04 [Quickstart]

```Cloud Computing``` ```Ubuntu 18.04``` ```Let's Encrypt``` ```Nginx``` ```Quickstart``` ```VS Code```

## Introduction


code-server is Microsoft Visual Studio Code running on a remote server and accessible directly from your browser. This means that you can use various devices, running different operating systems, and always have a consistent development environment on hand.


In this tutorial, you will set up the code-server cloud IDE platform on your Ubuntu 18.04 machine and expose it at your domain, secured with Let’s Encrypt. For a more detailed version of this tutorial, please refer to How To Set Up the code-server Cloud IDE Platform on Ubuntu 18.04.


# Prerequisites


- 
A server running Ubuntu 18.04 with at least 2GB of RAM, root access, and a sudo, non-root account. You can set this up by following the Initial Server Setup Guide for Ubuntu 18.04.

- 
Nginx installed on your server. For a guide on how to do this, complete Steps 1 to 4 of How To Install Nginx on Ubuntu 18.04.

- 
A fully registered domain name to host code-server, pointed to your server. This tutorial will use code-server.your-domain throughout. You can purchase a domain name on Namecheap, get one for free on Freenom, or use the domain registrar of your choice.

- 
Both of the following DNS records set up for your server. You can follow this introduction to DigitalOcean DNS for details on how to add them.

An A record with your-domain pointing to your server’s public IP address.
An A record with www.your-domain pointing to your server’s public IP address.


- An A record with your-domain pointing to your server’s public IP address.
- An A record with www.your-domain pointing to your server’s public IP address.

# Step 1 — Installing code-server


Create the directory to store all data for code-server:


```
mkdir ~/code-server


```


Navigate to it:


```
cd ~/code-server


```


Visit the Github releases page of code-server and pick the latest Linux build. Download it using:


```
wget https://github.com/cdr/code-server/releases/download/2.1692-vsc1.39.2/code-server2.1692-vsc1.39.2-linux-x86_64.tar.gz


```


Unpack the archive:


```
tar -xzvf code-server2.1692-vsc1.39.2-linux-x86_64.tar.gz


```


Navigate to the directory containing the code-server executable:


```
cd code-server2.1692-vsc1.39.2-linux-x86_64


```


To access the code-server executable across your system, copy it with:


```
sudo cp code-server /usr/local/bin


```


Create a folder for code-server to store user data:


```
sudo mkdir /var/lib/code-server


```


Create a systemd service, code-server.service, in the /lib/systemd/system directory:


```
sudo nano /lib/systemd/system/code-server.service


```


Add the following lines:


/lib/systemd/system/code-server.service
```
[Unit]
Description=code-server
After=nginx.service

[Service]
Type=simple
Environment=PASSWORD=your_password
ExecStart=/usr/local/bin/code-server --host 127.0.0.1 --user-data-dir /var/lib/code-server --auth password
Restart=always

[Install]
WantedBy=multi-user.target

```


- --host 127.0.0.1 binds it to localhost.
- --user-data-dir /var/lib/code-server sets its user data directory.
- --auth password specifies that it should authenticate visitors with a password.

Remember to replace your_password with your desired password.


Save and close the file.


Start the code-server service:


```
sudo systemctl start code-server


```


Check that it’s started correctly:


```
sudo systemctl status code-server


```


You’ll see output similar to:


```
Output● code-server.service - code-server
   Loaded: loaded (/lib/systemd/system/code-server.service; disabled; vendor preset: enabled)
   Active: active (running) since Mon 2019-12-09 20:07:28 UTC; 4s ago
 Main PID: 5216 (code-server)
    Tasks: 23 (limit: 2362)
   CGroup: /system.slice/code-server.service
           ├─5216 /usr/local/bin/code-server --host 127.0.0.1 --user-data-dir /var/lib/code-server --auth password
           └─5240 /usr/local/bin/code-server --host 127.0.0.1 --user-data-dir /var/lib/code-server --auth password
...

```


Enable the code-server service to start automatically after a server reboot:


```
sudo systemctl enable code-server


```


# Step 2 — Exposing code-server


Now you will configure Nginx as a reverse proxy for code-server.


Create code-server.conf to store the configuration for exposing code-server at your domain:


```
sudo nano /etc/nginx/sites-available/code-server.conf


```


Add the following lines to set up your server block with the necessary directives:


/etc/nginx/sites-available/code-server.conf
```
server {
	listen 80;
	listen [::]:80;

	server_name code-server.your_domain;

	location / {
		proxy_pass http://localhost:8080/;
		proxy_set_header Upgrade $http_upgrade;
		proxy_set_header Connection upgrade;
		proxy_set_header Accept-Encoding gzip;
	}
}

```


Replace code-server.your_domain with your desired domain, then save and close the file.


To make this site configuration active, create a symlink of it:


```
sudo ln -s /etc/nginx/sites-available/code-server.conf /etc/nginx/sites-enabled/code-server.conf


```


Test the validity of the configuration:


```
sudo nginx -t


```


You’ll see the following output:


```
Outputnginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

```


For the configuration to take effect, restart Nginx:


```
sudo systemctl restart nginx


```


# Step 3 — Securing Your Domain


Now you’ll secure your domain using a Let’s Encrypt TLS certificate.


Add the Certbot package repository to your server:


```
sudo add-apt-repository ppa:certbot/certbot


```


Install Certbot and its Nginx plugin:


```
sudo apt install python-certbot-nginx


```


Configure ufw to accept encrypted traffic:


```
sudo ufw allow https


```


The output will be:


```
OutputRule added
Rule added (v6)

```


Reload it for the configuration to take effect:


```
sudo ufw reload


```


The output will show:


```
OutputFirewall reloaded

```


Navigate to your code-server domain.





Enter your code-server password. You’ll see the interface exposed at your domain.





To secure it, install a Let’s Encrypt TLS certificate using Certbot.


Request a certificate for your domain with:


```
sudo certbot --nginx -d code-server.your_domain


```


Provide an email address for urgent notices, accept the EFF’s Terms of Service, and decide whether to redirect all HTTP traffic to HTTPS.


The output will be similar to this:


```
OutputIMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/code-server.your_domain/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/code-server.your_domain/privkey.pem
   Your cert will expire on ... To obtain a new or tweaked
   version of this certificate in the future, simply run certbot again
   with the "certonly" option. To non-interactively renew *all* of
   your certificates, run "certbot renew"
...

```


Certbot has successfully generated TLS certificates and applied them to the Nginx configuration for your domain.


# Conclusion


You now have code-server, a versatile cloud IDE, installed on your Ubuntu 18.04 server, exposed at your domain and secured using Let’s Encrypt certificates. For further information, see the Visual Studio Code documentation for additional features and detailed instructions on other components of code-server.


