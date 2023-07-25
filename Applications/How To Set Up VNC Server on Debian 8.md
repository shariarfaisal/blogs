# How To Set Up VNC Server on Debian 8

```Applications``` ```Debian```

## Introduction


VNC (Virtual Network Computing) is a system that enables users to connect and interact with graphical desktops of remote computers. It can transmit screen updates, and keyboard and mouse events, over the network.


VNC is useful when you need a graphical desktop environment for your server.


XFCE is a lightweight desktop environment. Because it has low system resource requirements, and because many VNC users are familiar with it, we will use XFCE in this tutorial. However, you can also use your favorite desktop environment, such as Gnome or KDE, instead.


In this tutorial we will set up a Debian 8 server, install the XFCE desktop environment on it, and connect it to via VNC. Additionally, we will create a startup script for VNC Server and secure it over SSH.


# Prerequisities


Please complete the following prerequisites.


- Debian 8 (or 8.1) Droplet with root access. 512 MB of RAM is enough to run VNC and XFCE, but you might need a bigger Droplet depending on what you plan to do with the graphical interface
- VNC viewer (client) on your computer to connect to your server. In this tutorial, we will use UltraVNC on Windows, but you can use other VNC clients. You can download UltraVNC here. OS X comes with a built in VNC client called Screen Sharing
- SSH client to establish a secure connection over SSH. We will use PuTTY for Windows. You can download PuTTY here. On OS X, just use the built in Terminal application

# Step 1 — Installing VNC and XFCE


In this step, we will install VNC Server and the XFCE desktop environment, with additional software and an icon pack.


Update your server’s package lists:


```
apt-get update


```


Upgrade the packages themselves:


```
apt-get -y upgrade


```


Then, we will install tightvncserver and XFCE4 with some useful add-ons, and an icon theme:


```
apt-get install xfce4 xfce4-goodies gnome-icon-theme tightvncserver


```


By default there is no browser installed. You can install iceweasel (which is a rebranded version of Mozilla Firefox for Debian) if you want to access the web from your VNC connection:


```
apt-get install iceweasel


```


# Step 2 — Creating a VNC User


We will create a separate user for VNC connections, to keep things secure and tidy. Using sudo is highly recommended, instead of using the root user directly for your VNC server.


You can add a user named vnc to your Debian Droplet by using this command:


```
adduser vnc


```


Give a password to your new user. You can skip all other questions by simply pressing ENTER.


Install sudo by executing this command:


```
apt-get install sudo


```


Add your new vnc user to the sudo group, which will give permissions to that user to execute root commands.


```
gpasswd -a vnc sudo


```


Let’s switch to the vnc user:


```
su - vnc


```


# Step 3 — Starting and Stopping Your VNC Server


As our newly created vnc user, we can start VNC Server and test our connection.


Start VNC Server:


```
vncserver


```


As it is your first time running the server, you will be asked to set a password that clients will use to connect. Keep this password in mind for later! You can also set a view-only password, which will allow users to see the screen but not interact with it. Passwords should be 6-8 characters.


You will get a notice about your display number when the server is started.


Output
```
xauth:  file /home/vnc/.Xauthority does not exist

New 'X' desktop is vnc:1

Creating default startup script /home/vnc/.vnc/xstartup
Starting applications specified in /home/vnc/.vnc/xstartup
Log file is /home/vnc/.vnc/vnc:1.log

```


By default, VNC connections are served on ports starting at 5901 for the first display. Your second display will be served on port 5902, etc.


Don’t stop the server now, but we’re including the stop command for reference.



Use this command to stop your VNC server on Display 1 (and port 5901):
vncserver -kill :1


:1 is the display number you want to kill.
You can start VNC Server manually when you want to connect again. We’ll create a service for VNC Server in a later step.

# Step 4 — Connecting from a VNC Client


You can now connect to your VNC server. Open your local VNC client, which will vary depending on your operating system.


On Windows, you can use UltraVNC here.


On OS X, you can use the built-in Screen Sharing app or access this app through Safari. In Safari, you can enter vnc://your_server_ip:5901


For your VNC Server address, enter your_server_ip:5901 and use the password you just set for your VNC connection.


You can select the Use default config button on the XFCE welcome screen to get started easily:





Now you can use your remote desktop!


# Step 5 — Creating a systemd Service to Start VNC Server Automatically


In this section we’ll add VNC Server to systemd. Using a service can be useful to start and stop your VNC server, and also to start it automatically when your Droplet is rebooted.


