# How To Enable User Quotas

```Linux Basics```



Status: Deprecated
This article is deprecated and no longer maintained.
Reason:
The information in this article is out of date and untested. More recent information about quotas is listed below.
See Instead:
How To Set Filesystem Quotas on Ubuntu 18.04 or Debian 9.

Quotas can be used on servers to set limits on how much diskspace an individual server user can take up on a server. They can be edited in the /etc/fstab file.


In order to enable quotas,  first open the /etc/fstab file:


```
nano /etc/fstab
```


Within that file, edit the following line, adding in the word, “usrquota”:


```
LABEL=DOROOT	   /               ext4    errors=remount-ro,usrquota 0       1
```


Save and exit


The /etc/fstab file should now look like this:


```
LABEL=DOROOT       /               ext4    errors=remount-ro,usrquota 0       1
none             /dev/shm      tmpfs   defaults                    0 0
```


To finish up, remount the file system whose fstab entry has ben changed:


```
 mount -o remount /
```


By Etel Sverdlov
