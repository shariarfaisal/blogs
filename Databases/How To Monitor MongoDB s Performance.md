# How To Monitor MongoDB s Performance

```Databases``` ```MongoDB``` ```Monitoring``` ```NoSQL```

The author selected the Open Internet/Free Speech Fund to receive a donation as part of the Write for DOnations program.


## Introduction


Monitoring is a key part of database administration, as it allows you to understand your database’s performance and overall health. By monitoring your database’s performance you can have a better sense of its current capacity, observe how its workload changes over time, and plan ahead to scale the database once it starts approaching its limits. It can also help you notice underlying hardware problems or abnormal behavior like an unexpected spike in database use. Lastly, monitoring can help to diagnose issues with applications using the database, like application queries that cause bottlenecks.


MongoDB comes installed with a variety of tools and utilities you can use to observe your database’s performance. In this tutorial, you’ll learn how to monitor database metrics on-demand using built-in commands and tools. You’ll also become familiar with MongoDB’s database profiler which can help you detect poorly optimized queries.


# Prerequisites


To follow this tutorial, you will need:


- A server with a regular, non-root user with sudo privileges and a firewall configured with UFW. This tutorial was validated using a server running Ubuntu 20.04, and you can prepare your server by following this initial server setup tutorial for Ubuntu 20.04.
- MongoDB installed on your server. To set this up, follow our tutorial on How to Install MongoDB on Ubuntu 20.04.
- Your server’s MongoDB instance secured by enabling authentication and creating an administrative user. To secure MongoDB like this, follow our tutorial on How To Secure MongoDB on Ubuntu 20.04.
- Familiarity with querying MongoDB collections and filtering results. To learn how to use MongoDB queries, follow our guide on How To Create Queries in MongoDB.


Note: The linked tutorials on how to configure your server, install MongoDB, and secure the MongoDB installation refer to Ubuntu 20.04. This tutorial concentrates on MongoDB itself, not the underlying operating system. It will generally work with any MongoDB installation regardless of the operating system as long as authentication has been enabled.

# Step 1 — Preparing the Test Data


In order to explain how you can monitor MongoDB’s performance, this step outlines how to open the MongoDB shell to connect to your locally-installed MongoDB instance and create a sample collection within it.


To create the sample collection used in this guide, connect to the MongoDB shell as your administrative user. This tutorial follows the conventions of the prerequisite MongoDB security tutorial and assumes the name of this administrative user is AdminSammy and its authentication database is admin. Be sure to change these details in the following command to reflect your own setup, if different:


```
mongo -u AdminSammy -p --authenticationDatabase admin


```


Enter the password set during installation to gain access to the shell. After providing the password, you’ll see the > prompt sign.



Note: On a fresh connection, the MongoDB shell will connect to the test database by default. You can safely use this database to experiment with MongoDB and the MongoDB shell.
Alternatively, you could switch to another database to run all of the example commands given in this tutorial. To switch to another database, run the use command followed by the name of your database:
use database_name



Database monitoring isn’t very practical or useful when working with a small data set, since the database system will only need to scan a few records for any given query. To illustrate MongoDB’s performance monitoring features, you’ll need a database with enough data that it will take MongoDB a significant length of time to execute queries.


To this end, the examples throughout this guide refer to a sample collection named accounts containing a large number of documents. Each document represents an individual bank account with a randomly-generated account balance. Each document in the collection will have a structure like this:


An example bank account document
```
{
    "number": "1000-987321",
    "currency": "USD",
    "balance": 431233431
}

```


This example document contains the following information:


- number: This field represents the account number for the given account. In this collection, each account number will have a prefix of 1000- followed by an incrementing numerical identifier.
- currency: This field indicates what kind of currency each account’s balance is stored in. Each account’s currency value will either be USD or EUR.
- balance: This shows the balance for each given bank account. In this sample database, each document’s balance field will have a randomly-generated value.

