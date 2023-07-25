# How To Set Up a Remote Desktop with X2Go on Ubuntu 20 04

```Linux Basics``` ```Ubuntu``` ```Networking``` ```Open Source```

The author selected Software in the Public Interest (SPI) to receive a donation as part of the Write for DOnations program.


## Introduction


Usually, Linux-based servers don’t come with a graphical user interface (GUI) pre-installed. Whenever you want to run GUI applications on your instance, the typical solution is to employ Virtual Network Computing (VNC). Unfortunately, VNC solutions can be sluggish and insecure; many also require a lot of manual configuration. By contrast, X2Go provides a working “cloud desktop,” complete with all the advantages of an always-online, remotely-accessible, and easily-scalable computing system with a fast network. It is also more responsive and more secure than many VNC solutions.


In this tutorial, you’ll use X2Go to create an Ubuntu 20.04 XFCE desktop environment that you can access remotely. This cloud desktop will include the same utilities that you would obtain had you installed Ubuntu 20.04 and the XFCE environment on your personal computer (almost identical to a Xubuntu setup).


The setup described in this tutorial is useful when:


- You need access to a Linux-based operating system, complete with a desktop environment, but can’t install it on your personal computer.
- You use multiple devices in multiple locations and want a consistent work environment with the same tools, look, files, and performance.
- Your Internet service provider gives you very little bandwidth, but you need access to tens or hundreds of gigabytes of data.
- Long-running jobs make your local computer unavailable for hours or days. Imagine that you have to compile a large project, which will take 8 hours on your laptop. You won’t be able to watch movies or do anything else very resource-intensive while your project compiles. But if you run that job on your server, now your computer is free to perform other tasks.
- You’re working with a team, and it benefits them to have a shared computer that they can access to collaborate on a project.

# Prerequisites


Before starting this tutorial, you’ll need:


- 
An Ubuntu 20.04 x64 instance with 2GB of RAM or more. 2GB is minimal, but a server with 4GB or more is ideal if you have memory-hungry applications that you plan to run. You can use a DigitalOcean Droplet if you like.

- 
A user with sudo privileges and an SSH key. Follow this guide to get started: Initial Server Setup with Ubuntu 20.04. Make sure you complete Step 4 and configure your firewall to restrict all connections except for OpenSSH.


# Step 1 — Installing the Desktop Environment on Your Server


With your server up and your firewall configured, you are now ready to install the graphical environment for the X2Go server.


First, update the package manager’s information about the latest software available:


```
sudo apt-get update


```


In this tutorial, you are installing XFCE as the desktop environment. XFCE doesn’t use graphical effects like compositing, making it more compatible with X2Go and optimizing screen updates. For reference, the LXDE desktop environment and the MATE desktop environment (with compositing disabled) also work fine, but you’ll have to change the command in this tutorial where you install the desktop environment. For example, instead of sudo apt-get install xubuntu-desktop, you would type sudo apt-get install lubuntu-desktop to install LXDE.


There are two ways to install XFCE; the Minimal Desktop Environment or the Full Desktop Environment. The best choice for you will depend on your needs, which we will cover next. Choose one of the two.


## The Full Desktop Environment


Recommended for most use cases. If you don’t want to handpick every component you need and would rather have a default set of packages, like a word processor, web browser, email client, and other accessories pre-installed, you can choose xubuntu-desktop.


Install and configure the Full Desktop Environment. The Full Desktop Environment is similar to what you would get if you installed Xubuntu from a bootable DVD/USB memory stick to your local PC:


```
sudo apt-get install xubuntu-desktop


```


When prompted to choose a display manager, pick lightdm.





## The Minimal Desktop Environment


Alternately, if you want to install a small, core set of packages and then build on top of them by manually adding whatever you need, you can use the xubuntu-core meta-package.


A meta-package doesn’t contain a single package; instead, a meta-package includes an entire package collection. Installing a meta-package saves the user from manually installing numerous components.


Install xfce4 and all of the additional dependencies needed to support it:


```
sudo apt-get install xubuntu-core


```


You have installed a graphical environment. Now you will establish a way to view it remotely.


# Step 2 — Installing X2Go on the Server


X2Go comes with two main components: the server, which starts and manages the graphical session on the remote machine, and the client, which you install on your local computer to view and control the remote desktop or application.


In previous versions of Ubuntu (before 18.04), x2goserver wasn’t included in the default repositories, so you’d have to follow steps like these to get the software package. We’re leaving the link here, just for reference, in case the package gets dropped in future versions of Ubuntu. Fortunately, Ubuntu 20.04, codenamed Focal Fossa, includes the package you need in its default repositories, so the installation is faster.


