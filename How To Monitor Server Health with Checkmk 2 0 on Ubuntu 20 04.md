# How To Monitor Server Health with Checkmk 2 0 on Ubuntu 20 04

```Monitoring``` ```Open Source``` ```Ubuntu```

The author selected the Open Internet/Free Speech Fund to receive a donation as part of the Write for DOnations program.


## Introduction


As a systems administrator, it’s best to know the current state of your infrastructure and services. Ideally, you want to notice failing disks or application downtimes before your users do. Monitoring tools like Checkmk can help administrators detect these issues and maintain healthy servers.


Generally, monitoring software can track your servers’ hardware, uptime, and service statuses, and it can raise alerts when something goes wrong. In a basic scenario, a monitoring system would alert you if any services go down. In a more robust one, the notifications would come after any suspicious signs arose, such as increased memory usage or an abnormal amount of TCP connections.


Many monitoring solutions offer varying degrees of complexity and feature sets, both free and commercial. In many cases, installing, configuring, and managing these tools is difficult and time-consuming.


Checkmk is a monitoring solution that is both robust and simple to install. It is a self-contained software bundle that combines Nagios (a popular open-source alerting service) with add-ons for gathering, monitoring, and graphing data. Checkmk also comes with a web interface — a comprehensive tool that complements Nagios. It offers a dashboard, a notification system, and a repository of monitoring agents for many Linux distributions. Without for Checkmk’s web interface, you would have to use different views for various tasks. It wouldn’t be possible to configure all these features without resorting to extensive file modifications.


In this guide, you will set up Checkmk on an Ubuntu 20.04 server and monitor two separate hosts. You will monitor the Ubuntu server and a separate CentOS 8 server, but you could use the same approach to add any number of additional hosts to the monitoring configuration.


# Prerequisites


- One Ubuntu 20.04 server with a regular, non-root user with sudo privileges. You can prepare your server by following this initial server setup tutorial.
- One CentOS 8 server with a regular, non-root user with sudo privileges. To prepare this server, you can follow this initial server setup tutorial.

# Step 1 — Installing Checkmk on Ubuntu


To use the monitoring site, first, you must install Checkmk on the Ubuntu server. This installation will give you all the needed tools. Checkmk provides official ready-to-use Ubuntu package files that you can use to install the software bundle.


First, update the packages list so that you have the most recent version of the repository listings:


```
sudo apt update


```


To browse the packages, go to the package listing site. Ubuntu 20.04, among others, can be selected in the page menu.


Now download the package:


```
wget https://download.checkmk.com/checkmk/2.0.0p18/check-mk-raw-2.0.0p18_0.focal_amd64.deb


```


Then install the newly downloaded package:


```
sudo apt install -y ./check-mk-raw-2.0.0p18_0.focal_amd64.deb


```


This command will install the Checkmk package and all necessary dependencies, including the Apache web server used to provide web access to the monitoring interface.


After the installation completes, you can now access the omd command. Try it out:


```
sudo omd


```


This omd command will output the following:


```
OutputUsage (called as root):

 omd help                        Show general help

...

General Options:
 -V <version>                    set specific version, useful in combination with update/create
 omd COMMAND -h, --help          show available options of COMMAND

```


The omd command can manage all Checkmk sites on your server. It can start and stop all the monitoring services at once, and you will use it to create Checkmk sites. First, however, you have to update the firewall settings to allow outside access to the default web ports.


# Step 2 — Adjusting the Firewall Settings


Before you can work with Checkmk, it’s necessary to allow outside access to the web server in the firewall configuration. If you followed the firewall configuration steps in the prerequisites, you’ll have a UFW firewall set up to restrict access to your server.


During installation, Apache registers itself with UFW to provide an easy way to enable or disable access to Apache through the firewall.


To allow access to Apache, use the following command:


```
sudo ufw allow Apache


```


Now verify the changes:


```
sudo ufw status


```


You’ll see that Apache is listed among the allowed services:


```
OutputStatus: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
Apache                     ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
Apache (v6)                ALLOW       Anywhere (v6)

```


This will allow you to access the Checkmk web interface.


In the next step, you’ll create the first Checkmk monitoring site.


# Step 3 — Creating a Checkmk Monitoring Site


Checkmk uses the concept of sites, or individual installations, to isolate multiple Checkmk copies on a server. In this step, you’ll set up your initial site. In most cases, only one copy of Checkmk is enough, and that’s how you will configure the software in this guide.


First, give the new site a name; in this example, it’ll be monitoring. To create the site, type:


```
sudo omd create monitoring


```


The omd tool will set up everything automatically. The command output will look similar to the following:


```
OutputAdding /opt/omd/sites/monitoring/tmp to /etc/fstab.
Creating temporary filesystem /omd/sites/monitoring/tmp...OK
Updating core configuration...
Generating configuration for core (type nagios)...Precompiling host checks...OK
OK
Executing post-create script "01_create-sample-config.py"...OK
Restarting Apache...OK
Created new site monitoring with version 2.0.0p18.cre.

  The site can be started with omd start monitoring.
  The default web UI is available at http://your_ubuntu_server/monitoring/

  The admin user for the web applications is cmkadmin with password: your-default-password
  For command line administration of the site, log in with 'omd su monitoring'.
  After logging in, you can change the password for cmkadmin with 'htpasswd etc/htpasswd cmkadmin'.

```


The URL address, default username, and password for accessing the monitoring interface are highlighted in this output. The site is now created, but it still needs to be started. To start the site, type:


```
sudo omd start monitoring


```


Now all the necessary tools and services will be started at once. At the end, you’ll see an output verifying that all services have started successfully:


```
OutputTemporary filesystem already mounted
Starting mkeventd...OK
Starting rrdcached...OK
Starting npcd...OK
Starting nagios...OK
Starting apache...OK
Starting redis...OK
Initializing Crontab...OK

```


The site is up and running.


To access the Checkmk site, open http://your_ubuntu_server_ip/monitoring/ in the web browser. You will be prompted for a password. Use the default credentials displayed in the previous output; you will change these defaults later on.


The Checkmk screen opens with a dashboard, which shows all services and server statuses in lists, and it uses practical graphs resembling honeycombs. Immediately after installation, these graphs are empty, but you will shortly make them display statuses for services and systems.





The left menu, the main entry point to all Checkmk features, is divided into three main tabs. The Monitor section refers to all activities for daily monitoring work and checking the status of the services. The Customize tab is used to tailor the web interface experience to your liking. The Setup tab holds all the configuration options. Each of the main menu tabs uncovers robust submenus when clicked.


In the next step, you will change the default password to secure the site using this interface.


# Step 4 — Changing Your Administrative Password


Checkmk generates a random password for the cmkadmin administrative user during installation. This password is meant to be changed upon installation, and as such, it is often short and not secure. You can change this via the web interface.


First, open the Users page from the Setup menu on the left. The list will show all users that currently have access to the Checkmk site. On a fresh installation, it will list only two users. The first one, automation, is intended for use with automated tools; the second is the cmkadmin user you used to log in to the site.





Click on the pencil icon next to the cmkadmin user to change its details, including the password.





Update the password, add an admin email, and make any other desired changes.


After saving the changes by clicking the Save button above the form, you will be asked to log in again using the new credentials. Do so and return to the dashboard to fully apply the new configuration.


Next, open the Users page from the Setup menu on the left. The orange warning sign in the top right corner labelled as 1 change tells you that you have made some changes to the configuration of Checkmk, and that you need to save and activate them.


Checkmk initially saves all changes you make in a sandbox configuration environment that does not yet impact the ongoing monitoring. The changes are reflected in the production configuration after you activate them. This will happen every time you change the configuration of your monitoring system, not only after editing a user’s credentials.


To save and activate pending changes, you have to click on the orange warning notification and agree to activate the listed changes by clicking the Activate on selected sites option on the following screen.







After activating the changes, the new user’s data is written to the configuration files and will be used by all the system’s components. Checkmk automatically takes care of notifying individual monitoring system components, reloading them when necessary, and managing all the needed configuration files.


The Checkmk installation is now ready for use. In the next step, you will add the first host to the monitoring system.


# Step 5 — Monitoring the First Host


You are now ready to monitor the first host. To accomplish this, you will first install check-mk-agent on the Ubuntu server. Then, you’ll restrict access to the monitoring data using xinetd.


The components installed with Checkmk are responsible for receiving, storing, and presenting monitoring information. They do not provide the information itself.


To gather the actual data, you will use Checkmk agent. Designed specifically for the job, a Checkmk agent can monitor all vital system components at once and report that information back to the Checkmk instance.


## Installing the Agent


The first host you will monitor will be your_ubuntu_server—the server on which you have installed the Checkmk instance itself.


To begin, you must install the Checkmk agent. Packages for all major distributions, including Ubuntu, are available directly from the web interface. Open the Linux page under the Agents section from the Setup menu on the left. You will see the available agent downloads with the most popular packages under the first section labelled Packaged Agents.





The package check-mk-agent_2.0.0p18-1_all.deb is the one suited for Debian based distributions, including Ubuntu. Copy the download link for that package from the web browser and use that address to download the package:


```
wget http://your_ubuntu_server_ip/monitoring/check_mk/agents/check-mk-agent_2.0.0p18-1_all.deb


```


