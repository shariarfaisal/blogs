# How to Customize your Bash Prompt on a Linux VPS

```Linux Basics```

## Introduction



As you manage Linux servers, you’ll spend quite a bit of time using the command line.  For most people, this means spending a lot of time with the Bash shell.


While most distributions provide sensible defaults for the styling of user and root prompts, it can be helpful to customize your prompt to add your own preferences.  You can include a lot of useful information that can help you stay oriented and remind you when you are operating with elevated privileges.


We will be using an Ubuntu 12.04 VPS to experiment, but almost all modern Linux distributions should operate in a similar manner.


# Verify that your Shell is Bash



Before we begin to actually customize the shell, you should verify that your current shell actually is Bash.


This should be true for the vast majority of systems, but sometimes distribution maintainers opt for a different shell or users test out a new shell.


It is easy to verify by checking the /etc/passwd file.  Open the file in a pager like this:


```
less /etc/passwd

```


Each line in this file contains information about a different user.  Find your user and the root user in the first column, as separated by a colon (:).  In the last field, the default login shell for that user will be listed:


```
root:x:0:0:root:/root:/bin/bash
. . .
demouser:x:1000:1000:,,,:/home/demouser/bin/bash

```


If the last field is /bin/bash, then you are all set.


If the last field is not /bin/bash and you wish to change your default shell to Bash, you can edit this file with root privileges and change the last field associated with your user:


```
sudo nano /etc/passwd

```


After you make the change, log out and back in to use the Bash shell.


# Looking at Current Values



To get started, let’s explore what is already in our configuration files to define Bash prompts.


Bash configures its prompt by using the PS1 and the PS2 environmental variables.


PS1 defines the primary prompt that you will see.  You see this every time you log in.  By default, in Ubuntu systems, this should take the form of:


<pre>
<span class=“highlight”>username</span>@<span class=“highlight”>hostname</span>: <span class=“highlight”>current_directory</span>$
</pre>


Note the $ at the end.  This signifies that the shell is a normal user shell.  For the root user, this is replaced with a # to differentiate and make you aware that you are operating with elevated privileges.


The PS2 prompt is used for multi-line commands.  You can see what the current PS2 variable is set to by typing this in your terminal:


```
echo \

```


Press enter directly after the backslash to see the prompt.  By default, in Ubuntu, this is >.


Usually, we define what these variables will hold in our ~/.bashrc file, which is read when our interactive shell starts.


Inside this file on Ubuntu 12.04, we can find a section like this:


```
# uncomment for a colored prompt, if the terminal has the capability; turned
# off by default to not distract the user: the focus in a terminal window
# should be on the output of commands, not on the prompt
# force_color_prompt=yes

if [ -n "$force_color_prompt" ]; then
    if [ -x /usr/bin/tput ] && tput setaf 1 >&/dev/null; then
        # We have color support; assume it's compliant with Ecma-48
        # (ISO/IEC-6429). (Lack of such support is extremely rare, and such
        # a case would tend to support setf rather than setaf.)
        color_prompt=yes
    else
        color_prompt=
    fi
fi

if [ "$color_prompt" = yes ]; then
    PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
else
    PS1='${debian_chroot:+($debian_chroot)}\u@\h:\w\$ '
fi
unset color_prompt force_color_prompt

```


We can see some logic dictating when the prompt will be colorized.  You need to uncomment the force_color_prompt=yes line if you want a colorized prompt.  Do this now to take advantage of the customizations we will be doing later.


```
force_color_prompt=yes

```


The part that we care about is the part that sets up the prompt.  This is nested in an if-else construct with different prompts based on whether you are using color:


```
if [ "$color_prompt" = yes ]; then
    PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
else
    PS1='${debian_chroot:+($debian_chroot)}\u@\h:\w\$ '
fi
unset color_prompt force_color_prompt

```


The top section adds color support.  Let’s take a look at the second section, without color to grasp some of the basics first:


