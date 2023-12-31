# What s New in Debian 10 Buster

```Linux Basics``` ```Miscellaneous``` ```Conceptual``` ```Debian``` ```Debian 10```

## Introduction


The Debian operating system’s most recent stable release, version 10 (Buster), was published on July 6, 2019, and will be supported until 2022. Long term support may be provided through 2024 as part of the Debian LTS Project.


This guide is a brief overview of the new features and significant changes to Debian since the previous release. It focuses mainly on changes that will affect users running Debian in a typical server environment. It synthesizes information from the official Debian 10 release notes, the Debian 10 release blog post, kernelnewbies.org, and other sources.


# Summary of Changes and Major Package Versions


Generally, Debian stable releases contain very few surprises or major changes. This remains the case with Debian 10. Beyond a few networking and security changes — which we will cover in subsequent sections — most updates are small modifications to the base system and new versions of available software packages.


The following list summarizes a select list of Debian 10 software updates. The versions that shipped in Debian 9 are included in ( ) parentheses:


##$ System


- Linux kernel 4.19 (from 4.9)
- systemd 241 (from 232)

##$ Web Servers


- Apache 2.4.38 (from 2.4.25)
- nginx 1.14 (from 1.10)

##$ Programming Languages


- Go 1.11 (from 1.7)
- Node.js 10.15.2 (from 4.8.2)
- PHP 7.3 (from 7.0)
- Python 3.7.2 (from 3.5.3)
- Ruby 2.5 (from 2.3)
- Rust 1.34 (from 1.24)

##$ Databases


- MariaDB 10.3 (from 10.1)
- PostgreSQL 11 (from 9.6)

The following sections explain some of the more extensive changes to Debian 10.


# Linux Kernel 4.19


The Linux kernel has been updated to version 4.19. This is a long-term support kernel that was released on October 22, 2018 and will be supported until December of 2020. For more information on the different types of Linux kernel releases, take a look at the official Linux kernel release and support schedule.


Some new features and updates that were released between kernels 4.9 and 4.19 include:


- Virtual GPU support, which enables GPU hardware to be shared between multiple virtual machines instead of being passed-through directly to one. (4.10)
- Performance improvements for large-scale SSD-based swap. (4.11)
- Improved in-kernel TLS acceleration. (4.13)
- Improvements to the Ext4 filesystem, including support for billions of directory entries, and extended attributes that can be up to 64k in size. (4.13)
- Support for 4 petabytes of physical memory, up from 64 terabytes. (4.14)
- Meltdown and Spectre vulnerability updates, along with other CPU vulnerability patches. (4.15)
- Support for using cgroups to set I/O latency targets for block devices. (4.19)

For more information on Linux kernel updates, kernelnewbies.org maintains a detailed and beginner-friendly changelog summary for each release.


# AppArmor Enabled by Default


AppArmor is an access control system that focuses on limiting the resources an application can use. It is supplemental to more traditional user-based access control mechanisms.


AppArmor works by loading application profiles into the kernel, and then using those profiles to enforce limits on capabilities such as file reads and writes, networking access, mounts, and raw socket access.


Debian 10 ships with AppArmor enabled and  some default profiles for common applications such as Apache, Bash, Python, and PHP. More profiles can be installed via the apparmor-profiles-extra package.


See the AppArmor documentation for more information, including guidelines on how to write your own AppArmor application profiles.


# nftables Replaces iptables for Packet Filtering


In Debian Buster the iptables subsystem is replaced by nftables, a newer packet filtering system with improved syntax, streamlined ipv4/ipv6 support, and built-in support for data sets such as dictionaries and maps. You can read a more detailed list of differences on the nftables wiki.


Compatibility with existing iptables scripts is provided by the iptables-nft command. The nftables wiki also has advice on transitioning from iptables to nftables.


# Security-related Updates to Apt


Apt supports https repositories by default in Debian 10. Users no longer need to install additional packages before using https-based package repos.


Additionally, unattended-upgrades — the system Debian uses to perform automatic updates from the security repository — now also supports automating point-release upgrades from any repo. These upgrades are usually small bug fixes and security updates.


# Conclusion


While this guide is not exhaustive, you should now have a general idea of the major changes and new features in Debian 10 Buster. Please refer to the official Debian 10 release notes for more information.