After downloading, install the package:


```
sudo apt install -y ./check-mk-agent_2.0.0p18-1_all.deb


```


Now verify the agent installation:


```
check_mk_agent


```


The command will output a long text block that looks like gibberish but combines all vital information about the system in one place:


```
Output<<<check_mk>>>
Version: 2.0.0p18
AgentOS: linux
...
monitoring;2.0.0p18.cre;1
<<<local:sep(0)>>>

```


Checkmk uses the output from this command to gather status data from monitored hosts. Now, you’ll restrict access to the monitoring data with xinetd.


## Restricting Access to Monitoring Data Using xinetd


By default, the data from check_mk_agent is served using xinetd, a mechanism that outputs data on a specific network port upon accessing it. This means that you can access the check_mk_agent by using telnet to port 6556 (the default port for Checkmk) from any other computer on the internet unless your firewall configuration disallows it.


It is not a good security policy to publish vital information about servers to anyone on the internet. You should allow only hosts that run Checkmk and are under your supervision to access this data so that only your monitoring system can gather it.


If you have followed the initial server setup, including the steps about setting up a firewall, then access to Checkmk agent is blocked by default. It is, however, a good practice to enforce these access restrictions directly in the service configuration and not rely only on the firewall to guard it.


To restrict access to the agent data, edit the configuration file at /etc/xinetd.d/check_mk. Open the configuration file in your favorite editor. To use nano, type:


```
sudo nano /etc/xinetd.d/check_mk


```


Locate this section:


/etc/xinetd.d/check_mk
```
...
# configure the IP address(es) of your Nagios server here:
#only_from      = 127.0.0.1 10.0.20.1 10.0.20.2
...

```


The only_from setting restricts access to specific IP addresses. Because you are now monitoring the same server that Checkmk is running on, it is okay to allow only localhost to connect. Uncomment and update the configuration setting to:


/etc/xinetd.d/check_mk
```
...
# configure the IP address(es) of your Nagios server here:
only_from      = 127.0.0.1
...

```


Save and exit the file.


The xinetd daemon has to be restarted for changes to take place:


```
sudo systemctl restart xinetd


```


Now the agent is up and running and restricted to accept only local connections. You can proceed to configure monitoring for that host using Checkmk.


## Configuring the Host in the Checkmk Web Interface


To add a new host to monitor, go to the Hosts menu in the Setup menu on the left. From here, click Add host to the monitoring. You will be asked for some information about the host.





The Hostname is the familiar name that Checkmk will use for the monitoring. It may be a fully-qualified domain name, but it is not necessary. In this example, you will name the host monitoring, just like the name of the Checkmk instance itself. Because monitoring is not resolvable to the IP address, you also have to provide the IP address of your server. Since you are monitoring the local host, the IP will simply be 127.0.0.1. Check the IPv4 Address box to enable the manual IP input and enter 127.0.0.1 in the text field.


The default configuration of the Monitoring agents section relies on Checkmk agent to provide monitoring data, which is fine and does not need any changes.


To save the host and configure which services will be monitored, click the Save & go to service configuration button.





Checkmk will do an automatic inventory, which means it will gather the output from the agent and decipher it to know what kinds of services it can monitor. All available services for monitoring will be on the Undecided services (currently not monitored) list, including CPU load, memory usage, and free space on disks.


To enable monitoring of all discovered services, click the Fix all button. This will refresh the page, but now all services will be listed under the Monitored services section, informing you that they are indeed being monitored.


As was the case when changing the user password, these new changes must be saved and activated before they go live. Click the 2 changes notification in the upper right and accept the changes using the Activate on selected sites button. After that, the host monitoring will be up and running.


Now you are ready to work with your server data with the main dashboard.


## Working with Monitoring Data


Take a look at the main dashboard using the Main dashboard item from the Monitor menu on the left:





The honeycomb now has a green border, and the table says that one host is up with no problems. You can see the complete host list, which currently consists of a single host, on the All hosts view from the Monitor menu on the left.





There you will see how many services are in good health (shown in green), how many are failing, and how many are pending to be checked. Click on the hostname, and you will see the list of all services with their full statuses and their Perf-O-Meters. The Perf-O-Meter shows the performance of a single service relative to what Checkmk considers to be good health.





All services that return graphable data display a graph icon next to their name. You can use that icon to access graphs associated with the service. Since the host monitoring is fresh, there is almost nothing on the graphs — but after some time, the charts will provide valuable information on how the service performance changes over time.





When any of these services fails or recovers, the information will be shown on the dashboard. A red error will be shown for failing services, and the problem will also be visible on the honeycomb graph.





After recovery, everything will be shown in green as working correctly, but the event log on the right will contain information about past failures.





