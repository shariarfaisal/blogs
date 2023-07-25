# How To Use Triggers in MySQL

```Database``` ```MySQL``` ```SQL```

The author selected the Diversity in Tech Fund to receive a donation as part of the Write for DOnations program.


## Introduction


When working with relational databases and Structured Query Language (SQL), most operations on the data are performed as a result of explicitly executed queries, such as SELECT, INSERT, or UPDATE.


However, SQL databases can also be instructed to perform pre-defined actions automatically every time a specific event occurs through triggers. For instance, you can use triggers to keep the audit trail log of all DELETE statements or automatically update aggregated statistical summaries every time the rows are updated or appended to the table.


In this tutorial, you’ll use different SQL triggers to automatically perform actions where rows are inserted, updated, or deleted.


# Prerequisites


To follow this guide, you will need a computer running a SQL-based relational database management system (RDBMS). The instructions and examples in this guide were validated using the following environment:


- A server running Ubuntu 20.04, with a non-root user with administrative privileges and a firewall configured with UFW, as described in our initial server setup guide for Ubuntu 20.04.
- MySQL installed and secured on the server, as outlined in How To Install MySQL on Ubuntu 20.04. This guide was verified with a non-root MySQL user, created using the process described in Step 3.
- Basic familiarity with executing SELECT, INSERT, UPDATE, and DELETE queries to manipulate data in the database as described in our How To SELECT Rows FROM Tables in SQL, How To Insert Data in SQL, How To Update Data in SQL, and How To Delete Data in SQL guides.
- Basic familiarity with using nested queries as described in our How To Use Nested Queries in SQL guide.
- Basic familiarity with using aggregate mathematical functions as described in our How To Use Mathematical Expressions and Aggregate Functions in SQL guide.


Note: Many RDBMSs use their own implementation of SQL. Although triggers are mentioned as a part of the SQL standard, the standard does not enforce their syntax or the strict way of implementing them. As a result, their implementation differs across different databases. The commands outlined in this tutorial use the syntax for the MySQL database and may not work on other database engines.

You’ll also need a database with some tables loaded with sample data so that you can practice using functions. We encourage you to go through the following Connecting to MySQL and Setting up a Sample Database section for details on connecting to a MySQL server and creating the testing database used in examples throughout this guide.


# Connecting to MySQL and Setting up a Sample Database


In this section, you will connect to a MySQL server and create a sample database so that you can follow the examples in the following sections.


For this guide, you’ll use an imaginary collectibles collection. You’ll store details about currently owned collectibles, keep their total worth readily available, and ensure that removing a collectible always leaves a trace.


If your SQL database system runs on a remote server, SSH into your server from your local machine:


```
ssh sammy@your_server_ip


```


Then open up the MySQL server prompt, replacing sammy with the name of your MySQL user account:


```
mysql -u sammy -p


```


Create a database named collectibles:


```
CREATE DATABASE collectibles;


```


If the database was created successfully, you’ll receive output like this:


```
OutputQuery OK, 1 row affected (0.01 sec)

```


To select the collectibles database, run the following USE statement:


```
USE collectibles;


```


You will receive the following output:


```
OutputDatabase changed

```


After selecting the database, you can create sample tables within it. The table collectibles will contain simplified data about collectibles in the database. It will hold the following columns:


- name: This column holds the name for each collectible, expressed using the varchar data type with a maximum of 50 characters.
- value: This column stores the collectible’s market value using the decimal data type with a maximum of 5 values before the decimal point and 2 values after it.

Create the sample table with the following command:


```
CREATE TABLE collectibles (
    name varchar(50),
    value decimal(5, 2)
);


```


If the following output prints, the table has been created:


```
OutputQuery OK, 0 rows affected (0.00 sec)

```


The next table will be called collectibles_stats and will be used to keep track of the accumulated worth of all the collectibles in the collection. It will hold a single row of data with the following columns:


- count: This column holds the number of owned collectibles, expressed using the int data type.
- value: This column stores the accumulated worth of all collectibles using the decimal data type with a maximum of 5 values before the decimal point and 2 values after it.

Create the sample table with the following command:


```
CREATE TABLE collectibles_stats (
    count int,
    value decimal(5, 2)
);


```


