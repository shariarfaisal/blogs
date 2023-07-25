# How To Share PHP Sessions on Multiple Memcached Servers on Ubuntu 14 04

```Ubuntu``` ```PHP``` ```Caching``` ```Scaling``` ```LAMP Stack```

## Introduction


Memcached is a distributed object caching system which stores information in memory, rather than on disk, for faster access. PHP’s Memcache module can be used to handle sessions which would otherwise be stored on the file system. Storing PHP sessions in Memcached has the advantage of being able to distribute them to multiple cloud servers running Memcached, so as to maintain session redundancy.


Without this Memcached setup, if your application is being load balanced on multiple servers, it would be necessary to configure session stickiness on the load balancer. This maintains user experience and prevents them from being logged off suddenly. Configuring Memcached to handle sessions will ensure all cloud servers in the Memcached pool have the same set of session data, which eliminates the need to be sticky with one server to preserve the session.


## Prerequisites


This tutorial assumes you are familiar with setting up LAMP servers in Ubuntu.
This setup will make use of 3 Droplets with the Ubuntu 14.04 image.


Droplet 1


- Name: lamp01
- Public IP: 1.1.1.1
- Private IP: 10.1.1.1

Droplet 2


- Name: lamp02
- Public IP: 2.2.2.2
- Private IP: 10.2.2.2

Droplet 3


- Name: lamp03
- Public IP: 3.3.3.3
- Private IP: 10.3.3.3

Ensure that the Private Networking checkbox is ticked when creating the Droplets. Also, make note of the private IP addresses as we will need them later.


Install LAMP on all three servers.


First, update the repository and install Apache.


```
apt-get update
apt-get install apache2

```


Install PHP and Apache’s mod_php extension.


```
apt-get install php5 libapache2-mod-php5 php5-mcrypt

```


For more information, see this article.


# Step One - Install Memcache Packages


On lamp01, install the Memcached daemon and PHP’s Memcache module.


```
apt-get install php5-memcache memcached

```


PHP has two packages: php5-memcache and php5-memcached (notice the “d” at the end). We will be using the first package (memcache) as it is lighter without any dependencies. Read the comparison between memcache and memcached.


The Memcached service listens only on localhost (127.0.0.1). This has to be changed to accept connections from the private network.


```
nano /etc/memcached.conf

```


Find the following line:


```
-l 127.0.0.1

```


Change it to listen on this server’s private IP address.


```
-l 10.1.1.1

```


Restart the memcached service.


```
service memcached restart

```


Repeat these steps on the other two servers, replacing 127.0.0.1 with the appropriate private IP address.


lamp02


```
-l 10.2.2.2

```


lamp03


```
-l 10.3.3.3

```


Restart the memcached service on the second two servers.


# Step Two - Set Memcache as PHP’s Session Handler


On lamp01, open the php.ini file for editing.


```
nano /etc/php5/apache2/php.ini

```


This file is located at /etc/php5/fpm/php.ini on PHP-FPM installations.


Find the following configuration directives:


```
session.save_handler =
session.save_path =

```


Modify them to use Memcache as follows. Use all three private IP addresses in the session.save_path.


```
session.save_handler = memcache
session.save_path = 'tcp://10.1.1.1:11211,tcp://10.2.2.2:11211,tcp://10.3.3.3:11211'

```


You may need to uncomment session.save_path by removing the semicolon at the beginning. Remember to enter the port number 11211 after each IP address, as Memcached listens on this port.


Add exactly the same settings on the other two servers.


On lamp02:


```
session.save_handler = memcache
session.save_path = 'tcp://10.1.1.1:11211,tcp://10.2.2.2:11211,tcp://10.3.3.3:11211'

```


On lamp03:


```
session.save_handler = memcache
session.save_path = 'tcp://10.1.1.1:11211,tcp://10.2.2.2:11211,tcp://10.3.3.3:11211'

```


This configuration has to be exactly same on all the Droplets for session sharing to work properly.


# Step Three - Configure Memcache for Session Redundancy


On lamp01, edit the memcache.ini file.


```
nano /etc/php5/mods-available/memcache.ini

```


Add the following configuration directives to the end of this file.


```
memcache.allow_failover=1
memcache.session_redundancy=4

```


The memcache.session_redundancy directive must be equal to the number of memcached servers + 1 for the session information to be replicated to all the servers. This is due to a bug in PHP.


These directives enable session failover and redundancy, so PHP writes the session information to all servers specified in session.save_path; similar to a RAID-1 setup.


Restart the web server or the PHP FPM daemon depending on what is being used.


```
service apache2 reload

```


Repeat these steps exactly on lamp02 and lamp03.


# Step Four - Test Session Redundancy


To test this setup create the following PHP script on all the Droplets.


/var/www/html/session.php


