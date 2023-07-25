# How To Configure MySQL Group Replication on Ubuntu 20 04

```Backups``` ```High Availability``` ```Scaling``` ```Ubuntu 20.04``` ```Databases``` ```MySQL``` ```Ubuntu```

## Introduction


MySQL replication reliably mirrors the data and operations from one database to another.  Conventional replication involves a primary server configured to accept database write operations with secondary servers that copy and apply actions from the primary server’s log to their own data sets.  These secondary servers can be used for reads, but are usually unable to execute data writes.


Group replication is a way of implementing a more flexible, fault-tolerant replication mechanism.  This process involves establishing a pool of servers that are each involved in ensuring data is copied correctly.  If the primary server experiences problems, member elections can select a new primary from the group.  This allows the remaining nodes to continue operating, even in the face of problems.  Membership negotiation, failure detection, and message delivery are provided through an implementation of the Paxos consensus algorithm.


In this tutorial, you will set up MySQL group replication using a set of three Ubuntu 20.04 servers.  Note that three is the minimum number of MySQL instances you need to deploy group replication in MySQL, while nine is the maximum.  As you work through this tutorial, you will have the option to set the group up as either a single-primary or multi-primary replication group.



Note: Database servers can have one of two roles in a replication setup: they can either be a primary instance (also known as a source instance), which users can write data to; or a replica (or secondary instance), which stores a copy of all the data on the source. Historically, these roles have instead been referred to as the master instance and the slave instance, respectively. In a blog post published in July of 2020, the MySQL team acknowledged the negative origin of this terminology and announced their efforts to update the database program and its documentation to use more inclusive language.
However, this is an ongoing process. Although MySQL’s documentation and much of the commands in version 8 of the program have been updated to instead refer to the servers in a replication topology as the primary and its secondaries (or the source and its replicas), there are places where the negative terminology still appears. This guide will default to the more inclusive terminology wherever possible, but there are a few instances where the older terms unavoidably come up.

# Prerequisites


To complete this guide, you will need:


- Three servers running Ubuntu 20.04.  Each should have a non-root administrative user with sudo privileges and a firewall configured with UFW.  Follow our initial server setup guide for Ubuntu 20.04 to set up each server.
- MySQL installed on each server.  This guide assumes that you’re using the latest version of MySQL available from the default Ubuntu repositories which, as of this writing, is version 8.0.28.  To install this on all of your servers, follow our guide on How To Install MySQL on Ubuntu 20.04 for each machine.

To help keep things clear, this guide will refer to the three servers as member1, member2, and member3.  In the examples throughout this guide, these members will have the following IP addresses:





Member
IP address




member1
203.0.113.1


member2
203.0.113.2


member3
203.0.113.3




Any commands that must be run on member1 will have a blue background, like this:


```



```


Likewise, any commands that must be run on member2 will have a red background:


```



```


And any commands that must be run on member3 will have a green background:


```



```


Lastly, any commands that must be run on each of the three servers will have a standard background:


```



```


# Step 1 — Generating a UUID to Identify the MySQL Group


Before opening the MySQL configuration file to configure the group replication settings, you need to generate a UUID that you can use to identify the MySQL group you’ll be creating.


On member1, use the uuidgen command to generate a valid UUID for the group:


```
uuidgen


```


```
Output168dcb64-7cce-473a-b338-6501f305e561

```


Copy the value you receive, since you will have to reference this in a moment when configuring a group name for your pool of servers.


# Step 2 — Setting Up Group Replication in the MySQL Configuration File


Now you are ready to modify MySQL’s configuration file.  Open up the main MySQL configuration file on each MySQL server using you preferred text editor. Here, we’ll use nano:


```
sudo nano /etc/mysql/my.cnf


```


On Ubuntu, MySQL comes installed with a number of different files you can use to define various configuration changes.  By default, the my.cnf file is only used to source additional files from subdirectories.  You will have to add your own configuration beneath the !includedir lines.  This will allow you to override any settings from the included files.


To begin, start a new section by including a [mysqld] header and then add the settings you need to enable group replication, as highlighted in the following example.  Note that these settings are modified from the minimum settings required for group replication outlined in the official MySQL documentation.  The loose- prefix allows MySQL to handle options it does not recognize gracefully and without failure.  You will need to fill in and customize some of these settings shortly:


