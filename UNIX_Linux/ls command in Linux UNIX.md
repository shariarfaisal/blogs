# ls command in Linux UNIX

```UNIX/Linux```

The ls command is one of the most commonly used commands in daily Linux/UNIX operations. The command is used in listing contents inside a directory and is one of the few commands beginners learn from the onset. In this guide, we will discuss Common ls commands in Linux and other parameters as well that may be used alongside the command.


# Listing Files with ls command without any arguments


The ls command without any options lists files and directories in a plain format without displaying much information like file types, permissions, modified date and time to mention just but a few. Syntax


```
$ ls

```





# Listing files in reverse order


To list files in reverse order, append the -r flag as shown Syntax


```
$ ls -r

```


 As you can see above, the order of the listing has changed from the last to the first in comparison to the previous image.


# Listing file and directory permissions with -l option


using the -l flag, you can list the permissions of the files and directories as well as other attributes such as folder names, file and directory sizes, and modified date and time. Syntax


```
$ ls -l

```





# Viewing files in a human-readable format


As you may have noticed, the file and folder sizes displayed are not easy to decipher and make sense of at first glance. To easily identify the file sizes as kilobytes (kB), Megabytes (MB) or Gigabytes (GB), append the -lh flag as shown Syntax


```
$ ls -lh

```





# Viewing Hidden files


You can view hidden files by appending the -a flag. Hidden files are usually system files that begin with a full stop or a period. Syntax


```
$ ls -a

```





# Listing files recursively


To display the directory tree of files and folders use the ls -R command as shown Syntax


```
$ ls -R

```





# Listing files and directories with the ‘/’ character at the end


If you wish to go ahead and further distinguish files from folders, use the -F flag such that folder will appear with a forward slash character ‘/’ at the end. Syntax


```
$ ls -F

```





# Displaying the inode number of files and directories


To display the inode number of files and directories, append the -i flag at the end of the ls command as shown Syntax


```
$ ls -i

```





# Displaying the UID & GID of files and directories


If you want to display the UID as well as the GId of files and directories, append the -n parameter as shown Syntax


```
$ ls -n

```





# Defining ls command in aliases


Aliases are customized or modified commands in Linux shell which are used in the place of the original commands. We can create an alias for the ls command this way Syntax


```
$ alias="ls -l"

```


What this does is that it tells the system to execute the ls -l command instead of the ls command. Be sure to observe that the output you get when running the ls command thereafter, will be as though you run the ls -l command.  To remove the added alias, run


```
unalias ls

```


# colorizing ls command output


To add some flair to the output display based on the types of files, you may want to colorize your output to easily distinguish files, folders and other attributes such as file and directory permissions. To achieve this run Syntax


```
ls --color

```


# Displaying the ls command version


If you are a bit curious as to what version of ls you are running, execute the command below


```
# ls --v
ls (GNU coreutils) 8.22
Copyright (C) 2013 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <https://gnu.org/licenses/gpl.html>.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Written by Richard M. Stallman and David MacKenzie.
#

```


You can also execute the command ls --version to print the ls command version.


# Displaying the ls command help page


To view more options and what you can do with ls simply run]


```
ls --help

```





# Accessing ls man pages


Alternatively, you can view the manpages to find out more about its usage by running


```
man ls

```


 That’s all we had for you today. We hope at this point, you will be more comfortable using the ls command in your day to day operations. Feel free to weigh in your feedback. Thanks!


