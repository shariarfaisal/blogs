# How To Use Unions in SQL

```Databases``` ```SQL```

The author selected the Diversity in Tech Fund to receive a donation as part of the Write for DOnations program.


## Introduction


Many databases spread information across different tables based on their meaning and context. Often, when retrieving information about the data held within the database, you will want to refer to more than one table at a time.


Structured Query Language (SQL) offers multiple approaches for retrieving data from different tables, such as set operations. More specifically, the set operator UNION is widely supported across most of the relational database systems. The UNION operation takes the results of two queries with matching columns and merges them into one.


In this guide, you will use UNION operations to retrieve data from more than one table simultaneously and then combine the results. You will also combine the UNION operator with filtering to order the results.


# Prerequisites


In order to follow this guide, you will need a computer running a SQL-based relational database management system (RDBMS). The instructions and examples in this guide were validated using the following environment:


- A server running Ubuntu 20.04, with a non-root user with administrative privileges and a firewall configured with UFW, as described in our initial server setup guide for Ubuntu 20.04.
- MySQL installed and secured on the server, as outlined in How To Install MySQL on Ubuntu 20.04. This guide was verified with a non-root MySQL user, created using the process described in Step 3.
- Basic familiarity with executing SELECT queries to select data from the database, as described in our How To SELECT Rows FROM Tables in SQL guide.


Note: Please note that many RDBMSs use their own unique implementations of SQL. Although the commands outlined in this tutorial will work on most RDBMSs and are a part of standard SQL syntax, the exact syntax or output may differ if you test them on a system other than MySQL.

You’ll also need a database with some tables loaded with sample data so that you can practice using UNION operations. We encourage you to go through the following section, Connecting to MySQL and Setting up a Sample Database, for details on connecting to a MySQL server and creating the sample database used in the examples throughout this guide.


# Connecting to MySQL and Setting up a Sample Database


In this section, you will connect to a MySQL server and create a sample database so that you can follow the examples in this guide.


If your SQL database system runs on a remote server, SSH into your server from your local machine:


```
ssh sammy@your_server_ip


```


Then open up the MySQL server prompt, replacing sammy with the name of your MySQL user account:


```
mysql -u sammy -p


```


Create a database named bookstore:


```
CREATE DATABASE bookstore;


```


If the database was created successfully, you’ll receive output like this:


```
OutputQuery OK, 1 row affected (0.01 sec)

```


To select the bookstore database, run the following USE statement:


```
USE bookstore;


```


You will receive the following output:


```
OutputDatabase changed

```


After selecting the database, you can create sample tables within it. For the purpose of this guide, you’ll use an imaginary bookstore that offers both book purchases and leases. Both services are managed separately; thus, data about purchases and leases are stored in separate tables.



Note: The database schema for this example is simplified for educational purposes. In real-life scenarios, the table structures would be more complex and involve primary keys and foreign keys. For more information about how databases organize data, see our tutorial on Understanding Relational Databases.

The first table, book_purchases, will contain data about purchased books and the customers who made purchases. It will hold four columns:


- purchase_id: This column holds the purchase identifier, represented by the int data type. This column will become the table’s primary key, with each value becoming a unique identifier for its respective row.
- customer_name: This column will hold customer’s name, expressed using the varchar data type with a maximum of 30 characters.
- book_title: This column will hold the purchased book’s title, expressed using the varchar data type with a maximum of 200 characters.
- date: Using the date data type, this column will hold the date of each purchase.

Create the sample table with the following command:


```
CREATE TABLE book_purchases (
    purchase_id int,
    customer_name varchar(30),
    book_title varchar(40),
    date date,
    PRIMARY KEY (purchase_id)
);


```


If the following output prints, the first table has been created:


```
OutputQuery OK, 0 rows affected (0.00 sec)

```


The second table will be called book_leases and will store information about borrowed books. It is structured similarly to the previous one, but leases are characterized by two distinct dates: lease date and lease duration. To represent that, the leases table will hold five columns:


- lease_id: This column holds the lease identifier, represented by the int data type. This column will become the table’s primary key, with each value becoming a unique identifier for its respective row.
- customer_name: This column will hold customer’s name, expressed using the varchar data type with a maximum of 30 characters.
- book_title: This column will hold borrowed book’s title, expressed using the varchar data type with a maximum of 200 characters.
- date_from: Using the date data type, this column will hold the start date of a lease.
- date_to: Using the date data type, this column will hold the end date of a lease.

