# Web Servers Checkpoint

```Apache``` ```Configuration Management``` ```Nginx```

## Introduction


This checkpoint is intended to help you assess what you learned from our introductory articles to Web Servers, where we introduced practical implementations and popular options for configuring your web server. You can use this checkpoint to test your knowledge on these topics, review key terms and commands, and find resources for continued learning.


Web servers act as the intermediary between a request (such as a URL you enter into your browser) and a response to that request (such as the web pages that are tied to the specific URL). After you have installed a web server to handle requests, you will be able to build the stack needed to develop your web apps.


In this checkpoint, you’ll find three sections that synthesize the central ideas from the introductory articles: defining what a web server is, using the Linux command to configure and modify your web server, and understanding networking protocols. In each of these sections, there are interactive components to help you test your knowledge. At the end of this checkpoint, you will find opportunities for continued learning about security and web applications.


## Resources


- Introduction to Web Servers.
- How To Install Apache.
- How To Install Nginx.
- Apache vs Nginx: Practical Considerations.
- How To Secure Apache with Let’s Encrypt.
- How To Secure Nginx with Let’s Encrypt.

# What Is a Web Server?


A web server serves the files needed to render a web application in your browser through HTTP and HTTPS protocol. A web application is software that can be accessed through the browser and which end users can interact with.



Check Yourself
What features and common goals should you consider when evaluating a web server solution?

Get the answers with the dropdown feature.

High uptime with stable use and fast loading
Concurrent access by many users at once
Scalability as audience and userbase grows
Straightforward and repeatable installation and setup
Clear documentation
Developer support, including short-term patches and long-term software updates
Community support and adoption of the software



To understand web servers, it’s important to have familiarity with several key computing and networking terms.



Terms To Know
Define each of the following terms, then use the dropdown feature to check your work.

Port
A port is an address on a single machine that can be tied to a specific piece of software. It is not a physical interface or location, but it allows your server to communicate using more than one application.


Protocol
A protocol is a set of rules and standards that defines a language that devices can use to communicate.
Some low level protocols include TCP, UDP, IP, and ICMP. Some familiar examples of application layer protocols, built on these lower protocols, are HTTP (for accessing web content), SSH, TLS/SSL, and FTP.


Firewall
A firewall is a system that provides network security by filtering incoming and outgoing network traffic based on a set of user-defined rules.
Commonly used firewalls include Iptables, UFW, and Fail2Ban. For more on how firewalls work, you can review What is a Firewall and How Does It Work?.


DNS
DNS, or domain name system, is an application layer protocol that connects a domain name (such as a custom URL) with a specific IP address.


HTTP and HTTPS Servers
An HTTP server is a web server that uses the Hypertext Transfer Protocol (HTTP) to facilitate communication over a computer network.
HTTPS refers to the Hypertext Protocol Transfer Secure, which encrypts communication with Transfer Layer Security (TLS) or Secure Sockets Layer (SSL). To enable an HTTPS server and SSL/TLS encryption, you will need to generate and provide an SSL certificate through a certificate authority such as Let’s Encrypt.


Proxy Server
As an additional pass-through layer, a proxy server acts as a gatekeeper of the internet between clients and servers, routing HTTP request traffic to web servers behind it.
Nginx was designed to function as a reverse proxy server, and Apache can also be used as a reverse proxy server. For more on using Nginx as a reverse proxy, read Understanding Nginx HTTP Proxying, Load Balancing, Buffering, and Caching.
For more on proxies, review our Introduction to Proxies.


Installing a web server to serve files for your web apps is critical when building your apps. Web servers can handle both static and dynamic content, especially in response to HTTP requests that call on your site’s domain name. To ensure your web server can handle those requests, you can manage configuration settings with the command line.


# Using the Command Line to Configure Your Web Server


You were introduced to using the Linux command line in the introduction to Cloud Servers. By installing and configuring web servers like Apache and Nginx, you have continued to navigate the command line by modifying config files, writing the files for web pages, and running startup scripts. You’ve also used the APT package manager to update your server.


You can now configure and update your web server using commands such as:


- chown to change file ownership.
- chmod to set or change the permissions for accessing files.
- curl to transfer data with a specified location (the URL).
- hostname to set or display the hostname and domain name.
- systemctl to control the systemd service.

