# How To Add Swap on Ubuntu 12 04

```Linux Basics``` ```Ubuntu```











# Status: Deprecated


This article covers a version of Ubuntu that is no longer supported.  If you are currently operate a server running Ubuntu 12.04, we highly recommend upgrading or migrating to a supported version of Ubuntu:


- Upgrade to Ubuntu 14.04.
- Upgrade from Ubuntu 14.04 to Ubuntu 16.04
- Migrate the server data to a supported version

Reason:
Ubuntu 12.04 reached end of life (EOL) on April 28, 2017 and no longer receives security patches or updates.  This guide is no longer maintained.


See Instead:
This guide might still be useful as a reference, but may not work on other Ubuntu releases.  If available, we strongly recommend using a guide written for the version of Ubuntu you are using.   You can use the search functionality at the top of the page to find a more recent version.


## About Linux Swapping


Linux RAM is composed of chunks of memory called pages. To free up pages of RAM, a “linux swap” can occur and a page of memory is copied from the RAM to preconfigured space on the hard disk. Linux swaps allow a system to harness more memory than was originally physically available.


However, swapping does have disadvantages. Because hard disks have a much slower memory than RAM, virtual private server performance may slow down considerably. Additionally, swap thrashing can begin to take place if the system gets swamped from too many  files being  swapped in and out.


Note
Although swap is generally recommended for systems utilizing traditional spinning hard drives, using swap with SSDs can cause issues with hardware degradation over time.  Due to this consideration, we do not recommend enabling swap on DigitalOcean or any other provider that utilizes SSD storage.  Doing so can impact the reliability of the underlying hardware for you and your neighbors.
If you need to improve the performance of your server, we recommend upgrading your Droplet.  This will lead to better results in general and will decrease the likelihood of contributing to hardware issues that can affect your service.
# Check for Swap Space


Before we proceed to set up a swap file, we need to check if any swap files have been enabled on the VPS by looking at the summary of swap usage.


```
sudo swapon -s
```


An empty list will confirm that you have no swap files enabled:


```
Filename				Type		Size	Used	Priority
```


# Check the File System


After we know that we do not have a swap file enabled on the virtual server, we can check how much space we have on the server with the df command. The swap file will take 256MB— since we are only using up about 8% of the /dev/sda, we can proceed.


```
df
Filesystem     1K-blocks    Used Available Use% Mounted on
/dev/sda        20907056 1437188  18421292   8% /
udev              121588       4    121584   1% /dev
tmpfs              49752     208     49544   1% /run
none                5120       0      5120   0% /run/lock
none              124372       0    124372   0% /run/shm
```


# Create and Enable the  Swap File


Now it’s time to create the swap file itself using the dd command :


```
sudo dd if=/dev/zero of=/swapfile bs=1024 count=256k
```


“of=/swapfile” designates the file’s name. In this case the name is swapfile.


Subsequently we are going to prepare the swap file by creating a linux swap area:


```
sudo mkswap /swapfile
```


The results display:


```
Setting up swapspace version 1, size = 262140 KiB
no label, UUID=103c4545-5fc5-47f3-a8b3-dfbdb64fd7eb
```


Finish up by activating the swap file:


```
sudo swapon /swapfile
```


You will then be able to see the new swap file when you view the swap summary.


```
swapon -s
Filename				Type		Size	Used	Priority
/swapfile                               file		262140	0	-1
```


This file will last on the virtual private server until the machine reboots.  You can ensure that the swap is permanent by adding  it to the fstab file.


Open up the file:


```
sudo nano /etc/fstab
```


Paste in the following line:


```
 /swapfile       none    swap    sw      0       0 
```


Swappiness in the file should be set to 10. Skipping this step may cause both poor performance, whereas setting it to 10 will cause swap to act as an emergency buffer, preventing out-of-memory crashes.


You can do this with the following commands:


```
echo 10 | sudo tee /proc/sys/vm/swappiness
echo vm.swappiness = 10 | sudo tee -a /etc/sysctl.conf
```


To prevent the file from being world-readable, you should set up the correct permissions on the swap file:


```
sudo chown root:root /swapfile 
sudo chmod 0600 /swapfile
```


By Etel Sverdlov
