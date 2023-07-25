# How To Install and Use PostgreSQL on CentOS 7

```Databases``` ```PostgreSQL``` ```CentOS```

## Introduction


Relational database management systems are a key component of many web sites and applications. They provide a structured way to store, organize, and access information.


PostgreSQL, or Postgres, is a relational database management system that provides an implementation of the SQL querying language. It is a popular choice for many small and large projects and has the advantage of being standards-compliant and having many advanced features like reliable transactions and concurrency without read locks.


In this guide, you will install Postgres on a CentOS 7 server and go over some basic ways to use it.


# Prerequisites


To follow along with this tutorial, you will need:


- A CentOS 7 server that has been configured by following our Initial Server Setup with CentOS 7 guide and our Additional Recommended Steps for New CentOS 7 Servers tutorial, including a non-root user with sudo privileges and a firewall set up with firewalld.
- To set up firewalld, follow the Configuring a Basic Firewall section of the Additional Recommended Steps for New CentOS 7 Servers tutorial.
- Databases can be particularly vulnerable to system time changes if they are very active and have timestamps on internal records. To prevent some inconsistent behavior that can arise from out-of-sync clocks, make sure to set up Network Time Protocol (NTP) synchronization by following the Configure Timezones and Network Time Protocol Synchronization section of the Additional Recommended Steps for New CentOS 7 Servers tutorial.

# Step 1 — Installing PostgreSQL


Postgres can be installed using default CentOS repositories. But as of the writing of this tutorial, the version that is available in the CentOS 7 Base repository is obsolete. Therefore, this tutorial will use the official Postgres repository.


Before you move on to setting up a new repository, exclude the search for postgresql packages from the CentOS-Base repository. Otherwise, dependencies might resolve to the postgresql supplied by the base repository.


Open the repository configuration file using your preferred text editor. This tutorial will use vim:


```
sudo vi /etc/yum.repos.d/CentOS-Base.repo


```


Find the [base] and [updates] sections, enter insert mode by pressing i, and insert the exclude=postgresql* line in both sections. As a result, your file will look like the following, with new lines highlighted:


/etc/yum.repos.d/CentOS-Base.repo
```
...
[base]
name=CentOS-$releasever - Base
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

exclude=postgresql*

#released updates
[updates]
name=CentOS-$releasever - Updates
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/updates/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
exclude=postgresql*
...

```


When you’re finished, press ESC to leave insert mode, then :wq and ENTER to save and exit the file. To learn more about the text editor vi and its successor vim, check out our Installing and Using the Vim Text Editor on a Cloud Server tutorial.


Now, install a repository configuration package using the official PostgreSQL repository for CentOS:


```
sudo yum install https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm


```


When given the prompt, confirm the installation with y.


The PostgreSQL repository includes information for all available PostgreSQL releases. You can see all available packages and versions using the following command:


```
yum list postgresql*


```


Choose and install the desired version of PostgreSQL. In this guide, you will use the PostgreSQL 11 release.


To install the PostgreSQL server use the following command:


```
sudo yum install postgresql11-server


```


During the installation process you will be asked about importing a GPG key with a prompt like the following:


```
...
Importing GPG key 0x442DF0F8:
 Userid     : "PostgreSQL RPM Building Project <pgsqlrpms-hackers@pgfoundry.org>"
 Fingerprint: 68c9 e2b9 1a37 d136 fe74 d176 1f16 d2e1 442d f0f8
 Package    : pgdg-redhat-repo-42.0-5.noarch (installed)
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-PGDG
Is this ok [y/N]:

```


Confirm it with y so the installation can complete.


Now that the software is installed, you will perform some initialization steps to prepare a new database cluster for PostgreSQL.


# Step 2 — Creating a New PostgreSQL Database Cluster


You have to create a new PostgreSQL database cluster before you can use your Postgres database. A database cluster is a collection of databases that are managed by a single server instance. Creating a database cluster consists of creating the directories in which the database data will be placed, generating the shared catalog tables, and creating the template1 and postgres databases.


The template1 database is needed to create a new database. Everything that is stored in it will be placed in a new database when it is created. A postgres database is a default database designed for use by users, utilities, and third-party applications.


Create a new PostgreSQL database cluster with initdb:


```
sudo /usr/pgsql-11/bin/postgresql-11-setup initdb


```


You will see the following output:


```
OutputInitializing database ... OK

```


Now start and enable PostgreSQL using systemctl:


```
sudo systemctl start postgresql-11
sudo systemctl enable postgresql-11


```


This will give the following output


```
OutputCreated symlink from /etc/systemd/system/multi-user.target.wants/postgresql-11.service to /usr/lib/systemd/system/postgresql-11.service.

```


Now that PostgreSQL is up and running, you will go over using roles to learn how Postgres works and how it is different from similar database management systems you may have used in the past.


# Step 3 — Using PostgreSQL Roles and Databases


By default, Postgres uses a concept called roles to handle in authentication and authorization. These are, in some ways, similar to regular Unix-style accounts, but Postgres does not distinguish between users and groups and instead prefers the more flexible term role.


