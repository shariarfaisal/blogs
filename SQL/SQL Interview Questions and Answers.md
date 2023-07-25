# SQL Interview Questions and Answers

```Interview Questions``` ```SQL```

SQL interview questions are asked in almost all interviews because database operations are very common in applications. SQL stands for Structured Query Language, which is a domain-specific programming language used for database communications and relational database management. SQL consists of standard commands for database interactions such as SELECT, INSERT, CREATE, DELETE, UPDATE, DROP etc. With these, it allows the user to retrieve and upload data from databases, create and delete table elements, and implement dynamic database interactions between servers and programs.


# SQL Interview Questions


 Listed below are various SQL interview questions and answers that reaffirms your knowledge about SQL and provide new insights and learning about the language. Go through these SQL interview questions to refresh your knowledge before any interview.


1. 
What is SQL?


SQL is a domain-specific programming language that allows you to query and manipulate data within database management systems arranged in an optimized manner and categorization. This is possible through implementing commands in SQL that allows you to read, write, select and delete entries or even columns of the same attribute or tables. SQL also provides a very efficient way of creating a dynamic accessway between your programs, websites, or mobile apps to a database. For example, by inputting your login details on a user website, these log information is passed on to a database by SQL for verification and user restriction.3.  ### What is the difference between a Database and a Relational Database?


Database or Database Management System(DBMS) and Relational Database Management System(DBMS) are both used by SQL to store data and structures. However, each type of Database Management System is preferred with respect to different uses.  The main difference between the two is that DBMS saves your information as files whereas RDMS saves your information in tabular form. Also, as the keyword Relational implies, RDMS allows different tables to have relationships with one another using Primary Keys, Foreign Keys etc. This creates a dynamic chain of hierarchy between tables which also offers helpful restriction on the tables. DBMS sorts out its tables through a hierarchal manner or navigational manner. This is useful when it comes to storing data in tables that are independent of one another and you don’t wish to change other tables while a table is being filled or edited.5.  ### What is the Basic Structure of an SQL?


SQL is framed upon the structure of relational operations. It is based on certain modifications and enhancements. A very basic SQL query form is:


```
select A1, A2, ..., An

from R1, R2, ..., Rm

where P

```


Here An are attributes, Rm is the relations within the database and P is the predicate or filter.7.  ### What are different categories of SQL commands?


SQL command falls into following four categories:


- DML (Data Manipulation Language) which provides data manipulation features
- DDL (Data Definition Language) which is used to manipulate database structures
- TCL (Transaction Control Language) that takes in charge data transaction verification and error handling
- DCL (Data Control Language) are security statements that feature user restrictions and data access permissions to promote security of your data.

1. 
What is SQL used for?


SQL is used and is popular for server-side programmers for its ability to process a large number of entries in a database in a very fast and easy manner. This opens up to large improvements in data retrieval and manipulation. To expound on this, SQL provides the ability to execute, retrieve, insert, update, delete entries to and from a database. It also allows to create structures such as tables, views, and databases provided a unique name is given.10.  ### Define SELECT, INSERT, CREATE, DELETE, UPDATE, DROP keywords


- SELECT keyword is used to highlight and get entries in rows from tables or views. It can also be accompanied by AS keyword to provide an alias. To filter the SELECT statement, WHERE clause may be included to provide filter conditions and select only the wished entries that satisfy the condition.
- INSERT allows to add or insert a row or multiple rows in a database table. Accompanied by VALUES keyword lets you add a row with specific values. INSERT may also be accompanied with SELECT to insert the preselected row.
- CREATE is a keyword used to create elements in SQL. It is usually accompanied with the keyword to be created such as CREATE DATABASE, CREATE TABLE, CREATE VIEW, etc.
- DELETE keyword is used to deletes record(s) in a database. You should always use it carefully to avoid unwanted data loss. You may delete records you didn’t want to delete. Use WHERE clause to specify the range of the records you wish to delete.
- UPDATE keyword updates or changes the existing data within an existing record. Be sure to note that the record must be existent.
- DROP keyword drops or deletes a table within the database.

1. 
What are the key differences between SQL and P/L SQL?


SQL or Structured Query Language is a language which is used to communicate with a relational database. It provides a way to manipulate and create databases. On the other hand, PL/SQL is a dialect of SQL which is used to enhance the capabilities of SQL. It was developed by Oracle Corporation in the early '90s. It adds procedural features of programming languages in SQL.13.  ### What is data definition language?


DDL or Data Definition Language pertains to the SQL commands directly affecting the database structure. DDL is a category of SQL command classifications that also include DML (Data Manipulation Language), Transactions, and Security. A particular attribute of DDL commands is statements that can manipulate indexes, objects, tables, views, triggers, etc. Three popular DDL keywords in SQL are:


1. 
CREATE – which is used to create a table
CREATE TABLE tableName (name data_type);


