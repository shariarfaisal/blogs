# How To Install and Configure scponly on CentOS 7

```Security``` ```System Tools``` ```CentOS```

## Introduction


scponly is a secure alternative to anonymous FTP.  It gives the administrator the ability to setup a secure user account with restricted remote file access and without access to an interactive shell.


Why Use scponly Instead of Normal SSH? With scponly you are giving the user remote access to download and upload specific files.  They will not have an interactive shell, meaning they can’t execute commands.  The user can only access the server via scp, sftp, or clients that support these protocols.  From a security perspective, this lowers your attack surface by limiting unneeded access to an interactive shell on a server.


# Prerequisites


For this tutorial, you will need a fresh CentOS 6 or 7 Droplet.


All the commands in this tutorial should be run as a non-root user. If root access is required for the command, it will be preceded by sudo. If you don’t already have that set up, follow this tutorial: Initial Server Setup on CentOS 6 or Initial Server Setup for CentOS 7.


# Step 1 —  Install Packages


scponly is available in some third party repositories, but these builds of scponly are outdated and are missing some of the features we will be adding when we build scponly from source.


To build scponly from source you will need to install the following 5 packages:


- wget (To download files via the command line)
- gcc (To compile scponly from source)
- man (To read man pages)
- rsync (To provide advanced file copying)
- openssh-client-tools (To provide various ssh tools)

We will use yum to install the prerequisite packages needed to build scponly.  During the yum install we will pass the required package names as well as -y which automatically answers yes to any prompts.


Install wget, gcc, man, rsync, and openssh-clients using the yum install command:


```
sudo yum install wget gcc man rsync openssh-clients -y


```


# Step 2 — Download and Extract scponly


In this section we will be downloading the latest build of scponly from sourceforge using wget and extracting the files using tar.


Before downloading scponly, change to the /opt directory. This directory is usually designated for optional software.


```
cd /opt


```


As of this article the latest snapshot of scponly is 2011.05.26.  You can check the Sourceforge page for a later release and adjust the wget command accordingly.


Download the scponly source using wget:


```
sudo wget http://sourceforge.net/projects/scponly/files/scponly-snapshots/scponly-20110526.tgz

```


Extract the scponly source code:


```
sudo tar -zxvf scponly-20110526.tgz


```


# Step 3 — Build and Install scponly


In this section we will use 3 main commands to build scponly: configure, make, and make install.  These are the 3 commands most often used when you are downloading and installing software from source code.


Change to the directory that contains the scponly source code you just uncompressed:


```
cd /opt/scponly-20110526


```


First, run the configure command to build a makefile with all the features you want enabled or disabled when building from source:


```
sudo ./configure --enable-chrooted-binary --enable-winscp-compat --enable-rsync-compat --enable-scp-compat --with-sftp-server=/usr/libexec/openssh/sftp-server 


```


The following options were used:


- --enable-chrooted-binary: Installs chrooted binary scponlyc
- --enable-winscp-compat: Enables compatibility with WinSCP, a Windows scp/sftp client
- --enable-rsync-compat: Enable compatibility with rsync, a very versatile file copying utility
- --enable-scp-compat: Enables compatibility with the UNIX style scp commands

Next we will build scponly with the make command.  The make command take all your options that you passed using the configure command and builds it into the binaries that will be installed and run on the OS.


```
sudo make


```


Next we will install the binaries with make install:


```
sudo make install


```


Finally add the scponly shells to the /etc/shells file:


```
sudo /bin/su -c "echo "/usr/local/bin/scponly" >> /etc/shells"


```


The /etc/shells file tells the operating system which shells are available to the users.  So we are telling the operating system that we added a new shell to the system called scponly and that the binary is located at /usr/local/bin/scponly.


# Step 4 — Create scponly Group


Now we will create a group called scponly so we can easily manage all the users who will be accessing the server with scponly.


```
sudo groupadd scponly


```


# Step 5 — Create an Upload Directory and Set Proper Permissions


In this section we will create a centralized upload directory for the scponly group.  This allows you control over where and how much data can be uploaded to the server.


