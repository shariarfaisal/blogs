#  dev null in Linux

```UNIX/Linux```

/dev/null in Linux is a null device file. This will discard anything written to it, and will return EOF on reading.


This is a command-line hack that acts as a vacuum, that sucks anything thrown to it.


Let’s take a look at understanding what it means, and what we can do with this file.



# /dev/null Properties


This will return an End of File (EOF) character if you try to read it using the cat command.


```
cat /dev/null

```


This is a valid file, which can be verified using


```
stat /dev/null

```


This gives me an output of


```
  File: /dev/null
  Size: 0               Blocks: 0          IO Block: 4096   character special file
Device: 6h/6d   Inode: 5           Links: 1     Device type: 1,3
Access: (0666/crw-rw-rw-)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2020-02-04 13:00:43.112464814 +0530
Modify: 2020-02-04 13:00:43.112464814 +0530
Change: 2020-02-04 13:00:43.112464814 +0530

```


This shows that this file has a size of 0 bytes, has zero blocks allocated to it. The file permissions are also set that anyone can read/write to it, but cannot execute it.


Since it is not an executable file, we cannot use piping using | operator to redirect to /dev/null. The only way is to use file redirections (>, >>, or <, <<).


The below diagram shows that /dev/null is indeed a valid file.


File Tables
Let’s now take a look at some common use cases for /dev/null.



# Redirection to /dev/null in Linux


We can discard any output of a script that we use by redirecting to /dev/null.


For example, we can try discarding echo messages using this trick.


```
echo 'Hello from JournalDev' > /dev/null

```


You will not get any output since it is discarded!


Let’s try running a command incorrectly and pipe it’s output to /dev/null.


```
cat --INCORRECT_OPTION > /dev/null

```


We still get an output like this:


```
cat: unrecognized option '--INCORRECT'
Try 'cat --help' for more information.

```


Why is this happening? This is because the error messages are coming from stderr, but we are only discarding output from stdout.


We need to take stderr into account as well.


## Discard error messages


Let us redirect the stderr to /dev/null, along with stdout. We can use the file descriptor for stderr(=2) for this.


```
cat --INCORRECT_OPTION > /dev/null 2>/dev/null

```


This will give us what we need!


There is another way of doing the same; by redirecting stderr to stdout first, and then redirect stdout to /dev/null.


The syntax for this will be:


```
command > /dev/null 2>&1

```


Notice the 2>&1 at the end. We redirect stderr(2) to stdout(1). We use &1 to mention to the shell that the destination file is a file descriptor and not a file name.


```
cat --INCORRECT_OPTION > dev/null 2>&1

```


So if we use 2>1, we will only redirect stderr to a file called 1. This is not what we want!



# Conclusion


Hopefully, this clears things up a bit, so that you can now use /dev/null in Linux, knowing what it means! Feel free to ask questions in the comment section below.



# References


- StackOverflow Question