/etc/mysql/my.cnf
```
. . .
!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/mysql.conf.d/

[mysqld]

# General replication settings
disabled_storage_engines="MyISAM,BLACKHOLE,FEDERATED,ARCHIVE,MEMORY"
gtid_mode = ON
enforce_gtid_consistency = ON
master_info_repository = TABLE
relay_log_info_repository = TABLE
binlog_checksum = NONE
log_slave_updates = ON
log_bin = binlog
binlog_format = ROW
transaction_write_set_extraction = XXHASH64
loose-group_replication_bootstrap_group = OFF
loose-group_replication_start_on_boot = OFF
loose-group_replication_ssl_mode = REQUIRED
loose-group_replication_recovery_use_ssl = 1

# Shared replication group configuration
loose-group_replication_group_name = ""
loose-group_replication_ip_whitelist = ""
loose-group_replication_group_seeds = ""

# Single or Multi-primary mode? Uncomment these two lines
# for multi-primary mode, where any host can accept writes
#loose-group_replication_single_primary_mode = OFF
#loose-group_replication_enforce_update_everywhere_checks = ON

# Host specific replication configuration
server_id = 
bind-address = ""
report_host = ""
loose-group_replication_local_address = ""

```


To explain all of these configuration options with greater clarity, they have been broken up into the following subsections.  Please read through them carefully, as some sections present you with choices about how you want to deploy your replication group or require you to enter details specific to your own configuration.


## Boilerplate Group Replication Settings


The first section contains general settings required for group replication that require no modification:


/etc/mysql/my.cnf
```
. . .
# General replication settings
disabled_storage_engines="MyISAM,BLACKHOLE,FEDERATED,ARCHIVE,MEMORY"
gtid_mode = ON
enforce_gtid_consistency = ON
master_info_repository = TABLE
relay_log_info_repository = TABLE
binlog_checksum = NONE
log_slave_updates = ON
log_bin = binlog
binlog_format = ROW
transaction_write_set_extraction = XXHASH64
loose-group_replication_bootstrap_group = OFF
loose-group_replication_start_on_boot = OFF
loose-group_replication_ssl_mode = REQUIRED
loose-group_replication_recovery_use_ssl = 1
. . .

```


One particular requirement for group replication in MySQL is that the data must be stored in the InnoDB storage engine.  The MySQL documentation recommends explicitly disabling the use of other storage engines that could cause errors in a manner like the first uncommented line in this section.


The remaining settings turn on global transaction IDs, configure the binary logging that is required for group replication, and configure SSL for the group.  This configuration also sets up a few other items that aid in recovery and bootstrapping.  You don’t need to modify anything in this section and it should be identical on all three of your servers, so you can move on after adding it in.


## Shared Group Replication Settings


The second section sets up shared settings for the group.  You will have to customize this once and then use the same settings on each of your nodes.  Specifically, you must add the group’s UUID (which you created in the previous step), a list of authorized group members, and the seed members to contact to obtain the initial data when joining the group.


Set the loose-group_replication_group_name to the UUID value you generated previously with the uuidgen command.  Make sure you place the UUID between the empty pair of double quotes.


Next, set loose-group_replication_ip_whitelist to a list of all of your MySQL server IP addresses, separated by commas.  The loose-group_replication_group_seeds setting should be almost the same as the whitelist, but should append a designated group replication port to the end of each member.  For the purposes of this guide, use the recommended group replication port, 33061:


/etc/mysql/my.cnf
```
. . .
# Shared replication group configuration
loose-group_replication_group_name = "168dcb64-7cce-473a-b338-6501f305e561"
loose-group_replication_ip_whitelist = "203.0.113.1,203.0.113.2,203.0.113.3"
loose-group_replication_group_seeds = ""203.0.113.1:33061,203.0.113.2:33061,203.0.113.3:33061"
. . .

```


This section should be the same on each of your MySQL servers, so make sure to copy it carefully on each.


## Choosing Single Primary or Multi-Primary


Next, you need to decide whether to configure a single-primary or multi-primary group.  In a single-primary configuration, MySQL designates a single primary server (almost always the first group member) to handle write operations.  A multi-primary group allows any of the group members to perform writes.


