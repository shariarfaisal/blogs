# How To Configure Remote Access for MongoDB on CentOS 8

```Databases``` ```Security``` ```MongoDB``` ```NoSQL``` ```Firewall``` ```Networking``` ```CentOS``` ```CentOS 8```

An earlier version of this tutorial was written by Melissa Anderson.


## Introduction


MongoDB, also known as Mongo, is an open-source document database used in many modern web applications.  By default, it only allows connections that originate on the same server where it’s installed.  If you want to manage MongoDB remotely or connect it to a separate application server, there are a few changes you’d need to make to the default configuration.


In this tutorial, you will configure a MongoDB installation to securely allow access from a trusted remote computer.  To do this, you’ll update your firewall rules to provide the remote machine access to the port on which MongoDB is listening for connections and then update Mongo’s configuration file to change its IP binding setting.  Then, as a final step, you’ll test that your remote machine is able to make the connection to your database successfully.


# Prerequisites


To complete this tutorial, you’ll need:


- A server running CentOS 8.  This server should have a non-root administrative user and a firewall configured with firewalld.  Set this up by following our initial server setup guide for CentOS 8.
- MongoDB installed on your server.  This tutorial assumes that you have MongoDB 4.4 or newer installed.  You can install this version by following our tutorial on How To Install MongoDB on CentOS 8.
- A second computer from which you’ll access your MongoDB instance.  For simplicity, this tutorial assumes that this machine is another CentOS 8 server. Like your MongoDB server, this machine should have a non-root administrative user and a firewall configured with firewalld as described in our initial server setup guide for CentOS 8. However, Steps 1 and 2, which describe the actual procedure for enabling remote connectivity on the database server, will work regardless of what operating system the remote machine is running.

Lastly, while it isn’t required to complete this tutorial, we strongly recommend that you secure your MongoDB installation by creating an administrative user account for the database and enabling authentication.  To do this, follow our tutorial on How To Secure MongoDB on CentOS 8.


# Step 1 — Adjusting the Firewall


Assuming you followed the prerequisite initial server setup tutorial and set up firewalld on your server, your MongoDB installation will be inaccessible from the internet.  If you intend to use MongoDB only locally with applications running on the same server, this is the recommended and secure setting.  However, if you would like to be able to connect to your MongoDB server from a remote location, you have to allow incoming connections to the port where the database is listening by adding a new firewall rule.


Start by checking which port your MongoDB installation is listening on with the netstat command.  netstat is a command line utility that displays information about active TCP network connections.


The following command will redirect the output produced by sudo netstat -plunt to a grep command that searches for any lines containing the string mongo:


```
sudo netstat -plunt | grep mongo


```


This example output indicates that MongoDB is listening for connections to 127.0.0.1, a special loopback address that represents localhost, on its default port, 27017:


```
Outputtcp        0      0 127.0.0.1:27017         0.0.0.0:*               LISTEN      15918/mongod    

```


In most cases, MongoDB should only be accessed from certain trusted locations, such as another server hosting an application.  One way to configure this with firewalld is to run the following firewall-cmd command on your MongoDB server, which opens up access on MongoDB’s default port while explicitly only allowing the IP address of another trusted server.


Run the following command, making sure to change trusted_server_ip to the IP address of the trusted remote machine you’ll use to access your MongoDB instance:



Note: If the previous command’s output indicated that your installation of MongoDB is listening on a non default port, use that port number in place of 27017 in this command.

```
sudo firewall-cmd --permanent --zone=public --add-rich-rule='rule family="ipv4" source address="trusted_server_ip" port protocol="tcp" port="27017" accept'


```


This command permanently adds a rich rule to the firewall’s public zone. Rich rules are features in firewalld that let you have more granular control over who has access to your server through the use of a number of options. The rule provided in this command specifies that only the trusted_server_ip address should be allowed to make connections through the wall. It also specifies that it may only do so using the TCP protocol to connect to port 27017.


If the rule was added successfully, the command will return success in the output:


```
Outputsuccess

```


Reload the firewall to put the new rule into effect:


```
sudo firewall-cmd --reload


```


In the future, if you ever want to access MongoDB from another machine, run this command again with the new machine’s IP address in place of trusted_server_ip.


You can verify the change in firewall settings by running firewall-cmd with the --list-all option:


```
sudo firewall-cmd --list-all


```


The output will include the new rich rule allowing traffic to port 27017 from the remote server:


```
Outputpublic (active)
. . .
  rich rules: 
	rule family="ipv4" source address="157.230.58.94" port port="27017" protocol="tcp" accept

```


