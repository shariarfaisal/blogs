# How To Use Rsync to Sync Local and Remote Directories

```Linux Basics``` ```Backups```

## Introduction


Rsync, which stands for remote sync, is a remote and local file synchronization tool. It uses an algorithm to minimize the amount of data copied by only moving the portions of files that have changed.


In this tutorial, we’ll define Rsync, review the syntax when using rsync, explain how to use Rsync to sync with a remote system, and other options available to you.


# Prerequisites


In order to practice using rsync to sync files between a local and remote system, you will need two machines to act as your local computer and your remote machine, respectively. These two machines could be virtual private servers, virtual machines, containers, or personal computers as long as they’ve been properly configured.


If you plan to follow this guide using servers, it would be prudent to set them up with administrative users and to configure a firewall on each of them. To set up these servers, follow our Initial Server Setup Guide.


Regardless of what types of machines you use to follow this tutorial, you will need to have created SSH keys on both of them. Then, copy each server’s public key to the other server’s authorized_keys file as outlined in Step 2 of that guide.


This guide was validated on machines running Ubuntu 20.04, although it should generally work with any computers running a Linux-based operating system that have rsync installed.


# Defining Rsync


Rsync is a very flexible network-enabled syncing tool. Due to its ubiquity on Linux and Unix-like systems and its popularity as a tool for system scripts, it’s included on most Linux distributions by default.


# Understanding Rsync Syntax


The syntax for rsync operates similar to other tools, such as ssh, scp, and cp.


First, change into your home directory by running the following command:


```
cd ~


```


Then create a test directory:


```
mkdir dir1


```


Create another test directory:


```
mkdir dir2


```


Now add some test files:


```
touch dir1/file{1..100}


```


There’s now a directory called dir1 with 100 empty files in it. Confirm by listing out the files:


```
ls dir1


```


```
Outputfile1    file18  file27  file36  file45  file54  file63  file72  file81  file90
file10   file19  file28  file37  file46  file55  file64  file73  file82  file91
file100  file2   file29  file38  file47  file56  file65  file74  file83  file92
file11   file20  file3   file39  file48  file57  file66  file75  file84  file93
file12   file21  file30  file4   file49  file58  file67  file76  file85  file94
file13   file22  file31  file40  file5   file59  file68  file77  file86  file95
file14   file23  file32  file41  file50  file6   file69  file78  file87  file96
file15   file24  file33  file42  file51  file60  file7   file79  file88  file97
file16   file25  file34  file43  file52  file61  file70  file8   file89  file98
file17   file26  file35  file44  file53  file62  file71  file80  file9   file99

```


You also have an empty directory called dir2. To sync the contents of dir1 to dir2 on the same system, you will run rsync and use the -r flag, which stands for “recursive” and is necessary for directory syncing:


```
rsync -r dir1/ dir2


```


Another option is to use the -a flag, which is a combination flag and stands for “archive”. This flag syncs recursively and preserves symbolic links, special and device files, modification times, groups, owners, and permissions. It’s more commonly used than -r and is the recommended flag to use. Run the same command as the previous example, this time using the -a flag:


```
rsync -a dir1/ dir2


```


Please note that there is a trailing slash (/) at the end of the first argument in the syntax of the the previous two commands and highlighted here:


```
rsync -a dir1/ dir2


```


This trailing slash signifies the contents of dir1. Without the trailing slash, dir1, including the directory, would be placed within dir2. The outcome would create a hierarchy like the following:


```
~/dir2/dir1/[files]

```


Another tip is to double-check your arguments before executing an rsync command. Rsync provides a method for doing this by passing the -n or --dry-run options. The -v flag, which means “verbose”, is also necessary to get the appropriate output. You’ll combine the a, n, and v flags in the following command:


```
rsync -anv dir1/ dir2


```


```
Outputsending incremental file list
./
file1
file10
file100
file11
file12
file13
file14
file15
file16
file17
file18
. . .

```


Now compare that output to the one you receive when removing the trailing slash, as in the following:


```
rsync -anv dir1 dir2


```


```
Outputsending incremental file list
dir1/
dir1/file1
dir1/file10
dir1/file100
dir1/file11
dir1/file12
dir1/file13
dir1/file14
dir1/file15
dir1/file16
dir1/file17
dir1/file18
. . .

```