```
PS1='${debian_chroot:+($debian_chroot)}\u@\h:\w\$ '

```


This looks fairly complicated and there are some parts that don’t seem to match anything we see in our normal shell usage.


The parts that include debian_chroot indicate that if you are operating in a change root environment, the prompt will be modified to remind you.  You probably want to keep this part of prompt intact, since this is a helpful feature.


The rest of the prompt definition looks like this:


```
\u@\h:\w\$: 

```


This describes the primary prompt that we’ve been seeing this entire time, using some escape sequences.


## Bash Escape Sequences



We can find a whole list of possible escape sequences in the Bash man page:


<pre>
\a     an ASCII bell character (07)
\d     the date in “Weekday Month Date” format (e.g., “Tue May 26”)
\D{format}
the format is passed to strftime(3) and the result is inserted into the prompt string; an empty format results in a locale-
specific time representation.  The braces are required
\e     an ASCII escape character (033)
\h     the hostname up to the first `.’
\H     the hostname
\j     the number of jobs currently managed by the shell
\l     the basename of the shell’s terminal device name
\n     newline
\r     carriage return
\s     the name of the shell, the basename of $0 (the portion following the final slash)
\t     the current time in 24-hour HH:MM:SS format
\T     the current time in 12-hour HH:MM:SS format
@     the current time in 12-hour am/pm format
\A     the current time in 24-hour HH:MM format
\u     the username of the current user
\v     the version of bash (e.g., 2.00)
\V     the release of bash, version + patch level (e.g., 2.00.0)
\w     the current working directory, with $HOME abbreviated with a tilde (uses the value of the PROMPT_DIRTRIM variable)
\W     the basename of the current working directory, with $HOME abbreviated with a tilde
!     the history number of this command
#     the command number of this command
$     if the effective UID is 0, a #, otherwise a $
\nnn   the character corresponding to the octal number nnn
\     a backslash
[     begin a sequence of non-printing characters, which could be used to embed a terminal control sequence into the prompt
]     end a sequence of non-printing characters
</pre>


As you can see, there is some basic information, and some information that you probably won’t need (ASCII bell character, version of Bash, etc).


Our prompt right now has the username (\u), an @ symbol, the first part of the hostname (\h), the current working directory (\w), and finally, a “$” for regular users and a “#” for the root user.


Let’s exit out of the ~/.bashrc file so that we can test some of the other options.


# Testing New Bash Prompts



Although eventually we’ll want to edit our ~/.bashrc file to make our selections permanent, it is much easier to experiment with changing the prompt from the command line itself.


Before we start to modify things, let’s save our current value for PS1 into a new variable.  This will allow us to switch back to our original prompt easier without having to log out and back in again in case we make a mistake.


```
ORIG=$PS1

```


Now, we have an environmental variable called ORIG that will save a copy of what our default prompt is.


If we need to switch back to the original prompt, we can type:


```
PS1=$ORIG

```


Let’s start simple by just having our username and a $ for the actual prompt:


```
PS1="\u$"

```


We should get something that looks like this:


```
demouser$

```


Let’s space that our a bit to make it look nicer:


```
PS1="\u $: "

```



```
demouser $:

```


We probably don’t want to use the “$” literal character though.  We should use the \$ escape sequence instead.  This will dynamically modify the prompt based on whether we are root or not, allowing our PS1 to be used as root correctly:


```
PS1="\u \$: "

```


We can add any literal strings that we’d like in our prompt:


```
PS1="Hello, my name is \u! \$: "

```



```
Hello, my name is demouser! $:

```


We can also insert the results of arbitrary commands using regular shell functionality.


Here, we can have our prompt give the current load of our server by pulling the first column of the load metrics found at /proc/loadavg using backticks to insert the results of the command:


```
PS1="\u, load: `cat /proc/loadavg | awk '{ print $1; }'` \$: "