2. 
ALTER – used to modify entries or existing columns within a table.
ALTER TABLE tableName [additional syntax such as ADD, DROP, MODIFY]


3. 
DROP – used to Delete or Drop an existing table along with its entries, constraints, triggers, indexes, and permissions. Essentially deletes the table.
DROP TABLE tableName;


4. 
What is Data Manipulation Language?


DML or Data Manipulation Language is a set of commands that are classified pertaining to its capability to give users permission to change entries within the database. This may be through Inserting, Retrieving, Deleting or Updating data within tables. Popular DML statements arise from these core functionalities and are listed below:


- 
SELECT – used to highlight a row within a table and retrieve it.
SELECT [columnName] FROM [tableName]


- 
UPDATE – used to update entries from existing tables.
UPDATE [tableName] SET [value]


- 
INSERT – used to insert entries into an existing table.
INSERT INTO [tableName]


- 
DELETE – used to delete entries from an existing table
DELETE FROM [tableName]



1. 
What is Transaction Control Language (TCL)?


TCL is a category of SQL commands which primarily deals with the database transaction and save points. These keywords implement the SQL functions and logic defined by the developer into the database structure and behavior. Examples of these TCL commands are: COMMIT – used to commit a transaction ROLLBACK – in any advent of errors, transaction rollback is invoked by this keyword. SAVEPOINT – keyword representing the reverting point of rollback SET TRANSACTION – sets the specifics of the transaction19.  ### What is Data Control Language (DCL)?


Data Control Language or DCL oversees the issuance of access and restrictions to users, including the rights and permissions required within the SQL statements. Example DCL keywords are: GRANT – DCL keyword that provides access to certain databases to users. REVOKE – opposite of the GRANT keyword. Revokes or withdraws the privileges given to the user.20.  ### Define tables and fields in a database


In terms of databases, a table is referred to as an arrangement of organized entries. It is further divided into cells which contain different fields of the table row. A field pertains to a data structure that represents a single piece of entry. They are then further organized to records. They practically hold a single piece of data. They are the basic unit of memory allocation for data and is accessible.21.  ### What are different types of keys in SQL?


Keys are a vital feature in RDMS, they are essentially fields that link one table to another and promote fast data retrieval and logging through managing column indexes. Different types of keys are: Primary Key – a unique key that identifies records in database tables. By unique it means that it must not be Null and must be unique in the table. Candidate Key – a unique field which identifies for column or group of columns independently, without any required reference to other fields. Alternate Key – can be substituted in use for Primary Keys but are considered as a secondary. The difference is that Alternate Keys can have a Null value, provided that the columns have data within them. A type of Candidate Key which is also required to be unique. Unique Key – Keys that offer restriction to prevent duplicate data within rows except for null entries. The other keys available are Foreign Keys, Super Keys, and Composite Keys.22.  ### Name the different types of indexes in SQL and define them.


Unique Index: Prevents duplicate entries within uniquely indexed columns. They are automatically generated if a Primary Key is available. Clustered Index: Used to organize or edit the arrangement within the table, with respect to the key value. Each table is only allowed to have a single clustered index only. NonClustered Index: Conversely, NonClustered Index only manages the order of logic within entries. It does not manage the arrangement and tables can have multiple NonClustered Indexes.26.  ### What is the difference between SQL and MySQL?


SQL which stands for Standard Query Language is a server programming language that provides interaction to database fields and columns. While MySQL is a type of Database Management System, not an actual programming language, more specifically an RDMS or Relational Database Management System. However, MySQL also implements the SQL syntax.27.  ### What is UNION and UNION ALL keyword in SQL and what are their differences?


The UNION operator in SQL combines multiple sets highlighted in the SELECT statements. The restrictions of the set are: (1) column number must be identical, (2) Data Types in the set must be the same, and (3) the order of the column highlighted in the SELECT statement must be the same. It automatically eliminates duplicate rows within the results highlighted in the SELECT statement. UNION ALL does the same function as the UNION, but it includes all, including the duplicate rows.


```
SELECT C1, C2 FROM T1
UNION
SELECT Cx, Cy FROM T2;

```


1. 
What are the different types of joins in SQL?


The join keyword queries entries from multiple tables. It is used with different keys to find these entries and is conscious on the link between fields.


1. 
Inner Join: Returns rows which are common between the tables

2. 
Right Join: Returns rows of the right-hand side table, including the common rows.

3. 
Left Join: Returns rows of the left-hand side table, including the common rows.

4. 
Full Join: Returns all rows, regardless if common or not.

5. 
What is Normalization and Denormalization?


Normalization arranges the existing tables and its fields within the database, resulting in minimum duplication. It is used to simplify a table as much as possible while retaining the unique fields. Denormalization allows the retrieval of fields from all normal forms within a database. With respect to normalization, it does the opposite and puts redundancies into the table.32.  ### When can we use the WHERE clause and the HAVING clause?


