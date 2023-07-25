# 3 Ways to Securely Browse the Internet with OpenVPN on Debian 8

```Security``` ```VPN``` ```Debian```

## Introduction


Reasons for browsing the Internet with more privacy vary as much as the ways to achieve it.


In this tutorial we will explain in detail how to set up a virtual private network (VPN) on a server so it secures three important components of your Internet browsing experience:


- Privatize your web traffic by securing unencrypted traffic, preventing cookies and other trackers, and masking your local computer’s IP address
- Prevent your local ISP from logging DNS queries, by sending them from the VPN straight to Google’s DNS servers
- Scan for and prevent access to viruses and malicious applications

By running your own VPN server rather than using a commercial one, you can also avoid logging your browsing history (unless you choose to do so). Finally, you get to choose its physical location, so you can minimize latency. However, using a VPN is usually slower than using a direct Internet connection.


We’ll do this by installing and configuring the following applications on your Debian 8 server:


- 
ClamAV is an open source antivirus engine for detecting trojans, viruses, malware, other malicious threats

- 
Dnsmasq is a software package that provides DNS (and few more) services. We will use it only as a DNS cache

- 
HAVP HTTP AntiVirus proxy is a proxy with an anti-virus filter. It does not cache or filter content. It scans all the traffic with third-party antivirus engines. In this tutorial we will use HAVP as a Transparent Proxy and chain HAVPand Privoxy together

- 
OpenVPN Community Edition is a popular VPN server. It provides a secure connection to your trusted server, and can also push DNS Server settings to its clients. In this tutorial the term OpenVPN will be used as the shortened form of the VPN server’s name

- 
Privoxy is, from the official website, a non-caching web proxy with advanced filtering capabilities for enhancing privacy, modifying web page data and HTTP headers, controlling access, and removing ads and other obnoxious Internet junk


After completing this tutorial, you will have a privacy gateway that:


- Secures your connection when using public WiFi spots
- Blocks advertisements and tracking features from web sites
- Speeds up web page loading times by caching server-side DNS responses
- Scans the pages you visit and files you download for known viruses

## How It Works


The following diagram displays the path that a web request follows through the VPN we will set up in this tutorial.


The lanes with green backgrounds are the components of the VPN server. Green boxes represent the request steps, and blue and red boxes represent the response steps.





The traffic between your computer and the privacy server will flow through a VPN tunnel. When you open a web page in your browser, your request will be transferred to the VPN server. On the VPN server, your request will be redirected to HAVP and subsequently to Privoxy.


Privoxy will match the URL against its database of patterns. If the URL matches, it will block the URL and return a valid but empty response.


If the URL is not blocked, Privoxy acts as a non-caching proxy server to query DNS and retrieve the content of the URL. DNS queries are handled and cached by Dnsmasq.


HAVP receives the content from Privoxy and performs a virus scan via ClamAV. If any virus is found it returns an error page.


# Prerequisites


Please make sure you complete the following prerequisites:


- Debian 8 Droplet with 1 GB of RAM
- Initial Server Setup with Debian 8
- How To Set Up an OpenVPN Server on Debian 8

System Requirements


The server we will configure will be easy on CPU, RAM, and disk space. Select a Droplet with at least 1GB of RAM and that provides enough bandwidth to accommodate your browsing needs.


The operating system of choice for this tutorial is Debian 8. It should also work more or less the same way for other Debian-based Linux distros like Ubuntu.


Licenses


All of the software used in this tutorial is available from Debian repositories and subject to Debian policies.


Security


This server will intercept all of your HTTP requests. Someone who takes control of this server could act as a man-in-the-middle and monitor all of your HTTP traffic, redirect DNS requests, etc. You do need to secure your server. Please refer to the tutorials mentioned in the beginning of this section to set up sudo access and a firewall as an initial level of protection.


# Step 1 — Installing OpenVPN and Other Prerequisites


If you have not yet installed OpenVPN please do so now.


You can follow the tutorial How To Set Up an OpenVPN Server on Debian 8.


In the following steps we will install a few packages. To make sure your package indexes are up to date, please execute the following command.


```
sudo apt-get update


```


If you have not yet enabled ssh in your UFW firewall setup, pease do so with the following commands.


