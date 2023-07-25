# How To Use Bash s Job Control to Manage Foreground and Background Processes

```Linux Basics``` ```Linux Commands```

## Introduction


In a previous tutorial, we discussed how the ps, kill, and nice commands can be used to control processes on your system.  This guide highlights how bash, the Linux system, and your terminal come together to offer process and job control.


This article will focus on managing foreground and background processes and will demonstrate how to leverage your shell’s job control functions to gain more flexibility in how you run commands.


# Prerequisites


To follow along with this guide, you will need access to a computer running the bash shell interface. bash is the default shell on many Linux-based operating systems, and it is available on many Unix-like operating systems, including macOS. Note that this tutorial was validated using a Linux virtual private server running Ubuntu 20.04.


If you plan to use a remote server to follow this guide, we encourage you to first complete our Initial Server Setup guide. Doing so will set you up with a secure server environment — including a non-root user with sudo privileges and a firewall configured with UFW — which you can use to build your Linux skills.


# Managing Foreground Processes


Most processes that you start on a Linux machine will run in the foreground.  The command will begin execution, blocking use of the shell for the duration of the process.  The process may allow user interaction or may just run through a procedure and then exit.  Any output will be displayed in the terminal window by default.  We’ll discuss the basic way to manage foreground processes in the following subsections.


## Starting a Process


By default, processes are started in the foreground.  This means that until the program exits or changes state, you will not be able to interact with the shell.


Some foreground commands exit very quickly and return you to a shell prompt almost immediately.  For instance, the following command will print Hello World to the terminal and then return you to your command prompt:


```
echo "Hello World"


```


```
OutputHello World

```


Other foreground commands take longer to execute, blocking shell access for their duration.  This might be because the command is performing a more extensive operation or because it is configured to run until it is explicitly stopped or until it receives other user input.


A command that runs indefinitely is the top utility.  After starting, it will continue to run and update its display until the user terminates the process:


```
top


```


You can quit top by pressing q, but some other processes don’t have a dedicated quit function.  To stop those, you’ll have to use another method.


## Terminating a Process


Suppose you start a simple bash loop on the command line.  As an example, the following command will start a loop that prints Hello World every ten seconds.  This loop will continue forever, until explicitly terminated:


```
while true; do echo "Hello World"; sleep 10; done


```


Unlike top, loops like this have no “quit” key.  You will have to stop the process by sending it a signal.  In Linux, the kernel can send signals to running processes as a request that they exit or change states.  Linux terminals are usually configured to send the “SIGINT” signal (short for “signal interrupt”) to current foreground process when the user presses the CTRL + C key combination.  The SIGINT signal tells the program that the user has requested termination using the keyboard.


To stop the loop you’ve started, hold the CTRL key and press the C key:


```
CTRL + C

```


The loop will exit, returning control to the shell.


The SIGINT signal sent by the CTRL + C combination is one of many signals that can be sent to programs.  Most signals do not have keyboard combinations associated with them and must instead be sent using the kill command, which will be covered later on in this guide.


## Suspending Processes


As mentioned previously, foreground process will block access to the shell for the duration of their execution.  What if you start a process in the foreground, but then realize that you need access to the terminal?


Another signal that you can send is the “SIGTSTP” signal.  SIGTSTP is short for “signal terminal stop”, and is usually represented as signal number 20.  When you press CTRL + Z, your terminal registers a “suspend” command, which then sends the SIGTSTP signal to the foreground process.  Essentially, this will pause the execution of the command and return control to the terminal.


To illustrate, use ping to connect to google.com every 5 seconds.  The following command precedes the ping command with command, which will allow you to bypass any shell aliases that artificially set a maximum count on the command:


```
command ping -i 5 google.com


```


Instead of terminating the command with CTRL + C, press CTRL + Z instead. Doing so will return output like this:


```
Output[1]+  Stopped                 ping -i 5 google.com

```


The ping command has been temporarily stopped, giving you access to a shell prompt again.  You can use the ps process tool to show this:


```
ps T


```


```
Output  PID TTY      STAT   TIME COMMAND
26904 pts/3    Ss     0:00 /bin/bash
29633 pts/3    T      0:00 ping -i 5 google.com
29643 pts/3    R+     0:00 ps t

```


