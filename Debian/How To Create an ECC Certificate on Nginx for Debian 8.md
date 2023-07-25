# How To Create an ECC Certificate on Nginx for Debian 8

```Security``` ```Nginx``` ```Debian```

## Introduction


This article explains how to create an Elliptic Curve Cryptography (ECC) SSL certificate for Nginx. By the end of this tutorial, you will have a faster encryption mechanism for production use.


Traditional public-key cryptography relies on the near-impossibility of factoring large integers. On the other hand, ECC relies on the impossibility of resolving random elliptic curves into discrete logarithmic functions, a problem that’s called the “elliptic curve discrete logarithm problem” or ECDLP. In short, ECC offers smaller keys with similar security, and this in turn translates into higher encryption performance, applicable to digital signatures like SSL.


This tutorial, and all ECC certificates, depends on an elliptic-curve protocol which can come in several flavors. The National Institute of Standards and Technology (NIST) Suite B specifies two potential elliptical curves for use, P-256 and P-384, otherwise known as prime256v1 and secp384r1. For simplicity, we will use the former, prime256v1, as it is simple but practical.


# Prerequisites


To follow this tutorial, you will need:


- One fresh Debian 8.1 Droplet
- A sudo non-root user, which you can setup by following steps 2 and 3 of this tutorial
- OpenSSL installed and updated

To test, you will need one of two systems, with OpenSSL installed and updated:


- Another Linux Droplet
- Linux-based local system (Mac, Ubuntu, Debian, etc.)

# Step 1 — Install Nginx


In this step, we will use a built-in package installer called apt-get. It simplifies management drastically and facilitates a clean installation.


In the link specified in the prerequisites, you should have updated apt-get and installed the sudo package, as unlike other Linux distributions, Debian 8 does not come with sudo installed.


Nginx is the aforementioned HTTP server, focused on handling large loads with low memory usage. To install it, run the following:


```
sudo apt-get install nginx


```


For information on the differences between Nginx and Apache2, the two most popular open source web servers, see this article.


# Step 2 — Create Directory


This section is simply and short. We need to store the private key and certificate in a memorable location, so we need to create a new directory.


```
sudo mkdir /etc/nginx/ssl


```


# Step 3 — Create a Self Signed ECC Certificate


In this section, we will request a new certificate and sign it.


First, generate an ECC private key using OpenSSL’s ecparam tool.


- The out flag directs output to a file. For this tutorial, we will save the key in /etc/nginx/ssl/nginx.key.
- The name flag identifies the elliptic curve prime256v1.

```
sudo openssl ecparam -out /etc/nginx/ssl/nginx.key -name prime256v1 -genkey


```


Then, generate a certificate signing request.


- The key flag specifies the path to our key, generated in the previous command.
- The out flag specifies the path to our generated certificate.

```
sudo openssl req -new -key /etc/nginx/ssl/nginx.key -out /etc/nginx/ssl/csr.pem

```


Invoking this command will result in a series of prompts.


- Common Name: Specify your server’s IP address or hostname.
- Challenge Password: Do not supply one.
- Fill out all other fields at your own discretion. Hit ENTER to accept the defaults.

```
You are about to be asked to enter information that will be incorporated into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:US
State or Province Name (full name) [Some-State]:New York
Locality Name (eg, city) []:New York
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Digital Ocean Tutorial
Organizational Unit Name (eg, section) []:ECC Certificate Test
Common Name (e.g. server FQDN or YOUR name) []:example.com
Email Address []: webmaster@example.com

Please enter the following 'extra' attributes to be sent with your certificate request
A challenge password []:
An optional company name []:

```


Finally,  self-sign the certificate. The certificate is then used by the client to encrypt data only the server can read.


- x509 is the OpenSSL tool used to generate the certificate.
- The days flag specifies how long the certificate should remain valid. With this example, the certificate will last for one year.
- in specifies our previously-generated certificate request.

```
sudo openssl req -x509 -nodes -days 365 -key /etc/nginx/ssl/nginx.key -in /etc/nginx/ssl/csr.pem -out /etc/nginx/ssl/nginx.pem


```