If you wish to configure a multi-primary group, uncomment the loose-group_replication_single_primary_mode and loose-group_replication_enforce_update_everywhere_checks directives.  This will set up a multi-primary group.  For a single primary group, just leave those two lines commented:


/etc/mysql/my.cnf
```
. . .
# Single or Multi-primary mode? Uncomment these two lines
# for multi-primary mode, where any host can accept writes
#loose-group_replication_single_primary_mode = OFF
#loose-group_replication_enforce_update_everywhere_checks = ON
. . .

```


These settings must be the same on each of your MySQL servers.


You can change this setting at a later time, but after doing so you must restart each member of your MySQL group.  To change over to the new configuration, you will have to stop each of the MySQL instances in the group, start each member with the new settings, and then re-bootstrap the group replication.  This will not affect any of your data, but requires a small window of downtime.


## Host-Specific Configuration Settings


The fourth section contains settings that will be different on each of the servers, including:


- The server ID
- The address to bind to
- The address to report to other members
- The local replication address and listening port

The server_id directive must be set to a unique number.  For the first member, set this to 1 and increment the number on each additional host.  Set bind-address and report_host to the respective server’s IP address so that the MySQL instance will listen for external connections and report its address correctly to other hosts.  The loose-group_replication_local_address should also be set to the current server’s IP address with the group replication port (33061), appended to the IP address.


As an example, here’s this portion of the configuration for member1 using its sample IP address:


/etc/mysql/my.cnf
```
. . .
# Host specific replication configuration
server_id = 1
bind-address = "203.0.113.1"
report_host = "203.0.113.1"
loose-group_replication_local_address = "203.0.113.1:33061"

```


Complete this process on each of your MySQL servers.  Here’s the configuration for member2:


/etc/mysql/my.cnf
```
. . .
# Host specific replication configuration
server_id = 2
bind-address = "203.0.113.2"
report_host = "203.0.113.2"
loose-group_replication_local_address = "203.0.113.2:33061"

```


And here’s the configuration for member3:


/etc/mysql/my.cnf
```
. . .
# Host specific replication configuration
server_id = 3
bind-address = "203.0.113.3"
report_host = "203.0.113.3"
loose-group_replication_local_address = "203.0.113.3:33061"

```


Be sure to update each highlighted IP address to that of the server whose configuration you’re editing.


When you are finished, double check that the shared replication settings are the same on each host and that the host-specific settings are customized for each host.  Save and close the file on each host when you’re finished.  If you used nano to edit the file, you can do so by pressing CTRL + X, Y, and then ENTER.


Each of your servers’ MySQL configuration files now contains the directives required to bootstrap MySQL group replication.  To apply the new settings to the MySQL instance, restart the service on each of your servers with the following command:


```
sudo systemctl restart mysql


```


With that, you can move on to enabling remote access by updating each of your servers’ firewall rules.


# Step 3 — Updating Each Servers’ UFW Rules


Assuming you followed the prerequisite initial server setup guide you will have set up a firewall on each of the servers on which you’ve installed MySQL and enabled access for the OpenSSH UFW profile.  This is an important security measure, as these firewalls currently block connections to any port on your servers, save for ssh connections that present keys which align with those in each server’s respective authorized_keys file.


In the MySQL configuration file, you configured the service to listen for external connections on the default port 3306.  You also defined 33061 as the port that members should use for replication coordination.


On each of your member servers, you need to open up access to both of these ports for the other members in this group so they can all communicate with one another.  To open up access to these ports on member1 for member2, run the following ufw commands on member1:


```
sudo ufw allow from member2_server_ip to any port 3306
sudo ufw allow from member2_server_ip to any port 33061


```


Be sure to change member2_server_ip to reflect your member2 server’s actual IP address.  Then, to open up the same ports for member3, run these commands:


```
sudo ufw allow from member3_server_ip to any port 3306
sudo ufw allow from member3_server_ip to any port 33061


```


Next, update the firewall rules for your other two servers.  Run the following commands on member2, making sure to change the IP addresses to reflect those of member1 and member3, respectively:


```
sudo ufw allow from member1_server_ip to any port 3306
sudo ufw allow from member1_server_ip to any port 33061
sudo ufw allow from member3_server_ip to any port 3306
sudo ufw allow from member3_server_ip to any port 33061


```


Lastly, run these two commands on member3. Again, be sure that you enter the correct IP addresses for each server:


```
sudo ufw allow from member1_server_ip to any port 3306
sudo ufw allow from member1_server_ip to any port 33061
sudo ufw allow from member2_server_ip to any port 3306
sudo ufw allow from member2_server_ip to any port 33061


```


After adding these UFW rules, each of your three MySQL instances will be allowed access to the ports used by MySQL on the other two servers.


With access to the MySQL ports open, you can now create a replication user and enable the group replication plugin.


# Step 4 — Configuring Replication Users and Enabling Group Replication Plugin


In order to establish connections with the other servers in the replication group, each MySQL instance must have a dedicated replication user.


On each of your MySQL servers, log into your MySQL instance with the administrative user to start an interactive session:


```
sudo mysql


```



Note: Be sure to run each of the commands in this section on each of your MySQL instances.

Because each server will have its own replication user, you need to turn off binary logging during the creation process.  Otherwise, once replication begins, the group would attempt to propagate the replication user from the primary to the other servers, creating a conflict with the replication user already in place.  Run the following command from the MySQL prompt on each of your servers:


```
SET SQL_LOG_BIN=0;


```


Now you can run a CREATE USER statement to create your replication user.  Run the following command, which creates a user named repl.  This command specifies that the replication user must connect using SSL.  Also, make sure to use a secure password in place of password when creating this replication user:


```
CREATE USER 'repl'@'%' IDENTIFIED BY 'password' REQUIRE SSL;


```


Next, grant the new user replication privileges on the server:


```
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';


```


Then flush the privileges to implement the changes:


```
FLUSH PRIVILEGES;


```


Following that, re-enable binary logging to resume normal operations:


```
SET SQL_LOG_BIN=1;


```


Next, set the group_replication_recovery channel to use your new replication user and their associated password.  Each server will then use these credentials to authenticate to the group:


```
CHANGE REPLICATION SOURCE TO SOURCE_USER='repl', SOURCE_PASSWORD='password' FOR CHANNEL 'group_replication_recovery';


```



Note: If you’re using a version of MySQL older than 8.0.23, you will need to use MySQL’s legacy syntax to set this up:
CHANGE MASTER TO MASTER_USER='repl', MASTER_PASSWORD='password' FOR CHANNEL 'group_replication_recovery';



With the replication user in place, you can enable the group_replication plugin to prepare to initialize the group:


```
INSTALL PLUGIN group_replication SONAME 'group_replication.so';


```


Verify that the plugin is active by running the following command:


```
SHOW PLUGINS;


```


The group_replication plugin will appear at the bottom of the list since it is the most recently added plugin:


```
Output+----------------------------+----------+--------------------+----------------------+---------+
| Name                       | Status   | Type               | Library              | License |
+----------------------------+----------+--------------------+----------------------+---------+
|                            |          |                    |                      |         |
| . . .                      | . . .    | . . .              | . . .                | . . .   |
|                            |          |                    |                      |         |
| group_replication          | ACTIVE   | GROUP REPLICATION  | group_replication.so | GPL     |
+----------------------------+----------+--------------------+----------------------+---------+
45 rows in set (0.00 sec)

```


This output confirms that the plugin was loaded and is currently active.  Before continuing on to the next step, make sure that you’ve run each command in this section on each of your MySQL instances.


# Step 5 — Starting Group Replication


Now that each MySQL server has a replication user configured and the group replication plugin enabled, you can begin to bring up your group.


## Bootstrapping the First Node


To start up the group, complete the following steps on a single member of the group.  For demonstration purposes, this guide will complete these steps on member1


Group members rely on existing members to send replication data, up-to-date membership lists, and other information when initially joining the group.  Because of this, you need to use a slightly different procedure to start up the initial group member so that it knows not to expect this information from other members in its seed list.


If set, the group_replication_bootstrap_group variable tells a member that it shouldn’t expect to receive information from peers and should instead establish a new group and elect itself the primary member.  You can turn this variable on with the following command:


```
SET GLOBAL group_replication_bootstrap_group=ON;


```


Then you can start replication for the initial group member:


```
START GROUP_REPLICATION;


```


Following that, you can set the group_replication_bootstrap_group variable back to OFF, since the only situation where this is appropriate is when there are no existing group members:


```
SET GLOBAL group_replication_bootstrap_group=OFF;


```


