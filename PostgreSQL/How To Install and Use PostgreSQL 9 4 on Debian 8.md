# How To Install and Use PostgreSQL 9 4 on Debian 8

```PostgreSQL``` ```Debian```

## Introduction


Relational databases are the cornerstone of data organization for a multitude of needs. They power everything from online shopping to rocket launches. A database that is both venerable but very much still in the game is PostgreSQL. PostgreSQL follows most of the SQL standard, has ACID transactions, has support for foreign keys and views, and is still in active development.


If the application that you’re running needs stability, package quality, and easy administration, Debian 8 (codename “Jessie”) is one of the best candidates for a Linux distribution. It moves a bit more slowly than other “distros,” but its stability and quality is well recognized. If your application or service needs a database, the combination of Debian 8 and PostgreSQL is one of the best in town.


In this article we will show you how to install PostgreSQL on a new Debian 8 Stable instance and get started.


# Prerequisites


The first thing is to get a Debian 8 Stable system going. You can follow the instructions from the Initial Server Setup with Debian 8 article. This tutorial assumes you have aø Debian 8 Stable Droplet ready.


Except otherwise noted, all of the commands in this tutorial should be run as a non-root user with sudo privileges. To learn how to create users and grant them sudo privileges, check out Initial Server Setup with Debian 8.


# Installing PostgreSQL


Before installing PostgreSQL, make sure that you have the latest information from the Debian repositories by updating the apt package list with:


```
sudo apt-get update


```


You should see the package lists being updated and the following message:


```
Reading package lists... Done.

```


There are several packages that start with postgresql:


- postgresql-9.4: The PostgreSQL server package
- postgresql-client-9.4: The client for PostgreSQL
- postgresql: A “metapackage” better explained by the Debian Handbook or the Debian New Maintainers’ Guide

To install directly the postgresql-9.4 package:


```
sudo apt-get install postgresql-9.4 postgresql-client-9.4


```


When asked, type Y to install the packages. If everything went fine, the packages are now downloaded from the repository and installed.


# Checking the Installation


To check that the PostgreSQL server was correctly installed and is running, you can use the command ps:


```
# ps -ef | grep postgre

```


You should see something like this on the terminal:


```
postgres 32164     1  0 21:58 ?        00:00:00 /usr/lib/postgresql/9.4/bin/postgres -D /var/lib/   postgresql/9.4/main -c config_file=/etc/postgresql/9.4/main/postgresql.conf
postgres 32166 32164  0 21:58 ?        00:00:00 postgres: checkpointer process
postgres 32167 32164  0 21:58 ?        00:00:00 postgres: writer process
postgres 32168 32164  0 21:58 ?        00:00:00 postgres: wal writer process
postgres 32169 32164  0 21:58 ?        00:00:00 postgres: autovacuum launcher process
postgres 32170 32164  0 21:58 ?        00:00:00 postgres: stats collector process 

```


Success! PostgreSQL has been successfully installed and is running.


# Accessing the PostgreSQL Database


On Debian, PostgreSQL is installed with a default user and default database both called postgres. To connect to the database, first you need to switch to the postgres user by issuing the following command while logged in as root (this will not work with sudo access):


```
su - postgres


```


You now should be logged as postgres. To start the PostgreSQL console, type psql:


```
psql


```


Done! You should be logged on the PostgreSQL console. You should see the following prompt:


```
psql (9.4.2)
Type "help" for help.

postgres=# 

```


To exit the psql console just use the command \q.


# Creating New Roles


By default, Postgres uses a concept called “roles” to aid in authentication and authorization. These are, in some ways, similar to regular Unix-style accounts, but PostgreSQL does not distinguish between users and groups and instead prefers the more flexible term “role”.


Upon installation PostgreSQL is set up to use “ident” authentication, meaning that it associates PostgreSQL roles with a matching Unix/Linux system account. If a PostgreSQL role exists, it can be signed in by logging into the associated Linux system account.


The installation procedure created a user account called postgres that is associated with the default Postgres role.


To create additional roles we can use the createuser command. Mind that this command should be issued as the user postgres,  not inside the PostgreSQL console:


```
createuser --interactive


```


This basically is an interactive shell script that calls the correct PostgreSQL commands to create a user to your specifications. It will ask you some questions: the name of the role, whether it should be a superuser, if the role should be able to create new databases, and if the role will be able to create new roles. The man page has more information:


```
man createuser


```


# Creating a New Database


PostgreSQL is set up by default with authenticating roles that are requested by matching system accounts. (You can get more information about this at postgresql.org). It also comes with the assumption that a matching database will exist for the role to connect to. So if I have a user called test1, that role will attempt to connect to a database called test1 by default.


You can create the appropriate database by simply calling this command as the postgres user:


```
createdb test1


```


The new database test1 now is created.


# Connecting to PostgreSQL with the New User


Let’s assume that you have a Linux account named test1, created a PostgreSQL test1 role to match it, and created the database test1. To change the user account in Linux to test1:


```
su - test1


```


Then, connect to the test1 database as the test1 PostgreSQL role using the command:


```
psql


```


Now you should see the PostgreSQL prompt with the newly created user test1 instead of postgres.


# Creating and Deleting Tables


Now that you know how to connect to the PostgreSQL database system, we will start to go over how to complete some basic tasks.


First, let’s create a table to store some data. Let’s create a table that describes playground equipment.


The basic syntax for this command is something like this:


```
CREATE TABLE table_name (
    column_name1 col_type (field_length) column_constraints,
    column_name2 col_type (field_length),
    column_name3 col_type (field_length)
);

```


As you can see, we give the table a name, and then define the columns that we want, as well as the column type and the max length of the field data. We can also optionally add table constraints for each column.


You can learn more about how to create and manage tables in Postgres in the How To Create, Remove, & Manage Tables in PostgreSQL on a Cloud Server article.


For our purposes, we’re going to create a simple table like this:


```
CREATE TABLE playground (
    equip_id serial PRIMARY KEY,
    type varchar (50) NOT NULL,
    color varchar (25) NOT NULL,
    location varchar(25) check (location in ('north', 'south', 'west', 'east', 'northeast', 'southeast', 'southwest', 'northwest')),
    install_date date
);

```


We have made a playground table that inventories the equipment that we have. This starts with an equipment ID, which is of the serial type. This data type is an auto-incrementing integer. We have given this column the constraint of primary key which means that the values must be unique and not null.


For two of our columns, we have not given a field length. This is because some column types don’t require a set length because the length is implied by the type.


We then give columns for the equipment type and color, each of which cannot be empty. We then create a location column and create a constraint that requires the value to be one of eight possible values. The last column is a date column that records the date that we installed the equipment.


To see the tables, use the command \dt on the psql prompt. The result would be similar to


```
             List of relations
 Schema |    Name    | Type  |  Owner 
--------+------------+-------+----------
 public | playground | table | postgres

```


As you can see, we have our playground table.


# Adding, Querying, and Deleting Data in a Table


Now that we have a table created, we can insert some data into it.


Let’s add a slide and a swing. We do this by calling the table we’re wanting to add to, naming the columns and then providing data for each column. Our slide and swing could be added like this:


```
INSERT INTO playground (type, color, location, install_date) VALUES ('slide', 'blue', 'south', '2014-04-28');
INSERT INTO playground (type, color, location, install_date) VALUES ('swing', 'yellow', 'northwest', '2010-08-16');

```


You should notice a few things. First, keep in mind that the column names should not be quoted, but the column values that you’re entering do need quotes.


Another thing to keep in mind is that we do not enter a value for the equip_id column. This is because this is auto-generated whenever a new row in the table is created.


We can then get back the information we’ve added by typing:


```
SELECT * FROM playground;

```


The output should be


```
 equip_id | type  | color  | location  | install_date 
----------+-------+--------+-----------+--------------
        1 | slide | blue   | south     | 2014-04-28
        2 | swing | yellow | northwest | 2010-08-16

```


Here, you can see that our equip_id has been filled in successfully and that all of our other data has been organized correctly. If our slide breaks, and we remove it from the playground, we can also remove the row from our table by typing:


```
DELETE FROM playground WHERE type = 'slide';

```


If we query our table again:


```
SELECT * FROM playground;

```


We will see our slide is no longer a part of the table:


```
 equip_id | type  | color | location | install_date 
----------+-------+-------+----------+--------------
        1 | slide | blue  | south    | 2014-04-28

```


# Useful Commands


Here are a few commands that can help you get an idea of your current environment:


- 
?: Get a full list of psql commands, including those not listed here.

- 
\h: Get help on SQL commands. You can follow this with a specific command to get help with the syntax.

- 
\q: Quit the psql program and exit to the Linux prompt.

- 
\d: List available tables, views, and sequences in current database.

- 
\du: List available roles

- 
\dp: List access privileges

- 
\dt: List tables

- 
\l: List databases

- 
\c: Connect to a different database. Follow this by the database name.

- 
\password: Change the password for the username that follows.

- 
\conninfo: Get information about the current database and connection.


With these commands you should be able to navigate the PostgreSQL databases, tables, and roles in no time.


# Conclusion


You should now have a fully functional PostgreSQL database up and running on your Debian system. Congratulations! There is a plethora of documentation to go from here:


- 
PostgreSQL Manuals

- 
Installing the package postgresql-doc: sudo apt-get install postgresql-doc

- 
README file installed at  /usr/share/doc/postgresql-doc-9.4/tutorial/README


For a full list of supported SQL commands in PostgreSQL follow this link:


- SQL Commands

To compare the different functionalities of the databases you can check out:


- SQLite vs MySQL vs PostgreSQL

For a better understanding of roles and permissions see:


- How to use Roles and Manage Grant Permission in PostgreSQL on a VPS

