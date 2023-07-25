# An Introduction to Useful Bash Aliases and Functions

```Linux Basics```

## Introduction


The more you operate on the command line, the more you will find that the majority of the commands you use are a very small subset of the available commands.  Most tasks are habitual and you may run these the same way every day.


While the makers of many of the most common command utilities have attempted to eliminate extraneous typing by using shortened names (think of how many keystrokes you save daily by typing “ls” instead of “list” and “cd” instead of “change-directory”), these are not ubiquitous.  Additionally, many people always run commands with the same few options enabled every time.


Luckily, bash allows us to create our own shortcuts and time-savers through  the use of aliases and shell functions.  In this guide, we’ll discuss how to make use of these and give you some useful examples to get you started in the right direction.


# How To Declare a Bash Alias


Declaring aliases in bash is very straight forward.  It’s so easy that you should try it now.


You can declare aliases that will last as long as your shell session by simply typing these into the command line.  The syntax looks like this:


```
alias alias_name="command_to_run"


```


Note that there is no spacing between between the neighbor elements and the equal sign.  This is not optional.  Spaces here will break the command.


Let’s create a common bash alias now.  One idiomatic command phrase that many people use frequently is ls -lha or ls -lhA (the second omits the current and parent directory listing).  We can create a shortcut that can be called as ll by typing:


```
alias ll="ls -lhA"

```


Now, we can type ll and we’ll get the current directory’s listing, in long format, including hidden directories:


```
ll

```



```
-rw-r--r-- 1 root root 3.0K Mar 20 18:03 .bash_history
-rw-r--r-- 1 root root 3.1K Apr 19  2012 .bashrc
drwx------ 2 root root 4.0K Oct 24 14:45 .cache
drwx------ 2 root root 4.0K Mar 20 18:00 .gnupg
-rw-r--r-- 1 root root    0 Oct 24 17:03 .mysql_history
-rw-r--r-- 1 root root  140 Apr 19  2012 .profile
drwx------ 2 root root 4.0K Oct 24 14:21 .ssh
-rw------- 1 root root 3.5K Mar 20 17:24 .viminfo

```


If you want to get rid of an alias, just use the unalias command:


```
unalias ll

```


The alias is now removed.


You can list all of your configured aliases by passing the alias command without any arguments:


```
alias

```


To temporarily bypass an alias (say we aliased ls to ls -a), we can type:


```
\ls

```


This will call the normal command found in our path, without using the aliased version.


Assuming you did not unset it, the ll alias will be available throughout the current shell session, but when you open a new terminal window, this will not be available.


To make this persistent, we need to add this into one of the various files that is read when a shell session begins.  Popular choices are ~/.bashrc and ~/.bash_profile.  We just need to open the file and add the alias there:


```
nano ~/.bashrc

```


At the bottom or wherever you’d like, add the alias you added on the command line.  Feel free to add a  comment declaring an entire section devoted to bash aliases:


```
#########
# Aliases
#########

alias ll="ls -lhA"

```


This alias or a variation might actually already be in your file.  Many distributions ship with a set of standard bash configuration files with a few useful aliases.


Save and close the file.  Any aliases you added will be available next time you start a new shell session.  To read any changes you made in your file into your current session, just tell bash to re-read the file now:


```
source ~/.bashrc

```


# Alias Examples


Now that you know how to create your own aliases, let’s talk about some popular ones that may be useful to you.  These can be found throughout the web, and some may also be included in your distribution’s default bash configuration as well.


## Navigation and Listing


Many of the most simple Linux commands are more helpful when you apply some formatting and options.


We discussed one ls example above, but there are many others you may find.


Make ls display in columns and with a file type indicator (end directories with “/”, etc) by default:


```
alias ls="ls -CF"

```


We can also anticipate some typos to make it call the correct command:


```
alias sl="ls"

```


Let’s also make an alias to pipe our output to less for viewing large directory listings with the long format:


```
alias lsl="ls -lhFA | less"

```