The group will be started with this server as the only member.  Verify this by checking the entries within the replication_group_members table in the performance_schema database:


```
SELECT * FROM performance_schema.replication_group_members;


```


This query will return a single row representing the current host:


```
Output+---------------------------+--------------------------------------+---------------+-------------+--------------+-------------+----------------+----------------------------+
| CHANNEL_NAME              | MEMBER_ID                            | MEMBER_HOST   | MEMBER_PORT | MEMBER_STATE | MEMBER_ROLE | MEMBER_VERSION | MEMBER_COMMUNICATION_STACK |
+---------------------------+--------------------------------------+---------------+-------------+--------------+-------------+----------------+----------------------------+
| group_replication_applier | 13324ab7-1b01-11e7-9dd1-22b78adaa992 | 203.0.113.1  |        3306 | ONLINE       | PRIMARY     | 8.0.28         | XCom                       |
+---------------------------+--------------------------------------+---------------+-------------+--------------+-------------+----------------+----------------------------+
1 row in set (0.00 sec)

```


The ONLINE value for MEMBER_STATE indicates that this node is fully operational within the group.


Next create a test database and table with some sample data.  Once more members are added to this group, this data will be replicated out to them automatically.


Start by creating a sample database named playground:


```
CREATE DATABASE playground;


```


Next create an example table named equipment within the playground database with the following command:


```
CREATE TABLE playground.equipment ( 
id INT NOT NULL AUTO_INCREMENT,
type VARCHAR(50),
quant INT,
color VARCHAR(25),
PRIMARY KEY(id)
);


```


This table contains the following four columns:


- id: This column will contain integer values that increment automatically, meaning you won’t have to specify values for this column when you load the table with sample data
- type: This column will contain string values describing what type of playground equipment the row represents
- quant: This column will contain integer values to represent the quantity of the given type of playground equipment
- color: This column will hold string values specifying the color of the given equipment

Also, note that the id column is specified as this table’s primary key.  In MySQL, every table replicated to a group must have a column designated as the table’s primary key.


Lastly, run the following command to insert one row of data into the table:


```
INSERT INTO playground.equipment (type, quant, color) VALUES ("slide", 2, "blue");


```


Query the table to make sure the data was entered correctly:


```
SELECT * FROM playground.equipment;


```


```
Output+----+-------+-------+-------+
| id | type  | quant | color |
+----+-------+-------+-------+
|  1 | slide |     2 | blue  |
+----+-------+-------+-------+
1 row in set (0.00 sec)

```


After verifying that this server is a member of the group and that it has write capabilities, the other servers can join the group.


## Starting Up the Remaining Nodes


Next, start group replication on member2.  Since you already have an active member, you don’t need to bootstrap the group and this member can join straightaway:


```
START GROUP_REPLICATION;


```


On member3, start group replication the same way:


```
START GROUP_REPLICATION;


```


Check the membership list again on any of the three servers.  This time, there will be three servers listed in the output:


```
SELECT * FROM performance_schema.replication_group_members;


```


```
Output
+---------------------------+--------------------------------------+---------------+-------------+--------------+-------------+----------------+----------------------------+
| CHANNEL_NAME              | MEMBER_ID                            | MEMBER_HOST   | MEMBER_PORT | MEMBER_STATE | MEMBER_ROLE | MEMBER_VERSION | MEMBER_COMMUNICATION_STACK |
+---------------------------+--------------------------------------+---------------+-------------+--------------+-------------+----------------+----------------------------+
| group_replication_applier | 13324ab7-1b01-11e7-9dd1-22b78adaa992 | 203.0.113.1  |        3306 | ONLINE       | PRIMARY     | 8.0.28         | XCom                       |
| group_replication_applier | 1ae4b211-1b01-11e7-9d89-ceb93e1d5494 | 203.0.113.2  |        3306 | ONLINE       | SECONDARY   | 8.0.28         | XCom                       |
| group_replication_applier | 157b597a-1b01-11e7-9d83-566a6de6dfef | 203.0.113.3  |        3306 | ONLINE       | SECONDARY   | 8.0.28         | XCom                       |
+---------------------------+--------------------------------------+---------------+-------------+--------------+-------------+----------------+----------------------------+
3 rows in set (0.00 sec)

```


