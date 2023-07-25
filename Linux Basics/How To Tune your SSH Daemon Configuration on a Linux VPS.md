# How To Tune your SSH Daemon Configuration on a Linux VPS

```Linux Basics``` ```Security``` ```System Tools```

## Introduction


SSH is the primary way to connect to remote Linux and Unix-like servers through the command line.  It provides a secure connection that you can use to run commands, interact with the system, and even tunnel unrelated traffic through.


Most user are aware of the basics of how to get started and connect to a remote server with a command like:


<pre>
ssh <span class=“highlight”>username</span>@<span class=“highlight”>remote_server</span>
</pre>


However, there are many more options you have with configuring your SSH daemon that can be helpful for increasing security, managing user connections, etc.  We will discuss some of the options that you have available to have more fine-grained control over your SSH access.


We will be demoing these concepts on an Ubuntu 12.04 VPS instance, but any modern Linux distribution should function in a similar fashion.


# Exploring the SSHD Configuration File


The main source of configuration for the SSH daemon itself is in the /etc/ssh/sshd_config file.  Note that this is different from the ssh_config file, which specifies client-side defaults.


Open the file now with administrative privileges:


```
sudo nano /etc/ssh/sshd_config

```


You will see a file with quite a few options and, hopefully (depending on your distribution), a lot of comments.  While most distributions do a fairly good job of establishing sane defaults, there is room for improvement and customization.


Let’s go over some of the options that have already been set in our file on Ubuntu 12.04:


## Ports and Protocols


- 
Port 22: This specifies the port that the SSH daemon will look for connections on.  By default, most clients and servers operate on port 22, but changing this to a different port can potentially reduce the amount of “drive-by” SSH login attempts by malicious users.

- 
Protocol 2: SSH has been through two protocol versions.  Unless you specifically need to support clients that only can operate on protocol 1, it is recommended to leave this as-is.


## Keys and Separation


- 
HostKey /etc/ssh/ssh_host…: These lines specify the host keys for the server.  These keys are used to identify the server to connecting clients.  If the client has already communicated with the server in the past, they can use this key to validate the new connection.

- 
UsePrivilegeSeparation yes: This option allows SSH to spawn child processes that only have the necessary privileges for their tasks.  This is a safety feature to isolate processes in case of a security exploit.

- 
KeyRegenerationInterval and ServerKeyBits: These options affect the server key that is generated for SSH protocol 1.  You should not have to worry about this if you demand that your connections adhere to protocol 2.


## Logging and Restrictions


- 
SyslogFacility and LogLevel: These options specify how activity will be logged.  The first option is for the facility code for logging messages, and the second tells the log level, or amount of detail, to log.

- 
LoginGraceTime 120: This dictates the number of seconds that the server waits before disconnecting from a client if there has been no successful login.

- 
PermitRootLogin yes: This option allows or denies the ability to SSH using the root account.  Since the root account is one that an attacker knows is present on the server machine, and because it provides unrestricted access to the machine, it is often a highly targeted account.  Setting this to “no” is recommended once you configure a regular user account with sudo privileges.

- 
StrictModes yes: This tells SSH to disregard any user-level configuration files that do not have the correct permissions.  If a user sets their configuration files to be world-readable, this has security implications.  It is safer to deny access until this is fixed.

- 
IgnoreRhosts and RhostsRSAAuthentication: These specify whether rhost style authentication will be accepted.  This is protocol 1 syntax that does not apply to protocol 2.

- 
HostbasedAuthentication no: This is the protocol 2 version of the above.  Basically, this allows authentication based on the host of the connecting client.  This is usually only acceptable for isolated environments, because it is possible to spoof source information.  You can specify host information in either a /etc/ssh/shosts.equiv file or a /etc/hosts.equiv file.  This is outside of the scope of this guide.

- 
PermitEmptyPasswords no: This option restricts SSH access for accounts that do not have a password when password authentication is allowed.  This could be a huge security risk and you should almost never change it.

- 
ChallengeResponseAuthentication: This line enables or disables a challenge-response authentication type that can be configured through PAM.  This is outside of the scope of this guide.


## Display


- 
X11Forwarding yes: This allows you to forward X11 graphical user interfaces for applications on the server to the client machine.  This means that you can start a graphical program on a server, and interact with it on a client.  The client must have an X windowing system available.  You can install these on OS X and any desktop Linux will have this capability.

- 
X11DisplayOffset 10: This is an offset for the sshd display number for X11 forwarding.  This offset allows the SSH spawned X11 windows to avoid clashing with the existing X server.

- 
PrintMotd no: This specifies that the SSH daemon itself should not read and display the message of the day file.  This is sometimes read by the shell itself though, so you may need to modify your shell preference files as well.

- 
PrintLastLog yes: This tells the SSH daemon to print out information about the last time you logged in.


## Connection and Environment


- 
TCPKeepAlive yes: This specifies whether TCP keepalive messages are sent to the client machine.  This can help the server recognize when there are problems and the connection will be killed.  If this option is disabled, the connections will not be killed when there is a short amount of network trouble, which can be good, but it also means that users can become disconnected and continue to lock resources up.

- 
AcceptEnv LANG LC*_: This option allows you to accept certain environmental variables from the client machine.  In this specific instance, we are accepting language variables, which can help the shell session display properly for the client.

- 
Subsystem sftp /usr/lib/openssh/sftp-server: This configures external subsystems that can be used with SSH.  This example specifies an SFTP server and the path to execute it.

- 
UsePAM yes: This specifies that PAM (Pluggable Authentication Modules) will be available to assist with authenticating users.