If the following output prints, the table has been created:


```
OutputQuery OK, 0 rows affected (0.00 sec)

```


The third and last table will be called collectibles_archive, which will keep track of all the collectibles that have been removed from the collection to ensure they never vanish. It will hold data similar to the collectibles table, augmented with the removal date. It uses the following columns:


- name: This column holds the name for each removed collectible, expressed using the varchar data type with a maximum of 50 characters.
- value: This column stores the collectible’s market value at the moment of deletion using the decimal data type with a maximum of 5 values before the decimal point and 2 values after it.
- removed_on: This column stores the date and time of deletion for each archived collectible using the timestamp data type with the default value of NOW(), meaning the current date whenever a new row is inserted into this table.

Create the sample table with the following command:


```
CREATE TABLE collectibles_archive (
    name varchar(50),
    value decimal(5, 2),
    removed_on timestamp DEFAULT CURRENT_TIMESTAMP
);


```


If the following output prints, the table has been created:


```
OutputQuery OK, 0 rows affected (0.00 sec)

```


Following that, load the collectibles_stats table with the initial state for the empty collectibles collection by running the following INSERT INTO operation:


```
INSERT INTO collectibles_stats SELECT COUNT(name), SUM(value) FROM collectibles;


```


The INSERT INTO operation will add a single row to the collectibles_stats with the values calculated using the aggregate functions to count all rows in the collectibles table and to sum the worth of all collectibles using the value column and the SUM function. The following output indicates that the row has been added:


```
OutputQuery OK, 1 row affected (0.002 sec)
Records: 1  Duplicates: 0  Warnings: 0

```


You can verify that by executing a SELECT statement on the table:


```
SELECT * FROM collectibles_stats;


```


Since there are no collectibles in the database yet, the initial number of items is 0 and the accumulated value says NULL:


```
Output+-------+-------+
| count | value |
+-------+-------+
|     0 |  NULL |
+-------+-------+
1 row in set (0.000 sec)

```


With that, you’re ready to follow the rest of the guide and begin using triggers in MySQL.


# Understanding Triggers


Triggers are statements defined for a particular table that get executed automatically by the database every time a specified event occurs in that table. Triggers can be used to guarantee that some actions will be performed consistently every time a specific statement is executed on a table, rather than database users needing to remember to execute them manually.


Every trigger associated with a table is identified with a user-defined name and a pair of conditions that instruct the database engine when to execute the trigger. Those can be grouped into two separate classes:


- Database event: The trigger can be executed when INSERT, UPDATE, or DELETE statements are run on a table.
- Event time: Additionally, triggers can be executed BEFORE or AFTER the statement in question.

Combining the two condition groups yields a total of six separate trigger possibilities that are executed automatically each time the joint condition is met. The triggers that happen before the statement meeting the condition is executed are BEFORE INSERT, BEFORE UPDATE, and BEFORE DELETE. These can be used to manipulate and validate data before it gets inserted or updated into the table, or to save the details of the deleted row for auditing or archival purposes.


The triggers that happen after the statement meeting the condition is executed are AFTER INSERT, AFTER UPDATE, and AFTER DELETE. These can be used to update summarized values in a separate table based on the final state of the database after the statement.


To perform actions such as validating and manipulating the input data or archiving the deleted row, the database allows accessing data values from within the triggers. For INSERT triggers, only the newly inserted data can be used. For UPDATE triggers, both the original and updated data can be accessed. Finally, with DELETE triggers, only the original row data is available to use (since there is no new data to refer to).


The data for use within the trigger body is exposed under the OLD record for the data currently in the database and the NEW record for the data the query will save. You can refer to individual columns using the syntax OLD.column_name and NEW.column_name.


The following example shows the general syntax of an SQL statement used to create a new trigger:


```
CREATE TRIGGER trigger_name trigger_condition
ON table_name
FOR EACH ROW
trigger_actions;


```


Let’s dissect the syntax into smaller parts:


- CREATE TRIGGER is the name of the SQL statement used to create a new trigger in the database.
- trigger_name is the user-defined name of the trigger, used to describe its role, similar to how table names and column names are used to describe their meaning.
- ON table_name tells the database that the trigger should monitor events happening on the table_name table.
- trigger_condition is one of the six possible choices defining when the trigger should run, for example, BEFORE INSERT.
- FOR EACH ROW tells the database that the trigger should be run for each row affected by the triggering event. Some databases support additional patterns of execution other than FOR EACH ROW; however, in the case of MySQL, running the statements from the trigger body for each row affected by the statement that caused the trigger to execute is the only option.
- trigger_actions is the trigger’s body and defines what happens when the trigger executes. It’s typically a single valid SQL statement. It is possible to include multiple statements in the trigger body to perform complex data operations using the BEGIN and END keywords to enclose the list of statements in a block.  This is, however, out of the scope of this tutorial. Check out the official documentation for triggers to learn more about the syntax used to define triggers.

In the following section, you will create triggers that manipulate data before INSERT and UPDATE operations.


# Manipulating Data with BEFORE INSERT and BEFORE UPDATE Triggers


In this section, you will use triggers to manipulate data before INSERT and UPDATE statements are executed.


In this example, you’ll use triggers to ensure that all collectibles in the database use uppercase names for consistency. Without using triggers, you would have to remember to use uppercase collectible names for each INSERT and UPDATE statement. If you forget, the database will save the data as-is, leading to possible mistakes in the dataset.


You’ll start by inserting an example collectible item called spaceship model worth $12.50. The item name will be written in lowercase to illustrate the issue. Execute the following statement:


```
INSERT INTO collectibles VALUES ('spaceship model', 12.50);


```


The following message confirms the item was added:


```
OutputQuery OK, 1 row affected (0.009 sec)

```


You can verify that the row was inserted by executing the SELECT query:


```
SELECT * FROM collectibles;


```


The following output will print to the screen:


```
Output+-----------------+-------+
| name            | value |
+-----------------+-------+
| spaceship model | 12.50 |
+-----------------+-------+
1 row in set (0.000 sec)

```


The collectible item has been saved as-is, with the name spelled with only lowercase letters.


To ensure that all future collectibles will always be written in uppercase, you’ll create a BEFORE INSERT trigger. Using a trigger that executes before the triggering statement is run allows you to manipulate the data that will be passed to the database before it happens.


Run the following statement:


```
CREATE TRIGGER uppercase_before_insert BEFORE INSERT
ON collectibles
FOR EACH ROW
SET NEW.name = UPPER(NEW.name);


```


This command creates a trigger named uppercase_before_insert that will be executed BEFORE all INSERT statements on the table named collectibles.


The statement in the trigger SET NEW.name = UPPER(NEW.name) will be executed for each inserted row. The SET SQL command assigns the value on the right side to the left side. In this case, NEW.name represents the value of the name column that the insertion statement will save. By applying the UPPER function on the collectible name and assigning it back to the column value, you are converting the letter case of the value that will be saved in the database.



Note: When running the CREATE TRIGGER command, you may encounter an error message similar to ERROR 1419 (HY000): You do not have the SUPER privilege, and binary logging is enabled (you *might* want to use the less safe log_bin_trust_function_creators variable).
Starting with MySQL 8, the MySQL database engine has binary logging enabled by default unless local installation configuration overrides that. The binary log keeps track of all SQL statements that modify database contents in the form of saved events describing the modifications. These logs are used in database replication to keep database replicas in sync and during point-in-time data recovery.
With binary logging enabled, MySQL disallows the creation of triggers and stored procedures as a precaution to guarantee data safety and integrity in replicated environments. Understanding how triggers and stored procedures can affect replication is out of the scope of this guide.
However, in a local environment and for learning purposes, you can safely override the way MySQL guards against creating triggers. The overridden setting is not persisted and will return to the original value when the MySQL server is restarted.
To override the default setting for binary logging, log in to MySQL as root and execute the following command:
SET GLOBAL log_bin_trust_function_creators = 1;


