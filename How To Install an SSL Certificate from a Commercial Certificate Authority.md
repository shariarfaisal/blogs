# How To Install an SSL Certificate from a Commercial Certificate Authority

```Apache``` ```Security``` ```Nginx```

## Introduction


This tutorial will show you how to acquire and install an SSL certificate from a trusted, commercial Certificate Authority (CA). SSL certificates allow web servers to encrypt their traffic, and also offer a mechanism to validate server identities to their visitors. Websites using SSL are accessed via the https:// protocol.


Before the mid-2010s, many smaller websites did not always use SSL or HTTPS. Since then, expectations of security have increased, and the Let’s Encrypt project was created to provide free, trusted SSL certificates at scale, allowing almost everyone to use HTTPS as needed.


However, there are some limitations to Let’s Encrypt’s certificates. They expire every 3 months, typically requiring you to have a functioning auto-renewal script in place, and can be awkward to use in environments where this is not possible. Let’s Encrypt also does not provide Extended Validation certificates which validate the legal ownership of your web presence, or Wildcard Certificates that will automatically match every possible subdomain of your website (such as shop.example.com) without you having to register each of them manually.


For most users, these will not be significant limitations. Let’s Encrypt is a popular option for many personal and commercial websites. However, if you have particular enterprise software requirements, or a very large commercial operation, you should consider purchasing a certificate from a commercial CA.


This tutorial covers how to select and deploy an SSL certificate from a trusted certificate authority. After you have acquired your SSL certificate, this tutorial will cover installing it on the Nginx and Apache web servers.


## Prerequisites


There are several prerequisites to attempting to obtain an SSL certificate from a commercial CA:


- 
A registered domain name. This tutorial will use example.com throughout. You can purchase a domain name from Namecheap, get one for free with Freenom, or use the domain registrar of your choice.

- 
Access to one of the email addresses on your domain’s WHOIS record or to an “admin type” email address at the domain itself. Certificate authorities that issue SSL certificates will typically validate domain control by sending a validation email to one of the addresses on the domain’s WHOIS record, or to a generic admin email address at the domain itself. To be issued an Extended Validation certificate, you will also be required to provide the CA with paperwork to establish the legal identity of the website’s owner, among other things.

- 
DNS records set up for your server. If you are using DigitalOcean, please see our DNS documentation for details on how to add them.


This tutorial will provide configuration instructions for a Ubuntu 22.04 server set up by following this initial server setup for Ubuntu 22.04 tutorial, including a sudo-enabled non-root user and a firewall. Most modern Linux flavors will work similarly.


You should also have a web server like Nginx or Apache installed, following How To Install Nginx on Ubuntu 22.04 or How To Install the Apache Web Server on Ubuntu 22.04. Be sure that you have a server block (or Apache virtual host) for your domain.


# Step 1 – Choosing Your Certificate Authority


If you are not sure which Certificate Authority to use, there are a few factors to consider.


## Root Certificate Program Memberships


The most crucial point is that the CA that you choose is a member of the root certificate programs of the most commonly used operating systems and web browsers, i.e. it is a “trusted” CA, and its root certificate is trusted by common browsers and other software. If your website’s SSL certificate is signed by a trusted CA, its identity is considered to be valid by software that trusts the CA.


Most commercial CAs that you will encounter will be members of the common root CA programs, but it does not hurt to check before making your certificate purchase. For example, Apple publishes its list of trusted SSL root certificates.


## Certificate Types


Ensure that you choose a CA that offers the certificate type that you require. Many CAs offer variations of these certificate types under a variety of names and pricing structures. Here is a short description of each type:


- Single Domain: Used for a single domain, e.g. example.com. Note that additional subdomains, such as www.example.com, are not included
- Wildcard: Used for a domain and any of its subdomains. For example, a wildcard certificate for *.example.com can also be used for www.example.com and store.example.com
- Multiple Domain: Known as a SAN or UC certificate, these can be used with multiple domains and subdomains that are added to the Subject Alternative Name field. For example, a single multi-domain certificate could be used with example.com, www.example.com, and example.net

In addition to the aforementioned certificate types, there are different levels of validations that CAs offer:


- Domain Validation (DV): DV certificates are issued after the CA validates that the requestor owns or controls the domain in question
- Organization Validation (OV): OV certificates can be issued only after the issuing CA validates the legal identity of the requestor
- Extended Validation (EV): EV certificates can be issued only after the issuing CA validates the legal identity, among other things, of the requestor, according to a strict set of guidelines. The purpose of this type of certificate is to provide additional assurance of the legitimacy of your organization’s identity to your site’s visitors. EV certificates can be single or multiple domain, but not wildcard

