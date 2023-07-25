# How To Create a Self-Signed SSL Certificate for Nginx in Ubuntu 18 04

```Security``` ```Ubuntu``` ```Nginx``` ```Ubuntu 18.04```

## Introduction


TLS, or transport layer security, and its predecessor SSL, which stands for secure sockets layer, are web protocols used to wrap normal traffic in a protected, encrypted wrapper.


Using this technology, servers can send traffic safely between the server and clients without the possibility of the messages being intercepted by outside parties. The certificate system also assists users in verifying the identity of the sites that they are connecting with.


In this guide, we will show you how to set up a self-signed SSL certificate for use with an Nginx web server on an Ubuntu 18.04 server.



Note: A self-signed certificate will encrypt communication between your server and any clients. However, because it is not signed by any of the trusted Certificate Authorities (CA) included with web browsers, users cannot use the certificate to validate the identity of your server automatically.
A self-signed certificate may be appropriate if you do not have a domain name associated with your server and for instances where the encrypted web interface is not user-facing. If you do have a domain name, in many cases it is better to use a CA-signed certificate. You can find out how to set up a free trusted certificate with the Let’s Encrypt project here.

# Prerequisites


To follow this tutorial, you will need:


- One Ubuntu 18.04 server set up with a non-root user configured with sudo privileges and a firewall. You can learn how to set up such a user account by following our initial server setup for Ubuntu 18.04.
- You will also need to have the Nginx web server installed. If you would like to install an entire LEMP (Linux, Nginx, MySQL, PHP) stack on your server, you can follow our guide on setting up LEMP on Ubuntu 18.04.
- If you only want the Nginx web server, you can instead follow our guide on installing Nginx on Ubuntu 18.04.

When you have completed the prerequisites, continue to the first step.


# Step 1 — Creating the SSL Certificate


TLS/SSL works by using a combination of a public certificate and a private key. The SSL key is kept secret on the server. It is used to encrypt content sent to clients. The SSL certificate is publicly shared with anyone requesting the content. It can be used to decrypt the content signed by the associated SSL key.


You can create a self-signed key and certificate pair with OpenSSL in a single command:


```
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/nginx-selfsigned.key -out /etc/ssl/certs/nginx-selfsigned.crt


```


Here’s a breakdown of what each part of this command does:


- sudo: The sudo command allows members of the sudo group to temporarily elevate their privileges to that of another user (the superuser, or root user, by default). This is necessary in this case since we’re creating the certificate and key pair under the /etc/ directory, which can only be accessed by the root user or other privileged accounts.
- openssl: This is the basic command line tool for creating and managing OpenSSL certificates, keys, and other files.
- req: This subcommand specifies that we want to use X.509 certificate signing request (CSR) management. The X.509 is a public key infrastructure standard that SSL and TLS adheres to for its key and certificate management. We want to create a new X.509 cert, so we are using this subcommand.
- -x509: This further modifies the previous subcommand by telling the utility that we want to make a self-signed certificate instead of generating a certificate signing request, as would normally happen.
- -nodes: This tells OpenSSL to skip the option to secure our certificate with a passphrase. We need Nginx to be able to read the file, without user intervention, when the server starts up. A passphrase would prevent this from happening because we would have to enter it after every restart.
- -days 365: This option sets the length of time that the certificate will be considered valid. We set it for one year here.
- -newkey rsa:2048: This specifies that we want to generate a new certificate and a new key at the same time. We did not create the key that is required to sign the certificate in a previous step, so we need to create it along with the certificate. The rsa:2048 portion tells it to make an RSA key that is 2048 bits long.
- -keyout: This line tells OpenSSL where to place the generated private key file that we are creating.
- -out: This tells OpenSSL where to place the certificate that we are creating.

As stated previously, these options will create both a key file and a certificate. After running this command, you’ll be asked a few questions about your server in order to embed the information correctly in the certificate.


Fill out the prompts appropriately. The most important line is the one that requests the Common Name (e.g. server FQDN or YOUR name). You need to enter the domain name associated with your server or, more likely, your server’s public IP address.


The entirety of the prompts will look like the following:


```
OutputCountry Name (2 letter code) [AU]:US
State or Province Name (full name) [Some-State]:New York
Locality Name (eg, city) []:New York City
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Bouncy Castles, Inc.
Organizational Unit Name (eg, section) []:Ministry of Water Slides
Common Name (e.g. server FQDN or YOUR name) []:server_IP_address
Email Address []:admin@your_domain.com

```


Both of the files you created will be placed in the appropriate subdirectories of the /etc/ssl directory.


While using OpenSSL, you should also create a strong Diffie-Hellman group, which is used in negotiating Perfect Forward Secrecy with clients.


You can do this by running the following:


```
sudo openssl dhparam -out /etc/nginx/dhparam.pem 4096


```


This will take a while, but when it’s done you will have a strong DH group at /etc/nginx/dhparam.pem that will be used during configuration.


# Step 2 — Configuring Nginx to Use SSL


Now that your key and certificate files under the /etc/ssl directory have been created, you’ll need to modify your Nginx configuration to take advantage of them.


