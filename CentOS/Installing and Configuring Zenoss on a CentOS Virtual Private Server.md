# Installing and Configuring Zenoss on a CentOS Virtual Private Server

```Monitoring``` ```CentOS```

## Introduction


Zenoss is a network and device management application that is built upon the Zope application server.  You can use Zenoss to monitor your various VPS instances in the cloud.


Currently, Zenoss officially supports 64-bit Red Hat Enterprise Linux and 64-bit CentOS.  We will be using a CentOS 6.4 64-bit image on our virtual private server.


Zenoss requires 4GB of RAM to operate correctly, so we will be using a Droplet with 4GB of RAM and 60GB of SSD space.


We will also be configuring two client VPS instances for Zenoss to monitor.  We will be using Ubuntu 12.04 on the smallest VPS size available.


# Installation


Once you have your CentOS VPS created, SSH into it as root.



Before we begin, we need to remove some MySQL libraries that CentOS includes by default.  The Zenoss installation will complain about version conflicts if it encounters these files:


```
It appears that the distro-supplied version of MySQL is at least partially installed,
or a prior installation attempt failed.

Please remove these packages, as well as their dependencies (often postfix), and then
retry this script:

mysql-libs-5.1.69-1.el6_4.x86_64
```


We will remove the offending files before we begin:


```
yum remove mysql-libs
```


Zenoss provides an installation script that will do the majority of the heavy lifting for our installation.  We will acquire the Zenoss files from their website:


```
cd ~
wget --no-check-certificate https://github.com/zenoss/core-autodeploy/tarball/4.2.4 -O auto.tar.gz
```


We can now unzip the files, move into the directory, and run the auto-install script:


```
tar xvf auto.tar.gz
cd zenoss-core-autodeploy-*
./core-autodeploy.sh
```


Press "Enter" to continue.


You will be presented with the License agreement.  Read and then press "Q" to continue:


```
Q
```


You will be asked if you accept the license.  Type "yes" to continue.


```
yes
```


Zenoss will now begin downloading and configuring the needed components.


At some point during the installation, you will be asked whether you wish to set the MySQL root password.  Type "Y" to choose a password:


```
MySQL is configured with a blank root password.
Configure a secure MySQL root password? [Yn]: Y
```


Choose a password and confirm it.


The installation can take quite a long time.  This is normal.


# Client Configuration


There are a few things we will configure to give us a useful environment for Zenoss to manage.


We will configure two "client" machines that the Zenoss machine will monitor.  Our clients will be running Ubuntu 12.04 on small VPS instances.


## SNMP Client Configuration


On one of the Ubuntu installations, we will install an SNMP daemon, which will allow Zenoss to gather information about the client.  On one client, type the following command:


```
sudo apt-get update
sudo apt-get install snmpd
```


After installation, we need to configure the daemon.  First, we will go to the configuration directory and move the default configuration file:


```
cd /etc/snmp/
sudo mv snmpd.conf snmpd.conf.bak
```


Now, we can create a new, simplified configuration file as root:


```
sudo nano snmpd.conf
```


Copy and paste the following line into the configuration file:


```
rocommunity public
```


Save and close the file.


Now that we have configured the SNMP daemon, we need to restart the service to implement our changes:


```
sudo service snmpd restart
```


The client will now respond to polling requests.


## SSH Client Configuration


For the other client, we will allow Zenoss to run information gathering commands remotely via SSH.


We do the configuration for this on the Zenoss machine, not the SSH client machine.


Begin by logging into the zenoss user and creating an RSA key:


```
su - zenoss
ssh-keygen -t rsa
```


Press "enter" to accept the defaults and use no passphrase.


Next, we will copy the SSH key to our SSH client computer.  Change the username and IP address to reflect your SSH client machine's configuration:


```
ssh-copy-id username@SSH.Client.IP.Address
```


You will be asked to authenticate with the remote machine via password, and then it will add your key to the remote server.


Test your ability to log in without a password by typing:


```
ssh username@SSH.Client.IP.Address
```


If you are successful, type "exit" to get back to the Zenoss machine:


```
exit
```


Type "exit" again to get back to the root shell:


```
exit
```


# Configuring Zenoss


Almost all of the Zenoss configuration is performed in application's web-based frontend.  Open your browser and navigate to:


```
Your.Zenoss.IP.Address:8080
```