## Additional Features


Many CAs offer a large variety of “bonus” features to differentiate themselves from the rest of the SSL certificate-issuing vendors. Some of these features can end up saving you money, so it is important that you weigh your needs against the offerings before making a purchase. Example of features to look out for include free certificate reissues or a single domain-priced certificate that works for www. and the domain basename, e.g. www.example.com with a SAN of example.com


# Step 2 – Generating a CSR and Private Key


After you have your prerequisites sorted, and you know the type of certificate you need, it’s time to generate a certificate signing request (CSR) and private key.


If you are planning on using Apache HTTP or Nginx as your web server, you can use the openssl command to generate your private key and CSR on your web server. In this tutorial, you can keep all of the relevant files in your home directory, but feel free to store them in any secure location on your server:


To generate a private key, called example.com.key, and a CSR, called example.com.csr, run this command (replace the example.com with the name of your domain):


```
openssl req -newkey rsa:2048 -nodes -keyout example.com.key -out example.com.csr


```


At this point, you will be prompted for several lines of information that will be included in your certificate request. The most important part is the Common Name field, which should match the name that you want to use your certificate with – for example, example.com, www.example.com, or (for a wildcard certificate request) *.example.com. If you are planning on getting an OV or EV certificate, ensure that all of the other fields accurately reflect your organization or business details. Providing a “challenge password” is not necessary.


For example:


```
OutputCountry Name (2 letter code) [AU]:US
State or Province Name (full name) [Some-State]:New York
Locality Name (eg, city) []:New York
Organization Name (eg, company) [Internet Widgits Pty Ltd]:My Company
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:example.com
Email Address []:sammy@example.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:

```


This will generate a .key and .csr file. The .key file is your private key, and should be kept secure. The .csr file is what you will send to the CA to request your SSL certificate.


```
ls example.com*


```


```
Outputexample.com.csr  example.com.key

```


You will need to copy and paste your CSR when submitting your certificate request to your CA.  To print the contents of your CSR, use cat:


```
cat example.com.csr

```


Now you are ready to buy a certificate from a CA.


# Step 3 – Purchasing and Obtaining a Certificate


There are many commercial CA providers, and you can compare and contrast the most appropriate options for your own setup. For example, Namecheap acts as an SSL certificate reseller, and has changed upstream CA providers in the past to provide the best value. Currently, they offer certificates from Comodo CA. Here is a sample of their offerings as of December 2022:





After making a selection, you will need to upload the CSR that you generated in the previous step. Your CA provider will also likely have an “Approver” step, which will send a validation request email to an address in your domain’s WHOIS record or to an administrator type address of the domain that you are getting a certificate for.


After approving the certificate, the certificate will be emailed to the named administrator. Copy and save them to your server in the same location that you generated your private key and CSR. Name the certificate with the domain name and a .crt extension, e.g. example.com.crt, and name the intermediate certificate intermediate.crt.


The certificate is now ready to be installed on your web server, but first, you may have to make some changes to your firewall.


# Step 4 – Updating your Firewall to Allow HTTPS


If you have the ufw firewall enabled as recommended by our Ubuntu 22.04 setup guide, you’ll need to adjust the settings to allow for HTTPS traffic. Nginx and Apache both register a few profiles with ufw upon installation.


You can see the current setting by typing:


```
sudo ufw status


```


If you receive output containing just Nginx HTTP or Apache, only HTTP traffic is allowed to the web server:


```
OutputStatus: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere                  
Nginx HTTP                 ALLOW       Anywhere                  
OpenSSH (v6)               ALLOW       Anywhere (v6)             
Nginx HTTP (v6)            ALLOW       Anywhere (v6)

```


To additionally let in HTTPS traffic, allow the Nginx Full or Apache Full` profile and delete the redundant HTTP profile allowance:


```
sudo ufw allow 'Nginx Full'
sudo ufw delete allow 'Nginx HTTP'