```



```
demouser, load: 0.01 $:

```


This is a good way to know if you are taxing your system.


If the date or time is important for you to have in your prompt, you could try something like this.  Let’s also compartmentalize our data a bit with some brackets and parenthesis to keep it organized.  Let’s also add the \w back in to keep track of our working directory:


```
PS1="[\u@\h, load: `cat /proc/loadavg | awk '{ print $1; }'`] (\d - \t) \w \$ "

```



```
[demouser@host, load: 0.01] (Thu Feb 20 - 13:15:20) ~ $

```


This is starting to get a bit unwieldy, especially if we change directories to something with a long path:


```
cd /etc/systemd/system/multi-user.target.wants

```



```
[demouser@host, load: 0.01] (Thu Feb 20 - 13:18:28) /etc/systemd/system/multi-user.target.wants $

```


If we still want all of this information, but want to make it shorter, one strategy is to divide the information between two lines with a \n newline character:


```
PS1="[\u@\h, load: `cat /proc/loadavg | awk '{ print $1; }'`] (\d - \t)\n\w \$ "

```



```
[demouser@host, load: 0.00] (Thu Feb 20 - 13:20:00)
/etc/systemd/system/multi-user.target.wants $

```


Some people dislike a multi-line prompt, but it is one way to provide more information in your prompt.


# Changing Prompt Colors



Now that we have a good grasp on the different ways we can affect our prompts, let’s try out some coloration.


Bash allows you to introduce color into your prompt by using special codes.  These are often a source of confusion, because the selected codes are not very descriptive.


Before we get to the actual color codes, we should talk about how we actually implement them.  There is a right and a wrong way to define color codes in a Bash setting.


First, you must wrap your entire color code description in \[ and \] tags.  These brackets indicate to bash that the characters that exist after the first sequence until the last sequence should be considered non-printing characters.


Bash needs to know this so that it can estimate how many characters to let print before it wraps to the next line.  If you do not enclose your color codes in \[ and \] tags, Bash will count all of the characters as literal characters and will wrap the line too soon.


Secondly, inside the bracket non-print sequence, you need to specify the beginning of a color prompt by typing either \e[ or \033[.  These both do the same thing and indicate the start of an escape sequence.  I will use \e[ in this guide because I think it is a bit more clear.


Before the \], you need an “m” to indicate that you are giving a color sequence.


So basically, each time we want to modify the color, we must enter something in our prompt that looks like:


<pre>
[\e[<span class=“highlight”>color_information</span>m]
</pre>


As you can see, this is the part that makes our prompts especially messy.


As for the color codes, the basic codes to change color of the foreground text are here:


- 30: Black
- 31: Red
- 32: Green
- 33: Yellow
- 34: Blue
- 35: Purple
- 36: Cyan
- 37: White

You can also modify these base values by setting an “attribute” before the base value, separated by a semi-colon (;).


Depending on what kind of terminal you are using, these can behave rather differently.  Some of the more common attributes are:


- 0: Normal text
- 1: Bold or light, depending on terminal
- 4: Underline text

So if you wanted underlined green text, you would use a sequence like this:


```
\[\e[4;32m\]

```


We would then type in the prompt we want.  Afterwards, we probably want to reset the color back to its original value so that the text that we type into the terminal is not colored strangely.


We can do this by using another color code that indicates that Bash should reset the prompt color.  This code is:


```
\[\e[0m\]

```


So together, a simple colored prompt with the username and host could look like this:


```
PS1="\[\e[4;32m\]\u@\h\[\e[0m\]$ "

```


We can also specify background colors for more complexity. Background colors cannot take attributes.  They are:


- 40: Black background
- 41: Red background
- 42: Green background
- 43: Yellow background
- 44: Blue background
- 45: Purple background
- 46: Cyan background
- 47: White background

Although you can specify background colors, attributes, and text color all in one go like this:


```
\[\e[42;1;36m\]

```


It usually works better if you separate the background information from the other information:


```
\[\e[42m\]\[\e[1;36m\]

```


Sometimes, if you are just using the “normal” text attribute (0), the coloration gets garbled in some terminals.  You can avoid this by not specify the normal value with 0, as it is the default anyways.


# Making your Prompt Changes Permanent



Now that we have played around with our prompt awhile, we should have a good idea of how we want our prompt to look.  You should be able to create both a colorized and none-colored prompt.


For our example, we will use the following colored prompt:


```
PS1="[\[\e[0;32m\]\u@\h, load: `cat /proc/loadavg | awk '{ print $1; }'`\[\e[00m\]] (\[\e[00;35m\]\d - \t\[\e[00m\])\n\w \$ "

```


We will use it’s non-colored equivalent also, for when we do not wish to have a colored bash prompt:


```
PS1="[\u@\h, load: `cat /proc/loadavg | awk '{ print $1; }'`] (\d - \t)\n\w \$ "

```


Now that we have both versions of the prompt that we want, we can edit the PS1 in our ~/.bashrc file.


```
nano ~/.bashrc

```


As we discussed in the beginning, the prompts that are in our file now have included functionality to make it apparent when we are in a chroot environment.  Let’s leave that part in.  It looks like this right now:


```
if [ "$color_prompt" = yes ]; then
    PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
else
    PS1='${debian_chroot:+($debian_chroot)}\u@\h:\w\$ '
fi
unset color_prompt force_color_prompt

```


Comment out the current PS1 assignments and copy the debian_chroot logic below, so that it looks like this:


<pre>
if [ “$color_prompt” = yes ]; then
# PS1=‘${debian_chroot:+($debian_chroot)}[\033[01;32m]\u@\h[\033[00m]:[\033[01;34m]\w[\033[00m]$ ’
<span class=“highlight”>PS1=’${debian_chroot:+($debian_chroot)}‘</span>
else
# PS1=’${debian_chroot:+($debian_chroot)}\u@\h:\w$ ’
<span class=“highlight”>PS1=‘${debian_chroot:+($debian_chroot)}’</span>
fi
unset color_prompt force_color_prompt
</pre>


At the end of the prompt, just before the last quotation closes, we can add the prompts that we want to implement.  Actually, since our prompt uses single quotes, we want to change the quotation type in the current prompt to use double quotes.


In the first PS1 assignment, use the colorized version of your prompt.  In the second, use the non-colored one.


<pre>
if [ “$color_prompt” = yes ]; then
# PS1=‘${debian_chroot:+($debian_chroot)}[\033[01;32m]\u@\h[\033[00m]:[\033[01;34m]\w[\033[00m]$ ’
PS1=<span class=“highlight”>"</span>${debian_chroot:+($debian_chroot)}<span class=“highlight”>[[\e[0;32m]\u@\h, load: cat /proc/loadavg | awk '{ print $1; }'[\e[00m]] ([\e[00;35m]\d - \t[\e[00m])\n\w $ "</span>
else
# PS1=’${debian_chroot:+($debian_chroot)}\u@\h:\w$ ’
PS1=<span class=“highlight”>"</span>${debian_chroot:+($debian_chroot)}<span class=“highlight”>[\u@\h, load: cat /proc/loadavg | awk '{ print $1; }'] (\d - \t)\n\w $ "</span>
fi
unset color_prompt force_color_prompt
</pre>


When we are finished, we can close and save the file.


Now, when you log out and back in, your prompt will change to the value you set.


# Conclusion



There are many different ways that you can personalize your configurations.  Colorizing certain items can make them stand out, and can help you locate the last prompt when scrolling through terminal history.


Another popular idea is to provide a special prompt for the root user so that the contrast will remind you of your privileges.  Get creative and try to find the balance that works for you between useful information and clutter.


<div class=“author”>By Justin Ellingwood</div>


