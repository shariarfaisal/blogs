# How To Set Up a Firewall with UFW on Debian 11

```Debian``` ```Debian 11``` ```Firewall``` ```Networking``` ```Security```

## Introduction


UFW, or Uncomplicated Firewall, is a simplified firewall management interface that hides the complexity of lower-level packet filtering technologies such as iptables and nftables. If you’re looking to get started securing your network, and you’re not sure which tool to use, UFW may be the right choice for you.


This tutorial will show you how to set up a firewall with UFW on Debian 11.


# Prerequisites


To follow this tutorial, you will need one Debian 11 server with a sudo non-root user, which you can set up by following Step 1 through 3 in the Initial Server Setup with Debian 11 tutorial.


# Step 1 – Installing UFW


Debian does not install UFW by default. If you followed the entire Initial Server Setup tutorial, you will have installed and enabled UFW. If not, install it now using apt:


```
sudo apt install ufw


```


You will set up UFW and enable it in the following steps.


# Step 2 — Using IPv6 with UFW (Optional)


This tutorial is written with IPv4 in mind but will work for IPv6 as well if you enable it. If your Debian server has IPv6 enabled, you will want to ensure that UFW is configured to support IPv6. This will ensure that UFW will manage firewall rules for IPv6 in addition to IPv4. To configure this, open the UFW configuration file /etc/default/ufw with nano or your favorite editor:


```
sudo nano /etc/default/ufw


```


Locate IPV6 in the file and ensure the value is yes:


/etc/default/ufw excerpt
```
IPV6=yes

```


Save and close the file. If you’re using nano, press CTRL+X, then Y, then ENTER to save and exit the file.


Now when UFW is enabled, it will be configured to write both IPv4 and IPv6 firewall rules. Before enabling UFW, however, you will want to ensure that your firewall is configured to allow you to connect via SSH. Start with setting the default policies.


# Step 3 — Setting Up Default Policies


If you’re just getting started with your firewall, the first rules to define are your default policies. These rules handle traffic that does not explicitly match any other rules. By default, UFW is set to deny all incoming connections and allow all outgoing connections. This means anyone trying to reach your server would not be able to connect, while any application within the server would be able to reach the outside world.


Set your UFW rules back to the defaults so you can be sure that you’ll be able to follow along with this tutorial. To set the defaults used by UFW, use these commands:


```
sudo ufw default deny incoming
sudo ufw default allow outgoing


```


You will receive output like the follow:


```
OutputDefault incoming policy changed to 'deny'
(be sure to update your rules accordingly)
Default outgoing policy changed to 'allow'
(be sure to update your rules accordingly)

```


These commands set the defaults to deny incoming and allow outgoing connections. These firewall defaults alone might suffice for a personal computer, but servers typically need to respond to incoming requests from outside users. You’ll start this process in the next step.


# Step 4 — Allowing SSH Connections


You can’t enable your UFW firewall yet, because it would deny all incoming connections including your attempts to access your server. This means that you will need to create rules that explicitly allow legitimate incoming connections — SSH or HTTP connections, for example — if you want your server to respond to those types of requests. If you’re using a cloud server, you will probably want to allow incoming SSH connections so you can connect to and manage your server.


To configure your server to allow incoming SSH connections, use this command:


```
sudo ufw allow ssh


```


This will create firewall rules that will allow all connections on port 22, which is the port that the SSH daemon listens on by default. UFW knows what port is tied to allow ssh because it’s listed as a service in the /etc/services file.


However, you can actually write the equivalent rule by specifying the port instead of the service name. For example, this command produces the same result as the one above:


```
sudo ufw allow 22


```


If you configured your SSH daemon to use a different port, you will have to specify the appropriate port. For example, if your SSH server is listening on port 2222, you can use this same command but replace 22 with 2222.


Now that your firewall is configured to allow incoming SSH connections, you can enable it.


# Step 5 — Enabling UFW


To enable UFW, use this command:


```
sudo ufw enable


```


You will receive a warning that says the command may disrupt existing SSH connections. You already set up a firewall rule that allows SSH connections, so it should be fine to continue. Respond to the prompt with y and hit ENTER.


The firewall is now active. To see the rules that you have set, run this command:


```
sudo ufw status verbose


```


The rest of this tutorial covers how to use UFW in more detail, including allowing and denying different types of connections.


# Step 6 — Allowing Other Connections


At this point, you should allow all of the other connections that your server needs to function properly. The connections that you should allow depend on your specific needs. You already know how to write rules that allow connections based on a service name or port; you did this for SSH on port 22.


You can do this for HTTP on port 80, which is what unencrypted web servers use. To allow this type of traffic, you would type:


```
sudo ufw allow http


```


You can also do this for HTTPS on port 443, which is what encrypted web servers use. To allow this type of traffic, you would type:


```
sudo ufw allow https


```


In both scenarios, specifying the ports would also work, with HTTP being 80 and HTTPS being 443. For example:


```
sudo ufw allow 80


```


There are other ways to allow connections, however, aside from specifying a port or known service. This is discussed next.


## Specific Port Ranges


You can specify port ranges with UFW. For example, some applications use multiple ports instead of a single port.


For example, to allow X11 connections, which use ports 6000-6007, use these commands:


```
sudo ufw allow 6000:6007/tcp
sudo ufw allow 6000:6007/udp


```


When specifying port ranges with UFW, you must specify the protocol (tcp or udp) that the rules should apply to. This was not mentioned before because not specifying the protocol automatically allows both protocols, which is OK in most cases.


