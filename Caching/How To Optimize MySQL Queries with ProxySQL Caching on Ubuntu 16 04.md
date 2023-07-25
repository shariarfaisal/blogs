# How To Optimize MySQL Queries with ProxySQL Caching on Ubuntu 16 04

```Databases``` ```MySQL``` ```Ubuntu 16.04``` ```Caching``` ```Server Optimization``` ```Open Source```

The author selected the Free Software Foundation to receive a donation as part of the Write for DOnations program.


## Introduction


ProxySQL is a SQL-aware proxy server that can be positioned between your application and your database. It offers many features, such as load-balancing between multiple MySQL servers and serving as a caching layer for queries. This tutorial will focus on ProxySQL’s caching feature, and how it can optimize queries for your MySQL database.


MySQL caching occurs when the result of a query is stored so that, when that query is repeated, the result can be returned without needing to sort through the database. This can significantly increase the speed of common queries. But in many caching methods, developers must modify the code of their application, which could introduce a bug into the codebase. To avoid this error-prone practice, ProxySQL allows you to set up transparent caching.


In transparent caching, only database administrators need to change the ProxySQL configuration to enable caching for the most common queries, and these changes can be done through the ProxySQL admin interface. All the developer needs to do is connect to the protocol-aware proxy, and the proxy will decide if the query can be served from the cache without hitting the back-end server.


In this tutorial, you will use ProxySQL to set up transparent caching for a MySQL server on Ubuntu 16.04. You will then test its performance using mysqlslap with and without caching to demonstrate the effect of caching and how much time it can save when executing many similar queries.


# Prerequisites


Before you begin this guide you’ll need the following:


- One Ubuntu 16.04 server with at least 2 GB of RAM, set up with a non-root user with sudo privileges and a firewall, as instructed in our Ubuntu 16.04 Initial Server Setup guide.

# Step 1 — Installing and Setting Up the MySQL Server


First, you will install MySQL server and configure it to be used by ProxySQL as a back-end server for serving client queries.


On Ubuntu 16.04, mysql-server can be installed using this command:


```
sudo apt-get install mysql-server


```


Press Y to confirm the installation.


You will then be prompted for your MySQL root user password. Enter a strong password and save it for later use.


Now that you have your MySQL server ready, you will configure it for ProxySQL to work correctly. You need to add a monitor user for ProxySQL to monitor the MySQL server, since ProxySQL listens to the back-end server via the SQL protocol, rather than using a TCP connection or HTTP GET requests to make sure that the backend is running. monitor will use a dummy SQL connection to determine if the server is alive or not.


First, log in to the MySQL shell:


```
mysql -uroot -p


```


-uroot logs you in using the MySQL root user, and -p prompts for the root user’s password. This root user is different from your server’s root user, and the password is the one you entered when installing the mysql-server package.


Enter the root password and press ENTER.


Now you will create two users, one named monitor for ProxySQL and another that you will use to execute client queries and grant them the right privileges. This tutorial will name this user sammy.


Create the monitor user:


```
CREATE USER 'monitor'@'%' IDENTIFIED BY 'monitor_password';


```


The CREATE USER query is used to create a new user that can connect from specific IPs. Using % denotes that the user can connect from any IP address. IDENTIFIED BY sets the password for the new user; enter whatever password you like, but make sure to remember it for later use.


With the user monitor created, next make the sammy user:


```
CREATE USER 'sammy'@'%' IDENTIFIED BY 'sammy_password';


```


Next, grant privileges to your new users. Run the following command to configure monitor:


```
GRANT SELECT ON sys.* TO 'monitor'@'%';


```


The GRANT query is used to give privileges to users. Here you granted only SELECT on all tables in the sys database to the monitor user; it only needs this privilege to listen to the back-end server.


Now grant all privileges to all databases to the user sammy:


```
GRANT ALL PRIVILEGES on *.* TO 'sammy'@'%';


```


This will allow sammy to make the necessary queries to test your database later.


Apply the privilege changes by running the following:


```
FLUSH PRIVILEGES;


```


Finally, exit the mysql shell:


```
exit;


```


You’ve now installed mysql-server and created a user to be used by ProxySQL to monitor your MySQL server, and another one to execute client queries. Next you will install and configure ProxySQL.


# Step 2 — Installing and Configuring ProxySQL Server


Now you can install ProxySQL server, which will be used as a caching layer for your queries. A caching layer exists as a stop between your application servers and database back-end servers; it is used to connect to the database and to save the results of some queries in its memory for fast access later.


The ProxySQL releases Github page offers installation files for common Linux distributions. For this tutorial, you will use wget to download the ProxySQL version 2.0.4 Debian installation file:


```
wget https://github.com/sysown/proxysql/releases/download/v2.0.4/proxysql_2.0.4-ubuntu16_amd64.deb


```


Next, install the package using dpkg:


```
sudo dpkg -i proxysql_2.0.4-ubuntu16_amd64.deb


```


Once it is installed, start ProxySQL with this command:


```
sudo systemctl start proxysql


```


You can check if ProxySQL started correctly with this command:


```
sudo systemctl status proxysql


```


You will get an output similar to this:


```
Outputroot@ubuntu-s-1vcpu-2gb-sgp1-01:~# systemctl status proxysql
● proxysql.service - LSB: High Performance Advanced Proxy for MySQL
   Loaded: loaded (/etc/init.d/proxysql; bad; vendor preset: enabled)
   Active: active (exited) since Wed 2019-06-12 21:32:50 UTC; 6 months 7 days ago
     Docs: man:systemd-sysv-generator(8)
    Tasks: 0
   Memory: 0B
      CPU: 0

```


Now it is time to connect your ProxySQL server to the MySQL server. For this purpose, use the ProxySQL admin SQL interface, which by default listens to port 6032 on localhost and has admin as its username and password.


Connect to the interface by running the following:


```
mysql -uadmin -p -h 127.0.0.1 -P6032


```


Enter admin when prompted for the password.


-uadmin sets the username as admin, and the -h flag specifies the host as localhost. The port is 6032, specified using the -P flag.


Here you had to specify the host and port explicitly because, by default, the MySQL client connects using a local sockets file and port 3306.


Now that you are logged into the mysql shell as admin, configure the monitor user so that ProxySQL can use it. First, use standard SQL queries to set the values of two global variables:


```
UPDATE global_variables SET variable_value='monitor' WHERE variable_name='mysql-monitor_username';
UPDATE global_variables SET variable_value='monitor_password' WHERE variable_name='mysql-monitor_password';


```


The variable mysql-monitor_username specifies the MySQL username that will be used to check if the back-end server is alive or not. The variable mysql-monitor_password points to the password that will be used when connecting to the back-end server. Use the password you created for the monitor username.


Every time you create a change in the ProxySQL admin interface, you need to use the right LOAD command to apply changes to the running ProxySQL instance. You changed MySQL global variables, so load them to RUNTIME to apply changes:


```
LOAD MYSQL VARIABLES TO RUNTIME;


```


Next, SAVE the changes to the on-disk database to persist changes between restarts. ProxySQL uses its own SQLite local database to store its own tables and variables:


```
SAVE MYSQL VARIABLES TO DISK;


```


Now, you will tell ProxySQL about the back-end server. The table mysql_servers holds information about each back-end server where ProxySQL can connect and execute queries, so add a new record using a standard SQL INSERT statement with the following values for hostgroup_id, hostname, and port:


```
INSERT INTO mysql_servers(hostgroup_id, hostname, port) VALUES (1, '127.0.0.1', 3306);


```


To apply the changes, run LOAD and SAVE again:


```
LOAD MYSQL SERVERS TO RUNTIME;
SAVE MYSQL SERVERS TO DISK;


```


Finally, you will tell ProxySQL which user will connect to the back-end server; set sammy as the user, and replace sammy_password with the password you created earlier:


```
INSERT INTO mysql_users(username, password, default_hostgroup) VALUES ('sammy', 'sammy_password', 1);


```


The table mysql_users holds information about users used to connect to the back-end servers; you specified the username, password, and default_hostgroup.


LOAD and SAVE the changes:


```
LOAD MYSQL USERS TO RUNTIME;
SAVE MYSQL USERS TO DISK;


```


Then exit the mysql shell:


```
exit;


```


To test that you can connect to your back-end server using ProxySQL, execute the following test query:


```
mysql -usammy -h127.0.0.1 -p -P6033 -e "SELECT @@HOSTNAME as hostname"


```


In this command, you used the -e flag to execute a query and close the connection. The query prints the hostname of the back-end server.



Note: ProxySQL uses port 6033 by default for listening to incoming connections.

The output will look like this, with your_hostname replaced by your hostname:


```
Output+----------------------------+
| hostname                   |
+----------------------------+
| your_hostname        |
+----------------------------+

```


To learn more about ProxySQL configuration, see Step 3 of How To Use ProxySQL as a Load Balancer for MySQL on Ubuntu 16.04.


So far, you configured ProxySQL to use your MySQL server as a backend and connected to the backend using ProxySQL. Now, you are ready to use mysqlslap to benchmark the query performance without caching.