```
sudo ufw allow ssh
sudo ufw enable


```


# Step 2 — Installing Dnsmasq


In this step we will install and configure Dnsmasq. Our privacy proxy server will use Dnsmasq to speed up and secure its DNS queries.


Every time you connect to a web page, your computer tries to resolve the Internet address of that server by asking a DNS (Domain Name System) server. Your computer uses the DNS servers of your ISP by default.


Using your own DNS server has the following advantages:


- Your ISP will not have any knowledge of the host names you connect to
- Your ISP cannot redirect your requests to other servers, which is one of the main methods of censorship
- Your DNS lookup speed will improve


The DNS servers you choose will know about all the DNS requests you make to them and can use this information to profile your browsing habits, redirect your searches to their own engines, or prevent your access to unapproved web sites. Choose your DNS servers wisely. OpenDNS and Google DNS servers are generally considered safe.

On a Debian system, nameserver configuration is kept in a file named /etc/resolv.conf.


Check your current nameserver configuration with the following command.


```
cat /etc/resolv.conf


```


Output:


/etc/resolv.conf
```
# Dynamic resolv.conf(5) file for glibc resolver(3) generated by resolvconf(8)
#     DO NOT EDIT THIS FILE BY HAND -- YOUR CHANGES WILL BE OVERWRITTEN
nameserver 8.8.8.8
nameserver 8.8.4.4

```


As you can see, the default nameservers on this system are set to Google’s DNS servers.


Now install dnsmasq with the following command:


```
sudo apt-get install dnsmasq


```


After the package is installed check your configuration again:


```
cat /etc/resolv.conf


```


Output:


/etc/resolv.conf
```
# Dynamic resolv.conf(5) file for glibc resolver(3) generated by resolvconf(8)
#     DO NOT EDIT THIS FILE BY HAND -- YOUR CHANGES WILL BE OVERWRITTEN
nameserver 127.0.0.1

```


The default nameserver is set to 127.0.0.1, which is the local interface Dnsmasq runs on.


You can test the installation with the following command. Take note of the query time in the output.


```
dig digitalocean.com @localhost


```


Output:


```
Output. . .

;; Query time: 20 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)

. . .

```


Now run the same command again and check the query time:


```
dig digitalocean.com @localhost


```


Output:


```
Output. . .

;; Query time: 1 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)

. . .

```


Our second query is answered by dnsmasq from cache. The response time went down from 20 milliseconds to 1 millisecond. Depending on the load of your system, the cached results are usually returned in under 1 millisecond.


# Step 3 — Installing ClamAV


Let’s install our antivirus scanner so our VPN will protect us from known malicious downloads.


## Install ClamAV


ClamAV is a widely used open-source antivirus scanner.


Install ClamAV and its scanner deamon:


```
sudo apt-get install clamav clamav-daemon


```


## Update Virus Database


ClamAV will update its database right after the installation and check for updates every hour.


ClamAV logs its database update status to /var/log/clamav/freshclam.log. You can check this file to see how its automatic updates are processing.


Now we will wait until automatic updates are completed; otherwise, our scanning proxy (HAVP) will complain and will not start.


```
sudo tail -f /var/log/clamav/freshclam.log


```


During update progress, the current status will be written to screen.


```
OutputFri Jun 19 12:56:03 2015 -> ClamAV update process started at Fri Jun 19 12:56:03 2015
Fri Jun 19 12:56:12 2015 -> Downloading main.cvd [100%]
Fri Jun 19 12:56:21 2015 -> main.cvd updated (version: 55, sigs: 2424225, f-level: 60, builder: neo)
Fri Jun 19 12:56:28 2015 -> Downloading daily.cvd [100%]
Fri Jun 19 12:56:34 2015 -> daily.cvd updated (version: 20585, sigs: 1430267, f-level: 63, builder: neo)
Fri Jun 19 12:56:35 2015 -> Downloading bytecode.cvd [100%]
Fri Jun 19 12:56:35 2015 -> bytecode.cvd updated (version: 260, sigs: 47, f-level: 63, builder: shurley)
Fri Jun 19 12:56:41 2015 -> Database updated (3854539 signatures) from db.local.clamav.net (IP: 200.236.31.1)
Fri Jun 19 12:56:55 2015 -> Clamd successfully notified about the update.
Fri Jun 19 12:56:55 2015 -> --------------------------------------

```