First, you will create a configuration snippet with the information about the SSL key and certificate file locations. Then, you will create a configuration snippet with a strong SSL setting that can be used with any certificates in the future. Finally, you will adjust your Nginx server blocks using the two configuration snippets you’ve created so that SSL requests can be handled appropriately.


This method of configuring Nginx will allow you to keep clean server blocks and put common configuration segments into reusable modules.


## Creating a Configuration Snippet Pointing to the SSL Key and Certificate


First, use your preferred text editor to create a new Nginx configuration snippet in the /etc/nginx/snippets directory. The following example uses nano:


To properly distinguish the purpose of this file, name it self-signed.conf:


```
sudo nano /etc/nginx/snippets/self-signed.conf


```


Within this file, set the ssl_certificate directive to your certificate file and the ssl_certificate_key to the associated key. This will look like the following:


/etc/nginx/snippets/self-signed.conf
```
ssl_certificate /etc/ssl/certs/nginx-selfsigned.crt;
ssl_certificate_key /etc/ssl/private/nginx-selfsigned.key;

```


Once you’ve added those lines, save the file and exit the editor. If you used nano, you can do this by pressing CTRL + X, then Y, and ENTER.


## Creating a Configuration Snippet with Strong Encryption Settings


Next, you will create another snippet that will define some SSL settings. This will set Nginx up with a strong SSL cipher suite and enable some advanced features that will help keep your server secure.


The parameters you set can be reused in future Nginx configurations, so you can give the file a generic name:


```
sudo nano /etc/nginx/snippets/ssl-params.conf


```


To set up Nginx SSL securely, we will adapt the recommendations from Cipherlist.eu. Cipherlist.eu is a useful and digestible resource for understanding encryption settings used for popular software.



Note: The suggested settings from Cipherlist.eu offer strong security. Sometimes, this comes at the cost of greater client compatibility. If you need to support older clients, there is an alternative list that can be accessed by clicking the link on the page labeled “Yes, give me a ciphersuite that works with legacy / old software.” If desired, you can substitute that list with the content of the next example code block.
The choice of which configuration to use will depend largely on what you need to support. They both will provide great security.

For our purposes, copy the provided settings in their entirety, but first, you will need to make a few small modifications.


First, add your preferred DNS resolver for upstream requests. We will use Google’s (8.8.8.8 and 8.8.4.4) for this guide.


Second, comment out the line that sets the strict transport security header. Before uncommenting this line, you should take a moment to read up on HTTP Strict Transport Security, or HSTS, and specifically about the “preload” functionality. Preloading HSTS provides increased security, but can also have far-reaching negative consequences if accidentally enabled or enabled incorrectly.


Add the following into your ssl-params.conf snippet file:


/etc/nginx/snippets/ssl-params.conf
```
ssl_protocols TLSv1.3;
ssl_prefer_server_ciphers on;
ssl_dhparam /etc/nginx/dhparam.pem; 
ssl_ciphers EECDH+AESGCM:EDH+AESGCM;
ssl_ecdh_curve secp384r1;
ssl_session_timeout  10m;
ssl_session_cache shared:SSL:10m;
ssl_session_tickets off;
ssl_stapling on;
ssl_stapling_verify on;
resolver 8.8.8.8 8.8.4.4 valid=300s;
resolver_timeout 5s;
# Disable strict transport security for now. You can uncomment the following
# line if you understand the implications.
#add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload";
add_header X-Frame-Options DENY;
add_header X-Content-Type-Options nosniff;
add_header X-XSS-Protection "1; mode=block";

```


Because you’re using a self-signed certificate, the SSL stapling will not be used. Nginx will output a warning, disable stapling for your self-signed certificate, but will then continue to operate correctly.


Save and close the file when you are finished.


## Adjusting the Nginx Configuration to Use SSL


Now that you have your snippets, you can adjust the Nginx configuration to enable SSL.


We will assume in this guide that you are using a custom server block configuration file in the /etc/nginx/sites-available directory. This guide also follows the conventions from the prerequisite Nginx tutorial and uses /etc/nginx/sites-available/your_domain for this example. Substitute your configuration filename as needed.


Before moving forward, back up your current configuration file:


```
sudo cp /etc/nginx/sites-available/your_domain /etc/nginx/sites-available/your_domain.bak


```


Now, open the configuration file to make adjustments:


```
sudo nano /etc/nginx/sites-available/your_domain


```


Inside, your server block probably begins similar to the following:


/etc/nginx/sites-available/your_domain.com
```
server {
    listen 80;
    listen [::]:80;

    server_name your_domain www.your_domain.com;

    root /var/www/your_domain.com/html;
    index index.html index.htm index.nginx-debian.html;

    . . .
}

```


Your file may be in a different order, and instead of the root and index directives, you may have some location, proxy_pass, or other custom configuration statements. This is fine since you only need to update the listen directives and include your SSL snippets. Then modify this existing server block to serve SSL traffic on port 443, and create a new server block to respond on port 80 and automatically redirect traffic to port 443.



