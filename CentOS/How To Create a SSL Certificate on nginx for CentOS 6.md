# How To Create a SSL Certificate on nginx for CentOS 6

```Security``` ```Nginx``` ```CentOS```










# Status: Deprecated


This article covers a version of CentOS that is no longer supported. If you are currently operating a server running CentOS 6, we highly recommend upgrading or migrating to a supported version of CentOS.


Reason:
CentOS 6 reached end of life (EOL) on November 30th, 2020 and no longer receives security patches or updates. For this reason, this guide is no longer maintained.


See Instead:
This guide might still be useful as a reference, but may not work on other CentOS releases. If available, we strongly recommend using a guide written for the version of CentOS you are using.


The following DigitalOcean tutorial may be of interest, as it outlines how to create an SSL certificate for Nginx on a CentOS 7 server:




- How To Create an SSL Certificate on Nginx for CentOS 7





## About Self-Signed Certificates


A SSL certificate is a way to encrypt a site's information and create a more secure connection. Additionally, the certificate can show the virtual private server's identification information to site visitors. Certificate Authorities can issue SSL certificates that verify the server's details while a self-signed certificate has no 3rd party corroboration.


# Intro


Make sure that nginx is installed on your VPS. If it is not, you can quickly install it with 2 steps.


Install the EPEL repository:


```
su -c 'rpm -Uvh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm' 
```


Install nginx


```
yum install nginx
```


#  Step One—Create a Directory for the Certificate


The SSL certificate has 2 parts main parts: the certificate itself and the public key.  To make all of the relevant files easy to access, we should create a directory to store them in:


```
 sudo mkdir /etc/nginx/ssl
```


We will perform the next few steps within the directory:


```
 cd /etc/nginx/ssl
```


#  Step Two—Create the Server Key and Certificate Signing Request


Start by creating the private server key. During this process,  you will be asked to enter a specific passphrase. Be sure to note this phrase carefully, if you forget it or lose it, you will not be able to access the certificate.


```
sudo openssl genrsa -des3 -out server.key 1024
```


Follow up by creating a certificate signing request:


```
sudo openssl req -new -key server.key -out server.csr
```


This command will prompt terminal to display a lists of fields that need to be filled in.


The most important line is "Common Name". Enter your official domain name here or, if you don't have one yet, your site's IP address. Leave the challenge password and optional company name blank.


```
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:US
State or Province Name (full name) [Some-State]:New York
Locality Name (eg, city) []:NYC
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Awesome Inc
Organizational Unit Name (eg, section) []:Dept of Merriment
Common Name (e.g. server FQDN or YOUR name) []:example.com                  
Email Address []:webmaster@awesomeinc.com
```


# Step Three—Remove the Passphrase


We are almost finished creating the certificate. However, it would serve us to remove the passphrase. Although having the passphrase in place does provide heightened security, the issue starts when one tries to reload nginx. In the event that nginx  crashes or needs to reboot, you will always have to re-enter your passphrase to get your entire web server back online.


Use this command to remove the passphrase:


```
sudo cp server.key server.key.org
sudo openssl rsa -in server.key.org -out server.key
```


# Step Four— Sign your SSL Certificate


Your certificate is all but done, and you just have to sign it. 
Keep in mind that you can specify how long the certificate should remain valid by changing the 365 to the number of days you prefer. As it stands, this certificate will expire after one year.


```
 sudo openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
```


You are now done making your certificate.


# Step Five—Set Up the Certificate


Open up the SSL config file:


```
 vi /etc/nginx/conf.d/ssl.conf
```


Uncomment within the section under the line HTTPS Server. Match your config to the information below, replacing the example.com in the "server_name" line with your domain name or IP address. If you are just looking to test your certificate, the default root there will work.


```
# HTTPS server

server {
    listen       443;
    server_name example.com;

    ssl on;
    ssl_certificate /etc/nginx/ssl/server.crt;
    ssl_certificate_key /etc/nginx/ssl/server.key; 
}
```


Then restart nginx:


```
 /etc/init.d/nginx restart
```


Visit https://youraddress.


You will see your self-signed certificate on that page!


## Resources


- http://wiki.nginx.org/HttpSslModule#Generate_Certificates

By Etel Sverdlov
