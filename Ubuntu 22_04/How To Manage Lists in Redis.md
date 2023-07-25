# How To Manage Lists in Redis

```Databases``` ```Ubuntu 22.04``` ```Redis``` ```NoSQL```

## Introduction


Redis is an open-source, in-memory key-value data store. In Redis, a list is a collection of strings sorted by insertion order, similar to linked lists. This tutorial covers how to create and work with elements in Redis lists.


##$ How To Use This Guide


This guide is written as a cheat sheet with self-contained examples. We encourage you to jump to any section that is relevant to the task you’re trying to complete.


The commands shown in this guide were tested on an Ubuntu 22.04 server running Redis version 6.0.16. To set up a similar environment, you can follow Step 1 of our guide on How To Install and Secure Redis on Ubuntu 22.04. We will demonstrate how these commands behave by running them with redis-cli, the Redis command line interface. If you’re using a different Redis interface — Redli, for example — the exact output of certain commands may differ.


Alternatively, you can provision a managed Redis database instance to test these commands, but depending on the level of control allowed by your database provider, some commands in this guide may not work as described. To provision a DigitalOcean Managed Database, follow our Managed Databases product documentation. Then, you must either install Redli or set up a TLS tunnel in order to connect to the Managed Database over TLS.


# Creating Lists


A key can only hold one list, but any list can hold over four billion elements. Redis reads lists from left to right, and you can add new list elements to the head of a list (the “left” end) with the lpush command or the tail (the “right” end) with rpush. You can also use lpush or rpush to create a new list:


```
lpush key value


```


Both commands output an integer showing how many elements are in the list. To illustrate, run the following commands to create a list containing the dictum “I think therefore I am”:


```
lpush key_philosophy1 "therefore"
lpush key_philosophy1 "think"
rpush key_philosophy1 "I"
lpush key_philosophy1 "I"
rpush key_philosophy1 "am"


```


The output from the last command will read:


```
Output(integer) 5

```


Note that you can add multiple list elements with a single lpush or rpush statement:


```
rpush key_philosophy1 "-" "Rene" "Decartes"


```


The lpushx and rpushx commands are also used to add elements to lists, but will only work if the given list already exists. If either command fails, it will return (integer) 0:


```
rpushx key_philosophy2 "Happiness" "is" "the" "highest" "good" "-" "Aristotle"


```


```
Output(integer) 0

```


To change an existing element in a list, run the lset command followed by the key name, the index of the element you want to change, and the new value:


```
lset key_philosophy1 5 "sayeth"


```


If you try adding a list element to an existing key that does not contain a list, it will lead to a clash in data types and return an error. For example, the following set command creates a key holding a string, so the following attempt to add a list element to it with lpush will fail:


```
set key_philosophy3 "What is love?"
lpush key_philosophy3 "Baby don't hurt me"


```


```
Output(error) WRONGTYPE Operation against a key holding the wrong kind of value

```


It isn’t possible to convert Redis keys from one data type to another, so to turn key_philosophy3 into a list you would need to delete the key and start over with an lpush or rpush command.


# Retrieving Elements from a List


To retrieve a range of items in a list, use the lrange command followed by a start offset and a stop offset. Each offset is a zero-based index, meaning that 0 represents the first element in the list, 1 represents the next, and so on.


The following command will return all the elements from the example list created in the previous section:


```
lrange key_philosophy1 0 7


```


```
Output1) "I"
2) "think"
3) "therefore"
4) "I"
5) "am"
6) "sayeth"
7) "Rene"
8) "Decartes"

```


The offsets passed to lrange can also be negative numbers. When used in this case, -1 represents the final element in the list, -2 represents the second-to-last element in the list, and so on. The following example returns the last three elements of the list held in key_philosophy1:


```
lrange key_philosophy1 -3 -1


```


```
Output1) "I"
2) "am"
3) "sayeth"

```


To retrieve a single element from a list, you can use the lindex command. However, this command requires you to supply the element’s index as an argument. As with lrange, the index is zero-based, meaning that the first element is at index 0, the second is at index 1, and so on:


```
lindex key_philosophy1 4


```


```
Output"am"

```


To find out how many elements are in a given list, use the llen command, which is short for “list length”:


```
llen key_philosophy1


```


```
Output(integer) 8

```


If the value stored at the given key does not exist, llen will return an error.


# Removing Elements from a List


The lrem command removes the first of a defined number of occurrences that match a given value. To experiment with this, create the following list:


```
rpush key_Bond "Never" "Say" "Never" "Again" "You" "Only" "Live" "Twice" "Live" "and" "Let" "Die" "Tomorrow" "Never" "Dies"


```


The following lrem example will remove the first occurrence of the value "Live":


```
lrem key_Bond 1 "Live"


```


This command will output the number of elements removed from the list:


```
Output(integer) 1

```


The number passed to an lrem command can also be negative. The following example will remove the last two occurrences of the value "Never":


```
lrem key_Bond -2 "Never"


```


```
Output(integer) 2

```


The lpop command removes and returns the first, or “leftmost” element from a list:


```
lpop key_Bond


```


```
Output"Never"

```


Likewise, to remove and return the last or “rightmost” element from a list, use rpop:


```
rpop key_Bond


```


```
Output"Dies"

```


Redis also includes the rpoplpush command, which removes the last element from a list and pushes it to the beginning of another list:


```
rpoplpush key_Bond key_AfterToday


```


```
Output"Tomorrow"

```


If the source and destination keys passed to rpoplpush command are the same, it will essentially rotate the elements in the list.


# Conclusion


This guide details a number of commands that you can use to create and manage lists in Redis. If there are other related commands, arguments, or procedures you’d like to learn about in this guide, please ask or make suggestions in the comments.


For more information on Redis commands, check out our tutorial series on How to Manage a Redis Database.


