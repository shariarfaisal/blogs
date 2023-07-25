# How To Use Primary Keys in SQL

```Development``` ```SQL```

The author selected the Diversity in Tech Fund to receive a donation as part of the Write for DOnations program.


## Introduction


One of the valuable traits of relational databases is molding the data into a well-defined structure. This structure is achieved by using tables with fixed columns, following strictly defined datatypes, and ensuring that each row has the same shape. When you store data as rows in tables, it is equally important to be able to find them and refer to them without ambiguity. In Structured Query Language (SQL), this can be achieved with primary keys, which serve as identifiers for individual rows in the tables in the relational database.


In this tutorial, you’ll learn about primary keys and use a few different kinds to identify unique rows in database tables. Using some sample datasets, you’ll create primary keys on single columns and multiple columns, and autoincrement sequential keys.


# Prerequisites


To follow this guide, you will need a computer running a SQL-based relational database management system (RDBMS). The instructions and examples in this guide were validated using the following environment:


- A server running Ubuntu 20.04, with a non-root user with administrative privileges and a firewall configured with UFW, as described in our initial server setup guide for Ubuntu 20.04.
- MySQL installed and secured on the server, as outlined in How To Install MySQL on Ubuntu 20.04. This guide was verified with a non-root MySQL user, created using the process described in Step 3.
- Basic familiarity with executing SELECT queries to retrieve data from the database as described in our How To SELECT Rows FROM Tables in SQL guide.


Note: Please note that many RDBMSs use their own unique implementations of SQL. Although the commands outlined in this tutorial will work on most RDBMSs and primary keys are a part of the SQL standard, some features are database-specific and thus the exact syntax or output may differ if you test them on a system other than MySQL.

You’ll also need an empty database in which you’ll be able to create tables demonstrating the use of primary keys. We encourage you to go through the following Connecting to MySQL and Setting up a Sample Database section for details on connecting to a MySQL server and creating the testing database used in examples throughout this guide.


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


Create a database named primary_keys:


```
CREATE DATABASE primary_keys;


```


If the database was created successfully, you’ll receive output like this:


```
OutputQuery OK, 1 row affected (0.01 sec)

```


To select the primary_keys database, run the following USE statement:


```
USE primary_keys;


```


You will receive the following output:


```
OutputDatabase changed

```


After selecting the database, you can create sample tables within it. You’re now ready to follow the rest of the guide and begin working with primary keys in MySQL.


# Introduction to Primary Keys


Data in a relational database is stored in tables with a particular, uniform structure of individual rows. The table definition describes what columns there are and what data types can be saved in individual columns. That alone is enough to store the information in the database and find it later using different filtering criteria by using the WHERE clause. However, this structure doesn’t guarantee the possibility of finding any single row unambiguously.


Imagine a database of all registered vehicles allowed to drive on public roads. The database would contain such information as the car brand, model, manufacture year, and paint color. However, if you were looking for a red Chevrolet Camaro made in 2007, you could find more than one. After all, car producers sell similar cars to multiple customers. That’s why registered cars have license plate numbers uniquely identifying each vehicle. If you looked up a car with a license plate of OFP857, you could be sure this criterion will find only one car. That’s because, by law, plate numbers uniquely identify registered vehicles. In a relational database, such a piece of data is called a primary key.


Primary keys are unique identifiers found in a single column or a set of columns that can unambiguously identify every row in a database table. A few rules reflect the technical properties of primary keys:


- A primary key must use unique values. If the primary key consists of more than one column, the combination of values in these columns must be unique across the whole table. Since the key is meant to identify every row uniquely, it can’t appear more than once.
- A primary key must not contain NULL values.
- Each database table can use only one primary key.

The database engine enforces these rules, so you can rely on those properties being true if the primary key is defined on a table.


In addition to these technical properties, you must also consider the content of the data to decide what kind of data is a good choice for becoming a primary key. Natural keys are identifiers already present in the data set, while surrogate keys are artificial identifiers.


Some data structures have primary keys that naturally occur within the data set, such as license plate numbers in a car database or social security numbers in the directory of US citizens. Sometimes such identifiers are not a single value but a pair or a combination of several values. For example, in a local city directory of houses, just a street name or a street number, cannot identify a house uniquely. There can be multiple houses on a single street, and the same number can appear on multiple streets. But a pair of street names and numbers can be assumed to be a unique house identifier. Such naturally occurring identifiers are called natural keys.


