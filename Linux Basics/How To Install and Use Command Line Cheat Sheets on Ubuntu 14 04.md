# How To Install and Use Command Line Cheat Sheets on Ubuntu 14 04

```Linux Basics``` ```Ubuntu``` ```Miscellaneous```

## Introduction


Cheat is a command line based Python program that allows system administrators to view and store helpful cheat sheets. It retrieves plain-text examples of a chosen command in order to remind the user of options, arguments, or common uses. Cheat is ideal for “commands that you use frequently, but not frequently enough to remember.”


Sheets are small portable text files that can be copied across multiple Linux/Unix systems; they are called and viewed like any other command line program. Base sheets for common programs are provided but you can add custom new sheets, too.


# Prerequisites


To follow this tutorial, you will need:


- 
One Ubuntu 14.04 Droplet

- 
A sudo non-root user, which you can set up by following the How To Add and Delete Users on an Ubuntu 14.04 VPS tutorial


# Step 1 — Installing Cheat


Before installing Cheat, we need make sure everything’s up to date on the system.


```
sudo apt-get update && sudo apt-get upgrade


```


Confirm by entering y for any prompts in this step.


Installing Cheat is best done with the Python package manager Pip, so install Pip next.


```
sudo apt-get install python-pip


```


Cheat itself only depends upon two Python packages, both of which are conveniently included with Pip’s Cheat package. Finally, install Cheat.


```
sudo pip install cheat


```


A successful install of Cheat will output these lines:


sudo pip install cheat output
```
Successfully installed cheat docopt pygments
Cleaning up...

```


We can confirm that Cheat is installed and working by running it with its -v option.


```
cheat -v


```


This outputs the version of Cheat that we have installed.


cheat -v output
```
cheat 2.1.10

```


# Step 2 — Setting the Text Editor


Before we can go on to create our own cheat sheets, Cheat needs to know which text editor we would like to use to edit sheets by default. To do this, we must create and set an environment variable called EDITOR. For more information on shell and environment variables, you can read the How To Read and Set Environmental and Shell Variables tutorial.


Because nano is already installed on Ubuntu and is generally easy to learn, we’ll set it as our preferred text editor with the command below. However, you can use vim, emacs, or your favorite text editor instead.


```
export EDITOR="/usr/bin/nano"


```


We can confirm this was successful by typing:


```
printenv EDITOR


```


This will output the new $EDITOR environment variable’s contents:


printenv EDITOR output
```
/usr/bin/nano

```


To make this change persistent and permanent across all future shell sessions, you must add the environment variable declaration to your .bashrc file. This is one of several files that are run at the start of a bash shell session.


Open this file for editing:


```
nano ~/.bashrc


```


Then add the same export command:


~/.bashrc
```
. . .
# If not running interactively, don't do anything
case $- in
    *i*) ;;
      *) return;;
esac

export EDITOR="/usr/bin/nano"

# don't put duplicate lines or lines starting with space in the history.
# See bash(1) for more options
HISTCONTROL=ignoreboth
. . .

```


Save and exit the file by pressing CTRL+X and then Y followed by ENTER.


# Step 3 — Customizing Cheat (Optional)


In this step, we’ll customize Cheat by enabling syntax highlighting and command line auto-completion.


When using a terminal emulator that has color support, you can enable syntax highlighting for your sheets by exporting a shell environment variable named CHEATCOLORS defined as true:


```
export CHEATCOLORS=true


```


Now whenever you retrieve cheat sheets, they will be formatted with colored syntax highlighting. If you like this feature, you can make it persistent and permanent across shell sessions by adding the export command to your .bashrc file.


Open the .bashrc file again:


```
nano ~/.bashrc


```


Then add the new CHEATCOLORS variable below the EDITOR variable:


~/.bashrc
```
. . .
# If not running interactively, don't do anything
case $- in
    *i*) ;;
      *) return;;
esac

export EDITOR="/usr/bin/nano"
export CHEATCOLORS=true

# don't put duplicate lines or lines starting with space in the history
. . .

```


Save and close the file.


Next, to enable command line auto-completion, we need to put a script in the /etc/bash_completion.d/ directory. Change to this directory.


```
cd /etc/bash_completion.d/


```


Then download the script we need from Cheat’s GitHub project page.


```
sudo wget https://raw.githubusercontent.com/chrisallenlane/cheat/master/cheat/autocompletion/cheat.bash


```


Now enter bash into the current shell to pick up the changes.


```
bash


```