Rather than manually inserting a large number of documents, you can execute the following JavaScript code to simultaneously create a collection named accounts and insert a million such documents into it:


```
for (let i = 1; i <= 1000000; ++i) {
    db.accounts.insertOne({
        "number": "1000-" + i,
        "currency": i > 500000 ? "USD" : "EUR",
        "balance": Math.random() * 100000
    })
}


```


This code executes a for loop that runs one million times in a row. Each time the loop iterates, it executes an insertOne() method on the accounts collection to insert a new document. In each iteration, the method gives a value to the number field made up of the 1000- prefix with the value held in the i value for that iteration. This means that the first time this loop iterates, the number field value will be set to 1000-1; the last time it iterates, it will be set to 1000-1000000.


The currency is always represented as USD for accounts with numbers higher than 500000 and as EUR for accounts with numbers lower than that. The balance field uses the Math.random() function to generate a random number between 0 and 1, and then multiplies the random number by 100000 to provide larger values.



Note: Executing this loop can take a long while, even beyond 10 minutes. It is safe to leave the operation running until it finishes.

The output will inform you of the success and return the ObjectId of the last document that was inserted:


```
Output{
        "acknowledged" : true,
        "insertedId" : ObjectId("61a38a4beedf737ac8e54e82")
}

```


You can verify that the documents were properly inserted by running the count() method with no arguments, which will retrieve the count of documents in the collection:


```
db.accounts.count()


```


```
Output1000000

```


In this step, you have successfully created the list of example documents that will serve as the test data used in this guide to explain the tools MongoDB provides for performance monitoring. In the next step, you’ll learn how to check the basic server usage statistics.


# Step 2 — Checking Server Usage Statistics


MongoDB automatically tracks a number of useful performance statistics, and checking on these regularly is a fundamental way to monitor your database. Be aware that these statistics won’t offer real-time insight into what’s happening with your database, but they can be useful for determining how the database performs and whether there are any imminent problems.



Warning: The MongoDB monitoring commands outlined in this guide return potentially sensitive information about your database and its performance. Because of this, some of these commands require advanced permissions.
Specifically, the serverStatus() method outlined in this step as well as the mongostat and mongotop commands highlighted in the next step all require users to have been granted the clusterMonitor role in order to run them. Likewise, the setProfilingLevel() method outlined in Step 4 requires the dbAdmin role.
Assuming you followed the prerequisite tutorial on How To Secure MongoDB on Ubuntu 20.04 and are connected to your MongoDB instance as the administrative user you created in that guide, you will need to grant it these additional roles to follow along with the examples in this guide.
First, switch to your user’s authentication database. This is admin in the following example, but connect to your own user’s authentication database if different:
use admin


Outputswitched to db admin

Then run a grantRolesToUser() method and grant your user the clusterMonitor role along with the dbAdmin role over the database where you created the accounts collection. The following example assumes the accounts collection is in the test database:
db.grantRolesToUser(
"AdminSammy",
[
"clusterMonitor",
{ role : "dbAdmin", db : "test" }
]
)


Please note that it’s generally considered more secure to have user profiles dedicated to specific purposes. This way, no user will have unnecessarily broad privileges. If you are working in a production environment, you may want to have a dedicated user whose sole purpose is to monitor the database.
The following example creates a MongoDB user named MonitorSammy and grants them the roles needed for you to follow along with the examples in this tutorial. Note that it also includes the readWriteAnyDatabase role, which will allow this user to read and write data to any database in the cluster:
db.createUser(
{
user: "MonitorSammy",
pwd: passwordPrompt(),
roles: [ { role : "dbAdmin", db : "test" }, "clusterMonitor", "readWriteAnyDatabase" ]
}
)


After granting your user the appropriate roles, navigate back to the database where your accounts collection is stored:
use test


Outputswitched to db test


Begin by checking the overall database statistics by executing the stats() method:


```
db.stats(1024*1024)


```


