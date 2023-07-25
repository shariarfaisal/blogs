# Databases Checkpoint

```Database``` ```Databases``` ```MongoDB``` ```MySQL``` ```NoSQL``` ```PostgreSQL``` ```Redis``` ```SQL``` ```SQLite```

## Introduction


This checkpoint is intended to help you assess what you learned from our introductory articles to Databases, where we defined databases and introduced common database management systems. You can use this checkpoint to test your knowledge on these topics, review key terms and commands, and find resources for continued learning.


A database is any logically modeled collection of information or data. When people refer to a “database” in the context of websites, applications, and the cloud, they often mean a computer program that manages data stored on a computer. These programs, known formally as database management systems (DBMS), can be combined with other programs (like a web server and a front-end framework) to form production-ready applications.


In this checkpoint, you’ll find two sections that synthesize the central ideas from the introductory articles: a brief explanation of what a database is (including subsections on relational and non-relational databases) and a section on how to interact with your DBMS through the command line or graphical user interfaces. In each of these sections, there are interactive components to help you test your knowledge. At the end of this checkpoint, you will find opportunities for continued learning about database management systems, fully managed databases, and building your apps with backend databases.


## Resources


- An Introduction to Databases
- Understanding SQL Constraints
- SQLite vs MySQL vs PostgreSQL: A Comparison Of Relational Database Management Systems
- A Comparison of NoSQL Database Management Systems and Models
- How To Install and Secure Redis on Ubuntu 22.04
- How To Perform CRUD Operations in MongoDB

# What Is a Database?


A database is any logically modeled collection of information, and a database management system is what most people think of when they think “I know what a database is!” You use a database management system (DBMS), which is a computer program designed to interact with the information, to access and manipulate the information stored in your database.



Terms to Know
Define the following terms, then use the dropdown feature to check your work.

Replication
Replication refers to the practice of synchronizing data across multiple separate databases. This practice provides redundancy, improves scalability, and reduces read latencies.


Sharding
Database sharding is an architectural practice of separating data into chunks called logical shards that are distributed across separate nodes called physical shards. For more on this practice, you can review our article on Understanding Database Sharding.


There are three common relational models used for database systems:





Relational Model
Relationship




One-to-one
In a one-to-one relationship, rows in one table (sometimes called the parent table) are related to one and only one row in another table (sometimes called the child table).


One-to-many
In a one-to-many relationship, a row in the initial table (sometimes called the parent table) can relate to multiple rows in another table (sometimes called the child table).


Many-to-many
In a many-to-many relationship, rows in one table can related to multiple rows in the other table, and vice versa. While these tables may also be referred to as parent and child tables, the multidirectional relationship does not necessitate a hierarchical relationship.




These relational models structure how databases can relate to each other.


There are two categories for database management: relational and non-relational databases. In the following subsections, you will learn about each type and the common DBMSs for those types.


## Relational Databases


A relational database organizes information through relations, which you might recognize as a table.



Check Yourself

What are the elements that make up a relation?
A relation is a set of tuples, or rows in a table, with each tuple sharing a set of attributes, or columns. A tuple is a unique instance of what type of data the table holds, whereas an attribute specifies the data type for what is allowed in the column.



What is the difference between a primary key and a foreign key?
A primary key refers to the column that will uniquely identify each row in a relational table, whereas a foreign key is a copy of the primary key inserted into a second relation in order to create a relationship between two tables.


When information is stored in a database and organized in the relation, it can be accessed through queries that make a structured request for information. Many relational databases use the Structured Query Language, commonly referred to as SQL to manage queries to the database.


You can use SQL constraints when designing your database. These constraints impose restrictions on what changes can be made to the data in the table.



Check Yourself

Why might you impose constraints on your database?

Business Rules: Constraints enable database administrators to ensure the database aligns with defined policies and procedures that meet business needs and expectations.
Data Integrity: Data entry is prone to input errors, so constraints provide additional parameters to ensure correct data.



What are the five constraints that are formally defined by the SQL standard?

PRIMARY KEY requires every entry in the given column to be both unique and not NULL, and allows you to use that column to identify each individual row in the table.
FOREIGN KEY requires that every entry in the given column must already exist in a specific column from another table.
UNIQUE prohibits any duplicate values from being added to the given column.
CHECK defines a requirement for a column, known as a predicate, that every value entered into it must meet.
NOT NULL prohibits any NULL values from being added to the given column.



