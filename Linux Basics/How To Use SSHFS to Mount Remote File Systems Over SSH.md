# How To Use SSHFS to Mount Remote File Systems Over SSH

```Linux Basics```

## Introduction


Transferring files over an SSH connection, by using either SFTP or SCP, is a popular method of moving small amounts of data between servers. In some cases, however, it may be necessary to share entire directories, or entire filesystems, between two remote environments. While this can be accomplished by configuring an SMB or NFS mount, both of these require additional dependencies and can introduce security concerns or other overhead.


As an alternative, you can install SSHFS to mount a remote directory by using SSH alone. This has the significant advantage of requiring no additional configuration, and inheriting permissions from the SSH user on the remote system. SSHFS is particularly useful when you need to read from a large set of files interactively on an individual basis.


# Prerequisites


- Two Linux servers which are configured to allow SSH access between them. One of these can be a local machine rather than a cloud server. You can accomplish this by following our Initial Server Setup Guide, by connecting directly from one machine to the other.

# Step 1 — Installing SSHFS


SSHFS is available for most Linux distributions. On Ubuntu, you can install it using apt.


First, use apt update to refresh your package sources:


```
sudo apt update


```


Then, use apt install to install the sshfs package.


```
sudo apt install sshfs


```



Note: SSHFS can be installed on Mac or Windows through the use of filesystem libraries called FUSE, which provide interoperability with Linux environments. They will use identical concepts and connection details to this tutorial, but may require you to use different configuration interfaces or install third-party libraries. This tutorial will cover SSHFS on Linux only, but you should be able to adapt these steps to Mac or Windows FUSE implementations.
You can install SSHFS for Windows from the project’s GitHub Repository.
You can install SSHFS for Mac from the macFUSE Project.

# Step 2 — Mounting the Remote Filesystem


Whenever you are mounting a remote filesystem in a Linux environment, you first need an empty directory to mount it in. Most Linux environments include a directory called /mnt that you can create subdirectories within for this purpose.



Note: On Windows, remote filesystems are sometimes mounted with their own drive letter like G:, and on Mac, they are usually mounted in the /Volumes directory.

Create a subdirectory within /mnt called droplet using the mkdir command:


```
sudo mkdir /mnt/droplet


```


You can now mount a remote directory using sshfs.


```
sudo sshfs -o allow_other,default_permissions sammy@your_other_server:~/ /mnt/droplet


```


The options to this command behave as follows:


- -o precedes miscellaneous mount options (this is the same as when running the mount command normally for non-SSH disk mounts). In this case, you are using allow_other to allow other users to have access to this mount (so that it behaves like a normal disk mount, as sshfs prevents this by default), and default_permissions (so that it otherwise uses regular filesystem permissions).
- sammy@your_other_server:~/ provides the full path to the remote directory, including the remote username, sammy, the remote server, your_other_server, and the path, in this case ~/ for the remote user’s home directory. This uses the same syntax as SSH or SCP.
- /mnt/droplet is the path to the local directory being used as a mount point.

If you receive a Connection reset by peer message, make sure that you have copied your SSH key to the remote system. sshfs uses an ordinary SSH connection in the background, and if it is your first time connecting to the remote system over SSH, you may be prompted to accept the remote host’s key fingerprint.


```
OutputThe authenticity of host '164.90.133.64 (164.90.133.64)' can't be established.
ED25519 key fingerprint is SHA256:05SYulMxeTDWFZtf3/ruDDm/3mmHkiTfAr+67FBC0+Q.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes

```



Note: If you need to mount a remote directory using SSHFS without requiring sudo permissions, you can create a user group called fuse on your local machine, by using sudo groupadd fuse, and then adding your local user to that group, by using sudo usermod -a -G fuse sammy.

You can use ls to list the files in the mounted directory to see if they match the contents of the remote directory:


```
ls /mnt/droplet


```


```
Outputremote_file1 remote_file2

```


Now you can work with files on your remote server as if it were a physical device attached to your local machine. For instance, if you create a file in the /mnt/droplet directory, the file will appear on your virtual server. Likewise, you can copy files into or out of the /mnt/droplet folder and they will be uploaded to or from your remote server in the background.


It is important to note that the mount command only mounts a remote disk for your current session. If the virtual server or local machine is powered off or restarted, you will need to use the same process to mount it again.


If you no longer need this mount, you can unmount it with the umount command:


```
sudo umount /mnt/droplet


```


In the last step, you’ll walk through an example of configuring a permanent mount.


# Step 3 — Permanently Mounting the Remote Filesystem


As with other types of disk and network mounts, you can configure a permanent mount using SSHFS. To do this, you’ll need to add a configuration entry to a file named /etc/fstab, which handles Linux filesystem mounts at startup.


Using nano or your favorite text editor, open /etc/fstab:


```
sudo nano /etc/fstab


```


At the end of the file, add an entry like this:


/etc/fstab
```
…
sammy@your_other_server:~/ /mnt/droplet fuse.sshfs noauto,x-systemd.automount,_netdev,reconnect,identityfile=/home/sammy/.ssh/id_rsa,allow_other,default_permissions 0 0

```


Permanent mounts often require a number of different options like this to ensure they behave as expected. They work as follows:


- sammy@your_other_server:~/ is the remote path again, just as before.
- /mnt/droplet is the local path again.
- fuse.sshfs specifies the driver being used to mount this remote directory.
- noauto,x-systemd.automount,_netdev,reconnect are a set of options that work together to ensure that permanent mounts to network drives behave gracefully in case the network connection drops from the local machine or the remote machine.
- identityfile=/home/sammy/.ssh/id_rsa specifies a path to a local SSH key so that the remote directory can be mounted automatically. Note that this example assumes that both your local and your remote username are sammy – this refers to the local path. It is necessary to specify this because /etc/fstab effectively runs as root, and would not otherwise know which username’s SSH configurations to check for a key that is trusted by the remote server.
- allow_other,default_permissions use the same permissions from the mount command above.
- 0 0 signifies that the remote filesystem should never be dumped or validated by the local machine in case of errors. These options may be different when mounting a local disk.

Save and close the file. If you are using nano, press Ctrl+X, then when prompted, Y and then ENTER. You can then test the /etc/fstab configuration by restarting your local machine, for example by using sudo reboot now, and verifying that the mount is recreated automatically.


It should be noted that permanent SSHFS mounts are not necessarily popular. The nature of SSH connections and SSHFS means that it is usually better suited to temporary, one-off solutions, when you don’t need to commit to an SMB or NFS mount which can be configured with greater redundancy and other options. That said, SSHFS is very flexible, and more importantly, acts as a full-fledged filesystem driver, which allows you to configure it in /etc/fstab like any other disk mount and use it as much as needed. Be careful that you do not accidentally expose more of the remote filesystem over SSH than you intend.


# Conclusion


In this tutorial, you configured an SSHFS mount from one Linux environment to another. Although it is not the most scalable or performant solution for a production deployment, SSHFS can be very useful with minimal configuration.


Next, you may want to learn about working with object storage which can be mounted concurrently across multiple servers.