```


That should produce a result like this:


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


In the final step, you’ll install the certificate.


# Step 5 – Installing a Certificate On Your Server


After acquiring your certificate from the CA of your choice, you need to install it on your web server. This involves adding a few SSL-related lines to your web server software configuration.


This tutorial will cover configuring Nginx and Apache on Ubuntu 22.04, but most modern Linux flavors will work similarly. This tutorial also makes these assumptions:


- The private key, SSL certificate, and, if applicable, the CA’s intermediate certificates are located in a home directory, at /home/sammy
- The private key is called example.com.key
- The SSL certificate is called example.com.crt
- The CA intermediate certificate(s) returned by your provider are in a file called intermediate.crt


Note: In a production environment, these files should be stored somewhere that only the web server process (usually root) can access, and the private key should be kept secure. For example, Let’s Encrypt stores the certificates it generates in /etc/letsencrypt. Production examples will vary due to the complexity of multi-server configurations.

## Nginx


These are the steps to manually deploy an SSL certificate on Nginx.


If your CA returned only an intermediate certificate, you must create a single “chained” certificate file that contains your certificate and the CA’s intermediate certificates.


Assuming your certificate file is called example.com.crt, you can use the cat command to append files together to create a combined file called example.com.chained.crt:


```
cat example.com.crt intermediate.crt > example.com.chained.crt


```


Using nano or your favorite text editor, open your default Nginx server block file for editing:


```
sudo nano /etc/nginx/sites-enabled/default


```


Find the listen directive, and modify it to listen 443 ssl:


/etc/nginx/sites-enabled/default
```
…
server {
    listen 443 ssl;
…

```


Next, find the server_name directive within that same server block, and make sure that its value matches the common name of your certificate. Also, add the ssl_certificate and ssl_certificate_key directives to specify the paths of your certificate and private key files:


/etc/nginx/sites-enabled/default
```
…
    server_name example.com;
    ssl_certificate /home/sammy/example.com.chained.crt;
    ssl_certificate_key /home/sammy/example.com.key;
…

```


To allow only the most secure SSL protocols and ciphers, add the following lines to the file:


/etc/nginx/sites-enabled/default
```
…
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    ssl_ciphers 'EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH';
…

```


Finally, to redirect HTTP requests to HTTPS by default, you can add an additional server block at the top of the file:


/etc/nginx/sites-enabled/default
```
server {
    listen 80;
    server_name example.com;
    rewrite ^/(.*) https://example.com/$1 permanent;
}
…

```


Save and close the file. If you are using nano, press Ctrl+X, then when prompted, Y and then Enter.


Before restarting Nginx, you can validate your configuration by using nginx -t:


```
sudo nginx -t


```


If there aren’t any problems, restart Nginx to enable SSL over HTTPS:


```
sudo systemctl restart nginx


```


Test it out by accessing your site via HTTPS, e.g. https://example.com. You will also want to try connecting via HTTP, e.g. http://example.com to ensure that the redirect is working properly.


## Apache


These are the steps to manually deploy an SSL certificate on Apache.


Using nano or your favorite text editor, open your default Apache virtual host file for editing:


```
sudo nano /etc/apache2/sites-available/000-default.conf


```


Find the <VirtualHost *:80> entry and modify it so your web server will listen on port 443:


/etc/apache2/sites-available/000-default.conf
```
…
<VirtualHost *:443>
…

```


Next, add the ServerName directive, if it doesn’t already exist:


/etc/apache2/sites-available/000-default.conf
```
…
ServerName example.com
…

```


Then add the following lines to specify your certificate and key paths:


/etc/apache2/sites-available/000-default.conf
```
…
SSLEngine on
SSLCertificateFile /home/sammy/example.com.crt
SSLCertificateKeyFile /home/sammy/example.com.key
SSLCACertificateFile /home/sammy/intermediate.crt
…

```


At this point, your server is configured to listen on HTTPS only (port 443), so requests to HTTP (port 80) will not be served. To redirect HTTP requests to HTTPS, add the following to the top of the file (substitute the name in both places):


/etc/apache2/sites-available/000-default.conf
```
<VirtualHost *:80>
   ServerName example.com
   Redirect permanent / https://example.com/
</VirtualHost>
…

```


Save and close the file. If you are using nano, press Ctrl+X, then when prompted, Y and then Enter.


Enable the Apache SSL module by running this command:


```
sudo a2enmod ssl


```


Now, restart Apache to load the new configuration and enable TLS/SSL over HTTPS.


```
sudo systemctl restart apache2


```


Test it out by accessing your site via HTTPS, e.g. https://example.com. You will also want to try connecting via HTTP, e.g. http://example.com to ensure that the redirect is working properly.


# Conclusion


In this tutorial, you learned how to determine when you might need to purchase an SSL certificate from a commercial CA, and how to compare and contrast the available options. You also learned how to configure Nginx or Apache for HTTPS support, and how to adapt their configurations for production.


Next, you may want to read about other SSL use cases, such as when working with load balancers.


