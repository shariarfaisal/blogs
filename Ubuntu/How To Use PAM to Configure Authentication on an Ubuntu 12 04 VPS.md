# How To Use PAM to Configure Authentication on an Ubuntu 12 04 VPS

```Linux Basics``` ```Security``` ```Ubuntu``` ```System Tools```


Status: Deprecated
This article covers a version of Ubuntu that is no longer supported.  If you are currently operate a server running Ubuntu 12.04, we highly recommend upgrading or migrating to a supported version of Ubuntu:

Upgrade to Ubuntu 14.04.
Upgrade from Ubuntu 14.04 to Ubuntu 16.04
Migrate the server data to a supported version

Reason:
Ubuntu 12.04 reached end of life (EOL) on April 28, 2017 and no longer receives security patches or updates.  This guide is no longer maintained.
See Instead:
This guide might still be useful as a reference, but may not work on other Ubuntu releases.  If available, we strongly recommend using a guide written for the version of Ubuntu you are using.   You can use the search functionality at the top of the page to find a more recent version.

## Introduction



PAM, or Pluggable Authentication Modules, is an abstraction layer that exists on Linux and Unix-like operating systems used to enable authentication between a variety of services.


Built as an intermediary between authentication services and the applications that require user authentication, this system allows these two layers to integrate gracefully and change authentication models without the need to rewrite code.  This is accomplished through the use of modules.


In this guide, we’ll look at the PAM system on an Ubuntu 12.04 VPS, although most modern Linux distributions should function in a similar way.


# Behind the Scenes: The Nuts and Bolts of PAM



## How Pluggable Authentication Works



Many normal applications that we interact with daily in a Linux environment actually use PAM under the hood.


Applications have to be written with PAM library support.  To get a list of the applications on your system that can use PAM in some way, type:


```
ldd /{,usr/}{bin,sbin}/* | grep -B 5 libpam | grep '^/'

```



```
/bin/login:
/bin/su:
/sbin/mkhomedir_helper:
/sbin/pam_tally2:
/usr/bin/chfn:
/usr/bin/chsh:
/usr/bin/passwd:
/usr/sbin/atd:
/usr/sbin/chpasswd:
/usr/sbin/cron:
/usr/sbin/newusers:
/usr/sbin/sshd:

```


You can check a specific application for PAM functionality by typing:


<pre>
ldd $(which <span class=“highlight”>prog_name</span>) | grep libpam
</pre>


If it returns anything, then it can use PAM.


As you can see, many common utilities and tools actually use PAM as an intermediary to perform their tasks.


## PAM Organization



Linux’s version of PAM divides module functionality into different categories depending on which part of the process they are involved in.  Here is a brief explanation of the categories:


- 
Authentication Functions: The authentication modules validate the user’s authentication credentials.  This means it checks if the user can supply valid credentials.

- 
Account Functions: These modules are responsible for deciding if the account that is trying to sign in has access to the resources that it is requesting at this time.  PAM allows you to specify controls that can deny or allow users based on predetermined criteria.

- 
Session Functions: These modules establish the environment that will be built up and torn down after user log in or log off.  Session files can determine which commands need to be run to prepare the environment.

- 
Password Functions: These modules are responsible for updating various services’ authentication details.  If a password needs to be changed for a service, this module can assist in communicating with the service and modifying the correct values.


The first two of these module categories will be referenced every time a program successfully uses PAM for authentication.  The session modules will be run if necessary after the first two.  The password modules are accessed on-demand.


In terms of directory structure, PAM configuration files are stored at /etc/pam.d:


```
ls /etc/pam.d

```



```
atd       chsh            common-password                cron      other   su
chfn      common-account  common-session                 login     passwd  sudo
chpasswd  common-auth     common-session-noninteractive  newusers  sshd

```


This directory generally has a configuration file for each application that will request PAM authentication.  If an application calls PAM but there is no associated configuration file, the “other” configuration file is applied.


Inside the configuration files, there are usually calls to include the configuration files that begin with “common-”.  These are general configuration files whose rules should be applied in most situations.


