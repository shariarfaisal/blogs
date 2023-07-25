# Command Line Basics  Symbolic Links

```Linux Basics``` ```Linux Commands```

## Introduction


Symbolic links allow you to link files and directories to other files and directories. They go by many names including symlinks, shell links, soft links, shortcuts, and aliases. From the user’s perspective, symbolic links are very similar to normal files and directories. However, when you interact with them, they will actually interact with the target at the other end. Think of them like wormholes for your file system.


This guide provides an overview of what symbolic links are and how to to create them from a Linux command line using the ln command.


# Prerequisites


To follow along with this guide, you will need access to a computer running a Linux-based operating system. This can either be a virtual private server which you’ve connected to with SSH or your local machine. Note that this tutorial was validated using a Linux server running Ubuntu 20.04, but the examples given should work on a computer running any version of any Linux distribution.


If you plan to use a remote server to follow this guide, we encourage you to first complete our Initial Server Setup guide. Doing so will set you up with a secure server environment — including a non-root user with sudo privileges and a firewall configured with UFW — which you can use to build your Linux skills.


# Setting Up Example Directories and Files


The system call necessary to create symbolic links tends to be readily available on Unix-like and POSIX-compliant operating systems. The command we’ll be using to create the links is the ln command.


You’re welcome to use existing files on your system to practice making symbolic links, but this section provides a few commands that will set up a practice environment you can use to follow along with this guide’s examples.


Begin by creating a couple directories within the /tmp/ directory. /tmp/ is a temporary directory, meaning that any files and directories within it will be deleted the next time the server boots up. This will be useful for the purposes of this guide, since you can create as many directories, files, and links as you’d like without having to worry about them clogging up your system later on.


The following mkdir command creates three directories at once. It creates a directory named symlinks/ within the /tmp/ directory, and two directories (one named one/ and another named two/) within symlinks/:


```
mkdir -p /tmp/symlinks/{one,two}


```


Navigate into the new symlinks/ directory:


```
cd /tmp/symlinks


```


From there, create a couple sample files, one for both of the subdirectories within symlinks/. The following command creates a file named one.txt within the one/ subdirectory whose only contents are a single line reading one:


```
echo "one" > ./one/one.txt


```


Similarly, this command creates a file named two.txt within the two/ subdirectory whose only contents are a single line reading two:


```
echo "two" > ./two/two.txt


```


If you were to run tree at this point to display the contents of the entire /tmp/symlinks directory and any nested subdirectories, its output would look like this:


```
tree


```


```
Output.
├── one
│   └── one.txt
└── two
    └── two.txt

2 directories, 2 files

```



Note: If tree isn’t installed on your machine by default, you can install it using your system’s package manager. On Ubuntu, for example, you can install it with apt:
sudo apt install tree



With these sample documents in place, you’re ready to practice making symbolic links.


# Understanding Hard Links


By default, the ln command will make hard links instead of symbolic, or soft, links.


Say you have a text file. If you make a symbolic link to that file, the link is only a pointer to the original file. If you delete the original file, the link will be broken as it no longer has anything to point to.


A hard link is instead a mirror copy of an original file with the exact same contents. Like symbolic links, if you edit the contents of the original file those changes will be reflected in the hard link. If you delete the original file, though, the hard link will still work, and you can view and edit it as you would a normal copy of the original file.


Hard links serve their purpose in the world, but they should be avoided entirely in some cases. For instance, you should avoid using hard links when linking inside of a git repository as they can cause confusion.


To ensure that you’re creating symbolic links, you can pass the -s or --symbolic option to the ln command.



Note: Because symbolic links are typically used more frequently than hard links, some may find it beneficial to alias ln to ln -s:
alias ln="ln -s"


This may save only a few keystrokes, but if you find yourself making a lot of symbolic links this could add up significantly.

# Working with Symbolic Links


As mentioned previously, symbolic linking is essentially like creating a file that contains the target’s filename and path. Because a symbolic link is just a reference to the original file, any changes that are made to the original will be immediately available in the symbolic link.


