# How To Install and Secure Memcached on Ubuntu 18 04

```Security``` ```Ubuntu``` ```Ubuntu 20.04``` ```Server Optimization``` ```Caching``` ```Firewall```

A previous version of this tutorial was written by Kathleen Juell.


## Introduction


Memory object caching systems like Memcached can optimize backend database performance by temporarily storing information in memory, retaining frequently or recently requested records. In this way, they reduce the number of direct requests to your databases.


In this guide, you will learn how to install and configure a Memcached server. You’ll also learn how to add authentication to secure Memcached using Simple Authentication and Security Layer (SASL). Finally, you’ll learn how to bind Memcached to a local or private network interface to ensure that it is only accessible on trusted networks, by authenticated users.


# Prerequisites


To follow this tutorial, you will need:


- One Ubuntu 18.04 server with a sudo non-root user and a firewall enabled. To set this up, you can follow our Initial Server Setup with Ubuntu 18.04 tutorial.

With these prerequisites in place, you will be ready to install and secure your Memcached server.


# Step 1 — Installing Memcached


If you don’t already have Memcached installed on your server, you can install it from the official Ubuntu repositories. First, make sure that your local package index is updated using the following command:


```
sudo apt update


```


Next, install the official package as follows:


```
sudo apt install memcached


```


You can also install libmemcached-tools, which is a package that contains various tools that you can use to examine, test, and manage your Memcached server. Add the package to your server with the following command:


```
sudo apt install libmemcached-tools


```


Memcached should now be installed as a service on your server, along with tools that will allow you to test its connectivity. Now you can move on to securing its configuration settings.


# Step 2 — Configuring Memcached Network Settings (Optional)


If your Memcached server only needs to support local IPv4 connections using TCP then you can skip this section and continue to Step 3 in this tutorial. Otherwise,  if you’d like to configure Memcached to use UDP sockets, Unix Domain Sockets, or add support for IPv6 connections, then proceed with the relevant steps in this section of the tutorial.


First, ensure that your Memcached instance is listening on the local IPv4 loopback interface 127.0.0.1. The current version of Memcached that ships with Ubuntu and Debian has its -l configuration parameter set to the local interface, which means that it is configured to only accept connections from the server where Memcached is running.


Verify that Memcached is currently bound to the local IPv4 127.0.0.1 interface and listening only for TCP connections by using the ss command:


```
sudo ss -plunt


```


The various flags will alter ss output in the following ways:


- -p adds the name of the process that is using a socket
- -l limits the output to listening sockets only, as opposed to also including connected sockets to other systems
- -u includes UDP based sockets in the output
- -n displays numeric values in the output instead of human readable names and values
- -t includes TCP based sockets in the output

You should receive output like the following:


```
OutputNetid      State       Recv-Q      Send-Q           Local Address:Port             Peer Address:Port      Process                                         
. . .
tcp        LISTEN      0           1024                 127.0.0.1:11211                 0.0.0.0:*          users:(("memcached",pid=8889,fd=26))
. . .

```


This output confirms that memcached is bound to the IPv4 loopback 127.0.0.1 address using the TCP protocol only.


Now that you have confirmed that Memcached is configured to support IPv4 with TCP connections only, you can edit /etc/memcached.conf to add support for UDP, Unix Domain Sockets, or IPv6 connections.


## Configuring IPv6


To enable IPv6 connections to Memcached, open the /etc/memcached.conf file with nano or your preferred editor:


```
sudo nano /etc/memcached.conf


```


First, find the following line in the file:


/etc/memcached.conf
```
. . .
-l 127.0.0.1

```


This line is where Memcached is configured to listen on the local IPv4 interface. To add IPv6 support, add a line containing the IPv6 local loopback address (::1) like this:


/etc/memcached.conf
```
. . .
-l 127.0.0.1
-l ::1

```


Save and close the file by pressing CTRL+O then ENTER to save, then CTRL+X to exit nano. Then restart Memcached using the systemctl command:


```
sudo systemctl restart memcached


```


Now you can verify that Memcached is also listening for IPv6 connections by repeating the ss command from the previous section:


```
sudo ss -plunt


```


You should receive output like the following:


```
OutputNetid      State       Recv-Q      Send-Q           Local Address:Port             Peer Address:Port      Process                                         
. . .
tcp        LISTEN      0           1024                 127.0.0.1:11211                 0.0.0.0:*          users:(("memcached",pid=8889,fd=26))           
. . .
tcp        LISTEN      0           1024                     [::1]:11211                    [::]:*          users:(("memcached",pid=8889,fd=27))

```


The highlighted portions of the output indicate that Memcached is now listening for TCP connections on the local IPv6 interface.


Note that if you would like to disable IPv4 support and only listen for IPv6 connections, you can remove the -l 127.0.0.1 line from /etc/memcached.conf and restart the service using the systemctl command.


## Configuring UDP


If you would like to use Memcached with UDP sockets, you can enable UDP support by editing the /etc/memcached.conf configuration file.


Open /etc/memcached.conf using nano or your preferred editor, then add the following line to the end of the file:


/etc/memcached.conf
```
. . .
-U 11211

```


If you do not need TCP support, find the -p 11211 line and change it to -p 0 to disable TCP connections.


When you are done editing the file, save and close it by entering CTRL+O to save, and then CTRL+X to exit.


Next, restart your Memcached service with the systemctl command to apply your changes:


```
sudo systemctl restart memcached


```


Verify that Memcached is listening for UDP connections using the ss -plunt command again:


```
sudo ss -plunt


```


If you disabled TCP support and have IPv6 connections enabled, then you should receive output like the following:


```
[secondary_label Output] 
Netid      State       Recv-Q      Send-Q           Local Address:Port             Peer Address:Port      Process                                         
. . .
udp        UNCONN      0           0                    127.0.0.1:11211                 0.0.0.0:*          users:(("memcached",pid=8889,fd=28))
udp        UNCONN      0           0                        [::1]:11211                    [::]:*          users:(("memcached",pid=8889,fd=29))
. . .

```


Note that your output may be different if you only have IPv4 connections enabled, and if you have left TCP connections enabled.


## Configuring Unix Domain Sockets


If you would like to use Memcached with Unix Domain Sockets, you can enable this support by editing the /etc/memcached.conf configuration file. Note that if you configure Memcached to use a Unix Domain Socket, Memcached will disable TCP and UDP support so be sure that your applications do not need to connect using those protocols before enabling socket support.


Open /etc/memcached.conf using nano or your preferred editor, then add the following lines to the end of the file:


/etc/memcached.conf
```
. . .
-s /var/run/memcached/memcached.sock
-a 660

```


The -a flag determines the permissions on the socket’s file. Ensure that the user that needs to connect to Memcached is a part of the memcache group on your server, otherwise it will receive a permission denied message when trying to access the socket.


Next, restart your Memcached service with the systemctl command to apply your changes:


```
sudo systemctl restart memcached


```


Verify that Memcached is listening for Unix Domain Socket connections using the ss -lnx command:


```
sudo ss -lnx | grep memcached


```


The -x flag limits the output of ss to display socket files. You should receive output like the following:


```
Outputu_str LISTEN 0      1024             /var/run/memcached/memcached.sock 20234658                              * 0

```


Now that you have configured Memcached’s network settings, you can proceed to the next step in this tutorial, which is adding SASL for authentication to Memcached.


# Step 3 — Adding Authorized Users


To add authenticated users to your Memcached service, you can use Simple Authentication and Security Layer (SASL), which is a framework that de-couples authentication procedures from application protocols. First you’ll add support for SASL to your server, and then you will configure a user with authentication credentials. With everything in place, you can then enable SASL within Memcached’s configuration file and confirm everything is working correctly.


## Adding an Authenticated User


To get started adding SASL support, you will need to install the sasl2-bin package, which contains administrative programs for the SASL user database. This tool will allow you to create an authenticated user or users. Run the following command to install it:


```
sudo apt install sasl2-bin


```


Next, create the directory and file that Memcached will check for its SASL configuration settings using the mkdir command:


```
sudo mkdir -p /etc/sasl2


```


Now create the SASL configuration file using nano or your preferred editor:


```
sudo nano /etc/sasl2/memcached.conf


```


Add the following lines:


/etc/sasl2/memcached.conf
```
log_level: 5
mech_list: plain
sasldb_path: /etc/sasl2/memcached-sasldb2

```


