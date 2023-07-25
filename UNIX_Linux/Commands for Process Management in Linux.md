# Commands for Process Management in Linux

```UNIX/Linux```

In this article, we’ll discuss process management in Linux. A process in Linux is nothing but a program in execution. It’s a running instance of a program. Any command that you execute starts a process.


# Types of Processes in Linux


In Linux processes can be of two types:


- 
Foreground Processes
depend on the user for input
also referred to as interactive processes

- 
Background Processes
run independently of the user
referred to as non-interactive or automatic processes


## Process States in Linux


A process in Linux can go through different states after it’s created and before it’s terminated. These states are:


- 
Running

- 
Sleeping

Interruptible sleep
Uninterruptible sleep


- Interruptible sleep
- Uninterruptible sleep
- 
Stopped

- 
Zombie

- 
A process in running state means that it is running or it’s ready to run.

- 
The process is in a sleeping state when it is waiting for a resource to be available.

- 
A process in Interruptible sleep will wakeup to handle signals, whereas a process in Uninterruptible sleep will not.

- 
A process enters a stopped state when it receives a stop signal.

- 
Zombie state is when a process is dead but the entry for the process is still present in the table.


# Different Commands for Process Management in Linux


There are two commands available in Linux to track running processes. These two commands are Top and Ps.


## 1. The top Command for Mananging Linux Processes


To track the running processes on your machine you can use the top command.


```
$ top 

```





Top command displays a list of processes that are running in real-time along with their memory and CPU usage. Let’s understand the output a little better:


- PID: Unique Process ID given to each process.
- User: Username of the process owner.
- PR: Priority given to a process while scheduling.
- NI: ‘nice’ value of a process.
- VIRT: Amount of virtual memory used by a process.
- RES: Amount of physical memory used by a process.
- SHR: Amount of memory shared with other processes.
- S: state of the process

‘D’ = uninterruptible sleep
‘R’ = running
‘S’ = sleeping
‘T’ = traced or stopped
‘Z’ = zombie


- ‘D’ = uninterruptible sleep
- ‘R’ = running
- ‘S’ = sleeping
- ‘T’ = traced or stopped
- ‘Z’ = zombie
- %CPU: Percentage of CPU used by the process.
- %MEM; Percentage of RAM used by the process.
- TIME+: Total CPU time consumed by the process.
- Command: Command used to activate the process.

You can use the up/down arrow keys to navigate up and down through the list. To quit press q. To kill a process, highlight the process with the up/down arrow keys and press ‘k’.


Alternatively, you can also use the kill command, which we will see later.


## 2. ps command


ps command is short for ‘Process Status’. It displays the currently-running processes. However, unlike the top command, the output generated is not in realtime.


```
$ ps

```





The terminology is as follows :











PID
process ID


TTY
terminal type


TIME
total time the process has been running


CMD
name of the command that launches the process




To get more information using ps command use:


```
$ ps -u

```





Here:


- %CPU represents the amount of computing power the process is taking.
- %MEM represents the amount of memory the process is taking up.
- STAT represents process state.

While ps command only displays the processes that are currently running, you can also use it to list all the processes.


```
$ ps -A 

```





This command lists even those processes that are currently not running.


## 3. Stop a process


To stop a process in Linux, use the 'kill’ command. kill command sends a signal to the process.


There are different types of signals that you can send. However, the most common one is ‘kill -9’ which is ‘SIGKILL’.


You can list all the signals using:


```
$ kill -L

```


Kill L
The default signal is 15, which is SIGTERM. Which means if you just use the kill command without any number, it sends the SIGTERM signal.


The syntax for killing a process is:


```
$ kill [pid]

```


Alternatively you can also use :


```
$ kill -9 [pid]

```


This command will send a ‘SIGKILL’ signal to the process. This should be used in case the process ignores a normal kill request.


## 4. Change priority of a process


In Linux, you can prioritize between processes. The priority value for a process is called the ‘Niceness’ value. Niceness value can range from -20 to 19. 0 is the default value.


The fourth column in the output of top command is the column for niceness value.





To start a process and give it a nice value other than the default one, use:


```
$ nice -n [value] [process name]

```


To change nice value of a process that is already running use:


```
renice [value] -p 'PID'

```


# Conclusion


This tutorial covered process management in Linux. Mainly the practical aspects of process management were covered. In theory, process management is a vast topic and covering it in its entirety is out of scope for this tutorial.