Create the second table with the following command:


```
CREATE TABLE book_leases (
    lease_id int,
    customer_name varchar(30),
    book_title varchar(40),
    date_from date,
    date_to date,
    PRIMARY KEY (lease_id)
);


```


The following output confirms the creation of the second table:


```
OutputQuery OK, 0 rows affected (0.00 sec)

```


Following that, load the purchases table with some sample data by running the following INSERT INTO operation:


```
INSERT INTO book_purchases
VALUES
(1, 'sammy', 'The Picture of Dorian Gray', '2022-10-01'),
(2, 'sammy', 'Pride and Prejudice', '2022-10-04'),
(3, 'sammy', 'The Time Machine', '2022-09-23'),
(4, 'bill', 'Frankenstein', '2022-07-23'),
(5, 'bill', 'The Adventures of Huckleberry Finn', '2022-10-01'),
(6, 'walt', 'The Picture of Dorian Gray', '2022-04-15'),
(7, 'walt', 'Frankenstein', '2022-10-13'),
(8, 'walt', 'Pride and Prejudice', '2022-10-19');


```


The INSERT INTO operation will add eight purchases with the specified values to the book_purchases table. The following output indicates that all eight rows have been added:


```
OutputQuery OK, 8 rows affected (0.00 sec)
Records: 8  Duplicates: 0  Warnings: 0

```


Then insert some sample data into the book_leases table:


```
INSERT INTO book_leases
VALUES
(1, 'sammy', 'Frankenstein', '2022-09-14', '2022-11-14'),
(2, 'sammy', 'Pride and Prejudice', '2022-10-01', '2022-12-31'),
(3, 'sammy', 'The Adventures of Huckleberry Finn', '2022-10-01', '2022-12-01'),
(4, 'bill', 'The Picture of Dorian Gray', '2022-09-03', '2022-09-18'),
(5, 'bill', 'Crime and Punishment', '2022-09-27', '2022-12-05'),
(6, 'kim', 'The Picture of Dorian Gray', '2022-10-01', '2022-11-15'),
(7, 'kim', 'Pride and Prejudice', '2022-09-08', '2022-11-17'),
(8, 'kim', 'The Time Machine', '2022-09-04', '2022-10-23');


```


You’ll receive the following output, which confirms that the sample data has been added:


```
OutputQuery OK, 8 rows affected (0.00 sec)
Records: 8  Duplicates: 0  Warnings: 0

```


Leases and purchases relate to similar customers and book titles, which will be useful for demonstrating the UNION operator behavior.


With that, you’re ready to follow the rest of the guide and begin using UNION operations in SQL.


# Understanding the UNION Operator Syntax


The UNION operator in SQL tells the database to merge two separate result sets retrieved through individual SELECT queries into one result set that contains rows returned from both queries.



Note: Databases don’t restrict the complexity of the SELECT queries used with UNION. The data retrieval queries can include JOIN statements, aggregations or subqueries. Often, UNION is used to merge results from complex statements. For educational purposes, the examples in this guide will use SELECT queries to focus on how the UNION operator behaves.

The following example shows the general syntax of an SQL statement that includes a UNION operator:


```
SELECT column1, column2 FROM table1
UNION
SELECT column1, column2 FROM table2;


```


This SQL fragment begins with a SELECT statement that returns two columns from table1, followed by the UNION operator and a second SELECT statement. The second SELECT query also returns two columns, but from table2. The UNION keyword tells the database to take the preceding and following queries, execute them separately, and then join their result sets into one. The whole code fragment, including both SELECT queries and the UNION keyword between them, is a single SQL statement. Because of that, the first SELECT query does not end with a semicolon, which appears only after the whole statement.


As an example, assume you want to list all customers who have either purchased or leased a book. The records for purchases are held in the book_purchases table, whereas leases are stored in the book_leases table. Run the following query:


```
SELECT customer_name FROM book_purchases
UNION
SELECT customer_name FROM book_leases;


```


Here is this query’s result set:


```
Output+---------------+
| customer_name |
+---------------+
| sammy         |
| bill          |
| walt          |
| kim           |
+---------------+
4 rows in set (0.000 sec)

```


