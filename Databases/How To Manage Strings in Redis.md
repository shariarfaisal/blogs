# How To Manage Strings in Redis

```Databases``` ```Ubuntu 22.04``` ```Redis``` ```NoSQL```

## Introduction


Redis is an open-source, in-memory key-value data store. In Redis, strings are the most basic type of value you can create and manage. This tutorial provides an overview of how to create and retrieve strings and how to manipulate the values held by string keys.


##$ How To Use This Guide


This guide is written as a cheat sheet with self-contained examples. We encourage you to jump to any section that is relevant to the task you’re trying to complete.


The commands shown in this guide were tested on an Ubuntu 22.04 server running Redis version 6.0.16. To set up a similar environment, you can follow Step 1 of our guide on How To Install and Secure Redis on Ubuntu 22.04. We will demonstrate how these commands behave by running them with redis-cli, the Redis command line interface. If you’re using a different Redis interface — Redli, for example — the exact output of certain commands may differ.


Alternatively, you can provision a managed Redis database instance to test these commands, but note that depending on the level of control allowed by your database provider, some commands in this guide may not work as described. To provision a DigitalOcean Managed Database, follow our Managed Databases product documentation. Then, you must either install Redli or set up a TLS tunnel in order to connect to the Managed Database over TLS.


# Creating and Managing Strings


Keys that hold strings can only hold one value. You cannot store more than one string in a single key. However, strings in Redis are binary-safe, meaning a Redis string can hold any kind of data, from alphanumeric characters to JPEG images. The only limit is that strings must be 512 MB long or less.


To create a string, use the set command. For example, the following set command creates a key named key_Welcome1 that holds the string "Howdy":


```
set key_Welcome1 "Howdy"


```


```
OutputOK

```


To set multiple strings in one command, use mset:


```
mset key_Welcome2 "there" key_Welcome3 "partners,"


```


You can also use the append command to create strings:


```
append key_Welcome4 "welcome to Texas"


```


If the string was created successfully, append will output an integer equal to how many characters the string includes:


```
Output(integer) 16

```


Note that append can also be used to change the contents of strings. Read the section on manipulating strings for details on this.


# Retrieving Strings


To retrieve a string, use the get command:


```
get key_Welcome1


```


```
Output"Howdy"

```


To retrieve multiple strings with one command, use mget:


```
mget key_Welcome1 key_Welcome2 key_Welcome3 key_Welcome4


```


```
Output1) "Howdy"
2) "there"
3) "partners,"
4) "welcome to Texas"

```


For every key passed to mget that doesn’t hold a string value or doesn’t exist at all, the command will return nil.


# Manipulating Strings


If a string is made up of an integer, you can run the incr command to increase it by one:


```
set key_1 3
incr key_1


```


```
Output(integer) 4

```


Similarly, you can use the incrby command to increase a numeric string’s value by a specific increment:


```
incrby key_1 16


```


```
Output(integer) 20

```


The decr and decrby commands work the same way, but they decrease the integer stored in a numeric string:


```
decr key_1


```


```
Output(integer) 19

```


```
decrby key_1 16


```


```
Output(integer) 3

```


If an alphabetic string already exists, append will append the value onto the end of the existing value and return the new length of the string. To illustrate, the following command appends ", y'all" to the string held by the key key_Welcome4, so now the string will read "welcome to Texas, y'all":


```
append key_Welcome4 ", y'all"


```


```
Output(integer) 23

```


You can also append integers to a string holding a numeric value. The following example appends 45 to 3, the integer held in key_1, so it will then hold 345. In this case, append will also return the new length of the string rather than its new value:


```
append key_1 45


```


```
Output(integer) 3

```


Because this key still only holds a numeric value, you can perform the incr and decr operations on it. You can also append alphabetic characters to an integer string, but if you do this then running incr and decr on the string will produce an error as the string value is no longer an integer.


# Conclusion


This guide details a number of commands used to create and manage strings in Redis. If there are other related commands, arguments, or procedures you’d like to learn about in this guide, please ask or make suggestions in the comments.


For more information on Redis commands, check out our tutorial series on How to Manage a Redis Database.