However, often data can’t be uniquely characterized by the values of a single column or a small subset of columns. Then, artificial primary keys are created, for example, using a sequence of numbers or randomly generated identifiers like UUIDs. Such keys are called surrogate keys.


In the following sections, you’ll create natural keys based on a single column or multiple columns, and generate surrogate keys on tables where a natural key is not an option.


# Creating a Primary Key on a Single Column


In many cases, a data set naturally contains a single column that can be used to identify rows in the table uniquely. In these cases, you can create a natural key to describe the data. Following the previous example of the database of registered cars, imagine a table with the following structure:


```
Sample table+---------------+-----------+------------+-------+------+
| license_plate | brand     | model      | color | year |
+---------------+-----------+------------+-------+------+
| ABC123        | Ford      | Mustang    | Red   | 2018 |
| CES214        | Ford      | Mustang    | Red   | 2018 |
| DEF456        | Chevrolet | Camaro     | Blue  | 2016 |
| GHI789        | Dodge     | Challenger | Black | 2014 |
+---------------+-----------+------------+-------+------+

```


The first and second rows both describe a red Ford Mustang from 2018. By using only car make and model, you wouldn’t be able to identify the car uniquely. The license plate differs in both instances, providing a good unique identifier for every row in the table. Because a license plate number is already part of the data, using that as a primary key would create a natural key. If you created the table without using a primary key on the license_plate column, you would risk a duplicate or an empty plate appearing in the data set at some point in time.


Next, you’ll create a table resembling the one above with the license_plate column used as a primary key and the following columns:


- license_plate: This column holds the license plate number, represented by the varchar data type.
- brand: This column holds the brand of the car, expressed using the varchar data type with a maximum of 50 characters.
- model: This column holds the car’s model, expressed using the varchar data type with a maximum of 50 characters.
- color: This column holds the color, expressed using the varchar data type with a maximum of 20 characters.
- year: This column holds the year the car was made, expressed using the int data type to store numerical data.

To create the cars table, execute the following SQL statement:


```
CREATE TABLE cars (
    license_plate varchar(8) PRIMARY KEY,
    brand varchar(50),
    model varchar(50),
    color varchar(20),
    year int
);


```


The PRIMARY KEY clause follows the license_plate data type definition. When dealing with primary keys based on single columns, you can use the simplified syntax to create the key, writing PRIMARY KEY in the column definition.


If the following output prints, the table has been created:


```
OutputQuery OK, 0 rows affected (0.00 sec)

```


Following that, load the table with the sample rows presented in the example above by running the following INSERT INTO operation:


```
INSERT INTO cars VALUES
    ('ABC123', 'Ford', 'Mustang', 'Red', 2018),
    ('CES214', 'Ford', 'Mustang', 'Red', 2018),
    ('DEF456', 'Chevrolet', 'Camaro', 'Blue', 2016),
    ('GHI789', 'Dodge', 'Challenger', 'Black', 2014);


```


The database will respond with the success message:


```
OutputQuery OK, 4 rows affected (0.010 sec)
Records: 4  Duplicates: 0  Warnings: 0

```


You can now verify, using the SELECT statement, that the newly created table contains the anticipated data and format:


```
SELECT * FROM cars;


```


The output will show a table similar to the one at the beginning of the section:


```
Output+---------------+-----------+------------+-------+------+
| license_plate | brand     | model      | color | year |
+---------------+-----------+------------+-------+------+
| ABC123        | Ford      | Mustang    | Red   | 2018 |
| CES214        | Ford      | Mustang    | Red   | 2018 |
| DEF456        | Chevrolet | Camaro     | Blue  | 2016 |
| GHI789        | Dodge     | Challenger | Black | 2014 |
+---------------+-----------+------------+-------+------+

```


Next, you can verify whether the rules for the primary key are guaranteed by the database engine. Try inserting a car with a duplicate plate number by executing:


```
INSERT INTO cars VALUES ('DEF456', 'Jeep', 'Wrangler', 'Yellow', 2019);


```


MySQL will respond with an error message, saying that the DEF456 license plate would result in a duplicate entry for the primary key:


```
OutputERROR 1062 (23000): Duplicate entry 'DEF456' for key 'cars.PRIMARY'

```