This output indicates that Sammy, Bill, Walt, and Kim either purchased or leased books at some point in time. To understand how this result set was generated, try executing the two SELECT statements separately: once for the purchases and once for the leases.


Run the following query to return the customers who purchased books:


```
SELECT customer_name FROM book_purchases;


```


The following output will print to the screen:


```
Output+---------------+
| customer_name |
+---------------+
| sammy         |
| sammy         |
| sammy         |
| bill          |
| bill          |
| walt          |
| walt          |
| walt          |
+---------------+
8 rows in set (0.000 sec)

```


Sammy, Bill, and Walt purchased books, but Kim hasn’t.


Next, run the query to return the customers who leased books:


```
SELECT customer_name FROM book_leases;


```


The following output will print to the screen:


```
Output+---------------+
| customer_name |
+---------------+
| sammy         |
| sammy         |
| sammy         |
| bill          |
| bill          |
| kim           |
| kim           |
| kim           |
+---------------+
8 rows in set (0.000 sec)

```


The leases table refers to Sammy, Bill, and Kim, but Walt never borrows books. By combining the two answers, you get the data for both leases and purchases.


The important difference between using UNION and executing two queries separately is that UNION removes duplicate values, in addition to merging the results: none of the customer names are repeated in the result.


In order to use UNION to merge the results of two separate queries correctly, both queries should return results in the same format. Some discrepancies will result in database engine errors, while others will give results that don’t match the intention of the query.


Consider the two following examples:


UNION with Mismatching Column Counts


Try executing a UNION between a SELECT statement returning a single column and another returning two columns:


```
SELECT purchase_id, customer_name FROM book_purchases
UNION
SELECT customer_name FROM book_leases;


```


The database server will respond with an error:


```
OutputThe used SELECT statements have a different number of columns

```


Performing UNION operations on result sets with different column counts is not possible.


UNION with Mismatching Column Order


Try executing a UNION between two SELECT statements returning the same values but in a different order:


```
SELECT customer_name, book_title FROM book_purchases
UNION
SELECT book_title, customer_name FROM book_leases;


```


The database server will not return an error, but the result set will not be correct:


```
Output+------------------------------------+------------------------------------+
| customer_name                      | book_title                         |
+------------------------------------+------------------------------------+
| sammy                              | The Picture of Dorian Gray         |
| sammy                              | Pride and Prejudice                |
| sammy                              | The Time Machine                   |
| bill                               | Frankenstein                       |
| bill                               | The Adventures of Huckleberry Finn |
| walt                               | The Picture of Dorian Gray         |
| walt                               | Frankenstein                       |
| walt                               | Pride and Prejudice                |
| Frankenstein                       | sammy                              |
| Pride and Prejudice                | sammy                              |
| The Adventures of Huckleberry Finn | sammy                              |
| The Picture of Dorian Gray         | bill                               |
| Crime and Punishment               | bill                               |
| The Picture of Dorian Gray         | kim                                |
| Pride and Prejudice                | kim                                |
| The Time Machine                   | kim                                |
+------------------------------------+------------------------------------+
16 rows in set (0.000 sec)

```


In this example, the UNION operation merges the first column of the first query with the first column of the second query and does the same for the second column, mixing customer names and book titles together.


# Using WHERE Clauses and Ordering Together with UNION


In the previous example, you merged result sets representing all rows in two corresponding tables. Often, you will need to filter rows prior to merging the results together. SELECT statements merged with the UNION operator can employ the WHERE clause to do so.


Assume you want to know which books Sammy reads with the help of your bookstore, either through purchases or leases. Run the following query:


```
SELECT book_title FROM book_purchases
WHERE customer_name = 'Sammy'
UNION
SELECT book_title FROM book_leases
WHERE customer_name = 'Sammy';


```


Both SELECT queries include the WHERE clause, filtering the rows from two separate tables to include only purchases and leases by Sammy. The result set for this query will print as follows:


```
Output+------------------------------------+
| book_title                         |
+------------------------------------+
| The Picture of Dorian Gray         |
| Pride and Prejudice                |
| The Time Machine                   |
| Frankenstein                       |
| The Adventures of Huckleberry Finn |
+------------------------------------+
5 rows in set (0.000 sec)

```


