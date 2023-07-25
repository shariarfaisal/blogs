# How to use PhotoRec to Recover Deleted Files in Linux Ubuntu

```UNIX/Linux```

Accidentally deleted files or photos? In this tutorial, we’ll learn how to recover deleted files in Linux using PhotoRec. In a previous tutorial, we discussed the steps to recover deleted files using a Linux utility named TestDisk and the PhotoRec utility is created by the same company. Let’s find out how to use PhotoRec to recover deleted files.


# What is PhotoRec?


TestDisk was created by CGSecurity to recover deleted partitions. PhotoRec, on the other hand, was created to recover media files that were deleted from SD Cards and other removable media. That’s why the name “PhotoRec” which is short for “Photo Recovery”. That’s not to say PhotoRec cannot be used for other file types, you sure can.


# How To Recover Deleted Files in Linux using PhotoRec?


Before we begin, we need to install PhotoRec on our Linux system. It comes packaged with the testdisk utility and not as a separate package.


## 1. Installing PhotoRec on Linux


To install PhotoRec, run the below command:


```
sudo apt -y install testdisk

```


Once the setup is complete, you can download and run the Photorec utility using the command below:


```
sudo photorec

```


## 2. Running PhotoRec and Begin Scanning For Deleted Files


For this demonstration, I’ve created a random image file and deleted it. Let’s go ahead and recover this file.


Photorec Delete File
Let’s fire up PhotoRec in our terminal. To make things easy, navigate to the directory that you want to run the recovery on prior to running the command.


```
sudo photorec

```


Photorec Default
When you’ve started PhotoRec, select the hard drive that you want to run the restore operation on and hit the enter key.


The next screen will ask you to select the partition that you want to run the recovery process on.


Photorec Partition Selection
Before you proceed, make sure you select the file type from the file options menu which you can access on the partition selection screen.


As we know, we’re only looking for our JPG file, I’ve selected that extension. Anything else is unnecessary and will just consume more time. Select the file type that you’re looking for and proceed.


Photorec File Type Options
Next is to select the partition type which in our case is ext4.


Photorec Partition Type
Now select if you want the utility to only look at free sectors or the entire drive.


Photo Rec Partition Search
You might have noticed, when I ran the command, I was in the ~/Desktop directory.


This is where the command will start looking at by default unless you navigate to a specific folder on the next screen.


Photorec Directory Selection
Once you’ve finalized the folder you want to start looking into, press the letter C and the program will begin searching for files.


## 3. Restoring Recovered Files


Great! So we’re all set to let PhotoRec restore deleted files for us. It may take some time depending on how many file types you’ve selected.


Photorec Files Recovery Complete
A folder named recup_dir will start to restore all the files that were recovered. You can access the files even when the recovery is in progress.


Photorec Recovered Files
Great, we now have a list of all the files that we deleted previously. You can look for the file that you want here since the filenames aren’t restored by PhotoRec.


# Why Data Recovery Works?


Noticed how saving a file on the hard disk takes time, but deleting is almost instantaneous? Let’s understand that first.


When you store data on your hard drive, the data is stored in blocks. Each block contains a piece of the data. The first block usually contains the metadata for the file in question. Each block of data is written one at a time at the speed of the hard drive.


But when we delete a file, only the first block which contains the metadata is deleted. The operating system no longer can detect the file because it’s metadata is lost and hence considers the blocks free for writing new data.


This is where recovery tools come in. Since only the metadata is lost, the job of the tools is to make the meta available for the operating system for reading.


They read the hard drive sectors one by one, block by block, and find correlated blocks. Once all the correlated blocks are found, the recovery utilities remake the metadata.


And that’s how you are able to recover a deleted file.


# How PhotoRec Works


Like other file recovery utilities, PhotoRec scans data sectors on the hard drive to find the data size. Once it finds the data size, and the hard drive and data is intact (not defragmented or overwritten), PhotoRec begins the data recovery process by looking for adjacent data blocks and recreating the meta for them.


Since the utility can’t search for a specific file, it will return all the files that are found and save it in a folder. You can then sort through the files and restore the one required.


At the end of the process, all the files that were still lying around on your hard drive will be available for you to restore.


# Conclusion


I hope that you’ve been able to use PhotoRec to recover deleted files on your Linux system. There are a lot of other utilities that you can also try if PhotoRec didn’t work for you.


Here’s a list of top 20 data recovery tools for Linux. I’m sure you’ll find one that suits your needs best!


