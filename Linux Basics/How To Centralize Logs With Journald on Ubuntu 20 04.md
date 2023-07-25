# How To Centralize Logs With Journald on Ubuntu 20 04

```Linux Basics``` ```Logging``` ```Ubuntu 20.04``` ```System Tools```

The author selected the Free and Open Source Fund to receive a donation as part of the Write for DOnations program.


## Introduction


System logs are an extremely important component of managing Linux systems. They provide an invaluable insight into how the systems are working and also how they are being used because, in addition to errors, they record operational information such as security events. The standard configuration for Linux systems is to store their logs locally on the same system where they occurred. This works for standalone systems but quickly becomes a problem as the number of systems increases. The solution to managing all these logs is to create a centralized logging server where each Linux host sends its logs, in real-time, to a dedicated log management server.


A centralized logging solution offers several benefits compared with storing logs on each host:


- Reduces the amount of disk space needed on each host to store log files.
- Logs can be retained for longer as the dedicated log server can be configured with more storage capacity.
- Advanced log analysis can be carried out that requires logs from multiple systems and also more compute resources than may be available on the hosts.
- Systems administrators can access the logs for all their systems that they may not be able to log in to directly for security reasons.

In this guide, you will configure a component of the systemd suite of tools to relay log messages from client systems to a centralized log collection server. You will configure the server and client to use TLS certificates to encrypt the log messages as they are transmitted across insecure networks such as the internet and also to authenticate each other.


# Prerequisites


Before you begin this guide you’ll need the following:


- Two Ubuntu 20.04 servers.
- A non-root user with sudo privileges on both servers. Follow the Initial Server Setup with Ubuntu 20.04 guide for instructions on how to do this. You should also configure the UFW firewall on both servers as explained in the guide.
- Two hostnames that point to your servers. One hostname for the client system that generates the logs and another one for the log collection server. Learn how to point hostnames to DigitalOcean Droplets by consulting the Domains and DNS documentation.

This guide will use the following two example hostnames:


- client.your_domain: The client system that generates the logs.
- server.your_domain: The log collection server.

Log in to both the client and server in separate terminals via SSH as the non-root sudo user to begin this tutorial.



Note: Throughout the tutorial command blocks are labeled with the server name (client or server) that the command should be run on.

# Step 1 — Installing systemd-journal-remote


In this step, you will install the systemd-journal-remote package on the client and the server. This package contains the components that the client and server use to relay log messages.


First, on both the client and server, run a system update to ensure that the package database and the system is current:


Client and Server
```
sudo apt update
sudo apt upgrade


```


Next, install the systemd-journal-remote package:


Client and Server
```
sudo apt install systemd-journal-remote


```


On the server, enable and start the two systemd components that it needs to receive log messages with the following command:


Server
```
sudo systemctl enable --now systemd-journal-remote.socket
sudo systemctl enable systemd-journal-remote.service


```


The --now option in the first command starts the services immediately. You did not use it in the second command because this service will not start until it has TLS certificates, which you will create in the next step.


On the client, enable the component that systemd uses to send the log messages to the server:


Client
```
sudo systemctl enable systemd-journal-upload.service


```


Next, on the server, open ports 19532  and 80 in the UFW firewall. This will allow the server to receive the log messages from the client. Port 80 is the port that certbot will use to generate the TLS certificate. The following commands will open these ports:


Server
```
sudo ufw allow in 19532/tcp
sudo ufw allow in 80/tcp


```


On the client, you only need to open port 80 with this command:


Client
```
sudo ufw allow in 80/tcp


```


You have now installed the required components and completed the base system configuration on the client and server. Before you can configure these components to start relaying log messages you will register the Let’s Encrypt TLS certificates for the client and server using the certbot utility.


# Step 2 — Installing Certbot and Registering Certificates


Let’s Encrypt is a Certificate Authority that issues free TLS certificates. These certificates allow computers to both encrypt the data that they send between them and also verify each other’s identity. These certificates are what allow you to secure your internet browsing with HTTPS. The same certificates can be used by any other application that wants the same level of security. The process of registering the certificate is the same no matter what you use them for.


In this step, you will install the certbot utility and use it to register the certificates. It will also automatically take care of renewing the certificates when they expire. The registration process here is the same on the client and server. You only need to change the hostname to match the host where you are running the registration command.


