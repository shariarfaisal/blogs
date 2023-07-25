# How To Install VirtualBox on CentOS 6 3 x64

```CentOS```










# Status: Deprecated


This article covers a version of CentOS that is no longer supported. If you are currently operating a server running CentOS 6, we highly recommend upgrading or migrating to a supported version of CentOS.


Reason:
CentOS 6 reached end of life (EOL) on November 30th, 2020 and no longer receives security patches or updates. For this reason, this guide is no longer maintained.


See Instead:
This guide might still be useful as a reference, but may not work on other CentOS releases. If available, we strongly recommend using a guide written for the version of CentOS you are using.


## Step 1 - Installation


```
yum -y groupinstall Desktop
yum -y install qt qt-devel SDL-devel tigervnc-server gcc make kernel-devel
export KERN_DIR=/usr/src/kernels/2.6.32-358.2.1.el6.x86_64
rpm -ivh http://download.virtualbox.org/virtualbox/4.2.10/VirtualBox-4.2-4.2.10_84104_el6-1.x86_64.rpm
/etc/init.d/vboxdrv setup

```


## Step 2 - Set VNC Server


```
vncpasswd
```


Add these two lines to /etc/sysconfig/vncservers :


```
VNCSERVERS="0:root"
VNCSERVERARGS[0]="-geometry 1024x768"

```


If you do not want to run VNC as user root, and would like to create an encrypted tunnel, create a separate user, and add -localhost to VNCSERVERARGS above.


You would then have to create a tunnel using Putty or some other SSH client, and map localhost:port -> localhost:5901 tunnel, and restart vncserver service for changes 
to take effect.


## Step 3 - Start VNC Server


```
service vncserver start
```


## Step 4 - Download TightVNC Viewer and connect to your droplet's IP port 5900.


When connecting with TightVNC Viewer, remote host address would be in IP::PORT format:


![](https://assets.digitalocean.com/articles/community/CentOS-TightVNC-Host.png)
Enter Your VNC Password from Step 2 above:


![](https://assets.digitalocean.com/articles/community/CentOS-TightVNC-Passwd.png)
## Step 5 - Upload your desired ISO Disk or Disk Image to be imported.


You can install any Linux / BSD / Windows OS with an uploaded ISO image.


If you have an existing Disk Image with filetypes such as VDI, VMDK, QCOW, or RAW, you can import them into VirtualBox.


## Step 6 - Launch VirtualBox and Install Virtual Machines


Once you are connected via VNC, click Applications -> System Tools -> Oracle VM VirtualBox


![](https://assets.digitalocean.com/articles/community/CentOS-VBox-LaunchMenu.png)
Using VirtualBox, you can even install Windows operating system, but they will be limited to 1 CPU Core.


Here is Windows XP running in VirtualBox:


![](https://assets.digitalocean.com/articles/community/CentOS-VirtualBox-WinXP.png)
By Bulat Khamitov
