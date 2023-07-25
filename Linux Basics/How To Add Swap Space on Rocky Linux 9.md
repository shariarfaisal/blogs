# How To Add Swap Space on Rocky Linux 9

```Linux Basics``` ```Rocky Linux``` ```Rocky Linux 9```

## Introduction


One way to guard against out-of-memory errors in applications is to add some swap space to your server. In this guide, we will cover how to add a swap file to a Rocky Linux 9 server.


# What is Swap?


Swap is a portion of hard drive storage that has been set aside for the operating system to temporarily store data that it can no longer hold in RAM. This lets you increase the amount of information that your server can keep in its working memory, with some caveats. The swap space on the hard drive will be used mainly when there is no longer sufficient space in RAM to hold in-use application data.


The information written to disk will be significantly slower than information kept in RAM, but the operating system will prefer to keep running application data in memory and use swap for the older data. Overall, having swap space as a fallback for when your system’s RAM is depleted can be a good safety net against out-of-memory exceptions on systems with non-SSD storage available.


# Step 1 – Checking the System for Swap Information


Before we begin, we can check if the system already has some swap space available. It is possible to have multiple swap files or swap partitions, but generally one should be enough.


We can see if the system has any configured swap by typing:


```
sudo swapon --show


```


If you don’t get back any output, this means your system does not have swap space available currently.


You can verify that there is no active swap using the free utility:


```
free -h


```


```
Output               total        used        free      shared  buff/cache   available
Mem:           1.7Gi       173Mi       1.2Gi       9.0Mi       336Mi       1.4Gi
Swap:             0B          0B          0B

```


As you can see in the Swap row of the output, no swap is active on the system.


# Step 2 – Checking Available Space on the Hard Drive Partition


Before we create our swap file, we’ll check our current disk usage to make sure we have enough space. Do this by entering:


```
df -h


```


```
OutputFilesystem      Size  Used Avail Use% Mounted on
devtmpfs        855M     0  855M   0% /dev
tmpfs           888M     0  888M   0% /dev/shm
tmpfs           355M  9.4M  346M   3% /run
/dev/vda1        59G  1.4G   58G   3% /
/dev/vda2       994M  155M  840M  16% /boot
/dev/vda15      100M  7.0M   93M   7% /boot/efi
tmpfs           178M     0  178M   0% /run/user/0

```


The device with / in the Mounted on column is our disk in this case. We have plenty of space available in this example (only 1.4G used). Your usage will probably be different.


Although there are many opinions about the appropriate size of a swap space, it really depends on your personal preferences and your application requirements. Generally, an amount equal to or double the amount of RAM on your system is a good starting point. Another good rule of thumb is that anything over 4G of swap is probably unnecessary if you are just using it as a RAM fallback.


# Step 3 – Creating a Swap File


Now that we know our available hard drive space, we can create a swap file on our filesystem. We will allocate a file of the size that we want called swapfile in our root (/) directory.


The best way of creating a swap file is with the fallocate program. This command instantly creates a file of the specified size.


Since the server in our example has 2G of RAM, we will create a 2G file in this guide. Adjust this to meet the needs of your own server:


```
sudo fallocate -l 1G /swapfile


```


We can verify that the correct amount of space was reserved by typing:


```
ls -lh /swapfile


```


```
-rw-r--r--. 1 root root 2.0G Sep 13 17:52 /swapfile


```


Our file has been created with the correct amount of space set aside.


# Step 4 – Enabling the Swap File


Now that we have a file of the correct size available, we need to actually turn this into swap space.


First, we need to lock down the permissions of the file so that only users with root privileges can read the contents. This prevents normal users from being able to access the file, which would have significant security implications.


Make the file only accessible to root by typing:


```
sudo chmod 600 /swapfile


```


Verify the permissions change by typing:


```
ls -lh /swapfile


```


```
Output-rw------- 1 root root 2.0G Sep 13 17:52 /swapfile

```


As you can see, only the root user has the read and write flags enabled.


We can now mark the file as swap space by typing:


```
sudo mkswap /swapfile


```


```
OutputSetting up swapspace version 1, size = 2 GiB (2147479552 bytes)
no label, UUID=585e8b33-30fa-481f-af61-37b13326545b

```


After marking the file, we can enable the swap file, allowing our system to start using it:


```
sudo swapon /swapfile


```


Verify that the swap is available by typing:


```
sudo swapon --show


```


```
OutputNAME      TYPE  SIZE USED PRIO
/swapfile file 2G   0B   -2

```


We can check the output of the free utility again to corroborate our findings:


```
free -h


```


```
Output               total        used        free      shared  buff/cache   available
Mem:           1.7Gi       172Mi       1.2Gi       9.0Mi       338Mi       1.4Gi
Swap:          2.0Gi          0B       2.0Gi

```


Our swap has been set up successfully and our operating system will begin to use it as necessary.


# Step 5 – Making the Swap File Permanent


Our recent changes have enabled the swap file for the current session. However, if we reboot, the server will not retain the swap settings automatically. We can change this by adding the swap file to our /etc/fstab file.


Back up the /etc/fstab file in case anything goes wrong:


```
sudo cp /etc/fstab /etc/fstab.bak


```


Add the swap file information to the end of your /etc/fstab file by typing:


```
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab


```


Next we’ll review some settings we can update to tune our swap space.


# Step 6 – Tuning your Swap Settings


There are a few options that you can configure that will have an impact on your system’s performance when dealing with swap.


## Adjusting the Swappiness Property


The swappiness parameter configures how often your system swaps data out of RAM to the swap space. This is a value between 0 and 100 that represents a percentage.


With values close to zero, the kernel will not swap data to the disk unless absolutely necessary. Remember, interactions with the swap file are “expensive” in that they take a lot longer than interactions with RAM and they can cause a significant reduction in performance. Telling the system not to rely on the swap much will generally make your system faster.


Values that are closer to 100 will try to put more data into swap in an effort to keep more RAM space free. Depending on your applications’ memory profile or what you are using your server for, this might be better in some cases.


We can see the current swappiness value by typing:


```
cat /proc/sys/vm/swappiness


```


```
Output60

```


For a Desktop, a swappiness setting of 60 is not a bad value. For a server, you might want to move it closer to 0.


We can set the swappiness to a different value by using the sysctl command.


For instance, to set the swappiness to 10, we could type:


```
sudo sysctl vm.swappiness=10


```


```
Outputvm.swappiness = 10

```


This setting will persist until the next reboot. We can set this value automatically at restart by adding the line to our /etc/sysctl.conf file.


The default text editor that comes with Rocky Linux 9 is vi. vi is an extremely powerful text editor, but it can be somewhat obtuse for users who lack experience with it. You might want to install a more user-friendly editor such as nano to facilitate editing configuration files on your Rocky Linux 9 server:


```
sudo dnf install nano


```


Now you can use nano to edit the sysctl.conf file:


```
sudo nano /etc/sysctl.conf


```


At the bottom, you can add:


/etc/sysctl.conf
```
vm.swappiness=10

```


Save and close the file when you are finished. If you are using nano, you can save and quit by pressing CTRL + X, then when prompted, Y and then Enter.


## Adjusting the Cache Pressure Setting


Another related value that you might want to modify is the vfs_cache_pressure. This setting configures how much the system will choose to cache inode and dentry information over other data.


This is access data about the filesystem. This is generally very costly to look up and very frequently requested, so it’s an excellent thing for your system to cache. You can see the current value by querying the proc filesystem again:


```
cat /proc/sys/vm/vfs_cache_pressure


```


```
Output100

```


As it is currently configured, our system removes inode information from the cache too quickly. We can set this to a more conservative setting like 50 by typing:


```
sudo sysctl vm.vfs_cache_pressure=50


```


```
Outputvm.vfs_cache_pressure = 50

```


Again, this is only valid for our current session. We can change that by adding it to our configuration file like we did with our swappiness setting:


```
sudo nano /etc/sysctl.conf


```


At the bottom, add the line that specifies your new value:


/etc/sysctl.conf
```
vm.vfs_cache_pressure=50

```


Save and close the file when you are finished.


# Conclusion


Following the steps in this guide will give you some breathing room in cases that would otherwise lead to out-of-memory exceptions. Swap space can be incredibly useful in avoiding some of these common problems.


If you are running into out of memory errors, or if you find that your system is unable to use the applications you need, the best solution is to optimize your application configurations or upgrade your server.