First, enable Ubuntu’s universe repository as the certbot utility resides in the universe repository. If you already have the universe repository enabled, running these commands will not do anything to your system and are safe to run:


Client and Server
```
sudo apt install software-properties-common
sudo add-apt-repository universe
sudo apt update


```


Next, install certbot on both hosts:


Client and Server
```
sudo apt install certbot


```


Now you’ve installed certbot, run the following command to register the certificates on the client and server:


Client and Server
```
sudo certbot certonly --standalone --agree-tos --email sammy@your_domain -d your_domain


```


The options in this command mean as follows:


- certonly: Register the certificate and make no other changes on the system.
- --standalone: Use certbot’s built-in web server to verify the certificate request.
- --agree-tos: Automatically agree to the Let’s Encrypt Terms of Service.
- --email your_email: This is the email address that Let’s Encrypt will use to notify you about certificate expiry and other important information.
- -d your_domain: The hostname that the certificate will be registered for. This must match the system where you run it.

When you run this command you will be asked if you want to share the email address with Let’s Encrypt so they can email you news and other information about their work. Doing this is optional, if you do not share your email address the certificate registration will still complete normally.


When the certificate registration process completes it will place the certificate and key files in /etc/letsencrypt/live/your_domain/ where your_domain is the hostname that you registered the certificate for.


Finally you need to download a copy of Let’s Encrypt’s CA and intermediate certificates and put them into the same file. journald will use this file to verify the authenticity of the certificates on the client and server when they communicate with each other.


The following command will download the two certificates from Let’s Encrypt’s website and put them into a single file called letsencrypt-combined-certs.pem in your user’s home directory.


Run this command on the client and server to download the certificates and create the combined file:


Client and Server
```
curl -s https://letsencrypt.org/certs/{isrgrootx1.pem.txt,letsencryptauthorityx3.pem.txt} > ~/letsencrypt-combined-certs.pem


```


Next, move this file into the Let’s Encrypt directory containing the certificates and keys:


Client and Server
```
sudo cp ~/letsencrypt-combined-certs.pem /etc/letsencrypt/live/your_domain/


```


You’ve now registered the certificates and keys. In the next step, you will configure the log collection server to start listening for and storing log messages from the client.


# Step 3 — Configuring the Server


In this step, you will configure the server to use the certificate and key files that you generated in the last step so that it can start accepting log messages from the client.


systemd-journal-remote is the component that listens for log messages. Open its configuration file at /etc/systemd/journal-remote.conf with a text editor to start configuring it on the server:


```
sudo nano /etc/systemd/journal-remote.conf


```


Next, uncomment all the lines under the [Remote] section and set the paths to point to the TLS files you just created:


/etc/systemd/journal-remote.conf
```
[Remote]
Seal=false
SplitMode=host
ServerKeyFile=/etc/letsencrypt/live/server.your_domain/privkey.pem
ServerCertificateFile=/etc/letsencrypt/live/server.your_domain/fullchain.pem
TrustedCertificateFile=/etc/letsencrypt/live/server.your_domain/letsencrypt-combined-certs.pem

```


Here are the options you’ve used here:


- Seal=false: Sign the log data in the journal. Enable this if you need maximum security; otherwise, you can leave it as false.
- SplitMode=host: The logs from the remote clients will be split by host in /var/log/journal/remote. If you would prefer all the logs to be added to a single file set this to SplitMode=false.
- ServerKeyFile: The server’s private key file.
- ServerCertificateFile: The server’s certificate file.
- TrustedCertificateFile: The file containing the Let’s Encrypt CA certificates.

Now, you need to change the permissions on the Let’s Encrypt directories that contain the certificates and key so that the systemd-journal-remote can read and use them.


First, change the permissions so that the certificate and private key are readable:


```
sudo chmod 0755 /etc/letsencrypt/{live,archive}
sudo chmod 0640 /etc/letsencrypt/live/server.your_domain/privkey.pem


```


Next, change the group ownership of the private key to systemd-journal-remote’s group:


```
sudo chgrp systemd-journal-remote /etc/letsencrypt/live/server.your_domain/privkey.pem


```


You can now start systemd-journal-remote:


```
sudo systemctl start systemd-journal-remote.service


```


Your log collection server is now running and ready to start accepting log messages from a client. In the next step, you will configure the client to relay the logs to your collection server.


# Step 4 — Configuring the Client


In this step, you will configure the component that relays the log messages to the log collection server. This component is called systemd-journal-upload.


