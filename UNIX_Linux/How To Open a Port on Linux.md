# How To Open a Port on Linux

```UNIX/Linux```

## Introduction


A port is a communication endpoint. Within an operating system, a port is opened or closed to data packets for specific processes or network services.


Typically, ports identify a specific network service assigned to them. This can be changed by manually configuring the service to use a different port, but in general, the defaults can be used.


The first 1024 ports (port numbers 0 to 1023) are referred to as well-known port numbers and are reserved for the most commonly used services. These include SSH (port 22), HTTP (port 80), HTTPS (port 443).


Port numbers above 1024 are referred to as ephemeral ports.


- Port numbers 1024 to 49151 are called the registered/user ports.
- Port numbers 49152 to 65535 are called the dynamic/private ports.

In this tutorial, you will open an ephemeral port on Linux, since the most common services use the well-known ports.


# Prerequisites


To complete this tutorial, you will need:


- Familiarity with using the terminal.

# List All Open Ports


Before opening a port on Linux, you must check the list of all open ports, and choose an ephemeral port to open that is not on that list.


Use the netstat command to list all open ports, including TCP and UDP, which are the most common protocols for packet transmission in the network layer.


```
netstat -lntu


```


This will print:


- all listening sockets (-l)
- the port number (-n)
- TCP ports (-t)
- UDP ports (-u)

```
OutputActive Internet connections (only servers)
Proto Recv-Q Send-Q Local Address    Foreign Address  State
tcp        0      0 127.0.0.1:5432   0.0.0.0:*        LISTEN
tcp        0      0 127.0.0.1:27017  0.0.0.0:*        LISTEN
tcp        0      0 127.0.0.1:6379   0.0.0.0:*        LISTEN
tcp        0      0 127.0.0.53:53    0.0.0.0:*        LISTEN
tcp        0      0 0.0.0.0:22       0.0.0.0:*        LISTEN
tcp6       0      0 ::1:5432         :::*             LISTEN
tcp6       0      0 ::1:6379         :::*             LISTEN
tcp6       0      0 :::22            :::*             LISTEN
udp        0      0 127.0.0.53:53    0.0.0.0:*        LISTEN

```



Note: If your distribution doesn’t have netstat, you can use the ss command to display open ports by checking for listening sockets.

Verify that you are receiving consistent outputs using the ss command to list listening sockets with an open port:


```
ss -lntu


```


This will print:


```
OutputNetid  State   Recv-Q  Send-Q    Local Address:Port   Peer Address:Port
udp    UNCONN  0       0         127.0.0.53%lo:53          0.0.0.0:*
tcp    LISTEN  0       128           127.0.0.1:5432        0.0.0.0:*
tcp    LISTEN  0       128           127.0.0.1:27017       0.0.0.0:*
tcp    LISTEN  0       128           127.0.0.1:6379        0.0.0.0:*
tcp    LISTEN  0       128       127.0.0.53%lo:53          0.0.0.0:*
tcp    LISTEN  0       128             0.0.0.0:22          0.0.0.0:*
tcp    LISTEN  0       128               [::1]:5432        0.0.0.0:*
tcp    LISTEN  0       128               [::1]:6379        0.0.0.0:*
tcp    LISTEN  0       128                [::]:22          0.0.0.0:*

```


This gives more or less the same open ports as netstat.


# Opening a Port on Linux to Allow TCP Connections


Now, open a closed port and make it listen for TCP connections.


For the purposes of this tutorial, you will be opening port 4000. However, if that port is not open in your system, feel free to choose another closed port. Just make sure that it’s greater than 1023.


Ensure that port 4000 is not used using the netstat command:


```
netstat -na | grep :4000


```


Or the ss command:


```
ss -na | grep :4000


```


The output must remain blank, thus verifying that it is not currently used, so that you can add the port rules manually to the system iptables firewall.


## For Ubuntu Users and ufw-based Systems


Use ufw - the command line client for the UncomplicatedFirewall.


Your commands will resemble:


```
sudo ufw allow 4000


```


Refer to How to Setup a ufw Firewall Setup for your distribution.



Note:

Ubuntu 14.0.4: “Allow Specific Port Ranges”
Ubuntu 16.0.4/18.0.4/20.0.4/22.0.4: “Allowing Other Connections / Specific Port Ranges”
Debian 9/10/11: “Allowing Other Connections / Specific Port Ranges”


## For CentOS and firewalld-based Systems


Use firewall-cmd - the command line client for the firewalld daemon.


Your commands will resemble:


```
firewall-cmd --add-port=4000/tcp


```


Refer to How to Set Up firewalld for your distribution.



Note:

CentOS 7/8: “Setting Rules for your Applications / Opening a Port for your Zones”
Rocky Linux 8/9: “Setting Rules for your Applications / Opening a Port for your Zones”


## For Other Linux Distributions


Use iptables to change the system IPv4 packet filter rules.


```
iptables -A INPUT -p tcp --dport 4000 -j ACCEPT


```


Refer to How To Set Up A Firewall Using iptables for your distribution.



Note:

Ubuntu 12.04: “A Basic Firewall”
Ubuntu 14.04: “Accept Other Necessary Connections”


# Test the Newly Opened Port for TCP Connections


Now that you have successfully opened a new TCP port, it is time to test it.


First, start netcat (nc) and listen (-l) on port (-p) 4000, while sending the output of ls to any connected client:


```
ls | nc -l -p 4000


```


Now, after a client has opened a TCP connection on port 4000, they will receive the output of ls. Leave this session alone for now.


Open another terminal session on the same machine.


Since you opened a TCP port, use telnet to check for TCP Connectivity. If the command doesn’t exist, install it using your package manager.


Input your server IP and the port number (4000 in this example) and run this command:


```
telnet localhost 4000


```


This command tries to open a TCP connection on localhost on port 4000.


You’ll get an output similar to this, indicating that a connection has been established with the listening program (nc):


```
OutputTrying ::1...
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
while.sh

```


The output of ls (while.sh, in this example) has also been sent to the client, indicating a successful TCP Connection.


Use nmap to check if the port (-p) is open:


```
nmap localhost -p 4000


```


This command will check the open port:


```
OutputStarting Nmap 7.60 ( https://nmap.org ) at 2020-01-18 21:51 UTC
Nmap scan report for localhost (127.0.0.1)
Host is up (0.00010s latency).
Other addresses for localhost (not scanned): ::1

PORT     STATE SERVICE
4000/tcp open  remoteanything

Nmap done: 1 IP address (1 host up) scanned in 0.25 seconds

```


The port has been opened. You have successfully opened a new port on your Linux system.



Note: nmap only lists opened ports that have a currently listening application. If you don’t use any listening application, such as netcat, this will display the port 4000 as closed since there isn’t any application listening on that port currently. Similarly, telnet won’t work either since it also needs a listening application to bind to. This is the reason why nc is such a useful tool. This simulates such environments in a simple command.

But this is only temporary, as the changes will be reset every time you reboot the system.


# Persisting Rules


The approach presented in this article will only temporarily update the firewall rules until the system shuts down or reboots. So similar steps must be repeated to open the same port again after a restart.


## For ufw Firewall


ufw rules do not reset on reboot. This is because it is integrated into the boot process, and the kernel saves the firewall rules using ufw by applying appropriate config files.


## For firewalld


You will need to apply the --permanent flag.


Refer to How to Set Up firewalld for your distribution.



Note:

CentOS 7/8: “Setting Rules for your Applications”
Rocky Linux 8/9: “Setting Rules for your Applications”


## For iptables


You will need to save the configuration rules. These tutorials recommend iptables-persistent.


Refer to How To Set Up a Firewall Using iptables for your distribution.



Note:

Ubuntu 12.04: “Saving Iptables Rules”
Ubuntu 14.04: “Saving your Iptables Configuration”


# Conclusion


In this tutorial, you learned how to open a new port on Linux and set it up for incoming connections. You also used netstat, ss, telnet, nc, and nmap.


Continue your learning with How the Iptables Firewall Works, A Deep Dive into Iptables and Netfilter Architecture, Understanding Sockets, and How To Use Top, Netstat, Du, & Other Tools to Monitor Server Resources.