This method’s argument (1024*1024) is the scale factor and tells MongoDB to return storage information in megabytes. If you omit this, the values will all be presented in bytes.


The stats() method returns a short, concise output with some important statistics relating to the current database:


```
Output{
        "db" : "test",
        "collections" : 3,
        "views" : 0,
        "objects" : 1000017,
        "avgObjSize" : 80.8896048767171,
        "dataSize" : 77.14365005493164,
        "storageSize" : 24.109375,
        "indexes" : 4,
        "indexSize" : 9.9765625,
        "totalSize" : 34.0859375,
        "scaleFactor" : 1048576,
        "fsUsedSize" : 4238.12109375,
        "fsTotalSize" : 24635.703125,
        "ok" : 1
}

```


This output provides an overview of the data that this MongoDB instance is storing. The following keys returned in this output can be particularly useful:


- The objects key shows the total number of documents in the database. You can use this to assess the size of the database and, when observed over time, its growth.
- avgObjectSize shows the average size of these documents, giving insight into whether the database is operating on large and complex documents or small ones. This value is always shown in bytes, regardless of whether you specify a scale factor.
- The collections and indexes keys tell how many collections and indexes are currently defined in the database.
- The totalSize key indicates how much storage the database takes up on disk.

This information returned by the stats() method can help give you an idea of how much data is currently stored on your database, but it doesn’t provide insight into its performance or existing problems. For that, the much more verbose serverStatus() method comes in handy:


```
db.serverStatus()


```


The output of this method is lengthy and provides a great deal of information about server usage:


```
Output{
        "host" : "ubuntu-mongo-rs",
        "version" : "4.4.6",
        "process" : "mongod",
        "pid" : NumberLong(658997),
        "uptime" : 976,
        . . .
        "ok" : 1
}

```


While all of this information could potentially be useful, this guide will focus on three sections in particular. First, find the connections section of this output:


```
Output        . . .
        "connections" : {
                "current" : 4,
                "available" : 51196,
                "totalCreated" : 4,
                "active" : 2,
                "exhaustIsMaster" : 1,
                "exhaustHello" : 0,
                "awaitingTopologyChanges" : 1
        },
        . . .

```


Each database server can support only so many connections at a time. The current key shows the number of clients currently connected to the database, whereas available is the number of remaining unused connections that the database has available. The totalCreated value holds the number of connections used from the server startup.


Most applications are designed to reuse existing connections and don’t open multiple connections often. Thus, a high number of connections, if not anticipated, can be an alarming sign of a misconfiguration of how clients are accessing the server.


If the high number of connections is anticipated by the character of workloads performed, you might consider adding one or more shards to a sharded cluster to distribute traffic across multiple MongoDB instances.


Next, find the globalLock section of the output. This section is relates to global locks across the whole database server:


```
Output        . . .
        "globalLock" : {
                "totalTime" : NumberLong(975312000),
                "currentQueue" : {
                        "total" : 0,
                        "readers" : 0,
                        "writers" : 0
                },
                "activeClients" : {
                        "total" : 0,
                        "readers" : 0,
                        "writers" : 0
                }
        },

```


MongoDB uses locking to ensure data consistency when performing multiple operations, guaranteeing no two queries will modify the same data at the same time. On heavily used servers, there is a chance that locking could result in bottlenecks, with one or more queries waiting for the locks to be released before they can be executed.


The currentQueue.total value shows the number of queries waiting on the locks to be released so they can be executed. If this value is high, it means the database’s performance is being affected and queries will take longer to complete.


This often stems from many long-running queries holding the locks and may be indicative of an ineffective use of indexes or poorly designed queries, among other possibilities.


Lastly, find the opcounters section:


```
Output        "opcounters" : {
                "insert" : NumberLong(10000007),
                "query" : NumberLong(6),
                "update" : NumberLong(6),
                "delete" : NumberLong(0),
                "getmore" : NumberLong(0),
                "command" : NumberLong(1298)
        },

```


