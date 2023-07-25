# How To Protect SSH with Fail2Ban on Rocky Linux 9

```Firewall``` ```Linux Basics``` ```Rocky Linux``` ```Rocky Linux 9``` ```Security```

## Introduction


SSH is the de facto method of connecting to a cloud server. It is durable, and it is extensible — as new encryption standards are developed, they can be used to generate new SSH keys, ensuring that the core protocol remains secure. However, no protocol or software stack is totally foolproof, and SSH being so widely deployed across the internet means that it represents a very predictable attack surface or attack vector through which people can try to gain access.


Any service that is exposed to the network is a potential target in this way. If you review the logs for your SSH service running on any widely trafficked server, you will often see repeated, systematic login attempts that represent brute force attacks by users and bots alike. Although you can make some optimizations to your SSH service to reduce the chance of these attacks succeeding to near-zero, such as disabling password authentication in favor of SSH keys, they can still pose a minor, ongoing liability.


Large-scale production deployments for whom this liability is completely unacceptable will usually implement a VPN such as WireGuard in front of their SSH service, so that it is impossible to connect directly to the default SSH port 22 from the outside internet without additional software abstraction or gateways. These VPN solutions are widely trusted, but will add complexity, and can break some automations or other small software hooks.


Prior to or in addition to committing to a full VPN setup, you can implement a tool called Fail2ban. Fail2ban can significantly mitigate brute force attacks by creating rules that  automatically alter your firewall configuration to ban specific IPs after a certain number of unsuccessful login attempts. This will allow your server to harden itself against these access attempts without intervention from you.


In this guide, you’ll see how to install and use Fail2ban on a Rocky Linux 9 server.


# Prerequisites


To complete this guide, you will need:


- 
A Rocky Linux 9 server and a non-root user with sudo privileges. You can learn more about how to set up a user with these privileges in our Initial Server Setup with Rocky Linux 9 guide. You should also have firewalld running on the server, which is covered in our initial server setup guide.

- 
Optionally, a second server that you can connect to your first server from, which you will use to test getting deliberately banned.


# Step 1 — Installing Fail2ban


Fail2ban is not available in Rocky’s default software repositories. However, it is available in the EPEL, or Enhanced Packages for Enterprise Linux repository, which is commonly used for third-party packages on Red Hat and Rocky Linux. If you have not already added EPEL to your system package sources, you can add the repository using dnf, like you would install any other package:


```
sudo dnf install epel-release -y


```


The dnf package manager will now check EPEL in addition to your default package sources when installing new software. Proceed to install Fail2ban:


```
sudo dnf install fail2ban -y


```


Fail2ban will automatically set up a background service after being installed. However, it is disabled by default, because some of its default settings may cause undesired effects. You can verify this by using the systemctl command:


```
systemctl status fail2ban.service


```


```
Output○ fail2ban.service - Fail2Ban Service
     Loaded: loaded (/lib/systemd/system/fail2ban.service; disabled; vendor preset: disabled
     Active: inactive (dead)
       Docs: man:fail2ban(1)

```


You could enable Fail2ban right away, but first, you’ll review some of its features.


# Step 2 – Configuring Fail2ban


The fail2ban service keeps its configuration files in the /etc/fail2ban directory. There is a file with defaults called jail.conf. Go to that directory and print the first 20 lines of that file using head -20:


```
cd /etc/fail2ban
head -20 jail.conf


```


```
Output#
# WARNING: heavily refactored in 0.9.0 release.  Please review and
#          customize settings for your setup.
#
# Changes:  in most of the cases you should not modify this
#           file, but provide customizations in jail.local file,
#           or separate .conf files under jail.d/ directory, e.g.:
#
# HOW TO ACTIVATE JAILS:
#
# YOU SHOULD NOT MODIFY THIS FILE.
#
# It will probably be overwritten or improved in a distribution update.
#
# Provide customizations in a jail.local file or a jail.d/customisation.local.
# For example to change the default bantime for all jails and to enable the
# ssh-iptables jail the following (uncommented) would appear in the .local file.
# See man 5 jail.conf for details.
#
# [DEFAULT]

```


As you’ll see, the first several lines of this file are commented out – they begin with # characters indicating that they are to be read as documentation rather than as settings. As you’ll also see, these comments are directing you not to modify this file directly. Instead, you have two options: either create individual profiles for Fail2ban in multiple files within the jail.d/ directory, or create and collect all of your local settings in a jail.local file. The jail.conf file will be periodically updated as Fail2ban itself is updated, and will be used as a source of default settings for which you have not created any overrides.