The modules that are referenced in the configuration files can be located with this command:


```
ls /lib/*/security

```



```
pam_access.so     pam_keyinit.so    pam_permit.so      pam_tally.so
pam_debug.so      pam_lastlog.so    pam_pwhistory.so   pam_time.so
pam_deny.so       pam_limits.so     pam_rhosts.so      pam_timestamp.so
pam_echo.so       pam_listfile.so   pam_rootok.so      pam_umask.so
. . .

```


Look in this directory to see what kind of modules are available to use.  Almost all of the modules have man pages, which can describe their usage.


## How PAM Evaluates Authentication



When an application queries the PAM system for authentication, PAM reads the relevant PAM configuration file.


The configuration files contain a list of PAM modules and how they should be handled.  Each module is called in turn and each call to a module generates a success or failure result.


Based on these values, the configuration file then decides if it should return an “authentication okay” message to the caller, or send it an authentication failure message.


Configurations can fail at the return of the first failure from a called module, or alternative policies can be configured.  For instance, a system may allow LDAP users to authenticate.  If that fails, it may then check against the list of local users.


The lines of each configuration file are evaluated top to bottom, unless a line evaluation causes the rest of the configuration file to be skipped.


## How To Read Policy Lines



Each line in a configuration file contains the following syntax, with each field delimited by white-space.


```
[service] type control module-path [module-arguments]

```


Files located within the /etc/pam.d exclude the service field and, instead, name the configuration file after the application it serves. If there is no pam.d directory, a /etc/pam.conf file is need instead, and the relevant application must be listed at the beginning of each line.


Type is the kind of service that is provided.  It is one of the four categories listed above (authentication, account, password, session).


Control specifies the action taken when it receives the return status of the module call.  It can be any of these values (and can additionally take a more complex syntax, which won’t be covered here):


- 
required: This will lead to an authentication failure if the module call results in a failure.  The remaining specified modules are still called, however.

- 
requisite: This behaves exactly like required, but immediately results in an authentication failure, instead of calling the remaining modules.

- 
sufficient: This control means that if this line succeeds (providing there wasn’t already a required module failure), the authentication will return immediately as successful without running the remaining modules.  A failure of this line simply skips to the next module call.

- 
optional: The success or failure of these lines have no bearing as to the success or failure of the overall authentication unless they are the only module call of their type.

- 
include: This line indicates that the lines of a given type should be read from another configuration script.  This is used often to reference the “common-” files.

- 
substack: This is similar to includes, but failures or successes do not cause the exit of the entire file, just the substack.


Module-path is the name of the pam module to call.


Module-arguments are the optional parameters passed to the module.  Sometimes these are necessary for the module to know what action to take if successful.


Individual files may also reference other files that must be checked using the following syntax:


<pre>
@include <span class=“highlight”>config_file</span>
</pre>


This will read in the entire configuration file, which is different from the “include” control type, which only reads in lines of the same type.


# Examining Example Configurations



We can get a sense how clients are configured by checking out some of the examples in the /etc/pam.d directory.


Open up the file that describes the authentication requirements for “atd”, which is a scheduling daemon.


```
less /etc/pam.d/atd

```



```
auth	required		pam_env.so
@include common-auth
@include common-account
@include common-session-noninteractive
session		required	pam_limits.so

```


The first line calls the “pam_env” module.  If you look at the man pages, you will see that this module is used to set some environmental variables, which are stored in /etc/security/pam_env.conf by default.  This is “required” meaning the whole configuration will fail if this module returns “fail”, but that shouldn’t happen.


The next three lines read in the “common-auth”, “common-account”, and “common-session-noninteractive” files to deal with their respective roles.


The final line calls the the “pam_limits” module, which checks the /etc/security/limits.conf file or the /etc/security/limits.d/ directory.  This can be used to enforce limits on the number of users who can use a service at the same time, among other things.


We can gain a more complete understanding of all of the checks taking place by looking at the files that are referenced.


## Checking the common-auth File



I’ve removed the comments from the file to condense the information.


