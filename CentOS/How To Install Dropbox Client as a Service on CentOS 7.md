# How To Install Dropbox Client as a Service on CentOS 7

```Miscellaneous``` ```CentOS```

## Introduction


In this tutorial, we’ll show you how to install the Dropbox client, and configure it to run as a headless service, on a CentOS 7 server. This will allow your server to connect to Dropbox so that you can keep a copy of your Dropbox files synchronized on your server.


# Prerequisites


You must have a non-root user with superuser privileges (sudo). To set that up, follow at least steps 1 through 3 in the Initial Server Setup with CentOS 7 tutorial. All of the commands in this tutorial will be executed as this non-root user.


Once you’re ready, we’ll install the Dropbox client.


# Install Dropbox Client


The latest version of the Linux Dropbox client can be downloaded to your home directory with these commands:


```
cd ~
curl -Lo dropbox-linux-x86_64.tar.gz https://www.dropbox.com/download?plat=lnx.x86_64


```


Now you will have a file called dropbox-linux-x86_64.tar.gz in your home directory.



Note: If you’re running a 32-bit distribution, use this command to download the 32-bit Linux client instead:
cd ~
curl -Lo dropbox-linux-x86.tar.gz https://www.dropbox.com/download?plat=lnx.x86


Next, extract the contents of the Dropbox archive to /opt/dropbox with these commands:


```
sudo mkdir -p /opt/dropbox
sudo tar xzfv dropbox-linux-x86_64.tar.gz --strip 1 -C /opt/dropbox

```


The Dropbox client is now on your server, but you need to link it with your Dropbox account.


# Link Dropbox Client


To link your Dropbox client with your Dropbox account, run this command (as the user whose home directory you want to store the Dropbox files in):


```
/opt/dropbox/dropboxd


```


This starts the Dropbox client in the foreground, so you won’t be able to enter any other commands at the moment. The first time you run the client, you should see output that looks like this:


```
Host ID Link:This computer isn't linked to any Dropbox account...
Please visit https://www.dropbox.com/cli_link_nonce?nonce=ac8d12e1f599137703d88f2949c265eb to link this device.

```


Visit the URL in the output (highlighted in the above example) in a web browser on your local computer.


Log in to Dropbox (if you aren’t already logged in), then click the connect button:





After seeing a success message in your web browser, you should see this output on your CentOS server:


```
Link success output:This computer is now linked to Dropbox. Welcome Sammy

```


Now your Dropbox account is linked with the client. You should now have a directory in your home directory called “Dropbox”. This is where it will store your synchronized Dropbox files.


Press Ctrl-C to quit running Dropbox for now.


The next step is to set up some scripts so that Dropbox will run as a service, so that you don’t need to be logged in for the client to keep running.


# Set Up Service Script


To start Dropbox as a service, you’ll need to create an init script and a Systemd unit file. To save yourself the trouble, you can use this command to download them:


```
sudo curl -o /etc/init.d/dropbox https://gist.githubusercontent.com/thisismitch/6293d3f7f5fa37ca6eab/raw/2b326bf77368cbe5d01af21c623cd4dd75528c3d/dropbox
sudo curl -o /etc/systemd/system/dropbox.service https://gist.githubusercontent.com/thisismitch/6293d3f7f5fa37ca6eab/raw/99947e2ef986492fecbe1b7bfbaa303fefc42a62/dropbox.service


```


Next, make the scripts executable with this command:


```
sudo chmod +x /etc/systemd/system/dropbox.service /etc/init.d/dropbox


```


The script expects the /etc/systemd/dropbox file to contain a list of system users that will run Dropbox. Create the file and open it for editing with this command:


```
sudo nano /etc/sysconfig/dropbox


```


Add a line that specifies that DROPBOX_USERS is equal to your system username. For example, if your username is “sammy”, it should look like this:


/etc/sysconfig/dropbox
```
DROPBOX_USERS="sammy"

```


