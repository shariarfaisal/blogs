# How To Add and Delete Users on CentOS 8

```Linux Basics``` ```CentOS``` ```CentOS 8``` ```Getting Started```

## Introduction


When you first start using a fresh Linux server, adding and removing users is often one of first things you’ll need to do. In this guide, we will cover how to create user accounts, assign sudo privileges, and delete users on a CentOS 8 server.


# Prerequisites


This tutorial assumes you are logged into a CentOS 8 server with a non-root sudo-enabled user. If you are logged in as root instead, you can drop the sudo portion of all the following commands, but they will work either way.


# Adding Users


Throughout this tutorial we will be working with the user sammy. Please susbtitute with the username of your choice.


You can add a new user by typing:


```
sudo adduser sammy


```


Next, you’ll need to give your user a password so that they can log in. To do so, use the passwd command:


```
sudo passwd sammy


```


You will be prompted to type in the password twice to confirm it. Now your new user is set up and ready for use!



Note: if your SSH server disallows password-based authentication, you will not yet be able to connect with your new username. Details on setting up key-based SSH authentication for the new user can be found in step 5 of Initial Server Setup with CentOS 8.

# Granting Sudo Privileges to a User


If your new user should have the ability to execute commands with root (administrative) privileges, you will need to give them access to sudo.


We can do this by adding the user to the wheel group (which gives sudo access to all of its members by default).


Use the usermod command to add your user to the wheel group:


```
sudo usermod -aG wheel sammy


```


Now your new user is able to execute commands with administrative privileges. To do so, append sudo ahead of the command that you want to execute as an administrator:


```
sudo some_command


```


You will be prompted to enter the password of the your user account (not the root password). Once the correct password has been submitted, the command you entered will be executed with root privileges.


## Managing Users with Sudo Privileges


While you can add and remove users from a group with usermod, the command doesn’t have a way to show which users are members of a group.


To see which users are part of the wheel group (and thus have sudo privileges), you can use the lid command. lid is normally used to show which groups a user belongs to, but with the -g flag, you can reverse it and show which users belong in a group:


```
sudo lid -g wheel


```


```
Output centos(uid=1000)
 sammy(uid=1001)

```


The output will show you the usernames and UIDs that are associated with the group. This is a good way of confirming that your previous commands were successful, and that the user has the privileges that they need.


# Deleting Users


If you have a user account that you no longer need, it’s best to delete it.


To delete the user without deleting any of their files, use the userdel command:


```
sudo userdel sammy


```


If you want to delete the user’s home directory along with their account, add the -r flag to userdel:


```
sudo userdel -r sammy


```


With either command, the user will automatically be removed from any groups that they were added to, including the wheel group if applicable. If you later add another user with the same name, they will have to be added to the wheel group again to gain sudo access.


# Conclusion


You should now have a good grasp on how to add and remove users from your CentOS 8 server. Effective user management will allow you to separate users and give them only the access that is needed for them to do their job.


You can now move on to configuring your CentOS 8 server for whatever software you need, such as a LAMP or LEMP web stack.