Upon installation, Postgres is set up to use ident authentication, meaning that it associates Postgres roles with a matching Unix/Linux system account. If a role exists within Postgres, a Unix/Linux username with the same name is able to sign in as that role.


The installation procedure created a user account called postgres that is associated with the default Postgres role. In order to use Postgres, you can log in to that account.


There are a few ways to use this account to access Postgres.


## Switching Over to the postgres Account


Switch over to the postgres account on your server by typing:


```
sudo -i -u postgres


```


You can now access a Postgres prompt immediately by typing:


```
psql


```


This will log you into the PostgreSQL prompt, and from here you are free to interact with the database management system right away.


Exit out of the PostgreSQL prompt by typing:


```
\q


```


This will bring you back to the postgres Linux command prompt. Now return to your original sudo account with the following:


```
exit


```


## Accessing a Postgres Prompt Without Switching Accounts


You can also run the command you’d like with the postgres account directly with sudo.


For instance, in the last example, you were instructed to get to the Postgres prompt by first switching to the postgres user and then running psql to open the Postgres prompt. You could do this in one step by running the single command psql as the postgres user with sudo, like this:


```
sudo -u postgres psql


```


This will log you directly into Postgres without the intermediary bash shell.


Again, you can exit the interactive Postgres session by typing:


```
\q


```


In this step, you used the postgres account to reach the psql prompt. But many use cases require more than one Postgres role. Read on to learn how to configure new roles.


# Step 4 — Creating a New Role


Currently, you just have the postgres role configured within the database. You can create new roles from the command line with the createrole command. The --interactive flag will prompt you for the name of the new role and also ask whether it should have superuser permissions.


If you are logged in as the postgres account, you can create a new user by typing:


```
createuser --interactive


```


If, instead, you prefer to use sudo for each command without switching from your normal account, type:


```
sudo -u postgres createuser --interactive


```


The script will prompt you with some choices and, based on your responses, execute the correct Postgres commands to create a user to your specifications. For this tutorial, create a sammy user and give it superuser privileges:


```
OutputEnter name of role to add: sammy
Shall the new role be a superuser? (y/n) y

```


You can get more control by passing some additional flags. Check out the options by looking at the man page:


```
man createuser


```


Your installation of Postgres now has a new user, but you have not yet added any databases. The next section describes this process.


# Step 5 — Creating a New Database


Another assumption that the Postgres authentication system makes by default is that for any role used to log in, that role will have a database with the same name which it can access.


This means that, if the user you created in the last section is called sammy, that role will attempt to connect to a database which is also called sammy by default. You can create the appropriate database with the createdb command.


If you are logged in as the postgres account, you would type something like:


```
createdb sammy


```


If, instead, you prefer to use sudo for each command without switching from your normal account, you would type:


```
sudo -u postgres createdb sammy


```


This flexibility provides multiple paths for creating databases as needed.


Now that you’ve created a new database, you will log in to it with your new role.


# Step 6 — Opening a Postgres Prompt with the New Role


To log in with ident based authentication, you’ll need a Linux user with the same name as your Postgres role and database.


If you don’t have a matching Linux user available, you can create one with the adduser command. You will have to do this from your non-root account with sudo privileges (meaning, not logged in as the postgres user):


```
sudo adduser sammy


```


Once this new account is available, you can either switch over and connect to the database by typing:


```
sudo -i -u sammy
psql


```


Or, you can do this inline:


```
sudo -u sammy psql


```


This command will log you in automatically.


If you want your user to connect to a different database, you can do so by specifying the database like this:


```
psql -d postgres


```


Once logged in, you can check your current connection information by typing:


```
\conninfo


```


This will show the following output:


```
OutputYou are connected to database "sammy" as user "sammy" via socket in "/var/run/postgresql" at port "5432".

```


This is useful if you are connecting to non-default databases or with non-default users.


Having connected to your database, you can now try out creating and deleting tables.


# Step 7 — Creating and Deleting Tables


Now that you know how to connect to the PostgreSQL database system, you can learn some basic Postgres management tasks.


First, create a table to store some data. As an example, you will make a table that describes some playground equipment.


The basic syntax for this command is as follows:


```
CREATE TABLE table_name (
    column_name1 col_type (field_length) column_constraints,
    column_name2 col_type (field_length),
    column_name3 col_type (field_length)
);

```


These commands give the table a name, and then define the columns as well as the column type and the max length of the field data. You can also optionally add table constraints for each column.


You can learn more at our How To Create, Remove & Manage Tables in PostgreSQL on a Cloud Server tutorial.


For demonstration purposes, create a simple table like this:


```
CREATE TABLE playground (
    equip_id serial PRIMARY KEY,
    type varchar (50) NOT NULL,
    color varchar (25) NOT NULL,
    location varchar(25) check (location in ('north', 'south', 'west', 'east', 'northeast', 'southeast', 'southwest', 'northwest')),
    install_date date
);

```


