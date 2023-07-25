# How To Configure OCSP Stapling on Apache and Nginx

```Apache``` ```Security``` ```Nginx```

## Introduction


OCSP stapling is a TLS/SSL extension which aims to improve the performance of SSL negotiation while maintaining visitor privacy. Before going ahead with the configuration, a short brief on how certificate revocation works. This article uses free certificates issued by StartSSL to demonstrate.


This tutorial will use the base configuration for Apache and Nginx outlined below:


- How To Set Up Multiple SSL Certificates on One IP with Apache on Ubuntu 12.04
- How To Set Up Multiple SSL Certificates on One IP with Nginx on Ubuntu 12.04

# About OCSP


OCSP (Online Certificate Status Protocol) is a protocol for checking if a SSL certificate has been revoked. It was created as an alternative to CRL to reduce the SSL negotiation time. With CRL (Certificate Revocation List) the browser downloads a list of revoked certificate serial numbers and verifies the current certificate, which increases the SSL negotiation time. In OCSP the browser sends a request to a OCSP URL and receives a response containing the validity status of the certificate. The following screenshot shows the OCSP URI of digitalocean.com.





# About OCSP stapling


OCSP has major two issues: privacy and heavy load on CA’s servers.


Since OCSP requires the browser to contact the CA to confirm certificate validity it compromises privacy. The CA knows what website is being accessed and who accessed it.


If a HTTPS website gets lots of visitors the CA’s OCSP server has to handle all the OCSP requests made by the visitors.


When OCSP stapling is implemented the certificate holder (read web server) queries the OCSP server themselves and caches the response. This response is “stapled” with the TLS/SSL Handshake via the Certificate Status Request extension response. As a result the CA’s servers are not burdened with requests and browsers no longer need to disclose users’ browsing habits to any third party.


# Check for OCSP stapling support


OCSP stapling is supported on


- Apache HTTP Server (>=2.3.3)
- Nginx (>=1.3.7)

Please check the version of your installation with the following commands before proceeding.


Apache:


```
apache2 -v

```


Nginx:


```
nginx -v

```


CentOS/Fedora users replace apache2 with httpd.


# Retrieve the CA bundle


Retrieve the root CA and intermediate CA’s certificate in PEM format and save them in a single file. This is for StartSSL’s Root and Intermediate CA certificates.


```
cd /etc/ssl
wget -O - https://www.startssl.com/certs/ca.pem https://www.startssl.com/certs/sub.class1.server.ca.pem | tee -a ca-certs.pem> /dev/null

```


If your CA provides certificates in DER format convert them to PEM. For example DigiCert provides certificates in DER format. To download them and convert to PEM use the following commands:


```
cd /etc/ssl
wget -O - https://www.digicert.com/CACerts/DigiCertHighAssuranceEVRootCA.crt | openssl x509 -inform DER -outform PEM | tee -a ca-certs.pem> /dev/null
wget -O - https://www.digicert.com/CACerts/DigiCertHighAssuranceEVCA-1.crt | openssl x509 -inform DER -outform PEM | tee -a ca-certs.pem> /dev/null

```


Both sets of commands use tee to write to the file, so you can use sudo tee if logged in as a non-root user.


# Configuring OCSP Stapling on Apache


Edit the SSL virtual hosts file and place these lines inside the <VirtualHost></VirtualHost> directive.


```
sudo nano /etc/apache2/sites-enabled/example.com-ssl.conf

```


```
SSLCACertificateFile /etc/ssl/ca-certs.pem
SSLUseStapling on

```


A cache location has to be specified outside <VirtualHost></VirtualHost>.


```
sudo nano /etc/apache2/sites-enabled/example.com-ssl.conf

```


```
SSLStaplingCache shmcb:/tmp/stapling_cache(128000)

```


If you followed this article to setup SSL sites on Apache, the virtual host file will look this:


/etc/apache2/sites-enabled/example.com-ssl.conf


```
<IfModule mod_ssl.c>
    SSLStaplingCache shmcb:/tmp/stapling_cache(128000)
    <VirtualHost *:443>

            ServerAdmin webmaster@localhost
            ServerName example.com
            DocumentRoot /var/www

            SSLEngine on

            SSLCertificateFile /etc/apache2/ssl/example.com/apache.crt
            SSLCertificateKeyFile /etc/apache2/ssl/example.com/apache.key

            SSLCACertificateFile /etc/ssl/ca-certs.pem
            SSLUseStapling on
    </VirtualHost>
</IfModule>

```


