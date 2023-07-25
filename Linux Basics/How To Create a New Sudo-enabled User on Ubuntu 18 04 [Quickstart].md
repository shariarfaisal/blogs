# How To Create a New Sudo-enabled User on Ubuntu 18 04 [Quickstart]

```Linux Basics``` ```Ubuntu``` ```Quickstart``` ```Ubuntu 18.04```

## Introduction


The sudo command provides a mechanism for granting administrator privileges — ordinarily only available to the root user — to normal users. This guide will show you how to create a new user with sudo access on Ubuntu 18.04, without having to modify your server’s /etc/sudoers file. If you want to configure sudo for an existing user, skip to step 3.


# Step 1 — Logging Into Your Server


SSH in to your server as the root user:


```
ssh root@your_server_ip_address


```


# Step 2 — Adding a New User to the System


Use the adduser command to add a new user to your system:


```
adduser sammy


```


Be sure to replace sammy with the user name that you want to create. You will be prompted to create and verify a password for the user:


```
OutputEnter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully

```


Next you’ll be asked to fill in some information about the new user. It is fine to accept the defaults and leave all of this information blank:


```
OutputChanging the user information for sammy
Enter the new value, or press ENTER for the default
    Full Name []:
    Room Number []:
    Work Phone []:
    Home Phone []:
    Other []:
Is the information correct? [Y/n]

```


# Step 3 — Adding the User to the sudo Group


Use the usermod command to add the user to the sudo group:


```
usermod -aG sudo sammy


```


Again, be sure to replace sammy with the username you just added. By default, on Ubuntu, all members of the sudo group have full sudo privileges.


# Step 4 — Testing sudo Access


To test that the new sudo permissions are working, first use the su command to switch to the new user account:


```
su - sammy


```


As the new user, verify that you can use sudo by prepending sudo to the command that you want to run with superuser privileges:


```
sudo command_to_run


```


For example, you can list the contents of the /root directory, which is normally only accessible to the root user:


```
sudo ls -la /root


```


The first time you use sudo in a session, you will be prompted for the password of that users account. Enter the password to proceed:


```
Output:[sudo] password for sammy:

```



Note: This is not asking for the root password! Enter the password of the sudo-enabled user, not a root password.

If your user is in the proper group and you entered the password correctly, the command that you issued with sudo will run with root privileges.


# Conclusion


In this quickstart tutorial we created a new user account and added it to the sudo group to enable sudo access. For more detailed information on setting up an Ubuntu 18.04 server, please read our Initial Server Setup with Ubuntu 18.04 tutorial.