This output now demonstrates that the directory itself was transferred, rather than only the files within the directory.


# Using Rsync to Sync with a Remote System


To use rsync to sync with a remote system, you only need SSH access configured between your local and remote machines, as well as rsync installed on both systems. Once you have SSH access verified between the two machines, you can sync the dir1 folder from the previous section to a remote machine by using the following syntax. Please note in this case, that you want to transfer the actual directory, so you’ll omit the trailing slash:


```
rsync -a ~/dir1 username@remote_host:destination_directory


```


This process is called a push operation because it “pushes” a directory from the local system to a remote system. The opposite operation is pull, and is used to sync a remote directory to the local system. If the dir1 directory were on the remote system instead of your local system, the syntax would be the following:


```
rsync -a username@remote_host:/home/username/dir1 place_to_sync_on_local_machine


```


Like cp and similar tools, the source is always the first argument, and the destination is always the second.


# Using Other Rsync Options


Rsync provides many options for altering the default behavior of the utility, such as the flag options you learned about in the previous section.


If you’re transferring files that have not already been compressed, like text files, you can reduce the network transfer by adding compression with the -z option:


```
rsync -az source destination


```


The -P flag is also helpful. It combines the flags --progress and --partial. This first flag provides a progress bar for the transfers, and the second flag allows you to resume interrupted transfers:


```
rsync -azP source destination


```


```
Outputsending incremental file list
created directory destination
source/
source/file1
              0 100%    0.00kB/s    0:00:00 (xfr#1, to-chk=99/101)
sourcefile10
              0 100%    0.00kB/s    0:00:00 (xfr#2, to-chk=98/101)
source/file100
              0 100%    0.00kB/s    0:00:00 (xfr#3, to-chk=97/101)
source/file11
              0 100%    0.00kB/s    0:00:00 (xfr#4, to-chk=96/101)
source/file12
              0 100%    0.00kB/s    0:00:00 (xfr#5, to-chk=95/101)
. . .

```


If you run the command again, you’ll receive a shortened output since no changes have been made. This illustrates Rsync’s ability to use modification times to determine if changes have been made:


```
rsync -azP source destination


```


```
Outputsending incremental file list
sent 818 bytes received 12 bytes 1660.00 bytes/sec
total size is 0 speedup is 0.00

```


Say you were to update the modification time on some of the files with a command like the following:


```
touch dir1/file{1..10}


```


Then, if you were to run rsync with -azP again, you’ll notice in the output how Rsync intelligently re-copies only the changed files:


```
rsync -azP source destination


```


```
Outputsending incremental file list
file1
            0 100%    0.00kB/s    0:00:00 (xfer#1, to-check=99/101)
file10
            0 100%    0.00kB/s    0:00:00 (xfer#2, to-check=98/101)
file2
            0 100%    0.00kB/s    0:00:00 (xfer#3, to-check=87/101)
file3
            0 100%    0.00kB/s    0:00:00 (xfer#4, to-check=76/101)
. . .

```


In order to keep two directories truly in sync, it’s necessary to delete files from the destination directory if they are removed from the source. By default, rsync does not delete anything from the destination directory.


You can change this behavior with the --delete option. Before using this option, you can use -n, the --dry-run option, to perform a test to prevent unwanted data loss:


```
rsync -an --delete source destination


```


If you prefer to exclude certain files or directories located inside a directory you are syncing, you can do so by specifying them in a comma-separated list following the --exclude= option:


```
rsync -a --exclude=pattern_to_exclude source destination


```


If you have a specified pattern to exclude, you can override that exclusion for files that match a different pattern by using the --include= option:


```
rsync -a --exclude=pattern_to_exclude --include=pattern_to_include source destination


```


Finally, Rsync’s --backup option can be used to store backups of important files. It’s used in conjunction with the --backup-dir option, which specifies the directory where the backup files should be stored:


```
rsync -a --delete --backup --backup-dir=/path/to/backups /path/to/source destination


```


# Conclusion


Rsync can streamline file transfers over networked connections and add robustness to local directory syncing. The flexibility of Rsync makes it a good option for many different file-level operations.


A mastery of Rsync allows you to design complex backup operations and obtain fine-grained control over how and what is transferred.