The log_bin_trust_function_creators setting controls whether users who create triggers and stored functions can be trusted not to create triggers causing unsafe events to be written to the binary log. By default, the setting’s value is 0, allowing only superusers to create triggers in an environment with binary logging enabled. By changing the value to 1, any user issuing CREATE TRIGGER statements will be trusted to understand the implications.
After updating the setting, log out as root, log back in as the user, and rerun the CREATE TRIGGER statement.
To learn more about binary logging and replication in MySQL and how it relates to triggers, we encourage you to refer to the official MySQL documentation: The Binary Log and Stored Program Binary Logging. You can also check out our tutorial, How to Set Up Replication in MySQL.
Before using triggers in a production environment with replication in place, or stringent point-in-time recovery requirements, make sure you have weighed their impact on binary log consistency.


Note: Depending on your MySQL user permissions, you may receive an error when executing the CREATE TRIGGER command: ERROR 1142 (42000): TRIGGER command denied to user 'user'@'host' for table 'collectibles'. To grant TRIGGER permissions to your user, log in to MySQL as root and execute the following commands, replacing the MySQL username and host as needed:
GRANT TRIGGER on *.* TO 'sammy'@'localhost';
FLUSH PRIVILEGES;


After updating the user permissions, log out as root, log back in as the user, and rerun the CREATE TRIGGER statement.

MySQL will print the following message to confirm that the trigger was created successfully:


```
OutputQuery OK, 1 row affected (0.009 sec)

```


Now try to insert a new collectible, again using a lowercase argument to the INSERT query:


```
INSERT INTO collectibles VALUES ('aircraft model', 10.00);


```


And once again, check the resulting rows in the collectibles table:


```
SELECT * FROM collectibles;


```


The following output will print to the screen:


```
Output+-----------------+-------+
| name            | value |
+-----------------+-------+
| spaceship model | 12.50 |
| AIRCRAFT MODEL  | 10.00 |
+-----------------+-------+
2 rows in set (0.000 sec)

```


This time, however, the new entry says AIRCRAFT MODEL with all letters in uppercase — different than the entry you tried to insert. The trigger ran in the background and converted the letter case before the row was saved in the database.


All new rows are now guarded by the trigger to ensure the names will be saved in uppercase. However, it is still possible to save unrestricted data using UPDATE statements. To guard UPDATE statements with the same effect, create another trigger:


```
CREATE TRIGGER uppercase_before_update BEFORE UPDATE
ON collectibles
FOR EACH ROW
SET NEW.name = UPPER(NEW.name);


```


The difference between the two triggers is in the trigger criteria. This time, it’s BEFORE UPDATE, meaning the trigger will execute each time an UPDATE statement is issued on the table — affecting existing rows on every update, in addition to new rows covered by the previous trigger.


MySQL will output a confirmation that the trigger was created successfully:


```
OutputQuery OK, 0 row affected (0.009 sec)

```


To verify the behavior of the new trigger, try updating the price value for the spaceship model:


```
UPDATE collectibles SET value = 15.00 WHERE name = 'spaceship model';


```


The WHERE clause filters the row to be updated by name, and the SET clause changes the value to 15.00.


You’ll receive the following output, confirming that the statement changed a single row:


```
OutputQuery OK, 1 row affected (0.002 sec)
Rows matched: 1  Changed: 1  Warnings: 0

```


Check the resulting rows in the collectibles table:


```
SELECT * FROM collectibles;


```


The following output will print to the screen:


```
Output+-----------------+-------+
| name            | value |
+-----------------+-------+
| SPACESHIP MODEL | 15.00 |
| AIRCRAFT MODEL  | 10.00 |
+-----------------+-------+
2 rows in set (0.000 sec)

```


Now, in addition to the price updating to 15.00 by the executed statement, the name now says SPACESHIP MODEL. When you ran the UPDATE statement, the trigger was executed, affecting the values on the updated row. The name column was converted into uppercase before saving.


In this section, you created two triggers working before INSERT and before UPDATE queries to conform data before saving it to the database. In the next section, you will use BEFORE DELETE triggers to copy deleted rows into a separate table for archiving.


# Using BEFORE DELETE Triggers to Execute Actions Before Deleting Rows


Even if you no longer own an item, you might want to leave an entry about the deletion in a separate table. At the beginning of this tutorial, you created a second table called collectibles_archive to keep track of all the collectibles that have been removed from the collection. In this section, you’ll archive deleted entries with a trigger that will execute before DELETE statements.


