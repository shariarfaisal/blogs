# How To Manage Redis Databases and Keys

```Databases``` ```Ubuntu 22.04``` ```Redis``` ```NoSQL```

## Introduction


Redis is an open-source, in-memory key-value data store. A key-value data store is a type of NoSQL database in which keys serve as unique identifiers for their associated values. Any given Redis instance includes a number of databases, each of which can hold many different keys of a variety of data types.


In this tutorial, you will learn how to select a database, move keys between databases, and manage and delete keys.


##$ How To Use This Guide


This guide is written as a cheat sheet with self-contained examples. We encourage you to jump to any section that is relevant to the task you’re trying to complete.


The commands shown in this guide were tested on an Ubuntu 22.04 server running Redis version 6.0.16. To set up a similar environment, you can follow Step 1 of our guide on How To Install and Secure Redis on Ubuntu 22.04. We will demonstrate how these commands behave by running them with redis-cli, the Redis command line interface. Note that if you’re using a different Redis interface — Redli, for example — the exact output of certain commands may differ.


Alternatively, you could provision a managed Redis database instance to test these commands, but depending on the level of control allowed by your database provider, some commands in this guide may not work as described. To provision a DigitalOcean Managed Database, follow our Managed Databases product documentation. Then, you must either install Redli or set up a TLS tunnel in order to connect to the Managed Database over TLS.


# Managing Databases


Out of the box, a Redis instance supports 16 logical databases. These databases are effectively siloed off from one another, and when you run a command in one database, it doesn’t affect any of the data stored in other databases in your Redis instance.


Redis databases are numbered from 0 to 15 and, by default, you connect to database 0 when you connect to your Redis instance. However, you can change the database you’re using with the select command after you connect:


```
select 15


```


If you’ve selected a database other than 0, it will be reflected in the redis-cli prompt:


```



```


To swap all the data held in one database with the data held in another, use the swapdb command. The following example will swap the data held in database 6 with the data in database 8, and any clients connected to either database will be able to implement changes immediately:


```
swapdb 6 8


```


swapdb will return OK if the swap is successful.


If you want to move a key to a different Redis instance, you can run migrate. This command ensures the key exists on the target instance before deleting it from the source instance. When you run migrate, the command must include the following elements in this order:


- The hostname or IP address of the destination database
- The target database’s port number
- The name of the key you want to migrate
- The database number where you want to store the key on the destination instance
- A timeout, in milliseconds, which defines the maximum amount of idle communication time between the two machines. Note that this isn’t a time limit for the operation, but means that the operation should always make some level of progress within the defined length of time

To illustrate, here’s an example:


```
migrate 203.0.113.0 6379 key_1 7 8000


```


Additionally, migrate allows the following options which you can add after the timeout argument:


- COPY: Specifies that the key should not be deleted from the source instance
- REPLACE: Specifies that if the key already exists on the destination, the migrate operation should delete and replace it
- KEYS: Instead of providing a specific key to migrate, you can enter an empty string ("") and then use the syntax from the keys command to migrate any key that matches a pattern. For more information on how keys works, read our tutorial on How To Troubleshoot Issues in Redis.

# Managing Keys


There are a number of Redis commands that are useful for managing keys regardless of what type of data they hold. Some of these commands are reviewed in the following section.


rename will rename the specified key. If it’s successful, it will return OK:


```
rename old_key new_key


```


You can use randomkey to return a random key from the currently selected database:


```
randomkey


```


```
Output"any_key"

```


Use type to determine what type of data the given key holds. This command’s output can be either string, list, hash, set, zset, or stream:


```
type key_1


```


```
Output"string"

```


If the specified key doesn’t exist, type will return none instead.


You can move an individual key to another database in your Redis instance with the move command. move takes the name of a key and the database where you want to move the key as arguments. For example, to move the key key_1 to database 8, you would run the following:


```
move key_1 8


```


move will return OK if moving the key was successful.


# Deleting Keys


To delete one or more keys of any data type, use the del command followed by one or more keys that you want to delete:


```
del key_1 key_2


```


If this command deletes the key(s) successfully, it will return (integer) 1. Otherwise, it will return (integer) 0.


The unlink command performs a similar function as del, with the difference being that del blocks the client as the server reclaims the memory taken up by the key. If the key being deleted is associated with a small object, the amount of time it takes for del to reclaim the memory is very small and the blocking time may not even be noticeable.


However, it can become inconvenient if, for example, the key you’re deleting is associated with many objects, such as a hash with thousands or millions of fields. Deleting such a key can take a noticeably long time, and you’ll be blocked from performing any other operations until it’s fully removed from the server’s memory.