In this tutorial, you’ll create jail.local. You can do that by copying jail.conf:


```
sudo cp jail.conf jail.local


```


Now you can begin making configuration changes. Open the file in vi or your favorite text editor:


```
sudo vi jail.local


```


While you are scrolling through the file, this tutorial will review some options that you may want to update. The settings located under the [DEFAULT] section near the top of the file will be applied to all of the services supported by Fail2ban. Elsewhere in the file, there are headers for [sshd] and for other services, which contain service-specific settings that will apply over top of the defaults.


/etc/fail2ban/jail.local
```
[DEFAULT]
. . .
bantime = 10m
. . .

```


The bantime parameter sets the length of time that a client will be banned when they have failed to authenticate correctly. This is measured in seconds. By default, this is set to 10 minutes.


/etc/fail2ban/jail.local
```
[DEFAULT]
. . .
findtime = 10m
maxretry = 5
. . .

```


The next two parameters are findtime and maxretry. These work together to establish the conditions under which a client is found to be an illegitimate user that should be banned.


The maxretry variable sets the number of tries a client has to authenticate within a window of time defined by findtime, before being banned. With the default settings, the fail2ban service will ban a client that unsuccessfully attempts to log in 5 times within a 10 minute window.


/etc/fail2ban/jail.local
```
[DEFAULT]
. . .
destemail = root@localhost
sender = root@<fq-hostname>
mta = sendmail
. . .

```


If you need to receive email alerts when Fail2ban takes action, you should evaluate the destemail, sendername, and mta settings. The destemail parameter sets the email address that should receive ban messages. The sendername sets the value of the “From” field in the email. The mta parameter configures what mail service will be used to send mail. By default, this is sendmail, but you may want to use Postfix or another mail solution.


/etc/fail2ban/jail.local
```
[DEFAULT]
. . .
action = %(action_)s
. . .

```


This parameter configures the action that Fail2ban takes when it wants to institute a ban. The value action_ is defined in the file shortly before this parameter. The default action is to update your firewall configuration to reject traffic from the offending host until the ban time elapses.


There are other action_ scripts provided by default which you can replace $(action_) with above:


/etc/fail2ban/jail.local
```
…
# ban & send an e-mail with whois report to the destemail.
action_mw = %(action_)s
            %(mta)s-whois[sender="%(sender)s", dest="%(destemail)s", protocol="%(protocol)s", chain="%(chain)s"]

# ban & send an e-mail with whois report and relevant log lines
# to the destemail.
action_mwl = %(action_)s
             %(mta)s-whois-lines[sender="%(sender)s", dest="%(destemail)s", logpath="%(logpath)s", chain="%(chain)s"]

# See the IMPORTANT note in action.d/xarf-login-attack for when to use this action
#
# ban & send a xarf e-mail to abuse contact of IP address and include relevant log lines
# to the destemail.
action_xarf = %(action_)s
             xarf-login-attack[service=%(__name__)s, sender="%(sender)s", logpath="%(logpath)s", port="%(port)s"]

# ban IP on CloudFlare & send an e-mail with whois report and relevant log lines
# to the destemail.
action_cf_mwl = cloudflare[cfuser="%(cfemail)s", cftoken="%(cfapikey)s"]
                %(mta)s-whois-lines[sender="%(sender)s", dest="%(destemail)s", logpath="%(logpath)s", chain="%(chain)s"]
…

```


For example, action_mw takes action and sends an email, action_mwl takes action, sends an email, and includes logging, and action_cf_mwl does all of the above in addition to sending an update to the Cloudflare API associated with your account to ban the offender there, too.


## Individual Jail Settings


Next is the portion of the configuration file that deals with individual services. These are specified by section headers, like [sshd].


Each of these sections needs to be enabled individually by adding an enabled = true line under the header, with their other settings.


/etc/fail2ban/jail.local
```
[jail_to_enable]
. . .
enabled = true
. . .

```


For this tutorial, you’ll enable the SSH jail. It should be at the top of the individual jail settings. The default parameters will work otherwise, but you’ll need to add a configuration line that says enabled = true under the [sshd] header.


/etc/fail2ban/jail.local
```
#
# JAILS
#

#
# SSH servers
#

[sshd]

# To use more aggressive sshd modes set filter parameter "mode" in jail.local:
# normal (default), ddos, extra or aggressive (combines all).
# See "tests/files/logs/sshd" or "filter.d/sshd.conf" for usage example and details.
#mode   = normal
enabled = true
port    = ssh
logpath = %(sshd_log)s
backend = %(sshd_backend)s

```