Some open-source relational database management systems built with SQL include MySQL, MariaDB, PostgreSQL, and SQLite. Continue learning about relational databases with Understanding Relational Databases and review common relational DBMS with SQLite vs MySQL vs PostgreSQL: A Comparison Of Relational Database Management Systems.



Relational Database Terms To Know
Through each of the articles, you have developed a vocabulary about relational databases. Define each of the following terms, then use the dropdown feature to check your work.

Constraint
A constraint is any rule applied to a column or table that limits what data can be entered into it.


Data Types
A data type dictates what kind of entries are allowed in a column.


Object Database
An object database uses object-oriented structures for information. PostgreSQL is a relational database that incorporates some features from object databases.


Serverless
A serverless database, like SQLite, allows any process that accesses the database to write and write to the database disk file directly. This behavior is in contrast to the interprocess communication that is implemented by other relational database engines.
You can write a serverless function to practice running a serverless application.


Signed and Unsigned Integers
Some numeric data types are signed, which means they can represent both positive and negative numbers, while others are unsigned and can only represent the positive number.


Now that you know about relational databases, you can understand their counterpart: non-relational databases.


## Non-relational and NoSQL Databases


If you need to store data in an unstructured way, a non-relational database provides an alternative model. Because a non-relational database does not use SQL, it is sometimes referred to as a NoSQL database.


There are a variety of available options for a non-relational database, such as key-value stores, columnar databases, document stores, and graph databases. Each of these models attends to possible issues with using a relational database, including horizontal scaling, eventual consistency across nodes, and unstructured data management.



Non-relational Database Terms to Know
Each of the non-relational database models has specific features that make it unique. Define the type of model, then use the dropdown feature to check your work.

Key-value databases
Key-value databases store and manage associate arrays that contain key-value pairs where the key is a unique identifier that retrieves its associated value.


Columnar databases
Columnar databases are column-oriented, which means that they store data in columns. The data appears in record order where the first entry in one column is related to the first entry in other columns.


Document-oriented databases
Also known as a document store, these are NoSQL databases that store data in the form of documents. Each document contains metadata to structure the data, and an API or query language can be used to retrieve documents.


Graph databases
Graph databases are a document-store subcategory, and this type of database highlights the relationships between documents.


You can check your knowledge about which popular non-relational databases management systems align with the type of database model with the following interactive dropdown feature.



Check Yourself
Match the following database management system to its operational database model.



Redis
Couchbase
Cassandra
OrientDB




MongoDB
Neo4j
MemcacheDB
Apache HBase




Compare your answers using the dropdown feature.



Operational Database Model
Example DBMSs




Key-value store
Redis, MemcacheDB


Columnar database
Cassandra, Apache HBase


Document store
MongoDB, Couchbase


Graph database
OrientDB, Neo4j





Whether you are using a relational or a non-relational database, you are likely building an application that includes a database management system as part of its stack.


## Building an Application Stack


The database management system is most often deployed as an essential aspect of a larger application. These applications are sometimes called stacks, such as the LAMP stack or Elastic stack.



Check Yourself
Use the dropdown feature to get the answers.

What makes up a LAMP stack?
LAMP is an acronym for the technology that makes up this stack:

Linux operating system
Apache web server
MySQL database
PHP for dynamic content processing

There are other L*MP options, such as the LEMP stack where E stands for Nginx or the LOMP stack where O stands for OpenLiteSpeed.


What makes up Elastic stack?
Elastic stack is built around Elasticsearch, which is both a search engine and a document-oriented database.


If you set up a remote server with your application stack, it is recommended that you encrypt your data to protect your system from malicious interference. You can encrypt communications using transport layer security (TLS), which will convert data in motion into a ciphertext that can only be decrypting by the right cipher. The static data stored in your database will remain unencrypted unless using a DBMS that offers data at rest encryption.


To manage your database, you may opt do so directly from the command line interface or through a graphical user interface.


# Using the Command Line with your DBMS


You began to use the Linux command line with our introductory articles on cloud servers and you configured a web server with the introductory articles on web server solutions. Through the articles on databases, you have continued to develop familiarity with the command line using commands such as:


- grep to search plain-text data for a specific text or string.
- netstat to check network configuration with the flags -lnp to show listening sockets (-l), numeric addresses (-n), and the PID and name of the program for each socket (-p).
- systemctl to control the systemd service.

You have also experimented with the command line tools that come with different database management systems in order to interact with the database installation. The CLI tool enables you to execute commands on the database server and work interactivley from your terminal window. The following table lists common DBMSs and their associated CLI tool:





DBMS
CLI tool




MongoDB
MongoDB shell


MySQL
mysql


PostgreSQL
psql


Redis
redis-cli




There are also third-party command line clients for some database management systems, such as Redli for Redis.


When you use the command line to work with your database system, you open a database-specific server prompt, typically associated with your user account for that database management system. For example, if you were to open a MySQL server prompt and log in with your MySQL user, you would review a database prompt like so:


```



```


Each DBMS command-line client has its own syntax for commands.


After learning about SQL constraints, you might use those constraints with a MySQL database by running these commands:


- CREATE DATABASE to create a database.
- USE to select a database.
- CREATE TABLE to create a table with specifications for the columns and constraints applied to those columns.
- ALTER TABLE with ADD to add constraints to an existing table and with DROP CONSTRAINT to delete a constraint from an existing table.

You can continue to develop your MySQL database skills with the How To Use SQL series.


With Redis, you installed and secured Redis with the following commands and experimented with renaming commands:


- auth to authenticate clients for database access.
- exit and quit to exit the Redis-CLI prompt.
- get to retrieve the key value.
- ping to test connectivity.
- set to set keys.

And, in MongoDB shell, you used binary JSON (known as BSON) to run CRUD  operations with the following methods of query filtering:


- count method to check the object count in a specified collection.
- deleteOne to remove the first document that matches the specifications.
- deleteMany to remove multiple objects at once.
- find to retrieve documents in your MongoDB database with the pretty printing feature to make the lines more readable.
- insertOne method to create individual documents.
- insertManymethod to insert multiple documents in a single operation or collection.
- ObjectId object datatype for storing object identifiers.
- updateOne to update a single document with specified keys.
- updateMany to update every document in a collection that matches the specified filters.

You will likely use CRUD operations to interact with your data across many database management systems.



Check Yourself

What does CRUD stand for?
CRUD is an acronym used to describe the following four fundamental data operations:

Create
Read
Update
Delete



While you may opt to manage your database directly from the command line, you can also use a graphical user interface (GUI) for many common database management systems.


## Using a Graphical User Interface


There are many different GUI tools for working with your database if you decide against using the designed CLI tool.


To handle MySQL administration over the web, you can use phpMyAdmin by installing and securing phpMyAdmin on many different operating systems or connecting remotely to a MySQL Managed Database. You can also use MySQL Workbench to connect to a MySQL server remotely.


Similar to phpMyAdmin, pgAdmin is a web interface for managing PostgreSQL. You can install and configure pgAdmin in server mode or use it to schedule automatic backups with pgAgent.


For MongoDB, you might consider using MongoDB Compass as the graphical interface to access your database.


Whether you choose to use the command line or a graphical interface to manage your database, you now have the tools necessary to manage your database system.


# What’s Next?


With a stronger understanding of databases and popular database management systems, you can store and manage your data or build an application that uses a database system.


For more about working with specific database management systems, you can follow our How To Use SQL and How To Manage Data with MongoDB series. If you run into issues with MySQL, you can debug with How To Troubleshoot Issues in MySQL. For MongoDB issues, assess how your issues related to How To Perform CRUD Operations in MongoDB.


When you’re ready to build your apps with databases, try following these tutorials for common application stack setups:


- How To Install Linux, Apache, MySQL, PHP (LAMP) Stack on Ubuntu 22.04
- How To Install Linux, Nginx, MySQL, PHP (LEMP stack) on Ubuntu 22.04
- How To Install Linux, OpenLiteSpeed, MariaDB, PHP (LOMP stack) on Ubuntu 22.04

If you prefer building your apps with fully managed databases, check out the DigitalOcean offerings for managed MongoDB clusters, MySQL or PostgreSQL hosting, and managed Redis. You can also choose a popular database option for a 1-click installation in the DigitalOcean Marketplace.


With your newfound knowledge of databases, you can also continue your cloud journey with containers and security. If you haven’t yet, check out our introductory articles on cloud servers and web servers.