Set the file permissions to protect your private key and certificate. For more information on the three-digit permissions code, see the tutorial on Linux permissions.


```
sudo chmod 600 /etc/nginx/ssl/*


```


Your certificate and the private key that protects it are now ready for setup.


# Step 4 — Setup the Certificate


In this section, we will configure Nginx virtual hosts with the key and certificate. In effect, our server will begin serving HTTPS instead of HTTP requests.


Open the server configuration file using nano or your favorite text editor.


```
sudo nano /etc/nginx/sites-enabled/default


```


At the top of the configuration file, you will find a block of code, akin to the following:


/etc/nginx/sites-enabled/default
```
...
# Default server configuration
#
server {
...
}

```


The next few edits will be made inside the server block.


1. First, comment out the first two lines of the server block, by preceding the line with a pound sign:

etc/nginx/sites-enabled/default
```
server {
	# listen 80 default_server;
	# listen [::]:80 default_server;

```


1. Then, uncomment the first listen line underneath SSL Configuration by removing the pound sign. Indent properly, and also remove ssl default_server.

/etc/nginx/sites-enabled/default
```
	# SSL Configuration
	#
	listen 443;
	# listen [::]:443 ssl default_server;
	#

```


1. 
Update the root directory, directly underneath the commented block. the original reads server_name _;. Change it to include your server ip, so that it reads server_name your_server_ip.

2. 
After server_name, add your SSL key and certificate paths.


/etc/nginx/sites-enabled/default
```
	    ssl on;
	    ssl_certificate /etc/nginx/ssl/nginx.pem;
	    ssl_certificate_key /etc/nginx/ssl/nginx.key;

```


1. Finally, add SSL settings.

/etc/nginx/sites-enabled/default
```
	    ssl_session_timeout 5m;
	    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
	    ssl_ciphers HIGH+kEECDH+AESGCM:HIGH+kEECDH:HIGH+kEDH:HIGH:!aNULL;
	    ssl_prefer_server_ciphers on;

```


Your final result should be identical to the following.


/etc/nginx/sites-enabled/default
```
# Default server configuration
#
server {
        # listen 80 default_server;
        # listen [::]:80 default_server;

        # SSL configuration
        #
        listen 443;
        # listen [::]:443 ssl default_server;
        #
        # Self signed certs generated by the ssl-cert package
        # Don't use them in a production server!
        #
        # include snippets/snakeoil.conf;

        root /var/www/html;

        # Add index.php to the list if you are using PHP
        index index.html index.htm index.nginx-debian.html;

        server_name your_server_ip;
        
        ssl on;
	   ssl_certificate /etc/nginx/ssl/nginx.pem;
	   ssl_certificate_key /etc/nginx/ssl/nginx.key;
	   ssl_session_timeout 5m;
	   ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
	   ssl_ciphers HIGH+kEECDH+AESGCM:HIGH+kEECDH:HIGH+kEDH:HIGH:!aNULL;
	   ssl_prefer_server_ciphers on;

        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                try_files $uri $uri/ =404;
        }

```


Once these changes have been made, save and exit out of the file.


Restart Nginx to apply the changes.


```
sudo service nginx restart


```


# Step 5 — Test Nginx with ECC


In this section, we will test the server, through the command line. Once again, this may be done on either (1) your local Linux-based system or (2) another Droplet. You may also run this command from the same shell window, but you may want a more solid proof of success.


Open connection via the HTTPS 443 port.


openssl s_client -connect your_server_ip:443


Scroll to the middle of the output after the key output, and you should find the following:


```
output---
SSL handshake has read 3999 bytes and written 444 bytes
---
...
SSL-Session:
...

```


Of course, the numbers are variable, but this is success. Congratulations!


Press CTRL+C to exit.


You can also visit your site in a web browser, using HTTPS in the URL (https://example.com). Your browser will warn you that the certificate is self-signed. You should be able to view the certificate and confirm that the details match what you entered in Step 4.


##Conclusion


This concludes our tutorial, leaving you with a working Nginx server, configured securely with an ECC certificate. For more information on working with OpenSSL, see the OpenSSL Essentials article.