You’ve also worked with systemd, which is an option for the initial process (init) that runs when a Linux system is booted up. For more on using systemd commands, review How To Use Systemctl to Manage Systemd Services and Units. System startup scripts are typically stored in the /etc/init.d directory, which requires root or sudo access, and logs can be found in /var/log.


You’ve also used Certbot with the --standalone option to handle certification from Let’s Encrypt for your domain.


## Configuring Your Server Files


Config files provide the settings for web servers and can be customized to suit your needs.



Check Yourself
Where do you find your system’s config files? Use the dropdown feature to get the answers.

Config files are typically found in …
Config files are typically found in the /etc directory and can be edited in the command line with an editor like nano or vim.


Web files being served as web pages are often located in …
Web files being served as web pages are often located in /var/www and can be updated with a command-line editor or by using an integrated development environment (IDE).


You have installed and configured common, open-source web servers, Apache or Nginx, on your remote server. Together, they serve approximately 50% of all web traffic.


A firewall will run on top of your web server or any other application servers that you have running in order to manage incoming and outgoing traffic to your web app. The default firewall for Ubuntu is the uncomplicated firewall or ufw, which manages the iptables firewall. You can read more on What Is a Firewall and How To Choose an Effective Firewall Policy, knowing that Apache and Nginx offer these profiles for commonly used ports:




Both Apache and Nginx register with a firewall upon installation, and each has three profiles tailored to common ports.



Port
Apache Profile
Nginx Profile




To open only port 80
Apache
Nginx HTTP


To open both port 80 and port 443
Apache Full
Nginx Full


To open only port 443
Apache Secure
Nginx HTTPS





## Using Apache


Apache uses .htaccess files for its decentralized configuration settings, which means that you can make configuration changes at the directory level. The .htaccess files provide granular options for customization beyond the main Apache configuration file.


You can check your knowledge about other aspects of Apache in the interactive components below.



Check Yourself
Use the dropdown feature to get the answers.

Where will you find Apache’s main config files?
Apache’s config files are held in the /etc/apache2 directory.


Where will you find other crucial Apache files, such as virtual hosts and access logs?
Apache’s per-site virtual hosts are held in /etc/apache2/sites-available/ and can be linked to /etc/apache2/sites-enabled/.
Configuration fragments unrelated to virtual hosts are stored in /etc/apache2/conf-available/ and linked to /etc/apache2/conf-enabled/.
Access logs are held in /var/log/apache2/access.log, and error messages are recorded in /var/log/apache2/error.log based on the LogLevel directive in the /etc/apache2/apache2.conf global config file.


Apache uses specific terminology to support its system setup. Assess your knowledge with the Apache terms to know:



Apache Terms to Know
Define each of the following terms, then use the dropdown feature to check your work.

Virtual Hosts
Each virtual host describes an individual site or domain that can be customized and configured independently, which lets you serve different content to different visitors.
For more on virtual hosts, follow the tutorial on How To Set Up Apache Virtual Hosts.


Multi-Processing Modules
Apache’s Multi-Processing Modules (MPMs) handle client requests, though only one MPM can be loaded on the server at a time. Commonly used modules, such as mpm_prefork and mpm_worker, rely on a threaded system to respond to requests.
Modules are held in the /etc/apache2/mods-available/ and /etc/apache2/mods-enabled/ directories.


## Using Nginx


Nginx uses centralized uniform resource identifier (URI) pattern matching for its configuration files. A uniform resource identifier is a unique sequence of characters to differentiate resources.


You can check your knowledge about other aspects of Nginx in the interactive components below.



Check Yourself
Use the dropdown feature to get the answers.

Where will you find the config tiles for Nginx?
Config files are held in the /etc/nginx directory. Per-site server blocks are in /etc/nginx/sites-available and linked to the /etc/nginx/sites-enabled directory.


Where will you find other crucial Nginx files, like access logs and error messages?
Access logs and error messages are both in the /var/log/nginx directory.
Access logs are in /var/log/nginx/access.log, and error messages are recorded in /var/log/nginx/error.log.


Nginx uses specific terminology to support its system setup. Assess your knowledge with the Nginx terms to know:



Nginx Terms to Know
Define each of the following terms, then use the dropdown feature to check your work.

