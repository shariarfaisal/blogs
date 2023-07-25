# How To Use Bash History Commands and Expansions on a Linux VPS

```Linux Basics``` ```Linux Commands```

## Introduction


While working in a server environment, you’ll spend a lot of your time on the command line.  Most likely, you’ll be using the bash shell, which is the default of most distributions.


During a terminal session, you’ll likely be repeating some commands often, and typing variations on those commands even more frequently.  While typing each command repeatedly can be good practice in the beginning, at some point, it crosses the line into being disruptive and an annoyance.


Luckily, the bash shell has some fairly well-developed history functions.  Learning how to effectively use and manipulate your bash history will allow you to spend less time typing and more time getting actual work done.  Many developers are familiar with the DRY philosophy of Don’t Repeat Yourself.  Effective use of bash’s history allows you to operate closer to this principle and will speed up your workflow.


# Prerequisites


To follow along with this guide, you will need access to a computer running a Linux-based operating system.  This can either be a virtual private server which you’ve connected to with SSH or your local machine.  Note that this tutorial was validated using a Linux server running Ubuntu 20.04, but the examples given should work on a computer running any version of any Linux distribution.


If you plan to use a remote server to follow this guide, we encourage you to first complete our Initial Server Setup guide.  Doing so will set you up with a secure server environment — including a non-root user with sudo privileges and a firewall configured with UFW — which you can use to build your Linux skills.


# Setting History Defaults


Before you begin actually using your command history, it can be helpful for you to adjust some bash settings to make it more useful. These steps are not necessary, but they can make it easier to find and execute commands that you’ve run previously.


Bash allows you to adjust the number of commands that it stores in history.  It actually has two separate options for this: the HISTFILESIZE parameter configures how many commands are kept in the history file, while the HISTSIZE controls the number stored in memory for the current session.


This means you can set a reasonable cap for the size of history in memory for the current session, and have an even larger history saved to disk that you can examine at a later time.  By default, bash sets very conservative values for these options, but you can expand these to take advantage of a larger history.  Some distributions already increase the default bash history settings with slightly more generous values.


Open your ~/.bashrc file with your preferred text editor to change these settings. Here, we’ll use nano:


```
nano ~/.bashrc


```


Search for both the HISTSIZE and HISTFILESIZE parameters.  If they are set, feel free to modify the values.  If these parameters aren’t in your file, add them now.  For the purposes of this guide, saving 10000 lines to disk and loading the last 5000 lines into memory will work fine.  This is a conservative estimate for most systems, but you can adjust these numbers down if you notice a performance impact:


~/.bashrc
```
. . .

# for setting history length see HISTSIZE and HISTFILESIZE in bash(1)
HISTSIZE=5000
HISTFILESIZE=10000

. . .

```


By default, bash writes its history at the end of each session, overwriting the existing file with an updated version.  This means that if you are logged in with multiple bash sessions, only the last one to exit will have its history saved.


You can work around this by setting the histappend setting, which will append instead of overwriting the history.  This may be set already, but if it is not, you can enable this by adding this line:


~/.bashrc
```
. . .
shopt -s histappend
. . .

```


If you want to have bash immediately add commands to your history instead of waiting for the end of each session (to enable commands in one terminal to be instantly be available in another), you can also set or append the history -a command to the PROMPT_COMMAND parameter, which contains commands that are executed before each new command prompt.


To configure this correctly, you’ll need to customize bash’s PROMPT_COMMAND to change the way commands are recorded in the history file and in the current shell session’s memory:


- First, you need to append to the history file immediately with history -a.
- Next, you must clear the current history in your shell session with history -c.
- Finally,  to load the updated history back into your shell session, use the history -r command.

Putting all these commands together in order in the PROMPT_COMMAND shell variable will result in the the following, which you can paste into your .bashrc file:


~/.bashrc
```
. . .
export PROMPT_COMMAND="history -a; history -c; history -r; $PROMPT_COMMAND"
. . .

```


When you are finished, save the file and exit.  If you edited your .bashrc file with nano, do so by pressing CTRL + X, Y, and then ENTER.


To implement your changes, either log out and back in again, or source the file by running:


```
source ~/.bashrc


```


With that, you’ve adjusted how your shell handles your command history.  You can now get some practice finding your previous commands with the history command.


# Reviewing your Previous Bash History