Wait until you see the text marked in red, Clamd successfully notified about the update..


Press CTRL+C on your keyboard to exit the tail. This will return you to the command prompt.


You can continue with the Configure ClamAV section if everything went normally.


## (Optional) Troubleshooting


If the virus update takes too long, you can invoke it manually. This will not be needed in normal circumstances.


Stop the autoupdate service.


```
sudo service clamav-freshclam stop


```


Invoke the updater manually and wait for its completion. Download progress will be shown in percentages.


```
sudo freshclam


```


Start the autoupdate service:


```
sudo service clamav-freshclam start


```


## Configure ClamAV


Now we will allow other groups to access ClamAV. This is needed because we will configure a virus scanning proxy (HAVP) to use ClamAV in the following steps.


Edit the ClamAV configuration file clamd.conf with your favorite text editor.


```
sudo vi /etc/clamav/clamd.conf


```


Set the following parameter to true.


/etc/clamav/clamd.conf
```
AllowSupplementaryGroups true

```


Save the configuration and exit.


Restart clamav-daemon


```
sudo service clamav-daemon restart


```


# Step 4 — Installing HAVP


HAVP is a virus scanning proxy server. It scans every item on the pages you visit and blocks malicious content. HAVP does not contain a virus scanner engine but can use quite a few third party engines. In this tutorial we will configure it with ClamAV.


Install HAVP from Debian repositories.


```
sudo apt-get install havp


```



If there is not enough memory for ClamAV libraries, HAVP might not start. You can ignore this error (for now) and continue with the setup.

Installation will take a while, so please be patient.


## Editing the Configuration File


Load HAVP’s configuration file in your favorite editor:


```
sudo vi /etc/havp/havp.config


```


We will need to set a few configuration options to make HAVP run with the ClamAV daemon.


HAVP can work with the ClamAV libraries (by default) or the ClamAV daemon. Library mode requires much more RAM than daemon (socket scanner) mode. If your Droplet has 4 GB or more of RAM, you can set ENABLECLAMLIB to true and use library mode.


Otherwise, use these settings, located near the bottom of the configuration file.


/etc/havp/havp.config
```
ENABLECLAMLIB false

. . .

ENABLECLAMD true

```


HAVP’s default configuration might interfere with some video streaming sites. To allow HTTP Range Requests, set the following parameter.


/etc/havp/havp.config
```
RANGE true

```


A lot of content on the Internet consists of images. Although there are some exploits that uses images as vectors, it is more or less safe not to scan images.


We recommend setting SCANIMAGES to false, but you can leave this setting as true if you want HAVP to scan images.


/etc/havp/havp.config
```
SCANIMAGES false

```


Do not scan files that have image, video, and audio MIME types. This setting will improve performance and enable you to watch streaming video content (provided the VPN as a whole has enough bandwidth). Uncomment this line to enable it.


/etc/havp/havp.config
```
SKIPMIME image/* video/* audio/*

```


There is one more parameter that we will change.


This parameter will tell HAVP not to log successful requests to the log file at /var/log/havp/access.log. Leave the default value (true) if you want to check the access logs to see if HAVP is working. For production, set this parameter to false in order to improve performance and privacy.


/etc/havp/havp.config
```
LOG_OKS false

```


Save your changes and exit the file.


## User Configuration


Remember when we configured ClamAV to be accessed by other groups?


Now, we will add the clamav user to the havp group and allow HAVP to access ClamAV. Execute the following command:


```
sudo gpasswd -a clamav havp


```


Output:


```
OutputAdding user clamav to group havp

```


We need to restart clamav-daemon for our changes to groups to take effect.


```
sudo service clamav-daemon restart


```


Now that we’ve configured HAVP, we can start it with the following command:


```
sudo service havp restart


```


Service restart commands should complete silently; there should be no messages displayed on the console.


## Checking the Logs


HAVP stores its log files in the /var/log/havp directory. Error and initialization messages goes into the error.log file. You can check the status of HAVP by checking this file.


```
sudo tail /var/log/havp/error.log


```