Create a directory named /pub/upload this will be a directory dedicated to uploads:


```
sudo mkdir -p /pub/upload


```


Change the group ownership of the /pub/upload directory to scponly:


```
sudo chown root:scponly /pub/upload


```


The next step is setting up permissions on the /pub/upload directory.  By setting the permissions on this directory to 770 we are giving access to only the root users and members of the scponly group.


Change permissions on the /pub/upload directory to read, write, and execute for the owner and group and remove all permissions for others:


```
sudo chmod 770 /pub/upload


```


# Step 6 — Create a User Account with scponly Shell


Now we are going to setup a test user account to verify our scponly configuration.


Create a user named testuser1 and specify scponly as an alternative group and /usr/local/bin/scponly as the shell:


```
sudo useradd -m -d /home/testuser1 -s "/usr/local/bin/scponly" -c "testuser1" -G scponly testuser1

```



Note:  Next is a very important step.  The user’s home directory should not be writable because they could modify certain SSH parameters  and possibly subvert the scponly shell.

Change permissions on the testuser1 home directory to read and execute only for the owner:


```
sudo chmod 500 /home/testuser1


```


Finally, set a password for the testuser1 user:


```
sudo passwd testuser1


```


# Step 7 — Verify User Does Not Have Access to Interactive Shell


Now we will test the scponly shell access and verify that it works as expected.


Let’s verify that the testuser1 account does not have access to a terminal.


Try to log into the server as testuser1:


```
su - testuser1


```


Your terminal will hang since you do not have access to an interactive shell.  Press CTRL+C to exit the scponly shell.


You can also test access from your local machine:


```
ssh testuser1@your_server_ip


```


Again, your terminal will hang because testuser1 is not allowed shell access. Press CTRL+C to exit the scponly shell.


# Step 8 — Test Users Ability to Download Files


In this section we will be connecting via sftp from your local machine to your DigitalOcean Droplet to verify that the testuser1 account can download files.


First create a 100 Megabyte file using fallocate:


```
sudo fallocate -l 100m /home/testuser1/testfile.img


```


Change ownership of the testfile.img file to testuser1:


```
sudo chown testuser1:testuser1 /home/testuser1/testfile.img


```


On your local system change directory to /tmp:


```
cd /tmp


```


Next sftp to your DigitalOcean server:


```
sftp testuser1@your_server_ip


```


You may be prompted to save the ssh key as you enter the password.


Once logged in issue ls -l at the sftp> prompt:


```
ls -l


```


Download the file using the get command:


```
get testfile.img


```


Once the file is finished downloading type quit to exit:


```
quit


```


Back on your local machine, verify that the file was downloaded successfully:


```
ls -l testfile.img


```


# Step 9 — Test Users Ability to Upload Files


In this section we will be testing the ability of the testuser1 account to upload files to the server using sftp.



Note:  In this section we will be restricting access to the /pub/upload directory.  This is not required but is an added security benefit for multiple reasons such as managing quotas or disk usage and easily monitoring all uploads in a central location.

On your local system create an 100 megabyte file called uploadfile.img using fallocate:


```
fallocate -l 100m /home/testuser1/uploadfile.img


```


From your local system connect to your DigitalOcean Droplet.


```
sftp testuser1@your_server_ip


```


Next upload the uploadfile.img to /pub/upload from the sftp prompt:


```
put uploadfile.img /pub/upload/


```


Verify the file was successfully uploaded by issuing the following command at the sftp prompt:


```
ls -ltr /pub/upload


```


The results should similar to:


```
-rw-r--r--    1 testuser1 testuser1 104857600 Jun  5 07:46 uploadfile.img

```


Finally type quit at the sftp prompt:


```
quit


```


# Conclusion


scponly should be in every admin’s toolbox.  It can be used as a secure alternative to anonymous FTP or as a way of giving authenticated users the ability to download and upload files without having an interactive shell. The logging of scponly occurs in the standard ssh log file /var/log/secure. As always read the man pages and keep your system updated.


For more information about scponly, go to the scponly GitHub page.


