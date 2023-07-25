# An Introduction to LVM Concepts  Terminology  and Operations

```Linux Basics``` ```System Tools``` ```Storage``` ```Conceptual```

## Introduction


LVM, or Logical Volume Management, is a storage device management technology that gives users the power to pool and abstract the physical layout of component storage devices for flexible administration. Utilizing the device mapper Linux kernel framework, the current iteration, LVM2, can be used to gather existing storage devices into groups and allocate logical units from the combined space as needed.


The main advantages of LVM are increased abstraction, flexibility, and control. Logical volumes can have meaningful names like “databases” or "root-backup”. Volumes can also be resized dynamically as space requirements change, and migrated between physical devices within the pool on a running system or exported. LVM also offers advanced features like snapshotting, striping, and mirroring.


In this guide, you’ll learn how LVM works and practice basic commands to get up and running quickly on a bare metal machine.


# LVM Architecture and Terminology


Before diving into LVM administrative commands, it is important to have a basic understanding of how LVM organizes storage devices and some of the terminology it employs.


## LVM Storage Management Structures


LVM functions by layering abstractions on top of physical storage devices. The basic layers that LVM uses, starting with the most primitive, are:


- 
Physical Volumes: The LVM utility prefix for physical volumes is pv.... This  physicallyl blocks devices or other disk-like devices (for example, other devices created by device mapper, like RAID arrays) and are used by LVM as the raw building material for higher levels of abstraction. Physical volumes are regular storage devices. LVM writes a header to the device to allocate it for management.

- 
Volume Groups: The LVM utility prefix for volume groups is vg....
LVM combines physical volumes into storage pools known as volume groups. Volume groups abstract the characteristics of the underlying devices and function as a unified logical device with combined storage capacity of the component physical volumes.

- 
Logical Volumes: The LVM utility prefix for logical volumes is  lv..., generic LVM utilities might begin with lvm.... A volume group can be sliced up into any number of logical volumes. Logical volumes are functionally equivalent to partitions on a physical disk, but with much more flexibility. Logical volumes are the primary component that users and applications will interact with.


LVM can be used to combine physical volumes into volume groups to unify the storage space available on a system. Afterwards, administrators can segment the volume group into arbitrary logical volumes, which act as flexible partitions.


## Understanding Extents


Each volume within a volume group is segmented into small, fixed-size chunks called extents. The size of the extents is determined by the volume group. All volumes within the group conform to the same extent size.


The extents on a physical volume are called physical extents, while the extents of a logical volume are called logical extents. A logical volume is a mapping that LVM maintains between logical and physical extents. Because of this relationship, the extent size represents the smallest amount of space that can be allocated by LVM.


Extents are behind much of the flexibility and power of LVM. The logical extents that are presented as a unified device by LVM do not have to map to continuous physical extents. LVM can copy and reorganize the physical extents that compose a logical volume without any interruption to users. Logical volumes can also be expanded or shrunk by adding extents to, or removing extents from, the volume.


# Common Use Cases


Now that you are familiar with some of the terminology and structures LVM uses, you can explore some common ways to use LVM. You’ll start with a procedure that will use two physical disks to form four logical volumes.


## Marking the Physical Devices as Physical Volumes


Begin by scanning the system for block devices that LVM can access and manage. You can do this with the following command:


```
sudo lvmdiskscan


```


The output will return all available block devices that LVM can interact with:


```
Output /dev/ram0 [   64.00 MiB] 
 /dev/sda  [   200.00 GiB] 
 /dev/ram1 [   64.00 MiB] 

 . . .

 /dev/ram15 [   64.00 MiB] 
 /dev/sdb  [   100.00 GiB] 
 2 disks
 17 partitions
 0 LVM physical volume whole disks
 0 LVM physical volumes

```


In this example, notice that there are currently two disks and 17 partitions. The partitions are mostly /dev/ram* partitions that are used in the system as a RAM disk for performance enhancements. The disks in this example are /dev/sda, which has 200G of space, and /dev/sdb, which has 100G.



Warning: Make sure to double-check that the devices you intend to use with LVM do not have any important data already written to them. Using these devices within LVM will overwrite the current contents. If you have important data on your server, make backups before proceeding.

Now that you know the physical devices you want to use, mark them as physical volumes within LVM using the pvcreate command:


```
sudo pvcreate /dev/sda /dev/sdb


```


```
Output Physical volume "/dev/sda" successfully created
 Physical volume "/dev/sdb" successfully created

```


This will write an LVM header to the devices to indicate that they are ready to be added to a volume group.


Verify that LVM has registered the physical volumes by running pvs:


```
sudo pvs


```


```
Output PV     VG  Fmt Attr PSize  PFree 
 /dev/sda    lvm2 --- 200.00g 200.00g
 /dev/sdb    lvm2 --- 100.00g 100.00g

```


Note that both of the devices are present under the PV column, which stands for physical volume.


## Adding the Physical Volumes to a Volume Group


Now that you have created physical volumes from your devices, you can create a volume group. Most of the time, you only have a single volume group per system for maximum flexibility in allocation. The following volume group example is named LVMVolGroup. You can name your volume group whatever you’d like.


To create the volume group and add both of your physical volumes to it, run:


```
sudo vgcreate LVMVolGroup /dev/sda /dev/sdb


```


```
Output Volume group "LVMVolGroup" successfully created

```


Checking the pvs output again will indicate that your physical volumes are now associated with the new volume group:


```
sudo pvs


```


