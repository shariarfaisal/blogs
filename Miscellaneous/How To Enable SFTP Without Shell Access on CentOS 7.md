# How To Enable SFTP Without Shell Access on CentOS 7

```Miscellaneous``` ```CentOS```

## Introduction


SFTP stands for SSH File Transfer Protocol. As its name suggests, it’s a secure way of transferring files to a server using an encrypted SSH connection. Despite the name, it’s a completely different protocol than FTP (File Transfer Protocol), though it’s widely supported by modern FTP clients.


SFTP is available by default with no additional configuration on all servers that have SSH access enabled. It’s secure and easy to use, but comes with a disadvantage: in a standard configuration, the SSH server grants file transfer access and terminal shell access to all users with an account on the system.


In some cases, you might want only certain users to be allowed file transfers and no SSH access. In this tutorial, we’ll set up the SSH daemon to limit SFTP access to one directory with no SSH access allowed on per user basis.


# Prerequisites


To follow this tutorial, you will need:


- One CentOS 7 server set up with this initial server setup tutorial, including a sudo non-root user.
- Optionally, the nano text editor installed with yum install nano. CentOS comes with the vi text editor by default, but nano can be more user friendly.

# Step 1 — Creating a New User


First, create a new user who will be granted only file transfer access to the server. Here, we’re using the username sammyfiles, but you can use any username you like.


```
sudo adduser sammyfiles


```


Next, assign a password to the new user:


```
sudo passwd sammyfiles


```


Enter a strong password, and repeat it again to verify it.


You have now created a new user that will be granted access to the restricted directory. In the next step we will create the directory for file transfers and set up the necessary permissions.


# Step 2 — Creating a Directory for File Transfers


In order to restrict SFTP access to one directory, we first have to make sure the directory complies with the SSH server’s permissions requirements, which are very particular.


Specifically, the directory itself and all directories above it in the filesystem tree must be owned by root and not writable by anyone else. Consequently, it’s not possible to simply give restricted access to a user’s home directory because home directories are owned by the user, not root.



Note: Some versions of OpenSSH do not have such strict requirements for the directory structure and ownership, but most modern Linux distributions (including CentOS 7) do.

There are a number of ways to work around this ownership issue. In this tutorial, we’ll create and use /var/sftp/uploads as the target upload directory. /var/sftp will be owned by root and will be unwritable by other users; the subdirectory /var/sftp/uploads will be owned by sammyfiles, so that user will be able to upload files to it.


First, create the directories.


```
sudo mkdir -p /var/sftp/uploads


```


Set the owner of /var/sftp to root.


```
sudo chown root:root /var/sftp


```


Give root write permissions to the same directory, and give other users only read and execute rights.


```
sudo chmod 755 /var/sftp


```


Change the ownership on the uploads directory to sammyfiles.


```
sudo chown sammyfiles:sammyfiles /var/sftp/uploads


```


Now that the directory structure is in place, we can configure the SSH server itself.


# Step 3 — Restricting Access to One Directory


In this step, we’ll modify the SSH server configuration to disallow terminal access for sammyfiles but allow file transfer access.


Let’s open the SSH server configuration file with vi or your favourite text editor (here’s a short introduction to vi)


```
sudo vi /etc/ssh/sshd_config


```


Scroll to the very bottom of the file and append the following configuration snippet:


/etc/ssh/sshd_config
```
. . .

Match User sammyfiles
ForceCommand internal-sftp
PasswordAuthentication yes
ChrootDirectory /var/sftp
PermitTunnel no
AllowAgentForwarding no
AllowTcpForwarding no
X11Forwarding no

```


Then save and close the file.


Here’s what each of those directives do:


- Match User tells the SSH server to apply the following commands only to the user specified. Here, we specify sammyfiles.
- ForceCommand internal-sftp forces the SSH server to run the SFTP server upon login, disallowing shell access.
- PasswordAuthentication yes allows password authentication for this user.
- ChrootDirectory /var/sftp/ ensures that the user will not be allowed access to anything beyond the /var/sftp directory. You can learn more about chroot in this Ubuntu-based chroot tutorial.
- AllowAgentForwarding no, AllowTcpForwarding no. and X11Forwarding no disables port forwarding, tunneling and X11 forwarding for this user.

This set of commands, starting with Match User, can be copied and repeated for different users too. Make sure to modify the username in the Match User line accordingly.



Note: You can omit the PasswordAuthentication yes line and instead set up SSH key access for increased security. Follow the Copying your Public SSH Key section of the SSH Essentials: Working with SSH Servers, Clients, and Keys tutorial to do so. Make sure to do this before you disable shell access for the user.
In the next step, we’ll test the configuration by SSHing locally with password access, but if you set up SSH keys, you’ll instead need access to a computer with the user’s keypair.

To apply the configuration changes, restart the service.


```
sudo systemctl restart sshd


```


You have now configured the SSH server to restrict access to file transfer only for sammyfiles. The last step is testing the configuration to make sure it works as intended.


# Step 4 — Verifying the Configuration


Let’s ensure that our new sammyfiles user can only transfer files.


Logging in to the server as sammyfiles using normal shell access should no longer be possible. Let’s try it:


```
ssh sammyfiles@localhost


```


You’ll see the following message before being returned to your original prompt:


```
Error messageThis service allows sftp connections only.
Connection to localhost closed.

```


This means that sammyfiles can no longer can access the server shell using SSH.


Next, let’s verify if the user can successfully access SFTP for file transfer.


```
sftp sammyfiles@localhost


```


Instead of an error message, this command will show a successful login message with an interactive prompt.


```
SFTP promptConnected to localhost.
sftp>

```


You can list the directory contents using ls in the prompt:


```
ls


```


This will show the uploads directory that was created in the previous step and return you to the sftp> prompt.


```
SFTP file list outputuploads

```


To verify that the user is indeed restricted to this directory and cannot access any directory above it, you can try changing the directory to the one above it.


```
cd ..


```


This command will not give an error, but listing the directory contents as before will show no change, proving that the user was not able to switch to the parent directory.


You have now verified that the restricted configuration works as intended. The newly created sammyfiles user can access the server only using he SFTP protocol for file transfer and has no ability to access the full shell.


# Conclusion


You’ve restricted a user to SFTP-only access to a single directory on a server without full shell access. While this tutorial uses only one directory and one user for brevity, you can extend this example to multiple users and multiple directories.


The SSH server allows more complex configuration schemes, including limiting access to groups or multiple users at once or limited access to certain IP addresses. You can find examples of additional configuration options and explanation of possible directives in the OpenSSH Cookbook. If you run into any issues with SSH, you can debug and fix them with this troubleshooting SSH series.


