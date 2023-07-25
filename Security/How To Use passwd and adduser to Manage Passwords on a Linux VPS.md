# How To Use passwd and adduser to Manage Passwords on a Linux VPS

```Linux Basics``` ```Security```

## Introduction


Passwords and authentication are concepts that every user must deal with when working in a Linux environment.  These topics span a number of different configuration files and tools.


In this guide, we will explore some basic files, like "/etc/passwd" and "/etc/shadow", as well as tools for configuring authentication, like the aptly-named "passwd" command and "adduser".


We will be using an Ubuntu 12.04 VPS to discuss these topics, but any modern Linux distribution should function in a similar way.


# What Is the "/etc/passwd" File?


The first file we will look at, called the "/etc/passwd" file, does not actually store passwords.


At one time, this file stored the hashed passwords of every user on the system.  However, this responsibility has been moved to a separate file for security reasons.


Let's look at what is in the "/etc/passwd" file:


```
less /etc/passwd
```


```
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/bin/sh
bin:x:2:2:bin:/bin:/bin/sh
sys:x:3:3:sys:/dev:/bin/sh
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/bin/sh
man:x:6:12:man:/var/cache/man:/bin/sh
. . .
```


The first thing to note is that this file is accessible by unprivileged users.


Everyone on the system has read privileges to this file.  This is why password information was moved out of this file.


Let's look at the format of the file.


## How To Read the "/etc/passwd" File


Each line in the file contains the login information of a single user on the system.  Some of these users might be created for use by daemons and background services.


Take a look at a single line to see what information it contains:


```
root:x:0:0:root:/root:/bin/bash
```


The fields of information are separated by a colon (:) character.  There are seven fields on each line in a typical Linux "/etc/passwd" file:


1. root: Account username.

As you add user accounts using commands like "adduser" and "useradd", or as you install more services, this file will grow.  New username information will be added to the bottom of this file.


In most cases, you should not have to edit this file by hand.  There are tools that manipulate this file and ensure that the proper syntax is maintained.


# What Is the "/etc/shadow" File?


The actual password data is stored in a file called "/etc/shadow".


This doesn't actually contain passwords in plain text.  Instead, it uses a key derivation function to create a hash.  This is what it stores in the file.


A key derivation function is basically an algorithm that will always create a certain hash when given the same input.  The same algorithm is run on the password that is given during authentication and this value is compared to the value in this file.


Note that this file, unlike the "/etc/passwd" file, is not readable by unprivileged users.


The root user has read and write permissions, and the "shadow" group, which contains users needed for authentication, has read permissions.


## How To Read the "/etc/shadow" File


Open the "/etc/shadow" file by typing:


```
sudo less /etc/shadow
```


```
root:$6$mJD3Rsj4$xUa7jru6EEGTXnhwTfTT26/j8M5XiQvUl6UH32cfAWT/6W9iSI5IuIw5OOw4khwrsOHPyMwfCLyayfYiVdhAq0:15952:0:99999:7:::
daemon:*:15455:0:99999:7:::
bin:*:15455:0:99999:7:::
sys:*:15455:0:99999:7:::
sync:*:15455:0:99999:7:::
games:*:15455:0:99999:7:::
man:*:15455:0:99999:7:::
. . .
```


Like the "/etc/passwd" file, each line defines a user's information and each field is delimited by a colon (:) character.


Note: The asterisk (*) value in the second field on some of the above lines means that the account cannot log in.  This is mainly used for services and is intended behavior.


Let's take a look at a single line again:


```
daemon:*:15455:0:99999:7:::
```


These are fields defined in the "/etc/shadow" file:


1. daemon: Account username.

# How Do You Change Passwords?


Users' passwords can be modified by issuing the "passwd" command.


By default, this command changes the current user's password and does not require special permissions.


```
passwd
```


If you would like to change another user's password, you will need administrative privileges.  The following syntax can be used:


```
sudo passwd username
```


You will be prompted for your password for the "sudo" command, and then you will be told to enter and confirm the new password you would like to use.


If you compare the hashed value in the "/etc/shadow" file, you will see that it changes after you issue the passwd command.


# How Do You Create New Users?


Users can be created using a few different commands.


The easiest method is probably with the "adduser" command, which we will cover here.  On Ubuntu systems, this is linked to a perl script that handles appropriate user creation.


You can call it like so:


```
adduser demo
```


```
Adding user `demo' ...
Adding new group `demo' (1000) ...
Adding new user `demo' (1000) with group `demo' ...
Creating home directory `/home/demo' ...
Copying files from `/etc/skel' ...
Enter new UNIX password: 
Retype new UNIX password: 
passwd: password updated successfully
Changing the user information for demo
Enter the new value, or press ENTER for the default
	Full Name []: test
	Room Number []: room
	Work Phone []: work phone
	Home Phone []: home phone
	Other []: other
Is the information correct? [Y/n]
```


You will be asked a series of questions that will help fill in the information in the "/etc/passwd" file and the "/etc/shadow" file.


We can see what entry it added to the "/etc/passwd" file by entering:


```
tail -1 /etc/passwd
```


```
demo:x:1000:1000:test,room,work phoneme phone,other:/home/demo:/bin/bash
```


You can see that this takes great advantage of the comments field.  The other fields are filled in as expected.


We can run a similar command to see the modifications made to the "/etc/shadow" file:


```
sudo tail -1 /etc/shadow
```


```
demo:$6$XvPCmWr4$HXWmaGSeU5SrKwK2ouAjc68SxbJgUQkQ.Fco9eTOex8232S7weBfr/CMHQkullQRLyJtCAD6rw5TVOXk39NAo/:15952:0:99999:7:::
```


# Conclusion


Using these simple tools, you can change the login information on your system.


It is important to test your ability to log in after making any changes.  It is also essential to keep the permissions on the authentication files the same to maintain functionality and security.


By Justin Ellingwood