In addition to specifying the logging level, mech_list is set to plain, which tells Memcached that it should use its own password file and verify a plaintext password. The last line that you added specifies the path to the user database file that you will create next. Save and close the file when you are finished.


Now you will create a SASL database with user credentials. You’ll use the saslpasswd2 command with the -c flag to create a new user entry in the SASL database. The user will be sammy here, but you can replace this name with your own user. The -f flag specifies the path to the database, which will be the path that you set in /etc/sasl2/memcached.conf:


```
sudo saslpasswd2 -a memcached -c -f /etc/sasl2/memcached-sasldb2 sammy


```


Finally, give the memcache user and group ownership over the SASL database with the following chown command:


```
sudo chown memcache:memcache /etc/sasl2/memcached-sasldb2


```


You now have an SASL configuration in place that Memcached can use for authentication. In the next section you’ll confirm that Memcached is running with its default settings first, and then reconfigure it and verify that it is working with SASL authentication.


## Configuring SASL Support


We can first test the connectivity of our Memcached instance with the memcstat command. This check will help us establish that Memcached is running and correctly configured before SASL and user authentication are enabled. After we make changes to our configuration files we’ll run the command again to check for different output.


To check that Memcached is up and running using the memcstat command, type the following:


```
memcstat --servers="127.0.0.1"


```


If you are using IPv6, substitute ::1 in place of the IPv4 127.0.0.1 address. If you are using a Unix Domain Socket, use the path to the socket in place of the IP address, for example --servers=/var/run/memcached/memached.sock.


When you run the memcstat command and connect to Memcached successfully, you should receive output like the following:


```
OutputServer: 127.0.0.1 (11211)
     pid: 2299875
     uptime: 2020
     time: 1632404590
     version: 1.5.22
     . . .

```



Note: If you are using Memcached with UDP support, the memcstat command will not be able to connect to the UDP port. You can use the following netcat command to verify connectivity:
nc -u 127.0.0.1 11211 -vz


