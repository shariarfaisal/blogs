# How To Use Transactions in MongoDB

```Databases``` ```MongoDB``` ```NoSQL```

The author selected the Open Internet/Free Speech Fund to receive a donation as part of the Write for DOnations program.


## Introduction


A transaction is a sequence of database operations that will only succeed if every operation within the transaction has been executed correctly. Transactions have been an important feature of relational databases for many years, but have been mostly absent from document-oriented databases until recently. The nature of document-oriented databases — where a single document can be a robust, nested structure, containing embedded documents and arrays rather instead of only simple values — streamlines storing related data within a single document. As such, modifying multiple documents as part of a single logical operation is often unnecessary, limiting the need for transactions in many applications.


There are, however, applications for which accessing and modifying multiple documents in a single operation with guaranteed integrity is required even with document-oriented databases. MongoDB introduced multi-document ACID transactions in version 4.0 of the database engine in order to meet the needs of such use cases. In this article, you’ll explore what transactions are, the ACID properties of a transaction, and how to use transactions in MongoDB.


# Prerequisites


Because of the way they’re implemented in MongoDB, transactions can only be performed on MongoDB instances that are running as part of a larger cluster. This could either be a sharded database cluster or a replica set. If you have an existing sharded MongoDB cluster or replica set running that you can use for testing purposes, then you can go on to the next section to learn about ACID transactions.


However, setting up a proper, functional replica set or a sharded MongoDB cluster requires you to have at least three running MongoDB instances, ideally running on separate servers. Additionally, this guide’s examples all involve working with a single node running as a member of a replica set. Rather than having you go through the work of provisioning multiple servers and configuring MongoDB on each of them only for you to use just one of them in this guide, you can convert a standalone MongoDB instance into a single-node replica set that you can use to practice running transactions.


This guide’s first step outlines how to do this, so in order to complete this tutorial, you will only need the following:


- One server with a regular, non-root user with sudo privileges and a firewall configured with UFW. This tutorial was validated using a server running Ubuntu 20.04, and you can prepare your server by following this initial server setup tutorial for Ubuntu 20.04.
- MongoDB installed on your server. To set this up, follow our tutorial on How to Install MongoDB on Ubuntu 20.04.

# Understanding ACID Transactions


A transaction is a set of database operations (such as reads and writes) that are executed in a sequential order and in an all-or-nothing manner. This means that for the results of running these operations to be saved within the database and visible outside the transaction to other database clients, all of the individual operations must succeed. If any of the operations fail to execute correctly, the transaction aborts and every change made from the beginning of the transaction will be undone. The database will then be restored to its previous state as if the operations never happened.


To illustrate why transactions are crucial to database systems, imagine you work at a bank and you need to transfer money from Customer A to Customer B. This means you have to decrease the balance for the source account and increase the balance for the destination account at the same time.


If either of the two operations fails individually and the other goes through, the banking records would become inconsistent. Either Customer B would get the money out of nowhere (if the balance of Customer A’s account was not decreased), or Customer A would lose money for no reason (if their balance was decreased, but Customer B wasn’t credited). To make sure the results are always consistent, both operations must be successful, or both must fail. Transactions are especially handy in situations like this, ensuring an all-or-nothing execution.


The four properties of database transactions that ensure such complex operations can be safely and reliably performed, guaranteeing data validity despite errors or interruptions, are abbreviated as ACID: atomicity, consistency, isolation, and durability. If the database system can guarantee all four of them for a set of operations grouped in a transaction, it can also guarantee the database will be left in a valid state even in the event of unexpected errors in execution.


- 
Atomicity means that all the actions in a transaction are treated as a single unit of work, and either all or none will be executed with nothing in between. The previous example of money debited from one account to be added to another highlights the atomicity principle. Note that, in MongoDB, updates within a single document (no matter how complex and nested the document structure is) are always atomic, even without using transactions. It’s only in cases when you’re dealing with more than one document that transactions provide stronger atomicity guarantees.

- 
Consistency means that any changes made to a database must adhere to the database’s existing constraints, otherwise the whole transaction will fail. If, for example, one of the operations violated a unique index or a schema validation rule, MongoDB would abort the transaction.

- 
Isolation is the idea that separate, concurrently running transactions are isolated from one another, and neither will affect the other’s outcomes. If two transactions are executed simultaneously, the isolation rule guarantees the end result will be the same as if they were executed one after another.

- 
Durability guarantees that as soon as the transaction succeeds, the client can be sure the data has been properly persisted. Even something like hardware failure or a power interruption won’t void the transaction.