How about we stray from ls and try some helpful commands for cd.


This one will change to your parent directory, even when you forget the space:


```
alias cd..="cd .."

```


You can also cut out the cd part entirely by making an alias for ..:


```
alias ..="cd .."

```


We can find files in our current directory easily by setting this alias:


```
alias fhere="find . -name "

```


## System Aliases


How about some of our monitoring and system stats commands?  I call these with the same options every time, so I might as well make some aliases.


This one will list our disk usage in human-readable units including filesystem type, and print a total at the bottom:


```
alias df="df -Tha --total"

```


We might as well add an alias for our preferred du output as well:


```
alias du="du -ach | sort -h"

```


Let’s keep going in the same direction by making our free output more human friendly:


```
alias free="free -mt"

```


We can do a lot with our listing process table.  Let’s start out by setting a default output:


```
alias ps="ps auxf"

```


How about we make our process table searchable.  We can create an alias that searches our process for an argument we’ll pass:


```
alias psg="ps aux | grep -v grep | grep -i -e VSZ -e"

```


Now, when we call it with the process name we’re looking for as an argument, we’ll get a nice, compact output:


```
psg bash

```



```
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
1001      5227  0.0  0.0  26320  3376 pts/0    Ss   16:29   0:00 bash

```


## Miscellaneous Aliases


One common option to the mkdir command that we use often is the -p flag to make any necessary parent directories.  We can make this the default:


```
alias mkdir="mkdir -p"

```


We might want to add a -v flag on top of that so we are told of every directory creation, which can help us recognize quickly if we had a typo which caused an accidental directory branch:


```
alias mkdir="mkdir -pv"

```


When downloading files from the internet with wget, in almost all circumstances, you’ll want to pass the -c flag in order to continue the download in case of problems.  We can set that with this:


```
alias wget="wget -c"

```


We can search our history easily like with a grep of the history command’s output.  This is sometimes more useful than using CTRL-R to reverse search because it gives you the command number to do more complex recalls afterwards:


```
alias histg="history | grep"

```


I have a few system tools that I prefer to upgrade from the standard version to more complex tools.  These will only work if you’ve downloaded the required utilities, but they can be very helpful.  Keep in mind that these may affect your other aliases.


This one replaces the conventional top command with an enhanced version that is much easier on the eyes and can be sorted, searched, and scrolled without complications:


```
alias top="htop"

```


In a similar way, the ncdu command can be downloaded which presents file and directory sizes in an interactive ncurses display that you can browse and use to perform simple file actions:


```
alias du="ncdu"

```


There’s an upgraded utility for df as well that’s called pydf.  It provides colorized output and text-based usage bars.  We can default to using this utility if we have it:


```
alias df="pydf"

```


Have you ever needed your public IP address from the command line when you’re behind a router using NAT?  Something like this could be useful:


```
alias myip="curl http://ipecho.net/plain; echo"

```


For my own purposes, I like to optimize the images I upload for articles to be 690px or less, so I use the ImageMagick package (sudo apt-get install imagemagick if not already available) which contains a command called mogrify that does just this.  I have this command in my ~/.bashrc file:


```
alias webify="mogrify -resize 690\> *.png"

```


This will resize all of the PNG images in the current directory, only if they are wider than 690px.


If I then have to upload them to a server, I can use sftp to connect and automatically change to a specific directory:


```
alias upload="sftp username@server.com</^>:/path/to/upload/directory<^>


```


# Getting Started with Bash Functions


Although aliases are quick and easy to implement, they are quite limited in their scope.  You’ll find as you’re trying to chain commands together that you can’t access arguments given at runtime very well, among other things.  Aliases can also be quite slow at times because they are read after all functions.


There is an alternative to aliases that is more robust and can help you bridge the gap between bash aliases and full shell scripts.  These are called shell functions.  They work in almost the same way as aliases but are more programmatic and accept input in a standard way.


