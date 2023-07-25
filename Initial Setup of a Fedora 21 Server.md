# Initial Setup of a Fedora 21 Server

```Linux Basics``` ```Fedora```

## Introduction


When you first log into a fresh Fedora 21 or RHEL server, it’s not ready for use as a production system. There are a number of recommended steps to take in order to customize and secure it, such as enabling a firewall.


This tutorial will show you how to give a fresh installation of a Fedora 21 server a better security profile and be ready for use.


# Prerequisites


To follow this tutorial, you will need:


- A Fedora 21 Droplet with root SSH keys.

You can follow this section of the SSH key tutorial to create keys if you don’t have them, and this section of the same tutorial to automatically embed your SSH key in your server’s root account when you create your Droplet.


# Step 1 — Creating a Standard User Account


First, log into your server as root.


```
ssh root@your_server_ip

```


Operating as root is a security risk, so in this step, we’ll set up a sudo non-root user account to use for system and other computing tasks. The username used in this tutorial is sammy, but you can use any name you like.


To add the user, type:


```
adduser sammy

```


Specify a strong password for the user using the command below. You’ll be prompted to enter the password twice.


```
passwd sammy

```


Then add the user to the wheel group, which gives it sudo privileges.


```
gpasswd -a sammy wheel

```


Log out of your server and add your SSH key to the new user account by running the following on your local machine.


```
ssh-copy-id sammy@your_server_ip

```


For more information on how to copy your SSH keys from your local machine to your server, you can read this section of the SSH tutorial.


Finally, log back in as the new sudo non-root user. You won’t be prompted for a password because this account now has SSH keys.


```
ssh sammy@your_server_ip

```


# Step 2 — Disallowing Root Login and Password Authentication


In this step, we’ll make SSH logins more secure by disabling root logins and password authentication.


To edit configuration files, you’ll need to install a text editor. We’ll use nano but you can use whichever is your favorite.


First, apply any available updates using:


```
sudo yum update

```


Then, to install nano, type:


```
sudo yum install -y nano

```


Now, open the the SSH daemon’s configuration file for editing.


```
sudo nano /etc/ssh/sshd_config

```


Inside that file, look for the PermitRootLogin directive. Uncomment it (that means remove the starting # character) and set it to no.


```
PermitRootLogin no

```


Similarly, look for the PasswordAuthentication directive and set it to no as well.


```
PasswordAuthentication no

```


Save and exit the file, then reload the configuration to put your changes into place.


```
sudo systemctl reload sshd

```


If anyone tries to log in as root now, the response should be Permission denied (publickey,gssapi-keyex,gssapi-with-mic).


# Step 3 ­— Configuring the Time Zone


In this step, you’ll read how to change the system clock to your local time zone. The default clock is set to UTC.


All the known timezones are under the /usr/share/zoneinfo/ directory. Take a look at the files and directories in /usr/share/zoneinfo/.


```
ls /usr/share/zoneinfo/

```


To set the clock to use the local timezone, find your country or geographical area in that directory, locate the zone file under it, then create a symbolic soft link from it to the /etc/localtime directory. For example, if you’re in the central part of the United States, where the timezone is Central, or CST, the zone file will be /usr/share/zoneinfo/US/Central.


Create a symbolic soft link from your zone file to /etc/localtime.


```
sudo ln -sf /usr/share/zoneinfo/your_zone_file /etc/localtime

```


Verify that the clock is now set to local time by viewing the output of the date command.


```
date

```


The output will look something like:


```
Wed Mar 25 14:41:20 CST 2015

```


The CST in that output confirms that it’s Central time.


# Step 4 — Enabling a Firewall


A new Fedora 21 server has no active firewall application. In this step, we’ll learn how to enable the IPTables firewall application and make sure that runtime rules persist after a reboot.


The IPTables package is already installed, but to be enable to enable it, you need to install the iptables-services package.


```
sudo yum install -y iptables-services

```


You may then enable IPTables so that it automatically starts on boot.


```
sudo systemctl enable iptables

```


Next, start IPTables.


```
sudo systemctl start iptables

```


IPTables on Fedora 21 ships with a default set of rules. One of those rules permits SSH traffic. To view the default rules, type:


```
sudo iptables -L

```


The output should read:


```
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
ACCEPT     all  --  anywhere             anywhere             state RELATED,ESTABLISHED
ACCEPT     icmp --  anywhere             anywhere
ACCEPT     all  --  anywhere             anywhere
ACCEPT     tcp  --  anywhere             anywhere             state NEW tcp dpt:ssh
REJECT     all  --  anywhere             anywhere             reject-with icmp-host-prohibited

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination
REJECT     all  --  anywhere             anywhere             reject-with icmp-host-prohibited

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination

```


Those rules are runtime rules and will be lost if the system is rebooted. To save the current runtime rules to a file so that they persist after a reboot, type:


```
sudo /usr/libexec/iptables/iptables.init save

```


The rules are now saved to a file called iptables in the /etc/sysconfig directory.


# Step 5 (Optional) — Allowing HTTP and HTTPS Traffic


In this section, we’ll cover how to edit the firewall rules to allow services for ports 80 (HTTP) and 443 (HTTPS).


The default IPTables rules allow SSH traffic in by default, but HTTP and its relatively more secure cousin, HTTPS, are services that many applications use, so you may want to allow these to pass through the firewall as well.


To proceed, open the firewall rules file by typing:


```
sudo nano /etc/sysconfig/iptables

```


All you need to do is add two rules, one for port 80 and another for port 443, after the rule for SSH (port 22) traffic. The lines below in red are the ones you will add; the lines before and after are included for context to help you find where to add the new rules.


```
-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 443 -j ACCEPT
-A INPUT -j REJECT --reject-with icmp-host-prohibited

```


To activate the new ruleset, restart IPTables.


```
sudo systemctl restart iptables

```


# Step 6 (Optional) - Installing Mlocate


The locate command is a very useful utility for looking up the location of files in the system. For example, to find a file called example, you would type:


```
locate example

```


That will scan the file system and print the location or locations of the file on your screen. There are more advanced ways of using locate, too.


To make the command available on your server, first you need to install the mlocate package.


```
sudo yum install -y mlocate

```


Then, run the updatedb command to update the search database.


```
sudo updatedb

```


After that, you should be able to use locate to find any file by name.


## Conclusion


After completing the last step, your Fedora 21 server should be configured, reasonably secure, and ready for use!