```
<?php
    header('Content-Type: text/plain');
    session_start();
    if(!isset($_SESSION['visit']))
    {
    	echo "This is the first time you're visiting this server\n";
    	$_SESSION['visit'] = 0;
    }
    else
            echo "Your number of visits: ".$_SESSION['visit'] . "\n";

    $_SESSION['visit']++;

	echo "Server IP: ".$_SERVER['SERVER_ADDR'] . "\n";
	echo "Client IP: ".$_SERVER['REMOTE_ADDR'] . "\n";
	print_r($_COOKIE);
?>

```


This script is for testing only and can be removed once the Droplets are set up.


Access this file on the first Droplet using curl and extract the cookie information.


```
curl -v -s http://1.1.1.1/session.php 2>&1 | grep 'Set-Cookie:'

```


This will return output similar to the following.


```
< Set-Cookie: PHPSESSID=8lebte2dnqegtp1q3v9pau08k4; path=/

```


Copy the PHPSESSID cookie and send the request to the other Droplets using this cookie. This session will be removed by PHP if no requests are made for 1440 seconds, so make sure you complete the test within this timeframe. Read about PHP’s session.gc-maxlifetime to learn more about this.


```
curl --cookie "PHPSESSID=8lebte2dnqegtp1q3v9pau08k4" http://1.1.1.1/session.php http://2.2.2.2/session.php http://3.3.3.3/session.php

```


You will find that the session is being carried over across all Droplets.


```
Your number of visits: 1
Server IP: 1.1.1.1
Client IP: 117.193.121.130
Array
(
    [PHPSESSID] => 8lebte2dnqegtp1q3v9pau08k4
)
Your number of visits: 2
Server IP: 2.2.2.2
Client IP: 117.193.121.130
Array
(
    [PHPSESSID] => 8lebte2dnqegtp1q3v9pau08k4
)
Your number of visits: 3
Server IP: 3.3.3.3
Client IP: 117.193.121.130
Array
(
    [PHPSESSID] => 8lebte2dnqegtp1q3v9pau08k4
)

```


To test failover, stop the memcached service and access this file on it.


```
service memcached stop

```


The Droplet transparently uses the session information stored on the other two servers.


```
curl --cookie "PHPSESSID=8lebte2dnqegtp1q3v9pau08k4" http://1.1.1.1/session.php

```


Output:


```
Your number of visits: 4
Server IP: 1.1.1.1
Client IP: 117.193.121.130
Array
(
    [PHPSESSID] => 8lebte2dnqegtp1q3v9pau08k4
)

```


Now a load balancer can be configured to distribute requests evenly without the hassle of configuring session stickyness.


Start memcached again once you are done testing:


```
service memcached start

```


# Step Five - Secure Memcached with IPTables


Even if Memcached is using the private network, other DigitalOcean users in the same data center can connect to your Droplet if they know your private IP. So, we will set up IPTables rules to allow only the cloud servers in our Memcached pool to communicate with each other.


We are doing this step after testing for session redundancy so that it is easier to troubleshoot problems which may arise if incorrect rules are applied.


Create firewall rules on lamp01 with the private IP addresses of lamp02 and lamp03.


```
iptables -A INPUT -s 10.2.2.2 -i eth1 -p tcp -m state --state NEW -m tcp --dport 11211 -j ACCEPT
iptables -A INPUT -s 10.3.3.3 -i eth1 -p tcp -m state --state NEW -m tcp --dport 11211 -j ACCEPT

```


On a typical LAMP server the following would be the complete set of rules:


```
iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp -m state --state NEW -m tcp --dport 443 -j ACCEPT
iptables -A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
iptables -A INPUT -s 10.2.2.2 -i eth1 -p tcp -m state --state NEW -m tcp --dport 11211 -j ACCEPT
iptables -A INPUT -s 10.3.3.3 -i eth1 -p tcp -m state --state NEW -m tcp --dport 11211 -j ACCEPT
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -p icmp -m icmp --icmp-type 8 -j ACCEPT
iptables -P INPUT DROP

```


Enter the firewall rules on lamp02 with the private IP addresses of lamp01 and lamp03.


```
iptables -A INPUT -s 10.1.1.1 -i eth1 -p tcp -m state --state NEW -m tcp --dport 11211 -j ACCEPT
iptables -A INPUT -s 10.3.3.3 -i eth1 -p tcp -m state --state NEW -m tcp --dport 11211 -j ACCEPT

```


Do the same on lamp03 with the private IP addresses of lamp01 and lamp02.


```
iptables -A INPUT -s 10.1.1.1 -i eth1 -p tcp -m state --state NEW -m tcp --dport 11211 -j ACCEPT
iptables -A INPUT -s 10.2.2.2 -i eth1 -p tcp -m state --state NEW -m tcp --dport 11211 -j ACCEPT

```


Repeat the tests in Step 4 to confirm that the firewall is not blocking our traffic.


## Additional Reading


- How To Install LAMP stack on Ubuntu 14.04
- How To Isolate Servers Within A Private Network Using IPTables
- How To Store PHP Sessions in Memcached

