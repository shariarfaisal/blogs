# How To Use Stored Procedures in MySQL

```Databases``` ```Development``` ```MySQL``` ```SQL```

The author selected the Diversity in Tech Fund to receive a donation as part of the Write for DOnations program.


## Introduction


Typically, when working with a relational database, you issue individual Structured Query Language (SQL) queries to retrieve or manipulate data, like SELECT, INSERT, UPDATE or DELETE, directly from within your application code. Those statements work on and manipulate underlying database tables directly. If the same statements or group of statements are used within multiple applications accessing the same database, they are often duplicated in individual applications.


MySQL, similar to many other relational database management systems, supports the use of stored procedures. Stored procedures help group one or multiple SQL statements for reuse under a common name, encapsulating common business logic within the database itself. Such a procedure can be called from the application that accesses the database to retrieve or manipulate data in a consistent way.


Using stored procedures, you can create reusable routines for common tasks to be used across multiple applications, provide data validation, or deliver an additional layer of data access security by restricting database users from accessing the underlying tables directly and issuing arbitrary queries.


In this tutorial, you’ll learn what stored procedures are and how to create basic stored procedures that return data and use both input and output parameters.


# Prerequisites


To follow this guide, you will need a computer running a SQL-based relational database management system (RDBMS). The instructions and examples in this guide were validated using the following environment:


- A server running Ubuntu 20.04, with a non-root user with administrative privileges and a firewall configured with UFW, as described in our initial server setup guide for Ubuntu 20.04.
- MySQL installed and secured on the server, as outlined in How To Install MySQL on Ubuntu 20.04. This guide was verified with a non-root MySQL user, created using the process described in Step 3.
- Basic familiarity with executing SELECT queries to retrieve data from the database as described in our How To SELECT Rows FROM Tables in SQL guide.


Note: Please note that many RDBMSs use their own unique implementations of SQL, and stored procedures syntax is not part of the official SQL standard. Although the commands outlined in this tutorial may work in other RDBMSs, stored procedures are database-specific, and thus the exact syntax or output may differ if you test them on a system other than MySQL.

You’ll also need an empty database in which you’ll be able to create tables demonstrating the use of stored procedures. We encourage you to go through the following Connecting to MySQL and Setting up a Sample Database section for details on connecting to a MySQL server and creating the testing database used in examples throughout this guide.


# Connecting to MySQL and Setting up a Sample Database


In this section, you will connect to a MySQL server and create a sample database so that you can follow the examples in this guide.


For this guide, you’ll use an imaginary car collection. You’ll store details about currently owned cars, with their make, model, build year, and value.


If your SQL database system runs on a remote server, SSH into your server from your local machine:


```
ssh sammy@your_server_ip


```


Then open up the MySQL server prompt, replacing sammy with the name of your MySQL user account:


```
mysql -u sammy -p


```


Create a database named procedures:


```
CREATE DATABASE procedures;


```


If the database was created successfully, you’ll receive output like this:


```
OutputQuery OK, 1 row affected (0.01 sec)

```


To select the procedures database, run the following USE statement:


```
USE procedures;


```


You will receive the following output:


```
OutputDatabase changed

```


After selecting the database, you can create sample tables within it. The table cars will contain simplified data about cars in the database. It will hold the following columns:


- make: This column holds the make for each owned car, expressed using the varchar data type with a maximum of 100 characters.
- model: This column holds the car model name, expressed using the varchar data type with a maximum of 100 characters.
- year: This column stores the car’s build year with int data type to hold numerical values.
- value: This column stores the car’s value using the decimal data type with a maximum of 10 digits and 2 digits after the decimal point.

Create the sample table with the following command:


```
CREATE TABLE cars (
    make varchar(100),
    model varchar(100),
    year int,
    value decimal(10, 2)
);


```


If the following output prints, the table has been created:


```
OutputQuery OK, 0 rows affected (0.00 sec)

```


Following that, load the cars table with some sample data by running the following INSERT INTO operation:


```
INSERT INTO cars
VALUES
('Porsche', '911 GT3', 2020, 169700),
('Porsche', 'Cayman GT4', 2018, 118000),
('Porsche', 'Panamera', 2022, 113200),
('Porsche', 'Macan', 2019, 27400),
('Porsche', '718 Boxster', 2017, 48880),
('Ferrari', '488 GTB', 2015, 254750),
('Ferrari', 'F8 Tributo', 2019, 375000),
('Ferrari', 'SF90 Stradale', 2020, 627000),
('Ferrari', '812 Superfast', 2017, 335300),
('Ferrari', 'GTC4Lusso', 2016, 268000);


```