This section of the serverStatus() output can help you get an idea of whether the database server is used mostly for reads or writes, or whether its use is well-balanced. In this example, after inserting the test documents, the counter for insert operations is much higher than for query operations. In a real life scenario, these values would likely be different.


Write-heavy databases can benefit from being scaled horizontally through sharding. Similarly, read-heavy MongoDB databases will usually benefit from replication.


These statistics can give an overall idea of how the server is used and whether there are performance issues such as long locking queues at the moment of accessing them. However, they don’t give any real-time information on how the server is used. For that, mongostat and mongotop commands are useful tools.


# Step 3 — Using mongostat and mongotop to Obtain Real-Time Database Statistics


While the commands used to access MongoDB’s server statistics can provide insights into how the server is used in retrospect, they can’t provide real-time information on which collections are being most actively used at the moment or what kind of queries are being executed.


MongoDB provides two useful system tools for real-time monitoring that analyze the database activity and continually refresh the information they provide: mongostat and mongotop. mongostat provides a brief overview of the MongoDB instance’s current status, while mongotop tracks how much time the instance spends on read and write operations. Both of these tools are run from the command line, instead of the MongoDB shell.


To use mongostat, keep your current MongoDB shell connection, and open another terminal window to access your server shell. In the second server shell, run the mongostat command:


```
mongostat -u AdminSammy --authenticationDatabase admin


```


As mentioned previously, mongostat requires advanced privileges. If you’ve enabled authentication on your MongoDB instance and set up a user with the appropriate roles, then you will have to authenticate as that user by providing their username and authentication database (as shown in this example) and then entering their password when prompted.


In a default configuration, mongostat prints the counters of currently executed queries in one second intervals:


```
Outputinsert query update delete getmore command dirty  used flushes vsize  res qrw arw net_in net_out conn                time
    *0    *0     *0     *0       0     1|0  0.0% 38.7%       0 1.54G 210M 0|0 1|0   223b   84.4k    7 Nov 28 15:40:40.621
    *0    *0     *0     *0       0     2|0  0.0% 38.7%       0 1.54G 210M 0|0 1|0   224b   84.8k    7 Nov 28 15:40:41.619
    *0    *0     *0     *0       0     1|0  0.0% 38.7%       0 1.54G 210M 0|0 1|0   223b   84.5k    7 Nov 28 15:40:42.621
    *0    *0     *0     *0       0     3|0  0.0% 38.7%       0 1.54G 210M 0|0 1|0   365b   85.0k    7 Nov 28 15:40:43.619

```


If the mongostat output shows a value of 0 for a given query type, it indicates that the database isn’t running any operations of that type. This example output shows 0 for each query type, meaning that there currently aren’t any queries actively running.


You should still have your first terminal window open and connected to your MongoDB shell. Insert some more test documents into the accounts collection and check if mongostat will notice the activity:


```
for (let i = 1; i <= 10000; ++i) {
    db.accounts.insertOne({
        "number": "2000-" + i,
        "currency": "USD",
        "balance": Math.random() * 100000
    })
}


```


This is a for-loop similar to the one you ran in Step 1. This time, though, the loop inserts only 10000 entries. The account numbers are prefixed with 2000, and the currency is always USD.


While the new documents are being inserted, check the mongostat output:


```
Output. . .
    *0    *0     *0     *0       0     1|0  0.0% 38.7%       0 1.54G 210M 0|0 1|0   112b   42.5k    4 Nov 28 15:50:33.294
    *0    *0     *0     *0       0     0|0  0.0% 38.7%       0 1.54G 210M 0|0 1|0   111b   42.2k    4 Nov 28 15:50:34.295
   755    *0     *0     *0       0     1|0  0.1% 38.8%       0 1.54G 210M 0|0 1|0   154k   79.4k    4 Nov 28 15:50:35.294
  2853    *0     *0     *0       0     0|0  0.4% 39.1%       0 1.54G 211M 0|0 1|0   585k    182k    4 Nov 28 15:50:36.295
  2791    *0     *0     *0       0     1|0  0.7% 39.4%       0 1.54G 212M 0|0 1|0   572k    179k    4 Nov 28 15:50:37.293
  2849    *0     *0     *0       0     0|0  1.0% 39.7%       0 1.54G 213M 0|0 1|0   584k    182k    4 Nov 28 15:50:38.296
   745    *0     *0     *0       0     2|0  1.1% 39.8%       0 1.54G 213M 0|0 1|0   153k   79.2k    4 Nov 28 15:50:39.294
    *0    *0     *0     *0       0     0|0  1.1% 39.8%       0 1.54G 213M 0|0 1|0   111b   42.2k    4 Nov 28 15:50:40.295
    *0    *0     *0     *0       0     2|0  1.1% 39.8%       0 1.54G 213M 0|0 1|0   167b   42.7k    4 Nov 28 15:50:41.293
. . .

```


While the query runs, the new lines returned by mongostat begin to show values other than 0. In the insert column showing the number of queries that are inserting new data to the database, the values were higher for several seconds. Since the mongostat shows data in one-second intervals, you can find not only the proportion of inserts relative to other kinds of database operations, but also how fast the database inserts the new data. In this example, the server achieved almost 3000 insertions per second.


You can use mongostat to monitor the current workload of the database server, grouped by query types. The second tool that MongoDB ships with — mongotop — shows the database server activity grouped by collections.


Stop mongostat from runing in your second terminal window by pressing CTRL + C. Then run mongotop in that same terminal. Again, if you have authentication enabled, you will need to authenticate as an appropriately-privileged user:


```
mongotop -u AdminSammy --authenticationDatabase admin


```


mongotop outputs a list of all the collections in the database, accompanied by the time spent on reads, writes and in total within the time window. Similarily to mongostat, the output is refreshed every second:


```
Output2021-11-28T15:54:42.290+0000    connected to: mongodb://localhost/

                    ns    total    read    write    2021-11-28T15:54:43Z
    admin.system.roles      0ms     0ms      0ms
  admin.system.version      0ms     0ms      0ms
config.system.sessions      0ms     0ms      0ms
   config.transactions      0ms     0ms      0ms
  local.system.replset      0ms     0ms      0ms
         test.accounts      0ms     0ms      0ms

. . .

```


Try inserting some more documents into the database to see if the activity registers in mongotop. In the MongoDB shell, execute the following for loop; after doing so, observe the terminal window with mongotop running:


```
for (let i = 1; i <= 10000; ++i) {
    db.accounts.insertOne({
        "number": "3000-" + i,
        "currency": "USD",
        "balance": Math.random() * 100000
    })
}


```


This time, the activity will be visible in the mongotop statistics:


```
Output. . .
                    ns    total    read    write    2021-11-28T15:57:27Z
         test.accounts    127ms     0ms    127ms
  admin.$cmd.aggregate      0ms     0ms      0ms
    admin.system.roles      0ms     0ms      0ms
  admin.system.version      0ms     0ms      0ms
config.system.sessions      0ms     0ms      0ms
   config.transactions      0ms     0ms      0ms
  local.system.replset      0ms     0ms      0ms

                    ns    total    read    write    2021-11-28T15:57:28Z
         test.accounts    130ms     0ms    130ms
  admin.$cmd.aggregate      0ms     0ms      0ms
    admin.system.roles      0ms     0ms      0ms
  admin.system.version      0ms     0ms      0ms
config.system.sessions      0ms     0ms      0ms
   config.transactions      0ms     0ms      0ms
  local.system.replset      0ms     0ms      0ms
. . .

```


Here, mongotop shows that all the database activity happened in accounts collection in the test database and that all operations in the time window have been write operations. All of this should align with the for loop operation you executed.


As with mongostat, you can stop mongotop from running by pressing CTRL + C.


