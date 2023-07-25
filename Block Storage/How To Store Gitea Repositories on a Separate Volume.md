# How To Store Gitea Repositories on a Separate Volume

```Block Storage``` ```Docker``` ```Git``` ```Ubuntu```

## Introduction


Gitea is a source code repository based on the version control system, Git. While there are several self-hosted solutions available such as GitLab and Gogs, Gitea has the benefit of being lightweight, meaning it can run on a relatively small server.


However, having a small server, especially in the realm of VPSes, often means being limited on space. Thankfully, many hosting providers also offer additional storage in the form of external volumes, block storage, or networked file storage (NFS). This gives users the option to save money on smaller VPS hosts for their applications without sacrificing storage.


With Gitea and the ability to decide where your source code is stored, you can ensure that your projects and files have room to expand. In this tutorial, you will mount an external storage volume to a mount point and ensure that Gitea is reading the appropriate information from that volume. By the end, you will have a Gitea installation that stores repositories and other important information on a separate storage volume.


# Prerequisites


Before you get started, you will need the following:


- An Ubuntu 20.04 server set up according to our initial server setup guide for Ubuntu 20.04, with a non-root user with sudo privileges and a firewall enabled.
- An external volume such as an NFS or block storage volume. If you’d like to set up a Block Storage Volume from DigitalOcean, follow our product documentation.
- An installation of Gitea on that server accessible via a domain name. This guide assumes you’ve installed Gitea on Docker, as outlined in our tutorial on How To Install Gitea on Ubuntu 20.04. Note that this will require you to have Docker and Docker Compose installed on your server. You can install these tools by following the steps in these tutorials:

Steps 1 and 2 of How to Install and Use Docker on Ubuntu 20.04
Step 1 of How to Install and Use Docker Compose on Ubuntu 20.04


- Steps 1 and 2 of How to Install and Use Docker on Ubuntu 20.04
- Step 1 of How to Install and Use Docker Compose on Ubuntu 20.04

Note that if you installed Gitea using a different method than outlined in these prerequisites, the names and locations for certain files and directories on your system may differ from what this guide mentions in examples. However, the concepts outlined in this tutorial should be applicable to any Gitea installation.


# Step 1 — Mounting a Block Storage Volume


A volume can take many different forms. It could be an NFS volume, which is storage available on the network provided via a file share. Another possibility is that it takes the form of block storage via a service such as DigitalOcean’s Volumes. In both cases, storage is mounted on a system using the mount command.


Volumes such as these will be visible as device files stored within /dev. These files are how the kernel communicates with the storage devices themselves; the files aren’t actually used for storage. In order to be able to store files on the storage device, you will need to mount them using the mount command.


First, you will need to create a mount point — that is, a folder which will be associated with the device, such that data stored within it winds up stored on that device. Mount points for storage devices such as this typically live in the /mnt directory.


Create a mount point named gitea as you would create a normal directory using the mkdir command:


```
sudo mkdir /mnt/gitea


```


From here, you can mount the device to that directory in order to use it to access that storage space. Use the mount command to mount the device:


```
sudo mount -t ext4 -o defaults,noatime /dev/disk/by-id/your_disk_id /mnt/gitea


```


The string ext4 option specifies the file system type, ext4 in this case, though depending on your volume’s file system type, it may be something like xfs or nfs; to check which type your volume uses, run the mount command with no options:


```
mount


```


This will provide you with a line of output for every mounted file system. Since you just mounted yours, it will likely be the last on the list:


```
Output. . .
/dev/sda on /mnt/gitea type ext4 (rw,noatime,discard)

```


This shows that the file system type is ext4.


This command mounts the device specified by its ID to /mnt/gitea. The -o option specifies the options used when mounting. In this case, you are using the default options which allow for mounting a read/write file system, and the noatime option specifies that the kernel shouldn’t update the last access time for files and directories on the device in order to be more efficient.


Now that you’ve mounted your device, it will stay mounted as long as the system is up and running. However, as soon as the system restarts, it will no longer be mounted (though the data will remain on the volume), so you will need to tell the system to mount the volume as soon as it starts using the /etc/fstab (‘file systems table’) file. This file lists the available file systems and their mount points in a tab-delimited format.


Using echo and tee, add a new line to the end of /etc/fstab:


```
echo '/dev/disk/by-id/your_disk_id /mnt/gitea ext4 defaults,nofail,noatime 0 0' | sudo tee /etc/fstab


```


This command appends the string /dev/disk/by-uid/your_disk_id to the fstab file and prints it to your screen. As with the previous mount command, it mounts the device onto the mount point using the defaults, nofail, and noatime options.


Once your changes have been made to /etc/fstab, the kernel will mount your volume on boot.



