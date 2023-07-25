# How To Create an SSL Certificate on Apache for CentOS 7

```Apache``` ```Security``` ```CentOS```

## Introduction


TLS, or transport layer security, and its predecessor SSL, which stands for secure sockets layer, are web protocols used to wrap normal traffic in a protected, encrypted wrapper.


Using this technology, servers can send traffic safely between the server and clients without the possibility of the messages being intercepted by outside parties. The certificate system also assists users in verifying the identity of the sites that they are connecting with.


In this guide, you will set up a self-signed SSL certificate for use with an Apache web server on a CentOS 7 server.



Note: A self-signed certificate will encrypt communication between your server and any clients. However, because it is not signed by any of the trusted certificate authorities included with web browsers, users cannot use the certificate to validate the identity of your server automatically. As a result, users will see a security error when visiting your site.
Because of this limitation, self-signed certificates are not appropriate for a production environment serving the public. They are typically used for testing, or for securing non-critical services used by a single user or a small group of users that can establish trust in the certificate’s validity through alternate communication channels.
For a more production-ready certificate solution, check out Let’s Encrypt, a free certificate authority. You can learn how to download and configure a Let’s Encrypt certificate in your setting up Apache with a Let’s Encrypt certificate on CentOS 7 tutorial.

# Prerequisites


Before you begin with this guide, there are a few steps that need to be completed first.


- You will need access to a CentOS 7 server with a non-root user that has sudo privileges. If you haven’t configured this yet, you can run through the CentOS 7 initial server setup guide to create this account.
- You will also need to have Apache installed as described in step 1 of the How to Install the Apache Web Server on CentOS 7 tutorial.
After these steps are complete, you can log in as your non-root user account through SSH and continue with the tutorial.

# Step 1 — Installing mod_ssl


In order to set up the self-signed certificate, you first have to be sure that mod_ssl, an Apache module that provides support for SSL encryption, is installed on the server. You can install mod_ssl with the yum command:


```
sudo yum install mod_ssl


```


The module will automatically be enabled during installation, and Apache will be able to start using an SSL certificate after it is restarted. You don’t need to take any additional steps for mod_ssl to be ready for use.


# Step 2 — Creating a New Certificate


Now that Apache is ready to use encryption, you can move on to generating a new SSL certificate. TLS/SSL works by using a combination of a public certificate and a private key. The SSL key is kept secret on the server. It is used to encrypt content sent to clients. The SSL certificate is publicly shared with anyone requesting the content. It can be used to decrypt the content signed by the associated SSL key.


The /etc/ssl/certs directory, which can be used to hold the public certificate, should already exist on the server. You will need to create an /etc/ssl/private directory as well, to hold the private key file. Since the secrecy of this key is essential for security, it’s important to lock down the permissions to prevent unauthorized access:


```
sudo mkdir /etc/ssl/private
sudo chmod 700 /etc/ssl/private


```


Now, you can create a self-signed key and certificate pair with OpenSSL in a single command by typing:


```
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crt


```


You will be asked a series of questions. Before going over that, let’s take a look at what is happening in the command:


- openssl: This is the basic command line tool for creating and managing OpenSSL certificates, keys, and other files.
- req: This subcommand specifies that you want to use X.509 certificate signing request (CSR) management. The “X.509” is a public key infrastructure standard that SSL and TLS adheres to for its key and certificate management. You want to create a new X.509 cert, so you are using this subcommand.
- -x509: This further modifies the previous subcommand by telling the utility that you want to make a self-signed certificate instead of generating a certificate signing request, as would normally happen.
- -nodes: This tells OpenSSL to skip the option to secure your certificate with a passphrase. You need Apache to be able to read the file, without user intervention, when the server starts up. A passphrase would prevent this from happening because you would have to enter it after every restart.
- -days 365: This option sets the length of time that the certificate will be considered valid. You set it for one year here.
- -newkey rsa:2048: This specifies that you want to generate a new certificate and a new key at the same time. You did not create the key that is required to sign the certificate in a previous step, so you need to create it along with the certificate. The rsa:2048 portion tells it to make an RSA key that is 2048 bits long.
- -keyout: This line tells OpenSSL where to place the generated private key file that you are creating.
- -out: This tells OpenSSL where to place the certificate that you are creating.

