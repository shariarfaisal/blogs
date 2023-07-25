# How To Create a New Sudo-enabled User on CentOS 8 [Quickstart]

```Linux Basics``` ```CentOS``` ```CentOS 8``` ```Quickstart```

## Introduction


The sudo command provides a mechanism for granting administrator privileges — ordinarily only available to the root user — to normal users. This guide will show you how to create a new user with sudo access on CentOS 8, without having to modify your server’s /etc/sudoers file.



Note: If you want to configure sudo for an existing CentOS user, skip to step 3.

# Step 1 — Logging Into Your Server


SSH in to your server as the root user:


```
ssh root@your_server_ip_address


```


Use your server’s IP address or hostname  in place of your_server_ip_address above.


# Step 2 — Adding a New User to the System


Use the adduser command to add a new user to your system:


```
adduser sammy


```


Be sure to replace sammy with the username you’d like to create.


Use the passwd command to update the new user’s password:


```
passwd sammy


```


Remember to replace sammy with the user that you just created. You will be prompted twice for a new password:


```
OutputChanging password for user sammy.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.

```


# Step 3 — Adding the User to the wheel Group


Use the usermod command to add the user to the wheel group:


```
usermod -aG wheel sammy


```


Once again, be sure to replace sammy with the username you’d like to give sudo priveleges to. By default, on CentOS, all members of the wheel group have full sudo access.


# Step 4 — Testing sudo Access


To test that the new sudo permissions are working, first use the su command to switch from the root user to the new user account:


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


The first time you use sudo in a session, you will be prompted for the password of that user’s account. Enter the password to proceed:


```
Output[sudo] password for sammy:

```



Note: This is not asking for the root password! Enter the password of the sudo-enabled user, not the root password.

If your user is in the proper group and you entered the password correctly, the command that you issued with sudo will run with root privileges.


# Conclusion


In this quickstart tutorial we created a new user account and added it to the wheel group to enable sudo access. For more detailed information on setting up a CentOS 8 server, please read our Initial Server Setup with CentOS 8 tutorial.