Transactions in MongoDB comply with these ACID principles and can be reliably used in cases when it’s necessary to alter multiple documents in a single go.


# Step 1 — Converting Your Standalone MongoDB Instance into a Replica Set


As mentioned previously, because of the way they’re implemented in MongoDB, transactions can only be performed on databases that are running as part of a larger cluster. This cluster can either be a sharded database cluster or a replica set.


If you’ve already configured a replica set or sharded cluster that you can use to practice running transactions, you can skip this step and use that cluster in Step 2. If not, this step outlines how to convert a standalone MongoDB instance into a single-node replica set.



Warning: A single-node replica set like the one you’ll configure in this step is useful for testing purposes, but it won’t be suitable for use in a production environment. The reason for this is that replica sets are meant to be run on multiple distributed nodes, as this helps to keep the database highly available: if any one node fails, there will still be others in the set that clients can connect to. This single-node set won’t have any of this redundancy, and should only be used for testing in situations that require the use of a replica set.
If you’d like to learn more about replication in MongoDB and the security implications it involves, we strongly encourage you to check out our tutorial on How To Configure a MongoDB Replica Set on Ubuntu 20.04.

To convert your standalone MongoDB instance into a replica set, begin by opening up the MongoDB configuration file using your preferred text editor. This example uses nano:


```
sudo nano /etc/mongod.conf


```


Find the section that reads #replication: towards the bottom of this file:


/etc/mongod.conf
```
. . .
#replication:
. . .

```