The tail command displays the last few lines of the file. If HAVP has started successfully, you will see something like the output shown below. Of course, the date and time will be your system’s:


```
Output17/06/2015 12:48:13 === Starting HAVP Version: 0.92
17/06/2015 12:48:13 Running as user: havp, group: havp
17/06/2015 12:48:13 --- Initializing Clamd Socket Scanner
17/06/2015 12:48:22 Clamd Socket Scanner passed EICAR virus test (Eicar-Test-Signature)
17/06/2015 12:48:22 --- All scanners initialized
17/06/2015 12:48:22 Process ID: 3896

```


# Step 5 — Testing HAVP


In this section we’ll make sure HAVP is actually blocking viruses.


The log shown above mentions something called the EICAR virus test.


On initialization HAVP tests the virus scanner engines with a specially constructed virus signature. All virus scanner software detects files that contain this (harmless) signature as a virus. You can get more information about EICAR on the EICAR Intended Use page.


Let’s do our own manual test with the EICAR file and see that HAVP and ClamAV block it.


We will use the wget command line utility to download file from EICAR web page.


First, download the EICAR test file without using a proxy:


```
wget http://www.eicar.org/download/eicar.com -O /tmp/eicar.com


```


Your server will download the file without complaint:


```
Outputconverted 'http://www.eicar.org/download/eicar.com' (ISO-8859-1) -> 'http://www.eicar.org/download/eicar.com' (UTF-8)
--2015-06-16 13:53:41--  http://www.eicar.org/download/eicar.com
Resolving www.eicar.org (www.eicar.org)... 188.40.238.250
Connecting to www.eicar.org (www.eicar.org)|188.40.238.250|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 68 [application/octet-stream]
Saving to: '/tmp/eicar.com'

/tmp/eicar.com       100%[=====================>]      68  --.-KB/s   in 0s

2015-06-16 13:53:41 (13.7 MB/s) - '/tmp/eicar.com' saved [68/68]

```


As you can see, wget downloaded the test file containing the virus signature without any complaints.


Now let’s try to download the same file with our newly-configured proxy. We will set the environment variable http_proxy to our HAVP address and port.


```
http_proxy=127.0.0.1:8080 wget http://www.eicar.org/download/eicar.com -O /tmp/eicar.com


```


Output:


```
Outputconverted 'http://www.eicar.org/download/eicar.com' (ISO-8859-1) -> 'http://www.eicar.org/download/eicar.com' (UTF-8)
--2015-06-25 20:47:38--  http://www.eicar.org/download/eicar.com
Connecting to 127.0.0.1:8080... connected.
Proxy request sent, awaiting response... 403 Virus found by HAVP
2015-06-25 20:47:39 ERROR 403: Virus found by HAVP.

```


Our proxy successfully intercepted the download and blocked the virus.


EICAR also provides a virus signature file hidden inside a ZIP compressed file.


You can test that HAVP scans files inside ZIP archives with the following command:


```
http_proxy=127.0.0.1:8080 wget http://www.eicar.org/download/eicarcom2.zip -O /tmp/eicarcom2.zip


```


Output:


```
Outputconverted 'http://www.eicar.org/download/eicarcom2.zip' (ISO-8859-1) -> 'http://www.eicar.org/download/eicarcom2.zip' (UTF-8)
--2015-06-25 20:48:28--  http://www.eicar.org/download/eicarcom2.zip
Connecting to 127.0.0.1:8080... connected.
Proxy request sent, awaiting response... 403 Virus found by HAVP
2015-06-25 20:48:28 ERROR 403: Virus found by HAVP.

```


HAVP (with ClamAV) found the virus again.


# Step 6 — Installing Privoxy


So far we have configured a proxy server to scan web pages for viruses. What about ads and tracking cookies? In this step we will install and configure Privoxy.



Blocking advertisements is harmful to the web sites that rely on advertisements to cover operational costs. Please consider adding exceptions to the sites that you trust and frequent.

Use the following command to install Privoxy:


```
sudo apt-get install privoxy


```


Privoxy’s configuration resides in the file /etc/privoxy/config. We need to set two parameters before we start using Privoxy.


Open the config file in your favorite editor.


```
sudo vi /etc/privoxy/config


```


Now uncomment and set the following two parameters:


/etc/privoxy/config
```
listen-address  127.0.0.1:8118

. . .

hostname your_server

```


The parameter listen-address determines on which IP and port privoxy runs. The default value is localhost:8118; we will change this to 127.0.0.1:8118.


The parameter hostname specifies the host Privoxy runs on and logs; set this to the hostname or DNS address of your server. It can be any valid hostname.


Now, restart Privoxy with its new configuration.


```
sudo service privoxy restart


```


# Step 7 — Chaining HAVP to Privoxy


HAVP and Privoxy both are essentially HTTP proxy servers. We will now chain these two proxies so that, when your client requests a web page from HAVP, it will forward this request to Privoxy. Privoxy will retrieve the requested web page, remove the privacy threats and ads, and then HAVP will further process the response and remove viruses and malicious code.


Load the HAVP configuration file into your favorite text editor:


```
sudo vi /etc/havp/havp.config


```


Uncomment the following lines (remove the # character at the beginning of the lines) and set their values as shown below. Privoxy runs on IP 127.0.0.1 and port 8118.


/etc/havp/havp.config
```
PARENTPROXY 127.0.0.1
PARENTPORT 8118

```


Save your changes and exit the file.


Restart HAVP for the changes to take effect:


```
sudo service havp restart


```


Check HAVP’s error log, taking note of the Use parent proxy: 127.0.0.1:8118 message.


```
sudo tail /var/log/havp/error.log


```


Output:


```
Output17/06/2015 12:57:37 === Starting HAVP Version: 0.92
17/06/2015 12:57:37 Running as user: havp, group: havp
17/06/2015 12:57:37 Use parent proxy: 127.0.0.1:8118
17/06/2015 12:57:37 --- Initializing Clamd Socket Scanner
17/06/2015 12:57:37 Clamd Socket Scanner passed EICAR virus test (Eicar-Test-Signature)
17/06/2015 12:57:37 --- All scanners initialized
17/06/2015 12:57:37 Process ID: 4646

```


Our proxy server configuration is now complete. Lets test it again with the EICAR virus test.


```
http_proxy=127.0.0.1:8080 wget http://www.eicar.org/download/eicarcom2.zip -O /tmp/eicarcom2.zip


```


If your configuration is good, you should again see the ERROR 403: Virus found by HAVP message.


# Step 8 — Setting DNS Options for OpenVPN Server


Although the default configuration of OpenVPN Server is adequate for our needs, it is possible to improve it a little bit more.


Load the OpenVPN server’s configuration file in a text editor:


```
sudo vi /etc/openvpn/server.conf


```


OpenVPN is configured to use OpenDNS’s servers by default. If you want to change it to use Google’s DNS servers, change the dhcp-option DNS parameters as below.


Add the new line push "register-dns", which some Windows clients might need in order to use the DNS servers.


Also, add the new line push "block-ipv6" to block IPv6 while connected to VPN. (IPv6 traffic can bypass our VPN server.)


Here’s what this section should look like:


/etc/openvpn/server.conf
```
push "dhcp-option DNS 8.8.8.8"
push "dhcp-option DNS 8.8.4.4"
push "register-dns"
push "block-ipv6"

```


If you want to allow multiple clients to connect with the same ovpn file, uncomment the following line. (This is convenient but NOT more secure!)


/etc/openvpn/server.conf
```
duplicate-cn

```


Restart the OpenVPN service for changes to take effect.


```
sudo service openvpn restart


```


# Step 9 — Configuring Your Transparent Proxy


We will now set up our privacy server to intercept the HTTP traffic between its clients (your browser) and the internet.


## Enable Packet Forwarding


For our server to forward HTTP traffic to the proxy server, we need to enable packet forwarding. You should have enabled it already in the OpenVPN setup tutorial.


Test the configuration with the following command.


```
sudo sysctl -p


```


It should display the changed parameters as below. If it does not, please revisit the OpenVPN tutorial.


```
Outputnet.ipv4.ip_forward = 1

```


## Configure UFW


We need to forward HTTP packets that originate from OpenVPN clients to HAVP. We will use ufw for this purpose.


First we need to allow traffic originating from OpenVPN clients


```
sudo ufw allow in on tun0 from 10.8.0.0/24


```


In the OpenVPN tutorial, you should have changed the /etc/ufw/before.rules file and added some rules for OpenVPN. Now we will revisit the same file and configure port redirection for the transparent proxy.


```
sudo vi /etc/ufw/before.rules


```


Change the lines you have added in the OpenVPN configuration as shown below. Add the lines in red.


/etc/ufw/before.rules
```
 # START OPENVPN RULES
 # NAT table rules
*nat
:PREROUTING ACCEPT [0:0]
:POSTROUTING ACCEPT [0:0]
# transparent proxy
-A PREROUTING -i tun+ -p tcp --dport 80 -j REDIRECT --to-port 8080
 # Allow traffic from OpenVPN client to eth0
-A POSTROUTING -s 10.8.0.0/8 -o eth0 -j MASQUERADE
COMMIT
 # END OPENVPN RULES

```


Reload your firewall configuration.


```
sudo ufw reload


```


Check UFW’s status:


```
sudo ufw status


```


Output:


```
OutputStatus: active

To                         Action      From
--                         ------      ----
22                         ALLOW       Anywhere
1194/udp                   ALLOW       Anywhere
Anywhere on tun0           ALLOW       10.8.0.0/24
22                         ALLOW       Anywhere (v6)
1194/udp                   ALLOW       Anywhere (v6)

```


## Enable HAVP’s Transparent Mode


In the previous steps, we forced all HTTP packets to go through HAVP. This configuration is called a transparent proxy.


We need to configure HAVP as such.


```
sudo vi /etc/havp/havp.config


```


Set the following parameter:


/etc/havp/havp.config
```
TRANSPARENT true

```


Restart the HAVP service:


```
sudo service havp restart


```


Our server is now ready to use.


# Step 10 — Testing Client Configuration


On your client (Windows, OS X, tablet …) connect your client to your OpenVPN server. Note that you can use the same .ovpn file from the original OpenVPN tutorial; all the changes are on the server side.



For detailed setup instructions for your OpenVPN client, please see Installing the Client Profile in the Ubuntu 14.04 tutorial.

After the VPN connection is established, you should see your preferred DNS settings in the OpenVPN client logs. The following sample is taken from the IOS client.


```
DNS Servers
	8.8.8.8
	8.8.4.4
Search Domains:

```


If you use Tunnelblick, you might see a line like this:


```
Changed DNS ServerAddresses setting from '8.8.8.8 208.67.222.222 8.8.4.4' to '8.8.8.8 8.8.4.4'

```


To test your configuration, go to the EICAR test page in your browser and attempt to download the EICAR test file. You should see a HAVP - Access Denied page.


- http://www.eicar.org/download/eicarcom2.zip
- http://www.eicar.org/85-0-Download.html




# Step 11 — Troubleshooting


This section will help you troubleshoot some common issues.


## Cannot watch videos or use my favorite site


Privoxy can be configured to be less strict with sites that are loading too slowly. This behavior is configured in the user.action configuration file.


Load the user action file in your favorite text editor.


```
sudo vi /etc/privoxy/user.action


```


Go to the end of file and add the following content with the additional site addresses you want.


/etc/privoxy/user.action
```
{ fragile -deanimate-gifs }
.googlevideo.com
.youtube.com
.imgur.com
.example.com

```


After these changes, you do not need to restart Privoxy. However, you should clear your browser’s cache and refresh a few times.


If you still experience problems, add whitelisted domains to the HAVP whitelist file. HAVP will check this file and not perform a virus scan if the host name matches.


```
vi /etc/havp/whitelist


```


Add your sites at the end of the file.


/etc/havp/whitelist
```
# Whitelist Windowsupdate, so RANGE is allowed too
*.microsoft.com/*
*.windowsupdate.com/*

*.youtube.com/*

```


## Browser stops responding during heavy use of Internet


If you open multiple web pages at once, your server’s memory might not be enough for HAVP to scan all your requests.


You can try to increase your Droplet’s RAM and/or add swap memory. Please refer to the How To Configure Virtual Memory (Swap File) on a VPS article.


Keep in mind that adding a VPN to your browsing experience will add some latency in most cases.


# Conclusion


After following this tutorial, you’ll have taken your VPN use to the next level with browsing privacy and security.


