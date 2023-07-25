# How To Manage Sets in Redis

```Databases``` ```Ubuntu 22.04``` ```Redis``` ```NoSQL```

## Introduction


Redis is an open-source, in-memory key-value data store. Sets in Redis are collections of strings stored at a given key. When held in a set, an individual record value is called a member. Unlike lists, sets are unordered and do not allow repeated values.


This tutorial explains how to create sets, retrieve and remove members, and compare the members of different sets.


##$ How To Use This Guide


This guide is written as a cheat sheet with self-contained examples. We encourage you to jump to any section that is relevant to the task you’re trying to complete.


The commands shown in this guide were tested on an Ubuntu 22.04 server running Redis version 6.0.16. To set up a similar environment, you can follow Step 1 of our guide on How To Install and Secure Redis on Ubuntu 22.04. We will demonstrate how these commands behave by running them with redis-cli, the Redis command line interface. If you’re using a different Redis interface — Redli, for example — the exact output of certain commands may differ.


Alternatively, you can provision a managed Redis database instance to test these commands, but note that depending on the level of control allowed by your database provider, some commands in this guide may not work as described. To provision a DigitalOcean Managed Database, follow our Managed Databases product documentation. Then, you must either install Redli or set up a TLS tunnel in order to connect to the Managed Database over TLS.


# Creating Sets


The sadd command allows you to create a set and add one or more members to it. The following example will create a set at a key named key_horror with the members "Frankenstein" and "Godzilla":


```
sadd key_horror "Frankenstein" "Godzilla"


```


If successful, sadd will return an integer showing how many members it added to the set:


```
Output(integer) 2

```


If you try adding members of a set to a key that’s already holding a non-set value, it will return an error. The first command in this block creates a list named key_action with one element, "Shaft":


```
rpush key_action "Shaft"


```


The next command tries to add a set member, "Shane", to the list, but this produces an error because of the clashing data types:


```
sadd key_action "Shane"


```


```
Output(error) WRONGTYPE Operation against a key holding the wrong kind of value

```


Note that sets don’t allow more than one occurrence of the same member:


```
sadd key_comedy "It's" "A" "Mad" "Mad" "Mad" "Mad" "Mad" "World"


```


```
Output(integer) 4

```


Even though this sadd command specifies eight members, it discards four of the duplicate "Mad" members resulting in a set size of four.


# Retrieving Members from Sets


In this section, we’ll review a number of Redis commands that return information about the members held in a set. To practice the commands outlined, run the following command, which will create a set with six members at a key called key_stooges:


```
sadd key_stooges "Moe" "Larry" "Curly" "Shemp" "Joe" "Curly Joe"


```


To return every member from a set, run the smembers command followed by the key you want to inspect:


```
smembers key_stooges


```


```
Output1) "Curly"
2) "Moe"
3) "Larry"
4) "Shemp"
5) "Curly Joe"
6) "Joe"

```


To check if a specific value is a member of a set, use the sismember command:


```
sismember key_stooges "Harpo"


```


If the element "Harpo" is a member of the key_stooges set, sismember will return 1. Otherwise, it will return 0:


```
Output(integer) 0

```


To confirm how many members are in a given set (in other words, to find the cardinality of a given set), run scard:


```
scard key_stooges


```


```
Output(integer) 6

```


To return a random element from a set, run srandmember:


```
srandmember key_stooges


```


```
Output"Larry"

```


To return multiple random, distinct elements from a set, you can follow the srandmember command with the number of elements you want to retrieve:


```
srandmember key_stooges 3


```


```
Output1) "Larry"
2) "Moe"
3) "Curly Joe"

```


If you pass a negative number to srandmember, the command is allowed to return the same element multiple times:


```
srandmember key_stooges -3


```


```
Output1) "Shemp"
2) "Curly Joe"
3) "Curly Joe"

```


The random element function used in srandmember is not perfectly random, although its performance improves in larger data sets. Read the command’s official documentation for more details.


# Removing Members from Sets


Redis comes with three commands used to remove members from a set: spop, srem, and smove.


spop randomly selects a specified number of members from a set and returns them, similar to srandmember, but then deletes them from the set. It accepts the name of the key containing a set and the number of members to remove from the set as arguments. If you don’t specify a number, spop will default to returning and removing a single value.


The following example command will remove and return two randomly-selected elements from the key_stooges set created in the previous section:


```
spop key_stooges 2


```


```
Output1) "Shemp"
2) "Larry"

```


srem allows you to remove one or more specific members from a set, rather than random ones:


```
srem key_stooges "Joe" "Curly Joe"


```


Instead of returning the members removed from the set, srem returns an integer representing how many members were removed:


```
Output(integer) 2

```


Use smove to move a member from one set to another. This command accepts as arguments the source set, the destination set, and the member to move, in that order. Note that smove only allows you to move one member at a time:


```
smove key_stooges key_jambands "Moe"


```


If the command moves the member successfully, it will return (integer) 1:


```
Output(integer) 1

```


If smove fails, it will instead return (integer) 0. Note that if the destination key does not already exist, smove will create it before moving the member into it.


# Comparing Sets


Redis also provides a number of commands that find the differences and similarities between sets. To demonstrate how these work, this section will reference three sets named presidents, kings, and beatles. If you’d like to try out the commands in this section yourself, create these sets and populate them using the following sadd commands:


```
sadd presidents "George" "John" "Thomas" "James"
sadd kings "Edward" "Henry" "John" "James" "George"
sadd beatles "John" "George" "Paul" "Ringo"


```


sinter compares different sets and returns the set intersection, or values that appear in every set:


```
sinter presidents kings beatles


```


```
Output1) "John"
2) "George"

```


sinterstore performs a similar function, but instead of returning the intersecting members, it  creates a new set at the specified destination containing these intersecting members. Note that if the destination already exists, sinterstore will overwrite its contents:


```
sinterstore new_set presidents kings beatles
smembers new_set


```


```
Output1) "John"
2) "George"

```


sdiff returns the set difference — members resulting from the difference of the first specified set from each of the following sets:


```
sdiff presidents kings beatles


```


```
Output1) "Thomas"

```


sdiff evaluates each member in the first given set and then compares those to members in each successive set. Any member in the first set that also appears in the following sets is removed, and sdiff returns the remaining members. Think of it as removing members of subsequent sets from the first set.


sdiffstore performs a function similar to sdiff, but instead of returning the set difference it creates a new set at a given destination, containing the set difference:


```
sdiffstore new_set beatles kings presidents
smembers new_set


```


```
Output1) "Paul"
2) "Ringo"

```


Like sinterstore, sdiffstore will overwrite the destination key if it already exists.


sunion returns the set union, or a set containing every member of every set you specify:


```
sunion presidents kings beatles


```


```
Output1) "Thomas"
2) "George"
3) "Paul"
4) "Henry"
5) "James"
6) "Edward"
7) "John"
8) "Ringo"

```


sunion treats the results like a new set in that it only allows one occurrence of any given member.


sunionstore performs a similar function, but creates a new set containing the set union at a given destination instead of returning the results:


```
sunionstore new_set presidents kings beatles


```


```
Output(integer) 8

```


As with sinterstore and sdiffstore, sunionstore will overwrite the destination key if it already exists.


# Conclusion


This guide details a number of commands used to create and manage sets in Redis. If there are other related commands, arguments, or procedures you’d like to learn about in this guide, please ask or make suggestions in the comments.


For more information on Redis commands, check out our tutorial series on How to Manage a Redis Database.


