# How To Monitor System Authentication Logs on Ubuntu

```Security``` ```Ubuntu``` ```System Tools``` ```Monitoring```

# How To Monitor System Logins


A fundamental component of authentication management is monitoring the system after you have configured your users.


We will be exploring these concepts on a Ubuntu 22.04 server, but you can follow along on any modern Linux distribution. You can set up a Ubuntu 22.04 server for this tutorial by following our guide to Initial Server Setup on Ubuntu 22.04.


# Review Authentication Attempts


Modern Linux systems log all authentication attempts in a discrete file. This is located at /var/log/auth.log. You can view this file using less:


```
sudo less /var/log/auth.log


```


```
OutputMay  3 18:20:45 localhost sshd[585]: Server listening on 0.0.0.0 port 22.
May  3 18:20:45 localhost sshd[585]: Server listening on :: port 22.
May  3 18:23:56 localhost login[673]: pam_unix(login:session): session opened fo
r user root by LOGIN(uid=0)
May  3 18:23:56 localhost login[714]: ROOT LOGIN  on '/dev/tty1'
Sep  5 13:49:07 localhost sshd[358]: Received signal 15; terminating.
Sep  5 13:49:07 localhost sshd[565]: Server listening on 0.0.0.0 port 22.
Sep  5 13:49:07 localhost sshd[565]: Server listening on :: port 22.
. . .

```


When you are finished viewing the file, you can use q to quit less.


# How To Use the “last” Command


Usually, you will only be interested in the most recent login attempts. You can see these with the last tool:


```
last


```


```
Outputdemoer   pts/1        rrcs-72-43-115-1 Thu Sep  5 19:37   still logged in   
root     pts/1        rrcs-72-43-115-1 Thu Sep  5 19:37 - 19:37  (00:00)    
root     pts/0        rrcs-72-43-115-1 Thu Sep  5 19:15   still logged in   
root     pts/0        rrcs-72-43-115-1 Thu Sep  5 18:35 - 18:44  (00:08)    
root     pts/0        rrcs-72-43-115-1 Thu Sep  5 18:20 - 18:20  (00:00)    
demoer   pts/0        rrcs-72-43-115-1 Thu Sep  5 18:19 - 18:19  (00:00)

```


This provides a formatted version of information saved in another file, /etc/log/wtmp.


Judging from the first and the third line, users are currently logged into the system. The total time spent logged into the system during other, already closed sessions is provided by a set of hyphen-separated values.


# How To Use the “lastlog” Command


You can also view the last time each user on the system logged in using the lastlog command.


This information is provided by accessing the /etc/log/lastlog file. It is then sorted according to the entries in the /etc/passwd file:


```
lastlog


```


```
OutputUsername         Port     From             Latest
root             pts/1    rrcs-72-43-115-1 Thu Sep  5 19:37:02 +0000 2013
daemon                                     **Never logged in**
bin                                        **Never logged in**
sys                                        **Never logged in**
sync                                       **Never logged in**
games                                      **Never logged in**
. . .

```


You can see the latest login time of every user on the system.


Notice how the system users will almost all have **Never logged in**. Many of these system accounts will not be used to log in directly, so this is normal.


# Conclusion


User authentication on Linux is a relatively flexible area of system management. There are many ways of accomplishing the same objective with widely available tools.


It is important to understand where the system keeps information about logins so that you can monitor your server for changes that do not reflect your usage.


Next, you may want to learn how to add and delete system users.