As stated above, these options will create both a key file and a certificate. You will be asked a few questions about your server in order to embed the information correctly in the certificate.


Fill out the prompts appropriately.



Note: It is important that you enter your domain name or your server’s public IP address when you’re prompted for the Common Name (e.g. server FQDN or YOUR name). The value here should match how your users access your server.

The entirety of the prompts will look something like this:


```
OutputCountry Name (2 letter code) [XX]:US
State or Province Name (full name) []:Example
Locality Name (eg, city) [Default City]:Example 
Organization Name (eg, company) [Default Company Ltd]:Example Inc
Organizational Unit Name (eg, section) []:Example Dept
Common Name (eg, your name or your server's hostname) []:your_domain_or_ip
Email Address []:webmaster@example.com

```


Both of the files you created will be placed in the appropriate subdirectories of the /etc/ssl directory.


As you are using OpenSSL, you should also create a strong Diffie-Hellman group, which is used in negotiating Perfect Forward Secrecy with clients.


You can do this by typing:


```
sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048


```


This may take a few minutes, but when it’s done you will have a strong DH group at /etc/ssl/certs/dhparam.pem that you can use in your configuration.


# Step 3 — Setting Up the Certificate


You now have all of the required components of the finished interface. The next thing to do is to set up the virtual hosts to display the new certificate.


Open a new file in the /etc/httpd/conf.d directory:


```
sudo vi /etc/httpd/conf.d/your_domain_or_ip.conf


```


Paste in the following minimal VirtualHost configuration:


/etc/httpd/conf.d/your_domain_or_ip.conf
```
<VirtualHost *:443>
    ServerName your_domain_or_ip
    DocumentRoot /var/www/html
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/apache-selfsigned.crt
    SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key
</VirtualHost>

```


Be sure to update the ServerName line to however you intend to address your server. This can be a hostname, full domain name, or an IP address. Make sure whatever you choose matches the Common Name you chose when making the certificate.


## Setting Up Secure SSL Parameters


Next, you will add some additional SSL options that will increase your site’s security. The options you will use are recommendations from Cipherlist.eu. This site is designed to provide easy-to-consume encryption settings for popular software.



Note: The default suggested settings on Cipherlist.eu offer strong security. Sometimes, this comes at the cost of greater client compatibility. If you need to support older clients, there is an alternative list that can be accessed by clicking on the link labeled “Yes, give me a ciphersuite that works with legacy / old software.”
The compatibility list can be used instead of the default suggestions in the configuration above between the two comment blocks. The choice of which config you use will depend largely on what you need to support.

There are a few pieces of the configuration that you may wish to modify. First, you can add your preferred DNS resolver for upstream requests to the resolver directive. You used Google’s for this guide, but you can change this if you have other preferences.


Finally, you should take a moment to read up on HTTP Strict Transport Security, or HSTS, and specifically about the “preload” functionality. Preloading HSTS provides increased security, but can have far reaching consequences if accidentally enabled or enabled incorrectly. In this guide, you will not preload the settings, but you can modify that if you are sure you understand the implications.


The other changes you will make are to remove +TLSv1.3 and comment out the SSLSessionTickets and SSLOpenSSLConfCmd directives, since these aren’t available in the version of Apache shipped with CentOS 7.


Paste in the settings from the site after the end of the VirtualHost block:


/etc/httpd/conf.d/your_domain_or_ip.conf
```

    . . .
</VirtualHost>
. . .

# Begin copied text
# from https://cipherli.st/

SSLCipherSuite EECDH+AESGCM:EDH+AESGCM
# Requires Apache 2.4.36 & OpenSSL 1.1.1
SSLProtocol -all +TLSv1.2
# SSLOpenSSLConfCmd Curves X25519:secp521r1:secp384r1:prime256v1
# Older versions
# SSLProtocol All -SSLv2 -SSLv3 -TLSv1 -TLSv1.1
SSLHonorCipherOrder On
# Disable preloading HSTS for now.  You can use the commented out header line that includes
# the "preload" directive if you understand the implications.
#Header always set Strict-Transport-Security "max-age=63072000; includeSubdomains; preload"
Header always set Strict-Transport-Security "max-age=63072000; includeSubdomains"
# Requires Apache >= 2.4
SSLCompression off
SSLUseStapling on
SSLStaplingCache "shmcb:logs/stapling-cache(150000)"
# Requires Apache >= 2.4.11
# SSLSessionTickets Off

```


When you are finished making these changes, you can save and close the file.


## Creating a Redirect from HTTP to HTTPS (Optional)


As it stands now, the server will provide both unencrypted HTTP and encrypted HTTPS traffic.  For better security, it is recommended in most cases to redirect HTTP to HTTPS automatically.  If you do not want or need this functionality, you can safely skip this section.


To redirect all traffic to be SSL encrypted, create and open a file ending in .conf in the /etc/httpd/conf.d directory:


```
sudo vi /etc/httpd/conf.d/non-ssl.conf


```


Inside, create a VirtualHost block to match requests on port 80.  Inside, use the ServerName directive to again match your domain name or IP address.  Then, use Redirect to match any requests and send them to the SSL VirtualHost.  Make sure to include the trailing slash:


/etc/httpd/conf.d/non-ssl.conf
```
<VirtualHost *:80>
       ServerName your_domain_or_ip
        Redirect "/" "https://your_domain_or_ip/"
</VirtualHost>

```


Save and close this file when you are finished.


# Step 4 — Applying Apache Configuration Changes


By now, you have created an SSL certificate and configured your web server to apply it to your site. To apply all of these changes and start using your SSL encryption, you can restart the Apache server to reload its configurations and modules.


First, check your configuration file for syntax errors by typing:


```
sudo apachectl configtest


```


As long as the output ends with Syntax OK, you are safe to continue.  If this is not part of your output, check the syntax of your files and try again:


```
Output. . .
Syntax OK

```


Restart the Apache server to apply your changes by typing:


```
sudo systemctl restart httpd.service


```


Next, make sure port 80 and 443 are open in your firewall.  If you are not running a firewall, you can skip ahead.


If you have a firewalld firewall running, you can open these ports by typing:


```
sudo firewall-cmd --add-service=http
sudo firewall-cmd --add-service=https
sudo firewall-cmd --runtime-to-permanent


```


If you have iptablesfirewall running, the commands you need to run are highly dependent on your current rule set.  For a basic rule set, you can add HTTP and HTTPS access by typing:


```
sudo iptables -I INPUT -p tcp -m tcp --dport 80 -j ACCEPT
sudo iptables -I INPUT -p tcp -m tcp --dport 443 -j ACCEPT


```


# Step 5 — Testing Encryption


Now, you’re ready to test your SSL server.


Open your web browser and type https:// followed by your server’s domain name or IP into the address bar:


```
https://your_domain_or_ip

```


Because the certificate you created isn’t signed by one of your browser’s trusted certificate authorities, you will likely see a scary looking warning like the one below:





This is expected and normal. You are only interested in the encryption aspect of your certificate, not the third party validation of your host’s authenticity. Click “ADVANCED” and then the link provided to proceed to your host anyway:





You should be taken to your site. If you look in the browser address bar, you will see some indication of partial security. This might be a lock with an “x” over it or a triangle with an exclamation point. In this case, this just means that the certificate cannot be validated. It is still encrypting your connection.


If you configured Apache to redirect HTTP requests to HTTPS, you can also check whether the redirect functions correctly:


```
http://your_domain_or_ip

```


If this results in the same icon, this means that your redirect worked correctly.


# Conclusion


You have configured your Apache server to handle both HTTP and HTTPS requests. This will help you communicate with clients securely and avoid outside parties from being able to read your traffic.


If you are planning on using SSL for a public website, you should probably purchase an SSL certificate from a trusted certificate authority to prevent the scary warnings from being shown to each of your visitors.


