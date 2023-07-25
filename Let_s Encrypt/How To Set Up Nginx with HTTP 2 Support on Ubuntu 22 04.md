# How To Set Up Nginx with HTTP 2 Support on Ubuntu 22 04

```Let's Encrypt``` ```Nginx``` ```Ubuntu``` ```Ubuntu 22.04```

A previous version of this tutorial was written by Sergey Zhukaev.


## Introduction


Nginx is a fast and reliable open-source web server. It gained its popularity due to its low memory footprint, high scalability, ease of configuration, and support for a wide variety of protocols.


HTTP/2 is a newer version of the Hypertext Transport Protocol, which is used on the Web to deliver pages from server to browser. HTTP/2 is the first major update of HTTP in almost two decades: HTTP1.1 was introduced to the public back in 1999 when webpages were much smaller in size. The Internet has dramatically changed since then, and we are now facing the limitations of HTTP 1.1. The protocol limits potential transfer speeds for most modern websites because it downloads parts of a page in a queue – the previous part must download completely before the download of the next part begins – and an average modern web page downloads dozens of individual CSS, javascript, and image assets.


HTTP/2 solves this problem because it brings a few fundamental changes:


- All requests are downloaded in parallel, not in a queue
- HTTP headers are compressed
- Pages transfer as a binary, not as a text file, which is more efficient
- Servers can “push” data even without the user’s request, which improves speed for users with high latency

Even though HTTP/2 does not require encryption, developers of the two most popular browsers, Google Chrome and Mozilla Firefox, have stated that for security reasons they will support HTTP/2 only for HTTPS connections. Hence, if you decide to set up servers with HTTP/2 support, you must also secure them with HTTPS.


This tutorial will help you set up a fast and secure Nginx server with HTTP/2 support.


# Prerequisites


Before getting started, you will need a few things:


- An Ubuntu 22.04 server set up by following the Ubuntu 22.04 initial server setup guide, including a sudo non-root user and a firewall.
- Nginx installed on your server, which you can do by following How To Install Nginx on Ubuntu 22.04.
- A domain name configured to point to your server. You can purchase one on Namecheap or get one for free on Freenom. You can learn how to point domains to DigitalOcean Droplets by following the documentation on How To Manage Your Domain With DigitalOcean.
- A TLS/SSL certificate configured for your server. You have two options:

You can get a free certificate from Let’s Encrypt by following How to Secure Nginx with Let’s Encrypt on Ubuntu 22.04.
You can also generate and configure a  self-signed certificate by following How to Create a Self-signed SSL Certificate for Nginx in Ubuntu 22.04.


- You can get a free certificate from Let’s Encrypt by following How to Secure Nginx with Let’s Encrypt on Ubuntu 22.04.
- You can also generate and configure a  self-signed certificate by following How to Create a Self-signed SSL Certificate for Nginx in Ubuntu 22.04.
- Nginx configured to redirect traffic from port 80 to port 443, which should be covered by the previous prerequisites.
- Nginx configured to use a 2048-bit or higher Ephemeral Diffie-Hellman (DHE) key, which should also be covered by the previous prerequisites.

# Step 1 —  Enabling HTTP/2 Support


If you followed the server block set up step in the Nginx installation tutorial, you should have a server block for your domain at /etc/nginx/sites-available/your_domain with the server_name directive already set appropriately. The first change we will make will be to modify your domain’s server block to use HTTP/2.


Open the configuration file for your domain using nano or your preferred editor:


```
sudo nano /etc/nginx/sites-enabled/your_domain


```


In the file, locate the listen variables associated with port 443:


/etc/nginx/sites-enabled/your_domain
```
...
    listen [::]:443 ssl ipv6only=on; 
    listen 443 ssl; 
...

```


The first one is for IPv6 connections. The second one is for all IPv4 connections. We will enable HTTP/2 for both.


Modify each listen directive to include http2:


/etc/nginx/sites-enabled/your_domain
```
...
    listen [::]:443 ssl http2 ipv6only=on; 
    listen 443 ssl http2; 
...

```


This tells Nginx to use HTTP/2 with supported browsers.


Save the configuration file and exit the text editor. If you are using nano, press Ctrl+X then, when prompted, Y and then Enter.


Whenever you make changes to Nginx configuration files, you should check the configuration for errors, using the -t flag, which runs Nginx’s built-in syntax check command:


```
sudo nginx -t


```


If the syntax is error-free, you will receive output like the following:


Output of sudo nginx -t
```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

```


Next, you’ll configure your Nginx server to use a more restrictive list of ciphers to improve your server’s security.


# Step 2 — Removing Old and Insecure Cipher Suites


HTTP/2 has a blocklist of old and insecure ciphers that should be avoided. Cipher suites are cryptographic algorithms that describe how the transferred data should be encrypted.


The method you’ll use to define the ciphers depends on how you’ve configured your TLS/SSL certificates for Nginx.