When observed during peak load, you can use mongotop to monitor how the database activity spreads across different collections to help you better understand your schema and plan for scaling. It also provides insight into whether a collection usage is more read or write-heavy.


# Step 4 — Using MongoDB’s Database Profiler to Identify Slow Queries


Database performance bottlenecks can come from many sources. Although scaling the database (either horizontally or vertically) is often the solution to performance bottlenecks, their cause may not actually be the limits of the database but issues with schema or query design.


If queries run for too long, the cause might be an ineffective use of indexes or errors in the query itself. Long-running queries often go under the radar during application development, typically because the test datasets are too small or conditions are different than in production.


You could potentially find the culprit by manually executing test queries and checking which ones underperform, though this would be very tedious. Fortunately, MongoDB’s database profiler tool can do that automatically.


MongoDB’s database profiler can log queries and statistics about their execution when they match certain conditions. The most important of these conditions is the query’s execution time: if a query takes longer than a specified amount time to execute, the profiler will automatically flag that query as problematic. Using the profiler, you can identify which queries perform poorly and then focus on fixing those particular issues.


Before using the profiler, execute the following query. This query will retrieve one of the accounts you inserted, though it is not as simple as it may seem at first glance:


```
db.accounts.find({"number": "1000-20"})


```


The command will retrieve the exact account you requested:


```
Output{ "_id" : ObjectId("61a38fd5eedf737ac8e54e96"), "number" : "1000-20", "currency" : "EUR", "balance" : 24101.14770458518 }

```


You may have noticed that the query was not executed immediately, and it took MongoDB a moment or two to find the account. In a real-world application, there could be many kinds of queries that perform poorly, and you might not notice their underperformance in practice.


You can configure MongoDB to help you pinpoint which queries take longer than expected. To do so, first enable the profiler by executing the following command:


```
db.setProfilingLevel(1, { slowms: 100 })


```


The setProfilingLevel() method takes two arguments. First is the profiling level, which can be either 0, 1, or 2:


- 0 disables the profiler
- 1 enables the profiler only on slow queries satisfying the condition
- 2 enables the profiler for all queries

In this example, the profiler will analyze queries that run for longer than 100 milliseconds, as defined by the second argument, { slowms: 100 }.



Note: Using the profiler degrades the performance, as MongoDB must now analyze queries in addition to executing them. It should be used sparingly when monitoring performance bottlenecks.
It is possible to further tailor the subset of queries the profiler will log at by configuring it to profile only a certain percentage of queries or filtering by query type. To learn more about how you can have greater control over the profiler, refer to the official documentation on the subject.

This method will return a success message:


```
Output{ "was" : 0, "slowms" : 100, "sampleRate" : 1, "ok" : 1 }

```


From now on, the database profiling will be enabled and MongoDB will actively monitor every query you execute to find any that take more than 100 milliseconds to complete.


Try this out by executing a few different queries. First, use the count command to find the number of documents in the accounts collection:


```
db.accounts.count()


```


This command will quickly return the number of documents in the collection:


```
Output1020000

```


Then, try looking up the first three bank accounts appearing in the collection:


```
db.accounts.find().limit(3)


```


Again, the database will return the results quickly:


```
Output{ "_id" : ObjectId("61ef40640f2ba52efc56ee17"), "number" : "1000-1", "currency" : "EUR", "balance" : 25393.132960293842 }
{ "_id" : ObjectId("61ef40640f2ba52efc56ee18"), "number" : "1000-2", "currency" : "EUR", "balance" : 63629.42056192393 }
{ "_id" : ObjectId("61ef40640f2ba52efc56ee19"), "number" : "1000-3", "currency" : "EUR", "balance" : 75602.12331602155 }

```


Finally, run the search query for the particular bank account once again:


```
db.accounts.find({"number": "1000-20"})


```


This query will return the result but, like before, it will take a moment or two longer than the previous operations:


```
Output{ "_id" : ObjectId("61a38fd5eedf737ac8e54e96"), "number" : "1000-20", "currency" : "EUR", "balance" : 24101.14770458518 }

```


The profiler doesn’t produce any output of its own even though the query was visibly slower. Instead, the details about slow operations are registered in a special collection within the database named system.profile. This collection is a capped collection that never exceeds 1 MB in size. That means it will always contain a list of only the most recent slow queries.


To retrieve information on queries identified by the profiler, you have to query the system.profile collection in a manner like this:


```
db.system.profile.find().sort({ "ts" : -1 }).pretty()


```


This query uses the find() method, as usual. It also includes a sort clause that contains { "ts" : -1 } as an argument. This will sort the result set with the latest queries first. Lastly, the pretty() method at the end will display the output in a more readable format.


Each slow query is represented as a regular document, and system.profile is like any regular collection. This means you can filter the results, sort them and even use them in aggregation pipelines to further narrow down or analyze the list of queries identified by the profiler.


Notice that the result consists only of a single document. The two other queries were executed fast enough not to trigger the profiler:


```
Output{
        "op" : "query",
        "ns" : "test.accounts",
        "command" : {
                "find" : "accounts",
                "filter" : {
                        "number" : "1000-20"
                },
                . . .
        },
        "nreturned" : 1,
        "keysExamined" : 0,
        "docsExamined" : 1030000,
        . . .
        "millis" : 434,
        "planSummary" : "COLLSCAN",
        . . .
}

```


This output provides a number of details about the slow query’s execution:


- The op key shows what kind of operation this information represents. Here, it’s a query, since it represents an operation in which you used the find() to retrieve data from the database.
- The ns key indicates which database and collection were involved in the operation. As the output shows, this operation queried the accounts collection in the test database.
- The command key provides further information about the query itself. In this case, the filter sub-key contains the entire filter document. Using the information coming from op and command fields, you can reconstruct the query in question.
- In the millis field, you will find the exact time it took to complete the query. In this example, almost half a second.
- The docsExamined field provides the number of documents scanned to return the result set.
- nreturned represents number of documents the query returned. In this example, only a single document was returned out of over a million scanned.
- The planSummary shows the method MongoDB used to execute the query. COLLSCAN corresponds to a full collection scan, meaning that it browsed every document in the collection one by one to find the matching bank account.

All together, this information underscores the need for an index that could help MongoDB execute this query faster. The database had to look over the whole collection to find a single document, as indicated by the big difference between the number of examined and returned documents, as well as the execution strategy.


In this particular example, creating an index to support queries that filter data based on the number field would provide an immediate boost to the performance of these kinds of queries. In real scenarios, the solutions to slow queries may differ and depend on the exact query that’s causing issues.


To finish the profiling session, you can disable the profiler by setting the profiling level to zero:


```
db.setProfilingLevel(0)


```


The operation will succeed with a confirmation message:


```
Output{ "was" : 1, "slowms" : 100, "sampleRate" : 1, "ok" : 1 }

```


Now the database returns to normal operation with no profiling happening behind the scenes.


Whenever you suspect that slow queries may be having a negative impact on your database’s performance, you can use the database profiler to find them and better understand their structure and how they are executed. With this information, you will be better equipped to adjust them and improve their performance.


# Conclusion


By following this guide, you learned how to find MongoDB’s server statistics and how to use diagnostic tools like mongotop, mongostat, as well as MongoDB’s database profiler mechanism. You can use these to get a better sense of your database’s workload, determine which collections are the most active, and whether the server predominantly performs writes or reads. You can also identify  slow queries that are impacting MongoDB’s performance in order to replace them with more efficient ones.


These are just a selection of tools and techniques you can use to monitor the health and performance of your MongoDB installation and act on it. Each of these tools can be further configured and customized to provide you with more targeted insight into the server performance. We encourage you to study the official MongoDB documentation to learn more about techniques you can use to monitor server performance and act on it.


