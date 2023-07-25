# Telnet Command Usage in Linux Unix

```UNIX/Linux```

# What is Telnet ?


Telnet is an old network protocol that is used to connect to remote systems over a TCP/IP network. It connects to servers and network equipment over port 23. Let’s take a look at Telnet command usage.


# Disclaimer


1. Telnet is not a secure protocol and is thus NOT RECOMMENDED!. This is because data sent over the protocol is unencrypted and can be intercepted by hackers.
2. Instead of using telnet, a more preferred protocol to use is SSH which is encrypted and more secure

Let’s see how you can install and use the telnet protocol.


# Installing Telnet


In this section, we will walk you through the process of installing telnet in RPM and DEB systems.


## Installation of Telnet in CentOS 7 / RHEL 7


To begin the installation process on the server, run the command


```
# yum install telnet telnet-server -y

```


Sample Output  Next, Start and enable the telnet service by issuing the command below


```
 
# systemctl start telnet.socket
# systemctl enable telnet.socket

```


Sample Output  Next, allow port 23 which is the native port that telnet uses on the firewall.


```
# firewall-cmd --permanent --add-port=23/tcp

```


Finally, reload the firewall for the rule to take effect.


```
# firewall-cmd --reload

```


Sample Output  To verify the status of telnet run


```
# systemctl status telnet.socket

```


Sample Output  Telnet protocol is now ready for use. Next, we are going to create a login user.


## Creating a login user


In this example, we will create a login user for logging in using the telnet protocol.


```
# adduser telnetuser

```


Create a password for the user.


```
# passwd telnetuser

```


Specify the password and confirm. To use telnet command to log in to a server, use the syntax below.


```
$ telnet server-IP address

```


For example


```
$ telnet 38.76.11.19

```


In the black console, specify the username and password.  To login using putty, enter the server’s IP address and click on the ‘Telnet’ radio button as shown.  Finally, click on the ‘Open’ button. On the console screen, provide the username and password of the user 


## Installation of Telnet in Ubuntu 18.04


To install telnet protocol in Ubuntu 18.04 execute:


```
$ sudo apt install telnetd -y

```


Sample Output  To check whether telnet service is running, execute the command.


```
$ systemctl status inetd

```


Sample Output  Next, we need to open port 23 in ufw firewall.


```
$ ufw allow 23/tcp

```


Sample Output  Finally, reload the firewall to effect the changes.


```
$ ufw reload

```


 Telnet has been successfully installed and ready for use. Like in the previous example in CentOS 7, you need to create a login user and log in using the same syntax.


# Using telnet to check for open ports


Telnet can also be used to check if a specific port is open on a server. To do so, use the syntax below.


```
$ telnet server-IP port

```


For example, to check if port 22 is open on a server, run


```
$ telnet 38.76.11.19  22

```


Sample Output 


# Summary


This tutorial is an educational guide that shows you how to use telnet protocol. We HIGHLY DISCOURAGE the use of telnet due to the high-security risks it poses due to lack of encryption. SSH is the recommended protocol when connecting to remote systems. The data sent over SSH is encrypted and kept safe from hackers.


