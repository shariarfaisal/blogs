# How to Configure NTP for Use in the NTP Pool Project on CentOS 7

```Miscellaneous``` ```Linux Commands``` ```Networking``` ```CentOS```

## Introduction


Accurate time keeping is critical for almost any service or software. Emails, loggers, event systems and schedulers, user authentication mechanisms, and services running on distributed platforms all need accurate timestamps to record events in chronological order. These services use the Network Time Protocol, or NTP, to synchronize the system clock with a trusted external source. This source can be an atomic clock, a GPS receiver, or another time server that already uses NTP.


This is where the NTP Pool Project project comes into play.  It’s a huge worldwide cluster of time servers that provides easy access to known “good time” for tens of millions of clients around the world. It’s the default time server for Ubuntu and most of the other major Linux distributions, as well as many networked appliances and software applications.


In this guide, you will set up NTP on your server and configure it to be part of the NTP Pool Project, so it provides accurate time to other users of the NTP Pool Project. Providing your spare CPU cycles and unused bandwidth is a perfect way to give something back to the community.


The required bandwidth is relatively low and can be adjusted depending on the amount you can provide and where your server resides. Each client will only send a couple of UDP packets every 20 minutes, so most servers only receive about a dozen NTP packets per second, with spikes a couple of times a day of up to one hundred packets per second. This translates to bandwidth usage of 10-15Kb/sec with spikes of 50-120Kb/sec.


There are three basic requirements you must satisfy before joining the NTP Pool Project:


1. Your server must have a static IP address.
2. Your server must have a permanent and stable internet connection.
3. Your IP address most not change, or only changes infrequently (once a year or less).

For most cloud-based servers, the first two requirements are usually met automatically. The third requirement emphasizes that joining the NTP Pool Project constitutes a long-term commitment. Of course, if your circumstances change, it’s fine to take a server out of the pool, but it will take a long time (mostly weeks, but sometimes  months or even years) before the traffic completely vanishes.


# Prerequisites


To complete this tutorial, you will need:


- One Centos 7 server with IPv6 networking configured. If you need to configure IPv6 networking on an exising Droplet, you can follow this tutorial.
- A sudo non-root user and a firewall, which you can set up by following the Initial Server Setup with CentOS 7 tutorial and the “Configuring a Basic Firewall” section in the Additional Recommended Steps for New CentOS 7 Servers tutorial.

# Step 1 — Installing NTP


The NTP package is not installed by default, so you’ll use the package manager to install it. First, update your packages:


```
sudo yum update


```


Then install NTP:


```
sudo yum install ntp


```


Once the installation completes, start the service and configure it so it starts automatically each time the server boots:


```
sudo systemctl start ntpd
sudo systemctl enable ntpd


```


If you’ve configured the firewall as specified in the prerequisites, you must allow UDP traffic for the NTP service in order to communicate with the NTP pool:


```
sudo firewall-cmd --permanent --add-service=ntp
sudo firewall-cmd --reload


```


For more on FirewallD, refer to How To Set Up a Firewall Using FirewallD on CentOS 7.


NTP is now installed, but it’s configured to use the default NTP pool time servers. Lets pick some specific time servers instead.


# Step 2 — Choosing a Suitable Upstream Server


The NTP Pool project asks operators who want to join the pool to choose good network-local time servers rather than using the default pool.ntp.org servers. This ensures that the NTP Pool Project remains reliable, fast, and healthy. When choosing your time source, you’ll want a stable network connection with no packet loss and as few hops as possible between the servers.


The multi-tiered and hierarchical NTP protocol separates the parties involved into primary servers, secondary servers, and clients. The primary servers are called Stratum 1 and are connected directly to the source of time, which is called Stratum 0. This source can be an atomic clock, a GPS receiver, or a radio navigation system. Secondary servers in the chain are called Stratum 2, Stratum 3 and so on.


Each server is also a client. A Stratum 2 client receives time from an upstream Stratum 1 server, and provides time to downstream Stratum 3 servers or other clients. For NTP Pool Project members to work properly, the NTP daemon needs at least three servers configured. The project recommends a minimum of four, and no more than seven sources.


