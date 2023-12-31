# What s New in Ubuntu 18 04 Bionic Beaver

```Linux Basics``` ```Ubuntu``` ```Miscellaneous``` ```Conceptual``` ```Ubuntu 18.04```

## Introduction


The Ubuntu operating system’s most recent Long Term Support (LTS) release, version 18.04 (Bionic Beaver), was released on April 26, 2018.


This guide is intended as a brief overview of new features and significant changes to Ubuntu Server since the previous LTS release, 16.04 (Xenial Xerus). It synthesizes information from the official Bionic Beaver release notes and other sources.


# What is a Long Term Support Release?


While new Ubuntu Desktop and Server releases occur every six months, LTS versions come every two years and are supported for five years after release. 18.04 will continue to receive security updates and critical bug fixes until April of 2023. This makes LTS releases a stable platform for deploying production systems.


You can view a timeline of the Ubuntu release lifecycle at the Ubuntu release end of life page.


# Summary of Changes and Major Package Versions


Generally, Ubuntu LTS releases contain very few surprises or major changes. This remains the case with Ubuntu 18.04. Beyond a few networking changes — which we will cover in subsequent sections — most updates are small changes to the base system and new versions of available software packages.


As a general summary, a selected list of Ubuntu 18.04 software versions follows. For comparison, the versions that shipped in Ubuntu 16.04 are included in ( ) parentheses:


##$ System


- Linux kernel 4.15 (from 4.4)
- systemd 237 (from 229)

##$ Web Servers


- Apache 2.4.29 (from 2.4.18)
- nginx 1.14.0 (from 1.10.3)

##$ Programming Languages


- Python 3.6.5 (from 3.5.1)
- Ruby 2.5 (from 2.3)
- Go 1.10 (from 1.6)
- PHP 7.2 (from 7.0)
- Node.js 8.10 (from 4.2.6)

##$ Databases


- MySQL 5.7.21 (from 5.7.21)
- MariaDB 10.1 (from 10)
- PostgreSQL 10 (from 9.5)
- MongoDB 3.6.3 (from 2.6.10)

More extensive changes are detailed in the following sections.


# Linux Kernel 4.15


The Linux kernel has been updated to version 4.15. This version includes updates to mitigate the Spectre and Meltdown vulnerabilities (these updates have also been backported to Ubuntu 16.04’s 4.4 kernel). Beyond that, the changes relevant to Ubuntu Server users are mostly filesystem bug fixes, performance improvements, and support for very large amounts of memory.


# LXD 3.0


LXD is a standardized interface to manage Linux containers. Unlike Docker it is oriented towards running entire OSes, more like a typical virtual machine hypervisor.


LXD 3.0 adds clustering support, where multiple identically configured LXD servers can function as one. There is also support for passing NVIDIA GPUs into containers, hotplugging devices, and proxying TCP connections between the host and its containers. For more details, see the LXD 3.0.0 release notes.


# Netplan and systemd-networkd


ifupdown (including the familiar ifup and ifdown utilities) has been replaced by Netplan. Netplan is a simplified interface for configuring Linux networking, where YAML files in /etc/netplan are used to generate configuration information for either NetworkManager or — in the case of new Ubuntu Server installations – systemd-networkd.


The ip link set command is a replacement for ifup and ifdown. You can learn more about it in the How To Configure Network Interfaces and Addresses section of our IPRoute2 Tools tutorial.


For more information on configuring Netplan, see the official documentation. Details on how to use and configure systemd-networkd are available in the systemd-networkd.service and systemd.network man pages.


The command networkctl can output a summary of your network devices:


```
networkctl


```


```
OutputIDX LINK             TYPE               OPERATIONAL SETUP
  1 lo               loopback           carrier     unmanaged
  2 eth0             ether              routable    configured

```


Run the command with the status flag and it will print the state of each IP address on the system:


```
networkctl status


```


```
Output●        State: routable
       Address: 192.0.2.10 on eth0
                203.0.113.241 on eth0
                2001:DB8:68be:caff:fe4c:c963 on eth0
       Gateway: 203.0.113.1 (ICANN, IANA Department) on eth0
           DNS: 203.0.113.2
                203.0.113.3

```


# Default DNS Resolver


The default DNS resolver is now systemd-resolved. The standard /etc/resolve.conf file is now managed by systemd-resolved, and configuration of the resolver should be done in /etc/systemd/resolved.conf.


Configuration information for systemd-resolved can be found in the resolved.conf man page.


# Default NTP Server


chrony replaces ntpd as the recommended NTP server in Ubuntu 18.04. Though the default Ubuntu system is set up to use systemd-timesyncd for simple sync needs, ntpd would often be required for more demanding time synchronization or to provide network time services for other clients. In 18.04 ntpd has been demoted to the universe repo, and is unavailable without updating your APT configuration.


The official chrony website has a comparison of NTP implementations to help you decide which is right for you.


# Conclusion


While this guide is not exhaustive, you should now have a general idea of the major changes and new features in Ubuntu 18.04.


The safest course of action in migrating to a major new release is usually to install the distribution from scratch, configure services with careful testing along the way, and migrate application or user data as a separate step.


If you prefer to upgrade in place, our tutorial  How To Upgrade to Ubuntu 18.04 LTS will provide details on the process.