If you used Certbot to obtain your certificates, it also created the file /etc/letsencrypt/options-ssl-nginx.conf that contains ciphers that aren’t secure enough for HTTP/2. However, modifying this file will prevent Certbot from applying updates in the future, so we’ll just tell Nginx not to use this file and we’ll specify our own list of ciphers.


Open the server block configuration file for your domain:


```
sudo nano /etc/nginx/sites-enabled/your_domain

```


Locate the line that includes the options-ssl-nginx.conf file and comment it out by adding a # character to the beginning of the line:


/etc/nginx/sites-enabled/your_domain
```

# include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot

```


Below that line, add this line to define the allowed ciphers:


/etc/nginx/sites-enabled/your_domain
```

ssl_ciphers EECDH+CHACHA20:EECDH+AES128:RSA+AES128:EECDH+AES256:RSA+AES256:EECDH+3DES:RSA+3DES:!MD5;

```


Save the file and exit the editor.


If you used self-signed certificates or used a certificate from a third party and configured it according to the prerequisites, open the file /etc/nginx/snippets/ssl-params.conf in your text editor:


```
sudo nano /etc/nginx/snippets/ssl-params.conf


```


Locate the following line:


/etc/nginx/snippets/ssl-params.conf
```
...
ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA384;
...

```


Modify it to use the following list of ciphers:


/etc/nginx/snippets/ssl-params.conf
```

...
ssl_ciphers EECDH+CHACHA20:EECDH+AES128:RSA+AES128:EECDH+AES256:RSA+AES256:EECDH+3DES:RSA+3DES:!MD5;

```


Save the file and exit your editor.


Once again, check the configuration for syntax errors using the nginx -t command:


```
sudo nginx -t


```


If you encounter any errors, address them and test again.


Once your configuration passes the syntax check, restart Nginx using the systemctl command:


```
sudo systemctl reload nginx.service


```


With the server restarted, let’s verify that it works.


# Step 3 — Verifying that HTTP/2 is Enabled


Let’s ensure the server is running and working with HTTP/2.


Use the curl command to make a request to your site and view the headers:


```
curl -I -L --http2 https://your_domain


```


You’ll receive output like the following:


```
HTTP/2 200
**Server**: nginx/1.18.0 (Ubuntu)
**Date**: Tue, 21 Jun 2022 22:19:09 GMT
**Content-Type**: text/html
**Content-Length**: 612
**Last-Modified**: Tue, 21 Jun 2022 22:17:56 GMT
**Connection**: keep-alive
**ETag**: "62b24394-264"
**Accept-Ranges**: bytes

```


You can also verify that HTTP/2 is in use in Google Chrome. Open Chrome and navigate to https://your_domain. Open the Chrome Developer Tools (View -> Developer -> Developer Tools) and reload the page (View -> Reload This Page). Navigate to the Network tab, right-click on the table header row that starts with Name, and select the Protocol option from the popup menu.


You’ll have a new Protocol column that contains h2 (which stands for HTTP/2), indicating that HTTP/2 is working.





At this point, you’re ready to serve content through the HTTP/2 protocol. Let’s improve security and performance by enabling HSTS.


# Step 4 — Enabling HTTP Strict Transport Security (HSTS)


Even though your HTTP requests redirect to HTTPS, you can enable HTTP Strict Transport Security (HSTS) to avoid having to do those redirects. If the browser finds an HSTS header, it will not try to connect to the server via regular HTTP again for a given time period. No matter what, it will exchange data using only encrypted HTTPS connection. This header also protects us from protocol downgrade attacks.


Open the server block configuration file for your domain again:


```
sudo nano /etc/nginx/your_domain

```


Add this line to the same block of the file containing the SSL ciphers in order to enable HSTS:


/etc/nginx/your_domain
```
server {
...
    ssl_ciphers EECDH+CHACHA20:EECDH+AES128:RSA+AES128:EECDH+AES256:RSA+AES256:EECDH+3DES:RSA+3DES:!MD5;
    add_header Strict-Transport-Security "max-age=15768000" always;
}
...

```


The max-age is set in seconds. The value 15768000 is equivalent to 6 months.


By default, this header is not added to subdomain requests. If you have subdomains and want HSTS to apply to all of them, you should add the includeSubDomains variable at the end of the line, like this:


/etc/nginx/your_domain
```
add_header Strict-Transport-Security "max-age=15768000; includeSubDomains" always;

```


Save the file, and exit the editor.


Once again, check the configuration for syntax errors:


```
sudo nginx -t


```


Finally, restart the Nginx server to apply the changes.


```
sudo systemctl reload nginx.service


```


# Conclusion


Your Nginx server is now serving HTTP/2 pages. If you want to test the strength of your SSL connection, please visit Qualys SSL Lab and run a test against your server. If everything is configured properly, you should get an A+ mark for security.


To learn more about how Nginx parses and implements server block rules, try reading Understanding Nginx Server and Location Block Selection Algorithms.


