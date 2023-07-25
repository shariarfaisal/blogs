# What is Load Average in Linux 

```UNIX/Linux```

Load Average in Linux is a metric that is used by Linux users to keep track of system resources. It also helps you monitor how the system resources are engaged.


While Load Average is one of the most fundamental metrics of resource usage, the metric is pointless unless you understand what it tells a user. In this tutorial, we will help you understand what Load Average in Linux means.


Further, we will discuss some easy methods to monitor the load average for your system.


# Basics of Load Average in Linux


To understand the Load Average in Linux, we need to know what do we define as load. In a Linux system, the load is a measure of CPU utilization at any given moment.


It refers to the number of processes which are either currently being executed by the CPU or are waiting for execution.


An idle system has a load of 0. With each process that is being executed or is on the waitlist, the load increases by 1.


On its own, the load doesn’t give any useful information to the user. The load can change in split seconds. This is because the number of processes using or waiting for the CPU time doesn’t remain constant. This is why we use Load Average in Linux to monitor resource usage.


# Getting familiar with the Load Average in Linux


Load average, as the name suggests, depicts the average load on a CPU for a set time interval. These values are the number of processes waiting for the CPU or using it in the given period.


While most people are used to the load percentages shown in Windows systems, Load Average in Linux is depicted as three different decimal values.


 Load Average
Have look at the image above where it says “load average: 0.03, 0.03, 0.01”


Going left to right:


- The first value depicts the average load on the CPU for the last minute.
- The second gives us the average load for the last 5-minute interval
- The third value gives us the 15-minute average load

This helps a user get an idea of how the CPU is being utilized by the processes on a system over time.


While a load of 1 can mean approximately 100% resource usage on a single processor system, such systems are practically non-existent today. Unless you haven’t upgraded your system in over a decade, your system should run on a multi-core processor.


For a dual-core processor, a load of 1 means that 1 core was 100% idle. This translates to approximately 50% CPU usage. Similarly, it would represent 25% CPU usage for a quad-core processor.


Load Average in Linux takes into account the waiting threads and tasks along with processes being executed. Also, it is an average value instead of being an instantaneous value.


However, an approximate idea of resource usage can be determined by the ratio of Load Average over the number of cores of your processor. While it is not an exact value for the CPU utilization at any given time, it can be helpful for resource monitoring.


# How to Check the Load Average in Linux


Now that we know what Load Average represents, we will discuss a few ways to check the Load Average in Linux. Load Average can be looked up in three common ways.


## 1. Using uptime command


The uptime command is one of the most common methods for checking the Load Average for your system. To use the uptime command, we simply open the command line and type the following.


```
uptime

```


Uptime Command Load Average
This displays the amount of time that our system has been up for, along with the number of active users and the Load Average for our system. The following screenshot shows what should you see when you use the uptime command on your system.


As you can see, the load average for the last minute is 0.03. For the last five minutes and fifteen minutes, the Load Average values are 0.03 and 0.01 respectively.


## 2. Using top command


Another way to monitor the Load Average on your system is to utilise the top command in Linux. To do so, simply open the terminal and type this.


```
top

```


This will open the top interface in your terminal. Unlike the uptime command, this gives an in-depth view of the resource usage for your system.


The following screenshot shows what should you see when you use the top command on your system.


Top Command Load Average
As you can see in the top-most line, the load average for the last minute is 0.34. For the last five minutes and fifteen minutes, the Load Average values are 0.14 and 0.405 respectively.


## 3. Using glances tool


The glances tool is a system monitoring tool which works similar to the top command. It gives a detailed overview of the system resource usage. To use the glances tool on your system, you need to install its package using this command.


```
sudo apt-get install glances

```


Once you are done with the installation, type the following in your terminal.


```
glances

```


This will open the glances interface in your terminal. Unlike the top command, this gives the number of processor cores available along with the Load Average for your system.


The following screenshot shows what should you see when you use the glances command on your system.


Glances Command Load Average
As you can see in the highlighted region, the load average for the last minute is 0.14. For the last five minutes and fifteen minutes, the Load Average values are 0.12 and 0.05 respectively.


# Wrapping up


The Load Average in Linux is an essential metric to monitor the usage of system resources easily. Keeping the load average in check helps ensure that your system does not experience a crash or sluggish sessions.


We hope this tutorial was able to help you to get familiar with the concept of Load Average in Linux.