Check if the archive table is fully empty by executing the following statement:


```
SELECT * FROM collectibles_archive;


```


The following output will print to the screen, confirming the collectibles_archive table is empty:


```
OutputEmpty set (0.000 sec)

```


Now, if you issue a DELETE query against the collectibles table, any row from the table could be deleted without a trace.


To remedy that, you’ll create a trigger that will execute before all DELETE queries on the collectibles table. The purpose of this trigger is to save a copy of the deleted object to the archive table before the deletion happens.


Run the following command:


```
CREATE TRIGGER archive_before_delete BEFORE DELETE
ON collectibles
FOR EACH ROW
INSERT INTO collectibles_archive (name, value) VALUES (OLD.name, OLD.value);


```


The trigger is named archive_before_delete and happens BEFORE any DELETE queries on the collectibles table. For each row that will be deleted, the INSERT statement will be executed. In turn, the INSERT statement inserts a new row into the collectibles_archive table with data values taken from the OLD record, which is the one slated for deletion: OLD.name becomes the name column and OLD.value becomes the value column.


The database will confirm the creation of the trigger:


```
OutputQuery OK, 0 row affected (0.009 sec)

```


With the trigger in place, try deleting a collectible from the main collectibles table:


```
DELETE FROM collectibles WHERE name = 'SPACESHIP MODEL';


```


The output confirms that the query ran successfully:


```
OutputQuery OK, 1 row affected (0.004 sec)

```


Now, list all the collectibles:


```
SELECT * FROM collectibles;


```


The following output will print to the screen:


```
Output+----------------+-------+
| name           | value |
+----------------+-------+
| AIRCRAFT MODEL | 10.00 |
+----------------+-------+
1 row in set (0.000 sec)

```


Only AIRCRAFT MODEL now remains; the SPACESHIP MODEL has been deleted and is no longer in the table. However, with the previously created trigger, this deletion should be registered in the collectibles_archive table. Let’s check that.


Execute another query:


```
SELECT * FROM collectibles_archive;


```


The following output will print to the screen:


```
Output+-----------------+-------+---------------------+
| name            | value | removed_on          |
+-----------------+-------+---------------------+
| SPACESHIP MODEL | 15.00 | 2022-11-20 11:32:01 |
+-----------------+-------+---------------------+
1 row in set (0.000 sec)

```


The deletion was automatically noted in that table by the trigger. The name and value columns have been filled with data from the row that was deleted. The third column, removed_on, is not explicitly set through the defined trigger, so it takes the default value decided during table creation: the date of any new row’s creation. Because of that, every entry added with the help of the trigger will be always annotated with the deletion date.


With this trigger in place, you can now be sure that all DELETE queries will result in a log entry in collectibles_archive, leaving behind information about previously owned collectibles.


In the next section, you will use triggers executed after the triggering statements to update the summary table with aggregated values based on all the collectibles.


# Using AFTER INSERT, AFTER UPDATE, and AFTER DELETE Triggers to Execute Actions After Data Manipulation


In both previous sections, you used triggers executed before the main statements to perform operations based on the original data before updating the database. In this section, you will update the summary table with an always up-to-date count and the accumulated worth of all collectibles using triggers that execute after the intended statements. This way, you will be sure the summary table data takes into account the current state of the database.


Start by examining the collectibles_stats table:


```
SELECT * FROM collectibles_stats;


```


Since you have not yet added information to this table, the number of owned collectible items is 0, and thus, the accumulated value is NULL:


```
Output+-------+-------+
| count | value |
+-------+-------+
|     0 |  NULL |
+-------+-------+
1 row in set (0.000 sec)

```


Since there are no triggers for this table, previously issued queries to insert and update collectibles didn’t affect this table.


The goal is to set the values in a single row in the collectibles_stats table to present up-to-date information about the collectibles’ count and total worth. You want to ensure that the table contents are updated after every INSERT, UPDATE, or DELETE operation.


You can do that by creating three separate triggers, all executed after the corresponding query. First, create the AFTER INSERT trigger:


```
CREATE TRIGGER stats_after_insert AFTER INSERT
ON collectibles
FOR EACH ROW
UPDATE collectibles_stats
SET count = (
    SELECT COUNT(name) FROM collectibles
), value = (
    SELECT SUM(value) FROM collectibles
);


```


The trigger is named stats_after_insert and will execute AFTER every INSERT query to the collectibles table, running the UPDATE statement in the trigger body. The UPDATE query affects the collectibles_stats and sets the count and value columns to the values returned by nested queries:


- SELECT COUNT(name) FROM collectibles will get the collectibles count.
- SELECT SUM(value) FROM collectibles will get the total worth of all collectibles.

The database will confirm the creation of the trigger:


```
OutputQuery OK, 0 row affected (0.009 sec)

```


Now, try to re-insert the previously deleted spaceship model into the collectibles table to check whether the summary table will be properly updated:


```
INSERT INTO collectibles VALUES ('spaceship model', 15.00);


```


The database will print the following success message:


```
OutputQuery OK, 1 row affected (0.009 sec)

```


You can list all owned collectibles by running:


```
SELECT * FROM collectibles;


```


The following output will print to the screen:


```
Output+-----------------+-------+
| name            | value |
+-----------------+-------+
| AIRCRAFT MODEL  | 10.00 |
| SPACESHIP MODEL | 15.00 |
+-----------------+-------+
2 rows in set (0.000 sec)

```


There are two collectible items worth 25.00 in total. To examine the summary table after the newly inserted item, execute the following query:


```
SELECT * FROM collectibles_stats;


```


This time, the summary table will list the number of all owned collectible items as 2 and the accumulated value as 25.00, which matches the previous output:


```
Output+-------+-------+
| count | value |
+-------+-------+
|     2 | 25.00 |
+-------+-------+
1 row in set (0.000 sec)

```


The stats_after_insert trigger runs after the INSERT query and updates the collectibles_stats table with current data (count and value) about the collection. Statistics are gathered about the whole collection contents, not just the last insert. Since the collection now contains two items (aircraft and spaceship models), the summary table lists two items and their summed value. At this point, adding any new collectible item to the collectibles table will update the summary table with the correct values.


However, updating existing items or deleting collectibles won’t affect the summary at all. To fill that gap, you’ll create two additional triggers, performing identical operations but triggered by different events:


```
CREATE TRIGGER stats_after_update AFTER UPDATE
ON collectibles
FOR EACH ROW
UPDATE collectibles_stats
SET count = (
    SELECT COUNT(name) FROM collectibles
), value = (
    SELECT SUM(value) FROM collectibles
);

CREATE TRIGGER stats_after_delete AFTER DELETE
ON collectibles
FOR EACH ROW
UPDATE collectibles_stats
SET count = (
    SELECT COUNT(name) FROM collectibles
), value = (
    SELECT SUM(value) FROM collectibles
);


```


You have now created two new triggers: stats_after_update and stats_after_delete. Both triggers will run on the collectible_stats table whenever you run an UPDATE or DELETE statement on the collectibles table.


The successful creation of those triggers will print the following output:


```
OutputQuery OK, 0 row affected (0.009 sec)

```


Now, update the price value for one of the collectibles:


```
UPDATE collectibles SET value = 25.00 WHERE name = 'AIRCRAFT MODEL';


```


The WHERE clause filters the row to be updated by name, and the SET clause changes the value to 25.00.


The output confirms that the statement changed a single row:


```
OutputQuery OK, 1 row affected (0.002 sec)
Rows matched: 1  Changed: 1  Warnings: 0

```


Once again, check the contents of the summary table after the update:


```
SELECT * FROM collectibles_stats;


```


The value now lists 40.00, which is the correct value after the update:


```
Output+-------+-------+
| count | value |
+-------+-------+
|     2 | 40.00 |
+-------+-------+
1 row in set (0.000 sec)

```


The last step is to verify the summary table will properly reflect deleting a collectible. Try deleting the aircraft model with the following statement:


```
DELETE FROM collectibles WHERE name = 'AIRCRAFT MODEL';


```


The following output confirms that the query ran successfully:


```
OutputQuery OK, 1 row affected (0.004 sec)

```


Now, list all the collectibles:


```
SELECT * FROM collectibles;


```


The following output will print to the screen:


