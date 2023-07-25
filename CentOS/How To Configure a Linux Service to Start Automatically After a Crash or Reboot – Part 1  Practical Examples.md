# How To Configure a Linux Service to Start Automatically After a Crash or Reboot – Part 1  Practical Examples

```Ubuntu``` ```System Tools``` ```Linux Commands``` ```CentOS``` ```Debian```

The author selected the Free and Open Source Fund to receive a donation as part of the Write for DOnations program.


## Introduction


In this two-part tutorial, you will learn how to configure a Linux service to restart automatically after a reboot or crash using systemd.


Part One covers general Linux service management concepts like the init daemon and runlevels. It ends with a demonstration of service management in systemd. Here you will examine targets, wants, requires, and unit files.


Part Two offers a step-by-step tutorial for completing a practical and common systemd task. Specifically, you will configure a MySQL database server to automatically start after a crash or reboot.



Note: You might also consider reading our very popular tutorial on using systemctl to control systemd services and units.

# Prerequisites


To complete this tutorial, you will need:


- A server running CentOS 8, including a non-root user with sudo privileges. To set all this up, including a firewall, you can create a DigitalOcean Droplet running CentOS 8 and then follow our Initial Server Setup Guide.

# Introducing the Service Management Daemon


Linux services can be made self-healing largely by changing the way they are handled by the service management daemon, also known as the init daemon.


init is the first process that starts in a Linux system after the machine boots and the kernel loads into memory. Among other things, it decides how a user process or a system service should load, in what order, and whether it should start automatically.


As Linux has evolved, so has the behavior of the init daemon. Originally, Linux started out with System V init, the same that was used in Unix. Since then, Linux has implemented the Upstart init daemon (created by Ubuntu), and now the systemd init daemon (first implemented by Fedora).


Most modern Linux distributions have gradually migrated away from System V and currently using systemd. Older style init (if used) is kept only for backward compatibility. FreeBSD, a variant of UNIX, uses a different System V implementation, known as BSD init.


We will cover systemd in this article as this is the most recent and common service manager used in Linux distributions today. However, we will also talk about System V and Upstart when necessary and see how systemd evolved from there. To give you an idea:


- System V is the oldest init system, used in
- Debian 6 and earlier
- Ubuntu 9.04 and earlier
- CentOS 5 and earlier
- Upstart came after System V and was used in
- Ubuntu 9.10 to Ubuntu 14.10, including Ubuntu 14.04
- CentOS 6
- systemd is the newest Linux service manager, used in
- Debian 7 and above
- Ubuntu 15.04 and above
- CentOS 7 and above

To understand the init daemon, let’s start with something called runlevel.


## Runlevels


A runlevel represents the current state of a Linux system. For example, a runlevel can be the shutdown state of a Linux server, a single-user mode, the restart mode, etc. Each mode will dictate what services can be running in that state.


Some services can run in one or more runlevel but not in others. Runlevels are denoted by a value between 0 and 6. The following list shows what each of these levels means:


- Runlevel 0: System shutdown
- Runlevel 1: Single-user, rescue mode
- Runlevels 2, 3, 4: Multi-user, text mode with networking enabled
- Runlevel 5: Multi-user, network-enabled, graphical mode
- Runlevel 6: System reboot

Runlevels 2, 3, and 4 vary by distribution. For example, some Linux distributions don’t implement runlevel 4, while others do. Some distributions have a clear distinction between these three levels. In general, runlevel 2, 3, or 4 means a state where Linux has booted in multi-user, network-enabled, text mode.


When you enable a service to auto-start, Linux is actually adding it to a runlevel. In System V, for example, the OS will start with a particular runlevel; and, when it starts, it will try to start all the services that are associated with that runlevel. In systemd, runlevel has become target, and when a service is made to auto-start, it’s added to a target. We’ll discuss targets later in this article.


# Introducing the System V init Daemon


System V uses an inittab file, which later init methods have kept for backward compatibility. Let’s run through System V’s startup sequence:


1. The init daemon is created from the binary file /sbin/init
2. The first file the init daemon reads is /etc/inittab
3. One of the entries in this file decides the runlevel the machine should boot into. For example, if the value for the runlevel is specified as 3, Linux will boot in multi-user, text mode with networking enabled. (This runlevel is known as the default runlevel)
4. Next, the init daemon looks further into the /etc/inittab file and reads what init scripts it needs to run for that runlevel

So, when the init daemon finds what init scripts its needs to run for the given runlevel, it’s actually finding out what services it needs to start up. These init scripts are where you can configure startup behavior for individual services.