We won’t go into extensive detail here, because these can be used in so many complex situations and bash is an entire scripting language, but we’ll go over some basic examples.


For starters, there are two basic ways to declare a bash syntax.  The first uses the function command and looks something like this:


```
function function_name {
    command1
    <^>command2</^>
}


```


The other syntax uses a set of parentheses which is more “C-like”:


```
function_name () {
    command1
    command2
}


```


We can compress this second form into one line and separate the commands with semicolons.  A semicolon must come after the last command too:


```
function_name () { command1; command2; }


```


Let’s start off by demonstrating an extremely useful bash function.  This one will create a directory and then immediately move into that directory.  This is usually exactly the sequence we take when making new directories:


```
mcd () {
    mkdir -p $1
    cd $1
}

```


Now, when we use use this function instead of the regular mkdir command to auto change into the directory after creation:


```
mcd test
pwd

```



```
/home/demouser/test

```


One cool function that you’ll see around is the extract function.  This combines a lot of utilities to allow you to decompress just about any compressed file format.  There are a number of variations, but this one comes from here:


```
function extract {
 if [ -z "$1" ]; then
    # display usage if no parameters given
    echo "Usage: extract <path/file_name>.<zip|rar|bz2|gz|tar|tbz2|tgz|Z|7z|xz|ex|tar.bz2|tar.gz|tar.xz>"
    echo "       extract <path/file_name_1.ext> [path/file_name_2.ext] [path/file_name_3.ext]"
    return 1
 else
    for n in $@
    do
      if [ -f "$n" ] ; then
          case "${n%,}" in
            *.tar.bz2|*.tar.gz|*.tar.xz|*.tbz2|*.tgz|*.txz|*.tar) 
                         tar xvf "$n"       ;;
            *.lzma)      unlzma ./"$n"      ;;
            *.bz2)       bunzip2 ./"$n"     ;;
            *.rar)       unrar x -ad ./"$n" ;;
            *.gz)        gunzip ./"$n"      ;;
            *.zip)       unzip ./"$n"       ;;
            *.z)         uncompress ./"$n"  ;;
            *.7z|*.arj|*.cab|*.chm|*.deb|*.dmg|*.iso|*.lzh|*.msi|*.rpm|*.udf|*.wim|*.xar)
                         7z x ./"$n"        ;;
            *.xz)        unxz ./"$n"        ;;
            *.exe)       cabextract ./"$n"  ;;
            *)
                         echo "extract: '$n' - unknown archive method"
                         return 1
                         ;;
          esac
      else
          echo "'$n' - file does not exist"
          return 1
      fi
    done
fi
}

```


This function takes the first argument and calls the appropriate utility program based on the file extension used.


# Conclusion


Hopefully this guide has given you some inspiration for creating your own aliases and bash functions.  Extensive use of these can help make your time in the shell more enjoyable and less complex.


Remember to be wary of redefining existing commands with behavior that is potentially destructive.  Even doing the opposite and aliasing a command to a safer variant (always asking for confirmation before deleting recursively, for instance) can come back to bite you the first time you’re on a system without it once you’ve come to rely on it.


To find candidates that might be good to create aliases for, it might be a good idea to search your history for your most commonly used commands.  A one-liner from here allows us to see our most used commands:


```
history | awk '{CMD[$2]++;count++;}END { for (a in CMD)print CMD[a] " " CMD[a]/count*100 "% " a;}' | grep -v "./" | column -c3 -s " " -t | sort -nr | nl |  head -n10

```



```
 1	247  24.7%  cd
 2	112  11.2%  vim
 3	90   9%     exit
 4	72   7.2%   ls
 5	70   7%     xset
 6	56   5.6%   apt-get
 7	40   4%     vlc
 8	40   4%     rm
 9	38   3.8%   screen
10	27   2.7%   htop

```


We can easily use this list as a starting point for commands that we frequently utilize.  In the comments section, feel free to share your favorite bash aliases and functions:


<div class=“author”>By Justin Ellingwood</div>