All members should have a MEMBER_STATE value of ONLINE.  For a new group, if any of the nodes are listed as RECOVERING for more than a few seconds, it’s usually an indication that an error has occurred or something has been misconfigured.  Check the logs at /var/log/mysql/error.log to get additional information about what went wrong.


Next, check whether the test database information has been replicated over on the new members:


```
SELECT * FROM playground.equipment;


```


```
Output+----+-------+-------+-------+
| id | type  | quant | color |
+----+-------+-------+-------+
|  1 | slide |     2 | blue  |
+----+-------+-------+-------+
1 row in set (0.01 sec)

```


If the data is available on the new members, it means that group replication is working correctly.


# Step 6 — Testing Write Capabilities of New Group Members


Next, you can try writing to the database from your new replication group members.  Whether this succeeds or not is a function of whether you chose to configure a single primary or multi-primary group.


## Testing Writes in a Single Primary Environment


In a single primary group, you should expect any write operations from a non-primary server to be rejected for consistency reasons.  You can find the current primary at any time by running the following query on any member of your replication group:


```
SHOW STATUS LIKE '%primary%';


```


```
Output+----------------------------------+--------------------------------------+
| Variable_name                    | Value                                |
+----------------------------------+--------------------------------------+
| group_replication_primary_member | 13324ab7-1b01-11e7-9dd1-22b78adaa992 |
+----------------------------------+--------------------------------------+
1 row in set (0.01 sec)

```


The value of the query will be a MEMBER_ID that you can match to a host by querying the group member list like you did before:


```
SELECT * FROM performance_schema.replication_group_members;


```


```
Output+---------------------------+--------------------------------------+--------------+-------------+--------------+
| CHANNEL_NAME              | MEMBER_ID                            | MEMBER_HOST  | MEMBER_PORT | MEMBER_STATE |
+---------------------------+--------------------------------------+--------------+-------------+--------------+
| group_replication_applier | 13324ab7-1b01-11e7-9dd1-22b78adaa992 | 203.0.113.1  |        3306 | ONLINE       |
| group_replication_applier | 1ae4b211-1b01-11e7-9d89-ceb93e1d5494 | 203.0.113.2  |        3306 | ONLINE       |
| group_replication_applier | 157b597a-1b01-11e7-9d83-566a6de6dfef | 203.0.113.3  |        3306 | ONLINE       |
+---------------------------+--------------------------------------+--------------+-------------+--------------+
3 rows in set (0.01 sec)

```


As this example output indicates, the host at 203.0.113.1 — member1 — is currently the primary server.  If you attempt to write to the database from another member, the operation will fail:


```
INSERT INTO playground.equipment (type, quant, color) VALUES ("swing", 10, "yellow");


```


```
OutputERROR 1290 (HY000): The MySQL server is running with the --super-read-only option so it cannot execute this statement

```


This is expected since the group is currently configured with a single write-capable primary.  If the primary server has issues and leaves the group, the group will automatically elect a new member to be the primary and accept writes.


## Testing Writes in a Multi-Primary Environment


For groups that have been configured in a multi-primary orientation, any member should be able to commit writes to the database.


You can double-check that your group is operating in multi-primary mode by checking the value of the group_replication_primary_member variable again:


```
SHOW STATUS LIKE '%primary%';


```


```
Output+----------------------------------+-------+
| Variable_name                    | Value |
+----------------------------------+-------+
| group_replication_primary_member |       |
+----------------------------------+-------+
1 row in set (0.02 sec)

```


If the variable is empty, this means that there is no designated primary host and that any member should be able to accept writes.


Test this on member2 by attempting to write some data to the equipment table:


```
INSERT INTO playground.equipment (type, quant, color) VALUES ("swing", 10, "yellow");


```


```
OutputQuery OK, 1 row affected (0.00 sec)

```


member2 committed the write operation without any errors.


On member3, run the following query to check whether the new item was added:


```
SELECT * FROM playground.equipment;


```


```
Output+----+-------+-------+--------+
| id | type  | quant | color  |
+----+-------+-------+--------+
|  1 | slide |     2 | blue   |
|  2 | swing |    10 | yellow |
+----+-------+-------+--------+
2 rows in set (0.00 sec)

```


This confirms that the member2’s write was successfully replicated.


Now, test write capabilities on member3 by running the following INSERT statement:


```
INSERT INTO playground.equipment (type, quant, color) VALUES ("seesaw", 3, "green");


```