The INSERT INTO operation will add ten sample sports cars to the table, with five Porsche and five Ferrari models. The following output indicates that all five rows have been added:


```
OutputQuery OK, 10 rows affected (0.00 sec)
Records: 10  Duplicates: 0  Warnings: 0

```


With that, you’re ready to follow the rest of the guide and begin using stored procedures in SQL.


# Introduction to Stored Procedures


Stored procedures in MySQL and in many other relational database systems are named objects that contain one or more instructions laid out and then executed by the database in a sequence when called. In the most basic example, a stored procedure can save a common statement under a reusable routine, such as retrieving data from the database with often-used filters. For example, you could create a stored procedure to retrieve online store customers who made orders within the last given number of months. In the most complex scenarios, stored procedures can represent extensive programs describing intricate business logic for robust applications.


The set of instructions in a stored procedure can include common SQL statements, such as SELECT or INSERT queries, that return or manipulate data. Additionally, stored procedures can make use of:


- Parameters passed to the stored procedure or returned through it.
- Declared variables to process retrieved data directly within the procedure code.
- Conditional statements, which allow the execution of parts of the stored procedure code depending on certain conditions, such as IF or CASE instructions.
- Loops, such as WHILE, LOOP, and REPEAT, allow executing parts of the code multiple times, such as for each row in a retrieved data set.
- Error handling instructions, such as returning error messages to the database users accessing the procedure.
- Calls to other stored procedures in the database.


Note: The extensive syntax supported by MySQL allows for writing robust programs and solving complex problems with stored procedures. This guide covers only the basic use of stored procedures with SQL statements enclosed in the stored procedure body, input, and output parameters. Executing conditional code, using variables, loops, and customized error handling is out of the scope for this guide. We encourage you to learn more about stored procedures in the official MySQL documentation.

When the procedure is called by its name, the database engine executes it as defined, instruction by instruction.


The database user must have the appropriate permissions to execute the given procedure. This permissions requirement provides a layer of security, disallowing direct database access while giving users access to individual procedures that are guaranteed safe to execute.


Stored procedures are executed directly on the database server, performing all computations locally and returning results to the calling user only when finished.


If you want to change the procedure behavior, you can update the procedure in the database, and the applications that are using it will automatically pick up the new version. All users will immediately start using the new procedure code without needing to adjust their applications.


Here is the general structure of the SQL code used to create a stored procedure:


```
DELIMITER //
CREATE PROCEDURE procedure_name(parameter_1, parameter_2, . . ., parameter_n)
BEGIN
    instruction_1;
    instruction_2;
    . . .
    instruction_n;
END //
DELIMITER ;


```


The first and last instructions in this code fragment are DELIMITER // and DELIMITER ;. Usually, MySQL uses the semicolon symbol (;) to delimit statements and indicate when they start and end. If you execute multiple statements in the MySQL console separated with semicolons, they will be treated as separate commands and executed independently, one after another. However, the stored procedure can enclose multiple commands that will be executed sequentially when it gets called. This poses a difficulty when trying to tell MySQL to create a new procedure. The database engine would encounter the semicolon sign in the stored procedure body and think it should stop executing the statement. In this situation, the intended statement is the whole procedure creation code, not a single instruction within the procedure itself, so MySQL would misinterpret your intentions.


To work around this limitation, you use the DELIMITER command to temporarily change the delimiter from ; to // for the duration of the CREATE PROCEDURE call. Then, all semicolons inside the stored procedure body will be passed to the server as-is. After the whole procedure is finished, the delimiter is changed back to ; with the last DELIMITER ;.


The heart of the code to create a new procedure is the CREATE PROCEDURE call followed by the name of the procedure: procedure_name in the example. The procedure name is followed by an optional list of parameters the procedure will accept. The last part is the procedure body, enclosed in BEGIN and END statements. Inside is the procedure code, which can contain a single SQL statement such as a SELECT query or more complex code.


The END command ends with //, a temporary delimiter, instead of a typical semicolon.