An init script is what controls a specific service in System V. init scripts for services were either provided by the application’s vendor or came with the Linux distribution (for native services). You can also create your own init script for a custom created service.


When a process or service such as MySQL Server starts under System V, its binary program file has to load into memory. Depending on how the service is configured, this program may continuously execute the background (and accept client connections). The job of starting, stopping, or reloading this binary application is handled by the service’s init script. It’s called the init script because it initializes the service.


In System V, an init script is a shell script. They are also called rc (run command) scripts. The scripts are located under the /etc/init.d directory. These scripts are symlinked to the /etc/rc directories. Within the /etc directory, there are a number of rc directories, each with a number in its name. The numbers represent different runlevels. So we have /etc/rc0.d, /etc/rc1.d, /etc/rc2.d and so on.


To make a service restart after a crash or reboot, you can usually add a line like this to the init script:


```
ms:2345:respawn:/bin/sh /usr/bin/service_name


```


To enable a System V service to start at system boot time, run this command:


```
sudo chkconfig service_name on


```


To disable it, run this command:


```
sudo chkconfig service_name off


```


To check the status (running or stopped), run this command


```
sudo service service_name status


```


# Introducing the Upstart Daemon


As the serialized way of loading jobs and services became more time consuming and complex with System V init, the Upstart daemon was introduced for faster loading of the OS, graceful clean-up of crashed services, and predictable dependency between system services.


Upstart init was better than System V init in a few ways:


- It didn’t deal with arcane shell scripts to load and manage services. Instead, it uses simple configuration files that are easy to understand and modify
- Services were not loaded serially like System V, which cuts down on system boot time
- It used a flexible event system to customize how services are handled in various states
- Upstart had better ways of handling how a crashed service should respawn.
- There was no need to keep a number of redundant symbolic links, all pointing to the same script

To keep things simple, Upstart is backward-compatible with System V. The /etc/init.d/rc script still runs to manage native System V services. Its main difference is the way it allows multiple events to be associated with a service. This event-based architecture allowed Upstart to be a flexible service manager. With Upstart, each event can fire off a shell script that takes care of that event. These events included:


- Starting
- Started
- Stopping
- Stopped

In between these events, a service can be in a number of states, like waiting, pre-start, starting, running, pre-stop, stopping, etc. Upstart could take action for each of these states as well, creating a very flexible architecture.


At startup, Upstart will run any System V init scripts normally. It will then look under the /etc/init directory and execute the shell commands in each service configuration file. Among other things, these files controlled the service’s startup behavior.


The files have a naming style of service_name.conf, and they have plain text content with different sections, called stanzas. Each stanza describes a different aspect of the service and how it should behave. To make a service start automatically after a crash or reboot, you can add the respawn command in its service configuration files, as shown below for the cron service.


/etc/init/cron.conf
```

...

description "regular background program processing daemon"

start on runlevel [2345]

stop on runlevel [!2345]

expect fork

**respawn**

exec cron

```


# Introducing the systemd Daemon


The latest in Linux init daemons is systemd. In fact, it’s more than an init daemon: systemd is a framework that encompasses many components of a modern Linux system.


One of its functions is to work as a system and service manager for Linux. In this capacity, systemd controls how a service should behave if it crashes or the machine reboots. You can read about systemd’s systemctl here.


systemd is backward-compatible with System V commands and initialization scripts. That means any System V service will also run under systemd. This is possible because most Upstart and System V administrative commands have been modified to work under systemd.


## systemd Configuration Files: Unit Files


At the heart of systemd are unit files. Each unit file represents a specific system resource. Information about the resource is kept track of in the unit file. Service unit files are simple text files (like Upstart .conf files) with a declarative syntax. This makes the files easy to understand and modify.


The main difference between systemd and the other two init methods is that systemd is responsible for the initialization of service daemons and other types of resources like device operating system paths, mount points, sockets, etc. The naming style for a unit file is service_name.unit_type. So, you will see files like dbus.service, sshd.socket, or home.mount.


## Directory Structure


In Red Hat-based systems like CentOS, unit files are located in two places. The main location is /lib/systemd/system/. Custom-created unit files or existing unit files modified by system administrators will live under /etc/systemd/system.


If a unit file with the same name exists in both locations, systemd will use the one under /etc. Suppose a service is enabled to start at boot time or any other target/runlevel. In that case, a symbolic link will be created for that service unit file under appropriate directories in /etc/systemd/system. Unit files under /etc/systemd/system are actually symbolic links to the files with the same name under /lib/systemd/system.