Note: Storage devices on Linux are very flexible and come in all different types, from a networked file system (NFS) to a plain old hard drive. To learn more about block storage and devices in Linux, you can read up more about storage concepts in our Introduction to Storage Terminology and Concepts in Linux.

# Step 2 — Configuring Gitea to Store Data on a Block Storage Volume


Gitea maintains all of its repositories in a central location. This includes repositories from all users and organizations. Unless configured otherwise, all information is kept in a single directory. This directory is named data in default installations. For the purposes of this tutorial, we will be using a version of Gitea running on Docker as in the tutorial linked above.


First, let’s get a sense of what this data directory contains. You can do this by moving to the data directory and running the ls command. Using the -l format will tell us more information about the files:


```
cd gitea
ls -l


```


This will provide a listing like the following:


```
Outputtotal 20
drwxr-xr-x  5 root  root  4096 Jun 23 22:34 ./
drwxrwxr-x  3 sammy sammy 4096 Jun 26 22:35 ../
drwxr-xr-x  5 git   git   4096 Jun 23 22:42 git/
drwxr-xr-x 12 git   git   4096 Jun 26 22:35 gitea/
drwx------  2 root  root  4096 Jun 23 22:34 ssh/

```


Let’s break down the output of this command. It lists one file or directory per line. In this case, it lists five directories. The entry for . is a special entry that just means the current directory, and .. stands for the directory one level up. This output shows that the current directory is owned by the root user, which is the case in this instance because Docker runs as a privileged user, and the directory one level up is owned by sammy.


The git directory is important to us because it contains all of the repositories that we might want to store on a separate volume. List the contents of the directory:


```
ls -l git


```


This will provide the long listing of the directory:


```
Outputtotal 24
drwxr-xr-x 5 git  git  4096 Jun 23 22:42 ./
drwxr-xr-x 6 root root 4096 Jun 27 14:21 ../
-rw-r--r-- 1 git  git   190 Jun 23 22:42 .gitconfig
drwxr-xr-x 2 root root 4096 Jun 23 22:34 .ssh/
drwxr-xr-x 2 git  git  4096 Jun 23 22:42 lfs/
drwxr-xr-x 5 git  git  4096 Jun 30 20:03 repositories/

```


Within it are two directories of note: the repositories directory that contains the git repositories managed by Gitea sorted by user/organization, and the lfs directory containing data for Git’s Large File Storage functionality. The gitea directory contains information that Gitea uses in the background, including archives of old repositories, as well the database that contains information such as users and repository information used by the web service. The ssh directory contains various SSH keypairs that Gitea uses.


Given that all of the information stored in this directory is important, you will want to include the entire directory’s contents on our attached volume.


There are two paths to follow from this point, depending on whether or not you are working with a new installation of Gitea and completing this tutorial during the installation process. In the first path, you will be able to specify locations before finishing the installation, and in the second, you will learn how to move an existing installation.


## Setting Up a New Installation of Gitea


If you are starting with a brand new installation of Gitea, you can specify where all of your information is stored during the configuration process. For example, if you are setting Gitea up using Docker Compose, you can map the volumes to your attached volume.


Open up the docker-compose.yml file with your preferred text editor. The following example uses nano:


```
nano docker-compose.yml


```


Once you have the file open, search for the volumes entry in the compose file and modify the mapping on the left side of the : to point to appropriate locations on your block storage volume for the Gitea data directory


docker-compose.yml
```
...

    volumes:
      - /mnt/gitea:/data
      - /home/git/.ssh/:/data/git/.ssh
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro

...

```


When you are finished setting the information, save and close the file. If you are using nano, you can do this by pressing CTRL + X, Y, and then ENTER.



Warning: SSH servers look for the .ssh directory in the home directory of the Git user (git, in this case). This directory contains all of the SSH keys that Gitea will use, so it is not advised to move the mount for this Docker volume. In order to have this location backed up on your volume, it would be best to use another solution such as a cron job to back up the directory. To learn more, check out this tutorial on using cron to manage scheduled tasks.

When you run docker-compose and Gitea installs, it will use your block storage volume to store its data.


If you are not using Docker volumes to manage the locations of your data — for example, if you are installing Gitea on your server via the binary releases per these instructions from Gitea — then you will need to modify the locations within the configuration file (usually /etc/gitea/app.ini) when you are setting all of the values and before you perform the final installation steps in the browser. For instance, you might set them as follows:


app.ini
```
...

# If you are using SQLite for your database, you will need to change the PATH
# variable in this section
[database]
...
PATH = /mnt/gitea/gitea.db

[server]
...
LFS_CONTENT_PATH = /mnt/gitea/lfs

[repository]
ROOT = /mnt/gitea/gitea-repositories

...

```



Note: Ensure that your git user has write access to this location. You can read up on Linux permissions here.

## Moving an Existing Installation of Gitea


If you already have an instance of Gitea installed and running, you will still be able to store your data on a separate volume, but it will require some care in ensuring that all of your data remains both safe and accessible to Gitea.



