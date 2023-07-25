# How To Use Functions in SQL

```Databases``` ```SQL```

The author selected the Diversity in Tech Fund to receive a donation as part of the Write for DOnations program.


## Introduction


When working with relational databases and Structured Query Language (SQL), you can store, manage, and retrieve data from the relational database management system. SQL can retrieve data intact, as stored within the database.


SQL can also perform calculations and manipulate data through the use of functions. For instance, you can use functions to retrieve product prices rounded to the nearest dollar, calculate the average number of product purchases, or determine the number of days until the warranty on a given purchase expires.


In this tutorial, you’ll use different SQL functions to perform mathematical calculations, manipulate strings and dates, and calculate summaries using aggregate functions.


# Prerequisites


To follow this guide, you will need a computer running a SQL-based relational database management system (RDBMS). The instructions and examples in this guide were validated using the following environment:


- A server running Ubuntu 20.04, with a non-root user with administrative privileges and a firewall configured with UFW, as described in our initial server setup guide for Ubuntu 20.04.
- MySQL installed and secured on the server, as outlined in How To Install MySQL on Ubuntu 20.04. This guide was verified with a non-root MySQL user, created using the process described in Step 3.
- Basic familiarity with executing SELECT queries to select data from the database, as described in our How To SELECT Rows FROM Tables in SQL guide.


Note: Many RDBMSs use their own implementation of SQL. Although the commands outlined in this tutorial will work on most RDBMSs, the standard SQL syntax specifies a limited number of functions. Moreover, support for the standard syntax varies across different database engines. The exact syntax or output may differ if you test them on a system other than MySQL.

You’ll also need a database with some tables loaded with sample data so that you can practice using functions. We encourage you to go through the following Connecting to MySQL and Setting up a Sample Database section for details on connecting to a MySQL server and creating the testing database used in examples throughout this guide.


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


After selecting the database, you can create sample tables within it. For this guide, we’ll use an imaginary bookstore that sells books by different authors.


The table inventory will contain data about books in the bookstore. It will hold the following columns:


- book_id: This column holds the identifier for each book, represented by the int data type. This column will become the table’s primary key, with each value becoming a unique identifier for its respective row.
- author: This column holds the book author’s name, expressed using the varchar data type with a maximum of 50 characters.
- title: This column holds the purchased book’s title, expressed using the varchar data type with a maximum of 200 characters.
- introduction_date: Using the date data type, this column holds the date each book was introduced by the bookstore.
- stock: This column holds the number of books the bookstore has in its inventory using the int integer data type.
- price: This column stores the book’s retail price using the decimal data type with a maximum of 5 values before the decimal point and 2 values after it.

Create the sample table with the following command:


```
CREATE TABLE inventory (
    book_id int,
    author varchar(50),
    title varchar(200),
    introduction_date date,
    stock int,
    price decimal(5, 2),
    PRIMARY KEY (book_id)
);


```


If the following output prints, the table has been created:


```
OutputQuery OK, 0 rows affected (0.00 sec)

```


Following that, load the purchases table with some sample data by running the following INSERT INTO operation:


```
INSERT INTO inventory
VALUES
(1, 'Oscar Wilde', 'The Picture of Dorian Gray', '2022-10-01', 4, 20.83),
(2, 'Jane Austen', 'Pride and Prejudice', '2022-10-04', 12, 42.13),
(3, 'Herbert George Wells', 'The Time Machine', '2022-09-23', 7, 21.99),
(4, 'Mary Shelley', 'Frankenstein', '2022-07-23', 9, 17.43),
(5, 'Mark Twain', 'The Adventures of Huckleberry Finn', '2022-10-01', 14, 23.15);


```


The INSERT INTO operation will add five books with the specified values to the inventory table. The following output indicates that all five rows have been added:


```
OutputQuery OK, 5 rows affected (0.00 sec)
Records: 5  Duplicates: 0  Warnings: 0

```


With that, you’re ready to follow the rest of the guide and begin using functions in SQL.


# Understanding SQL Functions


Functions are named expressions that take one or multiple values, perform calculations or transformations on the data, and return a new value as a result. You can think of SQL functions similarly to functions in mathematics. A function log(x) takes some x and returns the value of the logarithm for x.


Typically, to retrieve information from a relational database (without transforming it), you would use the SELECT query, asking the database to return values for individual columns you are interested in by specifying the column names in the statement.


For example, if you want to retrieve all books titles with their prices, ordered from the most to the least expensive, you could execute the following statement:


```
SELECT title, price, introduction_date FROM inventory ORDER BY price DESC;


```


You would receive the following output:


```
Output+------------------------------------+-------+-------------------+
| title                              | price | introduction_date |
+------------------------------------+-------+-------------------+
| Pride and Prejudice                | 42.13 | 2022-10-04        |
| The Adventures of Huckleberry Finn | 23.15 | 2022-10-01        |
| The Time Machine                   | 21.99 | 2022-09-23        |
| The Picture of Dorian Gray         | 20.83 | 2022-10-01        |
| Frankenstein                       | 17.43 | 2022-07-23        |
+------------------------------------+-------+-------------------+
5 rows in set (0.000 sec)

```


In this statement, title, price, and introduction_date are column names, and in the resulting output, the database presented intact values retrieved from those columns for each book: the complete book title, price, and the date the book entered the bookstore.


However, you may also want to retrieve values from the database after some form of processing or manipulation. You could be interested in book prices rounded up to the nearest dollar, the book titles shown in uppercase, or the introduction year, without the month or day included. This is when you would use a function.


SQL functions can be broadly categorized into several groups, depending on the type of data they operate on. These are the most commonly used functions:


- Mathematical Functions: Functions operating on numerical values and performing computations, such as rounding, logarithms, square roots, or powers.
- String Manipulation Functions: Functions operating on strings and text fields that perform text transformations, such as converting the text to uppercase, trimming, or replacing words within the values.
- Date and Time Functions: Functions operating on fields holding dates. These functions perform computations and transformations, such as adding a number of days to the given date or taking just a year from the full date.
- Aggregate Functions: A special case of mathematical functions that operate on values coming from multiple rows, such as calculating an average price for all rows.


Note: Most relational databases, including MySQL, extend the standard set of functions defined by the SQL standard with additional operations specific to that database engine. Many functions outside the standard set of SQL functions work similarly in many databases, whereas others are confined to a single RDBMS and its unique features. You can consult the documentation for your database of choice to learn more about the functions the database offers. In the case of MySQL, you can learn more in the Built-In Function and Operator Reference.

The following example shows the general syntax for using an imaginary, non-existent function called EXAMPLE to alter the results for the price values in the bookstore inventory database using the SELECT query:


```
SELECT EXAMPLE(price) AS new_price FROM inventory;


```


The function (EXAMPLE) takes the column name (price) as an argument enclosed in parentheses. This portion of the query tells the database to execute the function EXAMPLE over the values of the column price and return the results of this operation. The AS new_price tells the database to assign a temporary name (new_price) for the computed values during the duration of the query. With that, you can distinguish the function results in the output and can refer to the computed values using WHERE and ORDER BY clauses.


In the following section, you will use mathematical functions to perform commonly used computations.


# Using Mathematical Functions


Mathematical functions operate on numerical values, such as the book price or the number of books in stock in the sample database. They can be used to perform calculations within the database to conform the results to your requirements.


Rounding is one of the most commonly used applications of mathematical functions in SQL. Imagine you need to retrieve prices for all books, but you are interested in the values rounded to the nearest whole dollar. To do so, you can use the ROUND function, which performs the rounding operation.


Try executing the following statement:


```
SELECT title, price, ROUND(price) AS rounded_price FROM inventory;


```


The following output will print to the screen:


```
Output+------------------------------------+-------+---------------+
| title                              | price | rounded_price |
+------------------------------------+-------+---------------+
| The Picture of Dorian Gray         | 20.83 |            21 |
| Pride and Prejudice                | 42.13 |            42 |
| The Time Machine                   | 21.99 |            22 |
| Frankenstein                       | 17.43 |            17 |
| The Adventures of Huckleberry Finn | 23.15 |            23 |
+------------------------------------+-------+---------------+
5 rows in set (0.000 sec)

```


The query selects the values from the title and price columns intact, alongside a temporary rounded_price column with the results of the ROUND(price) function. This function takes one argument, the column name (in this case, it’s price), and returns the values from that column in the table rounded to the nearest integer value.


The rounding function can also accept the additional argument that defines the number of decimal places to which the rounding should occur, as well as arithmetic operations instead of a single column name. For example, try running the following query:


```
SELECT title, price, ROUND(price * stock, 1) AS stock_price FROM inventory;


```


You’ll receive the following output:


```
Output+------------------------------------+-------+-------+-------------+
| title                              | stock | price | stock_price |
+------------------------------------+-------+-------+-------------+
| The Picture of Dorian Gray         |     4 | 20.83 |        83.3 |
| Pride and Prejudice                |    12 | 42.13 |       505.6 |
| The Time Machine                   |     7 | 21.99 |       153.9 |
| Frankenstein                       |     9 | 17.43 |       156.9 |
| The Adventures of Huckleberry Finn |    14 | 23.15 |       324.1 |
+------------------------------------+-------+-------+-------------+
5 rows in set (0.000 sec)

```