Note: Use a 302 redirect until you have verified that everything is working properly. After, you’ll change this to a permanent 301 redirect.

In your existing configuration file, update the two listen statements to use port 443 and ssl, then include the two snippet files we created in previous steps:


/etc/nginx/sites-available/your_domain.com
```
server {
    listen 443 ssl;
    listen [::]:443 ssl;
    include snippets/self-signed.conf;
    include snippets/ssl-params.conf;

    server_name your_domain.com www.your_domain.com;

    root /var/www/your_domain.com/html;
    index index.html index.htm index.nginx-debian.html;

    . . .
}

```


Next, add a second server block into the configuration file after the closing bracket (}) of the first block:


/etc/nginx/sites-available/your_domain.com
```
. . .
server {
    listen 80;
    listen [::]:80;

    server_name your_domain.com www.your_domain.com;

    return 302 https://$server_name$request_uri;
}

```


This is a bare-bones configuration that listens on port 80 and performs the redirect to HTTPS. Save and close the file when you are finished editing it.


# Step 3 — Adjusting the Firewall


If you have the ufw firewall enabled, as recommended by the prerequisite guides, you’ll need to adjust the settings to allow for SSL traffic. Luckily, Nginx registers a few profiles with ufw upon installation.


You can review the available profiles by running the following:


```
sudo ufw app list


```


A list like the following will appear in the output:


```
OutputAvailable applications:
  Nginx Full
  Nginx HTTP
  Nginx HTTPS
  OpenSSH

```


You can also check the current setting by typing sudo ufw status:


```
sudo ufw status


```


It will probably generate the following output, meaning that only HTTP traffic is allowed to the web server:


```
OutputStatus: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
Nginx HTTP                 ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
Nginx HTTP (v6)            ALLOW       Anywhere (v6)

```


To allow HTTPS traffic, you can update permissions for the “Nginx Full” profile:


```
sudo ufw allow 'Nginx Full'


```


Then, delete the redundant “Nginx HTTP” profile allowance:


```
sudo ufw delete allow 'Nginx HTTP'


```


After running sudo ufw status, you should receive the following output:


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


This output confirms the adjustments to your firewall were successful and you are ready to enable the changes in Nginx.


# Step 4 — Enabling the Changes in Nginx


With the changes and adjustments to your firewall complete, you can restart Nginx to implement the new changes.


First, check that there are no syntax errors in our files. You can do this by running sudo nginx -t:


```
sudo nginx -t


```


If everything is successful, you will get a result that says the following:


```
Outputnginx: [warn] "ssl_stapling" ignored, issuer certificate not found for certificate "/etc/ssl/certs/nginx-selfsigned.crt"
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

```


Notice the warning in the beginning. As noted earlier, this particular setting generates a warning since your self-signed certificate can’t use SSL stapling. This is expected and your server can still encrypt connections correctly.


If your output matches our example, your configuration file should have no syntax errors. If this is the case, then you can safely restart Nginx to implement changes:


```
sudo systemctl restart nginx


```


Now that the system has been restarted with the new changes, you can proceed to testing.


# Step 5 — Testing Encryption


Now, you’re ready to test your SSL server.


Open your web browser and type https:// followed by your server’s domain name or IP into the address bar:


```
https://server_domain_or_IP

```


Depending on your browser, you will likely receive a warning since the certificate you created isn’t signed by one of your browser’s trusted certificate authorities:





This warning is expected and normal. We are only interested in the encryption aspect of our certificate, not the third-party validation of our host’s authenticity. Click “ADVANCED” and then the link provided to proceed to your host:





At this point, you should be taken to your site. In our example, the browser address bar shows a lock with an “x” over it, which means that the certificate cannot be validated. It is still encrypting your connection. Note that this icon may differ, depending on your browser.


If you configured Nginx with two server blocks, automatically redirecting HTTP content to HTTPS, you can also check whether the redirect functions correctly:


```
http://server_domain_or_IP

```


If this results in the same icon, this means that your redirect worked correctly.


# Step 6 — Changing to a Permanent Redirect


If your redirect worked correctly and you are sure you want to allow only encrypted traffic, you should modify the Nginx configuration to make the redirect permanent.


Open your server block configuration file again:


```
sudo nano /etc/nginx/sites-available/your_domain.com


```


Find the return 302 and change it to return 301:


/etc/nginx/sites-available/your_domain.com
```
	return 301 https://$server_name$request_uri;

```


Save and close the file.


Check your configuration for syntax errors:


```
sudo nginx -t


```


When you’re ready, restart Nginx to make the redirect permanent:


```
sudo systemctl restart nginx


```


After the restart, the changes will be implemented and your redirect is now permanent.


# Conclusion


You have configured your Nginx server to use strong encryption for client connections. This will allow you to serve requests securely, and prevent outside parties from reading your traffic. Alternatively, you may choose to use a self-signed SSL certificate that can be obtained from Let’s Encrypt, a certificate authority that installs free TLS/SSL certificates and enables encrypted HTTPS on a web server. Learn more from our tutorial on How To Secure Nginx with Let’s Encrypt on Ubuntu 18.04.