Do a configtest to check for errors.


```
apachectl -t

```


Reload if Syntax OK is displayed.


```
service apache2 reload

```


Access the website on IE (on Vista and above) or Firefox 26+ and check the error log.


```
tail /var/log/apache2/error.log

```


If the file defined in the SSLCACertificateFile directive is missing, a certificate an error similar to the following is displayed.


```
[Fri May 09 23:36:44.055900 2014] [ssl:error] [pid 1491:tid 139921007208320] AH02217: ssl_stapling_init_cert: Can't retrieve issuer certificate!
[Fri May 09 23:36:44.056018 2014] [ssl:error] [pid 1491:tid 139921007208320] AH02235: Unable to configure server certificate for stapling

```


If no such errors are displayed proceed to the final step.


# Configuring OCSP stapling on Nginx


Edit the SSL virtual hosts file and place the following directives inside the server {} section.


```
sudo nano /etc/nginx/sites-enabled/example.com.ssl

```


```
ssl_stapling on;
ssl_stapling_verify on;
ssl_trusted_certificate /etc/ssl/private/ca-certs.pem;

```


If you followed this article to setup SSL hosts on Nginx the complete virtual host file will look like this:


/etc/nginx/sites-enabled/example.com.ssl


```
server {

        listen   443;
        server_name example.org;

        root /usr/share/nginx/www;
        index index.html index.htm;

        ssl on;
        ssl_certificate /etc/nginx/ssl/example.org/server.crt;
        ssl_certificate_key /etc/nginx/ssl/example.org/server.key;

        ssl_stapling on;
        ssl_stapling_verify on;
        ssl_trusted_certificate /etc/ssl/private/ca-certs.pem;
}

```


Do a configtest to see if everything is correct.


```
service nginx configtest

```


Then reload the nginx service.


```
service nginx reload

```


Access the website on IE (on Vista and above) or Firefox 26+ and check the error log.


```
tail /var/log/nginx/error.log

```


If the file defined in ssl_trusted_certificate is missing a certificate an error similar to the following is displayed:


```
2014/05/09 17:38:16 [error] 1580#0: OCSP_basic_verify() failed (SSL: error:27069065:OCSP routines:OCSP_basic_verify:certificate verify error:Verify error:unable to get local issuer certificate) while requesting certificate status, responder: ocsp.startssl.com

```


If no such errors are displayed proceed to the next step.


# Testing OCSP Stapling


Two methods will be explained to test if OCSP stapling is working - the openssl command-line tool and SSL test at Qualys.


## The OpenSSL command


This command’s output displays a section which says if your web server responded with OCSP data. We grep this particular section and display it.


```
echo QUIT | openssl s_client -connect www.digitalocean.com:443 -status 2> /dev/null | grep -A 17 'OCSP response:' | grep -B 17 'Next Update'

```


Replace www.digitalocean.com with your domain name. If OCSP stapling is working properly the following output is displayed.


```
OCSP response:
======================================
OCSP Response Data:
    OCSP Response Status: successful (0x0)
    Response Type: Basic OCSP Response
    Version: 1 (0x0)
    Responder Id: 4C58CB25F0414F52F428C881439BA6A8A0E692E5
    Produced At: May  9 08:45:00 2014 GMT
    Responses:
    Certificate ID:
      Hash Algorithm: sha1
      Issuer Name Hash: B8A299F09D061DD5C1588F76CC89FF57092B94DD
      Issuer Key Hash: 4C58CB25F0414F52F428C881439BA6A8A0E692E5
      Serial Number: 0161FF00CCBFF6C07D2D3BB4D8340A23
    Cert Status: good
    This Update: May  9 08:45:00 2014 GMT
    Next Update: May 16 09:00:00 2014 GMT

```


No output is displayed if OCSP stapling is not working.


## Qualys online SSL test


To check this online go to this website and enter your domain name. Once testing completes check under the Protocol Details section.





# Additional reading


- Mozilla’s article on OCSP stapling - http://en.wikipedia.org/wiki/OCSP_stapling
- Wikipedia article on OCSP stapling - http://en.wikipedia.org/wiki/OCSP_stapling

