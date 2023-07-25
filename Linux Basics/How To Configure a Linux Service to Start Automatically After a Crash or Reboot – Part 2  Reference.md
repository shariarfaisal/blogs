# How To Configure a Linux Service to Start Automatically After a Crash or Reboot – Part 2  Reference

```Linux Basics``` ```Ubuntu``` ```System Tools``` ```Linux Commands``` ```CentOS``` ```Debian```

The author selected the Free and Open Source Fund to receive a donation as part of the Write for DOnations program.


## Introduction


In this tutorial you will use systemd to configure MySQL to restart automatically after a reboot or crash.


This is the second half of a two-part series. Part One covers general Linux service management concepts like the init daemon and runlevels. It ends with a demonstration of service management in systemd. Here you will examine targets, wants, requires, and unit files. This part, part two, provides a practical example using the MySQL database.



Note: You might also consider reading our very popular tutorial on using systemctl to control systemd services and units.

# Prerequisites


To complete this tutorial, you will need:


- 
A server running CentOS 8, including a non-root user with sudo privileges. To set all this up, including a firewall, you can create a DigitalOcean Droplet running CentOS 8 and then follow our Initial Server Setup Guide.

- 
MySQL installed. For detailed instructions, follow our tutorial, How To Install MySQL on CentOS 8.


# Configuring MySQL To Auto-start After Boot using systemd


With MySQL installed, check the status of your service:


```
sudo systemctl status mysqld.service


```


The output should show that the service is running, but the daemon is disabled:


```
Output mysqld.service - MySQL 8.0 database server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; disabled; vendor preset: disabled)
    Active: active (running) since Thu 2020-12-24 23:48:56 UTC; 1h 6min ago
   Process: 30423 ExecStartPost=/usr/libexec/mysql-check-upgrade (code=exited, status=0/SUCCESS)
   Process: 30294 ExecStartPre=/usr/libexec/mysql-prepare-db-dir mysqld.service (code=exited, status=0/SUCCESS)
   Process: 30270 ExecStartPre=/usr/libexec/mysql-check-socket (code=exited, status=0/SUCCESS)
 Main PID: 30378 (mysqld)
  Status: "Server is operational"
   Tasks: 40 (limit: 4763)
...

```


If the service is enabled, disable it. We want to first explore the disabled behavior before making our changes:


```
sudo systemctl disable mysqld.service


```


Next, run this command to check if MySQL is wanted by multi-user.target:


```
sudo systemctl show --property "Wants" multi-user.target | fmt -10 | grep mysql


```


Nothing will return. Now check if the symbolic link exists:


```
sudo ls -l /etc/systemd/system/multi-user.target.wants/mysql*


```


A message appears stating that the symlink file does not exist:


```
Outputls: cannot access '/etc/systemd/system/multi-user.target.wants/mysql*': No such file or directory

```


Now, if you like, reboot the server and check the MySQL service. It should not be running.


Whether you rebooted or not, now re-enable the MySQL service:


```
sudo systemctl enable mysqld.service


```


This time, the system will create a symbolic link under /etc/systemd/system/multi-user.target.wants/:


```
OutputCreated symlink /etc/systemd/system/multi-user.target.wants/mysqld.service → /usr/lib/systemd/system/mysqld.service.

```


Run the ls command again to confirm this:


```
sudo ls -l /etc/systemd/system/multi-user.target.wants/mysql*


```


You will receive an output like this:


```
Outputlrwxrwxrwx 1 root root 38 Aug  1 04:43 /etc/systemd/system/multi-user.target.wants/mysqld.service -> /usr/lib/systemd/system/mysqld.service

```


Enabling or disabling a systemd service creates or removes the symbolic link from the default target’s wants directory.


If you like, reboot the Droplet again, and when it comes back online run the ps -ef command to check the service status.


```
ps -ef | grep mysql


```


This command will provide information about MySQL, if it is running:


```
[secondary_label Output]\
mysql        851       1  2 04:26 ?        00:00:02 /usr/libexec/mysqld --basedir=/usr

```


You have now configured MySQL to restart after a reboot. Next you will account for crashes.


# Configuring MySQL To Auto-start After a Crash Using systemd


Being a modern application, MySQL already comes configured to auto-start after a crash. Let’s see how to disable that.


Open the MySQL service unit file in an editor:


```
sudo vi /etc/systemd/system/multi-user.target.wants/mysqld.service


```


After the header information, the contents of the file looks like this:


/etc/systemd/system/multi-user.target.wants/mysqld.service
```

[Unit]

Description=MySQL 8.0 database server
After=syslog.target
After=network.target

[Service]

Type=notify
User=mysql
Group=mysql

ExecStartPre=/usr/libexec/mysql-check-socke
ExecStartPre=/usr/libexec/mysql-prepare-db-dir %n
`# Note: we set --basedir to prevent probes that might trigger SELinux alarms,`
`# per bug #547485`
ExecStart=/usr/libexec/mysqld --basedir=/usr
ExecStartPost=/usr/libexec/mysql-check-upgrade
ExecStopPost=/usr/libexec/mysql-wait-stop

