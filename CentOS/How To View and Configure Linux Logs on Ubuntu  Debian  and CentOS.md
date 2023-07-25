# How To View and Configure Linux Logs on Ubuntu  Debian  and CentOS

```Linux Basics``` ```Ubuntu``` ```Logging``` ```CentOS``` ```Debian```

## Introduction


Linux system administrators often need to look at log files for troubleshooting purposes. This is one of the first things a sysadmin would do.


Linux and the applications that run on it can generate all different types of messages, which are recorded in various log files. Linux uses a set of configuration files, directories, programs, commands and daemons to create, store and recycle these log messages.  Knowing where the system keeps its log files and how to make use of related commands can therefore help save valuable time during troubleshooting.


In this tutorial, we will have a look at different parts of the Linux logging mechanism.



Disclaimer
The commands in this tutorial were tested in plain vanilla installations of CentOS 9, Ubuntu 22.10, and Debian 11.

# Step 1 - Checking the Default Log File Location


The default location for log files in Linux is /var/log. You can view the list of log files in this directory with the following command:


```
ls -l /var/log


```


You’ll see something similar to this on your CentOS system:


```
Output[root@centos-9-trim ~]# ls -l /var/log
total 49316
drwxr-xr-x. 2 root   root          6 Sep 27 19:17 anaconda
drwx------. 2 root   root         99 Jan  3 08:23 audit
-rw-rw----. 1 root   utmp    1234560 Jan  3 16:16 btmp
-rw-rw----. 1 root   utmp   17305344 Jan  1 00:00 btmp-20230101
drwxr-x---. 2 chrony chrony        6 Aug 10  2021 chrony
-rw-r--r--. 1 root   root     130466 Dec  8 22:12 cloud-init.log
-rw-r-----. 1 root   adm       10306 Dec  8 22:12 cloud-init-output.log
-rw-------. 1 root   root      36979 Jan  3 16:03 cron
-rw-------. 1 root   root      27360 Dec 10 23:15 cron-20221211
-rw-------. 1 root   root      94140 Dec 17 23:07 cron-20221218
-rw-------. 1 root   root      95126 Dec 24 23:14 cron-20221225
-rw-------. 1 root   root      95309 Dec 31 23:04 cron-20230101
…

```


# Step 2 - Viewing Log File Contents


Here are some common log files you will find under /var/log:


- wtmp
- utmp
- dmesg
- messages
- maillog or mail.log
- spooler
- auth.log or secure

The wtmp and utmp files keep track of users logging in and out of the system. You cannot directly read the contents of these files using cat commands in the terminal – there are other specific commands for that and you will use some of these commands.


To see who is currently logged in to the Linux server, use the who command. This command gets its values from the /var/run/utmp file (for CentOS and Debian) or /run/utmp (for Ubuntu).


Here is an example from Ubuntu:


```
Outputroot@ubuntu-22:~# who
root     pts/0        2023-01-03 16:23 (198.211.111.194)

```


In this particular case, we are the sole user of the system.


The last command tells you the login history of users:


```
Outputroot@ubuntu-22:~# last
root     pts/0        198.211.111.194  Tue Jan  3 16:23   still logged in
reboot   system boot  5.19.0-23-generi Thu Dec  8 21:48   still running

wtmp begins Thu Dec  8 21:48:51 2022

```


You can also use the last command with a pipe (|) to add a grep search for specific users.


To find out when the system was last rebooted, you can run the following command:


```
last reboot


```


The result may look like this in Debian:


```
Outputroot@debian-11-trim:~# last reboot
reboot   system boot  5.10.0-11-amd64  Thu Dec  8 21:49   still running

wtmp begins Thu Dec  8 21:49:39 2022

```


To see when did someone last logged in to the system, use lastlog:


```
lastlog


```


On a Debian server, you may see output like this:


```
Outputroot@debian-11-trim:~# lastlog
Username         Port     From             Latest
root             pts/0    162.243.188.66   Tue Jan  3 16:23:03 +0000 2023
daemon                                     **Never logged in**
bin                                        **Never logged in**
sys                                        **Never logged in**
sync                                       **Never logged in**
games                                      **Never logged in**
man                                        **Never logged in**
lp                                         **Never logged in**
mail                                       **Never logged in**
news                                       **Never logged in**
uucp                                       **Never logged in**
proxy                                      **Never logged in**
www-data                                   **Never logged in**
backup                                     **Never logged in**
list                                       **Never logged in**
irc                                        **Never logged in**
gnats                                      **Never logged in**
nobody                                     **Never logged in**
_apt                                       **Never logged in**
messagebus                                 **Never logged in**
uuidd                                      **Never logged in**
…

```