# Step 3 — Testing Using mysqlslap Without Caching


In this step, you will download a test database so you can execute queries against it with mysqlslap to test the latency without caching, setting a benchmark for the speed of your queries. You will also explore how ProxySQL keeps records of queries in the stats_mysql_query_digest table.


mysqlslap is a load emulation client that is used as a load testing tool for MySQL. It can test a MySQL server with auto-generated queries or with some custom queries executed on a database. It comes installed with the MySQL client package, so you do not need to install it; instead, you will download a database for testing purposes only, on which you can use mysqlslap.


In this tutorial, you will use a sample employee database. You will be using this employee database because it features a large data set that can illustrate differences in query optimization. The database has six tables, but the data it contains has more than 300,000 employee records. This will help you emulate a large-scale production workload.


To download the database, first clone the Github repository using this command:


```
git clone https://github.com/datacharmer/test_db.git


```


Then enter the test_db directory and load the database into the MySQL server using these commands:


```
cd test_db
mysql -uroot -p < employees.sql


```


This command uses shell redirection to read the SQL queries in employees.sql file and execute them on the MySQL server to create the database structure.


You will see output like this:


```
OutputINFO
CREATING DATABASE STRUCTURE
INFO
storage engine: InnoDB
INFO
LOADING departments
INFO
LOADING employees
INFO
LOADING dept_emp
INFO
LOADING dept_manager
INFO
LOADING titles
INFO
LOADING salaries
data_load_time_diff
00:00:32

```


Once the database is loaded into your MySQL server, test that mysqlslap is working with the following query:


```
mysqlslap -usammy -p -P6033 -h127.0.0.1  --auto-generate-sql --verbose


```


mysqlslap has similar flags to the mysql client; here are the ones used in this command:


- -u specifies the user used to connect to the server.
- -p prompts for the user’s password.
- -P connects using the specified port.
- -h connects to the specified host.
- --auto-generate-sql lets MySQL perform load testing using its own generated queries.
- --verbose makes the output show more information.

You will get output similar to the following:


```
OutputBenchmark
    Average number of seconds to run all queries: 0.015 seconds
    Minimum number of seconds to run all queries: 0.015 seconds
    Maximum number of seconds to run all queries: 0.015 seconds
    Number of clients running queries: 1
    Average number of queries per client: 0

```


In this output, you can see the average, minimum, and maximum number of seconds spent to execute all queries. This gives you an indication about the amount of time needed to execute the queries by a number of clients. In this output, only one client was used to execute queries.


Next, find out what queries mysqlslap executed in the last command by looking at ProxySQL’s stats_mysql_query_digest. This will give us information like the digest of the queries, which is a normalized form of the SQL statement that can be referenced later to enable caching.


Enter the ProxySQL admin interface with this command:


```
mysql -uadmin -p -h 127.0.0.1 -P6032


```


Then execute this query to find information in the stats_mysql_query_digest table:


```
SELECT count_star,sum_time,hostgroup,digest,digest_text FROM stats_mysql_query_digest ORDER BY sum_time DESC;


```


You will see output similar to the following:


```
+------------+----------+-----------+--------------------+----------------------------------+
| count_star | sum_time | hostgroup | digest             | digest_text                      |
+------------+----------+-----------+--------------------+----------------------------------+
| 1          | 598      | 1         | 0xF8F780C47A8D1D82 | SELECT @@HOSTNAME as hostname    |
| 1          | 0        | 1         | 0x226CD90D52A2BA0B | select @@version_comment limit ? |
+------------+----------+-----------+--------------------+----------------------------------+
2 rows in set (0.01 sec)

```


The previous query selects data from the stats_mysql_query_digest table, which contains information about all executed queries in ProxySQL. Here you have five columns selected:


- count_star: The number of times this query was executed.
- sum_time: Total time in milliseconds that this query took to execute.
- hostgroup: The hostgroup used to execute the query.
- digest: A digest of the executed query.
- digest_text: The actual query. In this tutorial’s example, the second query is parameterized using ? marks in place of variable parameters. select @@version_comment limit 1 and select @@version_comment limit 2, therefore, are grouped together as the same query with the same digest.

Now that you know how to check query data in the stats_mysql_query_digest table, exit the mysql shell:


```
exit;


```


The database you downloaded contains some tables with demo data. You will now test queries on the dept_emp table by selecting any records whose from_date is greater than 2000-04-20 and recording the average execution time.


Use this command to run the test:


```
mysqlslap -usammy -P6033 -p -h127.0.0.1  --concurrency=100 --iterations=20 --create-schema=employees --query="SELECT * from dept_emp WHERE from_date>'2000-04-20'" --verbose


```