These commands will create a table that inventories playground equipment. This starts with an equipment ID, which is of the serial type. This data type is an auto-incrementing integer. You’ve also given this column the constraint of primary key, which means that the values must be unique and not null.


For two of the columns (equip_id and install_date), the commands do not specify a field length. This is because some column types don’t require a set length because the length is implied by the type.


The next two commands create columns for the equipment type and color respectively, each of which cannot be empty. The command after these creates a location column and a constraint that requires the value to be one of eight possible values. The last command creates a date column that records the date on which you installed the equipment.


You can see your new table by typing:


```
\d


```


This will show the following output:


```
Output                  List of relations
 Schema |          Name           |   Type   | Owner 
--------+-------------------------+----------+-------
 public | playground              | table    | sammy
 public | playground_equip_id_seq | sequence | sammy
(2 rows)

```


Your playground table is here, but there’s also something called playground_equip_id_seq that is of the type sequence. This is a representation of the serial type that you gave your equip_id column. This keeps track of the next number in the sequence and is created automatically for columns of this type.


If you want to see just the table without the sequence, you can type:


```
\dt


```


This will yield the following:


```
Output          List of relations
 Schema |    Name    | Type  | Owner 
--------+------------+-------+-------
 public | playground | table | sammy
(1 row)

```


In this step, you created a sample table. In the next step, you will try out adding, querying, and deleting entries in that table.


# Step 8 — Adding, Querying, and Deleting Data in a Table


Now that you have a table, you can insert some data into it.


As an example, add a slide and a swing by calling the table you want to add to, naming the columns, and then providing data for each column, like this:


```
INSERT INTO playground (type, color, location, install_date) VALUES ('slide', 'blue', 'south', '2017-04-28');
INSERT INTO playground (type, color, location, install_date) VALUES ('swing', 'yellow', 'northwest', '2018-08-16');


```


You should take care when entering the data to avoid a few common hangups. For one, do not wrap the column names in quotation marks, but the column values that you enter do need quotes.


Another thing to keep in mind is that you do not enter a value for the equip_id column. This is because this is automatically generated whenever a new row in the table is created.


Retrieve the information you’ve added by typing:


```
SELECT * FROM playground;


```


You will see the following output:


```
Output equip_id | type  | color  | location  | install_date 
----------+-------+--------+-----------+--------------
        1 | slide | blue   | south     | 2017-04-28
        2 | swing | yellow | northwest | 2018-08-16
(2 rows)

```


Here, you can see that your equip_id has been filled in successfully and that all of your other data has been organized correctly.


If the slide on the playground breaks and you have to remove it, you can also remove the row from your table by typing:


```
DELETE FROM playground WHERE type = 'slide';


```


Query the table again:


```
SELECT * FROM playground;


```


You will see the following:


```
Output equip_id | type  | color  | location  | install_date 
----------+-------+--------+-----------+--------------
        2 | swing | yellow | northwest | 2018-08-16
(1 row)

```


Notice that your slide is no longer a part of the table.


Now that you’ve added and deleted entires in your table, you can try adding and deleting columns.


# Step 9 — Adding and Deleting Columns from a Table


After creating a table, you can modify it to add or remove columns. Add a column to show the last maintenance visit for each piece of equipment by typing:


```
ALTER TABLE playground ADD last_maint date;


```


If you view your table information again, you will see the new column has been added (but no data has been entered):


```
SELECT * FROM playground;


```


You will see the following:


```
Output equip_id | type  | color  | location  | install_date | last_maint 
----------+-------+--------+-----------+--------------+------------
        2 | swing | yellow | northwest | 2018-08-16   | 
(1 row)

```


Deleting a column is just as simple. If you find that your work crew uses a separate tool to keep track of maintenance history, you can delete the column by typing:


```
ALTER TABLE playground DROP last_maint;


```


This deletes the last_maint column and any values found within it, but leaves all the other data intact.


Having now added and deleted columns, you can try updating existing data in the final step.


# Step 10 — Updating Data in a Table


So far, you’ve learned how to add records to a table and how to delete them, but this tutorial hasn’t yet covered how to modify existing entries.


You can update the values of an existing entry by querying for the record you want and setting the column to the value you wish to use. You can query for the swing record (this will match every swing in your table) and change its color to red:


```
UPDATE playground SET color = 'red' WHERE type = 'swing';


```


You can verify that the operation was successful by querying the data again:


```
SELECT * FROM playground;


```


You will see the following:


```
Output equip_id | type  | color | location  | install_date 
----------+-------+-------+-----------+--------------
        2 | swing | red   | northwest | 2010-08-16
(1 row)

```


As you can see, your slide is now registered as being red.


# Conclusion


You are now set up with PostgreSQL on your CentOS 7 server. However, there is still much more to learn with Postgres. Here are some more guides that cover how to use Postgres:


- A comparison of relational database management systems
- Learn how to create and manage tables with Postgres
- Get better at managing roles and permissions
- Craft queries in Postgres with Select