This takes care of the default enabled options on our Ubuntu 12.04 machine.  Next, let’s talk about some other options that may be helpful for you to set or modify.


# Other SSHD Options


There are quite a few other options that we can set for our SSH daemon.  Some of these may be immediately helpful to you, while others may only be useful in specific circumstances.  We will not go over every option here, but will go over some of the useful ones.


## User and Group Filtering


Some options allow you to control exactly which users will have the ability to log in through SSH.  These options should be considered mutually exclusive.  For instance, the AllowUsers option implies that all other users are denied access.


- 
AllowGroups: This option allows you to specify the names of groups on the server.  Only users who are a member of one of these groups will have the ability to log in.  This builds a white-list of groups that should have access.

- 
AllowUsers: This is similar to the option above, but it specifies specific users who are allowed to log in.  Any user not in this list will be unable to login.  This operates as a user white-list.

- 
DenyGroups: This option sets a black-list of groups that should not be allowed to log into the system. Users who belong to these groups will not be allowed access.

- 
DenyUsers: This is a black-list for users.  It specifies specifically which users should not be given access to log in through SSH.


In addition, there are some other restrictive options available.  These can be used in conjunction with any of the above options:


- 
Match: This option allows for much more fine-grained control over who can authenticate under what circumstances.  It specifies a different set of options that should be used when a specific user or group connects.  We will discuss this in more detail later.

- 
RevokedKeys: This allows you to specify a list of revoked public keys.  This will prevent the listed keys from being used to log into the system.


## Miscellaneous Options


There are a number of options that we can use to configure what network traffic the SSH daemon will pay attention to:


- 
AddressFamily: This option specifies what kind of addresses you’ll accept connections from.  By default, the value is “any”, but you can place “inet” for IPv4 addresses or “inet6” for IPv6 addresses.

- 
ListenAddress: This option allows you to tell the SSH daemon to listen on a specific address and port.  By default, the daemon will listen on all addresses that are configured for this machine.


Other types of options that are available are those used to set up certificate-based authentication, connection limiting options like ClientAliveCountMax and ClientAliveInterval, and options like ChrootDirectory, which can be used to lock a logged in user to a specific, pre-configured chroot environment.


# Restricting User Logins


We mentioned above some of the tools you have to restrict access to users and groups.  Let’s go into a bit more detail here.


The most basic syntax for using these is something like:


```
AllowUsers demouser fakeuser madeupuser

```


As you can see, we can specify multiple, space-separated users in each of these directives.


We can also use wild cards and negate entries.  For instance, if we wanted to allow everybody but the user “john” to login, we could try something like this:


```
AllowUsers * !john

```


This specific example would probably be better expressed with a DenyUsers line:


```
DenyUsers john

```


We can also use a ? character to match exactly one character.  For instance, we could use:


```
AllowUsers ?im

```


This would allow logins from accounts like “tim”, “jim”, or “vim”.


We can get more specific however.  In both of the user specifications, we can use the form user@hostname in order to restrict logins to specific client source locations.  For instance, you can have something like:


```
AllowUsers demouser@host1.com fakeuser

```


This will let “fakeuser” log in from anywhere, but will only allow “demouser” to login from a specific host.


We can also limit access on a host-by-host basis outside of the sshd_config file through TCP wrappers.  This is configured through the /etc/hosts.allow and the /etc/hosts.deny files.


For instance, we could restrict access specifically based on SSH traffic by adding lines like this to the hosts.allow file:


```
sshd: .example.com

```


Assuming that we have a companion line in the hosts.deny file that looks like this:


```
sshd: ALL

```


This would restrict logins to only those coming from a example.com or a subdomain.


# Using Match Options to Add Exceptions


We can control our options even further by using “match” options.  Match options work by specifying a criteria pattern that will decide whether the options that follow will be applied.


We begin a match by using the Match option and then specifying criteria key-value pairs.  The available keys are “User”, “Group”, “Host”, and “Address”.  We can separate criteria with spaces and separate the patterns (user1,user2) with commas.  We can also use wild cards and negation:


```
Match User !demouser,!fakeuser Group sshusers Host *.example.com

```


The line above will match only if the user is not demouser or fakeuser, if the user is a member of the group sshusers, and if they are connecting from example.com or a subdomain.


The “Address” criteria can use CIDR netmask notation.


The options that follow a Match specification are conditionally applied.  The scope of these conditional options is until the end of the file or until the next match specification.  Because of this, it is advised to put your default values at the top of the file and put your exceptions at the bottom.


Because of this conditional block, the options under a match are often indented to indicate that they are applied only at the above match.  For instance, the condition above could have a block under it like this:


```
Match User !demouser,!fakeuser Group sshusers Host *.example.com
    AuthorizedKeysFile /sshusers/keys/%u
    PasswordAuthentication yes
    X11Forwarding
    X11DisplayOffset 15

```


You only have access to a subset of options when dealing with match specifications.  To see a complete list, check out the sshd_config man page:


```
man sshd_config

```


Search for the “Match” section to see the list of available options.


# Conclusion


As you can see, you can adjust many values on the server side of SSH that will affect the ability of users to log in and the quality of their experience.  Make sure to carefully test your changes prior to wide-scale implementation in order to catch errors and ensure that your restrictions are not accidentally affecting too few or too many users.


Becoming familiar with your /etc/ssh/sshd_config file is a great first step towards understanding how to carefully control access to your server.  This is an important skill for any Linux system administrator.


<div class=“author”>By Justin Ellingwood</div>