```
OutputQuery OK, 1 row affected (0.02 sec)

```


Back on member1, test to make sure that the write operations from both of the new members were replicated back:


```
SELECT * FROM playground.equipment;


```


```
Output+----+--------+-------+--------+
| id | type   | quant | color  |
+----+--------+-------+--------+
|  1 | slide  |     2 | blue   |
|  2 | swing  |    10 | yellow |
|  3 | seesaw |     3 | green  |
+----+--------+-------+--------+
3 rows in set (0.01 sec)

```


This confirms that replication is working in each direction and that each member is capable of performing write operations.


# Step 7 — Bringing the Group Back Up


Once the group is bootstrapped, individual members can join and leave without affecting availability, so long as there are enough members to elect primary servers.  However, if certain configuration changes are made (like switching between single and multi-primary environments), or all members of the group leave, you might need to re-bootstrap the group the same way that you did initially.


On member1, set the group_replication_bootstrap_group variable to ON:


```
SET GLOBAL GROUP_REPLICATION_BOOTSTRAP_GROUP=ON;


```


Then initialize the group:


```
START GROUP_REPLICATION;


```


Following that, you can set the group_replication_bootstrap_group variable back to OFF:


```
SET GLOBAL GROUP_REPLICATION_BOOTSTRAP_GROUP=OFF;


```


Once the first member has started the group, other members can join:


```
START GROUP_REPLICATION;


```


Follow this process for any additional members:


```
START GROUP_REPLICATION;


```


The group should now be online with all members available:


```
SELECT * FROM performance_schema.replication_group_members;


```


```
Output+---------------------------+--------------------------------------+--------------+-------------+--------------+
| CHANNEL_NAME              | MEMBER_ID                            | MEMBER_HOST  | MEMBER_PORT | MEMBER_STATE |
+---------------------------+--------------------------------------+--------------+-------------+--------------+
| group_replication_applier | 13324ab7-1b01-11e7-9dd1-22b78adaa992 | 203.0.113.1  |        3306 | ONLINE       |
| group_replication_applier | 1ae4b211-1b01-11e7-9d89-ceb93e1d5494 | 203.0.113.2  |        3306 | ONLINE       |
| group_replication_applier | 157b597a-1b01-11e7-9d83-566a6de6dfef | 203.0.113.3  |        3306 | ONLINE       |
+---------------------------+--------------------------------------+--------------+-------------+--------------+
3 rows in set (0.01 sec)

```


This process can be used to start the group again whenever necessary.


# Step 8 — Joining a Group Automatically When MySQL Starts


With the current settings, if a member server reboots, it will not automatically rejoin the group on start up.  If you want members to automatically rejoin the group, you can modify the configuration file slightly.


The setting outlined in this step is helpful when you want members to automatically join when they boot up.  However, there are some things you should be aware of.  First, this setting only affects when the MySQL instance itself is started.  If the member is removed from the group because of timeout issues, but the MySQL instance remains online, the member will not automatically rejoin.


Secondly, having this setting enabled when first bootstrapping a group can be harmful.  When there is not an existing group to join, the MySQL process will take a long while to start because it will attempt to contact other, non-existent members to initialize.  Only after a lengthy timeout will it give up and start normally.  Afterwards, you will have to use the procedure outlined above to bootstrap the group.


With the above caveats in mind, if you wish to configure nodes to join the group automatically when MySQL starts, open up the main MySQL configuration file:


```
sudo nano /etc/mysql/my.cnf


```


Inside, find the loose-group_replication_start_on_boot variable, and set it to ON:


/etc/mysql/my.cnf
```

[mysqld]
. . .
loose-group_replication_start_on_boot = ON
. . .

```


Save and close the file when you are finished.  The member should automatically attempt to join the group the next time its MySQL instance is started.


# Conclusion


By completing this tutorial, you learned how to configure MySQL group replication between three Ubuntu 20.04 servers.  For single-primary setups, the members will automatically elect a write-capable primary when necessary.  For multi-primary groups, any member can perform writes and updates.


Group replication provides a flexible replication topology that allows members to join or leave at will while simultaneously providing guarantees about data consistency and message ordering.  MySQL group replication may be a bit more complex to configure, but it provides capabilities that aren’t possible in traditional replication.


