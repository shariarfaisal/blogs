# How To Install MongoDB on CentOS 8

```Databases``` ```CentOS 8``` ```CentOS``` ```MongoDB``` ```NoSQL```

An earlier version of this tutorial was written by Melissa Anderson.


## Introduction


MongoDB, also known as Mongo, is an open-source document database used in many modern web applications.  It’s classified as a NoSQL database because it doesn’t rely on a traditional table-based relational database structure.


Instead, it uses JSON-like documents with dynamic schemas, meaning that, unlike relational databases, MongoDB does not require a predefined schema before you add data to a database.  You can alter the schema at any time and as often as is necessary without having to set up a new database with an updated schema.


In this tutorial you’ll install MongoDB on a CentOS 8 server, test it, and learn how to manage it as a systemd service.


# Prerequisites


To complete this tutorial, you will need a server running CentOS 8.  This server should have a non-root user with administrative privileges and a firewall configured with firewalld.  To set this up, follow our Initial Server Setup guide for CentOS 8.


# Step 1 — Installing MongoDB


There’s no official MongoDB package available in the standard CentOS repositories.  In order to install Mongo on your server, you’ll need to add a repository file that points to MongoDB’s official repo.  Your package manager will then read this file when searching for packages, and will be able to use it to install Mongo and any of its dependencies.


In this guide, we’ll install Mongo with the DNF package manager, so you’ll need to add the repository file to the /etc/yum.repos.d/ directory.  DNF checks any files in this directory that end with the .repo suffix when searching for package sources.


You can use vi — a widely-used text editor that’s installed on CentOS systems by default — to create the repository file, but vi can be somewhat unintuitive for users who aren’t experienced with it.  As an alternative, we recommend nano, a more user-friendly editor available from the standard CentOS repositories.


To install nano with DNF, run the following command:


```
sudo dnf install nano


```


During this installation process, the system will ask you to confirm that you want to install the software. To do so, press y, and then ENTER:


```
OutputTransaction Summary
================================================================================
Install  1 Package

Total download size: 581 k
Installed size: 2.2 M
Is this ok [y/N]: y

```


Once the installation is done, run the following command to create and open a repository file for editing:


```
sudo nano /etc/yum.repos.d/mongodb-org.repo


```


Then add the following content to the empty file.  This will install version 4.4 of MongoDB (the latest version at the time of this writing):


/etc/yum.repos.d/mongodb-org.repo
```
[mongodb-org]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/4.4/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-4.4.asc

```



Note: You can check whether there’s a newer version of MongoDB available by consulting the database’s official documentation.  In a web browser, navigate to the Configure the package management system section of MongoDB’s RedHat and CentOS installation instructions.
There, you’ll find a code block with the repository information for the latest version of MongoDB. If different from the previous file contents, you can copy that configuration and add it to your .repo file instead.

Here’s what each of these directives do:


- [mongodb-org]: the first line of a .repo file is a single string of characters, wrapped in brackets, that serves as an identifier for the repository
- name: this directive defines a human-readable name to describe the repository.  You could enter whatever name you’d like here, but for clarity you can enter MongoDB Repository
- baseurl: this points to the URL of a directory where the repository’s repodata directory, which contains the repository’s metadata, can be found
- gpgcheck: setting this directive to 1 tells the DNF package manager to enable GPG signature-checking on this repository, requiring it to verify whether any packages you want to install from it have been corrupted or tampered with
- enabled: setting this directive to 1 will tell DNF to include this repository as a package source; setting it to 0 would disable this behavior
- gpgkey: this directive specifies the URL of the GPG key that should be imported to verify the signatures of packages from this repository

After adding the repository information, save and close the file.  If you used nano to create the repo file, do so by pressing CTRL + X, Y, then ENTER.


Before continuing, you can test whether DNF is able to find and use this repository by running the program’s repolist command:


```
dnf repolist


```


If the repository is available to your server’s package manager, you’ll find it listed in the output:


```
Outputrepo id                                                                           repo name
AppStream                                                                         CentOS-8 - AppStream
BaseOS                                                                            CentOS-8 - Base
extras                                                                            CentOS-8 - Extras
mongodb-org                                                                       MongoDB Repository

```


Following that, you can install the mongodb-org package with this command:


```
sudo dnf install mongodb-org


```


Again, you’ll be asked to confirm that you want to install the package by pressing y then ENTER.  DNF may also ask you to confirm the import of Mongo’s signing key; if this is the case, do so by once more pressing y and then ENTER.


Once the command finishes, MongoDB will be installed on your server. Before you start up the database, though, the MongoDB documentation recommends that you disable Transparent Huge Pages on your server to optimize performance.


# Step 2 — Disabling Transparent Huge Pages to Improve Performance


CentOS enables Transparent Huge Pages (THP), a Linux memory management system, by default. THP uses extra large memory pages to reduce the impact of Translation Lookaside Buffer lookups on machines with large amounts of memory. However, this system can have a negative impact on database performance and the MongoDB documentation recommends that you disable THP.


