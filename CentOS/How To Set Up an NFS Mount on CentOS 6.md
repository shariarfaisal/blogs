# How To Set Up an NFS Mount on CentOS 6

```CentOS```










# Status: Deprecated


This article covers a version of CentOS that is no longer supported. If you are currently operating a server running CentOS 6, we highly recommend upgrading or migrating to a supported version of CentOS.


Reason:
CentOS 6 reached end of life (EOL) on November 30th, 2020 and no longer receives security patches or updates. For this reason, this guide is no longer maintained.


See Instead:
This guide might still be useful as a reference, but may not work on other CentOS releases. If available, we strongly recommend using a guide written for the version of CentOS you are using.


## About NFS (Network File System) Mounts


NFS mounts work to share a directory between several servers. This has the advantage of saving disk space, as the home directory is only kept on one server, and others can connect to it over the network. When setting up mounts, NFS is most effective for permanent fixtures that should always be accessible.


# Setup


An NFS mount is set up between at least two servers. The machine hosting the shared network is called the server, while the ones that connect to it are called ‘clients’.


This tutorial requires 2 servers: one acting as the server and one as the client. We will set up the server machine first, followed by the client. The following IP addresses will refer to each one:


Master: 12.34.56.789


Client: 12.33.44.555


The system should be set up as root. You can access the root user by typing


```
sudo su
```


# Setting Up the NFS Server


## Step One—Download the Required Software


```
yum install nfs-utils nfs-utils-lib
```


```
chkconfig nfs on 
service rpcbind start
service nfs start
```


## Step Two—Export the Shared Directory


The next step is to decide which directory we want to share with the client server. The chosen directory should then be added to the /etc/exports file, which specifies both the directory to be shared and the details of how it is shared.


Suppose we wanted to share the directory, /home.


We need to export the directory:


```
vi /etc/exports
```


Add the following lines to the bottom of the file, sharing the directory with the client:


```
/home           12.33.44.555(rw,sync,no_root_squash,no_subtree_check)
```


These settings accomplish several tasks:


- rw: This option allows the client server to both read and write within the shared directory
- sync: Sync confirms requests to the shared directory only once the changes have been committed. 
-  no_subtree_check: This option prevents the subtree checking. When a shared directory is the subdirectory of a larger filesystem, nfs performs scans  of every directory above it, in order to verify its permissions and details. Disabling the subtree check may increase the reliability of NFS, but reduce security.
- no_root_squash: This phrase allows root to connect to the designated directory

Once you have entered in the settings for each directory, run the following command to export them:


```
exportfs -a
```


# Setting Up the NFS Client


## Step One—Download the Required Software


Start off by using apt-get to install the nfs programs.


```
yum install nfs-utils nfs-utils-lib
```


## Step Two—Mount the Directories


Once the programs have been downloaded to the the client server, create the directory that will contain the NFS shared files


```
mkdir -p /mnt/nfs/home
```


Then go ahead and mount it


```
mount 12.34.56.789:/home /mnt/nfs/home
```


You can use the df  -h command to check that the directory has been mounted. You will see it last on the list.


```
df -h
```


```
Filesystem            Size  Used Avail Use% Mounted on
/dev/sda               20G  783M   18G   5% /
12.34.56.789:/home       20G  785M   18G   5% /mnt/nfs/home
```


Additionally, use the mount command to see the entire list of mounted file systems.


```
mount
```


Your list should look something like this:


```
/dev/sda on / type ext4 (rw,errors=remount-ro)
none on /proc/sys/fs/binfmt_misc type binfmt_misc (rw)
sunrpc on /var/lib/nfs/rpc_pipefs type rpc_pipefs (rw)
nfsd on /proc/fs/nfsd type nfsd (rw)
12.34.56.789:/home on /mnt/nfs/home type nfs (rw,noatime,nolock,bg,nfsvers=2,intr,tcp,actimeo=1800,addr=12.34.56.789)
```


# Testing the NFS  Mount


Once you have successfully mounted your NFS directory, you can test that it works by creating a file on the Client and checking its availability on the Server.


Create a file in the directory to try it out:


```
touch /mnt/nfs/home/example
```


You should then be able to find the files on the Server in the /home.


```
ls /home
```


You can ensure that the mount is always active by adding the directory to the fstab file on the client. This will ensure that the mount starts up after the server reboots.


```
vi /etc/fstab
```


```
12.34.56.789:/home  /mnt/nfs/home   nfs      auto,noatime,nolock,bg,nfsvers=3,intr,tcp,actimeo=1800 0 0
```


You can learn more about the fstab options by typing in:


```
man nfs
```


After any subsequent server reboots, you can use a single command to mount directories specified in the fstab file:


```
mount -a
```


You can check the mounted directories with the two earlier commands:


```
df -h
```


```
mount
```


# Removing the NFS Mount


Should you decide to remove a directory, you can unmount it using the umount command:


```
cd
sudo umount /directory name
```


You can see that the mounts were removed by then looking at the filesystem again.


```
df -h
```


You should find your selected mounted directory gone.


By Etel Sverdlov