Executing ROUND(price * stock, 1) will first multiply the single book price by the number of books in stock and then round the resulting price to the first decimal place. The result will be presented in the stock_price temporary column.


Other mathematical functions built into MySQL include trigonometric functions, square roots, powers, logarithms, and exponentials. You can learn more about using mathematical functions in SQL in the How To Use Mathematical Expressions and Aggregate Functions in SQL tutorial.


In the next section, you’ll manipulate text from the database using SQL functions.


# Using String Manipulation Functions


String manipulation functions in SQL enable you to alter the values stored in columns holding text when processing the SQL query. They can be used, among others, to convert cases, concatenate data from multiple columns, or perform search and replace operations.


You’ll start using string functions by retrieving all the book titles converted to lowercase. Execute the following statement:


```
SELECT LOWER(title) AS title_lowercase FROM inventory;


```


The following output will print to the screen:


```
Output+------------------------------------+
| title_lowercase                    |
+------------------------------------+
| the picture of dorian gray         |
| pride and prejudice                |
| the time machine                   |
| frankenstein                       |
| the adventures of huckleberry finn |
+------------------------------------+
5 rows in set (0.001 sec)

```


The SQL function named LOWER takes a single argument and converts its contents to lowercase. Through the column alias AS title_lowercase, the resulting data is presented in the temporary column named title_lowercase.


Now retrieve all authors, this time converted into uppercase. Try running the following SQL query:


```
SELECT UPPER(author) AS author_uppercase FROM inventory;


```


You’ll receive the following output:


```
Output+----------------------+
| author_uppercase     |
+----------------------+
| OSCAR WILDE          |
| JANE AUSTEN          |
| HERBERT GEORGE WELLS |
| MARY SHELLEY         |
| MARK TWAIN           |
+----------------------+
5 rows in set (0.000 sec)

```


Instead of the LOWER function, you used the UPPER function, which works similarly but converts the text into uppercase. Both functions can be used if you want to guarantee character case consistency when retrieving data.


Another useful string manipulation function is CONCAT, which takes multiple arguments holding textual values and puts them together. Try retrieving book authors and titles combined within a single column. To do so, execute the following statement:


```
SELECT CONCAT(author, ': ', title) AS full_title FROM inventory;


```


This statement returns the following output:


```
Output+------------------------------------------------+
| full_title                                     |
+------------------------------------------------+
| Oscar Wilde: The Picture of Dorian Gray        |
| Jane Austen: Pride and Prejudice               |
| Herbert George Wells: The Time Machine         |
| Mary Shelley: Frankenstein                     |
| Mark Twain: The Adventures of Huckleberry Finn |
+------------------------------------------------+
5 rows in set (0.001 sec)

```


The CONCAT function concatenated multiple strings together, and it was executed with three arguments. The first, author, refers to the author column holding author names. The second, : , is an arbitrary string value to delimit authors and book titles with a colon. The last one, title, refers to the column holding book titles.


In the result of this query, authors and titles are returned in a single temporary column named full_title, concatenated directly by the database engine.


Other string functions built into MySQL include functions for searching and replacing strings, retrieving substrings, padding and trimming string values, and applying regular expressions, among others. You can learn more about using SQL functions for concatenating multiple values in the How To Manipulate Data with CAST Functions and Concatenation Expressions in SQL tutorial. You can also refer to the String Functions and Operators in the MySQL documentation.


In the next section, you’ll use SQL functions to manipulate dates from the database.


# Using Date and Time Functions


Date and time functions in SQL enable you to manipulate the values stored in columns holding dates and timestamps when processing the SQL query. They can be used to extract parts of the date information, perform date arithmetics, or format dates and timestamps into required output formats.


Let’s assume you will need to get the book introduction date split separately into the year, month, and day instead of having a single date column in the output.


Try executing the following statement:


```
SELECT introduction_date, YEAR(introduction_date) as year, MONTH(introduction_date) as month, DAY(introduction_date) as day FROM inventory;


```


You will receive this output:


```
Output+-------------------+------+-------+------+
| introduction_date | year | month | day  |
+-------------------+------+-------+------+
| 2022-10-01        | 2022 |    10 |    1 |
| 2022-10-04        | 2022 |    10 |    4 |
| 2022-09-23        | 2022 |     9 |   23 |
| 2022-07-23        | 2022 |     7 |   23 |
| 2022-10-01        | 2022 |    10 |    1 |
+-------------------+------+-------+------+
5 rows in set (0.000 sec)

```


This SQL statement used three individual functions: YEAR, MONTH, and DAY. Each function takes the column name where dates are stored as an argument and extracts just a single part of the complete date: a year, a month, or a day, respectively. Using these functions, you can access individual date fragments within SQL queries.


