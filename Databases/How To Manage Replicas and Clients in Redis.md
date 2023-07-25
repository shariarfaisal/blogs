# How To Manage Replicas and Clients in Redis

```Databases``` ```Ubuntu 22.04``` ```Redis``` ```NoSQL```

## Introduction


Redis is an open-source, in-memory key-value data store. One of its most sought-after features is its support for replication. Any Redis server can replicate its data to any number of replicas, allowing for high read scalability and strong data redundancy. Additionally, Redis was designed to allow many clients (up to 10000, by default) to connect and interact with data, making it a good choice for cases where many users need access to the same dataset.


This tutorial will provide an overview of the commands used to manage Redis clients and replicas.


##$ How To Use This Guide


This guide is written as a cheat sheet with self-contained examples. We encourage you to jump to any section that is relevant to the task you’re trying to complete.


The commands shown in this guide were tested on an Ubuntu 22.04 server running Redis version 6.0.16. To set up a similar environment, you can follow Step 1 of our guide on How To Install and Secure Redis on Ubuntu 22.04. We will demonstrate how these commands behave by running them with redis-cli, the Redis command line interface. If you’re using a different Redis interface — Redli, for example — the exact output of certain commands may differ.


Alternatively, you could provision a managed Redis database instance to test these commands, but depending on the level of control allowed by your database provider, some commands in this guide may not work as described. To provision a DigitalOcean Managed Database, follow our Managed Databases product documentation. Then, you must either install Redli or set up a TLS tunnel to connect to the Managed Database over TLS.



Note: The Redis project uses the terms “master” in its documentation and in various commands to identify different roles in replication, though the project’s contributors are taking steps to change this language in cases where it doesn’t cause compatibility issues. DigitalOcean generally prefers to use the alternative terms “primary” and “replica”.
This guide will default to “primary” and “replica” whenever possible, but note that there are a few instances where the terms “master” unavoidably come up.

# Managing Replicas


One of Redis’s most distinguishing features is its built-in replication. When using replication, Redis creates exact copies of the primary instance. These secondary instances reconnect to the primary any time their connections break and will always aim to remain an exact copy of the primary.


If you’re not sure whether the Redis instance you’re currently connected to is a primary instance or a replica, you can check by running the role command:


```
role


```


This command will return either master or replica, or potentially sentinel if you’re using Redis Sentinel.


To designate a Redis instance as a replica of another instance on the fly, run the replicaof command. This command takes the intended primary server’s hostname or IP address and port as arguments:


```
replicaof hostname_or_IP port


```


If the server is already a replica of another primary, it will stop replicating the old server and immediately start synchronizing with the new one. It will also discard the old dataset.


To promote a replica back to being a primary, run the following replicaof command:


```
replicaof no one


```


This will stop the instance from replicating the primary server, but will not discard the dataset it has already replicated. This syntax is useful in cases where the original primary fails. After running replicaof no one on a replica of the failed primary, the former replica can be used as the new primary and have its own replicas as a failsafe.


# Managing Clients


A client is any machine or software that connects to a server in order to access a service. Redis comes with several commands that help with tracking and managing client connections.


The client list command returns a set of human-readable information about current client connections:


```
client list


```


```
Output"id=18165 addr=[2001:db8:0:0::12]:47460 fd=7 name=jerry age=72756 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=ping
id=18166 addr=[2001:db8:0:1::12]:47466 fd=8 name= age=72755 idle=5 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=info
id=19381 addr=[2001:db8:0:2::12]:54910 fd=9 name= age=9 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=26 qbuf-free=32742 obl=0 oll=0 omem=0 events=r cmd=client
"

```


Here is what each of these fields mean:


- id: a unique 64-bit client ID
- name: the name of the client connection, as defined by a prior client setname command
- addr: the address and port from which the client is connecting
- fd: the file descriptor that corresponds to the socket over which the client is connecting
- age: the total duration of the client connection, in seconds
- flags: a set of one or more single-character flags that provide more granular detail about the clients; review the client list command documentation for more details
- db: the current database ID number that the client is connected to. It can range from 0 to 15
- sub: the number of channels the client is subscribed to
- psub: the number of the client’s pattern-matching subscriptions
- mutli: the number of commands the client has queued in a transaction. This will show -1 if the client hasn’t begun a transaction or 0 if it has only started a transaction and not queued any commands
- qbuf: the client’s query buffer length, with 0 meaning it has no pending queries
- qbuf-free: the amount of free space in the client’s query buffer, with 0 meaning that the query buffer is full
- obl: the client’s output buffer length
- oll: the length of the client’s output list, where replies are queued when its buffer is full
- omem: the memory used by the client’s output buffer
- events: the client’s file descriptor events. These can be r for “readable”, w for “writable,” or both
- cmd: the last command run by the client