The NTP Pool Project provides a list of public Stratum 1 and Startum 2  time servers. The lists designate the NTP time servers available for public access under stated restrictions. You’ll find three types:


- OpenAccess: This time server is open to any client complying with the NTP Pool usage recommendations.
- RestrictedAccess: This time server has some access restrictions in addition to the NTP Pool usage recommendations.
- ClosedAccess:  This time server is closed or requires prior arrangement.


Warning: Don’t use servers that are not listed as OpenAccess unless you’ve received approval to do so.

Visit the Stratum 1 Time Servers list. You’ll see a list like the following:





Sort the list by the ISO code column and find one or two servers that are geographically close to your server’s data center. When the server’s Access Policy column states OpenAccess, you can use it without issue. If it says “RestrictedAccess”, click to open the entry and read the instructions noted in the AccessDetails field. Often, you’ll find that NotificationMessage is set to Yes, which means you have to craft an informal email directed to the address provided in ServerContact, informing the server operator about your desire to use this time server as a time source for your NTP Pool Project member.


Once you’ve identified the servers you’d like to use, click the link for each server in the ISO column and copy its host name or IP address. You’ll use these addresses in Step 3.


Next, select three or four servers from the  Stratum 2 list, following the same process.


Once you have selected your time servers, it’s time to configure your NTP client to use them.


# Step 3 — Configuring NTP to Join the Pool


To use your server with the NTP pool, and configure your new time servers, you’ll need to make some modifications to your NTP daemon’s configuration. To do so, edit the /etc/ntp.conf file:


```
sudo vi /etc/ntp.conf


```


First, make sure a driftfile is configured. A driftfile stores the frequency offset between the system clock running at its nominal frequency, and the frequency required to remain in synchronization with correct time. It helps to achieve a stable and accurate time. You should find this at the top of your configuration file on a default installation:


/etc/ntp.conf
```
# For more information about this file, see the man pages
# ntp.conf(5), ntp_acc(5), ntp_auth(5), ntp_clock(5), ntp_misc(5), ntp_mon(5).

driftfile /var/lib/ntp/drift

...

```


Next, remove the default time source entries from the configuration. You’re looking for all lines which are of the pattern server 0.centos.pool.ntp.org iburst. If you’re using a default configuration, remove the highlighted lines as shown in the following example:


/etc/ntp.conf
```
...
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
server 0.centos.pool.ntp.org iburst
server 1.centos.pool.ntp.org iburst
server 2.centos.pool.ntp.org iburst
server 3.centos.pool.ntp.org iburst

```


Replace the lines you removed with the hand-picked servers you selected in the previous step.


/etc/ntp.conf
```
...
server ntp_server_hostname_1 iburst
server ntp_server_hostname_2 iburst
server ntp_server_hostname_3 iburst
server ntp_server_hostname_4 iburst
server ntp_server_hostname_5 iburst
...

```


We use the iburst option for each servers, per the NTP Pool recommendations. That way, if the server is unreachable, this will send a burst of eight packets instead of the usual one packet. Using the burst option in the NTP Pool Project is considered abuse as it will send those eight packets every poll interval, whereas iburst sends the eight packets only the first time.


Next, make sure the default configuration does not allow management queries. If you don’t, your server could be used in NTP reflection attacks, or could be vulnerable to ntpq and ntpdc queries that attempt to modify the state of the server. Check that the noquery option is added to the default restrict lines. Also make sure you add the options kod and limited as they restrict too eagerly asking clients and enforce rate limiting.


/etc/ntp.conf
```
...
# Permit time synchronization with our time source, but do not
# permit the source to query or modify the service on this system.
restrict default nomodify notrap nopeer noquery kod limited

# Permit all access over the loopback interface.  This could
# be tightened as well, but to do so would effect some of
# the administrative functions.
restrict 127.0.0.1
restrict ::1

```


You can find more information about the other options in the official documentation.


Your NTP daemon configuration file now should look like the following, although your file may have additional comments, which you can safely disregard:


/etc/ntp.conf
```

driftfile /var/lib/ntp/ntp.drift

restrict default nomodify notrap nopeer noquery kod limited

restrict 127.0.0.1
restrict ::1

server ntp_server_hostname_1 iburst
server ntp_server_hostname_2 iburst
server ntp_server_hostname_3 iburst
server ntp_server_hostname_4 iburst
server ntp_server_hostname_5 iburst

```


Save the file and exit the editor.


Now restart the NTP service and let your time server synchronize its clock to the upstream servers.


```
sudo systemctl restart ntpd


```


After a few minutes, check the health of your time server with the ntpq command:


```
ntpq -p


```


The output should look similar to this:


```
Output     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
 mizbeaver.udel. .INIT.          16 u    -   64    0    0.000    0.000   0.000
 montpelier.ilan .GPS.            1 u   25   64    7   55.190    2.121 130.492
+nist1-lnk.binar .ACTS.           1 u   28   64    7   52.728   23.860   3.247
*ntp.okstate.edu .GPS.            1 u   31   64    7   19.708   -8.344   6.853
+ntp.colby.edu   .GPS.            1 u   34   64    7   51.518   -5.914   6.669

```


The remote column tells you the hostname of the servers the NTP daemon is using, and the refid column tells you the source the servers are using. So for Stratum 1 servers, the refid field should show GPS, PPS, ACTS, or PTB, and Stratum 2 and higher servers will show the IP address of the upstream server. The st column shows the stratum, and delay, offset and jitter tell you about the quality of the time source. Lower values are better for these three fields.


Your time server is now able to serve time to the public. You can verify this by calling ntpdate from another host:


```
ntpdate -q your_server_ip


```


The output should look similar to this and it tells you it adjusted the time server and the offset:


```
Outputserver your_server_ip, stratum 2, offset 0.001172, delay 0.16428
 2 Mar 23:06:44 ntpdate[18427]: adjust time server your_server_ip offset 0.001172 sec

```


You are now ready to register your NTP server with the NTP Pool Project so others can use it.


# Step 4 — Adding the Server to the NTP Pool


To add your server so others can use it, visit manage.ntppool.org and sign up for an account. You will receive an email from NTP Pool help@ntppool.org requesting that you verify your account. Confirm your account by following the instructions in the email, and then log in to manage.ntppool.org.


Once logged in, you’ll see the simple interface for adding servers:





Enter your server’s IP address and click Submit.


The next screen asks you to verify that it identified the region of your server. If it shows your server in a different region than you expect, use the Comment box to let them know.





If you are happy, confirm the entry by clicking Yes, this is my server, add it!


Your server is now part of the NTP Pool Project. Visit http://www.pool.ntp.org/scores/your_server_ip to  see information the NTP Pool’s monitoring system has collected about your server. It checks your server a few times per hour and displays offset data, alog with the score of your system. As long as your server is keeping good time and is reachable, the score will rise untill it reaches 20 points. Only servers with a score higher than 10 are used in the pool.


## Troubleshooting Connectivity Issues


If you are having trouble getting your server to sync you might have a packet firewall in place dropping your outgoing packets on port 123. Take a look at How To Set Up a Firewall Using FirewallD on CentOS 7 to learn how to check the status of the firewall.


If the NTP Pool Project’s monitoring station can’t reach your NTP server and your server score is going down, or you can’t use your server to sync some other clock, you might have a packet firewall in place dropping your incoming traffic on port 123. Check your firewall status.


If you are certain that you have no firewall in place, or you have opened port 123 for both incoming and outgoing traffic, your server provider or another transit provider might be dropping your packets along the way. If you do not have the knowledge to solve those problems on your own, it’s best to turn to the community and reach for help. The NTP Pool Projects forum is a good place to start.  You can also join the mailing list or send an emaill to the NTP Pool Project operator. Just be sure you can show all the steps you’ve already tried to resolve the issue before asking for help.


# Conclusion


In this tutorial, you successfully set up your own time server and made it a member of the NTP Pool Project, serving time to the community.  To keep in touch with the time-keeping community. join the NTP Pool Projects forum or the mailing list. Be sure to monitor your server’s score and make any adjustments necessary.