Note: Under the hood, primary keys are implemented with unique indexes and share many properties with indexes you might create manually for other columns on the table. Most importantly, primary key indexes also improve the performance of querying the table against the column the index is defined on. To learn more about using indexes for that purpose, see the How to Use Indexes guide in this tutorial series.

You can now be sure that duplicate license plates are not allowed. Next, check whether it’s possible to insert a car with an empty license plate:


```
INSERT INTO cars VALUES (NULL, 'Jeep', 'Wrangler', 'Yellow', 2019);


```


This time, the database will respond with another error message:


```
OutputERROR 1048 (23000): Column 'license_plate' cannot be null

```


With these two rules enforced by the database, you can be sure that the license_plate identifies every row in the table uniquely. If you query the table against any license plate, you can expect a single row returned every time.


In the next section, you’ll learn how to use primary keys with multiple columns.


# Creating a Primary Key on Multiple Columns


When one column is not enough to identify a unique row in the table, you can create primary keys that use more than one column.


For example, imagine a registry of homes where neither the street name nor the street number alone is enough to identify any individual house:


```
Sample table+-------------------+---------------+-------------------+------+
| street_name       | street_number | house_owner       | year |
+-------------------+---------------+-------------------+------+
| 5th Avenue        | 100           | Bob Johnson       | 2018 |
| Broadway          | 1500          | Jane Smith        | 2016 |
| Central Park West | 100           | John Doe          | 2014 |
| Central Park West | 200           | Tom Thompson      | 2015 |
| Lexington Avenue  | 5001          | Samantha Davis    | 2010 |
| Park Avenue       | 7000          | Michael Rodriguez | 2012 |
+-------------------+---------------+-------------------+------+

```


The street name Central Park West appears more than once in the table, and so does the street number 100. However, no duplicate pairs of street names and street numbers are visible. In this case, while no single column could be a primary key, the pair of those two values can be used to identify each row in the table uniquely.


Next, you’ll create a table resembling the one shown above with the following columns:


- street_name: This column holds the name of the street where the house is located, represented by the varchar data type limited to 50 characters.
- street_number: This column holds the house’s street number, represented by the varchar data type. This column can store up to 5 characters. It doesn’t use numerical int data type because some street numbers might contain letters (for example, 200B).
- house_owner: This column holds the name of the house’s owner, represented by the varchar data type limited to 50 characters.
- year: This column holds the year the house was built, represented by the int data type to store numerical values.

This time, the primary key will use both the street_name and street_number columns, rather than a single column. To do so, execute the following SQL statement:


```
CREATE TABLE houses (
    street_name varchar(50),
    street_number varchar(5),
    house_owner varchar(50),
    year int,
    PRIMARY KEY(street_name, street_number)
);


```


This time, the PRIMARY KEY clause appears below the column definitions, unlike the previous example. The PRIMARY KEY statement is followed by parentheses with two column names inside: street_name and street_number. This syntax creates the primary key in the houses table spanning two columns.


If the following output prints, the table has been created:


```
OutputQuery OK, 0 rows affected (0.00 sec)

```


Following that, load the table with the sample rows presented in the previous example by running the following INSERT INTO operation:


```
INSERT INTO houses VALUES
    ('Central Park West', '100', 'John Doe', 2014),
    ('Broadway', '1500', 'Jane Smith', 2016),
    ('5th Avenue', '100', 'Bob Johnson', 2018),
    ('Lexington Avenue', '5001', 'Samantha Davis', 2010),
    ('Park Avenue', '7000', 'Michael Rodriguez', 2012),
    ('Central Park West', '200', 'Tom Thompson', 2015);


```


The database will respond with the success message:


```
OutputQuery OK, 6 rows affected (0.000 sec)
Records: 6  Duplicates: 0  Warnings: 0

```


You can now verify, using the SELECT statement, that the newly created table contains the anticipated data and format:


```
SELECT * FROM houses;


```


The output will show a table similar to the one at the beginning of the section:


```
Output+-------------------+---------------+-------------------+------+
| street_name       | street_number | house_owner       | year |
+-------------------+---------------+-------------------+------+
| 5th Avenue        | 100           | Bob Johnson       | 2018 |
| Broadway          | 1500          | Jane Smith        | 2016 |
| Central Park West | 100           | John Doe          | 2014 |
| Central Park West | 200           | Tom Thompson      | 2015 |
| Lexington Avenue  | 5001          | Samantha Davis    | 2010 |
| Park Avenue       | 7000          | Michael Rodriguez | 2012 |
+-------------------+---------------+-------------------+------+
6 rows in set (0.000 sec)

```


Now, let’s verify if the database will allow inserting rows that duplicate street names and street numbers, but restrict duplicate full addresses from appearing in the table. Let’s start by adding another house on Park Avenue:


```
INSERT INTO houses VALUES ('Park Avenue', '8000', 'Emily Brown', 2011);


```


MySQL will respond with a success message since the address 8000 Park Avenue didn’t appear in the table previously:


```
OutputQuery OK, 1 row affected (0.010 sec)

```


A similar result will happen when you add a house on 8000 Main Street, repeating the street number:


```
INSERT INTO houses VALUES ('Main Street', '8000', 'David Jones', 2009);


```


Once again, this will insert a new row properly since the entire address doesn’t duplicate:


```
OutputQuery OK, 1 row affected (0.010 sec)

```


However, try adding another house on 100 5th Avenue using the INSERT statement below:


```
INSERT INTO houses VALUES ('5th Avenue', '100', 'Josh Gordon', 2008);


```


The database will respond with an error message, notifying you that there is a duplicate entry for the primary key for the pair of values 5th Avenue and 100:


```
OutputERROR 1062 (23000): Duplicate entry '5th Avenue-100' for key 'houses.PRIMARY'

```


The database properly enforces the primary key rules, with the key defined on a pair of columns. You can be sure that the full address consisting of a street name and street number won’t get duplicated in the table.


In this section, you created a natural key with a pair of columns to identify every row in the house table uniquely. But it’s not always possible to derive primary keys from the dataset. In the next section, you’ll use artificial primary keys that don’t come directly from the data.


# Creating a Sequential Primary Key


Until now, you’ve created unique primary keys using existing columns in the sample data sets. But in some cases, the data will inevitably duplicate, preventing any columns from being good unique identifiers. In these cases, you can create sequential primary keys using generated identifiers. When the data at your disposal requires you to devise new identifiers to identify rows uniquely, the primary keys created on those artificial identifiers are called surrogate keys.


Imagine a list of book club members—an informal gathering where anyone can join without showing a government-issued ID. There is a chance people with matching names will join the club at some point in time:


```
Sample table+------------+-----------+
| first_name | last_name |
+------------+-----------+
| John       | Doe       |
| Jane       | Smith     |
| Bob        | Johnson   |
| Samantha   | Davis     |
| Michael    | Rodriguez |
| Tom        | Thompson  |
| Sara       | Johnson   |
| David      | Jones     |
| Jane       | Smith     |
| Bob        | Johnson   |
+------------+-----------+

```


The names Bob Johnson and Jane Smith are repeated in the table. You need to use an additional identifier to be sure who is who, and it’s not possible to identify rows uniquely in that table in any way. If you kept the list of book club members on paper, you could keep auxiliary identifiers to help distinguish people with the same names among the group.


You can do a similar thing in a relational database by using an additional column holding generated, factless identifiers with the sole purpose of uniquely separating all rows in the table. Let’s call it member_id.


However, it would be a burden to come up with such an identifier whenever you wanted to add another book club member to the database. To solve this, MySQL provides a feature for auto-incremented numerical columns, where the database automatically provides the column value by an incrementing sequence of integers.


Let’s create a table resembling the one shown above. You’ll add an additional auto-incrementing column (member_id) to hold the automatically assigned number for each club member. That automatically assigned number will act as the primary key for the table:


- member_id: This column holds an auto-incremented, numerical identifier represented by the int data type.
- first_name: This column holds the first name of the club members, represented by the varchar data type limited to 50 characters.
- last_name: This column holds the last name of the club members, represented by the varchar data type limited to 50 characters.

To create the table, execute the following SQL statement:


```
CREATE TABLE club_members (
    member_id int AUTO_INCREMENT PRIMARY KEY,
    first_name varchar(50),
    last_name varchar(50)
);


```


