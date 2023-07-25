# Installing the Cacti Server Monitor on Ubuntu 12 04 Cloud Server

```Apache``` ```Ubuntu``` ```Monitoring``` ```Server Optimization```











# Status: Deprecated


This article covers a version of Ubuntu that is no longer supported.  If you are currently operate a server running Ubuntu 12.04, we highly recommend upgrading or migrating to a supported version of Ubuntu:


- Upgrade to Ubuntu 14.04.
- Upgrade from Ubuntu 14.04 to Ubuntu 16.04
- Migrate the server data to a supported version

Reason:
Ubuntu 12.04 reached end of life (EOL) on April 28, 2017 and no longer receives security patches or updates.  This guide is no longer maintained.


See Instead:
This guide might still be useful as a reference, but may not work on other Ubuntu releases.  If available, we strongly recommend using a guide written for the version of Ubuntu you are using.   You can use the search functionality at the top of the page to find a more recent version.


## What the Red  Means


The lines that the user needs to enter or customize will be in red in this tutorial!


The rest should mostly be copy-and-pastable.


## Introduction


Cacti is a network monitoring tool that creates customized graphs of server performance.  It is accessed and managed through a web front-end.  Cacti can be used to log and graph multiple cloud servers from a single, unified interface.


## Table of Contents


1. Installation
2. SNMPD Configuration
3. Web Configuration
One-Time Setup
General Configuration

4. One-Time Setup
5. General Configuration
6. Creating Devices and Graphs
Device Settings
Graph Settings

7. Device Settings
8. Graph Settings

# Installation


Cacti and all of its dependencies can by installed through apt-get on Ubuntu 12.04.  This guide will also install cacti-spine, which is a faster way to poll servers for information than the default php script.


```
sudo apt-get update
sudo apt-get install snmpd cacti cacti-spine
```


The snmpd daemon should be installed and configured on each cloud server you would like to graph.  In this guide, we will only be graphing the VPS where cacti is installed.  The configuration of the snmpd daemon will happen later in the article.


This installation will pull in quite a few packages that require user-intervention.


If you have not set up MySQL, you will be prompted for a root user password.  Make your selection and confirm the password to continue.


```
  ?????????????????????? Configuring mysql-server-5.5 ???????????????????????
  ? While not mandatory, it is highly recommended that you set a password   ? 
  ? for the MySQL administrative "root" user.                               ? 
  ?                                                                         ? 
  ? If this field is left blank, the password will not be changed.          ? 
  ?                                                                         ? 
  ? New password for the MySQL "root" user:                                 ? 
  ?                                                                         ? 
  ? _______________________________________________________________________ ? 
  ?                                                                         ? 
  ?                                                                     ? 
  ?                                                                         ? 
  ??????????????????????????????????????????????????????????????????????????? 
```


Next, press “Return” or “Enter” to acknowledge a configuration change in php.


```
     ?????????????????????? Configuring libphp-adodb ??????????????????????
     ?                                                                    ? 
     ? WARNING: include path for php has changed!                         ? 
     ?                                                                    ? 
     ? libphp-adodb is no longer installed in /usr/share/adodb. New       ? 
     ? installation path is now /usr/share/php/adodb.                     ? 
     ?                                                                    ? 
     ? Please update your php.ini file. Maybe you must also change your   ? 
     ? web-server configuraton.                                           ? 
     ?                                                                    ? 
     ?                                                                ? 
     ?                                                                    ? 
     ?????????????????????????????????????????????????????????????????????? 
```


The initial configuration of Cacti also happens during installation.  There are a few questions you need to answer.  Select “Apache2” from the list of web servers.


```
 ????????????????????????????? Configuring cacti ?????????????????????????????
 ? Please select the webserver type for which cacti should be automatically  ? 
 ? configured.                                                               ? 
 ?                                                                           ? 
 ? Select "None/Others" if you would like to configure your webserver by     ? 
 ? hand.                                                                     ? 
 ?                                                                           ? 
 ? Webserver type                                                            ? 
 ?                                                                           ? 
 ?                                Apache2                                    ? 
 ?                                Lighttpd                                   ? 
 ?                                None/Others                                ? 
 ?                                                                           ? 
 ?                                                                           ? 
 ?                                                                       ? 
 ?                                                                           ? 
 ????????????????????????????????????????????????????????????????????????????? 
```


