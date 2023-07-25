# Create a Partition in  Linux - A Step-by-Step Guide

```UNIX/Linux```

In this tutorial, we’ll be covering the steps to create a partition in Linux. This can help you allocate different memory regions for specific uses. Creating partitions can also help you install multiple operating systems on your machine and minimize the damage in case of disk corruption.


# How to Create a Partition in Linux?


In this tutorial, we will utilize the fdisk command to create a disk partition. The fdisk utility is a text-based command-line utility for viewing and managing disk partitions on a Linux system.


Before we create a partition on our system, we need to list all the partitions on our system. This is essential as we need to choose a disk before we partition it.


To view all the partitions currently on your system, we use the following command.


```
sudo fdisk -l

```


You might be prompted to enter your password again to verify your sudo privileges. Here we called the fdisk command with the -l to list the partitions. You should get an output similar to the following.


Fdisk List
Now, we choose one disk from this list to partition. For this tutorial, we will choose the disk. To create partitions, we use the ‘command mode’ of the fdisk command. To enter the command mode, we use this command in our terminal.


```
sudo fdisk [disk path]

```


If you see an output similar to this, you have successfully entered the command mode.


Partitioning A Disk
## Using the command mode


Once we enter the command mode, many beginners might get confused due to the unfamiliar interface. The command mode of fdisk uses single character commands to specify the desired action for the system. You can get a list of available commands by pressing ‘m’, as shown below.


Fdisk M
## Creating a partition


Our main objective here is to create a partition. To create a new partition, we use the command ‘n’. This will prompt you to specify the type of partition which you wish to create.


If you wish to create a logical partition, choose ‘l’. Alternatively, you can choose ‘p’ for a primary partition. For this tutorial, we will create a primary partition.


Create Partition
Now, we will be asked to specify the starting sector for our new partition. Press ENTER to choose the first available free sector on your system. Next, you’ll be prompted to select the last sector for the partition.


Either press ENTER to use up all the available space after your first sector or specify the size for your partition.


Sector Type
As shown in the screenshot above, we chose to create a 10 MB partition for this demonstration. Here ‘M’ specifies the unit as megabytes. You can use ‘G’ for gigabytes.


If you don’t specify a unit, the unit will be assumed to be sectors. Hence +1024 will mean 1024 sectors from the starting sector.


## Setting the partition type


Once we create a partition, Linux sets the default partition type as ‘Linux’. However, suppose we wish my partition type to be the ‘Linux LVM’ partition. To change the ID for our partition, we will use the command ‘t’.


Now, we get prompted to enter the HEX code for our desired partition ID. We don’t remember the HEX code for the partition types on top of our heads.


So we will take the help of the ‘L’ command to list all the HEX codes for the available partition types. This list should look as shown below.


Partition Type List
We see that the HEX code 8e is the partition ID for the ‘Linux LVM’ partition type. Hence, we will enter the required HEX code. The following output gives us the confirmation that our partition ID has been changed successfully.


Partition Code
## Finalising the changes


Now that we have created a new partition and given it our desired partition ID, we need to confirm our changes. All the changes made until this point are saved in the memory, waiting to be written on our disk.


We use the command ‘p’ to see the detailed list of partitions for our current disk as seen in the screenshot below.


List of all partitions
This allows us to confirm all the changes we have done to the disk before making them permanent. Once you have verified the changes, press ‘w’ to write the new partition on your disk.


If you don’t wish to permanently write your new partition to the disk, you can enter the command ‘q’. This will exit the fdisk command mode without saving any changes.


## Formatting a partition


Once you create a new partition, it is advisable to format your new partition using the appropriate mkfs command.


This is because using a new partition without formatting it may cause issues in the future. To see the list of all available mkfs commands, we enter the following in our command line.


```
sudo mkfs

```


This gives us a list of available mkfs commands. If we wish to format a partition on our current disk with the ext4 file system, we use this command.


```
sudo mkfs.ext4 [partition path]

```


# Wrapping up


That’s it! You now know how to create a partition in Linux using the fdisk command… You can reserve space for specific tasks. And in case one partition gets corrupted, you don’t need to worry about data on other partitions.


As each partition is treated as a separate disk, data on other partitions remains safe. The fdisk utility is a powerful tool for the task of managing disk partitions, but it can often be confusing for new users.


We hope this tutorial was able to help you understand how to create a new disk partition in Linux using the fdisk utility. If you have any feedback, queries or suggestions, feel free to reach out to us in the comments below.