Uncomment this line by removing the pound sign (#). Then add a replSetName directive below this line followed by a name that MongoDB will use to identify the replica set:


/etc/mongod.conf
```
. . .
replication:
  replSetName: "rs0"
. . .

```


In this example, the replSetName directive’s value is "rs0". You can provide whatever name you’d like here, but it can be helpful to use a descriptive name.



Note: When replication is enabled, MongoDB also requires you to configure some means of authentication other than password authentication, like keyfile authentication or setting up x.509 certificates. If you followed our How To Secure MongoDB on Ubuntu 20.04 tutorial and enabled authentication on your MongoDB instance,  you will only have password authentication enabled.
Rather than setting up more advanced security measures, for the purposes of this tutorial it would be prudent to disable the security block in your mongod.conf file. Do so by commenting out every line in the security block with a pound sign:
/etc/mongod.conf
. . .

#security:
#  authorization: enabled

. . .

As long as you only plan to use this database for practicing transactions or other testing purposes, this won’t present a security risk. However, if you plan to use this MongoDB instance to store any sensitive data in the future, be sure to uncomment these lines to re-enable authentication.

Those are the only changes you need to make to this file, so you can save and close it. If you used nano to edit the file, you can do so by pressing CTRL + X, Y, and then ENTER.


Following that, restart the mongod service to implement the new configuration changes:


```
sudo systemctl restart mongod


```


After restarting the service, open up the MongoDB shell to connect to the MongoDB instance running on your server:


```
mongo


```


From the MongoDB prompt, run the following rs.initiate() method. This will turn your standalone MongoDB instance into a single-node replica set that you can use for testing:


```
rs.initiate()


```


If this method returns "ok" : 1 in its output, it means the replica set was started successfully:


```
Output{
. . .
	"ok" : 1,
. . .

```


Assuming this is the case, your MongoDB shell prompt will change to indicate that the instance the shell is connected to is now a member of the rs0 replica set:


```



```


Note that this example prompt reflects that this MongoDB instance is a secondary member of the replica set. This is to be expected, as there is usually a gap between the time when a replica set is initiated and the time when one of its members is promoted to become the primary member.


If you were to run a command or even just press ENTER after waiting a few moments, the prompt will update to reflect that you’re connected to the replica set’s primary member:


```



```


Your standalone MongoDB instance is now running as a single-node replica set that you can use for testing transactions. Keep the prompt open for now, as you’ll use the MongoDB shell in the next step to create an example collection and insert some sample data into it.


# Step 2 — Preparing the Sample Data


In order to explain how transactions in MongoDB work and how to use them, this step outlines how to open the MongoDB shell to connect to your replica set’s primary node. It also explains how to create a sample collection and insert a few sample documents into it. This guide will use this sample data to illustrate how to initiate and execute transactions.


If you skipped the previous step because you had an existing sharded MongoDB cluster or replica set, connect to any node that you can write data to:


```
mongo


```



Note: On a fresh connection, the MongoDB shell will automatically connect to the test database by default. You can safely use this database to experiment with MongoDB and the MongoDB shell.
Alternatively, you could also switch to another database to run all of the example commands given in this tutorial. To switch to another database, run the use command followed by the name of your database:
use database_name



To understand the behavior of transactions, you’ll need a set of documents to work with. This guide will use a collection documents representing a few of the most populated cities in the world. As an example, the following sample document represents Tokyo:


The Tokyo document
```
{
    "name": "Tokyo",
    "country": "Japan",
    "continent": "Asia",
    "population": 37.400
}

```


This document contains the following information:


- name: the city’s name.
- country: the country where the city is located.
- continent: the continent where the city is located.
- population: the city’s population, in millions.

Run the following insertMany() method, which will simultaneously create a collection named cities and insert three documents into it:


```
db.cities.insertMany([
    {"name": "Tokyo", "country": "Japan", "continent": "Asia", "population": 37.400 },
    {"name": "Delhi", "country": "India", "continent": "Asia", "population": 28.514 },
    {"name": "Seoul", "country": "South Korea", "continent": "Asia", "population": 25.674 }
])


```


The output will contain a list of object identifiers assigned to the newly inserted objects:


```
Output{
        "acknowledged" : true,
        "insertedIds" : [
                ObjectId("61646915c66c110cc07ca59b"),
                ObjectId("61646915c66c110cc07ca59c"),
                ObjectId("61646915c66c110cc07ca59d")
        ]
}

```


You can verify that the documents were properly inserted by running the find() method with no arguments, which will retrieve every document in the cities collection:


```
db.cities.find()


```


```
Output{ "_id" : ObjectId("61646915c66c110cc07ca59b"), "name" : "Tokyo", "country" : "Japan", "continent" : "Asia", "population" : 37.4 }
{ "_id" : ObjectId("61646915c66c110cc07ca59c"), "name" : "Delhi", "country" : "India", "continent" : "Asia", "population" : 28.514 }
{ "_id" : ObjectId("61646915c66c110cc07ca59d"), "name" : "Seoul", "country" : "South Korea", "continent" : "Asia", "population" : 25.674 }

```


Lastly, use the createIndex() method to create an index that will ensure every document in the collection has a unique name field value. This will be helpful for testing consistency requirements when running transactions later on in this guide:


```
db.cities.createIndex( { "name": 1 }, { "unique": true } )


```


MongoDB will confirm that the index was created successfully:


```
Output{
        "createdCollectionAutomatically" : false,
        "numIndexesBefore" : 1,
        "numIndexesAfter" : 2,
        "commitQuorum" : "votingMembers",
        "ok" : 1,
        . . .
}

```


With that, you have successfully created the list of example documents of the most populated cities that will serve as the test data for testing the use of transactions. Next, you’ll learn how to set up a transaction.


# Step 3 — Creating Your First Complete Transaction


This step outlines how to create a transaction consisting of a single operation that will insert a new document into the sample collection from the previous step.


Begin by opening two separate MongoDB shell sessions. One will be used to execute commands in the transaction, and the other will allow you to check what data is available to other users of the database outside the transaction at different points in time.



Note: To help keep things clear, this guide will use different-colored code blocks to distinguish between these two environments. The first instance, which you’ll use to execute transactions, will have a blue background, like this:



The second instance will be outside of the transaction, allowing you to check how any changes you make within the transaction are visible to clients outside of it. This second environment will have a red background, like this:




At this point, both shell sessions should list the three cities you inserted before if you query the cities collection. Verify that by issuing a find() query in both shell sessions, starting with the first one:


```
db.cities.find()


```


```
Output{ "_id" : ObjectId("61646915c66c110cc07ca59b"), "name" : "Tokyo", "country" : "Japan", "continent" : "Asia", "population" : 37.4 }
{ "_id" : ObjectId("61646915c66c110cc07ca59c"), "name" : "Delhi", "country" : "India", "continent" : "Asia", "population" : 28.514 }
{ "_id" : ObjectId("61646915c66c110cc07ca59d"), "name" : "Seoul", "country" : "South Korea", "continent" : "Asia", "population" : 25.674 }

```


Then run the same query in your second shell session:


```
db.cities.find()


```


```
Output{ "_id" : ObjectId("61646915c66c110cc07ca59b"), "name" : "Tokyo", "country" : "Japan", "continent" : "Asia", "population" : 37.4 }
{ "_id" : ObjectId("61646915c66c110cc07ca59c"), "name" : "Delhi", "country" : "India", "continent" : "Asia", "population" : 28.514 }
{ "_id" : ObjectId("61646915c66c110cc07ca59d"), "name" : "Seoul", "country" : "South Korea", "continent" : "Asia", "population" : 25.674 }

```


After confirming that this query’s output is consistent in both sessions, try to insert a new document into the collection. However, instead of using an insertOne() method, you’ll insert this document as part of a transaction.


Typically, transactions aren’t written and executed from the MongoDB Shell, as this guide outlines. More often than not, transactions are instead used by external applications. To ensure that any transactions it runs remain atomic, consistent, isolated, and durable, the application must start a session.


In MongoDB, a session is a database object managed by an application through the appropriate MongoDB driver. This allows the driver to associate a sequence of database statements with one another, meaning they will have a shared context and can have additional configurations applied to them as a group, like enabling the use of transactions. What happens inside a single session might not be immediately visible to the outside world, as this step will illustrate.


Rather than setting up an external application, this tutorial outlines the concepts and individual steps needed to work with a transaction directly in the MongoDB shell with a simplified JavaScript syntax.



Note: You can learn more on how to use transactions using different programming languages in the official MongoDB documentation. The code examples described in the official documentation will be more complex than what’s included in this guide, but both methods follow the same principles.

Even though this guide outlines how to use transactions through the MongoDB shell rather than in an application, it’s still necessary to start a session in order to execute a set of operations as a transaction. You can start a session with the following command:


```
var session = db.getMongo().startSession()


```


This command creates a session variable that will store the session object. Each time you refer to the session object in the following examples, you’re referring to the session you’ve just started.


With this session object available, you can start the transaction by calling the startTransaction method as follows:


```
session.startTransaction({
    "readConcern": { "level": "snapshot" },
    "writeConcern": { "w": "majority" }
})


```


Notice that the method is called on the session variable and not db, as the commands in the previous step did.

The startTransaction() method accepts two options: readConcern and writeConcern. The writeConcern setting can accept a few options, but this example only includes the w option, which requests that the cluster acknowledges when the transaction’s write operations have been accepted on a specified number of nodes in the cluster. Instead of a single number, this example specifies that the transaction will only be considered to have been saved successfully when a majority of nodes acknowledge the write operation.


Say that you start a transaction, but after you doing so another user adds a document to the collection you’re using on another node in the cluster. Should your transaction read this new data or only the data written on the node on which the transaction was started? Setting the readConcern level allows you to specify which data the transaction should read when you commit the transaction. Setting it to snapshot means that the transaction will read a snapshot of data that has been committed by a majority of nodes in the cluster.


Note that setting the transaction’s readConcern level requires you to set the writeConcern to majority. These values for read and write concerns are safe defaults to follow in most cases. They provide reliable guarantees of data persistence unless you have very particular requirements on performance and acknowledging writes across the replica set. You can learn more about different write and read concerns that MongoDB provides for use in transactions in the official MongoDB documentation.


The startTransaction method won’t return any output if it executed correctly. If this method was successful, you’ll be inside a running transaction and you can begin executing statements that will become part of the transaction.



Warning: By default, MongoDB will automatically abort any transaction that runs for more than 60 seconds. The reason for this is that transactions are not designed to be constructed interactively in the MongoDB shell but rather used in real-world applications.
Because of this, you might encounter unexpected errors while following this tutorial if you don’t execute each command within the 60 second time limit. If you encounter an error like the following, it means that MongoDB has aborted the transaction because the time limit has been exceeded:
Error message
Error: error: {
        "errorLabels" : [
                "TransientTransactionError"
        ],
        "operationTime" : Timestamp(1634032826, 1),
        "ok" : 0,
        "errmsg" : "Transaction 1 has been aborted.",
        "code" : 251,
        "codeName" : "NoSuchTransaction",
        "$clusterTime" : {
                "clusterTime" : Timestamp(1634032826, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        }
}

If this happens, you have to mark the transaction as having been terminated by running the abortTransaction() method, like so:
session.abortTransaction()


You’ll then have to reinitialize the transaction with the same startTransaction() method you ran previously:
session.startTransaction({
    "readConcern": { "level": "snapshot" },
    "writeConcern": { "w": "majority" }
})


Following that, you’ll need to execute each statement in the transaction again from the beginning. With this in mind, it may be helpful for you to first read through the rest of this step and then execute the commands within the 60 second time limit once you better understand the concepts involved.

While you’re working within the running transaction, any statements you run as part of the transaction must be within the shared context of the session represented by the session variable you created previously.


To a similar end, when working in a running transaction it can be helpful to create another variable that represents the collection you want to work with in the context of the session. The following operation will create a variable called cities by returning the cities collection from the test database. However, instead of pulling this directly from the db object it references the session object to ensure that this variable represents the cities collection in the context of the running session:


```
var cities = session.getDatabase('test').getCollection('cities')


```


From now until you commit the transaction, you can use the cities variable just like you would use db.cities to refer to the cities collection. This newly assigned variable will guarantee that you’re running statements in a session and, likewise, in the started transaction.


Test this out by checking that the object can be used to find documents from the collection:


```
cities.find()


```


The command will return the same list of documents as before since the data has not  yet been altered:


```
Output{ "_id" : ObjectId("61646915c66c110cc07ca59b"), "name" : "Tokyo", "country" : "Japan", "continent" : "Asia", "population" : 37.4 }
{ "_id" : ObjectId("61646915c66c110cc07ca59c"), "name" : "Delhi", "country" : "India", "continent" : "Asia", "population" : 28.514 }
{ "_id" : ObjectId("61646915c66c110cc07ca59d"), "name" : "Seoul", "country" : "South Korea", "continent" : "Asia", "population" : 25.674 }

```


Following that, insert a new document representing New York City into the collection as part of the running transaction. Use the insertOne method, but execute it on the cities variable to make sure it will be run in the session:


```
cities.insertOne({"name": "New York", "country": "United States", "continent": "North America", "population": 18.819 })


```


MongoDB will return a success message:


```
Output{
        "acknowledged" : true,
        "insertedId" : ObjectId("6164849d53abeea9d9dd10cf")
}

```


If you were to execute cities.find() again, you’ll notice that the newly inserted document is immediately visible in the same session:


```
cities.find()


```


```
Output{ "_id" : ObjectId("61646915c66c110cc07ca59b"), "name" : "Tokyo", "country" : "Japan", "continent" : "Asia", "population" : 37.4 }
{ "_id" : ObjectId("61646915c66c110cc07ca59c"), "name" : "Delhi", "country" : "India", "continent" : "Asia", "population" : 28.514 }
{ "_id" : ObjectId("61646915c66c110cc07ca59d"), "name" : "Seoul", "country" : "South Korea", "continent" : "Asia", "population" : 25.674 }
{ "_id" : ObjectId("6164822453abeea9d9dd10cf"), "name" : "New York", "country" : "United States", "continent" : "North America", "population" : 18.819 }

```


However, if you run db.cities.find() in your second MongoDB shell instance, the document representing New York will be absent:


```
db.cities.find()


```


```
Output{ "_id" : ObjectId("61646915c66c110cc07ca59b"), "name" : "Tokyo", "country" : "Japan", "continent" : "Asia", "population" : 37.4 }
{ "_id" : ObjectId("61646915c66c110cc07ca59c"), "name" : "Delhi", "country" : "India", "continent" : "Asia", "population" : 28.514 }
{ "_id" : ObjectId("61646915c66c110cc07ca59d"), "name" : "Seoul", "country" : "South Korea", "continent" : "Asia", "population" : 25.674 }

```


The reason for this is that the insert statement has been executed inside the running transaction, but the transaction itself has not been committed yet. At this point, the transaction can still either succeed and persist the data, or it could fail, which would undo all the changes and leave the database in the same state as before you initiated the transaction.


To commit the transaction and save the inserted document permanently to the database, run the commitTransaction method on the session object:


```
session.commitTransaction()


```


As with startTransaction, this command gives no output if it succeeds.


Now, list the documents from the cities collection in both MongoDB shells. Start with querying the cities variable in the running session:


```
cities.find()


```


```
Output{ "_id" : ObjectId("61646915c66c110cc07ca59b"), "name" : "Tokyo", "country" : "Japan", "continent" : "Asia", "population" : 37.4 }
{ "_id" : ObjectId("61646915c66c110cc07ca59c"), "name" : "Delhi", "country" : "India", "continent" : "Asia", "population" : 28.514 }
{ "_id" : ObjectId("61646915c66c110cc07ca59d"), "name" : "Seoul", "country" : "South Korea", "continent" : "Asia", "population" : 25.674 }
{ "_id" : ObjectId("6164822453abeea9d9dd10cf"), "name" : "New York", "country" : "United States", "continent" : "North America", "population" : 18.819 }

```


Then query the cities collection in the second shell running outside of the transaction:


```
db.cities.find()


```


```
Output{ "_id" : ObjectId("61646915c66c110cc07ca59b"), "name" : "Tokyo", "country" : "Japan", "continent" : "Asia", "population" : 37.4 }
{ "_id" : ObjectId("61646915c66c110cc07ca59c"), "name" : "Delhi", "country" : "India", "continent" : "Asia", "population" : 28.514 }
{ "_id" : ObjectId("61646915c66c110cc07ca59d"), "name" : "Seoul", "country" : "South Korea", "continent" : "Asia", "population" : 25.674 }
{ "_id" : ObjectId("6164849d53abeea9d9dd10cf"), "name" : "New York", "country" : "United States", "continent" : "North America", "population" : 18.819 }

```


This time, the newly inserted document is visible both inside the session and outside it. The transaction has been committed and ended successfully, persisting the changes it made to the database. You can now access the New York object both outside transactions and inside all further transactions.


Now that you know how to start and execute a transaction, you can move on to the next step, which outlines how to abort a transaction after starting one to roll back any changes you’ve made before executing it. Be sure to keep both of your MongoDB shell environments open, as you’ll continue using both for the remainder of this tutorial.


# Step 4 — Aborting a Transaction


This step follows a similar path to the previous one, in that it has you start a transaction in the same way. However, this step outlines how to abort the transaction instead of committing the changes. When doing so, all changes introduced by the transaction are rolled back, returning the database to its previous state as if the transaction never happened.


After following the previous step, you’ll have four cities in the collection, including the newly added one representing New York.


In the first MongoDB shell, start the session and assign it to the session variable again:


```
var session = db.getMongo().startSession()


```


Then start the transaction:


```
session.startTransaction({
    "readConcern": { "level": "snapshot" },
    "writeConcern": { "w": "majority" }
})


```


Again, this method won’t return any output if it succeeds. If it is successful, you’ll be inside a running transaction.


You will again need access to the cities collection inside the session context. You can do this by again creating a cities variable to represent the cities collection inside the session:


```
var cities = session.getDatabase('test').getCollection('cities')


```


From now on, you can use cities variable to act on the cities collection in the context of the session.


Now that the transaction is started, insert another new document into this collection as part of the running transaction. The document in this example will represent Buenos Aires. Use the insertOne method, but execute it on the cities variable to ensure it will be run in the session:


```
cities.insertOne({"name": "Buenos Aires", "country": "Argentina", "continent": "South America", "population": 14.967 })


```


MongoDB will return the success message:


```
Output{
        "acknowledged" : true,
        "insertedId" : ObjectId("6164887e322518cf706858b5")
}

```


Next, run the cities.find() query:


```
cities.find()


```


Notice that the newly inserted document is immediately visible in the same session within the transaction:


```
Output{ "_id" : ObjectId("61646915c66c110cc07ca59b"), "name" : "Tokyo", "country" : "Japan", "continent" : "Asia", "population" : 37.4 }
{ "_id" : ObjectId("61646915c66c110cc07ca59c"), "name" : "Delhi", "country" : "India", "continent" : "Asia", "population" : 28.514 }
{ "_id" : ObjectId("61646915c66c110cc07ca59d"), "name" : "Seoul", "country" : "South Korea", "continent" : "Asia", "population" : 25.674 }
{ "_id" : ObjectId("6164849d53abeea9d9dd10cf"), "name" : "New York", "country" : "United States", "continent" : "North America", "population" : 18.819 }
{ "_id" : ObjectId("6164887e322518cf706858b5"), "name" : "Buenos Aires", "country" : "Argentina", "continent" : "South America", "population" : 14.967 }

```


However, if you were to query the cities collection in your second MongoDB shell instance, which is not operating within the transaction, the returned list won’t contain the Buenos Aires document as the transaction hasn’t been committed:


```
db.cities.find()


```


```
Output{ "_id" : ObjectId("61646915c66c110cc07ca59b"), "name" : "Tokyo", "country" : "Japan", "continent" : "Asia", "population" : 37.4 }
{ "_id" : ObjectId("61646915c66c110cc07ca59c"), "name" : "Delhi", "country" : "India", "continent" : "Asia", "population" : 28.514 }
{ "_id" : ObjectId("61646915c66c110cc07ca59d"), "name" : "Seoul", "country" : "South Korea", "continent" : "Asia", "population" : 25.674 }
{ "_id" : ObjectId("6164849d53abeea9d9dd10cf"), "name" : "New York", "country" : "United States", "continent" : "North America", "population" : 18.819 }

```


Say you made a mistake and you no longer want to commit the transaction. Instead, you want to cancel any statements that you’ve run as part of this session and abort the transaction altogether. To do this, run the abortTransaction() method:


```
session.abortTransaction()


```


The abortTransaction() method tells MongoDB to discard all changes introduced in the transaction and return the database to the previous state. As with startTransaction and commitTransaction, this command gives no output if it succeeds.


After successfully aborting the transaction, list the documents from the cities collection in both MongoDB shells. First, run the following operation in the running session:


```
cities.find()


```


```
Output{ "_id" : ObjectId("61646915c66c110cc07ca59b"), "name" : "Tokyo", "country" : "Japan", "continent" : "Asia", "population" : 37.4 }
{ "_id" : ObjectId("61646915c66c110cc07ca59c"), "name" : "Delhi", "country" : "India", "continent" : "Asia", "population" : 28.514 }
{ "_id" : ObjectId("61646915c66c110cc07ca59d"), "name" : "Seoul", "country" : "South Korea", "continent" : "Asia", "population" : 25.674 }
{ "_id" : ObjectId("6164849d53abeea9d9dd10cf"), "name" : "New York", "country" : "United States", "continent" : "North America", "population" : 18.819 }

```


Then run the following query in your second shell instance which is running outside of the session:


```
db.cities.find()


```


```
Output{ "_id" : ObjectId("61646915c66c110cc07ca59b"), "name" : "Tokyo", "country" : "Japan", "continent" : "Asia", "population" : 37.4 }
{ "_id" : ObjectId("61646915c66c110cc07ca59c"), "name" : "Delhi", "country" : "India", "continent" : "Asia", "population" : 28.514 }
{ "_id" : ObjectId("61646915c66c110cc07ca59d"), "name" : "Seoul", "country" : "South Korea", "continent" : "Asia", "population" : 25.674 }
{ "_id" : ObjectId("6164849d53abeea9d9dd10cf"), "name" : "New York", "country" : "United States", "continent" : "North America", "population" : 18.819 }

```


Buenos Aires is absent from both lists. Aborting the transaction after the document was inserted but before the transaction was committed, it’s like inserting it never happened.


In this step, you learned how to terminate a transaction and roll back the changes introduced during its existence. However, transactions aren’t always aborted manually like this. Oftentimes, the reason MongoDB will terminate a transaction before it can be executed is because one of the operations within the transaction caused an error.


# Step 5 — Aborting Transactions Due to Errors


This step is similar to the previous one, but this time you’ll learn what happens when an error occurs during any of the statements executed inside the transaction.


You should at this point still have two open shell sessions. Your collection holds four cities, including the newly added document representing New York. However, the document representing Buenos Aires was not inserted, as it was discarded when you aborted the transaction in the previous step.


In the first MongoDB shell, start the session and assign it to the session variable:


```
var session = db.getMongo().startSession()


```


Then start the transaction:


```
session.startTransaction({
    "readConcern": { "level": "snapshot" },
    "writeConcern": { "w": "majority" }
})


```


Create the cities variable again:


```
var cities = session.getDatabase('test').getCollection('cities')


```


Following that, insert another new document into this collection as part of the running transaction. This example inserts a document representing Osaka, Japan:


```
cities.insertOne({"name": "Osaka", "country": "Japan", "continent": "Asia", "population": 19.281 })


```


MongoDB will return the success message:


```
Output{
        "acknowledged" : true,
        "insertedId" : ObjectId("61648bb3322518cf706858b6")
}

```


The newly inserted city will immediately be visible from inside the transaction:


```
cities.find()


```


```
Output{ "_id" : ObjectId("61646915c66c110cc07ca59b"), "name" : "Tokyo", "country" : "Japan", "continent" : "Asia", "population" : 37.4 }
{ "_id" : ObjectId("61646915c66c110cc07ca59c"), "name" : "Delhi", "country" : "India", "continent" : "Asia", "population" : 28.514 }
{ "_id" : ObjectId("61646915c66c110cc07ca59d"), "name" : "Seoul", "country" : "South Korea", "continent" : "Asia", "population" : 25.674 }
{ "_id" : ObjectId("6164849d53abeea9d9dd10cf"), "name" : "New York", "country" : "United States", "continent" : "North America", "population" : 18.819 }
{ "_id" : ObjectId("61648bb3322518cf706858b6"), "name" : "Osaka", "country" : "Japan", "continent" : "Asia", "population" : 19.281 }

```


However, the Osaka document won’t be visible in the second shell since it is outside the transaction:


```
db.cities.find()


```


```
Output{ "_id" : ObjectId("61646915c66c110cc07ca59b"), "name" : "Tokyo", "country" : "Japan", "continent" : "Asia", "population" : 37.4 }
{ "_id" : ObjectId("61646915c66c110cc07ca59c"), "name" : "Delhi", "country" : "India", "continent" : "Asia", "population" : 28.514 }
{ "_id" : ObjectId("61646915c66c110cc07ca59d"), "name" : "Seoul", "country" : "South Korea", "continent" : "Asia", "population" : 25.674 }
{ "_id" : ObjectId("6164849d53abeea9d9dd10cf"), "name" : "New York", "country" : "United States", "continent" : "North America", "population" : 18.819 }

```


The transaction is still running and can be used to perform further changes on the database.


Run the following operation to try to insert one more document into the collection as part of this transaction. This command will create another document representing New York City. However, because of the uniqueness constraint you applied to the name field when setting up this collection, and because this collection already has a document whose name field’s value is New York, this insertOne operation will conflict with that constraint and cause an error:


```
cities.insertOne({"name": "New York", "country": "United States", "continent": "North America", "population": 18.819 })


```


MongoDB will return the error message noting that this operation violated the unique constraint:


```
OutputWriteError({
        "index" : 0,
        "code" : 11000,
        "errmsg" : "E11000 duplicate key error collection: test.cities index: name_1 dup key: { name: \"New York\" }",
        "op" : {
                "_id" : ObjectId("61648bdc322518cf706858b7"),
                "name" : "New York",
                "country" : "United States",
                "continent" : "North America",
                "population" : 18.819
        }
})
. . .

```


This output indicates that this new document representing New York wasn’t inserted into the database. However, this doesn’t explain what happened to the document representing Osaka that you previously added as part of the transaction.


Say that attempting to add that second New York document was a mistake, but you did intend to keep the Osaka document in the collection. You might try committing the transaction to persist the Osaka document:


```
session.commitTransaction()


```


MongoDB will not allow this and instead throw an error:


```
Outputuncaught exception: Error: command failed: {
        "errorLabels" : [
                "TransientTransactionError"
        ],
        "operationTime" : Timestamp(1633979403, 1),
        "ok" : 0,
        "errmsg" : "Transaction 0 has been aborted.",
        "code" : 251,
        "codeName" : "NoSuchTransaction",
        "$clusterTime" : {
                "clusterTime" : Timestamp(1633979403, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        }
}
. . .

```


Any time an error occurs inside of a transaction it will cause MongoDB to automatically abort the transaction. Also, because transactions are executed in an all-or-nothing manner, no changes from within the transaction are persisted in such a case. The error caused by adding a second document representing New York caused MongoDB to abort the transaction and discard the document representing Osaka.


You can verify this by running the find() query both in both shells. In the first shell, run the query within the context of the session:


```
cities.find()


```


```
Output{ "_id" : ObjectId("61646915c66c110cc07ca59b"), "name" : "Tokyo", "country" : "Japan", "continent" : "Asia", "population" : 37.4 }
{ "_id" : ObjectId("61646915c66c110cc07ca59c"), "name" : "Delhi", "country" : "India", "continent" : "Asia", "population" : 28.514 }
{ "_id" : ObjectId("61646915c66c110cc07ca59d"), "name" : "Seoul", "country" : "South Korea", "continent" : "Asia", "population" : 25.674 }
{ "_id" : ObjectId("6164849d53abeea9d9dd10cf"), "name" : "New York", "country" : "United States", "continent" : "North America", "population" : 18.819 }

```


Then in the second shell, running outside of the session, run find() against db.cities:


```
db.cities.find()


```


```
Output{ "_id" : ObjectId("61646915c66c110cc07ca59b"), "name" : "Tokyo", "country" : "Japan", "continent" : "Asia", "population" : 37.4 }
{ "_id" : ObjectId("61646915c66c110cc07ca59c"), "name" : "Delhi", "country" : "India", "continent" : "Asia", "population" : 28.514 }
{ "_id" : ObjectId("61646915c66c110cc07ca59d"), "name" : "Seoul", "country" : "South Korea", "continent" : "Asia", "population" : 25.674 }
{ "_id" : ObjectId("6164849d53abeea9d9dd10cf"), "name" : "New York", "country" : "United States", "continent" : "North America", "population" : 18.819 }

```


Neither will show Osaka nor a duplicated New York entry. When MongoDB automatically aborted the transaction, it also made sure all changes were reverted. Osaka was briefly visible inside the transaction context but never was available outside the transaction to other database users.


# Conclusion


By reading this article, you familiarized yourself with ACID principles and multi-document transactions in MongoDB. You initiated a transaction, inserted documents as part of that transaction, and learned when the document becomes visible inside and outside the transaction. You learned how to commit the transaction and how to abort it and roll back any changes, as well as what happens when an error occurs inside the transaction.


With these new skills in hand, you can leverage the ACID guarantees of multi-document transactions in applications where they might be needed. Remember, though, that MongoDB is a document-oriented database. In many scenarios, the document model itself, as well as careful schema design, can reduce the need for working with multi-document transactions.


The tutorial provided only a brief introduction to transactions in MongoDB. We encourage you to study the official official MongoDB documentation to learn more about how transactions work.