In the next section, you’ll create a basic stored procedure with no parameters enclosing a single query.


# Creating a Stored Procedure Without Parameters


In this section, you’ll create your first stored procedure encapsulating a single SQL SELECT statement to return the list of owned cars ordered by their make and value in descending order.


Start by executing the SELECT statement that you’re going to use:


```
SELECT * FROM cars ORDER BY make, value DESC;


```


The database will return the list of cars from the cars table, first ordered by make and then, within a single make, by value in descending order:


```
Output+---------+---------------+------+-----------+
| make    | model         | year | value     |
+---------+---------------+------+-----------+
| Ferrari | SF90 Stradale | 2020 | 627000.00 |
| Ferrari | F8 Tributo    | 2019 | 375000.00 |
| Ferrari | 812 Superfast | 2017 | 335300.00 |
| Ferrari | GTC4Lusso     | 2016 | 268000.00 |
| Ferrari | 488 GTB       | 2015 | 254750.00 |
| Porsche | 911 GT3       | 2020 | 169700.00 |
| Porsche | Cayman GT4    | 2018 | 118000.00 |
| Porsche | Panamera      | 2022 | 113200.00 |
| Porsche | 718 Boxster   | 2017 |  48880.00 |
| Porsche | Macan         | 2019 |  27400.00 |
+---------+---------------+------+-----------+
10 rows in set (0.00 sec)

```


The most valuable Ferrari is at the top of the list, and the least valuable Porsche appears at the bottom.


Assume this query will be used frequently in multiple applications or by multiple users and assume you want to ensure everyone will use the exact same way of ordering the results. To do so, you want to create a stored procedure that will save that statement under a reusable named procedure.


To create this stored procedure, execute the following code fragment:


```
DELIMITER //
CREATE PROCEDURE get_all_cars()
BEGIN
    SELECT * FROM cars ORDER BY make, value DESC;
END //
DELIMITER ;


```


