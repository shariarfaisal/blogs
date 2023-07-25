# How To Set Up Master Slave Replication on PostgreSQL on an Ubuntu 12 04 VPS

```Ubuntu``` ```PostgreSQL``` ```Load Balancing``` ```Backups```


Status: Deprecated
This article covers a version of Ubuntu that is no longer supported.  If you are currently operate a server running Ubuntu 12.04, we highly recommend upgrading or migrating to a supported version of Ubuntu:

Upgrade to Ubuntu 14.04.
Upgrade from Ubuntu 14.04 to Ubuntu 16.04
Migrate the server data to a supported version

Reason:
Ubuntu 12.04 reached end of life (EOL) on April 28, 2017 and no longer receives security patches or updates.  This guide is no longer maintained.
See Instead:
This guide might still be useful as a reference, but may not work on other Ubuntu releases.  If available, we strongly recommend using a guide written for the version of Ubuntu you are using.   You can use the search functionality at the top of the page to find a more recent version.

## Introduction



PostreSQL, or postgres, is a popular database management system that can organize and manage the data associated with websites or applications.  Replication is a means of copying database information to a second system in order to create high availability and redundancy.


There are many ways to set up replication on a postgres system.  In this tutorial, we will cover how to configure replication using a hot standby, which has the advantage of being relatively simple to configure.


To do this, we will need two Ubuntu 12.04 VPS instances.  One will serve as the master database server and the other will function as a slave, which will replicate.


# Install PostgreSQL Software



The steps in this section should be performed on both the master and slave servers.


The postgres software is available in Ubuntu’s default repositories.  Install the appropriate packages with these commands.


```
sudo apt-get update
sudo apt-get install postgresql postgresql-contrib postgresql-client

```


PostgreSQL creates a user called “postgres” in order to handle its initial databases.  We will configure ssh access between our servers to make transferring files easier.


We will need to set a password for the postgres user so that we can transfer the key files initially.  If you desire, you can remove the password at a later time:


```
sudo passwd postgres

```


Switch over to the postgres user like this:


```
sudo su - postgres

```


Generate an ssh key for the postgres user:


```
ssh-keygen

```


Press “ENTER” to all of the prompts that follow.


Transfer the keys to the other server by typing:


<pre>
ssh-copy-id <span class=“highlight”>IP_address_of_opposite_server</span>
</pre>


You should now be able to ssh freely between your two servers as the postgres user.


# Configure the Master Server



We will begin by configuring our master server.  All of these commands should be executed with the postgres user.


First, we will create a user called “rep” that can be used solely for replication:


<pre>
psql -c “CREATE USER rep REPLICATION LOGIN CONNECTION LIMIT 1 ENCRYPTED PASSWORD ‘<span class=“highlight”>yourpassword</span>’;”
</pre>


Change the password to whatever you’d like to use.


Next, we will move to the postgres configuration directory:


```
cd /etc/postgresql/9.1/main

```


We will modify the access file with the user we just created:


```
nano pg_hba.conf

```


At any place not at the bottom of the file, add a line to let the new user get access to this server:


<pre>
host    replication     rep     <span class=“highlight”>IP_address_of_slave</span>/32   md5
</pre>


Save and close the file.


Next, we will open the main postgres configuration file:


```
nano postgresql.conf

```


Find these parameters.  Uncomment them if they are commented, and modify the values according to what we have listed below:


<pre>
listen_addresses = ‘localhost,<span class=“highlight”>IP_address_of_THIS_host</span>’
wal_level = ‘hot_standby’
archive_mode = on
archive_command = ‘cd .’
max_wal_senders = 1
hot_standby = on
</pre>


Save and close the file.


Restart the master server to implement your changes:


```
service postgresql restart

```


# Configure the Slave Server



Begin on the slave server by shutting down the postgres database software:


```
service postgresql stop

```


We will be making some similar configuration changes to postgres files, so change to the configuration directory:


```
cd /etc/postgresql/9.1/main

```


Adjust the access file to allow the other server to connect to this.  This is in case we need to turn the slave into the master later on down the road.


```
nano pg_hba.conf

```


Again, add this line somewhere not at the end of the file:


<pre>
host    replication     rep     <span class=“highlight”>IP_address_of_master</span>/32  md5
</pre>


