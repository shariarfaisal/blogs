# How to Install Linux Socket Monitor  LSM  on CentOS 6 4

```Security``` ```Monitoring``` ```System Tools``` ```CentOS```


Status: Deprecated
This article covers a version of CentOS that is no longer supported. If you are currently operating a server running CentOS 6, we highly recommend upgrading or migrating to a supported version of CentOS.
Reason:
CentOS 6 reached end of life (EOL) on November 30th, 2020 and no longer receives security patches or updates. For this reason, this guide is no longer maintained.
See Instead:
This guide might still be useful as a reference, but may not work on other CentOS releases. If available, we strongly recommend using a guide written for the version of CentOS you are using.

## Introduction



In a very broad sense, sockets are used by applications for streaming data between each other. When it comes to communicating over distance via networks (whether internal or external), principals applied for the task remain the same and sockets are used to exchange information.


In this DigitalOcean article, we will be looking at enhancing system security by increasing the level of monitoring performed. We are going to set up Linux Socket Monitor which sends out alerts when a new socket is created, which is a potential signal of an intrusion by an unknown application taking over its host.


Understanding Sockets, Ports and Connections


Applications designed to run on networks (e.g. the internet) need to bind themselves to certain ports depending on their role, in order to do their job of exchanging information over distance through data connections.


When an application is launched successfully (i.e. a web server running on port 80), it gets a socket bound to this port number, identified [basically] by the network address (i.e. the IP address), the protocol (i.e. the language of information exchange) used and the port number (e.g. 80), which listens for incoming connection requests.


Clients on the other hand, at the other side of the line, consist of applications (e.g. Firefox Web Browser) which make the requests in order to start communicating or [afterwards] sending and receiving data to/from these applications.


Finally, requests, made by clients, upon reaching its target application (i.e. the web server), go through certain procedures depending on the protocol in use (and rules set). If the server decides to accept it, a connection gets established between these two parties through sockets --and actually a new one, to which both the client and the server bind individually, so that the server can still keep listening to other incoming requests at the same time.


Understanding How To Identify Possible Threats


Sockets are bidirectional instances, used for both sending and receiving data between parties sharing the established connection. If (or when) there is a new port opened or an unexpected socket created, it means that there is an application ready to listen and pick up commands through incoming requests for execution on the host it resides, depending on permissions it has. And when this happens by an unwanted or infiltrated application, it is vital to quickly deal with it, which can only happen with rapid detection via socket monitoring.


# Linux Socket Monitor: The Application



## What is Linux Socket Monitor?



Linux Socket Monitor (LSM) is a monitoring tool which tracks changes to ports and sockets (both network and inter-process (IPC) ones used between applications on the same machine) by comparing snapshots it takes - either automatically (upon installation) or by your direction.


It uses the utility tool cron to schedule its scans at certain intervals --by default, once every 10 minutes-- and checks the ports/sockets in use. If it notices any differences, by sending an email alert, it lets you know.


## How does Linux Socket Monitor work?



Linux Socket Monitor is actually a very smart and compact bash script (program). It requires two commands to download, install and get started with monitoring. When you first install the tool, it scans the current ports and sockets to create the base comparison file --a snapshot of your network. It requires only a little configuring, whereby you enter your email address to have the alerts sent, and that’s it!


## How to install Linux Socket Monitor?



The latest version of LSM is located on its developer’s website located at: http://www.rfxn.com/downloads/lsm-current.tar.gz


In order to download the tape archive (tar, tarball), run the following:


```
 wget http://www.rfxn.com/downloads/lsm-current.tar.gz

```


This will download the archive to the current folder you have.


Let’s extract the contents from the tarball:


```
 tar -xvfz lsm-current.tar.gz

```


We are now ready to install LSM by running its installation script.


Enter the directory and run the installation:


```
$ cd lsm-0.6
$ ./install.sh

```


When the quick installation completes, you will be presented a summary of the application itself alongside the notification that the base comparison files have been generated, similar to the following:


```
.: LSM installed
Install path:    /usr/local/lsm
Config path:     /usr/local/lsm/conf.lsm
Executable path: /usr/local/sbin/lsm
LSM version 0.6 <lsm@r-fx.org>
Copyright (C) 2004, R-fx Networks
              2004, Ryan MacDonald
This program may be freely redistributed under the terms of the GNU GPL

generated base comparison files

```


We can now complete the set up by modifying the configuration file.


Open up the LSM configuration file using nano text editor:


```
 nano /usr/local/lsm/conf.lsm

```


Here you will see a relatively long list of values which are used by LSM to operate. The one that we need to modify is the third one on the list: USER="root" which is after the commented out sections located on top.


Using your arrow keys, go down to that line and replace root with your email address.


Example:


```
USER="system.alerts@mydomain.com"             # ..

```


You could also modify the following line on the document if you wanted to receive the alerts (i.d. emails) with a different subject line than the default.


In order to change the subject line of emails, please modify:


```
SUB="LSM Port Monitor Alert on $HOSTNAME"     # ..

```


to a subject of your liking. Make sure to keep $HOSTNAME as is, which will contain your machine’s hostname --a helpful aspect when you are dealing with more than one.



If you are unsure about how to set your machine’s host name, you can refer to this excellent article by Etel Sverdlov on DigitalOcean Help & Community section.

And we are good to go!


## Using the Linux Socket Monitor



Right after the installation, LSM already has taken a snapshot of your current ports and sockets, and set up a cron job to run once every 10 minutes.


At any given moment, you can delete or recreate the comparison files via two simple commands:


- Delete snapshots (camparison files): /usr/local/sbin/lsm -d
- Manually run a comparison test: /usr/local/sbin/lsm -c

And to recreate the snapshots:


- Generate base comparison files: /usr/local/sbin/lsm -g

Please Note: Sending emails from your system requires you to have a mail user agent application installed. There a various options you can choose from if you do not have one already. The follow up article will cover installing an agent.


<div class=“author”>Submitted by: <a href=“https://twitter.com/ostezer”>O.S. Tezer</div>