After Cacti configures apache, the installation sets up a MySQL account for the application.  Select 
“Yes” to allow a generic database configuration.


```
 ????????????????????????????? Configuring cacti ?????????????????????????????
 ?                                                                           ? 
 ? The cacti package must have a database installed and configured before    ? 
 ? it can be used.  This can be optionally handled with dbconfig-common.     ? 
 ?                                                                           ? 
 ? If you are an advanced database administrator and know that you want to   ? 
 ? perform this configuration manually, or if your database has already      ? 
 ? been installed and configured, you should refuse this option.  Details    ? 
 ? on what needs to be done should most likely be provided in                ? 
 ? /usr/share/doc/cacti.                                                     ? 
 ?                                                                           ? 
 ? Otherwise, you should probably choose this option.                        ? 
 ?                                                                           ? 
 ? Configure database for cacti with dbconfig-common?                        ? 
 ?                                                                           ? 
 ?                                                                  ? 
 ?                                                                           ? 
 ????????????????????????????????????????????????????????????????????????????? 
```


Provide the password for the administration of the Cacti database that you set up during the MySQL configuration.


```
  ???????????????????????????? Configuring cacti ????????????????????????????
  ? Please provide the password for the administrative account with which   ? 
  ? this package should create its MySQL database and user.                 ? 
  ?                                                                         ? 
  ? Password of the database's administrative user:                         ? 
  ?                                                                         ? 
  ? _______________________________________________________________________ ? 
  ?                                                                         ? 
  ?                                                             ? 
  ?                                                                         ? 
  ??????????????????????????????????????????????????????????????????????????? 
```


Next, it asks for a password for Cacti to use with the database.  This is an internal password that you should not ever have to use, so it is okay if you just press “Enter” to create a random password.


```
    ?????????????????????????? Configuring cacti ??????????????????????????
    ? Please provide a password for cacti to register with the database   ? 
    ? server.  If left blank, a random password will be generated.        ? 
    ?                                                                     ? 
    ? MySQL application password for cacti:                               ? 
    ?                                                                     ? 
    ? ___________________________________________________________________ ? 
    ?                                                                     ? 
    ?                                                         ? 
    ?                                                                     ? 
    ??????????????????????????????????????????????????????????????????????? 
```


The installation should complete as expected.


# SNMPD Configuration


The snmpd daemon must be configured to work with Cacti.  The configuration file is located at “/etc/snmp/snmpd.conf”.  Make sure you are editing the snmpd.conf file and not the snmp.conf file.


```
sudo nano /etc/snmp/snmpd.conf
```


First, edit the Agent Behavior, which should be located near the top of the file.  Comment out the line for "connections from the local system only" and uncomment the line for listening for "connections on all interfaces".


```
#  Listen for connections from the local system only
#agentAddress  udp:127.0.0.1:161
#  Listen for connections on all interfaces (both IPv4 *and* IPv6)
agentAddress udp:161,udp6:[::1]:161
```


Next, search for and find the ACCESS CONTROL section.  Uncomment and edit the line for “rocommunity secret 10.0.0.0/16”.  We will be changing this to reference our specific Cacti server.  Use either your cloud server's domain name or its IP address.


```
rocommunity secret  CactiServerIpAddress
```


You can find the IP address of your VPS by typing this command.


```
ifconfig eth0 | grep inet | awk '{ print $2 }'
```


You may also want to edit the system information that will be associated with your data in the SYSTEM INFORMATION section.  You can add the physical location of your server and a contact email.  These may be helpful for distinguishing machines if you are monitoring a large number of cloud servers.


```
sysLocation    Your System Location
sysContact     contact@email.com
```


After you are done with your modifications, save the file, exit and restart the snmpd service.


```
sudo service snmpd restart
```


# Web Configuration


## One-Time Setup


The rest of the configuration will be done through a web browser.  Open your web browser and navigate to your server ip address or domain name with “/cacti” on the end.


```
mydomain.com/cacti
```


The first page you will see is an introduction to the Cacti software.  Click “Next >>” when you are finished reading.  Click "Next >>" again on the following page since this is a new installation.