The default configuration for systemd-journal-upload is that it uses a temporary user that only exists while the process is running. This makes allowing systemd-journal-upload to read the TLS certificates and keys more complicated. To resolve this you will create a new system user with the same name as the temporary user that will get used in its place.


First, create the new user called systemd-journal-upload on the client with the following adduser command:


```
sudo adduser --system --home /run/systemd --no-create-home --disabled-login --group systemd-journal-upload


```


These options to the command are:


- --system: Create the new user as a system user. This gives the user a UID (User Identifier) number under 1000. UID’s over 1000 are usually given to user accounts that a human will use to log in with.
- --home /run/systemd: Set /run/systemd as the home directory for this user.
- --no-create-home: Don’t create the home directory set, as it already exists.
- --disabled-login: The user cannot log in to the server via, for example, SSH.
- --group: Create a group with the same name as the user.

Next, set the permissions and ownership of the Let’s Encrypt certificate files:


```
sudo chmod 0755 /etc/letsencrypt/{live,archive}
sudo chmod 0640 /etc/letsencrypt/live/client.your_domain/privkey.pem
sudo chgrp systemd-journal-upload /etc/letsencrypt/live/client.your_domain/privkey.pem


```


Now, edit the configuration for systemd-journal-upload, which is at /etc/systemd/journal-upload.conf. Open this file with a text editor:


```
sudo nano /etc/systemd/journal-upload.conf


```


Edit this file so that it looks like the following:


/etc/systemd/journal-upload.conf
```
[Upload]
URL=https://server.your_domain:19532
ServerKeyFile=/etc/letsencrypt/live/client.your_domain/privkey.pem
ServerCertificateFile=/etc/letsencrypt/live/client.your_domain/fullchain.pem
TrustedCertificateFile=/etc/letsencrypt/live/client.your_domain/letsencrypt-combined-certs.pem

```


Finally, restart the systemd-journal-upload service so it uses the new configuration:


```
sudo systemctl restart systemd-journal-upload.service


```


Your client is now set up and running and is sending its log messages to the log collection server. In the next step, you will check that the logs are being sent and recorded correctly.


# Step 5 — Testing the Client and Server


In this step, you will test that the client is relaying log messages to the server and that the server is storing them correctly.


The log collection server stores the logs from the clients in a directory at /var/log/journal/remote/. When you restarted the client at the end of the last step it began sending log messages so there is now a log file in /var/log/journal/remote/. The file will be named after the hostname you used for the TLS certificate.


Use the ls command to check that the client’s log file is present on the server:


Server
```
sudo ls -la /var/log/journal/remote/


```


This will print the directory contents showing the log file:


```
Outputtotal 16620
drwxr-xr-x  2 systemd-journal-remote systemd-journal-remote     4096 Jun 30 16:17  .
drwxr-sr-x+ 4 root                   systemd-journal            4096 Jun 30 15:55  ..
-rw-r-----  1 systemd-journal-remote systemd-journal-remote 8388608 Jul  1 10:46 'remote-CN=client.your_domain'

```


Next, write a log message on the client to check that the server is receiving the client’s messages as you expect. You will use the logger utility to create a custom log message on the client. If everything is working systemd-journal-upload will relay this message to the server.


On the client run the following logger command:


Client
```
sudo logger -p syslog.debug "### TEST MESSAGE from client.your_domain ###"


```


The -p syslog.debug in this command sets the facility and severity of the message. Setting this to syslog.debug will make clear it’s a test message. This command will record the message ### TEST MESSAGE from client.your_domain ### to the client’s journal, which systemd-journal-upload then relays to the server.


Next, read the client’s journal file on the server to check that the log messages are arriving from the client. This file is a binary log file so you will not be able to read it with tools like less. Instead, read the file using journalctl with the --file= option that allows you to specify a custom journal file:


Server
```
sudo journalctl --file=/var/log/journal/remote/remote-CN=client.your_domain.journal


```


The log message will appear as follows:


```
Test log message. . .
Jun 29 13:10:09 client root[3576]: ### TEST MESSAGE from client.your_domain ###

```


Your log centralization server is now successfully collecting logs from your client system.


# Conclusion


In this article, you set up a log central collection server and configured a client to relay a copy of its system logs to the server. You can configure as many clients as you need to relay messages to the log collection server using the client configuration steps you used here.


