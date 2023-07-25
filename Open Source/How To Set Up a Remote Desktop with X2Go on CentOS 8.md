# How To Set Up a Remote Desktop with X2Go on CentOS 8

```Networking``` ```Open Source``` ```CentOS```

The author selected Software in the Public Interest (SPI) to receive a donation as part of the Write for DOnations program.


## Introduction


Usually, Linux-based servers don’t come with a graphical user interface (GUI) pre-installed. Whenever you want to run GUI applications on your instance, the typical solution is to employ Virtual Network Computing (VNC). Unfortunately, VNC solutions can be sluggish and insecure; many also require a lot of manual configuration. By contrast, X2Go provides a working “cloud desktop,” complete with all the advantages of an always-online, remotely-accessible, and easily-scalable computing system with a fast network. It is also more responsive and more secure than many VNC solutions.


In this tutorial, you’ll use X2Go to create an XFCE desktop environment that you can access remotely.


The setup described in this tutorial is useful when:


- You need access to a Linux-based operating system, complete with a desktop environment, but can’t install it on your personal computer.
- You use multiple devices in multiple locations (e.g., personal laptop, work laptop) but want a persistent work environment that has the same tools, look, files, and performance. Imagine a scenario where you forget to take your work laptop with you, or it malfunctions and you have to send it for repair. Since everything on a remote desktop stays the same, you don’t have to reinstall the whole suite of utilities you like to use to do your job. You just log in to the remote desktop and everything will be there, exactly as you left it, no matter what device you are logging in from.
- Your Internet service provider gives you very little bandwidth, but you need access to tens or hundreds of gigabytes of data. For example, you can download 100GB of data, taking advantage of the server’s very fast network. Since the server itself is the one using bandwidth you don’t use the quota your Internet Service Provider allocates to you monthly. You only pay for the network bandwidth required to send you images of your remote desktop, which is very small, in comparison.
- Long-running jobs make your local computer unavailable for hours or days. Imagine that you have to compile a large project, which will take 8 hours on your laptop. You won’t be able to watch movies or do anything else very resource-intensive while your project compiles. But if you run that job on your server, now your computer is free to perform other tasks.
- You’re working with a team, and it benefits all members to have a common, shared computer that they can access to collaborate on a project.

# Prerequisites


Before starting this tutorial, you’ll need:


- 
A CentOS 8 x64 instance with 2GB of RAM or more. 2GB is minimal, but a server with 4GB or more is ideal if you have memory-hungry applications that you plan to run. You can use a DigitalOcean Droplet if you like.

- 
A user with sudo privileges and an SSH key. Follow this guide to get started: Initial Server Setup with CentOS 8. At Step 4, configure the firewall as instructed, but don’t enter the command firewall-cmd --permanent --add-service=http. This command opens post 80 on your server, which you don’t need.


# Step 1 — Installing the Desktop Environment on Your Server


With your server up and your firewall configured, you are now ready to install the graphical environment.


First, upgrade all software packages on your instance:


```
sudo dnf upgrade


```


In this tutorial, you are installing XFCE as the desktop environment. XFCE doesn’t use graphical effects like compositing, making it more compatible with X2Go and allowing it to optimize how screen updates are sent across the network. In other words, it’s probably the easiest one to get working with X2Go since everything immediately works well out of the box. LXDE should also work out of the box. MATE and KDE desktop environments might work too, but some workarounds might be needed, for example, disabling compositing to improve performance and responsiveness.


If you prefer a different desktop environment, you would have to replace a command such as sudo dnf groupinstall Xfce from this tutorial with sudo dnf group install "KDE Plasma Workspaces" to install KDE, and then configure the Session type in your X2Go client to KDE. You can also install multiple desktop environments, alongside each other, and then pick the one you prefer to launch, each time you log in with your X2Go client.


The software packages you now need to install aren’t contained in CentOS’ default repositories, so you have to enable the Extra Packages for Enterprise Linux (EPEL) repositories:


```
sudo dnf install epel-release


```


Some X2Go server utilities also depend on some packages found in the PowerTools repository, which you can enable with the next command:


```
sudo dnf config-manager --set-enabled PowerTools


```


Finally, you can now install the XFCE desktop environment:


```
sudo dnf groupinstall Xfce


```


CentOS offers a rather minimal XFCE environment, but you can add any other utilities you need (e.g., web browsers, image editors) with simple sudo dnf install name_of_application commands.


With the desktop environment now installed, it’s time to establish a way to view it on your local computer.


# Step 2 — Installing X2Go on the Server


X2Go comes with two main components: the server, which starts and manages the graphical session on the remote machine, and the client, which you install on your local computer to view and control the remote desktop or application.


To install X2Go on your server, type the following command:


```
sudo dnf install x2goserver


```


At this point, your server requires no further setup. However, keep in mind that if you followed the recommendation of setting up SSH keys in the Initial Server Setup with CentOS 8, then you will need to have your SSH private key available on every local machine that you intend to log in from, to your remote desktop session. If you didn’t set up an SSH private key, make sure you choose a strong password.



Note: Remember that if you run out of RAM, the Linux kernel might abruptly terminate some applications, resulting in lost work. If you are using a DigitalOcean Droplet and you notice that your programs require more RAM, you can temporarily power off your Droplet and upgrade (resize) to one with more memory.

You have configured your server. Type exit or close your terminal window. The rest of the steps will focus on configuring the client on your local machine.


# Step 3 — Installing the X2Go Client Locally


X2Go is ready to use out of the box. If you’re using Windows or macOS on your local machine, you can download the X2Go client software here. If you’re using Debian or Ubuntu you can install the X2Go client with this command on your local machine:


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


In this tutorial, you used X2Go to create a robust and remote GUI-environment for the CentOS operating system. Now that you are up and running, here are a few ideas about using this desktop:


- You could centralize your development work by creating a git repository.
- You could install an IDE/code editor like NetBeans or Eclipse. You could also use Visual Studio Code for remote development via the Remote-SSH Plugin.
- You could configure a web server for testing web applications. By opening up a browser on your remote desktop, and navigating to localhost in the address bar, you can test the web application you are working on, without having to constantly re-upload files to some remote server. And everything is encapsulated within the secure and private SSH connection used by X2Go to view your remote desktop. This means that although you are testing your website on a server connected to the Internet, the web app you are working on can only be viewed by you, it is not publicly accessible, so you don’t have to worry about accidentally leaking unfinished and unpolished work.
- You could also enhance your remote desktop with a good backup scheme to preserve your work environment and essential data in case something ever goes wrong. With DigitalOcean, you can also snapshot your Droplets when you’re happy with a particular setup. This way, you can test risky changes and quickly come back to a known, working state, if it’s ever required, something that is not so easy to do on a typical Windows computer. Snapshots also allow you to clone a particular setup. This would make it possible to give all team members their own, private, remote desktops, where they can perform work in similar environments, as far as development tools are concerned. But this way, they would each be able to customize their environment as they like, and work with their own sets of files.

All in all, you can create a complete development environment, with all the bells and whistles, from the code editor you use, to the software required to run and test that code.


If you’d like to learn more, visit X2Go’s official documentation website.