## systemd init Sequence: Target Units


A special type of unit file is a target unit.


A target unit filename is suffixed by .target. Target units are different from other unit files because they don’t represent one particular resource. Rather, they represent the state of the system at any one time. Target units do this by grouping and launching multiple unit files that should be part of that state. systemd targets can therefore be loosely compared to System V runlevels, although they are not the same.


Each target has a name instead of a number. For example, we have multi-user.target instead of runlevel 3 or reboot.target instead of runlevel 6. When a Linux server boots with say, multi-user.target, it’s essentially bringing the server to runlevel 2, 3, or 4, which is the multi-user text mode with networking enabled.


How it brings the server up to that stage is where the difference lies. Unlike System V, systemd does not bring up services sequentially. Along the way, it can check for the existence of other services or resources and decide the order of their loading. This makes it possible for services to load in parallel.


Another difference between target units and runlevels is that in System V, a Linux system could exist in only one runlevel. You could change the runlevel, but the system would exist in that new runlevel only. With systemd, target units can be inclusive, which means when a target unit activates, it can ensure other target units are loaded as part of it.


For example, a Linux system that boots with a graphical user interface will have the graphical.target activated, which in turn will automatically ensure multi-user.target is loaded and activated as well. In System V terms, that would be like having runlevels 3 and 5 activated simultaneously.


The table below compares runlevels and targets:





Runlevel (System V init)
Target Units (Systemd)




runlevel 0
poweroff.target


runlevel 1
resuce.target


runlevel 2, 3, 4
multi-user.target


runlevel 5
graphical.target


runlevel 6
reboot.target




## systemd default.target


systemd default.target is equivalent to the System V default runlevel.


System V had the default runlevel defined in a file called inittab. In systemd, that file is replaced by default.target. The default target unit file lives under /etc/systemd/system directory. It’s a symbolic link to one of the target unit files under /lib/systemd/system.


When you change the default target, you are essentially recreating that symbolic link and changing the system’s runlevel.


The inittab file in System V also specified which directory Linux will execute its init scripts from: it could be any of the rcn.d directories. In systemd, the default target unit determines which resource units will be loaded at boot time.


As units are activated, they are activated all in parallel or all in sequence. How a resource unit loads may depend on other resource units it wants or requires.


## systemd Dependencies: Wants and Requires


systemd wants and requires control how systemd addresses dependency among service daemons.


As mentioned before, Upstart ensures parallel loading of services using configuration files. In System V, a service could start in a particular runlevel, but it also could be made to wait until another service or resource became available. In a similar fashion, systemd services can be made to load in one or more targets, or wait until another service or resource became active.


In systemd, a unit that requires another unit will not start until the required unit is loaded and activated. If the required unit fails for some reason while the first unit is active, the first unit will also stop.


This ensures system stability. A service that requires a particular directory to be present can thus be made to wait until the mount point to that directory is active. On the other hand, a unit that wants another unit will not impose such restrictions. It won’t stop if the wanted unit stops when the caller is active. An example of this would be the non-essential services that come up in graphical-target mode.


## Practical Example: Understanding the systemd Startup Sequence


To understand service startup behavior under systemd, we are using a CentOS 8.3 Droplet. We will follow the .target rabbit-trail as far as we can. systemd’s startup sequence follows a long chain of dependencies.


First, let’s run this command to list the default target unit file:



```
sudo ls -l /etc/systemd/system/default.target


```


This shows output like the following:


```
Outputlrwxrwxrwx. 1 root root 37 Dec  4 17:42 /etc/systemd/system/default.target -> /lib/systemd/system/multi-user.target

```


As you can see, the default target is actually a symbolic link to the multi-user target file under /lib/systemd/system/. So, the system is supposed to boot under multi-user.target, which is similar to runlevel 3 in System V init.


## multi-user.target.wants


Next, let’s run the following command to check all the services the multi-user.target file wants:


```
sudo ls -l /etc/systemd/system/multi-user.target.wants/*.service


```


This should show a list of symbolic link files, pointing back to actual unit files under /usr/lib/systemd/system/:


```
Outputlrwxrwxrwx. 1 root root 38 Dec  4 17:38 /etc/systemd/system/multi-user.target.wants/auditd.service -> /usr/lib/systemd/system/auditd.service
lrwxrwxrwx. 1 root root 39 Dec  4 17:39 /etc/systemd/system/multi-user.target.wants/chronyd.service -> /usr/lib/systemd/system/chronyd.service
lrwxrwxrwx. 1 root root 37 Dec  4 17:38 /etc/systemd/system/multi-user.target.wants/crond.service -> /usr/lib/systemd/system/crond.service
lrwxrwxrwx. 1 root root 42 Dec  4 17:39 /etc/systemd/system/multi-user.target.wants/irqbalance.service -> /usr/lib/systemd/system/irqbalance.service
lrwxrwxrwx. 1 root root 37 Dec  4 17:41 /etc/systemd/system/multi-user.target.wants/kdump.service -> /usr/lib/systemd/system/kdump.service
...

```


Other than multi-user.target, there are different types of targets like system-update.target or basic.target. To see what targets the multi-user target depends on, run the following command:


```
sudo systemctl show --property "Requires" multi-user.target | fmt -10


```


The output shows:


```
OutputRequires=basic.target

```


## basic.target


You can run the following command to see if there are any required units for basic.target:


```
sudo systemctl show --property "Requires" basic.target | fmt -10


```


As it turns out, the basic.target requires sysinit.target:


```
OutputRequires=sysinit.target
-.mount

```


And it wants a few targets as well:


```
sudo systemctl show --property "Wants" basic.target | fmt -10


```


The command will return the following:


```
OutputWants=slices.target
paths.target
timers.target
microcode.service
sockets.target
sysinit.target

```


Going recursively, you can see if the sysinit.target will require any other targets to be running as well:


```
sudo systemctl show --property "Requires" sysinit.target | fmt -10


```


There will be none. However, there will be other targets wanted by sysinit.target:


```
systemctl show --property "Wants" sysinit.target | fmt -10


```


An output like this will appear:


```
OutputWants=systemd-random-seed.service
dev-mqueue.mount
rngd.service
systemd-modules-load.service
proc-sys-fs-binfmt_misc.automount
local-fs.target
sys-fs-fuse-connections.mount
systemd-sysusers.service
systemd-update-done.service
systemd-update-utmp.service
systemd-journal-flush.service
dev-hugepages.mount
dracut-shutdown.service
swap.target
systemd-udevd.service
import-state.service
sys-kernel-debug.mount
nis-domainname.service
systemd-journald.service
selinux-autorelabel-mark.service
kmod-static-nodes.service
loadmodules.service
ldconfig.service
cryptsetup.target
systemd-sysctl.service
systemd-ask-password-console.path
systemd-journal-catalog-update.service
systemd-udev-trigger.service
systemd-tmpfiles-setup.service
systemd-hwdb-update.service
sys-kernel-config.mount
systemd-binfmt.service
systemd-tmpfiles-setup-dev.service
systemd-machine-id-commit.service
systemd-firstboot.service

```


# Examining a systemd Unit File


Going a step further now, let’s look inside a service unit file, the one for sshd:


```
sudo vi /etc/systemd/system/multi-user.target.wants/sshd.service


```


It looks like this:


/etc/systemd/system/multi-user.target.wants/sshd.service
```
[Unit]
Description=OpenSSH server daemon
Documentation=man:sshd(8) man:sshd_config(5)
After=network.target sshd-keygen.target
Wants=sshd-keygen.target

[Service]
Type=notify
EnvironmentFile=-/etc/crypto-policies/back-ends/opensshserver.config
EnvironmentFile=-/etc/sysconfig/sshd
ExecStart=/usr/sbin/sshd -D $OPTIONS $CRYPTO_POLICY
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure
RestartSec=42s

[Install]
WantedBy=multi-user.target

```


You can see the service unit file is clean and easy to understand.


The first important part is the After clause in the [Unit] section. This says the sshd service needs to load after the network.target and the sshd-keygen.target are loaded.


The [Install] section shows the service is wanted by the multi-user.target. This means multi-user.target will load the sshd daemon, but it won’t shut down or crash if sshd fails during loading.


Since multi-user.target is the default target, sshd daemon is supposed to start at boot time. In the [Service] section, the Restart parameter has the value on-failure. This setting allows the sshd daemon to restart if it crashes or has an unclean exit.


# Conclusion


In this article you learned about System V, Upstart, and systemd service management daemons. You explored startup scripts and configuration files, important parameters, startup sequence, and the commands that control service startup behavior.


In part two of this article, we will apply these skills to a real-life example and use systemd to configure MySQL. Once completed, your MySQL instance will automatically restart after a reboot or crash. And while you will use MySQL as your example application, you can substitute any number of services, such as the Nginx or Apache web servers.