## Specific IP Addresses


When working with UFW, you can also specify IP addresses. For example, if you want to allow connections from a specific IP address, such as a work or home IP address of 203.0.113.4, you need to specify from and then the IP address:


```
sudo ufw allow from 203.0.113.4


```


You can also specify a specific port that the IP address is allowed to connect to by adding to any port followed by the port number. For example, if you want to allow 203.0.113.4 to connect to port 22 (SSH), use this command:


```
sudo ufw allow from 203.0.113.4 to any port 22


```


## Subnets


If you want to allow a subnet of IP addresses, you can do so using CIDR notation to specify a netmask. For example, if you want to allow all of the IP addresses ranging from 203.0.113.1 to 203.0.113.254 you can use this command:


```
sudo ufw allow from 203.0.113.0/24


```


Likewise, you may also specify the destination port that the subnet 203.0.113.0/24 is allowed to connect to. Again, using port 22 (SSH) as an example:


```
sudo ufw allow from 203.0.113.0/24 to any port 22


```


## Connections to a Specific Network Interface


If you want to create a firewall rule that only applies to a specific network interface, you can do so by specifying allow in on, followed by the name of the network interface.


It will be helpful to look up your network interfaces before continuing. To do so, use this command:


```
ip addr


```


```
Output. . .
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state
. . .
3: eth1: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default
. . .

```


The highlighted output indicates the network interface names. They typically have names like eth0 or enp3s2.


If your server has a public network interface named eth0, for example, you could allow HTTP traffic to it with this command:


```
sudo ufw allow in on eth0 to any port 80


```


Doing so would allow your server to receive HTTP requests from the public internet.


Or, if you want a MySQL database server (port 3306) to listen for connections on the private network interface eth1, you could use this command:


```
sudo ufw allow in on eth1 to any port 3306


```


This would allow other servers on your private network to connect to your MySQL database.


# Step 7 — Denying Connections


If you haven’t changed the default policy for incoming connections, UFW is configured to deny all incoming connections. Generally, this simplifies the process of creating a secure firewall policy by requiring you to create rules that explicitly allow specific ports and IP addresses through.


Sometimes you will want to deny specific connections based on the source IP address or subnet, however, perhaps because you know that your server is being attacked from there. Also, if you want to change your default incoming policy to allow (which is not recommended), you would need to create deny rules for any services or IP addresses that you don’t want to allow connections for.


To write deny rules, you can use the commands described above, replacing allow with deny.


For example, to deny HTTP connections, you could use this command:


```
sudo ufw deny http


```


Or if you want to deny all connections from 203.0.113.4 you could use this command:


```
sudo ufw deny from 203.0.113.4


```


Now, you can learn how to implement delete rules.


# Step 8 — Deleting Rules


Knowing how to delete firewall rules is just as important as knowing how to create them. There are two ways to specify which rules to delete: by the rule number or by the rule itself. This is similar to how the rules were specified when they were created.


## By Rule Number


If you’re using the rule number to delete firewall rules, the first thing you’ll want to do is get a list of your firewall rules. The UFW status command has the numbered option, which displays numbers next to each rule:


```
sudo ufw status numbered


```


```
OutputStatus: active

     To                         Action      From
     --                         ------      ----
[ 1] 22                         ALLOW IN    15.15.15.0/24
[ 2] 80                         ALLOW IN    Anywhere

```


If you decide that you want to delete rule 2, which allows HTTP connections on port 80, you can specify this in the following UFW delete command:


```
sudo ufw delete 2


```


This will show a confirmation prompt, which you can answer with y/n. Typing y will then delete rule 2. Note that if you have IPv6 enabled, you will want to delete the corresponding IPv6 rule as well.


## By Actual Rule


The alternative to rule numbers is to specify the actual rule to delete. For example, if you want to remove the allow http rule, you could write it like this:


```
sudo ufw delete allow http


```


You can also specify the rule with allow 80 instead of the service name:


```
sudo ufw delete allow 80


```


This method will delete both IPv4 and IPv6 rules, if they exist.


# Step 9 — Checking UFW Status and Rules


At any time, you can check the status of UFW with this command:


```
sudo ufw status verbose


```


If UFW is disabled, which is the default, the output will be this:


```
OutputStatus: inactive

```


If UFW is active, which it should be if you followed Step 3, the output will say that it’s active and will list any rules that you have set. For example, if the firewall is set to allow SSH (port 22) connections from anywhere, the output might include something like this:


```
OutputStatus: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere

```


Use the status command if you want to check how UFW has configured the firewall.


# Step 10 — Disabling or Resetting UFW (optional)


If you decide you don’t want to use UFW, you can disable it with this command:


```
sudo ufw disable


```


Any rules that you created with UFW will no longer be active. You can always run sudo ufw enable if you need to activate it later.


If you already have UFW rules configured, but you decide that you want to start over, you can use the reset command:


```
sudo ufw reset


```


This will disable UFW and delete any rules that you have previously defined. Keep in mind that the default policies won’t change to their original settings if you modified them at any point. This should give you a fresh start with UFW.


# Conclusion


Your firewall is now configured to allow (at least) SSH connections. Be sure to allow any other incoming connections that your server needs, while also limiting unnecessary connections. This will ensure that your server is both functional and secure.


To learn about more common UFW configurations, check out this tutorial on UFW Essentials: Common Firewall Rules and Commands.


