# How To Install Vagrant on a VPS Running Ubuntu 12 04

```Linux Basics``` ```Ubuntu``` ```Miscellaneous``` ```System Tools```











# Status: Deprecated


This article covers a version of Ubuntu that is no longer supported.  If you are currently operate a server running Ubuntu 12.04, we highly recommend upgrading or migrating to a supported version of Ubuntu:


- Upgrade to Ubuntu 14.04.
- Upgrade from Ubuntu 14.04 to Ubuntu 16.04
- Migrate the server data to a supported version

Reason:
Ubuntu 12.04 reached end of life (EOL) on April 28, 2017 and no longer receives security patches or updates.  This guide is no longer maintained.


See Instead:
This guide might still be useful as a reference, but may not work on other Ubuntu releases.  If available, we strongly recommend using a guide written for the version of Ubuntu you are using.   You can use the search functionality at the top of the page to find a more recent version.


## About Vagrant


Vagrant is a great open source software for configuring and deploying multiple development environments. It works on Linux, Mac OS X, or Windows and although by default it uses VirtualBox for managing the virtualization, it can be used with other providers such as VMware or AWS.


The good thing about Vagrant is that by using a central place for configuration, you can deploy virtual private machines packed with all you need. Moreover, it allows team members to run multiple environments with the same exact configuration.


# Installation


To install Vagrant on your cloud server, you need to download and run the installation kit. Before going further, be sure that you have dpkg and Virtual box installed:


```
sudo apt-get install dpkg-dev virtualbox-dkms
```


Go to the downloads page of Vagrant and check for the latest release. Once you are looking at the different versions of the latest release, right click over the one with the .deb extension and copy the link address. Then head back to your terminal and run the following command:


```
wget http://files.vagrantup.com/packages/0219bb87725aac28a97c0e924c310cc97831fd9d/vagrant_1.2.4_i686.deb
```


Replace the URL you see above (following the wget command) with the one you just copied. This will download Vagrant to your system. Next, install the package with the following command:


```
dpkg -i vagrant_1.2.4_i686.deb
```


Make sure you replace the file name above with the one that was downloaded before. Next up, you’ll need to take care a couple of more things. First, install Kernel headers:


```
sudo apt-get install linux-headers-$(uname -r)
```


Then, reconfigure the VirtualBox DKMS:


```
sudo dpkg-reconfigure virtualbox-dkms
```


And now you are ready for the good stuff.


# Getting Started with Vagrant


The idea behind Vagrant is to be able to quickly deploy development environments. That means you can set up some basic configuration once and then you or your team members can quickly deploy virtual private machines with identical software and settings. Part of what this makes work are virtual images called boxes.


So let’s install a box that can be used by multiple Vagrant environments later on. This is done with the vagrant box add command. Run the following command to install the precise32 box from the Vagrant website:


```
vagrant box add precise32 http://files.vagrantup.com/precise32.box
```


You should get the following success message: "Successfully added box 'precise32' with provider 'virtualbox'!". Now you have an image for a VPS with the Ubuntu 12.04 operating system on it.


Each project you run is created using such a box. This means that if you have 3 different projects based on the same box, changing one or another does not affect the original box. Now let’s set up a first project that will be deployed based on the precise32 box we just added to Vagrant.


Create a root directory for your project and navigate in it:


```
mkdir test_project
cd test_project
```


Next, run the initialization command:


```
vagrant init
```


This will create a Vagrantfile in this folder which will be the central file for your project configuration. But before we can deploy the guest machine using the box we just added, edit the Vagrantfile:


```
nano Vagrantfile
```


Find the following line:


```
config.vm.box = "base"
```


And replace it with:


```
config.vm.box = "precise32"
```


This will tell it to use this new box. Save the file and exit. Now you can deploy the guest machine with the following command:


```
vagrant up
```


This will bring up a VPS running Ubuntu 12.04 LTS. To make use of it, you can easily SSH into it:


```
vagrant ssh
```


This will put you directly into an SSH session with the new guest machine. A cool thing is that by default, Vagrant will share the project root folder from the host machine (the one containing the Vagrantfile) with a folder on the guest machine, /vagrant. This means that you can save files on the guest machine that will remain on the host, and vice versa.


After you are done working with the guest machine, you can exit and go back to the host with the following command:


```
exit
```


And if you want to stop and remove the guest machine and all traces of it, run the following command from the host machine:


```
vagrant destroy
```


Please note that the files that were synchronized with the host machine will not be removed from the host. Additionally, you can redeploy the guest machine again for this configuration using the same vagrant up command.


# Conclusion


In this tutorial, you learned how to set up Vagrant and how to configure a simple Ubuntu VPS. In the next tutorial, we will go a bit deeper and talk more about boxes, operating systems and the automatic installation of various software on the guest machines.


Vagrant—Article #1


Vagrant—Article #2


Vagrant—Article #3


Article Submitted by: Danny