The way to review your bash history is to use the history command.  This will print out our recent commands, one command per line.  This should output, at most, the number of lines you selected for the HISTSIZE variable.  It will probably be fewer at this point:


```
history


```


```
Output  . . .

   43  man bash
   44  man fc
   45  man bash
   46  fc -l -10
   47  history
   48  ls -a
   49  vim .bash_history
   50  history
   51  man history
   52  history 10
   53  history

```


Each command history returns is associated with a number for easy reference.  This guide will go over how this can be useful later on.


You can truncate the output by specifying a number after the command.  For instance, if you want to only return the last 5 commands that were executed, you can type:


```
history 5


```


```
Output   50  history
   51  man history
   52  history 10
   53  history
   54  history 5

```


To find all of the history commands that contain a specific string, you can pipe the results into a grep command that searches each line for a given string.  For example, you can search for the lines that have cd by typing:


```
history | grep cd


```


```
Output   33  cd Pictures/
   37  cd ..
   39  cd Desktop/
   61  cd /usr/bin/
   68  cd
   83  cd /etc/
   86  cd resolvconf/
   90  cd resolv.conf.d/

```


There are many situations where being able to retrieve a list of commands you’ve previously ran can be helpful.  If you want to run one of those commands again, your instinct may be to copy one of the commands from your output and paste it into your prompt.  This works, but bash comes with a number of shortcuts that allow you to retrieve and then automatically execute commands from your history.


# Executing Commands from your Bash History


Printing your command history can be useful, but it doesn’t really help you access those commands other than as reference.


You can recall and immediately execute any of the commands returned by a history operation by its number preceded by an exclamation point (!).  Assuming your history results aligned with those from the previous section, you could check out the man page for the history command quickly by typing:


```
!51


```


This will immediately recall and execute the command associated with the history number 51.


You can also execute commands relative to your current position by using the !-n syntax, where “n” is replaced by the number of previous commands you want to recall.


As an example, say you ran the following two commands:


```
ls /usr/share/common-licenses
echo hello


```


If you wanted to recall and execute the command you ran before your most recent one (the ls command), you could type !-2:


```
!-2


```


To re-execute the last command you ran, you could run !-1.  However, bash provides a shortcut consisting of two exclamation points which will substitute the most recent command and execute it:


```
!!


```


Many people use this when they type a command but forgot that they needed sudo privileges for it to execute.  Typing sudo !! will re-execute the command with sudo in front of it:


```
touch /etc/hello


```


```
Outputtouch: cannot touch `/etc/hello': Permission denied

```


```
sudo !!


```


```
Outputsudo touch /etc/hello
[sudo] password for sammy:

```


This demonstrates another property of this syntax: these shortcuts are pure substitutions, and can be incorporated within other commands at will.


# Scrolling through Bash History


There are a few ways that you can scroll through your bash history, putting each successive command on the command line to edit.


The most common way of doing this is to press the up arrow key at the command prompt.  Each additional press of the up arrow key will take you further back in your command line history.


If you need to go the other direction, the down arrow key traverses the history in the opposite direction, finally bringing you back to your current empty prompt.


If moving your hand all the way over to the arrow keys seems like a big hassle, you can move backwards in your command history using the CTRL + P combination and use the CTRL + N combination to move forward through your history again.


If you want to jump back to the current command prompt, you can do so by pressing META + >.  In most cases, the “meta” key is the ALT key, so META + > will mean pressing ALT + SHIFT + ..  This is useful if you find yourself far back in your history and want to get back to your empty prompt.


You can also go to the first line of your command history by doing the opposite maneuver and typing META + <.  This typically means pressing ALT + SHIFT + ,.


To summarize, these are some keys and key combinations you can use to scroll through the history and jump to either end:


- UP arrow key: Scroll backwards in history
- CTRL + P: Scroll backwards in history
- DOWN arrow key: Scroll forwards in history
- CTRL + N: Scroll forwards in history
- ALT + SHIFT + .: Jump to the end of the history (most recent)
- ALT+ SHIFT + ,: Jump to the beginning of the history (most distant)

# Searching through Bash History


Although piping the history command through grep can be a useful way to narrow down the results, it isn’t ideal in many situations.


Bash includes search functionality for its history.  The typical way of using this is through searching backwards in history (most recent results returned first) using the CTRL + R key combination.


For instance, you can type CTRL + R, and begin typing part of the previous command.  You only have to type out part of the command.  If it matches an unwanted command instead, you can press CTRL + R again for the next result.


If you accidentally pass the command you wanted, you can move in the opposite direction by typing CTRL + S.  This also can be useful if you’ve moved to a different point in your history using the keys in the last section and wish to search forward.


Be aware that, in many terminals, the CTRL + S combination is mapped to suspend the terminal session.  This will intercept any attempts to pass CTRL + S to bash, and will “freeze” your terminal.  To unfreeze, type CTRL + Q to unsuspend the session.


This suspend and resume feature is not needed in most modern terminals, and you can turn it off without any problem by running the following command:


```
stty -ixon