This output indicates that the ping process is still listed, but that the “STAT” column has a “T” in it.  Per the ps man page, this means that a job that has been “stopped by [a] job control signal”.


This guide will outline how to change process states in greater depth, but for now you can resume execution of the command in the foreground again by typing:


```
fg


```


Once the process has resumed, terminate it with CTRL + C:


# Managing Background Processes


The main alternative to running a process in the foreground is to allow it to execute in the background.  A background process is associated with the specific terminal that started it, but does not block access to the shell.  Instead, it executes in the background, leaving the user able to interact with the system while the command runs.


Because of the way that a foreground process interacts with its terminal, there can be only a single foreground process for every terminal window.  Because background processes return control to the shell immediately without waiting for the process to complete, many background processes can run at the same time.


## Starting Processes


You can start a background process by appending an ampersand character (&) to the end of your commands.  This tells the shell not to wait for the process to complete, but instead to begin execution and to immediately return the user to a prompt.  The output of the command will still display in the terminal (unless redirected), but you can type additional commands as the background process continues.


For instance, you can start the same ping process from the previous section in the background by typing:


```
command ping -i 5 google.com &


```


The bash job control system will return output like this:


```
Output[1] 4287

```


You’ll then receive the normal output from the ping command:


```
OutputPING google.com (74.125.226.71) 56(84) bytes of data.
64 bytes from lga15s44-in-f7.1e100.net (74.125.226.71): icmp_seq=1 ttl=55 time=12.3 ms
64 bytes from lga15s44-in-f7.1e100.net (74.125.226.71): icmp_seq=2 ttl=55 time=11.1 ms
64 bytes from lga15s44-in-f7.1e100.net (74.125.226.71): icmp_seq=3 ttl=55 time=9.98 ms

```


However, you can also type commands at the same time.  The background process’s output will be mixed among the input and output of your foreground processes, but it will not interfere with the execution of the foreground processes.


## Listing Background Processes


To list all stopped or backgrounded processes, you can use the jobs command:


```
jobs


```


If you still have the previous ping command running in the background, the jobs command’s output will be similar to this:


```
Output[1]+  Running                 command ping -i 5 google.com &

```


This indicates that you currently have a single background process running.  The [1] represents the command’s job spec or job number.  You can reference this with other job and process control commands, like kill, fg, and bg by preceding the job number with a percentage sign.  In this case, you’d reference this job as %1.


## Stopping Background Processes


You can stop the current background process in a few ways.  The most straightforward way is to use the kill command with the associated job number.  For instance, you can kill your running background process by typing:


```
kill %1


```


Depending on how your terminal is configured, either immediately or the next time you hit ENTER, the job termination status will appear in your output:


```
Output[1]+  Terminated              command ping -i 5 google.com

```


If you check the jobs command again, there won’t be any current jobs.


# Changing Process States


Now that you know how to start and stop processes in the background, you can learn about changing their state.


This guide already outlined one way to change a process’s state: stopping or suspending a process with CTRL + Z.  When processes are in this stopped state, you can move a foreground process to the background or vice versa.


## Moving Foreground Processes to the Background


If you forget to end a command with & when you start it, you can still move the process to the background.


The first step is to stop the process with CTRL + Z again. Once the process is stopped, you can use the bg command to start it again in the background:


```
bg


```


You will receive the job status line again, this time with the ampersand appended:


```
Output[1]+ ping -i 5 google.com &

```


By default, the bg command operates on the most recently-stopped process.  If you’ve stopped multiple processes in a row without starting them again, you can reference a specific process by its job number to move the correct process to the background.


Note that not all commands can be backgrounded.  Some processes will automatically terminate if they detect that they have been started with their standard input and output directly connected to an active terminal.


## Moving Background Processes to the Foreground


You can also move background processes to the foreground by typing fg:


```
fg


```


This operates on your most recently backgrounded process (indicated by the + in the jobs command’s output).  It immediately suspends the process and puts it into the foreground.  To specify a different job, use its job number:


```
fg %2


```


Once a job is in the foreground, you can kill it with CTRL + C, let it complete, or suspend and move it to the background again.


# Dealing with SIGHUPs


Whether a process is in the background or in the foreground, it is rather tightly tied with the terminal instance that started it.  When a terminal closes, it typically sends a SIGHUP signal to all of the processes (foreground, background, or stopped) that are tied to the terminal.  This signals for the processes to terminate because their controlling terminal will shortly be unavailable.


