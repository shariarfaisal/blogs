# How To Install Zentyal on Ubuntu 14 04

```Ubuntu``` ```Control Panels``` ```Email``` ```Networking``` ```Messaging```

# Introduction


Most businesses require several server types such as file servers, print servers, email servers, etc. Zentyal combines these services and more, as a complete small business server for Linux.


Zentyal servers are simple to use because of the Graphical User Interface (GUI).  The GUI provides an easy and intuitive interface for use by novice and experienced administrators alike. Command-line administration is available, too. We’ll be showing how to use both of these methods in this tutorial.


To see a list of the specific software that can be installed with Zentyal, please see either of the Installing Packages sections.


Some people may be familiar with the Microsoft Small Business Server (SBS), now called Windows Server Essentials. Zentyal is a similar product that is based on Linux, and more specifically Ubuntu. Zentyal is also a drop-in replacement for Microsoft SBS and Microsoft Exchange Servers. Since Zentyal is open source, it is a cost-effective choice.


## Zentyal Editions


There are two types of Zentyal available. The first is the Community Edition and the other is the Commercial Edition.


The Community Edition has all the latest features, stable or otherwise. No official support is offered by the company for technical issues. No cloud services are provided with the Community Edition. A new version is released every three months with unofficial support for the most recent release. Users are unlimited.


The Commercial Edition has all the latest features, stable and tested. Support is offered based on the Small and Medium Business Edition. Cloud Services are integrated into the server and based on the SMB Edition. The number of users supported by the Commercial Edition is based on the SMB Edition purchased. A new Commercial Edition is released every two years and supported for four years.


Note: The Community Edition cannot be upgraded to the Commercial Edition.


## Zentyal Requirements


Zentyal is Debian-based and built on the latest Ubuntu Long Term Support (LTS) version. The current hardware requirements for Zentyal 3.5 are based on Ubuntu Trusty 14.04.1 LTS (kernel 3.5). Zentyal uses the LXDE desktop and the Openbox window manager.


The minimum hardware requirements for Ubuntu Server Edition include 300 MHz CPU, 128 MB of RAM, and 500 MB of disk space. Of course, these are bare minimums and would produce undesired responses on a network when running multiple network services.


Keep in mind that every network service requires different hardware resources and the more services installed, the more hardware requirements are increased. In most cases, it is best to start with the basic services you require and then add other services as needed. If the server starts to lag in processing user requests, you should consider upgrading your server plan.


Depending on your number of users, and which Zentyal services you plan to run, your hardware requirements will change. These are the Zentyal recommendations. For DigitalOcean deployments, you should go by the RAM column:





Profile
Number of Users
CPU
RAM
Disk Space
Network Cards




Gateway
<50
P4
2 GB
80 GB
2+



50+
Xeon dual core
4 GB
160 GB
2+


Infrastructure
<50
P4
1 GB
80 GB
1



50+
P4
2 GB
160 GB
1


Office
<50
P4
1 GB
250 GB
1



50+
Xeon dual core
2 GB
500 GB
1


Communications
<100
Xeon dual core
4 GB
250 GB
1



100+
Xeon dual core
8 GB
500 GB
1




We’ll talk more about the profiles and different types of Zentyal services later in the article.


# Installing Zentyal


Create a 1 GB Droplet running Ubuntu 14.04.


Add a user with sudo access.


First, you need to add the Zentyal repository to your repository list with the following command:


```
sudo add-apt-repository "deb http://archive.zentyal.org/zentyal 3.5 main extra"

```


After the packages are downloaded they should be verified using a public key from Zentyal. To add the public key, execute the following two commands:


```
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 10E239FF
wget -q http://keys.zentyal.org/zentyal-3.5-archive.asc -O- | sudo apt-key add -

```


Now that the repository list is updated, you need to update the package lists from the repositories. To update the package lists, execute this command:


```
sudo apt-get update

```


Once the package list is updated, you can install Zentyal by running:


```
sudo apt-get install zentyal

```


When prompted, set a secure root password (twice) for MySQL. Confirm port 443.


Zentyal is now installed.


If you prefer to use the command line to install your Zentyal packages, read the next section. Or, if you prefer to use a dashboard, skip to the Accessing Zentyal Dashboard section.


# Installing Packages (Command Line)


Now, you can start installing the specific services you require. There are four basic profiles which install many related modules at once. These profiles are:


- 
zentyal-office — The profile is for setting up an office network to share resources.  Resources can include files, printers, calendars, user profiles, and groups.

