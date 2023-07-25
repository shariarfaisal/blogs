# How To Install iRedMail On CentOS 6 5 x64

```Email``` ```CentOS```










# Status: Deprecated


This article covers a version of CentOS that is no longer supported. If you are currently operating a server running CentOS 6, we highly recommend upgrading or migrating to a supported version of CentOS.


Reason:
CentOS 6 reached end of life (EOL) on November 30th, 2020 and no longer receives security patches or updates. For this reason, this guide is no longer maintained.


See Instead:
This guide might still be useful as a reference, but may not work on other CentOS releases. If available, we strongly recommend using a guide written for the version of CentOS you are using.


If you would like to create your own online e-mail system, you can use iRedMail. In this article, we will explain how you can do it.


## Step 1 - Droplet Creation


We use a 2 CPU Core / 2GB RAM droplet with CentOS 6.5 x64 image.


If you have a domain name you want to use, name your droplet as that domain name, which will become its hostname and reverse DNS record.


We should also add 2GB of SWAP memory to this droplet for stability:


```
dd if=/dev/zero of=/swap bs=1024 count=2097152
mkswap /swap && chown root. /swap && chmod 0600 /swap && swapon /swap
echo /swap swap swap defaults 0 0 >> /etc/fstab
echo vm.swappiness = 0 >> /etc/sysctl.conf && sysctl -p

```


## Step 2 - Create a Domain Name


For our Cloud Mail purposes, we will register a free domain, cloudmail.tk from dot.tk


Once you have your domain name registered, point it to DigitalOcean's name servers:


ns1.digitalocean.com (69.55.55.74)


ns2.digitalocean.com (141.0.175.217)


ns3.digitalocean.com (69.55.62.20)


![](https://assets.digitalocean.com/articles/community/dottk-DNS-register.png)
Now open your Control Panel on DigitalOcean and click DNS, located under Labs section.


Click Add Domain and create a new record by pointing your new domain to your droplet's IP address:


![](https://assets.digitalocean.com/articles/community/DO-DNS-Create.png)
Create a new MX record, make sure to have a trailing dot at the end of your domain name:


![](https://assets.digitalocean.com/articles/community/DO-DNS-Create2.png)
Add SPF records to make sure others cant spoof emails by pretending to send them from your domain.


Make sure to have "-all" in your SPF record, and point it to your droplet's IP.


The record's format would be "v=spf1 ip4:IP_ADDRESS -all"


![](https://assets.digitalocean.com/articles/community/DO-DNS-Create3.png)
There will be one more record to add after you have finished installing iRedMail - DKIM key.


## Step 3 - iRedMail Installation


Make sure to set the hostname of your domain name, if you haven't done this during droplet creation:


```
wget https://bitbucket.org/zhb/iredmail/downloads/iRedMail-0.8.6.tar.bz2
tar jxvf iRedMail-0.8.6.tar.bz2 && cd iRedMail-0.8.6
hostname cloudmail.tk
bash iRedMail.sh

```


You are greeted with a Graphical User Interface Installer by iRedMail:


![](https://assets.digitalocean.com/articles/community/CentOS63-CloudMail1.png)
If you have several droplets, you can even use GlusterFS for distributed, replicated e-mail storage, providing further redundancy:


![](https://assets.digitalocean.com/articles/community/CentOS63-CloudMail2.png)
For backend, we chose MySQL.  You can also use OpenLDAP and PostgreSQL:


![](https://assets.digitalocean.com/articles/community/CentOS63-CloudMail3.png)
Since we have registered a domain in Step 2, we will place it here:


![](https://assets.digitalocean.com/articles/community/CentOS63-CloudMail3-2.png)
From package selection, you can omit phpMyAdmin and Fail2Ban:


![](https://assets.digitalocean.com/articles/community/CentOS63-CloudMail4.png)
When asked whether you would like to use firewall rules provided with iRedMail, select 'No'.


![](https://assets.digitalocean.com/articles/community/CentOS63-CloudMail5.png)
Firewall rules should be custom made for each server, and adopting a DROP ruleset from iRedMail's package is not recommended.We would also not recommend using Fail2Ban from their package, as it banned our own IP when we refreshed a page.


Reboot your droplet after completion.


All of the installation notes and logs can be found in iRedMail.tips file (  /root/iRedMail-0.8.6/iRedMail.tips ).


Here you will have information on passwords, SSL certificate locations, and DKIM records.


Add the DKIM record to DigitalOcean's DNS control panel for your domain:


![](https://assets.digitalocean.com/articles/community/DO-DNS-Create4.png)
## Step 4 - Add SSL Certificate


Although this step is optional if you just want to use self-generated certificate, we would still recommend getting a trusted SSL certificate.


By default, iRedMail will create a self-signed certificate and store it in /etc/pki/tls/certs/iRedMail_CA.pem and /etc/pki/tls/private/iRedMail.key


We can get a free SSL certificate from InstantSSL


You would need to create a CSR and private KEY first:


```
cd /etc/pki/tls/certs
openssl req -out cloudmail.tk.csr -new -newkey rsa:2048 -nodes -keyout cloudmail.tk.key

```


This will generate 2 files: cloudmail.tk.csr (your Certificate Signing Request file), and cloudmail.tl.key (your private SSL key which should not be shared with anyone).


You would provide the CSR file (cloudmail.tk.csr) to InstantSSL during SSL request.


After they have validated your request, you will receive the certificate file (in zip format) that contains two files:


cloudmail_tk.ca-bundle (your SSL certificate bundle)


cloudmail_tk.crt (your SSL certificate)


Place both files to /etc/pki/tls/certs and modify /etc/httpd/conf.d/ssl.conf


```
SSLCertificateFile /etc/pki/tls/certs/cloudmail.tk.crt
SSLCertificateKeyFile /etc/pki/tls/certs/cloudmail.tk.key
SSLCACertificateFile /etc/pki/tls/certs/cloudmail.tk.ca-bundle.crt

```


Restart Apache


```
service httpd restart
```


Now you should have SSL enabled, and you can proceed to logging in to iRedAdmin (https://cloudmail.tk/iredadmin/ ) with username postmaster@cloudmail.tk and password you provided during installation in Step 3.


From iRedAdmin, you can add new users, new admins, and new domains into your system:


![](https://assets.digitalocean.com/articles/community/Ubuntu1210-iRedAdmin.png)
Once you have created an e-mail account, you can access it at https://cloudmail.tk/mail/


![](https://assets.digitalocean.com/articles/community/cloudmail-Mail.png)
![](https://assets.digitalocean.com/articles/community/cloudmail-Mail2.png)
And you are all done!


By Bulat Khamitov