To disable Transparent Huge Pages on CentOS 8, you can create a systemd unit file that will disable it at boot. systemd is an init system and software suite used in many Linux operating systems with which you can control many aspects of a server. In systemd, a unit refers to any resource that the system knows how to operate on and manage. A unit file is a special configuration file that defines one of these resources.


Create this file in the /etc/systemd/system/ directory with sudo privileges:


```
sudo nano /etc/systemd/system/disable-thp.service


```


In the file, you’ll need to add a few separate sections. The first you’ll add is a [Unit] section which holds some general information about the service unit.


Add the following highlighted lines:


/etc/systemd/system/disable-thp.service
```
[Unit]
Description=Disable Transparent Huge Pages (THP)
After=sysinit.target local-fs.target
Before=mongod.service

```


After the [Unit] section header are the following options:


- Description: this defines a human-readable name for the unit
- After: this ensures that the disable-thp service unit will start up only after the two specified targets  — predefined groups of systemd units — finish starting up
- Before: this option ensures that the disable-thp service will always finish starting up before mongod, the MongoDB service unit, starts up

Next, add the highlighted [Service] section to the file:


/etc/systemd/system/disable-thp.service
```
. . .
Before=mongod.service

[Service]
Type=oneshot
ExecStart=/bin/sh -c 'echo never | tee /sys/kernel/mm/transparent_hugepage/enabled > /dev/null' 

```


This section’s Type option defines that this unit will be a oneshot type process, meaning that it will be a one-off task and systemd should wait for the process to exit before continuing.


The ExecStart option is used to specify the full path and any arguments of the command that will start the process.  The command included here will open up a shell process and then run the command between the single quotes.  This command echoes the string never and then pipes it into the following tee command.  The tee command then rewrites the /sys/kernel/mm/transparent_hugepage/enabled file with never as its only contents and passes any output to /dev/null, a null device which immediately discards any information written to it.  This is what will actually disable THP.


Following that, add this highlighted [Install] section:


/etc/systemd/system/disable-thp.service
```
. . .
ExecStart=/bin/sh -c 'echo never | tee /sys/kernel/mm/transparent_hugepage/enabled > /dev/null'

[Install]
WantedBy=basic.target

```


[Install] sections carry installation information for the unit.  This section only has one option, WantedBy, which will cause the disable-thp unit to start when basic.target is started.



Note: If you’d like to learn more about systemd units and services, we encourage you to check out our guide on Understanding Systemd Units and Unit Files.

The entire service unit file should look like this when you’ve finished adding all the lines:


/etc/systemd/system/disable-thp.service
```
[Unit]
Description=Disable Transparent Huge Pages (THP)
After=sysinit.target local-fs.target
Before=mongod.service

[Service]
Type=oneshot
ExecStart=/bin/sh -c 'echo never | tee /sys/kernel/mm/transparent_hugepage/enabled > /dev/null'

[Install]
WantedBy=basic.target

```


Save and close the file when finished.  Then, reload systemd to make your system aware of the new disable-thp service:


```
sudo systemctl daemon-reload


```


Next, start the disable-thp service:


```
sudo systemctl start disable-thp.service


```


You can confirm that THP has been disabled by checking the contents of the /sys/kernel/mm/transparent_hugepage/enabled file:


```
cat /sys/kernel/mm/transparent_hugepage/enabled


```


If THP was successfully disabled, never will be wrapped in brackets in the output:


```
Outputalways madvise [never]

```


Following that, enable the disable-thp service so that it will start up automatically and disable THP whenever the server boots up.  Notice that this command doesn’t include .service in the service file definition.  systemctl will append this suffix to whatever argument you pass automatically if it isn’t already present, so it isn’t necessary to include it:


```
sudo systemctl enable disable-thp


```


The disable-thp service will now start up whenever your server boots and disables THP before the MongoDB service starts.  However, there’s one more step you’ll need to take to ensure that THP remains disabled on your system.


By default, CentOS 8 also has tuned — a kernel tuning tool — installed and enabled.  tuned uses a number of preconfigured tuning profiles that can improve performance for a number of specific use cases.  You can edit these profiles or create new ones customized for your system.


The tuned tool can affect the THP setting on your system, so the MongoDB documentation also recommends that you also create a custom profile to ensure that THP doesn’t get enabled unexpectedly.


Get started with this by creating a new directory to hold the custom tuned profile:


```
sudo mkdir /etc/tuned/no-thp


```


Within this directory, create a configuration file named tuned.conf:


```
sudo nano /etc/tuned/no-thp/tuned.conf


```


Add the following two sections to the file:


/etc/tuned/no-thp/tuned.conf
```
[main]
include=virtual-guest

[vm]
transparent_hugepages=never

```


