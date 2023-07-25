# How To Troubleshoot Issues in Redis

```Databases``` ```Ubuntu 22.04``` ```Redis``` ```NoSQL```

## Introduction


Redis is an open-source, in-memory key-value data store. It comes with several commands that can help with troubleshooting and debugging issues. Because of Redis’s nature as an in-memory key-value store, many of these commands focus on memory management, but there are others that are valuable for providing an overview of the state of your Redis server. This tutorial will provide details on how to use some of these commands to help diagnose and resolve issues you may run into as you use Redis.


##$ How To Use This Guide


This guide is written as a cheat sheet with self-contained examples. We encourage you to jump to any section that is relevant to the task you’re trying to complete.


The commands shown in this guide were tested on an Ubuntu 22.04 server running Redis version 6.0.16. To set up a similar environment, you can follow Step 1 of our guide on How To Install and Secure Redis on Ubuntu 22.04. We will demonstrate how these commands behave by running them with redis-cli, the Redis command line interface. If you’re using a different Redis interface — Redli, for example — the exact output of certain commands may differ.


Alternatively, you could provision a managed Redis database instance to test these commands, but depending on the level of control allowed by your database provider, some commands in this guide may not work as described. To provision a DigitalOcean Managed Database, follow our Managed Databases product documentation. Then, you must either install Redli or set up a TLS tunnel in order to connect to the Managed Database over TLS.


# Troubleshooting Memory-related Issues


memory usage tells you how much memory is currently being used by a single key. It takes the name of a key as an argument and outputs the number of bytes it uses. First, set an example variable:


```
set key_meaningOfLife "Food"


```


Next, check the memory with memory usage:


```
memory usage key_meaningOfLife


```


```
Output(integer) 88

```


For a more general understanding of how your Redis server is using memory, you can run the memory stats command:


```
memory stats


```


This command outputs an array of memory-related metrics and their values. The following are the metrics reported by memory stats:


- peak.allocated: The peak number of bytes consumed by Redis
- total.allocated: The total number of bytes allocated by Redis
- startup.allocated: The initial number of bytes consumed by Redis at startup
- replication.backlog: The size of the replication backlog, in bytes
- clients.slaves: The total size of all replica overheads, meaning the output and query buffers and connection contexts
- clients.normal: The total size of all client overheads
- aof.buffer: The total size of the current and rewrite append-only file buffers
- db.0: The overheads of the main and expiry dictionaries for each database in use on the server, reported in bytes
- overhead.total: The sum of all overheads used to manage Redis’s keyspace
- keys.count: The total number of keys stored in all the databases on the server
- keys.bytes-per-key: The ratio of the server’s net memory usage and keys.count
- dataset.bytes: The size of the dataset, in bytes
- dataset.percentage: The percentage of Redis’s net memory usage taken by dataset.bytes
- peak.percentage: The percentage of peak.allocated taken out of total.allocated
- fragmentation: The ratio of the amount of memory currently in use divided by the physical memory Redis is actually using

memory malloc-stats provides an internal statistics report from jemalloc, the memory allocator used by Redis on Linux systems:


```
memory malloc-stats


```


If it seems like you’re running into memory-related issues, but parsing the output of the previous commands proves to be unhelpful, you can try running memory doctor:


```
memory doctor


```


This feature will output any memory consumption issues that it can find and suggest potential solutions.


# Getting General Information about Your Redis Instance


A debugging command that isn’t directly related to memory management is monitor. This command allows you to review a constant stream of every command processed by the Redis server:


```
monitor


```


```
OutputOK
1566157213.896437 [0 127.0.0.1:47740] "auth" "foobared"
1566157215.870306 [0 127.0.0.1:47740] "set" "key_1" "878"

```


Another command useful for debugging is info, which returns several blocks of information and statistics about the server:


```
info


```


```
Output# Server
redis_version:6.0.16
redis_git_sha1:00000000
redis_git_dirty:0
redis_build_id:a3fdef44459b3ad6
redis_mode:standalone
os:Linux 5.15.0-41-generic x86_64
. . .

```


This command returns a lot of information. If you only want to return only one info block, you can specify it as an argument to info:


```
info CPU


```


```
Output# CPU
used_cpu_sys:173.16
used_cpu_user:70.89
used_cpu_sys_children:0.01
used_cpu_user_children:0.04

```


Note that the information returned by the info command will depend on which version of Redis you’re using.


# Using the keys Command


The keys command is helpful in cases where you’ve forgotten the name of a key, or perhaps you’ve created one but accidentally misspelled its name. keys searches for keys that match a pattern:


```
keys pattern


```


The following glob-style variables are supported:


- ? is a wildcard standing for any single character, so s?mmy matches sammy, sommy, and sqmmy
- * is a wildcard that stands for any number of characters, including no characters at all, so sa*y matches sammy, say, sammmmmmy, and salmony
- You can specify two or more characters that the pattern can include by wrapping them in brackets, so s[ai]mmy will match sammy and simmy, but not summy
- To set a wildcard that disregards one or more letters, wrap them in brackets and precede them with a carrot (^), so s[^oi]mmy will match sammy and sxmmy, but not sommy or simmy
- To set a wildcard that includes a range of letters, separate the beginning and end of the range with a hyphen and wrap it in brackets, so s[a-o]mmy will match sammy, skmmy, and sommy, but not srmmy


Warning: The Redis documentation warns that keys should almost never be used in a production environment since it can have a major negative impact on performance.

# Conclusion


This guide details a number of commands that are helpful for troubleshooting and resolving issues one might encounter as they work with Redis. If there are other related commands, arguments, or procedures you’d like to learn about in this guide, please ask or make suggestions in the comments.


For more information on Redis commands, check out our tutorial series on How to Manage a Redis Database.