Setting client names is useful for debugging connection leaks in whatever application is using Redis. Every new connection starts without an assigned name, but client setname can be used to create one for the current client connection. There’s no limit to how long client names can be, although Redis usually limits string lengths to 512 MB. Note, though, that client names cannot include spaces:


```
client setname elaine


```


To retrieve the name of a client connection, use the client getname command:


```
client getname


```


```
Output"elaine"

```


To retrieve a client’s connection ID, use the client id command:


```
client id


```


```
Output(integer) "19492"

```


Redis client IDs are never repeated and are monotonically incremental. This means that if one client has an ID greater than another, then it was established at a later time.


# Blocking Clients and Closing Client Connections


Replication systems are typically described as being either synchronous or asynchronous. In synchronous replication, whenever a client adds or changes data, it must receive some kind of acknowledgment from a certain number of replicas for the change to register as having been committed. This helps to prevent nodes from having data conflicts, but it comes at a cost of latency since the client must wait to perform another operation until it has heard back from a certain number of replicas.


In asynchronous replication, the client receives a confirmation that the operation is finished as soon as the data is written to local storage. There can, however, be a lag between this and when the replicas actually write the data. If one of the replicas fails before it can write the change, that write will be lost forever. So while asynchronous replication allows clients to continue performing operations without the latency caused by waiting for replicas, it can lead to data conflicts between nodes and may require extra work on the part of the database administrator to resolve those conflicts.


Because of its focus on performance and low latency, Redis implements asynchronous replication by default. However, you can simulate synchronous replication with the wait command. wait blocks the current client connection for a specified amount of time (in milliseconds) until all the previous write commands are successfully transferred and accepted by a specified number of replicas. This command uses the following syntax:


```
wait number_of_replicas number_of_milliseconds


```


For example, if you want to block your client connection until all the previous writes are registered by at least 3 replicas within a 30-millisecond timeout, your wait syntax would be written as the following:


```
wait 3 30


```


The wait command returns an integer representing the number of replicas that acknowledged the write commands, even if not every replica does so:


```
Output2

```


To unblock a client connection that has been previously blocked, whether from a wait, brpop, or xread command, you can run a client unblock command with the following syntax:


```
client unblock client_id


```


To temporarily suspend every client currently connected to the Redis server, you can use the client pause command. This is useful in cases where you need to make changes to your Redis setup in a controlled way. For example, if you’re promoting one of your replicas to be the primary instance, you might pause every client beforehand so you can promote the replica and have the clients connect to it as the new primary without losing any write operations in the process.


The client pause command requires you to specify the amount of time (in milliseconds) you’d like to suspend the clients. The following example suspends all clients for one second:


```
client pause 1000


```


The client kill syntax allows you to close a single connection or a set of specific connections based on a number of different filters. The syntax is written as follows:


```
client kill filter_1 value_1 ... filter_n value_n


```


The following filters are available:


- addr: allows you to close a client connection from a specified IP address and port
- client-id: allows you to close a client connection based on its unique ID field
- type: closes every client of a given type, which can be either normal, master, replica, or pubsub
- skipme: the value options for this filter are yes and no:

if no is specified, the client calling the client kill command will not get skipped and will be killed if the other filters apply to it
if yes is specified, the client running the command will be skipped and the kill command will have no effect on the client. skipme is always yes by default


- if no is specified, the client calling the client kill command will not get skipped and will be killed if the other filters apply to it
- if yes is specified, the client running the command will be skipped and the kill command will have no effect on the client. skipme is always yes by default

# Conclusion


This guide details a number of commands used to manage Redis clients and replicas. If there are other related commands, arguments, or procedures you’d like to learn about in this guide, please ask or make suggestions in the comments.


For more information on Redis commands, check out our tutorial series on How to Manage a Redis Database.


