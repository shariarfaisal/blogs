# How To Use Find and Locate to Search for Files on Linux

```Linux Basics``` ```System Tools``` ```Linux Commands```

## Introduction


One problem users run into when first learning how to work with Linux is how to find the files they are looking for.


This guide will cover how to use the aptly named find command. This will help you search for files on your system using a variety of filters and parameters. It will also briefly cover the locate command, which can be used to search for files in a different way.


# Prerequisites


To follow along with this guide, you will need access to a computer running a Linux-based operating system. This can either be a virtual private server which you’ve connected to with SSH or your local machine. Note that this tutorial was validated using a Linux server running Ubuntu 20.04, but the examples given should work on a computer running any version of any Linux distribution.


If you plan to use a remote server to follow this guide, we encourage you to first complete our Initial Server Setup guide. Doing so will set you up with a secure server environment — including a non-root user with sudo privileges and a firewall configured with UFW — which you can use to build your Linux skills.



Note: To illustrate how the find and locate commands work, the example commands in this guide search for files stored under /, or the root directory. Because of this, if you’re logged into the terminal as a non-root user, some of the example commands may include Permission denied in their output.
This is to be expected, since you’re searching for files within directories that regular users typically don’t have access to. However, these example commands should still work and be useful for understanding how these programs work.

# Finding by Name


The most obvious way of searching for files is by their name.


To find a file by name with the find command, you would use the following syntax:


```
find -name "query"


```


This will be case sensitive, meaning a search for query is different from a search for Query.


To find a file by name but ignore the case of the query, use the -iname option:


```
find -iname "query"


```


If you want to find all files that don’t adhere to a specific pattern, you can invert the search with -not:


```
find -not -name "query_to_avoid"


```


Alternatively, you can invert the search using an exclamation point (!), like this:


```
find \! -name "query_to_avoid"


```


Note that if you use !, you must escape the character with a backslash (\) so that the shell does not try to interpret it before find can act.


# Finding by Type


You can specify the type of files you want to find with the -type parameter. It works like this:


```
find -type type_descriptor query


```


Here are some of the descriptors you can use to specify the type of file:


- f: regular file
- d: directory
- l: symbolic link
- c: character devices
- b: block devices

For instance, if you wanted to find all of the character devices on your system, you could issue this command:


```
find /dev -type c


```


This command specifically only searches for devices within the /dev directory, the directory where device files are typically mounted in Linux systems:


```
Output/dev/vcsa5
/dev/vcsu5
/dev/vcs5
/dev/vcsa4
/dev/vcsu4
/dev/vcs4
/dev/vcsa3
/dev/vcsu3
/dev/vcs3
/dev/vcsa2
/dev/vcsu2
/dev/vcs2
. . .

```


You can search for all files that end in .conf with a command like the following. This example searches for matching files within the /usr directory:


```
find /usr -type f -name "*.conf"


```


```
Output/usr/src/linux-headers-5.4.0-88-generic/include/config/auto.conf
/usr/src/linux-headers-5.4.0-88-generic/include/config/tristate.conf
/usr/src/linux-headers-5.4.0-90-generic/include/config/auto.conf
/usr/src/linux-headers-5.4.0-90-generic/include/config/tristate.conf
/usr/share/adduser/adduser.conf
/usr/share/ufw/ufw.conf
/usr/share/popularity-contest/default.conf
/usr/share/byobu/keybindings/tmux-screen-keys.conf
/usr/share/libc-bin/nsswitch.conf
/usr/share/rsyslog/50-default.conf
. . .

```



Note: The previous example combines two find query expressions; namely, -type f and -name "*.conf". For any file to be returned, it must satisfy both of these expressions.
You can combine expressions like this by separating them with the -and option, but as this example shows the -and is implied any time you include two expressions. You can also return results that satisfy either expression by separating them with the -or option:
find -name query_1 -or -name query_2


This example will find any files whose names match either query_1 or query_2.

# Filtering by Time and Size


find gives you a variety of ways to filter results by size and time.


## Size


You can filter files by their size using the -size parameter. To do this, you must add a special suffix to the end of a numerical size value to indicate whether you’re counting the size in terms of bytes, megabytes, gigabytes, or another size. Here are some commonly used size suffixes:


- c: bytes
- k: kilobytes
- M: megabytes
- G: gigabytes
- b: 512-byte blocks

To illustrate, the following command will find every file in the /usr directory that is exactly 50 bytes:


```
find /usr -size 50c


```


To find files that are less than 50 bytes, you can use this syntax instead:


```
find /usr -size -50c


```


To find files in the /usr directory that are more than 700 Megabytes, you could use this command:


```
find /usr -size +700M


```


## Time


For every file on the system, Linux stores time data about access times, modification times, and change times.


- 
Access Time: The last time a file was read or written to.

- 
Modification Time: The last time the contents of the file were modified.

- 
Change Time: The last time the file’s inode metadata was changed.


You can base your find searches on these parameters using the -atime, -mtime, and -ctime options, respectively. For any of these options, you must pass a value indicating how many days in the past you’d like to search. Similar to the size options outlined previously, you can prepend these options with the plus or minus symbols to specify “greater than” or “less than”.


For example, to find files in the /usr directory that were modified within the last day, run the following command:


```
find /usr -mtime 1


```


If you want files that were accessed less than a day ago, you could run this command:


```
find /usr -atime -1


```


To find files that last had their meta information changed more than 3 days ago, you might execute the following:


```
find /usr -ctime +3


```


These options also have companion parameters you can use to specify minutes instead of days:


```
find /usr -mmin -1


```


This will give the files that have been modified in the last minute.


find can also do comparisons against a reference file and return those that are newer:


```
find / -newer reference_file


```


This syntax will return every file on the system that was created or changed more recently than the reference file.


# Finding by Owner and Permissions


You can also search for files by the user or group that owns the file using the -user and -group parameters, respectively. To find every file in the /var directory that is owned by the syslog user run this command:


```
find /var -user syslog


```


Similarly, you can specify files in the /etc directory owned by the shadow group by typing:


```
find /etc -group shadow


```


You can also search for files with specific permissions.


If you want to match an exact set of permissions, you use can this syntax specifying the permissions using octal notation:


```
find / -perm 644


```


This will match files with exactly the permissions specified.


If you want to specify anything with at least those permissions, you can precede the permissions notation with a minus sign:


```
find / -perm -644


```


This will match any files that have additional permissions. A file with permissions of 744 would be matched in this instance.


# Filtering by Depth


In this section, you will create an example directory structure that you’ll then use to explore filtering files by their depth within the structure.


If you’re following along with the examples in this tutorial, it would be prudent to create these files and directories within the /tmp/ directory. /tmp/ is a temporary directory, meaning that any files and directories within it will be deleted the next time the server boots up. This will be useful for the purposes of this guide, since you can create as many directories, files, and links as you’d like without having to worry about them clogging up your system later on.


After running the commands in this section, your /tmp/ directory will contain three levels of directories, with ten directories at the first level. Each directory (including the temporary directory) will contain ten files and ten subdirectories.


Create the example directory structure within the /tmp/ directory with the following command:


```
mkdir -p /tmp/test/level1dir{1..10}/level2dir{1..10}/level3dir{1..10}


```


Following that, populate these directories with some sample files using the touch command:


```
touch /tmp/test/{file{1..10},level1dir{1..10}/{file{1..10},level2dir{1..10}/{file{1..10},level3dir{1..10}/file{1..10}}}}


```


With these files and directories in place, go ahead and navigate into the test/ directory you just created:


```
cd /tmp/test


```


To get a baseline understanding of how find will retrieve files from this structure, begin with a regular name search that matches any files named file1:


```
find -name file1


```


```
Output./level1dir7/level2dir8/level3dir9/file1
./level1dir7/level2dir8/level3dir3/file1
./level1dir7/level2dir8/level3dir4/file1
./level1dir7/level2dir8/level3dir1/file1
./level1dir7/level2dir8/level3dir8/file1
./level1dir7/level2dir8/level3dir7/file1
./level1dir7/level2dir8/level3dir2/file1
./level1dir7/level2dir8/level3dir6/file1
./level1dir7/level2dir8/level3dir5/file1
./level1dir7/level2dir8/file1
. . .

```


This will return a lot of results. If you pipe the output into a counter, you’ll find that there are 1111 total results:


```
find -name file1 | wc -l


```


```
Output1111

```


This is probably too many results to be useful to you in most circumstances. To narrow it down, you can specify the maximum depth of the search under the top-level search directory:


```
find -maxdepth num -name query


```


To find file1 only in the level1 directories and above, you can specify a max depth of 2 (1 for the top-level directory, and 1 for the level1 directories):