The next page shows the application paths of the “helper” applications that Cacti uses to operate.  All of the applications should be green and marked with “[FOUND]”.  Click “Finish” to continue.


![](https://assets.digitalocean.com/tutorial_images/q249p3z.png?1)
Next, you’ll be asked to enter the Cacti user name and password.  These are not the passwords you entered during installation.  Instead, enter the following default values.


```
User Name: admin
Password: admin
```


You’ll be prompted to enter a new password for administrating Cacti.  Choose a password and click “Save”.


You are now on your Cacti page.


![](https://assets.digitalocean.com/tutorial_images/Ei4eNqy.png?1)
## General Configuration


A few options must be changed to ensure that Cacti produces data correctly.  On the left-hand navigation panel, click on “Settings” under the Configuration heading.


In the General tab, we want to change some parameters.  Change these settings to match what is shown here.  Click “Save” when finished.


```
SNMP Version: Version 2
SNMP Community: secret
```


![](https://assets.digitalocean.com/tutorial_images/D3y79Ti.png?1)
Next, click the “Poller” tab on the navigation settings.  Change these options and match what is shown here.  Click “Save” when finished.


```
Poller Type: spine
Poller Interval: Every Minute
```


![](https://assets.digitalocean.com/tutorial_images/CfblVxl.png?1)
Whenever the Poller Interval is changed, the cache must be emptied.  To do this, click “System Utilities” under the Utilities heading on the left-hand navigation panel.


Click on “Rebuild Poller Cache” to empty the cache.


# Creating Devices and Graphs


## Device Settings


To begin graphing, we need to set up device profiles and tell Cacti what to graph.  Click “Devices” under the Management heading on the left-hand navigation panel.


First, delete the “Localhost” device because we will be recreating some of the same functionality in the device we will be setting up momentarily.  Click the checkbox on the right-hand side, make sure Choose an action has “Delete” selected, and click “Go”.  Confirm the delete on the following page.


In the upper-right corner of the page, click the “Add” button to add a new device.


Now, you need to fill out some information that describes your device.  Fill out the following fields.  Click “Create” when you are finished.


```
Description: Ubuntu Cacti Server
Hostname: YourIPAddress
Host Template: Local Linux Machine
SNMP Version: Version 2
SNMP Community: secret
```


![](https://assets.digitalocean.com/tutorial_images/OkWbJPs.png?1)
If there is an SNMP error in red at the top of the page,  open up a terminal on your cloud server and restart the snmpd daemon.  Click “Save” again and it should now populate correctly.


```
sudo service snmpd restart
```


## Graph Settings


Next, scroll down and create some associated graph templates and associated data queries.  Under Associated Graph Templates, select “Unix – Ping Latency” from the drop-down and click “Add”.  Your selection should match what’s shown below.


![](https://assets.digitalocean.com/tutorial_images/0Nsx7AP.png?1)
Complete the same steps in the Associated Data Queries section to add “SNMP – Get Mounted Partitions”, “SNMP – Get Processor Information”, and “SNMP – Interface Statistics”.  Add each of those and then click “Save”.


![](https://assets.digitalocean.com/tutorial_images/HoTdtlR.png?1)
Next, click “Create Graphs for this Host” at the top-right of the page.


Select each of the right-hand boxes in the light-blue subheadings to select all of the graphs.  Click “Create” at the bottom of the page.


On the next page, you can change the color of some of the graphing choices.  Make your selections and then click “Create”.


At the top of the page, click “Graphs” tab.  Click on the last tab in the top-right corner.  It should look like a graph.


![](https://assets.digitalocean.com/tutorial_images/VVEb7OV.png?1)
Your VPS will take a while to generate values for these graphs.  It might be five or ten minutes before you even see an empty graph.  Sometimes, it will appear that there is a broken image until there is enough data to graph.  If you come back in a few hours, you will have some colorful graphs showing some important system statistics.


![](https://assets.digitalocean.com/tutorial_images/eM6eV68.png?1)
Click on each graph to show daily, weekly, monthly, and yearly graphs for that same resource.


![](https://assets.digitalocean.com/tutorial_images/g0AxgeN.png?1)
Now you have access to Cacti's graphing capabilities.  Cacti becomes more useful with every new cloud server you tell it to monitor, so explore the possibility of adding more servers as Cacti devices.


By Justin Ellingwood