The first section, [main], must be included in every tuned configuration file.  Here, we only specify one include statement which will cause the no-thp profile to inherit the characteristics of another tuned profile named virtual-guest.



Note: Inheriting the characteristics of the virtual-guest profile like this will work in many cases, but it may not be optimal for every case.  We encourage you to review this documentation on provided tuned profiles to determine whether inheriting the characteristics of another profile would be more appropriate for your system.

The next section specifies a special plugin — vm — which is used specifically to enable or disable THP based on the value following the transparent_hugepages boolean option.


After adding these lines, save and close the file.  Then enable the new profile:


```
sudo tuned-adm profile no-thp


```


With that, you’ve disabled THP on your server.  You can now start the MongoDB service and test the database’s functionality.


# Step 3 — Starting the MongoDB Service and Testing the Database


The installation process described in Step 1 automatically configures MongoDB to run as a daemon controlled by systemd, meaning you can manage MongoDB using the various systemctl commands.  However, this installation procedure doesn’t automatically start the service.


Run the following systemctl command to start the MongoDB service:


```
sudo systemctl start mongod


```


Then check the service’s status:


```
sudo systemctl status mongod


```


This command will return output like the following, indicating that the service is up and running:


```
Output● mongod.service - MongoDB Database Server
   Loaded: loaded (/usr/lib/systemd/system/mongod.service; enabled; vendor preset: disabled)
   Active: active (running) since Thu 2020-10-01 20:31:26 UTC; 19s ago
     Docs: https://docs.mongodb.org/manual
  Process: 14208 ExecStart=/usr/bin/mongod $OPTIONS (code=exited, status=0/SUCCESS)
  Process: 14205 ExecStartPre=/usr/bin/chmod 0755 /var/run/mongodb (code=exited, status=0/SUCCESS)
  Process: 14203 ExecStartPre=/usr/bin/chown mongod:mongod /var/run/mongodb (code=exited, status=0/SUCCESS)
  Process: 14201 ExecStartPre=/usr/bin/mkdir -p /var/run/mongodb (code=exited, status=0/SUCCESS)
 Main PID: 14210 (mongod)
   Memory: 66.6M
   CGroup: /system.slice/mongod.service
           └─14210 /usr/bin/mongod -f /etc/mongod.conf

```


After confirming that the service is running as expected, enable the MongoDB service to start up at boot:


```
sudo systemctl enable mongod


```


You can further verify that the database is operational by connecting to the database server and executing a diagnostic command.  The following command will connect to the database and output its current version, server address, and port.  It will also return the result of MongoDB’s internal connectionStatus command:


```
mongo --eval 'db.runCommand({ connectionStatus: 1 })'


```


connectionStatus will check and return the status of the database connection.  A value of 1 for the ok field in the response indicates that the server is working as expected:


```
OutputMongoDB shell version v4.4.1
connecting to: mongodb://127.0.0.1:27017/?compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("460fe822-2881-477c-b095-aa3ccb49702d") }
MongoDB server version: 4.4.1
{
	"authInfo" : {
		"authenticatedUsers" : [ ],
		"authenticatedUserRoles" : [ ]
	},
	"ok" : 1
}

```


Also, note that the database is running on port 27017 on 127.0.0.1, the local loopback address representing localhost.  This is MongoDB’s default port number.


Next, we’ll look at how to manage the MongoDB server instance with systemd.


# Step 4 — Managing the MongoDB Service


As mentioned previously, the installation process described in Step 1 configures MongoDB to run as a systemd service.  This means that you can manage it using standard systemctl commands as you would with other CentOS system services.


Recall that the systemctl status command checks the status of the MongoDB service:


```
sudo systemctl status mongod


```


You can stop the service anytime by typing:


```
sudo systemctl stop mongod


```


To start the service when it’s stopped, run:


```
sudo systemctl start mongod


```


You can also restart the server when it’s already running:


```
sudo systemctl restart mongod


```


In Step 3, you enabled MongoDB to start automatically with the server.  If you ever wish to disable this automatic startup, type:


```
sudo systemctl disable mongod


```


Then to re-enable it to start up at boot, run the enable command again:


```
sudo systemctl enable mongod


```


For more information on how to manage systemd services, check out Systemd Essentials: Working with Services, Units, and the Journal.


# Conclusion


In this tutorial, you added the official MongoDB repository to your list of DNF repos and installed the latest version of the database.  You then disabled Transparent Huge Pages to optimize the database’s performance, tested Mongo’s functionality, and practiced some systemctl commands.


As an immediate next step, we strongly recommend that you harden your MongoDB installation’s security by following our guide on How To Secure MongoDB on CentOS 8.  Once it’s secured, you could then configure MongoDB to accept remote connections.


You can find more tutorials on how to configure and use MongoDB in these DigitalOcean community articles.  We also encourage you to check out the official MongoDB documentation, as it’s a great resource on the possibilities that MongoDB provides.