```
Output PV     VG     Fmt Attr PSize  PFree 
 /dev/sda  LVMVolGroup lvm2 a-- 200.00g 200.00g
 /dev/sdb  LVMVolGroup lvm2 a-- 100.00g 100.00g

```


List a short summary of the volume group with vgs:


```
sudo vgs


```


```
Output VG     #PV #LV #SN Attr  VSize  VFree 
 LVMVolGroup  2  0  0 wz--n- 299.99g 299.99g

```


Your volume group currently has two physical volumes, zero logical volumes, and has the combined capacity of the underlying devices.


## Creating Logical Volumes from the Volume Group Pool


Now that you have a volume group available, you can use it as a pool to allocate logical volumes from. Unlike conventional partitioning, when working with logical volumes, you do not need to know the layout of the volume since LVM maps and handles this for you. You only need to supply the size of the volume and a name.


In the following example, you’ll create four separate logical volumes out of your volume group:


- 10G “projects” volume
- 5G “www” volume for web content
- 20G “db” volume for a database
- “workspace” volume that will fill the remaining space

To create logical volumes, use the lvcreate command. You must pass in the volume group to pull from, and can name the logical volume with the -n option. To specify the size directly, you can use the -L option. If, instead, you wish to specify the size in terms of the number of extents, you can use the -l option.


Create the first three logical volumes with the -L option:


```
sudo lvcreate -L 10G -n projects LVMVolGroup
sudo lvcreate -L 5G -n www LVMVolGroup
sudo lvcreate -L 20G -n db LVMVolGroup


```


```
Output Logical volume "projects" created.
 Logical volume "www" created.
 Logical volume "db" created.

```


You can view the logical volumes and their relationship to the volume group by selecting a custom output from the vgs command:


```
sudo vgs -o +lv_size,lv_name


```


```
Output VG     #PV #LV #SN Attr  VSize  VFree  LSize LV   
 LVMVolGroup  2  3  0 wz--n- 299.99g 264.99g 10.00g projects
 LVMVolGroup  2  3  0 wz--n- 299.99g 264.99g 5.00g www   
 LVMVolGroup  2  3  0 wz--n- 299.99g 264.99g 20.00g db

```


In this example, you added the last two columns of the output. It indicates how much space is allocated to your logical volumes.


Now, you can allocate the rest of the space in the volume group to the "workspace" volume using the -l flag, which works in extents. You can also provide a percentage and a unit to better communicate your intentions. In this example, allocate the remaining free space, so you can pass in 100%FREE:


```
sudo lvcreate -l 100%FREE -n workspace LVMVolGroup


```


```
Output Logical volume "workspace" created.

```


Checking the volume group information with the custom vgs command, notice that you have used up all of the available space:


```
sudo vgs -o +lv_size,lv_name


```


```
Output VG     #PV #LV #SN Attr  VSize  VFree LSize  LV    
 LVMVolGroup  2  4  0 wz--n- 299.99g  0  10.00g projects 
 LVMVolGroup  2  4  0 wz--n- 299.99g  0  5.00g www   
 LVMVolGroup  2  4  0 wz--n- 299.99g  0  20.00g db    
 LVMVolGroup  2  4  0 wz--n- 299.99g  0 264.99g workspace

```


The workspace volume has been created and the LVMVolGroup volume group is completely allocated.


## Formatting and Mounting the Logical Volumes


Now that you have logical volumes, you can use them as normal block devices.


The logical devices are available within the /dev directory like other storage devices. You can access them in two places:


- /dev/volume_group_name/logical_volume_name
- /dev/mapper/volume_group_name-logical_volume_name

To format your four logical volumes with the Ext4 filesystem, run the following commands:


```
sudo mkfs.ext4 /dev/LVMVolGroup/projects
sudo mkfs.ext4 /dev/LVMVolGroup/www
sudo mkfs.ext4 /dev/LVMVolGroup/db
sudo mkfs.ext4 /dev/LVMVolGroup/workspace


```


Alternatively, you can run the following:


```
sudo mkfs.ext4 /dev/mapper/LVMVolGroup-projects
sudo mkfs.ext4 /dev/mapper/LVMVolGroup-www
sudo mkfs.ext4 /dev/mapper/LVMVolGroup-db
sudo mkfs.ext4 /dev/mapper/LVMVolGroup-workspace


```


After formatting, create mount points:


```
sudo mkdir -p /mnt/{projects,www,db,workspace}


```


Then mount the logical volumes to the appropriate location:


```
sudo mount /dev/LVMVolGroup/projects /mnt/projects
sudo mount /dev/LVMVolGroup/www /mnt/www
sudo mount /dev/LVMVolGroup/db /mnt/db
sudo mount /dev/LVMVolGroup/workspace /mnt/workspace


```


To make the mounts persistent, use your preferred text editor to add them to /etc/fstab file. The following example uses nano:


```
sudo nano /etc/fstab


```


/etc/fstab
```
. . .

/dev/LVMVolGroup/projects /mnt/projects ext4 defaults,nofail 0 0
/dev/LVMVolGroup/www /mnt/www ext4 defaults,nofail 0 0
/dev/LVMVolGroup/db /mnt/db ext4 defaults,nofail 0 0
/dev/LVMVolGroup/workspace /mnt/workspace ext4 defaults,nofail 0 0

```


After editing your file, save and exit. If you’re using nano, press CTRL+c, then y, then ENTER.


The operating system should now mount the LVM logical volumes automatically at boot.


# Conclusion


You now have an understanding of the various components that LVM manages to create a flexible storage system, and how to get storage devices up and running in an LVM setup.


To learn more about working with LVM, check out our guide to using LVM with Ubuntu 18.04.