You can learn more about firewalld in How To Set Up a Firewall Using firewalld on CentOS 8.


Next, you’ll bind MongoDB to the server’s public IP address so you can access it from your remote machine.


# Step 2 — Configuring a Public bindIP


At this point, even though the port is open, MongoDB is currently bound to 127.0.0.1, the local loopback network interface.  This means that MongoDB is only able to accept connections that originate on the server where it’s installed.


To allow remote connections, you must edit the MongoDB configuration file — /etc/mongod.conf — to additionally bind MongoDB to your server’s publicly-routable IP address.  This way, your MongoDB installation will be able to listen to connections made to your MongoDB server from remote machines.


Open the MongoDB configuration file in your preferred text editor.  The following example uses nano:


```
sudo nano /etc/mongod.conf


```


Find the network interfaces section, then the bindIp value:


/etc/mongod.conf
```
. . .
# network interfaces
net:
  port: 27017
  bindIp: 127.0.0.1

. . .

```


Append a comma to this line followed by your MongoDB server’s public IP address:


/etc/mongod.conf
```
. . .
# network interfaces
net:
  port: 27017
  bindIp: 127.0.0.1,mongodb_server_ip

. . .

```


Save and close the file.  If you used nano, do so by pressing CTRL + X, Y, then ENTER.


Then, restart MongoDB to put this change into effect:


```
sudo systemctl restart mongod


```


Following that, your MongoDB installation will be able to accept remote connections from whatever machines you’ve allowed to access port 27017.  As a final step, you can test whether the trusted remote server you allowed through the firewall in Step 1 can reach the MongoDB instance running on your server.


# Step 3 — Testing Remote Connectivity


Now that you configured your MongoDB installation to listen for connections that originate on its publicly-routable IP address and granted your remote machine access through your server’s firewall to Mongo’s default port, you can test that the remote machine is able to connect.



Note: As mentioned in the Prerequisites section, this tutorial assumes that your remote machine is another server running CentOS 8.  The procedure for enabling remote connections outlined in Steps 1 and 2 should work regardless of what operating system your remote machine runs, but the testing methods described in this step do not work universally across operating systems.

First, log into your trusted server using SSH:


```
ssh sammy@trusted_server_ip


```


One way to test that your trusted remote server is able to connect to the MongoDB instance is to use the nc command.  nc, short for netcat, is a utility used to establish network connections with TCP or UDP.  It’s useful for testing in cases like this because it allows you to specify both an IP address and a port number.


If you haven’t already, you may need to install nc. The version from the official CentOS repositories is actually an implementation called ncat, which was written by the Nmap Project as an update for netcat.


Install ncat by typing:


```
sudo dnf install nc


```


Press y and then ENTER when prompted to confirm that you want to install the package.


Then run the following nc command, which includes the -z option.  This limits nc to only scan for a listening daemon on the target server without sending it any data.  Recall from the prerequisite installation tutorial that MongoDB is running as a service daemon, making this option useful for testing connectivity.  It also includes the v option which increases the command’s verbosity, causing ncat to return some output which it otherwise wouldn’t.


Run the following nc command from your trusted remote server, making sure to replace mongodb_server_ip with the IP address of the server on which you installed MongoDB:


```
nc -zv mongodb_server_ip 27017


```


If the trusted server can access the MongoDB daemon, its output will indicate that it made a connection:


```
OutputNcat: Version 7.70 ( https://nmap.org/ncat )
Ncat: Connected to mongodb_server_ip:27017.
Ncat: 0 bytes sent, 0 bytes received in 0.02 seconds.

```


Assuming you have a compatible version of the mongo shell installed on your remote server, you can at this point connect directly to the MongoDB instance installed on the host server.


One way to connect is with a connection string URI, like this:


```
mongo "mongodb://mongo_server_ip:27017"


```



Note: If you followed the recommended How To Secure MongoDB on CentOS 8 tutorial, you will have closed off access to your database to unauthenticated users.  In this case, you’d need to use a URI that specifies a valid username, like this:
mongo "mongodb://username@mongo_server_ip:27017"


The shell will automatically prompt you to enter the user’s password.

With that, you’ve confirmed that your MongoDB server can accept connections from the trusted server.


# Conclusion


You can now access your MongoDB installation from a remote server.  At this point, you can manage your Mongo database remotely from the trusted server.  Alternatively, you could configure an application to run on the trusted server and use the database remotely.


If you haven’t configured an administrative user and enabled authentication, anyone who has access to your remote server can also access your MongoDB installation.  If you haven’t already done so, we strongly recommend that you follow our guide on How To Secure MongoDB on CentOS 8 to add an administrative user and lock things down further.


