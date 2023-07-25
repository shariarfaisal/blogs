# How To Import and Export Databases in MySQL or MariaDB

```Databases``` ```MySQL``` ```Ubuntu``` ```MariaDB``` ```Backups``` ```CentOS``` ```Debian``` ```Open Source```

The author selected the Free and Open Source Fund to receive a donation as part of the Write for DOnations program.


## Introduction


Importing and exporting databases is a common task in software development. You can use data dumps to back up and restore your information. You can also use them to migrate data to a new server or development environment.


In this tutorial, you will work with database dumps in MySQL or MariaDB (the commands are interchangeable). Specifically, you will export a database and then import that database from the dump file.


# Prerequisites


To import or export a MySQL or MariaDB database, you will need:


- A virtual machine with a non-root sudo user. If you need a server, go here to create a DigitalOcean Droplet running your favorite Linux distribution. After creation, choose your distribution from this list and follow our Initial Server Setup Guide.
- MySQL or MariaDB installed. To install MySQL, follow our tutorial, How To Install MySQL. To install MariaDB, follow our tutorial, How To Install MariaDB.
- A sample database created in your database server. To create one, follow “Creating a Sample Database” in our tutorial, “An Introduction to Queries in MySQL”.


Note: As an alternative to manual installation, you can explore the DigitalOcean Marketplace’s MySQL One-Click Application.

# Step 1 — Exporting a MySQL or MariaDB Database


The mysqldump console utility exports databases to SQL text files. This makes it easier to transfer and move databases. You will need your database’s name and credentials for an account whose privileges allow at least full read-only access to the database.


Use mysqldump to export your database:


```
mysqldump -u username -p database_name > data-dump.sql


```


- username is the username you can log in to the database with
- database_name is the name of the database to export
- data-dump.sql is the file in the current directory that stores the output.

The command will produce no visual output, but you can inspect the contents of data-dump.sql to check if it’s a legitimate SQL dump file.


Run the following command:


```
head -n 5 data-dump.sql


```


The top of the file should look similar to this, showing a MySQL dump for a database named database_name.


```
SQL dump fragment-- MySQL dump 10.13  Distrib 5.7.16, for Linux (x86_64)
--
-- Host: localhost    Database: database_name
-- ------------------------------------------------------
-- Server version       5.7.16-0ubuntu0.16.04.1

```


If any errors occur during the export process, mysqldump will print them to the screen.


# Step 2 — Importing a MySQL or MariaDB Database


To import an existing dump file into MySQL or MariaDB, you will have to create a new database. This database will hold the imported data.


First, log in to MySQL as root or another user with sufficient privileges to create new databases:


```
mysql -u root -p


```


This command will bring you into the MySQL shell prompt. Next, create a new database with the following command. In this example, the new database is called new_database:


```
CREATE DATABASE new_database;


```


You’ll see this output confirming the database creation.


```
OutputQuery OK, 1 row affected (0.00 sec)

```


Then exit the MySQL shell by pressing CTRL+D. From the normal command line, you can import the dump file with the following command:


```
mysql -u username -p new_database < data-dump.sql


```


- username is the username you can log in to the database with
- newdatabase is the name of the freshly created database
- data-dump.sql is the data dump file to be imported, located in the current directory

If the command runs successfully, it won’t produce any output. If any errors occur during the process, mysql will print them to the terminal instead. To check if the import was successful, log in to the MySQL shell and inspect the data. Selecting the new database with USE new_database and then use SHOW TABLES; or a similar command to look at some of the data.


# Conclusion


In this tutorial you created a database dump from a MySQL or MariaDB database. You then imported that data dump into a new database. mysqldump has additional settings that you can use to alter how the system creates data dumps. You can learn more about from the official mysqldump documentation page.


To learn more about MySQL, check out our MySQL resource page.


To learn more about MySQL queries, check out our tutorial, “An Introduction to Queries in MySQL”.