Server Blocks
Server blocks are configuration details that can be used to host more than one domain from a single server.
Typically, server block configurations will be held in the etc/nginx/sites-available directory, though the server block enabled by default on Ubuntu 22.04 can be found in the /var/www/html directory.


Symlinks
Symlinks, or symbolic links, function like a shortcut directing to another file or folder on the machine. A symlink does not have data but points to the file that does.
Files in the /etc/nginx/sites-enabled directory are symbolically linked to their counterparts in the /etc/nginx/sites-available directory.


Worker Processes
Nginx uses worker processes to handle client requests through an event loop. Unlike Apache’s multi-threaded system, the worker processes are single-threaded and connections are managed asynchronously within the loop.


# Understanding Network Communication and Security Protocols


Network communication protocols can often be layered, and a common combination is to use cryptography on top of TCP and UDP protocols. TCP stands for transmission control protocol, and TCP handles data communication through packet transfer in a three-way handshake. UDP stands for user datagram protocol and is also implemented for data transport.


TLS, or transport layer security (and its predecessor secure sockets layer, or SSL) is a cryptographic protocol that places normal traffic in a protected, encrypted wrapper.



Check Yourself
Use the dropdown to get the answers.

How do TCP and UDP differ?
TCP uses a handshake protocol to verify that both ends acknowledge the request. UDP does not complete this verification step when sending data to the host, which is faster but less reliable.
TCP and UDP can also impact how you choose an effective firewall policy.


How do TCP and HTTP relate?
The hypertext transfer protocol (HTTP) is an application-layer protocol that runs on top of the transmission control protocol (TCP). In short, TCP defines how packets are exchanged across a network, while HTTP and HTTPS provide instructions on how to process that packet data.


For more on networking and inter-process communication, you can review What Is a Socket? and Understanding Sockets.


When operating your own web server, you handle the security yourself. In the resource tutorials for this section, you have enabled TLS encryption with Let’s Encrypt on both Apache and Nginx web servers using the Certbot client.


While you’ve used Certbot to configure an SSL certificate for your domain from Let’s Encrypt, you can also generate a self-signed certificate to encrypt communication between your server and any clients, using the following tutorials for your chosen web server:


- How To Create a Self-Signed SSL Certificate for Apache in Ubuntu 22.04
- How To Create a Self-Signed SSL Certificate for Nginx in Ubuntu 22.04


Check Yourself
What is a concern with using a self-signed certificate?

Use the dropdown to get the answer.
A self-signed certificate is not signed by any of the trusted certificate authorities included with web browsers and operating systems, so users cannot use the certificate to validate the identity of your server automatically. As a result, your users will see a security error when visiting your site.


You can review the OpenSSL Essentials as a quick reference for working with SSL certificates, private keys, and certificate signing requests.


# What’s Next?


With your web server in place, you can now create additional server blocks or virtual hosts as needed, modifying configuration files to suit the needs of your web apps. You could try configuring Nginx as a reverse proxy for Apache.


To build your understanding of firewalls or to reconfigure your web server’s firewall settings, try these tutorials:


- What is a Firewall and How Does It Work?
- How To Choose an Effective Firewall Policy to Secure your Servers
- How To Set Up a Firewall with UFW on Ubuntu 22.04
- How To Implement a Basic Firewall Template with Iptables on Ubuntu 20.04
- How the Iptables Firewall Works
- How Fail2Ban Works to Protect Services on a Linux Server
- How To Protect an Nginx Server with Fail2Ban on Ubuntu 22.04
- Recommended Security Measures to Protect Your Servers

You might also consider setting up a collection of open-source software commonly used together to serve web applications, such as the LAMP, LEMP, or LOMP stack:


- Collection of LAMP Stack Tutorials
- How To Install Linux, Nginx, MySQL, PHP (LEMP stack) on Ubuntu 20.04
- How To Install Linux, OpenLiteSpeed, MariaDB, PHP (LOMP stack) on Ubuntu 20.04

To start building your web apps, try following our How To Code series, including:


- How To Code in JavaScript
- How To Code in Python
- How To Code in Go
- How To Code in React.js
- How To Code in TypeScript
- How To Code in Ruby
- How To Code in Node.JS
- How To Code in PHP

With your newfound knowledge on web servers, you can also continue your cloud journey with databases, containers, and security. If you haven’t yet, check out our introductory articles on cloud servers.