When you access it for the first time, you will see the Zenoss Setup page.


![](https://assets.digitalocean.com/tutorial_images/XvbcjHR.png?1)
Click on "Get Started!" to continue.	You will be taken to the "Set Up Initial Users" page.


Select a secure password for the "admin" account, which is used to perform administrative tasks.  Also, add a regular user name and password to use for normal operations.


![](https://assets.digitalocean.com/tutorial_images/s2Q0BeF.png?1)
Click the "Next" button to continue.  You will be taken to the "Specify or Discover Devices to Monitor" page.


Here, you will add your SNMP Client VPS.  Type its IP address into the "Hostnames/IP Addresses" field.  Leave the Device type as Linux Server (SNMP) and click "Save".  Click "Finish or Skip to Dashboard".


![](https://assets.digitalocean.com/tutorial_images/HmG5mai.png?1)
You should see the Zenoss Core Dashboard.  Click on "Infrastructure" at the top.  You will be taken to the "Devices" page.


## Adding the SSH Client


We will be adding the SSH client here. Click on the icon that looks like a computer monitor with a "plus" in the middle.  Choose "Add a Single Device".


![](https://assets.digitalocean.com/tutorial_images/BZhSyFf.png?1)
Type in the IP address of your SSH Client machine, and choose a name to identify it in the "Title" field.


Choose "/Server/SSH/Linux" for the Device Class, and uncheck the Model Device checkbox.


![](https://assets.digitalocean.com/tutorial_images/CfhHDTK.png?1)
Click "Add" at the bottom.


Refresh the page so that your SSH Client machine shows up.  Click the machine name to open the Device overview.


On the left-hand side, click on "Configuration Properties".


![](https://assets.digitalocean.com/tutorial_images/FF7Ev9l.png)
Search for the "zCommandUsername" property and double-click on the "value" column.  Enter the username that you use to ssh into the SSH Client VPS.


![](https://assets.digitalocean.com/tutorial_images/0Xbp9WG.png?1)
Search for the "zKeyPath" property and double-click on the "value" column.  Enter the full pathname to your RSA key.  If you have been following the tutorial, it would be:


```
/home/zenoss/.ssh/id_rsa
```


![](https://assets.digitalocean.com/tutorial_images/0iGQG1L.png?1)
In the bottom of the window, click on the gear icon and then select "Model Device".


![](https://assets.digitalocean.com/tutorial_images/r8O5ECG.png?1)
A window will pop up and show you the log information of the modeling commands that are being run.


![](https://assets.digitalocean.com/tutorial_images/gG6SSXy.png?1)
Click on the "X" in the upper-right corner when it is finished.


## Configuring Localhost


The configuration for localhost does not work correctly by default.  This means that your Zenoss VPS is not being modeled properly.


To fix the SNMP polling, log into your machine as root.  Move into the SNMP configuration directory and move the default snmp daemon configuration to a safe location.


```
cd /etc/snmp/
mv snmpd.conf snmpd.conf.bak
```


Now create a simple snmpd.conf file, just like you did for the SNMP client machine earlier.


```
nano snmpd.conf
```


```
rocommunity public
```


Now restart the service:


```
service snmpd restart
```


Back in the web interface, click on "Infrastructure" and then "Devices".  Click on the "localhost" link to open its configuration.


Now click the gear icon in the lower-left corner.  Select "Reset/Change IP Address".


![](https://assets.digitalocean.com/tutorial_images/avGcvLK.png?1)
In the dialog box that appears, type "127.0.0.1" to use the loopback network device.


![](https://assets.digitalocean.com/tutorial_images/CldZvAR.png?1)
Re-click on the gear icon and select "Model Device" to correct the previous problem.  Click the "X" when the log information is done.


You may have orange triangle alerts from earlier (either an SNMP alert for localhost or a "zCommandUsername" alert for the SSH client). You can clear them by going to "Events", selecting the alerts, and clicking the "X" button to close the events.


If the events do not re-occur, everything is configured correctly.


# Viewing the Results


Your Zenoss server should now be monitoring all three VPS instances.


Click on the "Reports" link at the top and click through the reports as they are generated.


![](https://assets.digitalocean.com/tutorial_images/jdF6GbX.png?1)
You may have to click the "Generate" button on a few of the report options.  As with all monitoring software, these will become more interesting the longer they run.


By Justin Ellingwood