Note: As with any operation involving your data, it is important to ensure that you have an up-to-date backup of everything. In this case, this means a backup of all of the files in your data directory (the SSH files, the gitea-repositories and lfs directories, the database, and so on) to a safe location.

There are two options for moving your data to a new volume. The first way is to copy your Gitea data to a secondary location, then turn the original location into a mount point for your volume. When you copy your data back into that location, you will be copying it onto that volume, and no changes will be required within Gitea itself; it will simply continue as it did before. If, however, you do not want to mount the entire device to that destination (for example, if your Gitea data will not be the only thing on that volume), then a second option is to move all of your Gitea data to a new location on that volume and instruct Gitea itself to use that location.


No matter which option you choose, first, stop the Gitea web service.


If you installed Gitea via Docker Compose, use docker-compose to stop the service. While inside the same directory containing the docker-compose.yml file, run:


```
docker-compose down


```


If you have installed it as a systemd service, use the systemctl command:


```
sudo systemctl stop gitea


```


Once Gitea has been stopped, move the entire contents of the data directory to the mount point made in Step 1:


```
sudo mv * /mnt/gitea


```


Ensure that all files have been moved by listing the current directory’s contents:


```
ls -la


```


This will only return the current and parent directory entries:


```
total 8
drwxrwxr-x 2 sammy sammy 4096 Jun 27 13:56 ./
drwxr-xr-x 7 sammy sammy 4096 Jun 27 13:56 ../

```


Once all of the data has been moved, change to the new data directory:


```
cd /mnt/gitea


```


Using ls, ensure that everything looks correct:


```
ls -l


```


This will show the contents of the directory:


```
Outputtotal 36
drwxr-xr-x  6 root root  4096 Jun 27 14:21 ./
drwxr-xr-x  3 root root  4096 Jun 27 14:21 ../
drwxr-xr-x  5 git  git   4096 Jun 23 22:42 git/
drwxr-xr-x 13 git  git   4096 Jul 11 08:25 gitea/
drwx------  2 root root 16384 Jun 27 03:46 lost+found/
drwx------  2 root root  4096 Jun 23 22:34 ssh/

```


As before, it should contain the ssh, git, and gitea directories. If you are using SQLite as a database to manage Gitea, it will also contain a file named gitea.db in the gitea directory.


When you’re sure that all data has been moved, it’s time to mount the volume to the data directory.


First, move to the parent directory of the data directory you were in previously. In this example using an installation of Gitea using Docker Compose as described in the tutorial linked in the prerequisites, this is the directory which contains your docker-compose.yml file.


```
cd ~/gitea/


```


As before, use the mount command, but this time, use the directory you just emptied as the destination:


```
sudo mount -o defaults,noatime /dev/disk/by-id/your_disk_id gitea


```


Now, when you list the contents of that directory, all of your files should be in place:


```
ls -la gitea


```


This command will output the expected information. Note that, depending on your volume’s file system type, you may find an additional directory named lost+found; this is normal and part of everyday file system use:


```
total 36
drwxr-xr-x  6 root  root   4096 Jun 27 13:58 ./
drwxrwxr-x  3 sammy sammy  4096 Jun 27 02:23 ../
drwxr-xr-x  5 git   git    4096 Jun 23 22:42 git/
drwxr-xr-x 12 git   git    4096 Jun 27 00:00 gitea/
drwx------  2 root  root   16384 Jun 27 03:46 lost+found/
drwx------  2 root  root   4096 Jun 23 22:34 ssh/

```


As mentioned, if you would like Gitea to use a directory within the block storage volume, there is an additional step you need to complete before bringing Gitea back up. For example, say that you want to use a folder named scm on your volume mounted on /mnt/gitea. After moving all of your Gitea data to /mnt/gitea/scm, you will need to create a symbolic link from your old data directory to the new one. For this you will use the ln command:


```
sudo ln -s /mnt/gitea/scm gitea


```


At this point, you can restart Gitea. If you are using Gitea as a systemd service, run:


```
sudo systemctl restart gitea


```


If you are running Gitea as a Docker container using Docker Compose, run:


```
docker-compose up -d


```


Now that everything is up and running, visit your Gitea instance in the browser and ensure that everything works as expected. You should be able to create new objects in Gitea such as repositories, issues, and so on. If you set Gitea up with an SSH shim, you should also be able to check out and push to repositories using git clone and git push.


# Conclusion


In this tutorial, you moved all of your Gitea data to a block storage volume. Volumes such as these are very flexible and provide many benefits.such as allowing you to store all of your data on larger disks, RAID volumes, networked file systems, or using block storage such as DigitalOcean Volumes to reduce storage expenses. It also allows you to snapshot entire disks for backup so that you can restore their contents in event of a catastrophic failure.