- 
zentyal-communication — Server can be used for business communications such as email, instant messaging, and Voice Over IP (VOIP).

- 
zentyal-gateway — The server will be a controlled gateway for the business to and from the Internet.  Internet access can be controlled and secured for internal systems and users.

- 
zentyal-infrastructure — The server can manage the network infrastructure for the business.  Managment consists of NTP, DHCP, DNS, etc.


You can see what’s installed with each profile here. To install a profile, run this command:


```
sudo apt-get install zenytal-office

```


You can also install each module individually as needed. For example, if you wanted only the antivirus module of the Office profile installed, you would execute the following:


```
sudo apt-get install zentyal-antivirus

```


You can also install all the profiles in one command:


```
sudo apt-get install zentyal-all

```


When you are installing certain packages, you will need to provide information about your systems via the interactive menus.


Some of the module names are straightforward, but here is a defined list of Zentyal packages:


- zentyal-all - Zentyal - All Component Modules (all Profiles)
- zentyal-office - Zentyal Office Suite (Profile)
- zentyal-antivirus - Zentyal Antivirus
- zentyal-dns – Zentyal DNS
- zentyal-ebackup - Zentyal Backup
- zentyal-firewall – Zentyal Firewall Services
- zentyal-ntp – NTP Services
- zentyal-remoteservices - Zentyal Cloud Client
- zentyal-samba - Zentyal File Sharing and Domain Services
- zentyal-communication - Zentyal Communications Suite
- zentyal-jabber - Zentyal Jabber (Instant Messaging)
- zentyal-mail - Zentyal Mail Service
- zentyal-mailfilter - Zentyal Mail Filter
- zentyal-gateway - Zentyal Gateway Suite
- zentyal-l7-protocols - Zentyal Layer-7 Filter
- zentyal-squid – HTTP Proxy
- zentyal-trafficshaping - Zentyal Traffic Shaping
- zentyal-infrastructure - Zentyal Network Infrastructure Suite
- zentyal-ca – Zentyal Certificate Authority
- zentyal-dhcp – DHCP Services
- zentyal-openvpn – VPN Services
- zentyal-webserver - Zentyal Web Server

Other modules which are not included in the profiles are as follows:


- zentyal-bwmonitor - Zentyal Bandwidth Monitor
- zentyal-captiveportal - Zentyal Captive Portal
- zentyal-ips - Zentyal Intrusion Prevention System
- zentyal-ipsec - Zentyal IPsec and L2TP/IPsec
- zentyal-monitor - Zentyal Monitor
- zentyal-nut - Zentyal UPS Management
- zentyal-openchange - Zentyal OpenChange Server
- zentyal-radius - Zentyal RADIUS
- zentyal-software - Zentyal Software Management
- zentyal-sogo - Zentyal OpenChange Webmail
- zentyal-usercorner - Zentyal User Corner
- zentyal-users - Zentyal Users and Computers
- zentyal-webmail - Zentyal Webmail Service

# Accessing the Zentyal Dashboard


Access the Zentyal dashboard by visiting the IP address or domain of your server in your browser, over HTTPS (port 443):


https://SERVER IP


The Zentyal server creates a self-signed SSL certificate for use when being accessed remotely. Any browser accessing the server’s dashboard remotely will be asked if the site is trusted and an exception will need to be made as shown below. The method will vary based on your browser.


Because of the SSL certificate, an error is generated that the site is untrusted. You need to click on the line I Understand the Risks. Then click on the Add Exception button. Select Confirm Security Exception. After the exception is added, it is a permanent listing that does not occur again unless the server IP Address should change.