As described in the previous section, the first and last commands (DELIMITER // and DELIMITER ;) tell MySQL to stop treating the semicolon character as the statement delimiter for the duration of procedure creation.


The CREATE PROCEDURE SQL command is followed by the procedure name get_all_cars, which you can define to best describe what the procedure does. After the procedure name, there is a pair of parentheses () where you can add parameters. In this example, the procedure doesn’t use parameters, so the parentheses are empty. Then, between the BEGIN and END commands defining the beginning and end of the procedure code block, the previously used SELECT statement is written verbatim.



Note: Depending on your MySQL user permissions, you may receive an error when executing the CREATE PROCEDURE command: ERROR 1044 (42000): Access denied for user 'sammy'@'localhost' to database 'procedures'. To grant permissions to create and execute stored procedures to your user, log in to MySQL as root and execute the following commands, replacing the MySQL username and host as needed:
GRANT CREATE ROUTINE, ALTER ROUTINE, EXECUTE on *.* TO 'sammy'@'localhost';
FLUSH PRIVILEGES;


After updating the user permissions, log out as root, log back in as the user, and rerun the CREATE PROCEDURE statement.
You can learn more about applying permissions regarding stored procedures to database users in the Stored Routines and MySQL Privileges documentation.

The database will respond with a success message:


```
OutputQuery OK, 0 rows affected (0.02 sec)

```


The get_all_cars procedure is now saved in the database, and when called, it will execute the saved statement as is.


To execute saved stored procedures, you can use the CALL SQL command followed by the procedure name. Try running the newly created procedure like so:


```
CALL get_all_cars;


```


The procedure name, get_all_cars, is all you need to use the procedure. You no longer need to manually type any part of the SELECT statement you used previously. The database will display the results just like the output from the SELECT statement run before:


```
Output+---------+---------------+------+-----------+
| make    | model         | year | value     |
+---------+---------------+------+-----------+
| Ferrari | SF90 Stradale | 2020 | 627000.00 |
| Ferrari | F8 Tributo    | 2019 | 375000.00 |
| Ferrari | 812 Superfast | 2017 | 335300.00 |
| Ferrari | GTC4Lusso     | 2016 | 268000.00 |
| Ferrari | 488 GTB       | 2015 | 254750.00 |
| Porsche | 911 GT3       | 2020 | 169700.00 |
| Porsche | Cayman GT4    | 2018 | 118000.00 |
| Porsche | Panamera      | 2022 | 113200.00 |
| Porsche | 718 Boxster   | 2017 |  48880.00 |
| Porsche | Macan         | 2019 |  27400.00 |
+---------+---------------+------+-----------+
10 rows in set (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

```


You have now successfully created a stored procedure without any parameters that return all cars from the cars table ordered in a particular way. You can use the procedure across multiple applications.


In the next section, you will create a procedure that accepts parameters to change the procedure behavior depending on user input.


# Creating a Stored Procedure with an Input Parameter


In this section, you’ll include input parameters to the stored procedure definition to allow users executing the procedure to pass data to it. For example, users could provide query filters.


The previously created stored procedure get_all_cars retrieved all cars from the cars table at all times. Let’s create another procedure to find cars from a given manufacturing year. To allow that, you’ll define a named parameter in the procedure definition.


Run the following code:


```
DELIMITER //
CREATE PROCEDURE get_cars_by_year(
    IN year_filter int
)
BEGIN
    SELECT * FROM cars WHERE year = year_filter ORDER BY make, value DESC;
END //
DELIMITER ;


```


There are several changes to the procedure creation code from the previous section.


First, the name is get_cars_by_year, which describes the procedure: retrieve cars based on their production year.


The previously empty parentheses now contain a single parameter definition: IN year_filter int. The IN keyword tells the database that the parameter will be passed by the calling user into the procedure. The year_filter is an arbitrary name for the parameter. You will use it to refer to the parameter in the procedure code. Finally, int is the data type. In this case, the production year is expressed as a numerical value.


The year_filter parameter defined after the procedure’s name appears in the SELECT statement in the WHERE year = year_filter clause, filtering the cars table against their production year.


The database will once again respond with a success message:


```
OutputQuery OK, 0 rows affected (0.02 sec)

```


Try executing the procedure without passing any parameters to it, just like you did previously:


```
CALL get_cars_by_year;


```


The MySQL database will return an error message:


```
Error messageERROR 1318 (42000): Incorrect number of arguments for PROCEDURE procedures.get_cars_by_year; expected 1, got 0

```


This time, the stored procedure expects a parameter to be provided, but none was given. To call a stored procedure with parameters, you can provide parameter values within parentheses in the same order as expected by the procedure. To retrieve cars manufactured in 2017, execute:


```
CALL get_cars_by_year(2017);


```


Now, the called procedure will execute correctly and return the list of cars from that year:


```
Output+---------+---------------+------+-----------+
| make    | model         | year | value     |
+---------+---------------+------+-----------+
| Ferrari | 812 Superfast | 2017 | 335300.00 |
| Porsche | 718 Boxster   | 2017 |  48880.00 |
+---------+---------------+------+-----------+
2 rows in set (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

```


In this example, you’ve learned how to pass input parameters to stored procedures and use them in queries inside a procedure to provide filtering options.


In the next section, you’ll use output parameters to create procedures returning multiple different values in a single run.


# Creating a Stored Procedure with Input and Output Parameters


In both previous examples, stored procedures you created called a SELECT statement to get a result set. But in some cases, you might need a stored procedure that will return multiple different values together instead of a single result set for an individual query.


Assume you want to create a procedure that will provide summary information about cars from a given year, including the quantity of cars in the collection and their market value (minimum, maximum, and average).


To do so, you can use OUT parameters when creating a new stored procedure. Similar to IN parameters, OUT parameters have names and data types associated with them. However, instead of passing data to the stored procedure, they can be filled with data by the stored procedure to return values to the calling user.


Create a get_car_stats_by_year procedure that will return summary data about the cars from a given production year using output parameters:


```
DELIMITER //
CREATE PROCEDURE get_car_stats_by_year(
    IN year_filter int,
    OUT cars_number int,
    OUT min_value decimal(10, 2),
    OUT avg_value decimal(10, 2),
    OUT max_value decimal(10, 2)
)
BEGIN
    SELECT COUNT(*), MIN(value), AVG(value), MAX(value)
    INTO cars_number, min_value, avg_value, max_value
    FROM cars
    WHERE year = year_filter ORDER BY make, value DESC;
END //
DELIMITER ;


```


This time, alongside the IN parameter year_filter used to filter cars by the production year, four OUT parameters are defined within the parentheses block. The cars_number parameter is represented with int data type and will be used to return the number of cars in the collection. The min_value, avg_value, and max_value parameters represent market value and are defined with the decimal(10, 2) type (similar to the value column in the cars table). These will be used to return information about the cheapest and most expensive cars from the collection, as well as the average price of all matching cars.


The SELECT statement queries four values from the cars table using SQL mathematical functions: COUNT to get the overall number of cars, and MIN, AVG, and MAX to get minimal, average and maximal value from the value column.



Note:
To learn more about using mathematical functions in SQL, you can follow the How To Use Mathematical Expressions and Aggregate Functions in SQL tutorial.

To tell the database that the results of that query should be stored in the output parameters of the stored procedure, a new keyword, INTO is introduced. After the INTO keyword, the names of four procedure parameters corresponding to the retrieved data are listed. With this, MySQL will save the COUNT(*) value into the cars_number parameter, the MIN(value) result into the min_value parameter, and so on.


The database will confirm successful procedure creation:


```
OutputQuery OK, 0 rows affected (0.02 sec)

```


Now, run the new procedure by executing:


```
CALL get_car_stats_by_year(2017, @number, @min, @avg, @max);


```


The four new parameters start with the @ sign. Those are local variable names in the MySQL console that you can use to temporarily store data. When you pass those to the stored procedure you’ve just created, the procedure will insert values into those variables.


The database will respond with:


```
OutputQuery OK, 1 row affected (0.00 sec)

```


That’s different from previous behavior, where the results were immediately displayed on the screen. That’s because the results of the stored procedure have been saved into output parameters and not returned as a query result. To access the results, you can SELECT them directly in the MySQL shell as follows:


```
SELECT @number, @min, @avg, @max;


```


With this query, you’re selecting values from the local variables, not calling the procedure again. The stored procedure saved its results in those variables, and the data will remain available until you disconnect from the shell.



Note:
To learn more about using user-defined variables in MySQL, refer to the User-Defined Variables section in the documentation. When used in application development, the ways to access data returned from stored procedures will differ in different programming languages and frameworks. When in doubt, consult the documentation of your language and framework of choice.

The output will display the values for the queried variables:


```
Output+---------+----------+-----------+-----------+
| @number | @min     | @avg      | @max      |
+---------+----------+-----------+-----------+
|       2 | 48880.00 | 192090.00 | 335300.00 |
+---------+----------+-----------+-----------+
1 row in set (0.00 sec)

```


The values correspond to the number of cars produced in 2017, as well as the minimal, average, and maximum market value of cars from this year of production.


In this example, you’ve learned how to use output parameters to return multiple different values from within the stored procedure for later use. In the next section, you’ll learn how to remove created procedures.


# Removing Stored Procedures


In this section, you’ll remove the stored procedures that are present in the database.


Sometimes the procedure you created may no longer be needed. In other circumstances, you might want to change the way the procedure works. MySQL doesn’t allow changing the procedure definition after creation, so the only way to do so is to remove the procedure first and re-create it with the desired changes.


Let’s remove the last procedure, get_car_stats_by_year. To do so, you can use the DROP PROCEDURE statement:


```
DROP PROCEDURE get_car_stats_by_year;


```


The database will confirm successful procedure deletion with a success message:


```
OutputQuery OK, 0 rows affected (0.02 sec)

```


You can verify that the procedure was deleted by trying to call it. Execute:


```
CALL get_car_stats_by_year(2017, @number, @min, @avg, @max);


```


This time, you’ll see an error message saying the procedure is not present in the database:


```
Error messageERROR 1305 (42000): PROCEDURE procedures.get_car_stats_by_year does not exist

```


In this section, you’ve learned how to delete existing stored procedures in the database.


# Conclusion


By following this guide, you learned what stored procedures are and how to use them in MySQL to save reusable statements into named procedures and execute them later. You created stored procedures without parameters and procedures that use input and output parameters to make them more flexible.


You can use stored procedures to create reusable routines and unify methods for accessing data across multiple applications, as well as implement complex behaviors exceeding the possibilities given by individual SQL queries. This tutorial covered only the basics of using stored procedures. To learn more about that, refer to the MySQL documentation on stored procedures.


If you’d like to learn more about different concepts around the SQL language and working with it, we encourage you to check out the other guides in the How To Use SQL series.