```


stty is a utility that allows you to change your terminal’s settings from the command line.  You could add this stty -ixon command to the end of your ~/.bashrc file to make this change permanent as well.


If you try searching with CTRL + S again now, it should work as expected to allow you to search forwards.


## Searching after You’ve Typed Part of the Command


A common scenario to find yourself in is to type in part of a command, only to then realize that you have executed it previously and can search the history for it.


The correct way of searching using what is already on your command line is to move your cursor to the beginning of the line with CTRL + A, call the reverse history with CTRL + R, paste the current line into the search with CTRL + Y, and then using the CTRL + R again to search in reverse.


For instance, suppose you want to update your package cache on an Ubuntu system.  You’ve already typed this out recently, but you didn’t think about that until after you’ve typed the sudo in the prompt again:


```
sudo


```


At this point, you realize that this is an operation you’ve definitely done in the past day or so.  You can press CTRL + A to move your cursor to the beginning of the line.  Then, press CTRL + R to call your reverse incremental history search.  This has a side effect of copying all of the content on the command line that was after our cursor position and putting it into your clipboard.


Next, press CTRL + Y to paste the command segments that you just copied from the command line into the search.  Lastly, press CTRL + R to move backwards in your history, searching for commands containing the content you’ve just pasted.


Using shortcuts like this may seem tedious at first, but it can be quite useful when you get used to it.  It is extremely helpful when you find yourself in a position where you’ve typed out half of a complex command and know you’re going to need the history to finish the rest.


Rather than thinking of these as separate key combinations, it may help you to think of them as a single compound operation.  You can just hold the CTRL key down and then press A, R, Y, and then the R key down in succession.


# Getting Familiar with More Advanced History Expansion


This guide has already touched on some of the most fundamental history expansion techniques that bash provides.  Some of the ones we’ve covered so far are:


- !!: Expand to the last command
- !n: Expand the command with history number “n”.
- !-n: Expand to the command that was “n” number of commands before the current command in history.

## Event Designators


The above three examples are instances of event designators.  These generally are ways of recalling previous history commands using certain criteria.  They are the selection portion of your available operations.


For example, you can execute the last ssh command that you ran by typing something like:


```
!ssh


```


This searches for lines in your command history that begin with ssh.  If you want to search for a string that isn’t at the beginning of the command, you can surround it with ? characters.  For instance, to repeat a previous apt-cache search command, you could likely run the following command to find and execute it:


```
!?search?


```


Another event designator you can try involves substituting a string within your last command for another.  To do this, enter a caret symbol (^) followed by the string you want to replace, then immediately follow that with another caret, the replacement string, and a final caret at the end.  Don’t include any spaces unless they’re part of the string you want to replace or part of the string you want to use as the replacement:


```
^original^replacement^


```


This will recall the previous command (just like !!), search for an instance of original within the command string, and replace it with replacement.  It will then execute the command using the replacement string.


This is useful for dealing with things like misspellings.  For instance, say you mistakenly run this command when trying to read the contents of the /etc/hosts file:


```
cat /etc/hosst


```


```
Outputcat: /etc/hosst: No such file or directory

```


Rather then rewriting the entire command, you could run the following instead:


```
^hosst^hosts^


```


This will fix the error in the previous command and execute it successfully.


## Word Designators


After event designators, you can add a colon (:) followed by a word designator to select a portion of the matched command.


It does this by dividing the command into “words”, which are defined as any chunk separated by whitespace.  This allows you some interesting opportunities to interact with your command parameters.


The word numbering starts at the initial command as “0”, the first argument as “1”, and continues on from there.


For instance, you could list the contents of a directory and then decide you want to navigate into that same directory.  You could do so by running the following operations back to back:


```
ls /usr/share/common-licenses
cd !!:1


