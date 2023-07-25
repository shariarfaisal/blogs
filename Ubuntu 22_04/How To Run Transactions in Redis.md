# How To Run Transactions in Redis

```Databases``` ```Ubuntu 22.04``` ```Redis``` ```NoSQL```

## Introduction


Redis is an open-source, in-memory key-value data store. Redis allows you to plan a sequence of commands and run them one after another, a procedure known as a transaction. Each transaction is treated as an uninterrupted and isolated operation, which ensures data integrity. Clients cannot run commands while a transaction block is being executed.


This tutorial discusses how to execute and cancel transactions, and also provides information on pitfalls commonly associated with transactions.


##$ How To Use This Guide


This guide is written as a cheat sheet with self-contained examples. We encourage you to jump to any section that is relevant to the task you’re trying to complete.


The commands shown in this guide were tested on an Ubuntu 22.04 server running Redis version 6.0.16. To set up a similar environment, you can follow Step 1 of our guide on How To Install and Secure Redis on Ubuntu 22.04. We will demonstrate how these commands behave by running them with redis-cli, the Redis command line interface. If you’re using a different Redis interface — Redli, for example — the exact output of certain commands may differ.


Alternatively, you could provision a managed Redis database instance to test these commands, but depending on the level of control allowed by your database provider, some commands in this guide may not work as described. To provision a DigitalOcean Managed Database, follow our Managed Databases product documentation. Then, you must either install Redli or set up a TLS tunnel in order to connect to the Managed Database over TLS.


# Running Transactions


The multi command tells Redis to begin a transaction block. Any subsequent commands will be queued up until you run an exec command, which will execute them.


The following commands form a single transaction block. The first command initiates the transaction, the second sets a key holding a string with the value of 1, the third increases the value by 1, the fourth increases its value by 40, the fifth returns the current value of the string, and the last one executes the transaction block:


```
multi
set key_MeaningOfLife 1
incr key_MeaningOfLife
incrby key_MeaningOfLife 40
get key_MeaningOfLife
exec


```


After running multi, redis-cli will respond to each of the following commands with QUEUED. After you run the exec command, it will show the output of each of those commands individually:


```
Output1) OK
2) (integer) 2
3) (integer) 42
4) "42"

```


Commands included in a transaction block are run sequentially in the order they’re queued. Redis transactions are atomic, meaning that either every command in a transaction block is processed (accepted as valid and queued to be executed) or none are. However, even if a command is successfully queued, it may still produce an error when executed. In such cases, the other commands in the transaction can still run, but Redis will skip the error-causing command. Read the section on understanding transaction errors for more details.


# Canceling Transactions


To cancel a transaction, run the discard command. This prevents any previously queued commands from running:


```
multi
set key_A 146
incrby key_A 10
discard


```


```
OutputOK

```


The discard command returns the connection to a normal state, which tells Redis to run single commands as usual. You’ll need to run multi again to tell the server you’re starting another transaction.


# Understanding Transaction Errors


Some commands are impossible to queue, such as commands with syntax errors. If you attempt to queue a syntactically incorrect command, Redis will return an error.


The following transaction creates a key named key_A and then attempts to increment it by 10. However, a spelling error in the incrby command causes an error and closes the transaction:


```
multi
set key_A 146
incrbuy key_A 10


```


```
Output(error) ERR unknown command 'incrbuy'

```


If you try to run an exec command after trying to queue a command with a syntax error like this example, you will receive another error message telling you that the transaction was discarded:


```
exec


```


```
Output(error) EXECABORT Transaction discarded because of previous errors.

```


In cases like this, you need to restart the transaction block and make sure you enter each command correctly.


Some impossible commands are possible to queue, such as running incr on a key containing only a string. Because such a command is syntactically correct, Redis won’t return an error if you try to include it in a transaction and won’t prevent you from running exec. In cases like this, all other commands in the queue will be executed, but the impossible command will return an error:


```
multi
set key_A 146
incrby key_A "ten"
exec


```


```
Output1) OK
2) (error) ERR value is not an integer or out of range

```


For more information on how Redis handles errors inside transactions, read the official documentation on the subject.


# Conclusion


This guide details a number of commands used to create, run, and cancel transactions in Redis. If there are other related commands, arguments, or procedures you’d like to learn about in this guide, please ask or make suggestions in the comments.


For more information on Redis commands, check out our tutorial series on How to Manage a Redis Database.