Once again, UNION ensures there will be no duplicates on the list of results. You can use WHERE clauses to limit what rows are returned in both SELECT queries or only one of them. Additionally, the WHERE clause can refer to different columns and conditions in both statements.


The results returned through the UNION operation don’t follow any particular order. To change that, you can utilize the ORDER BY clause. The ordering is performed on the final, merged results and not on the individual queries.


To sort the book titles alphabetically after retrieving a list of all books purchased or leased by Sammy, execute the following query:


```
SELECT book_title FROM book_purchases
WHERE customer_name = 'Sammy'
UNION
SELECT book_title FROM book_leases
WHERE customer_name = 'Sammy'
ORDER BY book_title;


```


The following output will print to the screen:


```
Output+------------------------------------+
| book_title                         |
+------------------------------------+
| Frankenstein                       |
| Pride and Prejudice                |
| The Adventures of Huckleberry Finn |
| The Picture of Dorian Gray         |
| The Time Machine                   |
+------------------------------------+
5 rows in set (0.001 sec)

```


This time, the results are returned in an order based on the book_title column containing merged results from both SELECT queries.


# Using UNION ALL to Retain the Duplicates


As the previous examples have demonstrated, the UNION operator automatically removes duplicate rows from the results. However, sometimes this behavior is not what you expect or intend to achieve with the query. For example, assume you are interested in books that have been either purchased or leased on October 1st, 2022. To retrieve these titles, you can follow a similar example as before:


```
SELECT book_title FROM book_purchases
WHERE date = '2022-10-01'
UNION
SELECT book_title FROM book_leases
WHERE date_from = '2022-10-01'
ORDER BY book_title;


```


You will get the following results:


```
Output+------------------------------------+
| book_title                         |
+------------------------------------+
| Pride and Prejudice                |
| The Adventures of Huckleberry Finn |
| The Picture of Dorian Gray         |
+------------------------------------+
3 rows in set (0.001 sec)

```


The returned book titles are correct, but the results won’t tell you whether these books were only purchased, only leased, or both. In cases where some books have been both purchased and leased, their titles would appear in both the book_purchases and book_leases table. However, as a result of UNION removing duplicate rows, that information is lost in the result.


Fortunately, SQL has a way to alter this behavior and retain duplicate rows. You can use the UNION ALL operator to merge results from two queries without removing duplicate rows. UNION ALL works similarly to UNION, but in cases where there are multiple occurrences of the same values, all will be present in the result.


Run the same query, but change UNION to UNION ALL:


```
SELECT book_title FROM book_purchases
WHERE date = '2022-10-01'
UNION ALL
SELECT book_title FROM book_leases
WHERE date_from = '2022-10-01'
ORDER BY book_title;


```


This time, the resulting list will be longer:


```
Output+------------------------------------+
| book_title                         |
+------------------------------------+
| Pride and Prejudice                |
| The Adventures of Huckleberry Finn |
| The Adventures of Huckleberry Finn |
| The Picture of Dorian Gray         |
| The Picture of Dorian Gray         |
+------------------------------------+
5 rows in set (0.000 sec)

```


Two books — The Adventures of Huckleberry Finn and The Picture of Dorian Gray — appear two times in the result set. This means that these titles appeared in both the book_purchases and book_leases tables. For duplicate entries, you can assume they have been both leased and purchased on that day.


Depending on whether you want duplicates removed or retained, you can choose between the UNION and UNION ALL operators, which can be used interchangeably.



Note: Executing UNION ALL is faster than executing UNION, as the database doesn’t need to scan the result set against duplicates. If you are merging results of two SELECT queries that you know won’t contain any duplicate rows, using UNION ALL can bring a noticeable performance gain on larger data sets.

# Conclusion


By following this guide, you retrieved data from multiple tables using UNION and UNION ALL operations. You also used WHERE clauses to filter the results and ORDER BY clauses to order them. Finally, you learned about possible errors and unexpected behaviors if the SELECT statements yield different data formats.


While the commands included here should work on most relational databases, be aware that every SQL database uses its own unique implementation of the language. (For more on these differences, see our guide, SQLite vs MySQL vs PostgreSQL: A Comparison Of Relational Database Management Systems.) You should consult your RDBMS’s official documentation for a more complete description of each command and its full set of options.


If you’d like to learn more about different concepts around the SQL language and working with it, we encourage you to check out the other guides in the How To Use SQL series.