Both clauses accept conditions that are used for the basis on retrieving fields. The difference is that the WHERE clause is only used for static non-aggregated columns while the HAVING clause is used for aggregated columns only.


```
select order_id, SUM(sale_amount) as TotalSale 
from SalesData 
where quantity>1 
group by order_id 
having SUM(sale_amount) > 100;

```


1. 
What is the difference among UNION, MINUS and INTERSECT?


The UNION keyword is used in SQL for combining multiple SELECT queries but deletes duplicates from the result set. The INTERSECT keyword is only used for retrieving common rows using SELECT queries between multiple tables. The MINUS keyword essentially subtracts between two SELECT queries. The result is the difference between the first query and the second query. Any row common across both the result set is removed from the final output.35.  ### How to select 10 records from a table?


MySQL: Using limit clause, example select * from Employee limit 10; Oracle: Using ROWNUM clause, example SELECT * FROM Employee WHERE ROWNUM < 10; SQL Server: Using TOP clause, example SELECT TOP 3 * FROM Employee;39.  ### How can you maintain the integrity of your database on instances where deleting an element in a table result in the deletion of the element(s) within another table?


This is possible by invoking an SQL trigger which listens for any elements that are deleted in Table A and deletes the corresponding linked elements from Table B.40.  ### What is the process of copying data from Table A to Table B?


```
INSERT INTO TableB (columnOne, columnTwo, columnThree, ...)

SELECT columnOne, columnTwo, columnThree, ...

FROM TableA

WHERE added_condtion;

```


1. 
What are the differences between IN and EXISTS clause?


The apparent difference between the two is that the EXISTS keyword is relatively faster at execution compared to IN keyword. This is because the IN keyword must search all existing records while EXISTS keywords automatically stop when a matching record has been found. Also, IN Statement operates within the ResultSet while EXISTS keyword operates on virtual tables. In this context, the IN Statement also does not operate on queries that associates with Virtual tables while the EXISTS keyword is used on linked queries.43.  ### What does the acronym ACID stand for in Database Management?


The ACID Acronym stands for Atomicity, Consistency, Isolation, and Durability. This property primarily takes charge of the process integrity of the database system. This means that whatever the user issues as a data transaction to the database must be done completely, accurately, and has withstanding property.44.  ### What is a trigger in SQL?


A database trigger is a program that automatically executes in response to some event on a table or view such as insert/update/delete of a record. Mainly, the database trigger helps us to maintain the integrity of the database.45.  ### What is Auto Increment feature in SQL?


Auto increment allows the user to create a unique number to be generated whenever a new record is inserted in the table. AUTO INCREMENT is the keyword for Oracle, AUTO_INCREMENT in MySQL and IDENTITY keyword can be used in SQL SERVER for auto-incrementing. Mostly this keyword is used to create the primary key for the table.46.  ### What is collation?


Collation is basically a set of rules on how to compare and sort characters, extended to strings. Collation in MSSQL and MySQL works pretty much the same way, except on certain collation options such as UTF-8. Besides the normal character-wise comparison, collation can also sort and compare strings on an ASCII representation perspective.47.  ### What is a recursive stored procedure?


A stored procedure which calls by itself until it reaches some boundary condition. This recursive function or procedure helps programmers to use the same set of code any number of times.48.  ### Which query operators in SQL is used for pattern matching?


The answer is the LIKE operator. LIKE operator is used for pattern matching, and it can be used as -.


- % – Matches zero or more characters.
- _(Underscore) – Matching exactly one character.

1. 
What is Hibernate and its relation to SQL?


Hibernate is Object Relational Mapping tool in Java. Hibernate let’s us write object-oriented code and internally converts them to native SQL queries to execute against a relational database. Hibernate uses its own language like SQL which is called Hibernate Query Language(HQL). The difference is that HQL boasts on being able to query Hibernate’s entity objects. It also has an object-oriented query language in Hibernate which is called Criteria Query. It proves very beneficial and helpful to developers who primarily use objects in their front-end applications and Criteria Query can cater to those objects in even add SQL-like features such as security and restriction-access.52.  ### How can we solve SQL Error: ORA-00904: invalid identifier?


This error usually appears due to syntax errors on calling a column name in Oracle database, notice the ORA identifier in the error code. Make sure you typed in the correct column name. Also, take special note on the aliases as they are the one being referenced in the error as the invalid identifier.53.  ### What is a SQL Profiler?


The SQL Profiler is a Graphical User Interface that allows database developers to monitor and track their database engine activities. It features activity logging for every event occurring and provides analysis for malfunctions and discrepancies. It basically is a diagnostic feature in SQL that debugs performance issues and provides a more versatile way of seeing which part in your trace file is causing a clog in your SQL transactions.54.  ### How can we link a SQL database to an existing Android App?


It will require a JDBC (Java Database Connectivity) driver to link these two. Also, you must add the corresponding dependencies to your build.gradle file along with the permissions and grants.


Reference: Wikipedia


