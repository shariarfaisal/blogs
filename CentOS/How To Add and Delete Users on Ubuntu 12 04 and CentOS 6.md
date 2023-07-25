# How To Add and Delete Users on Ubuntu 12 04 and CentOS 6

```Linux Basics``` ```Ubuntu``` ```CentOS```











# Status: Deprecated


This article covers a versions of Ubuntu and CentOS that are no longer supported.  If you currently operate a server running Ubuntu 12.04 or CentOS 6, we highly recommend upgrading or migrating to a supported version.


Reason:
Ubuntu 12.04 reached end of life (EOL) on April 28, 2017, CentOS 6 reached end of life (EOL) on November 30th, 2020, and neither of these operating system versions will receive future security patches or updates. For these reasons, this guide is no longer maintained.


See Instead:
This guide might still be useful as a reference, but may not work on other Ubuntu or CentOS releases.  If available, we strongly recommend using a guide written for the version of Ubuntu or CentOS you are using. You can find our collection of tutorials on how to add and delete users here.


## What the Red  Means


The lines that the user needs to enter or customize will be in red in this tutorial! The rest should mostly be copy-and-pastable.


When you log into a new freshly spun up droplet, you are accessing it from the root user.  Although this gives you the power to make any changes you need on the server,  you are much better off creating another new user with root privileges on the virtual private server. Additionally, if other people will be accessing the virtual server, you will need to make new users for them as well. This tutorial will go over creating a new user, granting them root privileges, and deleting users.


When you perform any root tasks with the new user, you will need to use the phrase “sudo” before the command. This is a helpful command for 2 reasons: 1) it prevents the user making any system-destroying mistakes 2) it stores all the commands run with sudo to a file where  can be reviewed later if needed. Keep in mind however, that this user is as powerful as the root user. If you only need a user for a limited number of tasks on the VPS, you do not need to give them root privileges.


# Setup


This tutorial requires access to the root user or a user with sudo privileges.


You should have received your root password from the welcome email after you launched your droplet.


# Users on Ubuntu 12.04


## How to Add a User on Ubuntu 12.04


To add a new user in Ubuntu, use the adduser command, replacing the “newuser” with your preferred username.


```
sudo adduser newuser
```


As soon as you type this command, Ubuntu will automatically start the process:


- Type in and confirm your password
- Enter in the user’s information. This is not required, pressing enter will automatically fill in the field with the default information
- Press Y (or enter) when Ubuntu asks you if the information is correct

Congratulations—you have just added a new user. You can log out of the root user by typing exit and then logging back in with the new username and password.


## How to Grant a User Root Privileges 


As mentioned earlier, you are much better off using a user with root privileges.


You can create the sudo user by opening the sudoers file with this command:


```
sudo /usr/sbin/visudo
```


Adding the user’s name and the same permissions as root under the the user privilege specification will grant them the sudo privileges.


```
# User privilege specification
root    ALL=(ALL:ALL) ALL 
newuser	ALL=(ALL:ALL) ALL
```


Press ‘cntrl x’ to exit the file and then ‘Y’ to save it.


## How to Delete a User


Should you find that you  find that you no longer want to have a specific user on the virtual private server you can delete them with a single command.


```
sudo userdel newuser
```


Finish up by the deleting the user’s home directory:


```
 sudo rm -rf /home/newuser
```


# Users on CentOS 6


## How to Add a User on CentOS 6


To add a new user in CentOS, use the adduser command, replacing the “newuser” with your preferred username.


```
sudo adduser newuser
```


Follow up by providing the user with a new password, typing and confirming the new password when prompted:


```
sudo passwd newuser
```


Congratulations—you have just added a new user and their password. You can log out of the root user by typing exit and then logging back in with the new username and password.


## How to Grant a User Root Privileges 


As mentioned earlier, you are much better off using a user with root privileges.


You can create the sudo user by opening the sudoers file with this command:


```
sudo /usr/sbin/visudo
```


You will find the section to make the user privilege modifications at the bottom of the file. Type “a” to start inserting text. Adding the user’s name and the same permissions as root under the the user privilege specification will grant them the sudo privileges.


```
## Allow root to run any commands anywhere
root    ALL=(ALL)       ALL
newuser ALL=(ALL)       ALL
```


Save and Exit the file by press “shift” ZZ.


## How to Delete a User


Should you find that you  find that you no longer want to have a specific user on the virtual private server you can delete them with a single command.


```
sudo userdel newuser
```


You can add the flag “-r” to the command if you would like to simultaneously remove the users’s home directory and files.


```
sudo userdel -r newuser
```


# Next Steps


Once you have set up the users will you need, you can start building up your VPS. A good place to start is to install the LAMP stack (a collection of basic web server software) on your droplet, using the tutorials below.


LAMP on Ubuntu 12.04 


LAMP on CentOS 6


LEMP on Ubuntu 12.04 


LEMP on CentOS 6


By Etel Sverdlov