If Memcached is responding, you should receive output like the following:
OutputConnection to 127.0.0.1 11211 port [udp/*] succeeded!

If you are using Memcached with IPv6 and UDP, the command should be the following:
nc -6 -u ::1 11211 -vz



Now you can move on to enabling SASL. To do so, add the -S parameter to /etc/memcached.conf. Open the file with nano or your preferred editor again:


```
sudo nano /etc/memcached.conf


```


At the bottom of the file, add the following:


/etc/memcached.conf
```
. . .
-S

```


Next, find and uncomment the -vv option, which will provide verbose output to /var/log/memcached. The uncommented line should look like this:


/etc/memcached.conf
```
. . .
-vv

```


Save and close the file.


Now restart the Memcached service using the following systemctl command:


```
sudo systemctl restart memcached


```


Next, check the journalctl logs for Memcached to be sure that SASL support is enabled:


```
sudo journalctl -u memcached |grep SASL


```


You should receive a line of output like the following, indicating that SASL support has been initialized:


```
OutputSep 23 17:00:55 memcached systemd-memcached-wrapper[2303930]: Initialized SASL.

```


Now you can check connectivity to Memcached again. With SASL support in place and initialized, the following memcstat command should fail without valid authentication credentials:


```
memcstat --servers="127.0.0.1"


```


The command should not produce output. Type the following shell command to check its status:


```
echo $?


```


$? will always return the exit code of the last command that exited. Typically, anything besides 0 indicates process failure. In this case, you should receive an exit status of 1, which indicates that the memcstat command failed.


Running memcstat again, this time with a username and password will confirm whether or not the authentication process worked. Run the following command with your credentials substituted in place of the sammy and your_password` values if you used different credentials:


```
memcstat --servers="127.0.0.1" --username=sammy --password=your_password


```


You should receive output like the following:


```
OutputServer: 127.0.0.1 (11211)
     pid: 3831
     uptime: 9
     time: 1520028517
     version: 1.4.25
     . . .

```


Your Memcached service is now configured and running with SASL support and user authentication.


# Step 4 — Allowing Access Over the Private Network (Optional)


By default Memcached is only configured to listen on the local loopback (127.0.0.1) interface, which protects the Memcached interface from exposure to outside parties. However, there may be instances where you will need to allow access from other servers. In this case, you can adjust your configuration settings to bind Memcached to a private network interface.



Note: We will cover how to configure firewall settings using UFW in this section, but it is also possible to use DigitalOcean Cloud Firewalls to create these settings. For more information on setting up DigitalOcean Cloud Firewalls, see our Introduction to DigitalOcean Cloud Firewalls. To learn more about how to limit incoming traffic to particular machines, check out the section of this tutorial on applying firewall rules using tags and server names and our discussion of firewall tags.

## Limiting IP Access With Firewalls


Before you adjust your configuration settings, it is a good idea to set up firewall rules to limit the machines that can connect to your Memcached server. First you will need to record the private IP address of any machine that you would like to use to connect to Memcached. Once you have the private IP address (or addresses), you will add an explicit firewall rule to allow the machine to access Memcached.


If you are using the UFW firewall, you can limit access to your Memcached instance by typing the following on your Memcached server:


```
sudo ufw allow from client_system_private_IP/32 to any port 11211


```


If more than one system is accessing Memcached via the private network, be sure to add additional ufw rules for each machine using the above rule as a template.


You can find out more about UFW firewalls by reading our ufw essentials guide.


With these changes in place, you can adjust the Memcached service to bind to your server’s private networking interface.


## Binding Memcached to the Private Network Interface


Now that your firewall is in place, you can adjust the Memcached configuration to bind to your server’s private networking interface instead of 127.0.0.1.


First, find the private network interface for your Memcached server using the following ip command


```
ip -brief address show


```


Depending on your server’s network configuration, the output may be different from the following example:


```
Outputlo               UNKNOWN        127.0.0.1/8 ::1/128
eth0             UP             203.0.113.1/20 10.10.0.5/16 2001:DB8::1/64 fe80::7ced:9ff:fe52:4695/64
eth1             UP             10.136.32.212/16 fe80::2cee:92ff:fe21:8bc4/64

```


In this example, the network interfaces are identified by their eth0 and eth1 names. The highlighted IPv4 addresses on the eth0 line are the public IP addresses for the server.


The highlighted 10.136.32.212 address on the eth1 line is the private IPv4 address for the server, and the fe80::2cee:92ff:fe21:8bc4 address is the IPv6 private address. Your IP addresses will be different but will always fall within the following ranges based on the RFC 1918 specification):


- 10.0.0.0 to 10.255.255.255   (10/8 prefix)
- 172.16.0.0 to 172.31.255.255   (172.16/12 prefix)
- 192.168.0.0 to 192.168.255.255 (192.168/16 prefix)

Once you have found your server’s private IP address or addresses, open the /etc/memcached.conf file again using nano or your preferred editor:


```
sudo nano /etc/memcached.conf


```


Find the -l 127.0.0.1 line that you checked or modified earlier, and change the address to match your server’s private networking interface:


/etc/memcached.conf
```
. . .
-l memcached_servers_private_IP
. . .

```


If you would like to have Memcached listen on multiple addresses, add another similar line for each address, either IPv4 or IPv6 using the -l memcached_servers_private_IP format.


Save and close the file when you are finished.


Next, restart the Memcached service:


```
sudo systemctl restart memcached


```


Check your new settings with ss to confirm the change:


```
sudo ss -plunt


```


```
OutputNetid      State       Recv-Q      Send-Q           Local Address:Port             Peer Address:Port      Process
. . .
tcp       LISTEN      0           1024                10.137.0.2:11211                 0.0.0.0:*          users:(("memcached",pid=8991,fd=27))
. . .

```


Test connectivity from your external client to ensure that you can still reach the service.  It is a good idea to also check access from a non-authorized client (try connecting without a user and password) to ensure that your SASL authentication is working as expected. It is also a good idea to try connecting to Memcached from a different server that is not allowed to connect to verify that the firewall rules you created are effective.


# Conclusion


In this tutorial you explored how to configure Memcached with IPv4, IPv6, TCP, UDP, and Unix Domain Sockets. You also learned how to secure your Memcached server by enabling SASL authentication. Finally, you learned how to bind Memcached to your local or private network interface and how to configure firewall rules to limit access to Memcached.


To learn more about Memcached, check out the project documentation.