unlink, however, first determines the cost of deallocating the memory taken up by the key. If it’s small, then unlink functions the same way as del by the key immediately while also blocking the client. However, if there’s a high cost to deallocate memory for a key, unlink will delete the key asynchronously by creating another thread and incrementally reclaim memory in the background without blocking the client:


```
unlink key_1


```


Since it runs in the background, it’s generally recommended that you use unlink to remove keys from your server to reduce errors on your clients, though del will also suffice in many cases.



Warning: The following two commands are considered dangerous. The flushdb and flushall commands will irreversibly delete all the keys in a single database and all the keys in every database on the Redis server, respectively. It’s recommended that you only run these commands if you are absolutely certain that you want to delete all the keys in your database or server.
It may be in your interest to rename these commands to something with a lower likelihood of being run accidentally.

To delete all the keys in the selected database, use the flushdb command:


```
flushdb


```


To delete all the keys in every database on a Redis server (including the currently selected database), run flushall:


```
flushall


```


Both flushdb and flushall accept the async option, which allows you to delete all the keys on a single database or every database in the cluster asynchronously. This allows them to function similarly to the unlink command, and they will create a new thread to incrementally free up memory in the background.


# Backing Up Your Database


To create a backup of the currently selected database, you can use the save command:


```
save


```


This will export a snapshot of the current dataset as an .rdb file, which is a database dump file that holds the data in an internal, compressed serialization format.


save runs synchronously and will block any other clients connected to the database. Hence, the save command documentation recommends that this command should almost never be run in a production environment. Instead, it suggests using the bgsave command. This tells Redis to fork the database: the parent will continue to serve clients while the child process saves the database before exiting:


```
bgsave


```


Note that if clients add or modify data while the bgsave operation is occurring, these changes won’t be captured in the snapshot.


You can also edit the Redis configuration file to have Redis save a snapshot automatically (known as snapshotting or RDB mode) after a certain amount of time if a minimum number of changes were made to the database. This is known as a save point. The following save point settings are enabled by default in the redis.conf file:


/etc/redis/redis.conf
```
. . .
save 900 1
save 300 10
save 60 10000
. . .
dbfilename "nextfile.rdb"
. . .

```


With these settings, Redis will export a snapshot of the database to the file defined by the dbfilename parameter every 900 seconds if at least one key is changed, every 300 seconds if at least 10 keys are changed, and every 60 seconds if at least 10000 keys are changed.


You can use the shutdown command to back up your Redis data and then close your connection. This command will block every client connected to the database and then perform a save operation if at least one save point is configured, meaning that it will export the database in its current state to an .rdb file while preventing clients from making any changes.


Additionally, the shutdown command will flush changes to Redis’s append-only file before quitting if append-only mode is enabled. The append-only file mode (AOF) involves creating a log of every write operation on the server in a file ending in .aof after every snapshot. AOF and RDB modes can be enabled on the same server, and using both persistence methods is an effective way to back up your data.


In short, the shutdown command is essentially a blocking save command that also flushes all recent changes to the append-only file and closes the connection to the Redis instance:



Warning: The shutdown command is considered dangerous. By blocking your Redis server’s clients, you can make your data unavailable to users and applications that depend on it. It’s recommended that you only run this command if you are testing out Redis’s behavior or if you are absolutely certain that you want to block all your Redis server’s clients.
In fact, it may be in your interest to rename this command to something with a lower likelihood of being run accidentally.

```
shutdown


```


If you’ve not configured any save points but still want Redis to perform a save operation, append the save option to the shutdown command:


```
shutdown save


```


If you have configured at least one save point but want to shut down the Redis server without performing a save, you can add the nosave argument to the command:


```
shutdown nosave


```


Note that the append-only file can grow to be very long over time, but you can configure Redis to rewrite the file based on certain variables by editing the redis.conf file. You can also instruct Redis to rewrite the append-only file by running the bgrewriteaof command:


```
bgrewriteaof


```


bgrewriteaof will create the shortest set of commands needed to bring the database back to its current state. As this command’s name implies, it will run in the background. However, if another persistence command is running in a background process already, that command must finish before Redis will execute bgrewriteaof.


# Conclusion


This guide details a number of commands used to manage databases and keys. If there are other related commands, arguments, or procedures you’d like to learn about in this guide, please ask or make suggestions in the comments.


For more information on Redis commands, check out our tutorial series on How to Manage a Redis Database.


