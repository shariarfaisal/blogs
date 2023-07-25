# How to Install Anbox on Linux Mint 

```UNIX/Linux```

This article goes over the steps to install Anbox on Linux Mint. Ever wondered how cool it would be to be able to run android applications on your Linux system? Well, Anbox helps you do exactly that.


Anbox is short for Android in a box and it is exactly what it sounds like! Anbox is a free and open-source environment that enables you to run Android applications on your Linux distribution.


It follows a container-based approach to run the android operating system on Linux.


# Steps to Install Anbox on Linux Mint


Here’s a quick summary of the step to install Anbox on your Linux Mint:


1. First, Install snapd
2. Install the required kernel modules
3. Install the Anbox package on Linux Mint
4. Steps to uninstall Anbox from Mint

You can install Anbox on your system from the Snap Store. As of now, Snap is the only way to get Anbox. The organization does not officially support any other distribution method of Anbox at the moment.


In case you haven’t heard about Snaps, don’t worry. Snaps are just software packages that are simple to create and install.


## 1. Install Snap


Snap is available for the following releases of Mint :


- 18.2 (Sonya)
- 18.3 (Sylvia)
- 19 (Tara)
- 19.1 (Tessa)
- 20 (Ulyana)

To install Snap on Linux Mint 20, you need to remove /etc/apt/preferences.d/nosnap.pref  first. The reason being, this file blocks the installation of snap.


This is done with the command :


```
sudo rm /etc/apt/preferences.d/nosnap.pref
sudo apt update

```


To install snapd on your system use the apt command as shown below:


```
$ sudo apt install snapd

```


Alternatively, you can download it form the Software Manager application. Search for snapd and click Install.


Install Snap
## 2. Install Kernel Modules


Before installing Anbox, you need to install two kernel modules. This is necessary to support the mandatory kernel subsystems ashmem and binder for the Android container.


You can do this with the following commands:


```
sudo add-apt-repository ppa:morphis/anbox-support
sudo apt update
sudo apt install linux-headers-generic anbox-modules-dkms

```


This will install anbox-modules-dkms package on your system.


After this, you need to manually load the kernel modules. This loading is a one-time thing. You can do this with the following commands:


```
sudo modprobe ashmem_linux
sudo modprobe binder_linux

```


This will add two new nodes in your system.


```
/dev/ashmen
/dev/binder 

```


## 3. Install Anbox on Linux Mint


Now after installing Snaps and the necessary modules on your system, you can install Anbox on your system using :


```
sudo snap install --devmode --beta anbox

```





To update to a newer version use the command:


```
sudo snap refresh --beta --devmode anbox

```


To get information about the anbox snap use the command:


```
snap info anbox 

```


Anbox Snap Info
## 4. Steps to Uninstall Anbox


If you need to uninstall Anbox, use the command :


```
 $ snap remove anbox

```


After uninstalling anbox, you can also uninstall the kernel modules using :


```
$ sudo apt install ppa-purge
$ sudo ppa-purge ppa:morphis/anbox-support

```


Running these commands will successfully uninstall Anbox from your system.


# That’s it!


In this tutorial, you saw how to install Anbox on your Linux Mint system. Do let us know if you have any questions in the comments below.


