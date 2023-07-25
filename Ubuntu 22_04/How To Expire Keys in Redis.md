# How To Expire Keys in Redis

```Databases``` ```Ubuntu 22.04``` ```Redis``` ```NoSQL```

## Introduction


Redis is an open-source, in-memory key-value data store. Redis keys are persistent by default, meaning that the Redis server will continue to store them unless they are deleted manually. There may, however, be cases where you’ve set a key but know you will want to delete it after a certain amount of time has passed. In other words, you want the key to be volatile.


This tutorial explains how to set keys to expire, check the time remaining until a key’s expiration, and cancel a key’s expiration setting.


##$ How To Use This Guide


This guide is written as a cheat sheet with self-contained examples. We encourage you to jump to any section that is relevant to the task you’re trying to complete.


The commands shown in this guide were tested on an Ubuntu 22.04 server running Redis version 6.0.16. To set up a similar environment, you can follow Step 1 of our guide on How To Install and Secure Redis on Ubuntu 22.04. We will demonstrate how these commands behave by running them with redis-cli, the Redis command line interface. If you’re using a different Redis interface — Redli, for example — the exact output of certain commands may differ.


Alternatively, you could provision a managed Redis database instance to test these commands, but depending on the level of control allowed by your database provider, some commands in this guide may not work as described. To provision a DigitalOcean Managed Database, follow our Managed Databases product documentation. Then, you must either install Redli or set up a TLS tunnel in order to connect to the Managed Database over TLS.


# Setting Keys to Expire


You can set an expiration time for an existing key with the expire command, which takes the name of the key and the number of seconds until expiration as arguments. To demonstrate this, run the following two commands. The first creates a string key named key_melon with a value of "cantaloupe":


```
set key_melon "cantaloupe"


```


The second command sets it to expire after 450 seconds:


```
expire key_melon 450


```


If the timeout was set successfully, the expire command will return (integer) 1. If setting the timeout fails, it will instead return (integer) 0.


Alternatively, you can set the key to expire at a specific point in time with the expireat command. Instead of the number of seconds before expiration, it takes a Unix timestamp as an argument. A Unix timestamp is the number of seconds since the Unix epoch, or 00:00:00 UTC on January 1, 1970. There are a number of tools online you can use to find the Unix timestamp of a specific date and time, such as EpochConverter or UnixTimestamp.com.


For example, to set key_melon to expire at 8:30pm GMT on May 1, 2025 (represented by the Unix timestamp 1746131400), you can use the following command:


```
expireat key_melon 1746131400


```


Note that if the timestamp you pass to expireat has already occurred, it will delete the key immediately.


# Checking How Long Until a Key Expires


Any time you set a key to expire, you can check the time remaining until expiration (in seconds) with ttl, which stands for “time to live”:


```
ttl key_melon


```


```
Output(integer) 79247184

```


For more granular information, you can run pttl which will instead return the amount of time until a key expires in milliseconds:


```
pttl key_melon


```


```
Output(integer) 79247156730

```


Both ttl and pttl will return (integer) -1 if the key hasn’t been set to expire and (integer) -2 if the key does not exist.


# Persisting Keys


If a key has been set to expire, any command that overwrites the contents of a key — like set or getset — will clear a key’s timeout value. To manually clear a key’s timeout, use the persist command:


```
persist key_melon


```


The persist command will return (integer) 1 if it is completed successfully, indicating that the key will no longer expire.


# Conclusion


This guide details a number of commands used to manipulate and check key persistence in Redis. If there are other related commands, arguments, or procedures you’d like to learn about in this guide, please ask or make suggestions in the comments.


For more information on Redis commands, check out our tutorial series on How to Manage a Redis Database.