```
find -maxdepth 2 -name file1


```


```
Output./level1dir7/file1
./level1dir1/file1
./level1dir3/file1
./level1dir8/file1
./level1dir6/file1
./file1
./level1dir2/file1
./level1dir9/file1
./level1dir4/file1
./level1dir5/file1
./level1dir10/file1

```


That is a much more manageable list.


You can also specify a minimum directory if you know that all of the files exist past a certain point under the current directory:


```
find -mindepth num -name query


```


You can use this to find only the files at the end of the directory branches:


```
find -mindepth 4 -name file1


```


```
Output./level1dir7/level2dir8/level3dir9/file1
./level1dir7/level2dir8/level3dir3/file1
./level1dir7/level2dir8/level3dir4/file1
./level1dir7/level2dir8/level3dir1/file1
./level1dir7/level2dir8/level3dir8/file1
./level1dir7/level2dir8/level3dir7/file1
./level1dir7/level2dir8/level3dir2/file1
. . .

```


Again, because of the branching directory structure, this will return a large number of results (1000).


You can combine the min and max depth parameters to focus in on a narrow range:


```
find -mindepth 2 -maxdepth 3 -name file1


```


```
Output./level1dir7/level2dir8/file1
./level1dir7/level2dir5/file1
./level1dir7/level2dir7/file1
./level1dir7/level2dir2/file1
./level1dir7/level2dir10/file1
./level1dir7/level2dir6/file1
./level1dir7/level2dir3/file1
./level1dir7/level2dir4/file1
./level1dir7/file1
. . .

```


Combining these options like this narrows down the results significantly, with only 110 lines returned instead of the previous 1000.


# Executing Commands on find Results


You can execute an arbitrary helper command on everything that find matches by using the -exec parameter using the following syntax:


```
find find_parameters -exec command_and_options {} \;


```


The {} is used as a placeholder for the files that find matches. The \; lets find know where the command ends.


For instance, assuming you’re still in the test/ directory you created within the /tmp/ directory in the previous step, you could find the files in the previous section that had 644 permissions and modify them to have 664 permissions:


```
find . -type f -perm 644 -exec chmod 664 {} \;


```


You could also change the directory permissions in a similar way:


```
find . -type d -perm 755 -exec chmod 700 {} \;


```


This example finds every directory with permissions set to 755 and then modifies the permissions to 700.


# Finding Files Using locate


An alternative to using find is the locate command. This command is often quicker and can search the entire file system with ease.


You can install the command on Debian or Ubuntu with apt by updating your package lists and then installing the mlocate package:


```
sudo apt update
sudo apt install mlocate


```


On Rocky Linux, CentOS, and other RedHat derived distributions, you can instead use the dnf command to install mlocate:


```
sudo dnf install mlocate


```


The reason locate is faster than find is because it relies on a database that lists all the files on the filesystem. This database is usually updated once a day with a cron script, but you can update it manually with the updatedb command. Run this command now with sudo privileges:


```
sudo updatedb


```


Remember, the locate database must always be up-to-date if you want to find new files. If you add new files before the cron script is executed or before you run the updatedb command, they will not appear in your query results.


locate allows you to filter results in a number of ways. The most fundamental way you can use it to find files is to use this syntax:


```
locate query


```


This will match any files and directories that contain the string query anywhere in their file path. To return only files whose names contain the query itself, instead of every file that has the query in the directories leading up to it, you can include the -b flag to search only for files whose “basename” matches the query:


```
locate -b query


```


To have locate only return results that still exist in the filesystem (meaning files that were not removed between the last updatedb call and the current locate call), use the -e flag:


```
locate -e query


```


You can retrieve statistics about the information that locate has cataloged using the -S option:


```
locate -S


```


```
OutputDatabase /var/lib/mlocate/mlocate.db:
    21015 directories
    136787 files
    7727763 bytes in file names
    3264413 bytes used to store database

```


This can be useful for getting a high-level understanding of how many files and directories exist on your system.


# Conclusion


Both the find and locate commands are useful tools for finding files on your system. Both are powerful commands that can be strengthened by combining them with other utilities through pipelines, but it’s up to you to decide which tool is appropriate for your given situation.


From here, we encourage you to continue experimenting with find and locate. You can read their respective man pages to learn about other options not covered in this guide, and you can analyze and manipulate search results by piping them into other commands like wc, sort, and grep.