Here you are using some new flags:


- --concurrency=100: This sets the number of users to simulate, in this case 100.
- --iterations=20: This causes the test to run 20 times and calculate results from all of them.
- --create-schema=employees: Here you selected the employees database.
- --query="SELECT * from dept_emp WHERE from_date>'2000-04-20'": Here you specified the query executed in the test.

The test will take a few minutes. After it is done, you will get results similar to the following:


```
OutputBenchmark
        Average number of seconds to run all queries: 18.117 seconds
        Minimum number of seconds to run all queries: 8.726 seconds
        Maximum number of seconds to run all queries: 22.697 seconds
        Number of clients running queries: 100
        Average number of queries per client: 1

```


Your numbers could be a little different. Keep these numbers somewhere in order to compare them with the results from after you enable caching.


After testing ProxySQL without caching, it is time to run the same test again, but this time with caching enabled.


# Step 4 — Testing Using mysqlslap With Caching


In this step, caching will help us to decrease latency when executing similar queries. Here, you will identify the queries executed, take their digests from ProxySQL’s stats_mysql_query_digest table, and use them to enable caching. Then, you will test again to check the difference.


To enable caching, you need to know the digests of the queries that will be cached. Log in to the ProxySQL admin interface using this command:


```
mysql -uadmin -p -h127.0.0.1 -P6032


```


Then execute this query again to get a list of queries executed and their digests:


```
SELECT count_star,sum_time,hostgroup,digest,digest_text FROM stats_mysql_query_digest ORDER BY sum_time DESC;


```


You will get a result similar to this:


```
Output+------------+-------------+-----------+--------------------+------------------------------------------+
| count_star | sum_time    | hostgroup | digest             | digest_text                              |
+------------+-------------+-----------+--------------------+------------------------------------------+
| 2000       | 33727110501 | 1         | 0xC5DDECD7E966A6C4 | SELECT * from dept_emp WHERE from_date>? |
| 1          | 601         | 1         | 0xF8F780C47A8D1D82 | SELECT @@HOSTNAME as hostname            |
| 1          | 0           | 1         | 0x226CD90D52A2BA0B | select @@version_comment limit ?         |
+------------+-------------+-----------+--------------------+------------------------------------------+
3 rows in set (0.00 sec)

```


Look at the first row. It is about a query that was executed 2000 times. This is the benchmarked query executed previously. Take its digest and save it to be used in adding a query rule for caching.


The next few queries will add a new query rule to ProxySQL that will match the digest of the previous query and put a cache_ttl value for it. cache_ttl is the number of milliseconds that the result will be cached in memory:


```
INSERT INTO mysql_query_rules(active, digest, cache_ttl, apply) VALUES(1,'0xC5DDECD7E966A6C4',2000,1);


```


In this command you are adding a new record to the mysql_query_rules table; this table holds all the rules applied before executing a query. In this example, you are adding a value for the cache_ttl column that will cause the matched query by the given digest to be cached for a number of milliseconds specified in this column. You put 1 in the apply column to make sure that the rule is applied to queries.


LOAD and SAVE these changes, then exit the mysql shell:


```
LOAD MYSQL QUERY RULES TO RUNTIME;
SAVE MYSQL QUERY RULES TO DISK;
exit;


```


Now that caching is enabled, re-run the test again to check the result:


```
mysqlslap -usammy -P6033 -p -h127.0.0.1  --concurrency=100 --iterations=20 --create-schema=employees --query="SELECT * from dept_emp WHERE from_date>'2000-04-20'" --verbose


```


This will give output similar to the following:


```
OutputBenchmark
        Average number of seconds to run all queries: 7.020 seconds
        Minimum number of seconds to run all queries: 0.274 seconds
        Maximum number of seconds to run all queries: 23.014 seconds
        Number of clients running queries: 100
        Average number of queries per client: 1

```


Here you can see the big difference in average execution time: it dropped from 18.117 seconds to 7.020.


# Conclusion


In this article, you set up transparent caching with ProxySQL to cache database query results. You also tested the query speed with and without caching to see the difference that caching can make.


You’ve used one level of caching in this tutorial. You could also try, web caching, which sits in front of a web server and caches the responses to similar requests, sending the response back to the client without hitting the back-end servers. This is very similar to ProxySQL caching but at a different level. To learn more about web caching, check out our Web Caching Basics: Terminology, HTTP Headers, and Caching Strategies primer.


MySQL server also has its own query cache; you can learn more about it in our How To Optimize MySQL with Query Cache on Ubuntu 18.04 tutorial.