There may be times, though, when you want to close a terminal but keep the background processes running. There are a number of ways of accomplishing this.  One of the more flexible ways is to use a terminal multiplexer like screen or tmux.  Another solution is to use a utility that provides the detach functionality of screen and tmux, like dtach.


However, this isn’t always an option.  Sometimes these programs aren’t available or you’ve already started the process you need to continue running.  Sometimes these could even be overkill for what you need to accomplish.


## Using nohup


If you know when starting the process that you will want to close the terminal before the process completes, you can start it using the nohup command.  This makes the started process immune to the SIGHUP signal.  It will continue running when the terminal closes and will be reassigned as a child of the init system:


```
nohup ping -i 5 google.com &


```


This will return a line like the following, indicating that the output of the command will be written to a file called nohup.out:


```
Outputnohup: ignoring input and appending output to ‘nohup.out’

```


This file will be placed in your current working directory if writeable, but otherwise it will be placed in your home directory. This is to ensure that output is not lost if the terminal window is closed.


If you close the terminal window and open another one, the process will still be running.  You will not find it in the output of the jobs command because each terminal instance maintains its own independent job queue.  Closing the terminal will cause the ping job to be destroyed even though the ping process is still running.


To kill the ping process, you’ll have to find its process ID (or “PID”).  You can do that with the pgrep command (there is also a pkill command, but this two-part method ensures that you are only killing the intended process).  Use pgrep and the -a flag to search for the executable:


```
pgrep -a ping


```


```
Output7360 ping -i 5 google.com

```


You can then kill the process by referencing the returned PID, which is the number in the first column:


```
kill 7360


```


You may wish to remove the nohup.out file if you don’t need it anymore.


## Using disown


The nohup command is helpful, but only if you know you will need it at the time you start the process.  The bash job control system provides other methods of achieving similar results with the built-in disown command.


The disown command, in its default configuration, removes a job from the jobs queue of a terminal.  This means that it can no longer be managed using the job control mechanisms discussed previously in this guide, like fg, bg, CTRL + Z, CTRL + C.  Instead, the job will immediately be removed from the list in the jobs output and no longer associated with the terminal.


The command is called by specifying a job number.  For instance, to immediately disown job 2, you could type:


```
disown %2


```


This leaves the process in a state not unlike that of a nohup process after the controlling terminal has been closed.  The exception is that any output will be lost when the controlling terminal closes if it is not being redirected to a file.


Usually, you don’t want to remove the process completely from job control if you aren’t immediately closing your terminal window.  You can pass the -h flag to the disown process instead in order to mark the process to ignore SIGHUP signals, but to otherwise continue on as a regular job:


```
disown -h %1


```


In this state, you could use normal job control mechanisms to continue controlling the process until closing the terminal.  Upon closing the terminal, you will, once again, be stuck with a process with nowhere to output if you didn’t redirect to a file when starting it.


To work around that, you can try to redirect the output of your process after it is already running.  This is outside the scope of this guide, but this post provides an explanation of how you could do that.


## Using the huponexit Shell Option


bash has another way of avoiding the SIGHUP problem for child processes.  The huponexit shell option controls whether bash will send its child processes the SIGHUP signal when it exits.



Note: The huponexit option only affects the SIGHUP behavior when a shell session termination is initiated from within the shell itself.  Some examples of when this applies are when the exit command or CTRL + D is pressed within the session.
When a shell session is ended through the terminal program itself (through closing the window, etc.), the command huponexit will have no affect.  Instead of bash deciding on whether to send the SIGHUP signal, the terminal itself will send the SIGHUP signal to bash, which will then correctly propagate the signal to its child processes.

Despite the aforementioned caveats, the huponexit option is perhaps one of the easiest to manage.  You can determine whether this feature is on or off by typing:


```
shopt huponexit


```


To turn it on, type:


```
shopt -s huponexit


```


Now, if you exit your session by typing exit, your processes will all continue to run:


```
exit


```


This has the same caveats about program output as the last option, so make sure you have redirected your processes’ output prior to closing your terminal if this is important.


# Conclusion


Learning job control and how to manage foreground and background processes will give you greater flexibility when running programs on the command line.  Instead of having to open up many terminal windows or SSH sessions, you can often get by with stopping processes early or moving them to the background as needed.