Another helpful date manipulation function is DATEDIFF, which allows you to retrieve the number of days between two dates. Now, try checking how many days have passed between the introduction date of each book and the current date.


Run the following query:


```
SELECT introduction_date, DATEDIFF(introduction_date, CURRENT_DATE()) AS days_since FROM inventory;


```


The following output will print to the screen:


```
Output+-------------------+------------+
| introduction_date | days_since |
+-------------------+------------+
| 2022-10-01        |        -30 |
| 2022-10-04        |        -27 |
| 2022-09-23        |        -38 |
| 2022-07-23        |       -100 |
| 2022-10-01        |        -30 |
+-------------------+------------+
5 rows in set (0.000 sec)

```


The DATEDIFF function takes two arguments: the start date and the end date. The DATEDIFF function calculates the number of days that separate these two points in time. The result may be a negative number if the end date comes earlier. In this example, the first argument is the introduction_date column name holding the dates in the inventory table. The second argument is another function, CURRENT_DATE, representing the current system date. Executing this query retrieves the number of days between these two points in time and puts the results in the days_since temporary column.



Note: DATEDIFF is not a part of the official SQL standard set of functions. While many databases support this function, the syntax often differs between different database engines. This example follows the syntax native to MySQL.

Other date manipulation functions built into MySQL include adding and subtracting date and time intervals, formatting dates for different language formats, retrieving day and month names, or creating new date values. You can learn more about working with dates in SQL in the How To Work with Dates and Times in SQL tutorial. You can also refer to the Date and Time Functions in the MySQL documentation.


In the next section, you’ll learn how to use aggregate functions.


# Using Aggregate Functions


In all the previous examples, you used SQL functions to apply transformations or calculations to individual column values within a single row, which represents a book in a bookstore. SQL provides a way to perform mathematical calculations across multiple rows to help you find aggregate information about the whole dataset.


The primary aggregate functions in SQL include the following:


- AVG for the average of the values the calculations are performed on.
- COUNT for the number of values the calculations are performed on.
- MAX for the maximum value.
- MIN for the minimum value.
- SUM for the sum of all values.

You can incorporate multiple aggregate functions in your SELECT query. Imagine you want to check the number of books listed in the bookstore, the maximum price of any book available, and the average price across the whole catalog. To do that, execute the following statement:


```
SELECT COUNT(title) AS count, MAX(price) AS max_price, AVG(price) AS avg_price FROM inventory;


```


This statement returns the following output:


```
Output+-------+-----------+-----------+
| count | max_price | avg_price |
+-------+-----------+-----------+
|     5 |     42.13 | 25.106000 |
+-------+-----------+-----------+
1 row in set (0.001 sec)

```


The above query uses three aggregate functions at the same time. The COUNT function counts the rows the query looks up. In this example, title is passed as the argument, but since the number of rows will be the same for every column checked, you could also use any other column name as the function’s argument. The MAX function calculates the maximum value from the price column: here, the column name is important, as the computation is done on that column’s values. The last function is the AVG function that computes the average across all prices from the price column.


Using aggregate functions this way results in the database returning a single row with temporary columns representing the values of aggregate computations. The source rows are used for calculation internally but are not returned through the query. In this example, you used aggregate functions to calculate statistical values from the whole inventory table at once, taking all rows into account for the summary.


With SQL, it is also possible to divide the rows in the table into groups, and then calculate aggregate values for those groups separately. For example, you could calculate the average price for books by different authors to find out which author publishes the most expensive titles. You can learn more about grouping rows for such computations in the How To Use GROUP BY and ORDER BY in SQL tutorial. You can also delve into more details about using aggregates by following the How To Use Mathematical Expressions and Aggregate Functions in SQL tutorial.


# Conclusion


By following this guide, you learned what SQL functions are and how to use them to manipulate numbers, strings, and dates using functions. You have used ROUND to round numerical values, CONCAT to concatenate multiple columns into one, and DATEDIFF to compute the number of days between two points in time. Finally, you have also used aggregate functions such as COUNT, SUM, or AVG to generate summaries across multiple rows.


You can use functions to offload some of the data manipulation and computation to the database engine. This tutorial covered only the basics of using functions for that purpose. To retrieve and analyze data in robust ways, you can combine functions with conditional queries using the WHERE clause and grouping described in How To Use GROUP BY and ORDER BY in SQL.


While the commands shown here should work on most relational databases, be aware that every SQL database uses its own implementation of the language. You should consult your DBMS’s official documentation for a complete description of each command and its full sets of options.


If you’d like to learn more about different concepts around the SQL language and working with it, we encourage you to check out the other guides in the How To Use SQL series.