To install X2Go on your server, type the following command:


```
sudo apt-get install x2goserver x2goserver-xsession


```


At this point, your server requires no further setup. However, keep in mind that if you followed the recommendation of setting up SSH keys in the Initial Server Setup with Ubuntu 20.04, then you will need to have your SSH private key available on every local machine that you intend to use. If you didn’t set up an SSH private key, make sure you choose a strong password.



Note: Remember that if you run out of RAM, the Linux kernel might abruptly terminate some applications, resulting in lost work. If you are using a DigitalOcean Droplet and you notice that your programs require more RAM, you can temporarily power off your Droplet and upgrade (resize) to one with more memory.

You have configured your server. Type exit or close your terminal window. The rest of the steps will focus on configuring the client on your local machine.


# Step 3 — Installing the X2Go Client Locally


X2Go is ready to use out of the box. If you’re using Windows or Mac OS X on your local machine, you can download the X2Go client software here. If you’re using Debian or Ubuntu you can install the X2Go client with this command on your local machine:


```
sudo apt-get install x2goclient


```


After downloading the software, you are ready to install it. Open the installer and select your preferred language. Now agree to the license and let the wizard guide you through the remaining steps. Typically, there shouldn’t be any reason to change the pre-filled, default values in these steps.


X2Go works well out of the box, but it is also highly customizable. If you’d like additional information, visit X2Go’s official documentation.


Now that you have installed the desktop client, you can configure its settings and connect to the X2Go server to use your remote XFCE desktop.


# Step 4 — Connecting To the Remote Desktop


When you first open the X2Go client, a window will appear. If it doesn’t, click Session in the top-left menu and then select New session ….





In the Session name field, enter something to help differentiate between servers. Using a session name is particularly useful if you plan on connecting to multiple machines.


Enter your server’s IP address or a fully qualified domain name (FQDN) in the Host field under Server.


Enter the username you used for your SSH connection in the Login field.


Since you installed XFCE in Step Two, choose XFCE as your Session type.


Finally, because you connect to the server with SSH keys, click the folder icon next to Use RSA/DSA key for ssh connection and browse to your private key. If you didn’t opt to use the more secure SSH keys, leave this empty; the X2Go client will ask for a password each time you log in.


The rest of the default settings will suffice for now, but as you get more familiar with the software, you can fine-tune the client based on your individual preferences.


After pressing the OK button, you can start your graphical session by clicking the white box that includes your session’s name on the box’s top-right side.





If you are running OS X on your local machine, OS X might prompt you to install XQuartz, which is required to run X11. If so, follow the instructions to install it now.


In a few seconds, your remote desktop will appear, and you can start interacting with it.


There are a few useful keyboard shortcuts you can use for a better experience on Windows and Linux-based operating systems.



Note: These first two options can exhibit buggy behavior on modern Windows editions. You can still test them at this point, in case later versions of X2Go fix the issues. If they fail, just avoid using the same keyboard shortcut in the future.

CTRL+ALT+F will toggle full-screen mode on and off. Working in full-screen mode can feel more like a local desktop experience.  The full-screen mode also helps the remote machine grab keyboard shortcuts instead of your local machine.


CTRL+ALT+M will minimize the remote view, even if you are in full-screen mode.


CTRL+ALT+T will disconnect from the session but leave the GUI running on the server. It’s just a quick way of disconnecting without logging off or closing applications on the server. The same will happen if you click the window’s close button.


Lastly, there are two ways you can end the remote session and close all of the graphical programs running in it. You can log off remotely from XFCE’s start menu, or you can click the button marked with a circle and a small line (like a power/standby icon) in the bottom-right corner of the main portion of the X2Go screen.


The first method is cleaner but may leave programs like session-managing software running. The second method will close everything but may do so forcefully if a process can’t exit cleanly. In either case, be sure to save your work before proceeding.





You have now successfully accessed and configured your remote desktop.


# Conclusion


In this tutorial, you used X2Go to create a robust and remote GUI-environment for the Ubuntu operating system. Now that you are up and running, here are a few ideas about using this desktop:


- You could centralize your development work by creating a git repository.
- You could install an IDE/code editor like NetBeans or Eclipse. You could also use Visual Studio Code for remote development via the Remote-SSH Plugin.
- You could configure a web server for testing web applications.
- You could also enhance your remote desktop with a good backup scheme to preserve your work environment and essential data in case something ever goes wrong. With DigitalOcean, you can also snapshot your Droplets when you’re happy with a particular setup. This way, you can test risky changes and always come back to a known, working state.

If you’d like to learn more, visit X2Go’s official documentation website.