While the PRIMARY KEY clause appears after the column type definition, just like a single column primary key, an additional attribute appears before it: AUTO_INCREMENT. It tells MySQL to auto-generate values for that column, if not explicitly provided, using a growing sequence of numbers.



Note: The AUTO_INCREMENT property for defining a column is specific to MySQL. Other databases often provide similar methods for generating sequential keys, but the syntax differs among engines. When in doubt, we encourage you to refer to the official documentation of your RDBMS.

If the following output prints, the table has been created:


```
OutputQuery OK, 0 rows affected (0.00 sec)

```


Following that, load the table with the sample rows presented in the example above by running the following INSERT INTO operation:


```
INSERT INTO club_members (first_name, last_name) VALUES
    ('John', 'Doe'),
    ('Jane', 'Smith'),
    ('Bob', 'Johnson'),
    ('Samantha', 'Davis'),
    ('Michael', 'Rodriguez'),
    ('Tom', 'Thompson'),
    ('Sara', 'Johnson'),
    ('David', 'Jones'),
    ('Jane', 'Smith'),
    ('Bob', 'Johnson');


```


The INSERT statement now includes the list of column names (first_name and last_name), which ensures the database knows that the member_id column is not supplied in the dataset, so the default value for it should be taken instead.


The database will respond with the success message:


```
OutputQuery OK, 10 rows affected (0.002 sec)
Records: 10  Duplicates: 0  Warnings: 0

```


Use the SELECT statement to verify the data in the newly created table:


```
SELECT * FROM club_members;


```


The output will show a table similar to the one at the beginning of the section:


```
Output+-----------+------------+-----------+
| member_id | first_name | last_name |
+-----------+------------+-----------+
|         1 | John       | Doe       |
|         2 | Jane       | Smith     |
|         3 | Bob        | Johnson   |
|         4 | Samantha   | Davis     |
|         5 | Michael    | Rodriguez |
|         6 | Tom        | Thompson  |
|         7 | Sara       | Johnson   |
|         8 | David      | Jones     |
|         9 | Jane       | Smith     |
|        10 | Bob        | Johnson   |
+-----------+------------+-----------+
10 rows in set (0.000 sec)

```


However, this time, the member_id column appears in the result and contains a sequence of numbers from 1 to 10. With this column in place, duplicate rows for Jane Smith and Bob Johnson are no longer indistinguishable since each name is associated with a unique identifier (member_id).


Now, let’s verify if the database will allow adding another Tom Thompson to the list of club members:


```
INSERT INTO club_members (first_name, last_name) VALUES ('Tom', 'Thompson');


```


MySQL will respond with a success message:


```
OutputQuery OK, 1 row affected (0.009 sec)

```


To check what numerical identifier was assigned to the new entry by the database, execute the SELECT query again:


```
SELECT * FROM club_members;


```


There is one more row in the output:


```
Output+-----------+------------+-----------+
| member_id | first_name | last_name |
+-----------+------------+-----------+
|         1 | John       | Doe       |
|         2 | Jane       | Smith     |
|         3 | Bob        | Johnson   |
|         4 | Samantha   | Davis     |
|         5 | Michael    | Rodriguez |
|         6 | Tom        | Thompson  |
|         7 | Sara       | Johnson   |
|         8 | David      | Jones     |
|         9 | Jane       | Smith     |
|        10 | Bob        | Johnson   |
|        11 | Tom        | Thompson  |
+-----------+------------+-----------+
11 rows in set (0.000 sec)

```


A new row was automatically assigned number 11 in the member_id column through the AUTO_INCREMENT feature of the database.


If the data you are working with doesn’t have natural candidates for primary keys, and you don’t want to provide invented identifiers every time you add new data to the database, you can safely rely on sequentially generated identifiers as primary keys.


# Conclusion


By following this guide, you learned what primary keys are and how to create the common types in MySQL to identify unique rows in database tables. You built natural primary keys, created primary keys spanning multiple columns, and used autoincrementing sequential keys where natural keys are not present.


You can use primary keys to shape the database structure further, ensuring data rows will be uniquely identifiable. This tutorial covered only the basics of using primary keys. To learn more about that, refer to the MySQL documentation on constraints. You can also check out our guides on Understanding SQL Constraints and How To Use Constraints in SQL.


If you’d like to learn more about different concepts around the SQL language and working with it, we encourage you to check out the other guides in the How To Use SQL series.


