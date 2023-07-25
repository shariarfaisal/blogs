# How To Change Redis s Configuration from the Command Line

```Databases``` ```Ubuntu 22.04``` ```Redis``` ```NoSQL``` ```Configuration Management```

# Introduction


Redis is an open-source, in-memory key-value data store. Redis has several commands that allow you to make changes to the Redis server’s configuration settings on the fly. This tutorial will discuss some of these commands, and also explain how to make these configuration changes permanent.


##$ How To Use This Guide


This guide is written as a cheat sheet with self-contained examples. We encourage you to jump to any section that is relevant to the task you’re trying to complete.


The commands shown in this guide were tested on an Ubuntu 22.04 server running Redis version 6.0.16. To set up a similar environment, you can follow Step 1 of our guide on How To Install and Secure Redis on Ubuntu 22.04. We will demonstrate how these commands behave by running them with redis-cli, the Redis command line interface. If you’re using a different Redis interface — Redli, for example — the exact output of certain commands may differ.


Alternatively, you could provision a managed Redis database instance to test these commands, but depending on the level of control allowed by your database provider, some commands in this guide may not work as described. To provision a DigitalOcean Managed Database, follow our Managed Databases product documentation. Then, you must either install Redli or set up a TLS tunnel in order to connect to the Managed Database over TLS.


Be aware that managed Redis databases typically do not allow users to alter the configuration file. If you’re working with a Managed Database from DigitalOcean, the commands outlined in this guide will result in errors.


# Changing Redis’s Configuration


The commands outlined in this section will only alter the Redis server’s behavior for the duration of the current session, or until you run config rewrite, which will make them permanent. You can alter the Redis configuration file directly by opening and editing it with your preferred text editor. For example, you can use nano to do so:


```
sudo nano /etc/redis/redis.conf


```



Warning: The config set command is considered dangerous. By changing your Redis configuration file, it’s possible that you will cause your Redis server to behave in unexpected or undesirable ways. We recommend that you only run the config set command if you are testing out its behavior or you’re absolutely certain that you want to make changes to your Redis configuration.
It may be in your interest to rename this command to something with a lower likelihood of being run accidentally.

config set allows you to reconfigure Redis at runtime without having to restart the service. It uses the following syntax:


```
config set parameter value


```


For example, if you wanted to change the name of the database dump file Redis will produce after you run a save command, you might run a command like the following:


```
config set "dbfilename" "new_file.rdb"


```


If the configuration change is valid, the command will return OK. Otherwise, it will return an error.



Note: Not every parameter in the redis.conf file can be changed with a config set operation. For example, you cannot change the authentication password defined by the requirepass parameter.

# Making Configuration Changes Permanent


config set does not permanently alter the Redis instance’s configuration file; it only changes Redis’s behavior at runtime. To edit redis.conf after running a config-set command and make the current session’s configuration permanent, run config rewrite:


```
config rewrite


```


This command prioritizes preserving the comments and overall structure of the original redis.conf file, with only minimal changes to match the settings currently used by the server.


Like config set, if the rewrite is successful, config rewrite will return OK.


# Checking Redis’s Configuration


To read the current configuration parameters of a Redis server, run the config get command. config get takes a single argument, which can be either an exact match of a parameter used in redis.conf or a glob pattern. For example:


```
config get repl*


```


Depending on your Redis configuration, this command will return something similar to the following:


```
Output 1) "repl-ping-slave-period"
 2) "10"
 3) "repl-timeout"
 4) "60"
 5) "repl-backlog-size"
 6) "1048576"
 7) "repl-backlog-ttl"
 8) "3600"
 9) "repl-diskless-sync-delay"
10) "5"
11) "repl-disable-tcp-nodelay"
12) "no"
13) "repl-diskless-sync"
14) "no"

```


You can also return all of the configuration parameters supported by config set by running config get *.


# Conclusion


This guide details the redis-cli commands used to make changes to a Redis server’s configuration file. If there are other related commands, arguments, or procedures you’d like to learn about in this guide, please ask or make suggestions in the comments.


For more information on Redis commands, check out our tutorial series on How to Manage a Redis Database.


