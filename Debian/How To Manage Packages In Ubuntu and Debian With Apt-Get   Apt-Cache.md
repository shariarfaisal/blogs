# How To Manage Packages In Ubuntu and Debian With Apt-Get   Apt-Cache

```Linux Basics``` ```Ubuntu``` ```Linux Commands``` ```Getting Started``` ```Debian```

## Introduction


Apt is a command line frontend for the dpkg packaging system and is the preferred way of managing software from the command line for many distributions. It is the main package management system in Debian and Debian-based Linux distributions like Ubuntu.


While a tool called “dpkg” forms the underlying packaging layer, apt and apt-cache provide user-friendly interfaces and implement dependency handling. This allows users to efficiently manage large amounts of software easily.


In this guide, we will discuss the basic usage of apt and apt-cache and how they can manage your software. We will be practicing on an Ubuntu 22.04 cloud server, but the same steps and techniques should apply on any other Ubuntu or Debian-based distribution.


# How To Update the Package Database with Apt


Apt operates on a database of known, available software. It performs installations, package searches, and many other operations by referencing this database.


Because of this, before beginning any packaging operations with apt, we need to ensure that our local copy of the database is up-to-date.


Update the local database with apt update. Apt requires administrative privileges for most operations:


```
sudo apt update


```


You will see a list of servers we are retrieving information from. After this, your database should be up-to-date.


# How to Upgrade Installed Packages with Apt


You can upgrade the packages on your system by using apt upgrade. You will be prompted to confirm the upgrades, and restart any updated system services:


```
sudo apt upgrade


```


# How to Install New Packages with Apt


If you know the name of a package you need to install, you can install it by using apt install:


```
sudo apt install package1 package2 …


```


You can see that it is possible to install multiple packages at one time, which is useful for acquiring all of the necessary software for a project in one step.


Apt installs not only the requested software, but also any software needed to install or run it.


You can install a program called sl by typing:


```
sudo apt install sl


```


After that, you’ll be able to run sl on the command line.


# How to Delete a Package with Apt


To remove a package from your system, run apt remove:


```
sudo apt remove package_name


```


This command removes the package, but retains any configuration files in case you install the package again later. This way, your settings will remain intact, even though the program is not installed.


If you need to clean out the configuration files as well as the program, use apt purge:


```
sudo apt purge package_name


```


This uninstalls the package and removes any configuration files associated with the package.


To remove any packages that were installed automatically to support another program, that are no longer needed, type the following command:


```
sudo apt autoremove


```


You can also specify a package name after the autoremove command to uninstall a package and its dependencies.


# Common Apt Option Flags


There are a number of additional options that can be specified by the use of flags. We will go over some common ones.


To do a “dry run” of a procedure in order to get an idea of what an action will do, you can pass the -s flag for “simulate”:


```
sudo apt install -s htop


```


```
OutputReading package lists... Done
Building dependency tree... Done
Reading state information... Done
Suggested packages:
  lm-sensors
The following NEW packages will be installed:
  htop
0 upgraded, 1 newly installed, 0 to remove and 1 not upgraded.
Inst htop (3.0.5-7build2 Ubuntu:22.04/jammy [amd64])
Conf htop (3.0.5-7build2 Ubuntu:22.04/jammy [amd64])

```


In place of actual actions, you can see an Inst and Conf section specifying where the package would be installed and configured if the “-s” was removed.


If you do not want to be prompted to confirm your choices, you can also pass the -y flag to automatically assume “yes” to questions.


```
sudo apt remove -y htop


```


If you would like to download a package, but not install it, you can issue the following command:


```
sudo apt install -d packagename


```


The files will be retained in /var/cache/apt/archives.


If you would like to suppress output, you can pass the -qq flag to the command:


```
sudo apt remove -qq packagename


```


# How to Find a Package with Apt-Cache


The apt packaging tool is actually a suite of related, complimentary tools that are used to manage your system software.


While apt is used to upgrade, install, and remove packages, apt-cache is used to query the package database for package information.


You can use apt-cache search to search for a package that suits your needs. Note that apt-cache doesn’t usually require administrative privileges:


```
apt-cache search what_you_are_looking_for


```


For instance, to find htop, an improved version of the top system monitor, you can use:


```
apt-cache search htop


```


```
Outputhtop - interactive processes viewer
aha - ANSI color to HTML converter
bashtop - Resource monitor that shows usage and stats
bpytop - Resource monitor that shows usage and stats
btop - Modern and colorful command line resource monitor that shows usage and stats
libauthen-oath-perl - Perl module for OATH One Time Passwords
pftools - build and search protein and DNA generalized profiles

```


You can search for more generic terms also. In this example, we’ll look for mp3 conversion software:


```
apt-cache search mp3 convert


```


```
Outputabcde - A Better CD Encoder
cue2toc - converts CUE files to cdrdao's TOC format
dir2ogg - audio file converter into ogg-vorbis format
easytag - GTK+ editor for audio file tags
ebook2cw - convert ebooks to Morse MP3s/OGGs
ebook2cwgui - GUI for ebook2cw
ffcvt - ffmpeg convert wrapper tool
. . .

```


# How to View Package Information with Apt-Cache


To view information about a package, including an extended description, use the following syntax:


```
apt-cache show package_name


```


This will also provide the size of the download and the dependencies needed for the package.


To see if a package is installed and to check which repository it belongs to, you can use apt-cache policy:


```
apt-cache policy package_name


```


# Conclusion


You should now know enough about apt-get and apt-cache to manage most of the software on your server.


While it is sometimes necessary to go beyond these tools and the software available in the repositories, most software operations can be managed by these tools.


Next, you can read about Ubuntu and Debian package management in detail.