Some other settings that are set here are the filter that will be used to decide whether a line in a log indicates a failed authentication and the logpath which tells fail2ban where the logs for that particular service are located.


The filter value is actually a reference to a file located in the /etc/fail2ban/filter.d directory, with its .conf extension removed. These files contain regular expressions (a common shorthand for text parsing) that determine whether a line in the log is a failed authentication attempt. We won’t be covering these files in-depth in this guide, because they are fairly complex and the predefined settings match appropriate lines well.


However, you can see what kind of filters are available by looking into that directory:


```
ls /etc/fail2ban/filter.d


```


If you see a file that looks related to a service you are using, you should open it with a text editor. Most of the files are fairly well commented and you should be able to at least tell what type of condition the script was designed to guard against. Most of these filters have appropriate (disabled) sections in the jail.conf file that we can enable in the jail.local file if desired.


For instance, imagine that you are serving a website using Nginx and realize that a password-protected portion of your site is getting slammed with login attempts. You can tell fail2ban to use the nginx-http-auth.conf file to check for this condition within the /var/log/nginx/error.log file.


This is actually already set up in a section called [nginx-http-auth] in your /etc/fail2ban/jail.conf file. You would just need to add the enabled parameter:


/etc/fail2ban/jail.local
```
. . .
[nginx-http-auth]

enabled = true
. . .

```


When you are finished editing, save and close the file. If you are using vi, use :x to save and quit. At this point, you can enable your Fail2ban service so that it will run automatically from now on. First, run systemctl enable:


```
sudo systemctl enable fail2ban


```


Then, start it manually for the first time with systemctl start:


```
sudo systemctl start fail2ban


```


You can verify that it’s running with systemctl status:


```
sudo systemctl status fail2ban


```


```
Output● fail2ban.service - Fail2Ban Service
     Loaded: loaded (/lib/systemd/system/fail2ban.service; enabled; vendor preset: disabled
     Active: active (running) since Wed 2022-09-14 20:48:40 UTC; 2s ago
       Docs: man:fail2ban(1)
   Main PID: 39396 (fail2ban-server)
      Tasks: 5 (limit: 1119)
     Memory: 12.9M
        CPU: 278ms
     CGroup: /system.slice/fail2ban.service
             └─39396 /usr/bin/python3.6 -s /usr/bin/fail2ban-server -xf start

Sep 14 20:48:40 rocky9-tester systemd[1]: Starting Fail2Ban Service...
Sep 14 20:48:40 rocky9-tester systemd[1]: Started Fail2Ban Service.
Sep 14 20:48:41 rocky9-tester fail2ban-server[39396]: Server ready

```


In the next step, you’ll demonstrate Fail2ban in action.


# Step 3 — Testing the Banning Policies (Optional)


From another server, one that won’t need to log into your Fail2ban server in the future, you can test the rules by getting that second server banned. After logging into your second server, try to SSH into the Fail2ban server. You can try to connect using a nonexistent name:


```
ssh blah@your_server


```


Enter random characters into the password prompt. Repeat this a few times. At some point, the error you’re receiving should change from Permission denied to Connection refused. This signals that your second server has been banned from the Fail2ban server.


On your Fail2ban server, you can see the new rule by checking the output of fail2ban-client. fail2ban-client is an additional command provided by Fail2ban for checking its running configuration.


```
sudo fail2ban-client status


```


```
OutputStatus
|- Number of jail:      1
`- Jail list:   sshd

```


If you run fail2ban-client status sshd, you can see the list of IPs that have been banned from SSH:


```
sudo fail2ban-client status sshd


```


```
OutputStatus for the jail: sshd
|- Filter
|  |- Currently failed: 2
|  |- Total failed:     7
|  `- Journal matches:  _SYSTEMD_UNIT=sshd.service + _COMM=sshd
`- Actions
   |- Currently banned: 1
   |- Total banned:     1
   `- Banned IP list:   134.209.165.184

```


The Banned IP list contents should reflect the IP address of your second server.


# Conclusion


You should now be able to configure some banning policies for your services. Fail2ban is a useful way to protect any kind of service that uses authentication. If you want to learn more about how fail2ban works, you can check out our tutorial on how fail2ban rules and files work.


For information about how to use fail2ban to protect other services, you can read about How To Protect an Nginx Server with Fail2Ban.