For other text-based log files, you can use cat, head or tail commands to read the contents.


In the example below, you’re trying to look at the last ten lines of the /var/log/messages file on a Debian server:


```
sudo tail /var/log/messages


```


You’ll receive an output similar to this:


```
Outputroot@debian-11-trim:~# tail /var/log/messages
Jan  1 00:10:14 debian-11-trim rsyslogd: [origin software="rsyslogd" swVersion="8.2102.0" x-pid="30025" x-info="https://www.rsyslog.com"] rsyslogd was HUPed
Jan  3 16:23:01 debian-11-trim DropletAgent[808]: INFO:2023/01/03 16:23:01 ssh_watcher.go:65: [SSH Watcher] Port knocking detected.
Jan  3 16:23:01 debian-11-trim DropletAgent[808]: INFO:2023/01/03 16:23:01 do_managed_keys_actioner.go:43: [DO-Managed Keys Actioner] Metadata contains 1 ssh keys and 1 dotty keys
Jan  3 16:23:01 debian-11-trim DropletAgent[808]: INFO:2023/01/03 16:23:01 do_managed_keys_actioner.go:49: [DO-Managed Keys Actioner] Attempting to update 1 dotty keys
Jan  3 16:23:01 debian-11-trim DropletAgent[808]: INFO:2023/01/03 16:23:01 do_managed_keys_actioner.go:70: [DO-Managed Keys Actioner] Updating 2 keys
Jan  3 16:23:01 debian-11-trim DropletAgent[808]: INFO:2023/01/03 16:23:01 do_managed_keys_actioner.go:75: [DO-Managed Keys Actioner] Keys updated

```


# Step 3 - Using the rsyslog Daemon


At the heart of the logging mechanism is the rsyslog daemon. This service is responsible for listening to log messages from different parts of a Linux system and routing the message to an appropriate log file in the /var/log directory. It can also forward log messages to another Linux server.


## The rsyslog Configuration File


The rsyslog daemon gets its configuration information from the rsyslog.conf file. The file is located under the /etc directory.


The rsyslog.conf file tells the rsyslog daemon where to save its log messages. This instruction comes from a series of two-part lines within the file.


This file can be found at rsyslog.d/50-default.conf on Ubuntu.


The two part instruction is made up of a selector and an action. The two parts are separated by white space.


The selector part specifies what the source and importance of the log message is and the action part says what to do with the message.


The selector itself is again divided into two parts separated by a dot (.). The first part before the dot is called facility (the origin of the message) and the second part after the dot is called priority (the severity of the message).


Together, the facility/priority and the action pair tell rsyslog what to do when a log message matching the criteria is generated.


You can see an excerpt from a CentOS /etc/rsyslog.conf file with this command:


```
cat /etc/rsyslog.conf


```


You should see something like this as the output:


```
Output# rsyslog configuration file

# For more information see /usr/share/doc/rsyslog-*/rsyslog_conf.html
# or latest version online at http://www.rsyslog.com/doc/rsyslog_conf.html 
# If you experience problems, see http://www.rsyslog.com/doc/troubleshoot.html

#### GLOBAL DIRECTIVES ####

# Where to place auxiliary files
global(workDirectory="/var/lib/rsyslog")

# Use default timestamp format
module(load="builtin:omfile" Template="RSYSLOG_TraditionalFileFormat")

# Include all config files in /etc/rsyslog.d/
include(file="/etc/rsyslog.d/*.conf" mode="optional")

#### MODULES ####

module(load="imuxsock"    # provides support for local system logging (e.g. via logger command)
       SysSock.Use="off") # Turn off message reception via local log socket; 
                          # local messages are retrieved through imjournal now.
module(load="imjournal"             # provides access to the systemd journal
       StateFile="imjournal.state") # File to store the position in the journal
#module(load="imklog") # reads kernel messages (the same are read from journald)
#module(load="immark") # provides --MARK-- message capability

# Provides UDP syslog reception
# for parameters see http://www.rsyslog.com/doc/imudp.html
#module(load="imudp") # needs to be done just once
#input(type="imudp" port="514")
…

```


To understand what this all means, let’s consider the different types of facilities recognized by Linux. Here is a list:


- auth or authpriv: Messages coming from authorization and security related events
- kern: Any message coming from the Linux kernel
- mail: Messages generated by the mail subsystem
- cron: Cron daemon related messages
- daemon: Messages coming from daemons
- news: Messages coming from network news subsystem
- lpr: Printing related log messages
- user: Log messages coming from user programs
- local0 to local7: Reserved for local use

And here is a list of priorities in ascending order:


- debug: Debug information from programs
- info: Simple informational message - no intervention is required
- notice: Condition that may require attention
- warn: Warning
- err: Error
- crit: Critical condition
- alert: Condition that needs immediate intervention
- emerg: Emergency condition

So now let’s consider the following line from the file:


```
Output…
# Log cron stuff
cron.*                                                  /var/log/cron
…

```


This just tells the rsyslog daemon to save all messages coming from the cron daemon in a file called /var/log/cron. The asterisk (*) after the dot means messages of all priorities will be logged. Similarly, if the facility was specified as an asterisk, it would mean all sources.


Facilities and priorities can be related in a number of ways.


In its default form, when there is only one priority specified after the dot, it means all events equal to or greater than that priority will be trapped. So the following directive causes any messages coming from the mail subsystem with a  priority of warning or higher to be logged in a specific file under /var/log:


```
Outputmail.warn			/var/log/mail.warn  

```


This will log every message equal to or greater than the warn priority, but leave everything below it. So messages with err, crit, alert, or emerg will also be recorded in this file.


Using an equal sign (=) after the dot will cause only the specified priority to be logged. So if we wanted to trap only the info messages coming from the mail subsystem, the specification would be something like the following:


```
Outputmail.=info			/var/log/mail.info

```


Again, if we wanted to trap everything from mail subsystem except info messages, the specification would be something like the following


```
Outputmail.!info			/var/log/mail.info  

```


or


```
Outputmail.!=info			/var/log/mail.info  

```


In the first case, the mail.info file will contain everything with a priority lower than info. In the second case, the file will contain all messages with a priority above info.


Multiple facilities in the same line can be separated by commas.


Multiple sources (facility.priority) in the same line are separated by a semicolon.


When an action is marked as an asterisk, it means all users. This is an entry in your CentOS rsyslog.conf file is saying exactly that:


```
Output# Everybody gets emergency messages
*.emerg                                                 :omusrmsg:*

```


Try to see what the rsyslog.conf is saying in your Linux system. Here is an excerpt from a Debian server for another example:


```
Output# /etc/rsyslog.conf configuration file for rsyslog
#
# For more information install rsyslog-doc and see
# /usr/share/doc/rsyslog-doc/html/configuration/index.html


#################
#### MODULES ####
#################

module(load="imuxsock") # provides support for local system logging
module(load="imklog")   # provides kernel logging support
#module(load="immark")  # provides --MARK-- message capability

# provides UDP syslog reception
#module(load="imudp")
#input(type="imudp" port="514")

# provides TCP syslog reception
#module(load="imtcp")
#input(type="imtcp" port="514")


###########################
#### GLOBAL DIRECTIVES ####
###########################

#
# Use traditional timestamp format.
# To enable high precision timestamps, comment out the following line.
#
$ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat

#
# Set the default permissions for all log files.
#
$FileOwner root
$FileGroup adm
$FileCreateMode 0640
$DirCreateMode 0755
$Umask 0022
…

```


As you can see, Debian saves all security/authorization level messages in /var/log/auth.log whereas CentOS saves it under /var/log/secure.


The configurations for rsyslog can come from other custom files as well. These custom configuration files are usually located in different directories under /etc/rsyslog.d. The rsyslog.conf file includes these directories using the $IncludeConfig directive.


Here is what it looks like in Ubuntu:


```
Output…
#
# Include all config files in /etc/rsyslog.d/
#
$IncludeConfig /etc/rsyslog.d/*.conf
…

```


Check out the contents under the /etc/rsyslog.d directory with:


```
ls -l /etc/rsyslog.d


```


You’ll see something similar to this in your terminal:


```
Output-rw-r--r-- 1 root root  314 Sep 19  2021 20-ufw.conf
-rw-r--r-- 1 root root  255 Sep 30 22:07 21-cloudinit.conf
-rw-r--r-- 1 root root 1124 Nov 16  2021 50-default.conf

```


