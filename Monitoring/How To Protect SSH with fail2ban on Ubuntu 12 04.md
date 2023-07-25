# How To Protect SSH with fail2ban on Ubuntu 12 04

```Security``` ```Ubuntu``` ```Monitoring```











# Status: Deprecated


This article covers a version of Ubuntu that is no longer supported.  If you are currently operate a server running Ubuntu 12.04, we highly recommend upgrading or migrating to a supported version of Ubuntu:


- Upgrade to Ubuntu 14.04.
- Upgrade from Ubuntu 14.04 to Ubuntu 16.04
- Migrate the server data to a supported version

Reason:
Ubuntu 12.04 reached end of life (EOL) on April 28, 2017 and no longer receives security patches or updates.  This guide is no longer maintained.


See Instead:
This guide might still be useful as a reference, but may not work on other Ubuntu releases.  If available, we strongly recommend using a guide written for the version of Ubuntu you are using.   You can use the search functionality at the top of the page to find a more recent version.


## About Fail2Ban


Servers do not exist in isolation, and those virtual private servers with only the most basic SSH configuration can be vulnerable to brute force attacks. fail2ban provides a way to automatically protect virtual servers from malicious behavior. The program works by scanning through log files and reacting to offending actions such as repeated failed login attempts.


# Step One—Install Fail2Ban


Use apt-get to install Fail2Ban


```
sudo apt-get install fail2ban
```


# Step Two—Copy the Configuration File


The default fail2ban configuration file is location at /etc/fail2ban/jail.conf. The configuration work should not be done in that file, however, and we should instead make a local copy of it.


```
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```


After the file is copied, you can make all of your changes within the new jail.local file. Many of possible services that may need protection are in the file already. Each is located in its own section, configured and turned off.


# Step Three—Configure the Defaults in Jail.Local


Open up the the new fail2ban configuration file:


```
sudo nano /etc/fail2ban/jail.local
```


The first section of defaults covers the basic rules that fail2ban will follow. If you want to set up more nuanced protection on your virtual server, you can customize the details in each section.


You can see the default section below.


```
[DEFAULT]

# "ignoreip" can be an IP address, a CIDR mask or a DNS host
ignoreip = 127.0.0.1/8
bantime  = 600
maxretry = 3

# "backend" specifies the backend used to get files modification. Available
# options are "gamin", "polling" and "auto".
# yoh: For some reason Debian shipped python-gamin didn't work as expected
#      This issue left ToDo, so polling is default backend for now
backend = auto

#
# Destination email address used solely for the interpolations in
# jail.{conf,local} configuration files.
destemail = root@localhost
```


Write your personal IP address into the ignoreip line. You can separate each address with a space. IgnoreIP allows you white list certain IP addresses and make sure that they are not locked out. Including your address will guarantee that you do not accidentally ban yourself from your own server.


The next step is to decide on a bantime, the number of seconds that a host would be blocked from the VPS if they are found to be in violation of any of the rules. This is especially useful in the case of bots, that once banned, will simply move on to the next target. The default is set for 10 minutes—you may raise this to an hour (or higher) if you like.


Maxretry is the amount of incorrect login attempts that a host may have before they get banned for the length of the ban time.


You can leave the backend as auto.


Destemail is the email that alerts get sent to. If you have a mail server set up on your droplet, Fail2Ban can email you when it bans an IP address.


## Additional Details—Actions


The Actions section is located below the defaults. The beginning looks like this:


```
#
# ACTIONS
#

# Default banning action (e.g. iptables, iptables-new,
# iptables-multiport, shorewall, etc) It is used to define
# action_* variables. Can be overridden globally or per
# section within jail.local file
banaction = iptables-multiport

# email action. Since 0.8.1 upstream fail2ban uses sendmail
# MTA for the mailing. Change mta configuration parameter to mail
# if you want to revert to conventional 'mail'.
mta = sendmail

# Default protocol
protocol = tcp
[...]
```


Banaction describes the steps that fail2ban will take to ban a matching IP address. This is a shorter version of the file extension where the config if is located. The default ban action, "iptables-multiport", can be found at /etc/fail2ban/action.d/iptables-multiport.conf


MTA refers to email program that fail2ban will use to send emails to call attention to a malicious IP. 


You can change the protocol from TCP to UDP in this line as well, depending on which one you want fail2ban to monitor.


# Step Four (Optional)—Configure the ssh-iptables Section in Jail.Local


The SSH details section is just a little further down in the config, and it is already set up and turned on. Although you should not be required to make any changes within this section, you can find the details about each line below.


```
[ssh]

enabled  = true
port     = ssh
filter   = sshd
logpath  = /var/log/auth.log
maxretry = 6
```


Enabled simply refers to the fact that SSH protection is on. You can turn it off with the word "false".


The port designates the port that fail2ban monitors. If you have set up your virtual private server on a non-standard port, change the port to match the one you are using:


```
 eg. port=30000
```


The filter, set by default to sshd, refers to the config file containing the rules that fail2ban uses to find matches. sshd refers to the /etc/fail2ban/filter.d/sshd.conf.


log path refers to the log location that fail2ban will track.


The  max retry line within the SSH section has the same definition as the default option. However, if you have enabled multiple services and want to have specific values for each one, you can set the new max retry amount for SSH here.


# Step Five—Restart Fail2Ban


After making any changes to the fail2ban config, always be sure to restart Fail2Ban:


```
sudo service fail2ban restart
```


You can see the rules that fail2ban puts in effect within the IP table:


```
sudo iptables -L
```


By Etel Sverdlov
