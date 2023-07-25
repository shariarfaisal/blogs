# How to Install and Configure VNC on Debian 10

```Miscellaneous``` ```Applications``` ```Debian```

## Introduction


Virtual Network Computing, or VNC, is a connection system that allows you to use your keyboard and mouse to interact with a graphical desktop environment on a remote server. It helps users who are not yet comfortable with the command line with managing files, software, and settings on a remote server.


In this guide, you’ll set up a VNC server on a Debian 10 server and connect to it securely through an SSH tunnel. You’ll use TightVNC, a fast and lightweight remote control package. This choice will ensure that the VNC connection will be smooth and stable even on slower internet connections.


# Prerequisites


To follow this tutorial, you’ll need:


- One Debian 10 server set up by following the Debian 10 initial server setup guide, including a non-root user with sudo access and a firewall.
- A local computer with a VNC client installed that supports VNC connections over SSH tunnels.

On Windows, you can use TightVNC, RealVNC, or UltraVNC.
On macOS, you can use the built-in Screen Sharing program, or can use a cross-platform app like RealVNC.
On Linux, you can choose from many options, including  vinagre, krdc, RealVNC, or TightVNC.


- On Windows, you can use TightVNC, RealVNC, or UltraVNC.
- On macOS, you can use the built-in Screen Sharing program, or can use a cross-platform app like RealVNC.
- On Linux, you can choose from many options, including  vinagre, krdc, RealVNC, or TightVNC.

Once you have everything set up, you can proceed to the first step.


# Step 1 — Installing the Desktop Environment and VNC Server


By default, a Debian 10 server does not come with a graphical desktop environment or a VNC server installed, so begin by installing those. Specifically, install packages for the latest Xfce desktop environment and the TightVNC package available in the official Debian repository.


On your server, update your list of packages:


```
sudo apt update


```


Now install the Xfce desktop environment on your server:


```
sudo apt install xfce4 xfce4-goodies


```


During the installation, you’ll be prompted to select your keyboard layout from a list of possible options. Choose the one that’s appropriate for your language and press Enter. The installation will continue.


Once the installation completes, install the TightVNC server:


```
sudo apt install tightvncserver


```


To complete the VNC server’s initial configuration after installation, use the vncserver command to set up a secure password and create the initial configuration files:


```
vncserver


```


Next there will be a prompt to enter and verify a password to access your machine remotely:


```
OutputYou will require a password to access your desktops.

Password:
Verify:

```


The password must be between six and eight characters long. Passwords with more than eight characters will be truncated automatically.


Once you verify the password, you have the option to create a view-only password. Users who log in with the view-only password will not be able to control the VNC instance with their mouse or keyboard. This is a helpful option if you want to demonstrate something to other people using your VNC server, but this isn’t required.


The process then creates the necessary default configuration files and connection information for the server:


```
OutputWould you like to enter a view-only password (y/n)? n
xauth:  file /home/sammy/.Xauthority does not exist

New 'X' desktop is your_hostname:1

Creating default startup script /home/sammy/.vnc/xstartup
Starting applications specified in /home/sammy/.vnc/xstartup
Log file is /home/sammy/.vnc/your_hostname:1.log

```


Next, configure it to launch Xfce and give access to the server through a graphical interface.


# Step 2 — Configuring the VNC Server


The VNC server needs to know which commands to execute when it starts up. Specifically, VNC needs to know which graphical desktop it should connect to.


These commands are located in a configuration file called xstartup in the .vnc folder under your home directory. The startup script was created when you ran the vncserver command in the previous step, but you’ll create your own to launch the Xfce desktop.


When VNC is first set up, it launches a default server instance on port 5901. This port is called a display port, and is referred to by VNC as :1. VNC can launch multiple instances on other display ports, like :2, :3, and so on.


Because you are going to be changing how the VNC server is configured, first stop the VNC server instance that is running on port 5901 with the following command:


```
vncserver -kill :1


```


The following is the output with a PID specific to your server environment:


```
OutputKilling Xtightvnc process ID 17648

```


Before you modify the xstartup file, back up the original:


```
mv ~/.vnc/xstartup ~/.vnc/xstartup.bak


```


Now create a new xstartup file and open it in your preferred text editor:


```
nano ~/.vnc/xstartup


```


Commands in this file are executed automatically whenever you start or restart the VNC server. You need VNC to start your desktop environment if it’s not already started. Add the following commands to the file:


~/.vnc/xstartup
```
#!/bin/bash
xrdb $HOME/.Xresources
startxfce4 &

```


Here is a brief overview of what each line is doing:


- 
#!/bin/bash: The first line is a shebang. In executable plain-text files on *nix platforms, a shebang tells the system what interpreter to pass that file to for execution. In this case, you’re passing the file to the Bash interpreter. This will allow each successive line to be executed as commands, in order.

- 
xrdb $HOME/.Xresources: This command tells VNC’s GUI framework to read the user’s .Xresources file. .Xresources is where a user can make changes to certain settings for the graphical desktop, like terminal colors, cursor themes, and font rendering.

- 
startxfce4 &: This command tells the server to launch Xfce. This is where you will find all the graphical software that you need to comfortably manage your server.


When you’re finished, save and exit out of your editor. If you’re using nano, you do so by pressing CTRL+X, then Y, then ENTER.


To ensure that the VNC server will be able to use this new startup file properly, you need to make it executable:


```
sudo chmod +x ~/.vnc/xstartup


```


Now, restart the VNC server:


```
vncserver


```


The output will be similar to the following:


```
OutputNew 'X' desktop is your_hostname:1

Starting applications specified in /home/sammy/.vnc/xstartup
Log file is /home/sammy/.vnc/your_hostname:1.log

```


With the configuration in place, you’re ready to connect to the VNC server from your local machine.


# Step 3 — Connecting the VNC Desktop Securely


VNC itself doesn’t use secure protocols when connecting. To connect securely, you’ll use an SSH tunnel to connect to your server, and then tell your VNC client to use that tunnel rather than making a direct connection.


Create an SSH connection on your local computer that securely forwards to the localhost connection for VNC. You can do this via the terminal on Linux or macOS with the following command. Remember to replace sammy and your_server_ip with your non-root username and the IP address of your server:


```
ssh -L 5901:127.0.0.1:5901 -C -N -l sammy your_server_ip


```


Please note there is no output returned after you run this command in your terminal on your local machine. You will have to use a VNC client to view the graphical interface.


Here’s what this ssh command’s options mean:


- The -L switch specifies the port bindings. In this case you’re binding port 5901 of the remote connection to port 5901 on your local machine.
- The -C switch enables compression to help minimize resource consumption and speed things up.
- The -N switch tells ssh that you don’t want to execute a remote command.
- The -l switch specifies the remote login name.

If you are using PuTTY to connect to your server, you can create an SSH tunnel by right-clicking on the top bar of the terminal window, and then select the Change Settings… option:





Find the Connection branch in the tree menu on the left-hand side of the PuTTY Reconfiguration window. Expand the SSH branch and click on Tunnels. On the Options controlling SSH port forwarding screen, enter 5901 as the Source Port and localhost:5901 as the Destination, like in the following:


Adding the port and destination information into PuTTy
Then click the Add button, and then the Apply button to implement the tunnel.


Once the tunnel is running, use a VNC client to connect to localhost:5901. You’ll be prompted to authenticate using the password you set in Step 1.


Once you are connected, the default Xfce desktop will appear as follows:


![The default Xfce graphical interface for the VNC connection to your Debian 10 server](https://assets.digitalocean.com/articles/vnc_debian10/default-xfce.png “Xfce desktop environment when you first access it via your VNC connection.”)


Select Use default config to configure your desktop.


You can access files in your home directory with the file manager or from the command line, as shown here:





On your local machine, press CTRL+C in your terminal to stop the SSH tunnel and return to your prompt. This will disconnect your VNC session.


Next, you will set up the VNC server as a service.


# Step 4 — Running VNC as a System Service


Next, you’ll set up the VNC server as a systemd service. You can start, stop, and restart it as needed, like any other service. This will also ensure that VNC starts up when your server reboots.


First, create a new unit file called /etc/systemd/system/vncserver@.service using your favorite text editor:


```
sudo nano /etc/systemd/system/vncserver@.service


```


The @ symbol at the end of the name will let you pass in an argument you can use in the service configuration. You’ll use this to specify the VNC display port you want to use when you manage the service.


Add the following lines to the file. Be sure to change the value of User, Group, WorkingDirectory, and the username in the value of PIDFILE to match your username:


/etc/systemd/system/vncserver@.service
```
[Unit]
Description=Start TightVNC server at startup
After=syslog.target network.target

[Service]
Type=forking
User=sammy
Group=sammy
WorkingDirectory=/home/sammy

PIDFile=/home/sammy/.vnc/%H:%i.pid
ExecStartPre=-/usr/bin/vncserver -kill :%i > /dev/null 2>&1
ExecStart=/usr/bin/vncserver -depth 24 -geometry 1280x800 :%i
ExecStop=/usr/bin/vncserver -kill :%i

[Install]
WantedBy=multi-user.target

```


The ExecStartPre command stops VNC if it’s already running. The ExecStart command starts VNC and sets the color depth to 24-bit color with a resolution of 1280x800. You can modify these startup options as well to meet your needs.


Save and close the file when you’re finished.


Next, make the system aware of the new unit file:


```
sudo systemctl daemon-reload


```


Then, enable the unit file:


```
sudo systemctl enable vncserver@1.service


```


The 1 following the @ sign signifies which display number the service should appear over, in this case the default :1 as was discussed in Step 2.


Stop the current instance of the VNC server if it’s still running.


```
vncserver -kill :1


```


Then start it as you would start any other systemd service.


```
sudo systemctl start vncserver@1


```


You can verify that it started with the following command:


```
sudo systemctl status vncserver@1


```


If it started correctly, the output will be similar to the following:


```
Output● vncserver@1.service - Start TightVNC server at startup
Loaded: loaded (/etc/systemd/system/vncserver@.service; enabled; vendor preset: enabled)
Active: active (running) since Fri 2022-08-19 16:21:36 UTC; 5s ago
Process: 24469 ExecStartPre=/usr/bin/vncserver -kill :1 > /dev/null 2>&1 (code=exited, status=2)
Process: 24474 ExecStart=/usr/bin/vncserver -depth 24 -geometry 1280x800 -localhost :1 (code=exited, status=0/SUCCESS)
Main PID: 24482 (Xtightvnc)
. . .

```


Your VNC server will now be available when you reboot the machine.


Start your SSH tunnel again:


```
ssh -L 5901:127.0.0.1:5901 -C -N -l sammy your_server_ip


```


Then make a new connection using your VNC client software to localhost:5901 to connect to your machine.


# Conclusion


You now have a secured VNC server up and running on your Debian 10 server. Now you’re able to manage your files, software, and settings with a user-friendly and familiar graphical interface. You can also run graphical software like web browsers remotely.