Save and exit the file by pressing Ctrl-x, then y, then Enter.


Reload the Systemd daemon, so that you can use the unit file:


```
sudo systemctl daemon-reload


```


Now Dropbox is ready to be started as a service. Run this command to start it:


```
sudo systemctl start dropbox


```


Then run this command to configure the service to start when your server boots:


```
sudo systemctl enable dropbox


```


Now the Dropbox client is running as a service and will start automatically when your server boots.


# Install Dropbox CLI


Dropbox also includes a command line interface (CLI) that you may want to install so that you can configure your Dropbox client.


To download it to your home directory, run these commands:


```
cd ~
curl -LO https://www.dropbox.com/download?dl=packages/dropbox.py

```


Now you will have a file called dropbox.py, the Dropbox CLI, in your home directory.


Use this command to make it executable:


```
chmod +x ~/dropbox.py

```


Then, in your home directory, make a symbolic link named .dropbox-dist that points to your Dropbox installation path. This is necessary because the Dropbox CLI expects ~/.dropbox-dist to contain your Dropbox installation:


```
ln -s /opt/dropbox ~/.dropbox-dist


```


Now you can run the Dropbox CLI from your home directory with this command:


```
~/dropbox.py


```


This will print out a basic help page. The next subsection will cover how use the Dropbox CLI to do a few basic things.


## How to Use the Dropbox CLI


Remember that running the CLI without any options with print out how to use it.


If you want to check the status of your Dropbox, use the status command:


```
~/dropbox.py status


```


If all of your files are synchronized, you should see this message:


```
Output:Up to date

```


You can also use it to turn off the automatic LAN sync feature, which tries to synchronize relevant files on your LAN:


```
~/dropbox.py lansync n


```


Another handy command is exclude. This will let you specify files and directories that should not be synchronized on your server. For example, if you don’t want your server to download the photos directory from Dropbox, you could run this command:


```
~/dropbox.py exclude  add ~/Dropbox/photos


```


Then you can verify which files and directories are excluded from your server with this command:


```
~/dropbox.py exclude list


```


Feel free to play with the CLI to see what else you can do with it.


# How to Link Additional Dropbox Accounts


If you want to link more Dropbox accounts, follow this section.


It is possible to link multiple Dropbox accounts to your server. However, you will require an additional system user for each Dropbox account that you want to link. If you don’t know how to add users to your CentOS server, follow this tutorial: How To Add and Delete Users on CentOS.


Once you have the system user account that you want to use, log in to your server as that user.


Run /opt/dropbox/dropboxd. As before, this will output a URL to link a Dropbox account to your server.


Log in to Dropbox under the account that you want to link to your server. Then visit the URL on your server, and click the connect button.


Next, edit /etc/default/dropbox:


```
sudo nano /etc/default/dropbox


```


Add the new system user to the list of Dropbox users. For example, if you have two system users running Dropbox, “sammy” and “ben”, it would look something like this.


/etc/default/dropbox
```
DROPBOX_USERS="sammy ben"

```


Save and exit the file by pressing Ctrl-x, then y, then Enter.


Now restart the Dropbox service:


```
sudo service dropbox restart


```


Now your server is linked to multiple Dropbox accounts.


To use the CLI on the new user, be sure to follow the Install Dropbox CLI section again as the new user.


# How To Unlink a Dropbox Account


If you want to unlink a Dropbox account, follow these steps.


First, stop the service:


```
sudo service dropbox stop


```


Then edit /etc/defaults/dropbox and remove the user from the list.


Then delete the user’s Dropbox directory. For example:


```
sudo rm -r ~/ben/Dropbox


```


Then, if your server still has other Dropbox accounts linked to it, start the Dropbox client again:


```
sudo service dropbox start


```


Lastly, if you want to restrict access completely, you can go to your Dropbox Account Security page and delete any linked devices.


# Conclusion


The Dropbox client is now installed and running on your server. Your server should now be linked and synchronized with your Dropbox account.