```
Output+-----------------+-------+
| name            | value |
+-----------------+-------+
| SPACESHIP MODEL | 15.00 |
+-----------------+-------+
1 row in set (0.000 sec)

```


Only SPACESHIP MODEL now remains. Next, check the values in the summary table:


```
SELECT * FROM collectibles_stats;


```


The following output will print:


```
Output+-------+-------+
| count | value |
+-------+-------+
|     1 | 15.00 |
+-------+-------+
1 row in set (0.000 sec)

```


The count column now shows that only one collectible in the main table. The total value is 15.00, matching the value of the SPACESHIP MODEL.


These three triggers work jointly after INSERT, UPDATE, and DELETE queries to keep the summary table in sync with the complete list of collectibles.


In the next section, you will learn how to manipulate existing triggers on the database.


# Listing and Deleting Triggers


In the previous sections, you created new triggers. Since triggers are named objects defined on the database, just like tables, you can also list them and manipulate them when needed.


To list all triggers, execute the SHOW TRIGGERS statement:


```
SHOW TRIGGERS;


```


The output will include all triggers, including their names, triggering event with time (BEFORE or AFTER statement execution), as well as statements that are part of the trigger body and other extensive details of the trigger definition:


```
Output, simplified for readability+-------------------------+--------+--------------+--------(...)+--------+(...)
| Trigger                 | Event  | Table        | Statement   | Timing |(...)
+-------------------------+--------+--------------+--------(...)+--------+(...)
| uppercase_before_insert | INSERT | collectibles | SET    (...)| BEFORE |(...)
| stats_after_insert      | INSERT | collectibles | UPDATE (...)| AFTER  |(...)
| uppercase_before_update | UPDATE | collectibles | SET    (...)| BEFORE |(...)
| stats_after_update      | UPDATE | collectibles | UPDATE (...)| AFTER  |(...)
| archive_before_delete   | DELETE | collectibles | INSERT (...)| BEFORE |(...)
| stats_after_delete      | DELETE | collectibles | UPDATE (...)| AFTER  |(...)
+-------------------------+--------+--------------+--------(...)+--------+(...)
6 rows in set (0.001 sec)

```


To delete existing triggers, you can use DROP TRIGGER SQL statements. Perhaps you no longer want to enforce uppercase letters for collectible names, so the uppercase_before_insert and uppercase_before_update are no longer needed. Execute the following commands to remove these two triggers:


```
DROP TRIGGER uppercase_before_insert;
DROP TRIGGER uppercase_before_update;


```


For both commands, MySQL will respond with a success message:


```
OutputQuery OK, 0 rows affected (0.004 sec)

```


Now, with the two triggers gone, let’s add a new collectible in lowercase:


```
INSERT INTO collectibles VALUES ('ship model', 10.00);


```


The database will confirm the insertion:


```
OutputQuery OK, 1 row affected (0.009 sec)

```


You can verify that the row was inserted by executing the SELECT query:


```
SELECT * FROM collectibles;


```


The following output will print to the screen:


```
Output+-----------------+-------+
| name            | value |
+-----------------+-------+
| SPACESHIP MODEL | 15.00 |
| ship model      | 10.00 |
+-----------------+-------+
2 rows in set (0.000 sec)

```


The newly added collectible is in lowercase letters. Since the name is unaltered from the original output, you have confirmed that the trigger that previously converted the letter case is no longer in use.


You now know how to list and delete triggers by name.


# Conclusion


By following this guide, you learned what SQL triggers are and how to use them in MySQL to manipulate data before INSERT and UPDATE queries. You learned how to use BEFORE DELETE trigger to archive the deleted row to a separate table, as well as to use AFTER statement triggers to keep the summaries consistently up to date.


You can use functions to offload some of the data manipulation and validation to the database engine, ensuring data integrity or hiding some of the database behaviors from the daily database user. This tutorial covered only the basics of using triggers for that purpose. You can build complex triggers consisting of multiple statements and use conditional logic to perform actions even more granularly. To learn more about that, refer to the MySQL documentation on triggers.


If you’d like to learn more about different concepts around the SQL language and working with it, we encourage you to check out the other guides in the How To Use SQL series.