Now that you have explored the dashboard a little, you’ll add a second host to your monitoring instance.


# Step 6 — Monitoring a Second CentOS Host


Monitoring becomes more useful when you have multiple hosts. You will now add a second server to your Checkmk instance, running CentOS 8.


As with the Ubuntu server, installing Checkmk agent is necessary to gather monitoring data on CentOS. This time, however, you will need an rpm package from the Linux page in the Monitoring agents section of the Setup tab in the web interface, called check-mk-agent-2.0.0p18-1.noarch.rpm.


First, you must install xinetd, which by default is not available on the CentOS installation. As mentioned previously, xinetd, is a daemon that is responsible for making the monitoring data provided by check_mk_agent available over the network.


On your CentOS server, install xinetd:


```
sudo dnf install -y xinetd


```


Now you can download and install the monitoring agent package needed for the CentOS server:


```
sudo dnf install -y http://your_ubuntu_server_ip/monitoring/check_mk/agents/check-mk-agent-2.0.0p18-1.noarch.rpm


```


Just like before, you can verify that the agent is working properly by executing check_mk_agent:


```
sudo check_mk_agent


```


The output will be similar to that from the Ubuntu server. Now you will restrict access to the agent.


## Restricting Access


This time you will not be monitoring a local host, so xinetd must allow connections coming from the Ubuntu server, where Checkmk is installed, to gather the data. To allow that, open your configuration file:


```
sudo vi /etc/xinetd.d/check_mk


```


Here you will see the configuration for your check_mk service, specifying how Checkmk agent can be accessed through the xinetd daemon. Find the following two commented lines:


/etc/xinetd.d/check_mk
```
...
# configure the IP address(es) of your Nagios server here:
#only_from      = 127.0.0.1 10.0.20.1 10.0.20.2
...

```


Now uncomment the second line and replace the local IP addresses with your_ubuntu_server_ip:


/etc/xinetd.d/check_mk
```
...
# configure the IP address(es) of your Nagios server here:
only_from      = your_ubuntu_server_ip
...

```


Save and exit the file by typing :x and then ENTER.


Restart the xinetd service using:


```
sudo systemctl restart xinetd


```


If you have configured a local firewall following the initial server setup, you also need to adjust the firewall settings. Without doing so, connections to Checkmk agent won’t be allowed. To do so, execute:


```
sudo firewall-cmd --add-port=6556/tcp --permanent


```


This would allow incoming TCP traffic to port 6556, which is used by Checkmk. The configuration will update after you reload the firewall:


```
sudo firewall-cmd --reload


```



Note: You can learn how to fine-tune the firewall settings by following the How To Set Up a Firewall Using firewalld on CentOS 8 guide.

You can now proceed to configure Checkmk to monitor the CentOS 8 host.


## Configuring the New Host in Checkmk


To add additional hosts to Checkmk, you use the Hosts menu from the Setup tab just like before. Click the Add host button to bring up the Add host page. Name the host centos and enter its IP address in the IPv4 Address field. Then, expand the additional settings for the Custom attributes section by clicking the arrow toggle next to the name, and then clicking the Show more link on the right side.


The Networking Segment setting will appear, which was previously invisible. Click the checkbox to reveal the drop-down, and then choose the WAN (high-latency) option since the monitor host is on another network. By default, Checkmk assumes the monitored servers are in the local network and expects them to be accessible with low latency. If you skipped this and left it with the default local value, Checkmk would soon alert you that the host is down, as it would expect the host to respond to agent queries much quicker than is possible over the internet.





Click Save & go to service configuration, which will show services available for monitoring on the CentOS server. The list will be similar to the one from the first host. Again, you also must click Fix all and then activate the changes using the orange notification in the top right corner.


After activating the changes, you can verify that the host is monitored on the All hosts page. Two hosts, monitoring and centos, will now be visible.





You are now monitoring an Ubuntu server and a CentOS server with Checkmk. It is possible to monitor even more hosts. There is no upper limit other than server performance, which should not be a problem until your number of hosts is in the hundreds. Moreover, the procedure is the same for any other host. Checkmk agents in deb and rpm packages work on Ubuntu, CentOS, and most other Linux distributions.


# Conclusion


In this guide, you’ve set up two servers with two different Linux distributions: Ubuntu and CentOS. You then installed and configured Checkmk to monitor both servers and explored Checkmk’s web interface.


Checkmk allows for the setup of a complete monitoring system, which moves the process of manual configuration to a web interface with various options and features. With these tools, you can monitor multiple hosts; set up email, SMS, or push notifications for problems; set up additional checks for more services; monitor accessibility and performance; and more.


To learn more about Checkmk, visit the official documentation.


