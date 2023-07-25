# How To Setup a Firewall with UFW on an Ubuntu and Debian Cloud Server

```Security``` ```Ubuntu``` ```Firewall``` ```IPv6``` ```Debian```

## Introduction


Setting up a functioning firewall is crucial to securing your cloud server. Previously, setting up a firewall was done through complicated or arcane utilities. Many of these utilities (e.g., iptables) have a lot of functionality built into them, but do require extra effort from the user to learn and understand them.


Another option is UFW, or Uncomplicated Firewall. UFW is a front-end to iptables that aims to provide a more user-friendly interface than other firewall management utilities. UFW is well-supported in the Linux community, and is typically installed by default on many distributions.


In this tutorial, you’ll set up a firewall using UFW to secure an Ubuntu or Debian cloud server. You’ll also learn how to set up UFW default rules to allow or deny connections for ports and IP addresses, delete rules you’ve created, disable and enable UFW, and reset everything back to default settings if you prefer.


# Prerequisites


To follow this tutorial, you will need a server that’s running either Ubuntu or Debian. Your server should have a non-root user with sudo privileges. To set this up for Ubuntu, follow our guide on Initial Server Setup with Ubuntu 20.04. To set this up for Debian, follow our guide on Initial Server Setup with Debian 11. Both of these initial server setup guides will ensure that you have UFW installed on your machine and that you have a secure environment you can use to practice creating firewall rules.


# Using IPv6 with UFW


If your Virtual Private Server (VPS) is configured for IPv6, ensure that UFW is configured to support IPv6 so that it configures both your IPv4 and IPv6 firewall rules. To do this, open the UFW configuration file in your preferred text editor. Here we’ll use nano:


```
sudo nano /etc/default/ufw


```


Confirm that IPV6 is set to yes:


```
/etc/default/ufw# /etc/default/ufw
#

# Set to yes to apply rules to support IPv6 (no means only IPv6 on loopback
# accepted). You will need to 'disable' and then 'enable' the firewall for
# the changes to take affect.
IPV6=yes
…

```


After you’ve made your changes, save and exit the file. If you’re using nano, press CTRL + X, Y, and then ENTER.


Now restart your firewall by first disabling it:


```
sudo ufw disable


```


```
OutputFirewall stopped and disabled on system startup

```


Then enable it again:


```
sudo ufw enable


```


```
OutputFirewall is active and enabled on system startup

```


Your UFW firewall is now set up to configure the firewall for both IPv4 and IPv6 when appropriate. Next, you’ll adjust default rules for connections to your firewall.


# Setting Up UFW Defaults


You can improve your firewall’s efficiency by defining default rules for allowing and denying connections. UFW’s default is to deny all incoming connections and allow all outgoing connections. This means anyone trying to reach your server would not be able to connect, while any application within the server is able to connect externally. To update the default rules set by UFW, first address the incoming connections rule:


```
sudo ufw default deny incoming


```


```
OutputDefault incoming policy changed to 'deny'
(be sure to update your rules accordingly)

```


Next, address the outgoing connections rule:


```
sudo ufw default allow outgoing


```


```
OutputDefault outgoing policy changed to 'allow'
(be sure to update your rules accordingly)

```



Note: If you want to be more restrictive, you can deny all outgoing requests. This option is based on personal preference. For example, if you have a public-facing cloud server, it could help prevent any kind of remote shell connections. Although, it does make your firewall more cumbersome to manage because you’ll have to set up rules for all outgoing connections as well. You can set this as the default with the following:
sudo ufw default deny outgoing



# Allowing Connections to the Firewall


Allowing connections requires changing the firewall rules, which you can do by issuing commands in the terminal. If you turned on your firewall now, for example, it would deny all incoming connections. If you’re connected over SSH to your server, this would be a problem because you would be locked out of your server. Prevent this from happening by enabling SSH connections to your server:


```
sudo ufw allow ssh


```


If your changes were successful, you’ll receive the following output:


```
OutputRule added
Rule added (v6)

```


UFW comes with some defaults such as the ssh command used in the previous example. Alternatively, you can allow incoming connections to port 22/tcp, which uses Transmission Control Protocol (TCP) to accomplish the same thing:


```
 sudo ufw allow 22/tcp


```


If you tried this after you’ve already run allow ssh, however, you’ll receive the following message since the rule already exists:


```
OutputSkipping adding existing rule
Skipping adding existing rule (v6)

```


If your SSH server is running on port 2222, you could allow connections with the same syntax, but replace it with port 2222. Please note that if you use the port number by itself, it effects tcp and udp as well:


```
sudo ufw allow 2222/tcp


```


```
OutputRule added
Rule added (v6)

```


## Securing Web Servers


To secure a web server with File Transfer Protocol (FTP) access, you’ll need to allow connections for port 80/tcp.


Allowing connections for port 80 is useful for web servers such as Apache and Nginx that listen to HTTP connection requests. To do this, allow connections to port 80/tcp:


```
sudo ufw allow 80/tcp


```


UFW typically provides the profiles with the rules required for the web server to function. If not, the web server profiles may be stored as “WWW” and open as ftp or tcp, as in the following examples:


```
sudo ufw allow www


```


You can also use ftp or port 21 to allow for FTP connections:


```
sudo ufw allow ftp


```


```
sudo ufw allow 21/tcp


```


For FTP connections, you also need to allow connections for port 20:


```
sudo ufw allow 20/tcp


```


Your adjustments will depend on what ports and services you need to open, and testing may be necessary. Remember to leave your SSH connection allowed as well.


## Specifying Port Ranges


You can also specify ranges of ports to allow or deny with UFW. To do this, you must first specify the port at the low end of the range, follow that with a colon (:), and then follow that with the high end of the range. Lastly, you must specify which protocol (either tcp or udp) you want the rules to apply to.


For example, the following command will allow TCP access to every port from 1000 to 2000, inclusive:


```
sudo ufw allow 1000:2000/tcp


```


Likewise, the following command will deny UDP connections to every port from 1234 to 4321:


```
 sudo ufw deny 1234:4321/udp


```


## Specifying IP Addresses


You can allow connections from a specific IP address such as in the following. Be sure to replace the IP address with your own information:


```
sudo ufw allow from your_server_ip


```


As these examples demonstrate, you have a lot of flexibility when it comes to adjusting firewall rules by selectively allowing certain ports and IP address connections. Check out ourguide to learn more about allowing incoming connections from a specific IP address or subnet.


# Denying Connections


If you wanted to open up all of your server’s ports — which is not recommended — you could allow all connections and then deny any ports you don’t want to give access to. The following example is how you would deny access to port 80:


```
sudo ufw deny 80/tcp


```


# Deleting Rules


If you want to delete some of the rules you’ve administered, use delete and specify the rule you want to eliminate:


```
sudo ufw delete allow 80/tcp


```


```
OutputRule deleted
Rule deleted (v6)

```


If the rules are long and complex, there’s an alternative two-step approach. First, generate a numbered list of current rules:


```
sudo ufw status numbered


```


Then, with this numbered list, review which rules are currently allowed and delete the rule by referring to its number:


```
sudo ufw delete number


```


```
OutputStatus: active

     To                         Action      From
     --                         ------      ----
[ 1] OpenSSH                    ALLOW IN    Anywhere
[ 2] 22/tcp                     ALLOW IN    Anywhere
[ 3] 2222/tcp                   ALLOW IN    Anywhere
[ 4] 80                         ALLOW IN    Anywhere
[ 5] 20/tcp                     ALLOW IN    Anywhere
…

```


For example, if port 80 is number 4 on the list, you’d use the following syntax. You may also be prompted with a question if you want to proceed with the operation. You can decide yes y or no n:


```
sudo ufw delete 4


```


```
OutputDeleting:
 allow 80
Proceed with operation (y|n)? y
Rule deleted (v6)

```


# Enabling UFW


Once you’ve defined all the rules you want to apply to your firewall, you can enable UFW so it starts enforcing them. If you’re connecting via SSH, make sure to set your SSH port, commonly port 22, to allow connections to be received. Otherwise, you could lock yourself out of your server:


```
sudo ufw enable


```


```
OutputFirewall is active and enabled on system startup

```


To confirm your changes went through, check the status to review the list of rules:


```
sudo ufw status


```


```
OutputStatus: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
22/tcp                     ALLOW       Anywhere
2222/tcp                   ALLOW       Anywhere
20/tcp                     ALLOW       Anywhere
80/tcp                     DENY        Anywhere
…

```


You can also use verbose for a more comprehensive output:


```
sudo ufw status verbose


```


To disable UFW, run the following:


```
sudo ufw disable


```


```
OutputFirewall stopped and disabled on system startup

```


# Resetting Default Settings


If for some reason you need to reset your cloud server’s rules to their default settings, you can do so with the ufw reset command. Please note that you’ll receive a prompt to write y or n before resetting everything since doing so can disrupt existing SSH connections:


```
sudo ufw reset


```


```
OutputResetting all rules to installed defaults. This may disrupt existing ssh
connections. Proceed with operation (y|n)? y
Backing up 'user.rules' to '/etc/ufw/user.rules.20220217_190530'
Backing up 'before.rules' to '/etc/ufw/before.rules.20220217_190530'
Backing up 'after.rules' to '/etc/ufw/after.rules.20220217_190530'
Backing up 'user6.rules' to '/etc/ufw/user6.rules.20220217_190530'
Backing up 'before6.rules' to '/etc/ufw/before6.rules.20220217_190530'
Backing up 'after6.rules' to '/etc/ufw/after6.rules.20220217_190530'

```


Resetting to default settings will disable UFW and delete any rules you previously defined. The default settings, however, will not change to their original settings if you modified them at all. Now you can start fresh with UFW and customize your rules and connections to your preference.


# Conclusion


In this tutorial, you learned how to set up and configure your cloud server to allow for or restrict access to a subset of ports or IP addresses. Additionally, you practiced deleting any rules you no longer want and confirming those changes were accounted for by disabling and then enabling your UFW firewall. Finally, you’ve learned how to reset your UFW firewall back to default settings. To read more about what’s possible with UFW, check out our guide on UFW Essentials: Common Firewall Rules and Commands.