```
less /etc/pam.d/common-auth

```



```
auth	[success=1 default=ignore] 		pam_unix.so nullok_secure
auth	requisite						pam_deny.so
auth	required						pam_permit.so
auth	optional						pam_ecryptfs.so unwrap

```


The first line is not something we’ve discussed yet.  We can tell that it is calling the “pam_unix” module, which provides standard unix authentication configured through the “/etc/nsswitch.conf” file.  Usually this just means checking the /etc/passwd and /etc/shadow files, as expected.


The “nullok_secure” argument being passed to the unix module specifies that accounts with no password are okay, as long as login information checks out with the /etc/securetty file.


The control field, which has “[success=1 default=ignore]” is the strange part of this example.  It replaces the simplified “required”, “sufficient”, etc parameters and allows for more fine-grained control.


In this instance, if the module returns success, it skips the next “1” line.  The default case, which handles every other return value of the module, results in the line being ignored and moving on.



The second line has the control value of “requisite” meaning that if it fails, the entire configuration returns a failure immediately.  It also calls on the “pam_deny” module, which returns a failure for every call.


This means that this will always fail.  The only exception is when this line is skipped, which happens when the first line returns successfully.



The third line is required and calls the “pam_permit” module, which returns success every time.  This simply resets the current “pass/fail” record at this point to ensure that there aren’t some strange values from earlier.



The fourth line is listed as optional, and calls the “pam_ecryptfs” module with the “unwrap” option.  This is used to unwrap a passphrase using the supplied password, which will then be used for mounting a private directory.  This is only relevant when you use this technology, which is why it is optional.


## Checking the common-account File



We will look at the next file referenced in the “atd” configuration.  Once again, I will be removing the comments for brevity.


```
less /etc/pam.d/common-account

```



```
account [success=1 new_authtok_reqd=done default=ignore] 	pam_unix.so
account	requisite		pam_deny.so
account	requisite		pam_permit.so

```


Almost everything in this file is similar to the one in the “common-auth” file.  The first line calls for an account check with the “pam_unix” module.


Modules can perform different functions depending on the “type” of the call.  For account calls, pam_unix checks that the account is not expired and is not controlled by time-based login restrictions.


If it passes, it skips the “pam_deny” call below it and then processes the permit rule at the end.  If, on the other hand, it fails, it moves down to the deny line and exits with a failure.


## Checking the common-session-noninteractive File



The next section provides session checks for non-interactive programs or environments.


```
less /etc/pam.d/common-session-noninteractive

```



```
session [default=1] 		pam_permit.so
session	requisite			pam_deny.so
session	required			pam_permit.so
session	optional			pam_umask.so
session	required			pam_unix.so
session	optional			pam_ecryptfs.so unwrap

```


The first three lines may look strange.  You should be able to understand the program flow by this point, but they probably seem arbitrary and redundant.  The first line is always successful, the next line is skipped ant the third line is always successful.


They are this way because many PAM configurations are auto-generated and would be modified to provide more substantial rules when additional PAM-aware authentication methods are available.  This just creates a framework where new rules can be inserted to affect the program flow later on.


The fourth line is a call to the “pam_umask” module and is marked optional.  This sets the file creation mask for the session.  It will check a number of different file locations to try to find a relevant umask location.


The fifth line is a required call to the pam_unix module again.  Because this is a “session” type of a call, the unix module behaves differently, yet again.  In this case, it implements logging with the system’s utilities.


The last line is a “pam_ecryptfs” call again.  It performs a similar function as its placement in the “auth” file.


# Conclusion



PAM can be very complicated to master initially.  However, a basic understanding of how the system works and how it links the different components together is essential for developing sane authentication procedures.


While you may not understand everything that goes into configuring PAM rules, it is good to be aware of what systems are in place to guard against illegitimate use of the computer.


Understanding PAM is especially important when you are implementing a new authentication scheme, like LDAP.  Authentication schemes must be changed over to use the new system, either completely, or conditionally.


<div class=“author”>By Justin Ellingwood</div>