First, let’s kill the current instance:


```
vncserver -kill :1


```


Create a simple script to manage and configure our VNC server easily:


As the vnc or other sudo user, create a script file by using your favorite text editor.


```
sudo nano /usr/local/bin/myvncserver


```


Add these contents exactly. This script provides VNC with a few parameters for startup.


/usr/local/bin/myvncserver
```
#!/bin/bash
PATH="$PATH:/usr/bin/"
DISPLAY="1"
DEPTH="16"
GEOMETRY="1024x768"
OPTIONS="-depth ${DEPTH} -geometry ${GEOMETRY} :${DISPLAY}"

case "$1" in
start)
/usr/bin/vncserver ${OPTIONS}
;;

stop)
/usr/bin/vncserver -kill :${DISPLAY}
;;

restart)
$0 stop
$0 start
;;
esac
exit 0

```


You can modify the script to change the color depth or resolution of your VNC connection.


If you are using nano, you can save the file via CTRL+O and exit via CTRL+X.


Make the file executable:


```
sudo chmod +x /usr/local/bin/myvncserver


```


Our script will help us to modify settings and start/stop VNC Server easily.



If you’d like, you can call the script manually to start/stop VNC Server on port 5901 with your desired configuration.
sudo /usr/local/bin/myvncserver start
sudo /usr/local/bin/myvncserver stop
sudo /usr/local/bin/myvncserver restart



We can now create a unit file for our service. Unit files are used to describe services and tell the computer what to do to start/stop or restart the service.


```
sudo nano /lib/systemd/system/myvncserver.service


```


Copy these commands to the service file. Our service will simply call the startup script above with the user vnc.


/lib/systemd/system/myvncserver.service
```
[Unit]
Description=Manage VNC Server on this droplet

[Service]
Type=forking
ExecStart=/usr/local/bin/myvncserver start
ExecStop=/usr/local/bin/myvncserver stop
ExecReload=/usr/local/bin/myvncserver restart
User=vnc

[Install]
WantedBy=multi-user.target

```


Now we can reload systemctl and  enable our service:


```
sudo systemctl daemon-reload
sudo systemctl enable myvncserver.service


```


You’ve enabled your new service now. Use these commands to start, stop or restart the service using the systemctl command:


```
sudo systemctl start myvncserver.service
sudo systemctl stop myvncserver.service
sudo systemctl restart myvncserver.service


```


Now you can run VNC Server as a service on your Droplet.


# Step 6 — Securing Your VNC Server with SSH Tunneling


By default VNC connections don’t use encryption, so it is recommended to use an SSH Tunnel to secure your session.


To do that, we will only let our VNC server serve on localhost.
You can do that by adding -localhost to the OPTIONS line in the startup script created in the previous step.


First, stop the VNC server:


```
sudo systemctl stop myvncserver.service


```


Edit your configuration script:


```
sudo nano /usr/local/bin/myvncserver


```


Change this line:


/usr/local/bin/myvncserver
```
. . .

OPTIONS="-depth ${DEPTH} -geometry ${GEOMETRY} :${DISPLAY}"

. . .

```


Replace it with:


/usr/local/bin/myvncserver
```
. . .

OPTIONS="-depth ${DEPTH} -geometry ${GEOMETRY} :${DISPLAY} -localhost"

. . .

```


Restart the VNC server:


```
sudo systemctl start myvncserver.service


```


Now you can’t directly connect to your VNC server from your remote computer.


Windows:


We will use PuTTY to create an SSH Tunnel and then connect through the tunnel we have created.


Open PuTTY.


From the left menu, go to the Connection->SSH->Tunnels section.


In the Add New Forwarded Port section, enter 5901 as Source port and localhost:5901 as Destination.


Click the Add button.





You can now go to the Session section in the left menu.


Enter your Droplet’s IP address in the Host Name (or IP address) field.


Click the Open button to connect. You can also save these options for later use.





Log in with your vnc user.


Keep the PuTTY window open while you make your VNC connection.


Now you can use your VNC viewer as usual. Just enter localhost::5901 as the address, and keep your SSH connection live in the background.





OS X:


To establish an SSH tunnel, use the following line in Terminal:


```
ssh vnc@your_server_ip -L 5901:localhost:5901

```


Authenticate as normal for the vnc user for SSH. Then, in the Screen Sharing app, use localhost:5901.


# Conclusion


Now you can use a shared remote desktop on your Debian 8 server.


Use it to configure your server, or share your screen with others.


