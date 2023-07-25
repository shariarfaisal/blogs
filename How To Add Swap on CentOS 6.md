# How To Add Swap on CentOS 6

```Linux Basics``` ```CentOS```










# Status: Deprecated


This article covers a version of CentOS that is no longer supported. If you are currently operating a server running CentOS 6, we highly recommend upgrading or migrating to a supported version of CentOS.


Reason:
CentOS 6 reached end of life (EOL) on November 30th, 2020 and no longer receives security patches or updates. For this reason, this guide is no longer maintained.


See Instead:
This guide might still be useful as a reference, but may not work on other CentOS releases. If available, we strongly recommend using a guide written for the version of CentOS you are using.


The following DigitalOcean tutorial may be of immediate interest, as it outlines adding swap space on a CentOS 7 server:




- How To Add Swap on CentOS 7





## About Linux Swapping


Linux RAM is composed of chunks of memory called pages. To free up pages of RAM, a “linux swap” can occur and a page of memory is copied from the RAM to preconfigured space on the hard disk. Linux swaps allow a system to harness more memory than was originally physically available.


However, swapping does have disadvantages. Because hard disks have a much slower memory than RAM, server performance may slow down considerably. Additionally, swap thrashing can begin to take place if the system gets swamped from too many  files being  swapped in and out.


Note
Although swap is generally recommended for systems utilizing traditional spinning hard drives, using swap with SSDs can cause issues with hardware degradation over time.  Due to this consideration, we do not recommend enabling swap on DigitalOcean or any other provider that utilizes SSD storage.  Doing so can impact the reliability of the underlying hardware for you and your neighbors.
If you need to improve the performance of your server, we recommend upgrading your Droplet.  This will lead to better results in general and will decrease the likelihood of contributing to hardware issues that can affect your service.
# Check for Swap Space


Before we proceed to set up a swap file, we need to check if any swap files have been enabled  by looking at the summary of swap usage.


```
swapon -s
```


If nothing is returned, the summary is empty and no swap file exists.


# Check the File System


After we know that we do not have a swap file enabled, we can check how much space we have on the server with the df command. The swap file will take 512MB— since we are only using up about 7% of the /dev/hda, we can proceed.


```
df
Filesystem           1K-blocks      Used Available Use% Mounted on
/dev/hda              20642428   1347968  18245884   7% /
```


# Create and Enable the  Swap File


Now it’s time to create the swap file itself using the dd command :


```
sudo dd if=/dev/zero of=/swapfile bs=1024 count=512k
```


“of=/swapfile” designates the file’s name. In this case the name is swapfile.


Subsequently we are going to prepare the swap file by creating a linux swap area:


```
sudo mkswap /swapfile
```


The results display:


```
Setting up swapspace version 1, size = 536866 kB
```


Finish up by activating the swap file:


```
sudo swapon /swapfile
```


You will then be able to see the new swap file when you view the swap summary.


```
 swapon -s
Filename				Type		Size	Used	Priority
/swapfile                               file		524280	0	-1
```


This file will last on the server until the machine reboots.  You can ensure that the swap is permanent by adding  it to the fstab file.


Open up the file:


```
sudo nano /etc/fstab
```


Paste in the following line:


```
/swapfile          swap            swap    defaults        0 0

```


To prevent the file from being world-readable, you should set up the correct permissions on the swap file:


```
chown root:root /swapfile 
chmod 0600 /swapfile
```


# How To Configure Swappiness


The operating system kernel can adjust how often it relies on swap through a configuration parameter known as swappiness.


To find the current swappiness settings, type:


```
<pre>cat /proc/sys/vm/swappiness</pre>
<pre>60</pre>

```


Swapiness can be a value from 0 to 100.  Swappiness near 100 means that the operating system will swap often and usually, too soon.  Although swap provides extra resources, RAM is much faster than swap space.  Any time something is moved from RAM to swap, it slows down.


A swappiness value of 0 means that the operating will only rely on swap when it absolutely needs to.  We can adjust the swappiness with the sysctl command:


```
<pre>sysctl vm.swappiness=10</pre>
<pre>vm.swappiness=10</pre>

```


If we check the system swappiness again, we can confirm that the setting was applied:


```
<pre>cat /proc/sys/vm/swappiness</pre>
<pre>10</pre>

```


To make your VPS automatically apply this setting every time it boots up, you can add the setting to the /etc/sysctl.conf file:


```
<pre>sudo nano /etc/sysctl.conf</pre>
<pre># Search for the vm.swappiness setting.  Uncomment and change it as necessary.
vm.swappiness=10</pre>

```


By Etel Sverdlov
