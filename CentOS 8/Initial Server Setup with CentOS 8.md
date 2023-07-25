# Initial Server Setup with CentOS 8

```Linux Basics``` ```CentOS``` ```Getting Started``` ```Initial Server Setup``` ```CentOS 8```

## Introduction


When you first create a new CentOS 8 server, there are a few configuration steps that you should take early on as part of the basic setup. This will increase the security and usability of your server and will give you a solid foundation for subsequent actions.


# Step 1 — Logging in as Root


To log into your server, you will need to know your server’s public IP address.  You will also need the password or, if you installed an SSH key for authentication, the private key for the root user’s account. If you have not already logged into your server, you may want to follow our documentation on how to connect to your Droplet with SSH, which covers this process in detail.


If you are not already connected to your server, log in as the root user now using the following command (substitute the highlighted portion of the command with your server’s public IP address):


```
ssh root@your_server_ip


```


Accept the warning about host authenticity if it appears. If you are using password authentication, provide your root password to log in. If you are using an SSH key that is passphrase protected, you may be prompted to enter the passphrase the first time you use the key each session. If this is your first time logging into the server with a password, you may also be prompted to change the root password.


## About Root


The root user is the administrative user in a Linux environment, and it has very broad privileges. Because of the heightened privileges of the root account, you are discouraged from using it on a regular basis. This is because part of the power inherent with the root account is the ability to make very destructive changes, even by accident.


As such, the next step is to set up an alternative user account with a reduced scope of influence for day-to-day work. This account will still be able to gain increased privileges when necessary.


# Step 2 — Creating a New User


Once you are logged in as root, you can create the new user account that we will use to log in from now on.


This example creates a new user called sammy, but you should replace it with any username that you prefer:


```
adduser sammy


```


Next, set a strong password for the sammy user:


```
passwd sammy


```


You will be prompted to enter the password twice. After doing so, your user will be ready to use, but first we’ll give this user additional privileges to use the sudo command. This will allow us to run commands as root when necessary.


# Step 3 — Granting Administrative Privileges


Now, we have a new user account with regular account privileges. However, we may sometimes need to do administrative tasks.


To avoid having to log out of our normal user and log back in as the root account, we can set up what is known as “superuser” or root privileges for our normal account. This will allow our normal user to run commands with administrative privileges by putting the word sudo before each command.


To add these privileges to our new user, we need to add the new user to the wheel group. By default, on CentOS 8, users who belong to the wheel group are allowed to use the sudo command.


As root, run this command to add your new user to the wheel group (substitute the highlighted word with your new username):


```
usermod -aG wheel sammy


```


Now, when logged in as your regular user, you can type sudo before commands to perform actions with superuser privileges.


# Step 4 — Setting Up a Basic Firewall


Firewalls provide a basic level of security for your server. These applications are responsible for denying traffic to every port on your server, except for those ports/services you have explicitly approved. CentOS has a service called firewalld to perform this function. A tool called firewall-cmd is used to configure firewalld firewall policies.



Note: If your servers are running on DigitalOcean, you can optionally use DigitalOcean Cloud Firewalls instead of firewalld. We recommend using only one firewall at a time to avoid conflicting rules that may be difficult to debug.

First install firewalld:


```
dnf install firewalld -y


```


The default firewalld configuration allows ssh connections, so we can turn the firewall on immediately:


```
systemctl start firewalld


```


Check the status of the service to make sure it started:


```
systemctl status firewalld


```


```
Output● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2020-02-06 16:39:40 UTC; 3s ago
     Docs: man:firewalld(1)
 Main PID: 13180 (firewalld)
    Tasks: 2 (limit: 5059)
   Memory: 22.4M
   CGroup: /system.slice/firewalld.service
           └─13180 /usr/libexec/platform-python -s /usr/sbin/firewalld --nofork --nopid

```


Note that it is both active and enabled, meaning it will start by default if the server is rebooted.


Now that the service is up and running, we can use the firewall-cmd utility to get and set policy information for the firewall.


First let’s list which services are already allowed:


```
firewall-cmd --permanent --list-all


```


```
Outputpublic (active)
  target: default
  icmp-block-inversion: no
  interfaces: eth0 eth1
  sources:
  services: cockpit dhcpv6-client ssh
  ports:
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:

```


To see the additional services that you can enable by name, type:


```
firewall-cmd --get-services


```


To add a service that should be allowed, use the --add-service flag:


```
firewall-cmd --permanent --add-service=http


```


This would add the http service and allow incoming TCP traffic to port 80. The configuration will update after you reload the firewall:


```
firewall-cmd --reload


```


Remember that you will have to explicitly open the firewall (with services or ports) for any additional services that you may configure later.


# Step 5 — Enabling External Access for Your Regular User


Now that we have a regular non-root user for daily use, we need to make sure we can use it to SSH into our server.



Note: Until verifying that you can log in and use sudo with your new user, we recommend staying logged in as root. This way, if you have problems, you can troubleshoot and make any necessary changes as root. If you are using a DigitalOcean Droplet and experience problems with your root SSH connection, you can log into the Droplet using the DigitalOcean Console.

The process for configuring SSH access for your new user depends on whether your server’s root account uses a password or SSH keys for authentication.


## If the Root Account Uses Password Authentication


If you logged in to your root account using a password, then password authentication is enabled for SSH. You can SSH to your new user account by opening up a new terminal session and using SSH with your new username:


```
ssh sammy@your_server_ip


```


After entering your regular user’s password, you will be logged in. Remember, if you need to run a command with administrative privileges, type sudo before it like this:


```
sudo command_to_run


```


You will be prompted for your regular user password when using sudo for the first time each session (and periodically afterwards).


To enhance your server’s security, we strongly recommend setting up SSH keys instead of using password authentication. Follow our guide on setting up SSH keys on CentOS 8 to learn how to configure key-based authentication.


## If the Root Account Uses SSH Key Authentication


If you logged in to your root account using SSH keys, then password authentication is disabled for SSH. You will need to add a copy of your public key to the new user’s ~/.ssh/authorized_keys file to log in successfully.


Since your public key is already in the root account’s ~/.ssh/authorized_keys file on the server, we can copy that file and directory structure to our new user account.


The simplest way to copy the files with the correct ownership and permissions is with the rsync command. This will copy the root user’s .ssh directory, preserve the permissions, and modify the file owners, all in a single command. Make sure to change the highlighted portions of the command below to match your regular user’s name:



Note: The rsync command treats sources and destinations that end with a trailing slash differently than those without a trailing slash.  When using rsync below, be sure that the source directory (~/.ssh) does not include a trailing slash (check to make sure you are not using ~/.ssh/).
If you accidentally add a trailing slash to the command, rsync will copy the contents of the root account’s ~/.ssh directory to the sudo user’s home directory instead of copying the entire ~/.ssh directory structure.  The files will be in the wrong location and SSH will not be able to find and use them.

```
rsync --archive --chown=sammy:sammy ~/.ssh /home/sammy


```


Now, back in a new terminal on your local machine, open up a new SSH session with your non-root user:


```
ssh sammy@your_server_ip


```


You should be logged in to the new user account without using a password. Remember, if you need to run a command with administrative privileges, type sudo before it like this:


```
sudo command_to_run


```


You will be prompted for your regular user password when using sudo for the first time each session (and periodically afterwards).


# Conclusion


At this point, you have a solid foundation for your server. You can install any of the software you need on your server now.