One potential use for symbolic links is to create local directories in a user’s home directory pointing to files being synchronized to an external application, like Dropbox. Another might be to create a symbolic link that points to the latest build of a project that resides in a dynamically-named directory.


Using the example files and directories from the first section, go ahead and try creating a symbolic link named three that points to the one directory you created previously:


```
ln -s one three


```


Now you should have 3 directories, one of which is pointing back to another. To get a more detailed overview of the current directory structure, you can use the ls command to print the contents of the current working directory:


```
ls


```


```
Outputone  three  two

```


There are now three directories within the symlinks/ directory. Depending on your system, it may signify that three is in fact a symbolic link. This is sometimes done by rendering the name of the link in a different color, or appending it with an @ symbol.


For even greater detail, you can pass the -l argument to ls to determine where the symbolic link is actually pointing:


```
ls -l


```


```
Outputtotal 8
drwxrwxr-x 2 sammy sammy 4096 Oct 30 19:51 one
lrwxrwxrwx 1 sammy sammy    3 Oct 30 19:55 three -> one
drwxrwxr-x 2 sammy sammy 4096 Oct 30 19:51 two

```


Notice that the three link is pointing to the one directory as expected. Also, it begins with an l, which indicates it’s a link. The other two begin with d, meaning that they are regular directories.


Symbolic links can also contain symbolic links. As an example, link the one.txt file from three to the two directory:


```
ln -s three/one.txt two/one.txt


```


You should now have a file named one.txt inside of the two directory. You can check with the following ls command:


```
ls -l two/


```


```
Outputtotal 4
lrwxrwxrwx 1 sammy sammy 13 Oct 30 19:58 one.txt -> three/one.txt
-rw-rw-r-- 1 sammy sammy  4 Oct 30 19:51 two.txt

```


Depending on your terminal configuration, the link (highlighted in this example output) may be rendered in red text, indicating a broken link. Although the link was created, the way this example specified the path was relative. The link is broken because the two directory doesn’t contain a three directory with the one.txt file in it.


Fortunately, you can remedy this situation by telling ln to create the symbolic link relative to the link location using the -r or --relative argument.


Even with the -r flag, though, you won’t be able to fix the broken symbolic link. The reason for this is the symbolic link already exists, and you won’t be able to overwrite it without including the -f or --force argument as well:


```
ln -srf three/one.txt two/one.txt


```


With that, you now have two/one.txt which was linked to three/one.txt which is a link to one/one.txt.


Nesting symbolic links like this can quickly get confusing, but many applications are equipped to make such linking structures more understandable. For instance, if you were to run the tree command, the link target being shown is actually that of the original file location and not the link itself:


```
tree


```


```
Output.
├── one
│   └── one.txt
├── three -> one
└── two
    ├── one.txt -> ../one/one.txt
    └── two.txt

3 directories, 3 files

```


Now that things are linked up nicely, you can begin exploring how symbolic links work with files by altering the contents of these sample files.


To get a sense of what your files contain, run the following cat command to print the contents of the one.txt file in each of the three directories you’ve created in this guide:


```
cat {one,two,three}/one.txt


```


```
Outputone
one
one

```


Next, update the contents of the original one.txt file from the one/ directory:


```
echo "1. One" > one/one.txt


```


Then check the contents of each file again:


```
cat {one,two,three}/one.txt


```


```
Output1. One
1. One
1. One

```


As this output indicates, any changes you make to the original file will be reflected in any of its symbolic links.


Now try out the reverse. Run the following command to change the contents of one of the symbolic links. This example changes the contents of the one.txt file within the three/ directory:


```
echo "One and done" > three/one.txt


```


Then check the contents of each file once again:


```
cat {one,two,three}/one.txt


```


```
OutputOne and done
One and done
One and done

```


Because the symbolic link you changed is just a pointer to another file, any change you make to the link will be immediately reflected in the original file as well as any of its other symbolic links.


# Conclusion


Symbolic links can be incredibly useful, but they do have certain limitations. Keep in mind that if you were to move or delete the original file or directory, all of your existing symbolic links pointed to it will become broken. There’s no automatic updating in that scenario. As long as you’re careful, though, you can find many uses for symbolic links as you continue working with the command line.