`# Give a reasonable amount of time for the server to start up/shut down`

TimeoutSec=300

`# Place temp files in a secure directory, not /tmp`

PrivateTmp=true

Restart=on-failure

RestartPreventExitStatus=1

`# Sets open_files_limit`

LimitNOFILE = 10000

`# Set enviroment variable MYSQLD_PARENT_PID. This is required for SQL restart command.`

Environment=MYSQLD_PARENT_PID=1

[Install]

WantedBy=multi-user.target

```


As you can see the value of the Restart parameter is set to on-failure. This means the MySQL service will restart for unclean exit codes or timeouts.


The man page for systemd service shows the following table for Restart parameters:





Restart settings/Exit causes
no
always
on-success
on-failure
on-abnormal
on-abort
on-watchdog




Clean exit code or signal

X
X






Unclean exit code

X

X





Unclean signal

X

X
X
X



Timeout

X

X
X




Watchdog

X

X
X

X




In a systemd service unit file, two parameters - Restart and RestartSec - control crash behavior. The first parameter specifies when the service should restart, and the second parameter defines how long it should wait before restarting.


To test the crash behavior, stop the MySQL process with a kill -9 signal. In our case the main PID is 851; replace the PID with your own:


ps -ef | grep mysql


```
sudo kill -9 851


```


Wait for a few seconds and then check the status:


```
sudo systemctl status mysqld.service


```


The output will show that MySQL has restarted with a new PID (in our case the new Process ID is 1513):


```
Output  mysqld.service - MySQL 8.0 database server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2020-12-25 04:47:48 UTC; 55s ago
  Process: 1420 ExecStopPost=/usr/libexec/mysql-wait-stop (code=exited, status=0/SUCCESS)
  Process: 1559 ExecStartPost=/usr/libexec/mysql-check-upgrade (code=exited, status=0/SUCCESS)
  Process: 1476 ExecStartPre=/usr/libexec/mysql-prepare-db-dir mysqld.service (code=exited, status=0/SUCCESS)Process: 1451 ExecStartPre=/usr/libexec/mysql-check-socket (code=exited, status=0/SUCCESS)
 Main PID: 1513 (mysqld)
   Status: "Server is operational"
...

```


Next, reopen the unit file:


```
sudo vi /etc/systemd/system/multi-user.target.wants/mysqld.service


```


Comment out the Restart directive in the MySQL daemon’s unit file and save it. This will disable the restart behavior:


/etc/systemd/system/multi-user.target.wants/mysqld.service
```

`# Restart=on-failure`

```


After this, reload the systemd daemon, followed by a restart of the mysqld service:


```
sudo systemctl daemon-reload
sudo systemctl restart mysqld.service


```


You can find the main PID of the service by running this command:


```
sudo systemctl status mysqld.service


```


```
Output. . .
Main PID: 1895 (mysqld)

```


Using the kill -9 command, kill the main PID of the MySQL PID in your environment (we are using the PID in our test environment).


```
sudo kill -9 1895

```


Check the status for MySQL:


```
sudo systemctl status mysqld.service


```


It will show that the service has failed:


```
Outputmysqld.service - MySQL 8.0 database server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; vendor preset: disabled)
   Active: **failed** (Result: signal) since Fri 2020-12-25 05:07:22 UTC; 1min 14s ago
  Process: 1976 ExecStopPost=/usr/libexec/mysql-wait-stop (code=exited, status=0/SUCCESS)
  Process: 1940 ExecStartPost=/usr/libexec/mysql-check-upgrade (code=exited, status=0/SUCCESS)
  Process: 1895 ExecStart=/usr/libexec/mysqld --basedir=/usr (code=killed, signal=KILL)
  Process: 1858 ExecStartPre=/usr/libexec/mysql-prepare-db-dir mysqld.service (code=exited, status=0/SUCCESS
  Process: 1833 ExecStartPre=/usr/libexec/mysql-check-socket (code=exited, status=0/SUCCESS)
  Main PID: 1895 (code=**killed**, signal=KILL)
   ...

```


Try to find the service status a few times. Each time the service will show as failed.


So, we have emulated a crash where the service has stopped and hasn’t come back. This is because we have instructed systemd not to restart the service after an unclean stop. If you edit the mysqld.service unit file to uncomment the Restart parameter, save it, reload the systemctl daemon, and finally restart the service, that will restore normal function.


This is how you can configure a native systemd service to auto-start after a crash. All you have to do is add an extra directive for Restart (and optionally RestartSec) under the [Service] section of the service unit file.


# Conclusion


In this two-part series, you learned about the service management daemons used across the Linux ecosystem. You then explored the fundamentals of systemd and applied those fundamentals to a practical example: configuring a database to restart after a reboot or crash. If you wish to learn more about systemd, consider exploring our comprehensive tutorial on using systemctl.


