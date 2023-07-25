# A guide to atop command in Linux

```UNIX/Linux```

The atop command is a tool for monitoring system resources in Linux. It displays tons of information related to the amount of load on the system’s resources at the process level. There can be indefinite advantages to the user if this utility is mastered.


First things first, we have to install the atop command on the system. Debian/Ubuntu users can do so by:


```
sudo apt install atop

```


Other Linux users can use their standard package manager, followed by the 'atop' keyword.


This command has the ability to display multiple confidential information related to the system. In order to prevent any data abstraction, we can get elevated access using 'sudo su' or 'sudo -s'. We have complete documentation on sudo.


# Basic Output of the atop Command


To display all the process-level use of the system’s resources, we can simply run 'atop' in the terminal.


```
atop

```


Basic Output of ‘atop’
As we can see, the whole layout is divided into two panels. The upper panel provides the cumulative use of the system’s resources, whereas the bottom one, displays disintegrated information for each process. Let’s see each of the


## Cumulative Statistics of the atop command


Each entry in this view, focuses on a particular system resource.


##$ 1. Process-related Statistics


- PRC - stands for “process”.

The first two values are the time consumed by the 'sys' (system) and 'user' processes.
It is followed by the total number of processes as '#proc'.
The next value is the number of threads currently running in the system. ('#trun')
'#tslpi' denotes the number of threads that are currently sleeping and interruptible.
'#tslpu' denotes the number of threads that are currently sleeping and uninterruptible.
The following value is the number of zombie processes.
Next up is the number of clone system calls.
The last value is the number of processes that ended during the elapsed time. ('#exit')


- The first two values are the time consumed by the 'sys' (system) and 'user' processes.
- It is followed by the total number of processes as '#proc'.
- The next value is the number of threads currently running in the system. ('#trun')
- '#tslpi' denotes the number of threads that are currently sleeping and interruptible.
- '#tslpu' denotes the number of threads that are currently sleeping and uninterruptible.
- The following value is the number of zombie processes.
- Next up is the number of clone system calls.
- The last value is the number of processes that ended during the elapsed time. ('#exit')

##$ 2. Performance-related Statistics


- CPU - relates to CPU utilization.

The first two values show the percentage utilization of all the cores by the system and user processes.
The percentage of CPU used for interrupt requests. ('irq')
The next value is the idle percentage for all the cores combined.
The following value denotes the waiting each CPU core had to do.
Next up is the percentage for the steal time.
'guest' denotes the guest-percentage, which is the CPU time spent on other virtual machines.
The last two values indicate the current frequency of the CPU.


- The first two values show the percentage utilization of all the cores by the system and user processes.
- The percentage of CPU used for interrupt requests. ('irq')
- The next value is the idle percentage for all the cores combined.
- The following value denotes the waiting each CPU core had to do.
- Next up is the percentage for the steal time.
- 'guest' denotes the guest-percentage, which is the CPU time spent on other virtual machines.
- The last two values indicate the current frequency of the CPU.
- Now, the 'atop' displays the above statistics for each core independently.
- CPL - refers to as CPU Load.

The first three values are the average loads with different periods: 1, 5, and 15 minutes.
This is followed by the number of context switches ('csw')
Next up is the number of interrupts ('intr')
The last value is number of available CPUs.


- The first three values are the average loads with different periods: 1, 5, and 15 minutes.
- This is followed by the number of context switches ('csw')
- Next up is the number of interrupts ('intr')
- The last value is number of available CPUs.

##$ 3. Memory-related Statistics


- 
MEM - Memory Utilization


The total physical memory supported.


The memory currently free.


The current cache memory.


'buff' as in “buffer” is the amount of memory consumed in filesystem meta-data.


The sum of memory for kernel’s memory allocation shown as 'slab'.


The amount of shared memory.



- 
The total physical memory supported.

- 
The memory currently free.

- 
The current cache memory.

- 
'buff' as in “buffer” is the amount of memory consumed in filesystem meta-data.

- 
The sum of memory for kernel’s memory allocation shown as 'slab'.

- 
The amount of shared memory.