Tab auto-completion for Cheat is now enabled. If you type cheat followed by a space, pressing the TAB key twice will give you a list of commands.


cheat tab auto-completion output
```
cheat 
7z           asciiart     chown        df           du          
grep         indent       jrnl         mkdir        netstat
. . .

```


# Step 4 — Running Cheat


To run Cheat in its most basic form, you call it like any other command, followed by an existing cheat sheet name.


Here is an example of how to do this with one of the default sheets that comes included with Cheat, for the tail command (which outputs the last few lines of a file).


```
cheat tail


```


You’ll then see this output:


cheat tail output
```
# To show the last 10 lines of file
tail file

# To show the last N lines of file
tail -n N file

# To show the last lines of file starting with the Nth
tail -n +N file

# To show the last N bytes of file
tail -c N file

# To show the last 10 lines of file and to wait for file to grow
tail -f file

```


To see what other other existing cheat sheets are available to us, run Cheat with its  -l option.


```
cheat -l


```


This lists all available sheets and their location on the server.


# Step 5 — Creating and Editing Cheat Sheets


Although the base provisional sheets included with Cheat are useful and varied, they are not all inclusive of every shell command or program available to us. The real benefit we can get from Cheat comes with adding our own custom sheets.


For example, there is no sheet for the networking program ping:


```
cheat ping


```


cheat ping output
```
No cheatsheet found for ping

```


Let’s make one to serve as an example of how to create and add a new sheet. First, invoke Cheat on the command line again, this time followed by -e and the name of the sheet we are making it for.


```
cheat -e ping


```


Cheat will create and open the relevant file for editing using the $EDITOR variable we set earlier.


Add a useful ping command example to the beginning of this new sheet, complete with a comment (indicated by #) that explains what the command does when entered. Here is one such command you could enter in the file:


~/.cheat/ping
```
# ping a host with a total count of 15 packets overall.    
ping -c 15 www.example.com

```


Save and exit the file as before. Next let’s test the new sheet by running cheat ping again.


```
cheat ping


```


This time, we’ll see the cheat sheet we just added.


cheat ping output
```
# ping a host with a total count of 15 packets overall.    
ping -c 15 www.example.com

```


To modify an existing sheet, we can use the -e option again.


```
cheat -e ping 


```


The ping sheet is now open and we can add more examples or content. For example, we can add the following:


~/.cheat/ping
```
# ping a host with a total count of 15 packets overall.    
ping -c 15 www.example.com

# ping a host with a total count of 15 packets overall, one every .5 seconds (faster ping). 
ping -c 15 -i .5 www.example.com

```


# Step 6 — Searching Cheat Sheets


Cheat has a built-in search function triggered with the -s option. This will pick up any and all occurrences of the text you provide it with. For example:


```
cheat -s packets


```


This command will output all the lines featuring the term “packets” and the sheet they originate from.


cheat -s packets utput
```
nmap:
  # --min-rate=X => min X packets / sec

ping:
  # ping a host with a total count of 15 packets overall.    
  # ping a host with a total count of 15 packets overall, one every .5 seconds (faster ping). 

route:
  # To add a default  route (which will be used if no other route matches).  All packets using this route will be gatewayed through "mango-gw". The device which will actually be used for that route depends on how we can reach "mango-gw" - the static route to "mango-gw" will have to be set up before.

tcpdump:
  # and other packets being transmitted or received over a network. (cf Wikipedia).

. . .

```


# Conclusion


Because everything Cheat displays is plain-text and directed through the shell’s standard output, we can use any text processing commands (like grep) with it. You can read the Using Grep & Regular Expressions to Search for Text Patterns in Linux tutorial for more information on grep.


Additionally, version control system such as Git with GitHub are ideal for storing your custom cheat sheets centrally, so you can get hold of them on multiple platforms via cloning a repository. A sheet is classed as custom if have you added to it, amended it, or created it yourself through Cheat.


All custom cheat sheets are stored in your Linux user’s home directory, inside a hidden folder named .cheat. You can find this location by running cheat -d, which will output two directories: the first is the location of your custom sheets, and the second is the location of the default sheets you get with Cheat upon install.


To access your library of custom sheets on other systems, you need only to copy this .cheat folder onto them. The cheat sheets are small plain-text files so this makes them perfect for tracking with version control. For a complete solution to making your cheat sheets and configuration files accessible at all times, you can read the How To Use Git to Manage your User Configuration Files on a Linux VPS tutorial.