```


In cases like this where you are operating on the last command, you can shorten this by removing the second ! as well as the colon:


```
cd !1


```


This will operate in the same way.


You can refer to the first argument with a caret (^) and the final argument with a dollar sign ($) if that makes sense for your purposes.  These are more helpful when using ranges instead of specific numbers.  For instance, you have three ways you can get all of the arguments from a previous command into a new command:


```
!!:1*
!!:1-$
!!:*


```


The lone * expands to all portions of the command being recalled other than the initial command.  Similarly, you can use a number followed by * to mean that everything after the specified word should be included.


## Modifiers


Another thing you can do to augment the behavior of the history line you’re recalling is to modify the behavior of the recall to manipulate the text itself.  To do this, you can add modifiers after a colon (:) character at the end of an expansion.


For instance, you can chop off the path leading up to a file by using the h modifier (it stands for “head”), which removes the path up until the final slash (/) character.  Be aware that this won’t work the way you want it to if you are using this to truncate a directory path and the path ends with a trailing slash.


A common use case for this is if you are modifying a file and realize you’d like to change to the file’s directory to do operations on related files.


For instance, say you run this command to print the contents of an open-source software license to your output:


```
cat /usr/share/common-licenses/Apache-2.0


```


After being satisfied that the license suits your needs, you may want to change into the directory where it’s held.  You can do this by calling the cd command on the argument chain and chopping off the filename at the end:


```
cd !!:$:h


```


If you run pwd, which prints your current working directory, you’ll find that you’ve navigated to the directory included in the previous command:


```
pwd


```


```
Output/usr/share/common-licenses

```


Once you’re there, you may want to open that license file again to double check, this time in a pager like less.


To do this, you could perform the reverse of the previous manipulation by chopping off the path and using only the filename with the t modifier, which stands for “tail”.  You can search for your last cat operation and use the t flag to pass only the file name:


```
less !cat:$:t


```


You could just as easily keep the full absolute path name and this command would work correctly in this instance.  However, there may be other times when this isn’t true.  For instance, you could be looking at a file nested within a few subdirectories below your current working directory using a relative path and then change to the subdirectory using the “h” modifier.  In this case, you wouldn’t be able to rely on the relative path name to reach the file any more.


Another extremely helpful modifier is the r modifier which strips the trailing extension.  This could be useful if you are using tar to extract a file and want to change into the resulting directory afterwards.  Assuming the directory produced is the same name as the file, you could do something like:


```
tar xzvf long-project-name.tgz
cd !!:$:r


```


If your tarball uses the tar.gz extension instead of tgz, you can just pass the modifier twice:


```
tar xzvf long-project-name.tgz
cd !!:$:r:r


```


A similar modifier, e, removes everything besides the trailing extension.


If you do not want to execute the command that you are recalling and only want to find it, you can use the p modifier to have bash echo the command instead of executing it.


This is useful if you are unsure of if you’re selecting the correct piece.  This not only prints it, but also puts it into your history for further editing if you’d like to modify it.


For instance, imagine you ran a find command on your home directory and then realized that you wanted to run it from the root (/) directory.  You could check that you’re making the correct substitutions like this (assuming the original command is associated with the number 119 in your history):


```
find ~ -name "file1"
!119:0:p / !119:2*:p


```


```
Outputfind / -name "file1"

```


If the returned command is correct, you can execute it with the CTRL + P  key combination.


You can also make substitutions in your command by using the s/original/new/ syntax.


For instance, you could have accomplished that by typing:


```
!119:s/~/\//


```


This will substitute the first instance of the search pattern (~).


You can substitute every match by also passing the g flag with the s.  For instance, if you want to create files named file1, file2, and file3, and then want to create directories called dir1, dir2, dir3, you could do this:


```
touch file1 file2 file3
mkdir !!:*:gs/file/dir/


```


Of course, it may be more intuitive to just run mkdir dir1 dir2 dir3 in this case.  However, as you become comfortable using modifiers and the other bash history expansion tools, you can greatly expand your capabilities and productivity on the command line.


# Conclusion


By reading this guide, you should now have a good idea of how you can leverage the history operations available to you.  Some of these will probably be more useful than others, but it is good to know that bash has these capabilities in case you find yourself in a position where it would be helpful to dig them up.


If nothing else, the history command alone, the reverse search, and the basic history expansions can do a lot to help you speed up your workflow.


