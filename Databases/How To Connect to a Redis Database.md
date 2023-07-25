# How To Connect to a Redis Database

```Databases``` ```Ubuntu 22.04``` ```Redis``` ```NoSQL```

## Introduction


Redis is an open-source, in-memory key-value data store. Whether you’ve installed Redis locally or you’re working with a remote instance, you need to connect to it to perform most operations. In this tutorial, you will learn how to connect to Redis from the command line, how to authenticate and test your connection, as well as how to close a Redis connection.


##$ How To Use This Guide


This guide is written as a cheat sheet with self-contained examples. We encourage you to jump to any section that is relevant to the task you’re trying to complete.


The commands shown in this guide were tested on an Ubuntu 22.04 server running Redis version 6.0.16. To set up a similar environment, you can follow Step 1 of our guide on How To Install and Secure Redis on Ubuntu 22.04. We will demonstrate how these commands behave by running them with redis-cli, the Redis command line interface. Note that if you’re using a different Redis interface — Redli, for example — the exact output of certain commands may differ.


Alternatively, you could provision a managed Redis database instance to test these commands, but depending on the level of control allowed by your database provider, some commands in this guide may not work as described. To provision a DigitalOcean Managed Database, follow our Managed Databases product documentation. Then, you must either install Redli or set up a TLS tunnel in order to connect to the Managed Database over TLS.


# Connecting to Redis Locally


If you have redis-server installed locally, you can connect to the Redis instance with the redis-cli command:


```
redis-cli


```


This will take you into redis-cli’s interactive mode which presents you with a read-eval-print loop (REPL) where you can run Redis’s built-in commands and receive replies.


In interactive mode, your command line prompt will change to reflect your connection. In this example and others throughout this guide, the prompt indicates a connection to a Redis instance hosted locally at 127.0.0.1 and accessed over Redis’s default port 6379:


```



```


The alternative to running Redis commands in interactive mode is to run them as arguments to the redis-cli command, as in the following:


```
redis-cli redis_command


```


# Connecting to Redis Remotely


If you want to connect to a remote Redis datastore, you can specify its host and port numbers with the -h and -p flags, respectively. Also, if you’ve configured your Redis database to require a password, you can include the -a flag followed by your password to authenticate:


```
redis-cli -h host -p port_number -a password


```


If you’ve set a Redis password, clients will be able to connect to Redis even if they don’t include the -a flag in their redis-cli command. However, they won’t be able to add, change, or query data until they authenticate. To authenticate after connecting, use the auth command followed by the password:


```
auth password


```


If the password passed to auth is valid, the command will return OK. Otherwise, it will return an error.


If you’re working with a managed Redis database, your cloud provider may give you a URI that begins with redis:// or rediss:// which you can use to access your data store. If the connection string begins with redis://, you can include it as an argument to redis-cli to connect.



Note: If you have a connection string that begins with rediss://, that means your managed database requires connections over TLS/SSL. redis-cli does not support TLS connections, so you need to use a different tool that supports the rediss protocol in order to connect with the URI. For DigitalOcean Managed Databases, which require connections to be made over TLS, we recommend using Redli to access the Redis instance.

Use the following syntax to connect to a database with Redli. Note that this example includes the --tls option, which specifies that the connection should be made over TLS, and the -u flag, which declares that the following argument will be a URI connection:


```
redli --tls -u rediss://connection_URI


```


If you attempted to connect to an unavailable instance, redis-cli will go into disconnected mode and the prompt will reflect this as in the following:


```



```


Redis will attempt to reestablish the connection every time you run a command when it’s in a disconnected state.


# Testing Connections


The ping command is useful for testing whether the connection to a database is alive. Note that this is a Redis-specific command and is different from the ping networking utility. However, the two share a similar function to check a connection between two machines.


If the connection is up and no arguments are included, the ping command will return PONG:


```
ping


```


```
OutputPONG

```


If you provide an argument to the ping command, it will return that argument instead of PONG if the connection is successful:


```
ping "hello Redis!"


```


```
Output"hello Redis!"

```


If you run ping or any other command in disconnected mode, you will receive an output like the following:


```
ping


```


```
OutputCould not connect to Redis at host:port: Connection refused

```


Note that ping is also used by Redis internally to measure latency.


# Disconnecting from Redis


To disconnect from a Redis instance, use the quit command:


```
quit


```


Running exit will also exit the connection:


```
exit


```


Both quit and exit will close the connection, but only as soon as all pending replies have been written to clients.


# Conclusion


This guide details a number of commands used to establish, test, and close connections to a Redis server. If there are other related commands, arguments, or procedures you’d like to learn about in this guide, please ask or make suggestions in the comments.


For more information on Redis commands, check out our tutorial series on How to Manage a Redis Database.