The destination for a log message does not necessarily have to be a log file; the message can be sent to a user’s console. In this case, the action field will contain the username. If more than one user needs to receive the message, their usernames are separated by commas. If the message needs to be broadcast to every user, it’s specified by an asterisk (*) in the action field.


Because of being part of a network operating system, rsyslog daemon can’t only save log messages locally, it can also forward them to another Linux server in the network or act as a repository for other systems. The daemon listens for log messages in UDP port 514. The example below will forward kernel critical messages to a server called “texas”.


```
Outputkern.crit			@texas  

```


# Step 4 - Creating and Testing Your Own Log Messages


So now it’s time for you to create your own log files. To test this, you will do the following:


- Add a log file specification in /etc/rsyslog.conf file
- Restart the rsyslog daemon
- Test the configuration using the logger utility

In the following example, you’ll add two new lines in your CentOS Linux system’s rsyslog.conf file. As you can see with the following command, each of them are coming from a facility called local4 and they have different priorities.


```
vi /etc/rsyslog.conf  


```


Here’s the output:


```
Output…
# New lines added for testing log message generation  
	   
local4.crit                                             /var/log/local4crit.log  
local4.=info                                            /var/log/local4info.log  

```


Next, the service you’ll restart so the config file data is reloaded:


```
/etc/init.d/rsyslog restart


```


To generate the log message now, the logger application is called:


```
logger -p local4.info " This is a info message from local 4"  


```


Looking under the /var/log directory now shows two new files:


```
Output…
-rw-------  1 root root      0 Jan  3 11:21 local4crit.log  
-rw-------  1 root root     72 Jan  3 11:22 local4info.log
…

```


The size of the local4info.log is non-zero. So when you open it, you’ll see the message has been recorded:


```
cat /var/log/local4info.log  


```


```
OutputJan  3 11:22:32 TestLinux root:  This is a info message from local 4  

```


# Step 5 - Rotating Log Files


As more and more information is written to log files, they get bigger and bigger. This obviously poses a potential performance problem. Also, the management of the files becomes cumbersome.


Linux uses the concept of rotating log files instead of purging or deleting them. When a log is rotated, a new log file is created and the old log file is renamed and optionally compressed. A log file can thus have multiple old versions remaining online. These files will go back over a period of time and will represent the backlog. Once a certain number of backlogs have been generated, a new log rotation will cause the oldest log file to be deleted.


The rotation is initiated through the logrotate utility.


## The logrotate Configuration File


Like rsyslog, logrotate also depends on a configuration file and the name of this file is logrotate.conf. It’s located under /etc.


Here is what you see in the logrotate.conf file of your Debian server:


```
cat /etc/logrotate.conf


```


```
Output# see "man logrotate" for details

# global options do not affect preceding include directives

# rotate log files weekly
weekly

# keep 4 weeks worth of backlogs
rotate 4

# create new (empty) log files after rotating old ones
create

# use date as a suffix of the rotated file
#dateext

# uncomment this if you want your log files compressed
#compress

# packages drop log rotation information into this directory
include /etc/logrotate.d

# system-specific logs may also be configured here.

```


By default, log files are to be rotated weekly with four backlogs remaining online at any one time. When the program runs, a new, empty log file will be generated and optionally the old ones will be compressed.


The only exception is for wtmp and btmp files. wtmp keeps track of system logins and btmp keeps track of bad login attempts. Both these log files are to be rotated every month and no error is returned if any previous wtmp or btmp file can be found.


Custom log rotation configurations are kept under the /etc/logrotate.d directory. These are also included in the logrotate.conf with the include directive. The Debian installation shows you the content of this directory:


```
ls -l /etc/logrotate.d


```


```
Outputtotal 32
-rw-r--r-- 1 root root 120 Jan 30  2021 alternatives
-rw-r--r-- 1 root root 173 Jun 10  2021 apt
-rw-r--r-- 1 root root 130 Oct 14  2019 btmp
-rw-r--r-- 1 root root 160 Oct 19  2021 chrony
-rw-r--r-- 1 root root 112 Jan 30  2021 dpkg
-rw-r--r-- 1 root root 374 Feb 17  2021 rsyslog
-rw-r--r-- 1 root root 235 Feb 19  2021 unattended-upgrades
-rw-r--r-- 1 root root 145 Oct 14  2019 wtmp

```


The contents of the rsyslog shows how to recycle a number of log files:


```
cat /etc/logrotate.d/rsyslog


```