Save and close the file.


Next, open the postgres configuration file:


```
nano postgresql.conf

```


You can use the same configuration options you set for the master server, modifying only the IP address to reflect the slave server’s address:


<pre>
listen_addresses = ‘localhost,<span class=“highlight”>IP_address_of_THIS_host</span>’
wal_level = ‘hot_standby’
archive_mode = on
archive_command = ‘cd .’
max_wal_senders = 1
hot_standby = on
</pre>


Save and close the file.


# Replicating the Initial database:



Before the slave can replicate the master, we need to give it the initial database to build off of.  This is because it reads the logs off of the master server and applies the changes to its own database.  We need that database to match the master database.


On the master server, we can use an internal postgres backup start command to create a backup label command. We then will transfer the database data to our slave and then issue an internal backup stop command to clean up:


<pre>
psql -c “select pg_start_backup(‘initial_backup’);”
rsync -cva --inplace --exclude=pg_xlog /var/lib/postgresql/9.1/main/ <span class=“highlight”>slave_IP_address</span>:/var/lib/postgresql/9.1/main/
psql -c “select pg_stop_backup();”
</pre>


The rsync command may have had an error on modifying the certificate files, but this is okay for our use.  The master’s data should now be on the slave.


We now have to configure a recovery file on our slave.  On the slave navigate to the data directory:


```
cd /var/lib/postgresql/9.1/main

```


Here, we need to create a recovery file called recovery.conf:


```
nano recovery.conf

```


Fill in the following information.  Make sure to change the IP address of your master server and the password for the rep user you created:


<pre>
standby_mode = ‘on’
primary_conninfo = ‘host=<span class=“highlight”>master_IP_address</span> port=5432 user=rep password=<span class=“highlight”>yourpassword</span>’
trigger_file = ‘/tmp/postgresql.trigger.5432’
</pre>


The last line in the file, trigger_file, is one of the most interesting parts of the entire configuration.  If you create a file at that location on your slave machine, your slave will reconfigure itself to act as a master.


This will break your current replication, especially if the master server is still running, but is what you would need to do if your master server goes down.  This will allow the slave to begin accepting writes.  You can then fix the master server and turn that into the slave.


You should now have the pieces in place to start your slave server.  Type:


```
service postgresql start

```


You’ll want to check the logs to see if there are any problems.  They are located on both machines here:


```
less /var/log/postgresql/postgresql-9.1-main.log

```


You should see that it is successfully connecting to the master server.


# Test the Replication



We will see first-hand if our servers are replicating correctly by making some changes on the master server and then querying the slave.


On the master server, as the postgres user, log into the postgres system by typing:


```
psql

```


Your prompt will change to indicate that you are now communicating with the database software.


We will create a test table to create some changes:


```
CREATE TABLE rep_test (test varchar(40));

```


Now, we can insert some values into the table with the following commands:


```
INSERT INTO rep_test VALUES ('data one');
INSERT INTO rep_test VALUES ('some more words');
INSERT INTO rep_test VALUES ('lalala');
INSERT INTO rep_test VALUES ('hello there');
INSERT INTO rep_test VALUES ('blahblah');

```


You can now exit out of this interface by typing:


```
\q

```


Now, on the slave, enter the database interface in the same way:


```
psql

```


Now, we can see if the data we entered in the master database has been replicated on the slave:


```
SELECT * FROM rep_test;

```



```
      test       
-----------------
 data one
 some more words
 lalala
 hello there
 blahblah
(5 rows)

```


Excellent!  Our data has been written to both the master and slave servers.


Let’s see if we can insert more data into the table on our slave:


```
INSERT INTO rep_test VALUES ('oops');

```



```
ERROR:  cannot execute INSERT in a read-only transaction

```


As you can see, we are unable to insert data into the slave.  This is because the data is only being transferred in one direction.  In order to keep the databases consistent, postgres must make the slave read-only.


# Conclusion



You should now have a master and slave PostgreSQL server configured to communicate effectively.  If you have an application that will be writing to and querying the databases, you could set up a load balancing scheme to always write to the master, but split the reads between the master and slave.  This could increase the performance of your database interactions.


<div class=“author”>By Justin Ellingwood</div>