![Certificate warning](https://assets.digitalocean.com/articles/Install_Zentyal/Figure 2.jpg)


![Certificate exception](https://assets.digitalocean.com/articles/Install_Zentyal/Figure 3.jpg)


You should see the dashboard login page.


![Zentyal dashboard login page](https://assets.digitalocean.com/articles/Install_Zentyal/Figure 1.jpg)


Your Zentyal username and password are the same user and password that you use to SSH to your Ubuntu server. This user must be added to the sudo group. (Granting full permissions to the user by some other method will NOT work.) If an existing user account needs to be added to the sudo group, run the following command:


```
sudo adduser username sudo

```


To add more Zentyal users, add new Ubuntu users. To add a new user use the following command to create the user and also add the user to the sudo group:


```
sudo adduser username --ingroup sudo

```


Once you log into the Zentyal server, you will see a collection of packages available for installation.


![Zentyal dashboard package list](https://assets.digitalocean.com/articles/Install_Zentyal/Figure 4.jpg)


You can also see a module list at https://SERVER IP/Software/EBox as shown below.


![Zentyal dashboard component list](https://assets.digitalocean.com/articles/Install_Zentyal/Figure 5.jpg)


# Installing Packages (Dashboard)


You can install Zentyal packages from the dashboard. There are four basic profiles which install many related modules at once. You can see what’s installed with each profile here. Or, check the list below:


Office:


This profile sets up shared office resources like files, printers, calendars, user profiles, and groups.


- 
Samba4

- 
Heimdal Kerberos

- 
CUPS

- 
Duplicity


Communication:


This profile includes email, instant messaging, and Voice Over IP (VOIP).


- 
Postfix

- 
Dovecot

- 
Roundcube

- 
Sieve

- 
Fetchmail

- 
Spamassassin

- 
ClamAV

- 
Postgrey

- 
OpenChange

- 
Roundcube

- 
ejabbered


Gateway:


This profile includes software to control and secure Internet access.


- 
Corosync

- 
Pacemaker

- 
Netfilter

- 
Iproute2


Linux networking subsystem:


- 
Iproute2

- 
Squid

- 
Dansguardian

- 
ClamAV

- 
FREERadius

- 
OpenVPN

- 
OpenSWAN

- 
xl2tpd

- 
Suricata

- 
Amavisd-new

- 
Spamassasin

- 
ClamAV

- 
Postgrey


Infrastructure:


This profile allows you to manage the office network, including NTP, DHCP, DNS, etc.


- 
ISC DHCP

- 
BIND 9

- 
NTPd

- 
OpenSSL

- 
Apache

- 
NUT


In the left-hand navigation, go to “Software Management” then “Zentyal Components” – You’ll see the four profiles at the top. (Or, click View basic mode to see the four profiles.)


![Zentyal dashboard component list, profiles](https://assets.digitalocean.com/articles/Install_Zentyal/Figure 11.jpg)


Below the profiles is a list of all the modules you can install individually.


![Zentyal dashboard component list, modules](https://assets.digitalocean.com/articles/Install_Zentyal/Figure 12.jpg)


The previous images show the basic view. If you click on View advanced mode, the screen should look like this:


![Zentyal dashboard component list, advanced mode](https://assets.digitalocean.com/articles/Install_Zentyal/Figure 13.jpg)


Once you have selected your modules, click the INSTALL button at the bottom of the page.


Once the packages are installed, you’ll see links for them in the dashboard navigation menu on the left. You can start setting up your new software through the Zentyal dashboard by navigating to the appropriate menu item in the control panel.


# Updating Packages (Dashboard)


It’s important to keep your system up to date with the latest security patches and features.


Let’s install some updates from the dashboard. Click the Dashboard link on the left. In the image below, you can see there are 26 System Updates, with 12 of them being Security Updates. To start the system update, simply click on 26 system updates (12 security).


![Zentyal dashboard update notification](https://assets.digitalocean.com/articles/Install_Zentyal/Figure 6.jpg)


This will take you to the System updates page with a list of all updates available for the Zentyal server.


![Zentyal dashboard update list](https://assets.digitalocean.com/articles/Install_Zentyal/Figure 7.jpg)


Here you can check the items you wish to update. At the bottom is an item to Update all packages as shown below.


![Zentyal dashboard update notification](https://assets.digitalocean.com/articles/Install_Zentyal/Figure 8.jpg)


Once you have selected the necessary updates, you can click on the UPDATE button at the bottom of the page. The download and installation of the update packages will begin as shown below.


![Zentyal dashboard update notification](https://assets.digitalocean.com/articles/Install_Zentyal/Figure 9.jpg)


Once done, you should see a screen similar to the one below, which shows that the update successfully completed.


![Zentyal dashboard update notification](https://assets.digitalocean.com/articles/Install_Zentyal/Figure 10.jpg)


Once the update is completed, you can press the UPDATE LIST button to verify that no other updates are available.


# Conclusion


For a small or medium business, Zentyal is a server that can do it all. Services can be enabled as they are needed and disabled when they are not needed. Zentyal is also user-friendly enough that novice administrators can perform system updates and profile/module installation, using the command line or the Graphical User Interface (GUI).


If needed, multiple Zentyal servers can be used to distribute the services required by the business to create a more efficient network.