```
Output/var/log/syslog
/var/log/mail.info
/var/log/mail.warn
/var/log/mail.err
/var/log/mail.log
/var/log/daemon.log
/var/log/kern.log
/var/log/auth.log
/var/log/user.log
/var/log/lpr.log
/var/log/cron.log
/var/log/debug
/var/log/messages
{
        rotate 4
        weekly
        missingok
        notifempty
        compress
        delaycompress
        sharedscripts
        postrotate
                /usr/lib/rsyslog/rsyslog-rotate
        endscript
}

```


As you can see, the messages file will be reinitialized every day with four days’ worth of logs being kept online. Other log files are rotated every week.


Also worth noting is the postrotate directive. This specifies the action that happens after the whole log rotation has completed.


# Step 6 -  Testing the Rotation


logrotate can be manually run to recycle one or more files. And to do that, you specify the relevant configuration file as an argument to the command.


To see how this works, here is a partial list of log files under /var/log directory in a test CentOS server:


```
ls -l /var/log


```


```
Outputtotal 49324
…
-rw-------. 1 root   root      84103 Jan  3 17:20 messages
-rw-------. 1 root   root     165534 Dec 10 23:12 messages-20221211
-rw-------. 1 root   root     254743 Dec 18 00:00 messages-20221218
-rw-------. 1 root   root     217810 Dec 25 00:00 messages-20221225
-rw-------. 1 root   root     237726 Dec 31 23:45 messages-20230101
drwx------. 2 root   root          6 Mar  2  2022 private
drwxr-xr-x. 2 root   root          6 Feb 24  2022 qemu-ga
lrwxrwxrwx. 1 root   root         39 Mar  2  2022 README -> ../../usr/share/doc/systemd/README.logs
-rw-------. 1 root   root    2514753 Jan  3 17:25 secure
-rw-------. 1 root   root    2281107 Dec 10 23:59 secure-20221211
-rw-------. 1 root   root    9402839 Dec 17 23:59 secure-20221218
-rw-------. 1 root   root    8208657 Dec 25 00:00 secure-20221225
-rw-------. 1 root   root    7081010 Dec 31 23:59 secure-20230101
drwxr-x---. 2 sssd   sssd          6 Jan 17  2022 sssd
-rw-------. 1 root   root          0 Dec  8 22:11 tallylog
-rw-rw-r--. 1 root   utmp       2688 Jan  3 16:22 wtmp

```


The partial contents of the logrotate.conf file looks like this:


```
cat /etc/logrotate.conf


```


```
Output# see "man logrotate" for details

# global options do not affect preceding include directives

# rotate log files weekly
weekly

# keep 4 weeks worth of backlogs
rotate 4

# create new (empty) log files after rotating old ones
create

# use date as a suffix of the rotated file
dateext

# uncomment this if you want your log files compressed
#compress

# packages drop log rotation information into this directory
include /etc/logrotate.d

# system-specific logs may be also be configured here.

```


Next you run the logrotate command:


```
logrotate -fv /etc/logrotate.conf


```


Messages scroll over as new files are generated, errors are encountered etc. When the dust settles, you check for new mail, secure, or messages files:


```
ls -l /var/log/mail*


```


```
Output-rw-------  1 root root    0 Dec 17 18:34 /var/log/maillog 
-rw-------. 1 root root 1830 Dec 16 16:35 /var/log/maillog-20131216  
-rw-------  1 root root  359 Dec 17 18:25 /var/log/maillog-20131217  

```


```
ls -l /var/log/messages*


```


```
Output-rw-------  1 root root    148 Dec 17 18:34 /var/log/messages
-rw-------. 1 root root 180429 Dec 16 16:35 /var/log/messages-20131216 
-rw-------  1 root root  30554 Dec 17 18:25 /var/log/messages-20131217 

```


```
ls -l /var/log/secure*

```


```
-rw-------  1 root root    0 Jan 3 12:34 /var/log/secure
-rw-------. 1 root root 4187 Jan 3 16:41 /var/log/secure-20230103 
-rw-------  1 root root  591 Jan 3 18:28 /var/log/secure-20230103 ```

```


As we can see, all three new log files have been created. The maillog and secure files are still empty, but the new messages file already has some data in it.


# Conclusion


Hopefully this tutorial has given you some ideas about Linux logging. You can try to look into your own development or test systems to have a better idea. Once you are familiar with the location of the log files and their configuration settings, use that knowledge for supporting your production systems. Then you can create some aliases to point to these files to save some typing time as well.