- 
SWP - Swap Memory.


##$ 3. Disk-related Statistics


- DSK - Disk usage

The first value denotes the percentage of time the system is busy handling requests.
The reading requests issued.
The writing requests issued.
The rate at which data (in KB) is read per reading request.
The rate at which data (in KB) is written per writing request.
The next two values are time rates for reading and writing on the disk in Megabytes.
The last value is the average number of milliseconds spent in handling requests.


- The first value denotes the percentage of time the system is busy handling requests.
- The reading requests issued.
- The writing requests issued.
- The rate at which data (in KB) is read per reading request.
- The rate at which data (in KB) is written per writing request.
- The next two values are time rates for reading and writing on the disk in Megabytes.
- The last value is the average number of milliseconds spent in handling requests.

##$ 4. Network-related Statistics


- NET - Network Statistics at the Transport Layer

'transport' signifies the Transport layer in Networking, which deals with the data protocols.
The number of segments received by the system following the TCP protocol. ('tcpi')
The number of segments transmitted. ('tcpo')
The similar statistics for UCP protocol. ('udpi' for UDP in) and ('udpo' for UDP out).
'tcpao' is the number of active TCP open connections.
Opposite to previous 'tcppo' is the number of passive TCP connections, but still open.
The figure of TCP retransmissions as 'tcprs'.
The figure of UDP input errors as 'udpie'.


- 'transport' signifies the Transport layer in Networking, which deals with the data protocols.
- The number of segments received by the system following the TCP protocol. ('tcpi')
- The number of segments transmitted. ('tcpo')
- The similar statistics for UCP protocol. ('udpi' for UDP in) and ('udpo' for UDP out).
- 'tcpao' is the number of active TCP open connections.
- Opposite to previous 'tcppo' is the number of passive TCP connections, but still open.
- The figure of TCP retransmissions as 'tcprs'.
- The figure of UDP input errors as 'udpie'.
- NET - Network Statistics at the Network Layer

'network' signifies the Network Layer, which deals with Internet Protocols, IPv4 and IPv6 combined.
The number of IP packets received by the network interfaces. ('ipi')
The number of IP packets transmitted out from the interfaces. ('ipo')
The quantity of IP packets forwarded to other interfaces. ('ipfrw')
The quantity of IP packets delivered. ('deliv')
The last two entries are the number of ICMP packets received and transmitted by the network interfaces.


- 'network' signifies the Network Layer, which deals with Internet Protocols, IPv4 and IPv6 combined.
- The number of IP packets received by the network interfaces. ('ipi')
- The number of IP packets transmitted out from the interfaces. ('ipo')
- The quantity of IP packets forwarded to other interfaces. ('ipfrw')
- The quantity of IP packets delivered. ('deliv')
- The last two entries are the number of ICMP packets received and transmitted by the network interfaces.
- The following lines refer to each active network interface.

The first value is the name of the network interface, like 'wlp19s0'.
The following two packets are the number of packets that were received and transmitted through the particular interface. ('pcki' and 'pcko')
The network speed in Megabits (Mbps) as 'sp'.
The rate at which bits are received and transmitted per second. ('si' and 'so')
The number of errors in packets received and transmitted. ('erri' and 'erro').
The last two values are the dropped packets in both ways. ('drpi' and 'drpo')


- The first value is the name of the network interface, like 'wlp19s0'.
- The following two packets are the number of packets that were received and transmitted through the particular interface. ('pcki' and 'pcko')
- The network speed in Megabits (Mbps) as 'sp'.
- The rate at which bits are received and transmitted per second. ('si' and 'so')
- The number of errors in packets received and transmitted. ('erri' and 'erro').
- The last two values are the dropped packets in both ways. ('drpi' and 'drpo')

This concludes the explanation of the top panel of the atop command.


## System Resources for each process


It is worth noticing that the values in 'atop' command keeps updating after certain time intervals.


The generic output of the 'atop' command displays the following details for each process entry:


- PID - The process ID.
- SYSCPU - The amount of CPU consumed by the process while system handling.
- USRCPU - The amount of CPU consumed by the process, during its running in user mode.
- VGROW - The amount of virtual memory that the process has occupied since the last value update.
- RGROW - The amount of resident (physical) memory grown since last value update.
- RDDSK - The size of data transferred during disk reads.
- WRDSK - The size of data transferred during disk writes.
- RUID - The real user ID under which the process is being executed.
- EUID - The effective user ID under which the process is being executed.
- ST - The current status of the process.
- EXC - The exit code after the process terminates
- THR - The number of threads within the process.
- S - The current status of the primary thread of the process.
- CPU - The percentage of CPU utilization of the entire process.
- CMD - The name of the process.

In this generic output, the processes are sorted on the basis of percentage CPU utilization. As we can see, in this particular output, we get small amount of information for every type of system resource.


Let us try to study process-level information for each type of system resource.


# The memory-based output of the atop command


The 'atop' command provides the opportunity to study the memory consumption for each process running in the system. We can do so by running:


```
atop -m

```


Memory-based output
As we can see, the top panel remains constant even if we added the memory option, '-m'. Let us now understand the columns for each process entry.


- PID - The process ID.
- TID - The thread ID.
- MINFLT - The number of minor page faults that have been solved by accessing data from free pages.
- MAJFLT - The number of major page faults that have been solved by especially retrieving data from disk.
- VSTEXT - The virtual memory occupied by the process text.
- VSLIBS - The virtual memory occupied by the shared libraries of the process.
- VDATA - The virtual memory size of the private data of the process.
- VSTACK - The virtual memory size of the private stack of the process.
- VSIZE - The total virtual memory size of the process.
- RSIZE - The total resident memory occupied by the process.
- MEM - The percentage of RAM consumed by the process.

The processes are sorted with respect to the 'MEM' column.



Since, 'atop' is somewhat an interactive command utility, we can alter the columns from within itself. All we have to do is type the specific option while it is displaying information.
For instance, after running 'atop' on the terminal, we can switch to memory-specific output by just typing 'm'.

# Disk-specific output using the atom command in Linux


To extract information related to disk utilization, we can use the '-d' option along with 'atop' command.


```
atop -d

```


Disk-specific output
There is not a lot of stuff to notice in the disk-specific output. Some of the key findings are:


- RDDSK - The size of data transferred during disk reads.
- WRDSK - The size of data transferred during disk writes.
- WCANCL - The size of data initially written, but later withdrawn
- DSK - The percentage of Disk occupied.
- CMD - The name of the process.

It must be noted that, the processes are sorted on the bases of 'DSK' column.


# Find Commands Running in the Background with atop command


This gives us the commands that are running in the background as processes in a command-line output format.


```
atop -c

```


Command-line for each process
If you copy-paste the lines under the command line column, you can re-run the same process. This output tells us exactly what command was run in the background to initiate the process.


# Thread-based Information


Instead of just inspecting process information, atop command provides the ability to check for thread-specific resource utilization. To access this output we can either run:


```
atop -y

```


or just press 'y' key when the command is already displaying system resource information.


Thread-specific output
It is clear that none of the system resource columns have changed. All that has been added is the thread count of their respective process.


# Miscellaneous information


There can numerous kinds of information that can be extracted using the 'atop' command. Some of the useful ones are:


## 1. Find process start-times


Using the '-v' option, we can get process characteristics.


```
atop -v

```


Process start-times
## 2. Number of processes for each user in the system


```
atop -au

```


User-based processes
## 3. Which core a process is working on?


This specific kind of information comes under scheduling characteristics for a process. It can be accessed by using '-s' option.


```
atop -s

```


Core number for each process
# Few ‘atop’ tricks


There are certain 'atop' command tricks that might be useful:


- Pausing the 'atop' screen - using 'z' key.
- Changing the time interval of value updates - using 'i' key followed by the number of seconds, we wish to change it to.
- Interrupt to instantly update the values - using 't' key.
- Quitting the display - using 'q' key.

# Conclusion


We know that 'atop' command can be too much to handle for any Linux user. It takes patience and perseverance to learn about this brilliant command. For any queries, feel free to ping us down in the comments section.


